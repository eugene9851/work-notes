# AI Summary 동시성 문제 해결을 위한 Azure Service Bus Session 도입

## 문제 정의

초기 구조에서는 Azure Queue 기반으로
AI Summary 요청을 비동기 처리하고,
Worker 단에서 id 기준 Lock을 획득하는 방식을 사용했다.

그러나 다음과 같은 문제가 발생했다.

- 동일 환자(patient)에 대한 여러 메시지가
  거의 동시에 dequeue 됨
- 여러 Worker가 동일 patientId에 대해 Lock 획득 시도
- Lock 충돌로 인해 메시지가 retry 상태로 전환
- retry 횟수(5회)를 초과한 메시지가 poison queue로 이동
- 실제로는 오류가 아닌데도 불필요한 실패로 처리됨

이는 **동시성 제어를 애플리케이션 레벨에만 의존한 구조의 한계**였다.

---

## 요구사항 재정의

문제 분석을 통해 요구사항을 다음과 같이 명확히 재정의했다.

1. 동일 patientId에 대한 메시지는 **반드시 순차 처리**
2. 서로 다른 patientId에 대한 메시지는 **병렬 처리 가능**
3. 메시지 처리 순서 보장
4. Lock 충돌로 인한 불필요한 retry 및 poison queue 방지
5. 전체 처리 지연 최소화
6. Worker 수평 확장 가능 구조

---

## 해결책 개요

### Azure Queue → Azure Service Bus + Session

위 요구사항을 충족하기 위해
메시징 인프라를 **Azure Service Bus + Session** 기반으로 변경했다.

Service Bus Session을 사용하면
메시지 그룹 단위의 순차 처리를
애플리케이션이 아닌 **인프라 레벨에서 보장**할 수 있다.

---

## 핵심 개념

| 개념      | 설명                                |
| --------- | ----------------------------------- |
| Session   | 동일한 SessionId를 가진 메시지 그룹 |
| SessionId | `patientId` 사용                    |
| 처리 보장 | 동일 SessionId → 순차 처리          |
| 병렬성    | 서로 다른 SessionId → 병렬 처리     |

즉,

- 같은 patientId → 같은 Session
- 다른 patientId → 다른 Session

---

## Worker 처리 흐름

1. Worker는 Service Bus에서 **Session을 하나씩 획득**
2. 획득한 Session 내 메시지를 **순차적으로 처리**
3. Session 내 메시지 처리가 완료되면 Session 반환
4. 다른 Worker들은 동시에 다른 Session을 병렬 처리

이 구조를 통해:

- 동일 환자에 대한 동시 처리 문제 제거
- Lock 충돌 및 불필요한 retry 제거
- Worker 수에 따른 자연스러운 병렬 확장 가능

---

## 기존 방식 대비 개선점

### 기존 (Queue + Lock)

- Lock 충돌 가능성 존재
- retry 증가
- poison queue 발생 가능
- 동시성 제어 로직이 애플리케이션에 집중됨

### 개선 후 (Service Bus + Session)

- 환자 단위 순차성 인프라 레벨 보장
- retry 및 poison queue 최소화
- 코드 복잡도 감소
- 안정적인 병렬 처리 구조 확보

---

## 정리

- 문제의 본질은 AI 처리 성능이 아니라 **데이터 정합성**
- 동일 환자 단위 상태 변경은 반드시 직렬화 필요
- Azure Service Bus Session은
  이러한 요구사항을 가장 자연스럽게 충족하는 선택이었다
- 결과적으로 안정성, 확장성, 운영 복잡도를 모두 개선할 수 있었다
