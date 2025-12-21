# User Management 분리 이후 DICOM 업로드 회귀 테스트

## 테스트 목적

User Management DB 분리 및 쿼리 수정 이후,
DICOM 업로드 관련 주요 기능이
정상적으로 동작하는지 검증하기 위함이다.

## 테스트 시나리오

### 1. 신규 업로드 (New Upload)

**조건**

- 존재하지 않는 study에 대해 DICOM 파일 업로드

**기대 결과**

- 새로운 study가 생성됨
- Study List에 정상적으로 표시됨

---

### 2. 기존 Study 업로드 (Update Upload)

**조건**

- 이미 존재하는 study에 대해 DICOM 파일 업로드

**기대 결과**

- 새로운 study는 생성되지 않음
- 기존 study에 하위 데이터만 추가됨

---

### 3. Study 삭제

**조건**

- Delete Study 선택 후 삭제 버튼 클릭

**기대 결과**

- 해당 study 삭제
- 연관된 하위 데이터도 함께 삭제됨

---

## 비고

- 각 테스트 시나리오는 화면 녹화로 검증
- 구조 변경 이후 주요 사용자 플로우에 대한 회귀 테스트를 중점으로 수행
