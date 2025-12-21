# User Management 분리 이후 DICOM 업로드 실패 이슈

## 이슈 개요

User Management 시스템을 기존 DB에서 분리한 이후,
db에 업로드된 DICOM 정보를 db에도 저장하는 기능이 있는 function app 서비스에서
DICOM 파일 업로드가 정상적으로 동작하지 않는 문제가 발생했다.

업로드 요청 자체는 발생했지만,
내부 처리 과정에서 업로드 대상 DICOM 서비스 정보를 찾지 못해
function app 호출에 실패했다.

## 영향받은 서비스

- MPRT-DICOM-INTEGRATOR

## 증상

### 사용자 관점

- DICOM 파일 업로드 시 실패
- 업로드 실패에 대한 명확한 에러 메시지 없음

### 서버 / 서비스 관점

- 업로드 요청은 정상적으로 수신됨
- 내부적으로 organization 정보 조회 실패
- dicomservices 정보 조회 실패
- 결과적으로 업로드 대상 DICOM 서비스 엔드포인트를 결정하지 못함

## 발생 시점

- User Management 시스템이 기존 DB에서 분리된 이후
- `organization`, `dicomservices` 관련 데이터가
  기존 DB 구조에서 별도 DB / 서비스로 이관된 이후 발생

## 원인 분석

기존 구조에서는 `MPRT-DICOM-INTEGRATOR`가
동일한 DB 내에서 `organization`, `dicomservices`, `studies` 등의 정보를
직접 조회하는 구조였다.

User Management 분리 이후:

- 관련 데이터는 별도의 DB / 서비스에 존재
- 기존 Beanie 쿼리는 이전 DB 구조를 전제로 작성되어 있었음
- 이로 인해 내부 조회 결과가 `null`로 반환됨

## 해결 방법

- `MPRT-DICOM-INTEGRATOR`의 데이터 접근 로직 수정
- `organization`, `dicomservices` 조회 경로를
  분리된 User Management DB / 서비스 기준으로 변경
- 업로드 대상 DICOM 서비스 엔드포인트 결정 로직 정상화

## 정리하며 느낀 점

- DB 분리는 단순 데이터 이관 문제가 아님
- 서비스 간 경계가 바뀌면 쿼리 전제 조건도 반드시 재검토해야 함
- 암묵적으로 공유되던 DB 구조는 분리 시 가장 큰 리스크가 됨
