# AI Summary 동시성 제어 및 메시징 구조 결정

## 요약

여러 사용자가 동일 환자(patient)에 대해
동시에 AI Summary 생성을 요청할 경우,
summary 결과를 저장하는 과정에서
데이터 충돌(Data Race Condition)이 발생할 수 있다.

이를 해결하기 위해 비동기 처리 및 순차 처리를 도입했으며,
초기에는 Azure Queue Storage를 검토했으나
요구사항을 충족하기에 한계가 있어
최종적으로 Azure Service Bus를 사용한 구조로 결정했다.

이 문서는 **동시성 문제 정의 → 기술 선택 → 최종 처리 흐름**을 정리한 기록이다.

---

## 핵심 문제

- AI 요청은 병렬 처리 가능
- 동일 환자에 대한 summary 업데이트는 순차 처리 필요
- 환자 단위 데이터 일관성 보장이 핵심 요구사항

---

## 최종 선택 요약

- 메시징 시스템: **Azure Service Bus**
- 처리 방식:
  - 환자 단위 순차성 보장
  - 환자 간 요청 병렬 처리
- 목적:
  - 데이터 정합성 확보
  - 확장성과 운영 안정성 확보

---

## 관련 문서

- `architecture.md`  
  → 최종 AI Summary 처리 흐름 및 구조
- `decision.md`  
  → Azure Queue → Azure Service Bus 선택 이유와 비교
