# Azure Service Bus 기반 AI Summary 처리 흐름

## 배경

동일 환자(patient)에 대해
여러 AI Summary 요청이 동시에 발생할 경우,
summary 결과를 저장하는 과정에서
Data Race Condition이 발생할 수 있다.

이를 방지하기 위해
AI Summary 생성 요청을 비동기로 처리하고,
환자 단위로 순차성을 보장하는 구조를 설계했다.

---

## 최종 처리 흐름

1. 사용자 액션으로 AI Summary 요청 발생
2. 요청 메시지를 Azure Service Bus에 전송
3. Azure Function App이 메시지 수신
4. 메시지 처리 시 `sessionId` 기준으로 동시성 제어
5. AI Summary 생성 수행
6. summary 결과를 patient 데이터에 반영
7. 처리 완료 후 메시지 완료(Complete)

---

## 동시성 제어 기준

- 동일 patientId에 대한 요청:
  - 순차적으로 처리
- 서로 다른 patientId에 대한 요청:
  - 병렬 처리 허용

이를 통해:

- 데이터 정합성 유지
- 불필요한 전체 직렬 처리 방지
