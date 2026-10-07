---
name: be-deep-test
description: FAST Trace에서 이미 확정된 ServiceImpl 파일과 Method를 직접 입력받아 URL, Controller, Service 재탐색 없이 해당 Backend Method 내부 실행 흐름만 깊게 분석한다.
argument-hint: "<ServiceImpl 상대경로> <Method명>"
allowed-tools: Grep, Read
---

# BE Deep Test v0.1

## 1. 목적

FAST Trace에서 이미 확보한
ServiceImpl 파일과 Method를 직접 입력받는다.

URL부터 다시 분석하지 않는다.

Controller를 다시 찾지 않는다.

Service Interface를 다시 찾지 않는다.

Mapper XML과 SQL도
Deep Trace의 첫 단계에서는 다시 찾지 않는다.

목적:

확정된 ServiceImpl Method 내부에서
실제 실행 흐름을 깊게 추적하는 데
얼마나 시간이 걸리는지 측정한다.

---

## 2. 입력

두 값을 입력한다.

SERVICE_IMPL_FILE

SERVICE_METHOD

예:

gipms-api-material/src/main/java/com/company/material/service/impl/MaterialServiceImpl.java

createMaterial

---

## 3. 입력 Source 신뢰

SERVICE_IMPL_FILE은
FAST Trace에서 이미 확정된 Source이다.

따라서 파일을 다시 찾지 않는다.

금지:

- 파일명 Repository 검색
- ServiceImpl 후보 검색
- Service Interface 검색
- Controller 검색
- Backend URL 검색

입력된 파일을 바로 Read한다.

---

## 4. 분석 시작

즉시 출력한다.

[BE-DEEP] START

ServiceImpl File:
{SERVICE_IMPL_FILE}

Method:
{SERVICE_METHOD}

---

## 5. Source Boundary

주 분석 대상:

{SERVICE_IMPL_FILE}

추가 Source 탐색은
현재 Method에서 실제 호출된 대상에 한해서만 허용한다.

Java:

gipms-api-*/src/main/java/**

---

## 6. Hard Exclude

다음은 탐색하지 않는다.

- **/*.jar
- **/*.class
- **/target/**
- **/build/**
- **/.gradle/**
- **/.m2/**
- **/libs/**
- **/lib/**
- **/BOOT-INF/**
- **/WEB-INF/lib/**
- **/node_modules/**
- **/generated/**
- **/test/**
- **/tests/**
- **/sample/**
- **/samples/**
- docs/**
- .git/**

---

## 7. 이번 버전의 분석 범위

이번 DEEP v0.1에서는 다음을 분석한다.

- Entry ServiceImpl Method
- Method 내부 실행 순서
- 조건문
- Validation 성격의 조건
- Local/private Method 호출
- 다른 Business Service 호출
- Mapper 호출
- External/SAP/RFC 호출 존재 여부
- Return
- Throw

단:

다른 Business Service 내부로는 아직 들어가지 않는다.

External/SAP/RFC 내부로도 들어가지 않는다.

Mapper XML/SQL도 다시 분석하지 않는다.

---

## 8. 핵심 원칙

이번 Skill은:

찾는 Skill

이 아니라:

읽는 Skill

이다.

이미 FAST Trace에서
분석 시작 Source가 확정되어 있다.

따라서 Repository 탐색보다
입력된 Method의 실행 흐름 분석을 우선한다.

---

# PHASE 1. Entry Method

## 9. Method 찾기

입력된 SERVICE_IMPL_FILE 안에서만
SERVICE_METHOD 선언을 찾는다.

Repository 전체 Grep을 하지 않는다.

정확한 Method 이름을 사용한다.

---

## 10. Method Read

Method 시작 위치에서:

약 100줄

을 먼저 Read한다.

Method가 끝나지 않으면
필요한 만큼만 추가 Read한다.

Method 종료가 확인되면
즉시 Read를 중단한다.

---

## 11. Entry Method에서 수집

실행 순서대로 다음을 수집한다.

- 조건문
- 값 설정
- Validation
- Local Method 호출
- Business Service 호출
- Mapper 호출
- External 호출
- Return
- Throw

---

## 12. CHECKPOINT 1

Entry Method 분석이 완료되면 출력한다.

[BE-DEEP] CHECKPOINT 1/3 - ENTRY METHOD READ

Method:
{SERVICE_METHOD}

Local Calls:
{COUNT}

Business Service Calls:
{COUNT}

Mapper Calls:
{COUNT}

External Calls:
{COUNT}

---

# PHASE 2. Local Method

## 13. Local Method 대상

Entry Method에서 실제 호출된 Method 중:

동일 SERVICE_IMPL_FILE 안에
Method 선언이 존재하는 경우에만

Local Method로 처리한다.

---

## 14. Local Method 검색 범위

Local Method를 찾을 때:

SERVICE_IMPL_FILE

하나만 사용한다.

Repository 전체 Grep 금지.

다른 Java 파일 검색 금지.

---

## 15. Local Method 탐색 최적화

Local Method 후보마다
무조건 Repository 검색을 하지 않는다.

현재 Method에서 실제 호출된 이름만 사용한다.

정확한 Method 이름으로
SERVICE_IMPL_FILE 안에서만 선언 위치를 찾는다.

---

## 16. Local Method Read

확정된 Local Method의 시작 위치에서:

약 80줄

을 먼저 Read한다.

Method 종료가 보이지 않을 때만
추가 Read한다.

---

## 17. Recursive Local Trace

Local Method 안에서
또 다른 Local Method가 실제 호출되면
동일한 규칙으로 따라간다.

예:

createMaterial()

→ validateMaterial()

→ validatePlant()

→ checkPlant()

같은 파일 안에 실제 선언되어 있으면
계속 추적한다.

---

## 18. VISITED

분석한 Local Method는:

VISITED_LOCAL_METHODS

에 기록한다.

같은 Method는 다시 Read하지 않는다.

---

## 19. 순환 호출

예:

methodA()

→ methodB()

→ methodA()

이면:

methodA

를 두 번째로 발견했을 때
다시 Read하지 않는다.

호출 관계만 기록한다.

---

## 20. Local Method에서 수집

다음만 수집한다.

- 조건문
- Validation
- Local Method 호출
- Business Service 호출
- Mapper 호출
- External 호출
- Return
- Throw

---

## 21. CHECKPOINT 2

Local Method 추적이 끝나면 출력한다.

[BE-DEEP] CHECKPOINT 2/3 - LOCAL TRACE COMPLETE

Local Method Count:
{COUNT}

Local Flow:

{ENTRY_METHOD}
→ {LOCAL_METHOD}
→ {LOCAL_METHOD}

Local Method가 없으면:

Local Method Count:
0

Local Flow:
NONE

---

# PHASE 3. Call Classification

## 22. Business Service

현재까지 읽은 Method에서
다른 Business Service 호출이 보이면 기록한다.

예:

materialCheckService.validate(...)

historyService.save(...)

이번 버전에서는 내부 Source로 들어가지 않는다.

기록:

Business Service:
{VARIABLE}.{METHOD}

---

## 23. Mapper

Mapper 호출이 보이면 기록한다.

예:

materialMapper.selectMaterial(...)

historyMapper.insertHistory(...)

이번 버전에서는:

Mapper Variable
Mapper Method
Called From

만 기록한다.

XML과 SQL은 FAST Trace 결과를 재사용하는 것을 전제로 한다.

XML을 다시 찾지 않는다.

---

## 24. External / SAP / RFC

다음과 같은 호출이 보이면 기록한다.

sapService.*

rfc*

externalClient.*

httpClient.*

restTemplate.*

webClient.*

또는 Source상 명확한 외부 연동 객체

이번 버전에서는 내부로 들어가지 않는다.

---

## 25. Validation

다음과 같은 실제 실행 조건을 기록한다.

if

switch

null check

empty check

값 비교

상태 비교

예외 Throw 조건

Validation Library 호출

단순히 모든 if를 Validation이라고 부르지 않는다.

입력값 또는 업무 수행 가능 여부를 검사하는 조건만
Validation으로 분류한다.

---

## 26. Branch

조건에 따라
서로 다른 실행 경로가 존재하면 기록한다.

예:

if A

→ Mapper A

else

→ Mapper B

실제 Source 기준으로 작성한다.

---

## 27. Return

Method의 최종 Return을 확인한다.

예:

return result

return response

return null

void

---

## 28. Exception

직접 확인되는:

throw

catch

예외 변환

만 기록한다.

관련 Exception Class를
Repository 전체에서 추가 분석하지 않는다.

---

## 29. CHECKPOINT 3

분류가 끝나면 출력한다.

[BE-DEEP] CHECKPOINT 3/3 - FLOW CLASSIFIED

Validation:
{COUNT}

Branches:
{COUNT}

Business Services:
{COUNT}

Mapper Calls:
{COUNT}

External Calls:
{COUNT}

Throw:
{COUNT}

---

# PHASE 4. Final Result

## 30. 실행 흐름 출력

실제 Source 순서에 맞춰 출력한다.

예:

ENTRY

createMaterial()

↓

Validation

materialId == null

├─ YES
│  └─ throw IllegalArgumentException
│
└─ NO
   ↓

validateMaterial()

↓

materialMapper.selectMaterial()

↓

조건

existingMaterial != null

├─ YES
│  └─ update 처리
│
└─ NO
   └─ insert 처리

↓

return result

실제 Source에 없는 흐름은 만들지 않는다.

---

## 31. 최종 출력

파일을 생성하지 않는다.

다음 형식으로 출력한다.

=== BE DEEP TEST v0.1 ===

SERVICE IMPL FILE:
{SERVICE_IMPL_FILE}

ENTRY METHOD:
{SERVICE_METHOD}

ENTRY EVIDENCE:
{PATH:LINES}

LOCAL METHODS

1.

Method:
{METHOD}

Called From:
{CALLER}

Evidence:
{PATH:LINES}

BUSINESS SERVICE CALLS

1.

Called From:
{METHOD}

Call:
{SERVICE_VARIABLE}.{SERVICE_METHOD}

MAPPER CALLS

1.

Called From:
{METHOD}

Call:
{MAPPER_VARIABLE}.{MAPPER_METHOD}

EXTERNAL CALLS

1.

Called From:
{METHOD}

Call:
{CALL}

VALIDATION

1.

Condition:
{CONDITION}

Success:
{FLOW}

Failure:
{FLOW}

Evidence:
{PATH:LINES}

BRANCHES

1.

Condition:
{CONDITION}

TRUE:
{FLOW}

FALSE:
{FLOW}

Evidence:
{PATH:LINES}

RETURN

{RETURN}

EXCEPTION

{EXCEPTION|NONE}

EXECUTION FLOW

{ENTRY_METHOD}

→ ...

→ ...

→ Return

STATUS:
COMPLETED

---

## 32. Evidence

Evidence는 Method를 읽으면서
이미 확인한 위치를 사용한다.

Evidence 때문에
추가 검색하지 않는다.

Project Root 기준 상대경로를 사용한다.

절대경로는 출력하지 않는다.

---

## 33. 성능 핵심

이 Skill에서는
다음 검색이 발생하면 안 된다.

Backend URL 검색

Controller 검색

Service Interface 검색

ServiceImpl 후보 검색

Mapper XML 검색

SQL 검색

Entry Source는 이미 입력으로 받는다.

Deep 분석은:

SERVICE_IMPL_FILE

에서 바로 시작한다.

---

## 34. 중요한 STOP 규칙

다른 Business Service 호출을 발견해도
이번 버전에서는 내부로 들어가지 않는다.

Mapper 호출을 발견해도
XML/SQL로 들어가지 않는다.

External/SAP/RFC 호출을 발견해도
내부로 들어가지 않는다.

이번 테스트는:

Entry ServiceImpl
+
Local Method
+
실행 Branch

까지만 측정한다.

---

## 35. 완료

최종 결과 출력 후 즉시 종료한다.

추가 Source 탐색을 하지 않는다.

다음 분석을 자동 실행하지 않는다.