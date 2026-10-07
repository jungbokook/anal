---
name: be-analysis
description: 화면명, FE Base Name, HTTP Method와 Backend URL을 기준으로 실제 Backend Call Path를 끝까지 추적하고 BE-REFERENCE 형식의 Backend 분석 문서를 생성한다. MyBatis는 XML 위치와 Statement ID까지만 확인하며 SQL은 분석하지 않는다.
argument-hint: "<화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep, Glob, Write
---

# Backend Analysis

## 1. 목적

선택한 Backend URL 하나의 실제 실행 흐름을 분석하고
개발 참조용 Backend 분석 문서를 생성한다.

분석은 Layer 목록을 만드는 것이 아니라
실제 Source 실행 순서를 보존하는 것을 목적으로 한다.

기본 흐름:

```text
Backend URL
↓
Controller
↓
Service / ServiceImpl
↓
Validation / 조건 / 분기
↓
Local / private Method
↓
다른 Business Service / Common Service
↓
Mapper
↓
MyBatis XML 위치 + Statement ID
↓
Caller 복귀
↓
SAP / RFC / REST / 기타 외부 연동
↓
Caller 복귀
↓
후속 Business Logic
↓
Response
```

실제 Source에 존재하지 않는 단계는 생성하지 않는다.

---

# 2. 입력

입력 형식:

```text
/be-analysis <화면명> <FE Base Name> <HTTP Method|UNKNOWN> <Backend URL>
```

입력 항목:

```text
화면명
FE Base Name
HTTP Method
Backend URL
```

예:

```text
/be-analysis generator-add FE-ACT-010-create POST /generator/create
```

HTTP Method를 모르면:

```text
/be-analysis generator-add FE-ACT-010-create UNKNOWN /generator/create
```

을 허용한다.

각 입력값의 역할:

```text
화면명
  ↓
Backend 문서 저장 화면 결정

FE Base Name
  ↓
현재 Backend가 속한 FE Action 결정

HTTP Method
  ↓
Controller Mapping 확인

Backend URL
  ↓
분석할 Backend Endpoint 결정
```

FE Base Name 예:

```text
FE-ACT-001-search
FE-ACT-002-save
FE-ACT-010-create
```

FE Base Name은 사용자가 입력한 값을 사용한다.

임의로 변경하거나 새 Action ID를 생성하지 않는다.

UNKNOWN인 경우 Controller Mapping Source에서
실제 HTTP Method를 확인한다.

동일 URL에 여러 HTTP Method가 존재하고
하나로 확정할 수 없으면 임의 선택하지 않는다.

확인되지 않음으로 기록하고 STOP 한다.

---

# 3. FE Base Name

FE Base Name은 현재 Backend 분석의 Parent Key다.

예:

```text
FE-ACT-010-create
```

하나의 FE Action에서 Backend URL이 여러 개 호출될 수 있다.

예:

```text
FE-ACT-010-create
│
├─ Backend URL #1
├─ Backend URL #2
└─ Backend URL #3
```

각 Backend 분석 문서는 동일 FE Base Name 아래에서
독립적인 BE ID를 가진다.

예:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
FE-ACT-010-create-BE-003.md
```

API ID는 사용하지 않는다.

현재 구조는:

```text
SCREEN
↓
FE
↓
BE
```

이다.

---

# 4. BE ID 및 OUTPUT_PATH 선확정

Backend Source 분석을 시작하기 전에
반드시 다음 순서로 문서 식별자를 먼저 확정한다.

```text
화면명
↓
FE Base Name
↓
기존 Backend 문서 확인
↓
BE ID 확정
↓
OUTPUT_PATH 확정
↓
Backend Source 분석 시작
```

Worker 또는 분석 중간 단계가
파일명을 임의로 결정하면 안 된다.

---

# 5. BE ID 결정

Backend 문서 저장 위치:

```text
docs/analysis/{화면명}/backend/
```

현재 FE Base Name과 동일한 기존 Backend 문서를 확인한다.

예:

```text
FE Base Name
  FE-ACT-010-create
```

기존 파일:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
```

이면 새 BE ID:

```text
BE-003
```

으로 결정한다.

최종 파일명:

```text
FE-ACT-010-create-BE-003.md
```

기존 파일이 하나도 없으면:

```text
BE-001
```

부터 시작한다.

BE 번호는 3자리 형식을 사용한다.

```text
BE-001
BE-002
BE-003
...
```

다른 FE Base Name의 BE 번호는
현재 FE Base Name의 번호 결정에 영향을 주지 않는다.

예:

```text
FE-ACT-001-search-BE-005.md
```

가 존재하더라도 현재 입력이:

```text
FE-ACT-010-create
```

이고 해당 FE Base Name의 Backend 문서가 없다면:

```text
FE-ACT-010-create-BE-001.md
```

로 시작한다.

기존 파일을 덮어쓰지 않는다.

번호가 중간에 비어 있어도
기존 최대 번호 이후 번호를 사용한다.

예:

```text
BE-001
BE-003
```

이 존재하면:

```text
BE-004
```

를 사용한다.

---

# 6. OUTPUT_PATH

BE ID를 확정한 직후 OUTPUT_PATH를 확정한다.

형식:

```text
docs/analysis/{화면명}/backend/{FE Base Name}-{BE ID}.md
```

예:

```text
화면명
  generator-add

FE Base Name
  FE-ACT-010-create

BE ID
  BE-003
```

이면:

```text
OUTPUT_PATH
  docs/analysis/generator-add/backend/FE-ACT-010-create-BE-003.md
```

OUTPUT_PATH를 확정한 후에는
분석 도중 파일명을 변경하지 않는다.

---

# 7. Reference

문서 작성 형식은 다음 Reference를 사용한다.

```text
.claude/references/BE-REFERENCE.md
```

Reference는 다음을 결정한다.

```text
문서 구조
표현 형식
ASCII Tree
Block 형식
상세 수준
Excel 단일 셀 복사 형식
호출 복귀 표현
Evidence 표현
```

Reference에 포함된 Sample:

```text
Class
Method
Path
URL
Parameter
Response
RFC
Mapper
XML
Statement ID
```

등은 Evidence가 아니다.

실제 분석 결과에 복사하지 않는다.

Reference는 분석 시작 시 한 번만 읽는다.

분석 중 다시 읽지 않는다.

Reference보다 실제 Source가 우선한다.

---

# 8. 핵심 분석 원칙

항상 다음 원칙을 우선한다.

```text
검색 범위는 좁게
호출 깊이는 끝까지
```

현재 실행 흐름에 실제 연결된 Source만 분석한다.

관련 있어 보인다는 이유만으로
다른 Source를 탐색하지 않는다.

---

# 9. Source 탐색 방식

기본 탐색:

```text
정확한 Symbol
↓
Grep
↓
위치 확인
↓
필요한 범위만 Read
↓
다음 실제 호출 Symbol
```

전체 파일 Read를 기본 방식으로 사용하지 않는다.

전체 Repository 구조를 먼저 조사하지 않는다.

전체 Service 목록을 만들지 않는다.

전체 Mapper 목록을 만들지 않는다.

전체 XML 목록을 만들지 않는다.

Local Method 후보를 미리 수집하지 않는다.

외부 연동 후보를 미리 수집하지 않는다.

---

# 10. Source 범위

Backend Java Source 기본 범위:

```text
gipms-api-*/src/main/java/**
```

Controller가 발견되면
해당 Backend 프로젝트를:

```text
CURRENT_PROJECT
```

로 사용한다.

현재 Call Path가 Project Root 아래의
다른 실제 Source Project로 연결되면
해당 Project Source까지 따라갈 수 있다.

예:

```text
gipms-api-common
gipms-api-interface
```

단순히 이름이 관련 있어 보인다는 이유로
다른 Project를 검색하지 않는다.

---

# 11. 제외 범위

다음은 탐색하지 않는다.

```text
docs/**
**/sample/**
**/samples/**
**/test/**
**/tests/**
**/target/**
**/build/**
**/generated/**
**/*.jar
**/*.class
**/node_modules/**
**/.git/**
**/.gradle/**
**/.m2/**
**/libs/**
**/lib/**
**/BOOT-INF/**
**/WEB-INF/lib/**
```

다음도 분석하지 않는다.

```text
JAR 내부
Decompiled Source
Dependency 내부
Project Root 외부 Source
```

---

# 12. Controller 탐색

Backend URL에서
식별력이 높은 Mapping 문자열을 사용하여
Controller를 찾는다.

확인:

```text
Class-level Mapping
+
Method-level Mapping
+
HTTP Method
```

입력이 UNKNOWN이면
Mapping Annotation에서 Method를 확인한다.

실제 요청을 처리하는 Controller Method가 확정되면
관련 없는 Controller 탐색을 중단한다.

---

# 13. Controller 분석

Controller에서는 실제 요청 Method를 기준으로 확인한다.

확인 대상:

```text
Controller Class
Controller Method
HTTP Method
Backend URL
Request 전달
실제 Service 호출
Controller Return
```

Controller에서 호출되는 실제 Service Method를 확보한 후
다음 단계로 이동한다.

---

# 14. Service / ServiceImpl

Controller에서 실제 호출되는 Service만 따라간다.

Service Interface가 존재하면
실제 구현체를 확인한다.

실제 호출된 Method만 분석한다.

전체 ServiceImpl을 분석하지 않는다.

---

# 15. 실제 실행 순서

ServiceImpl에서는
Source에 작성된 실제 실행 순서를 보존한다.

예:

```text
Validation
↓
DB #1
↓
조건
├─ YES → 다른 Service
└─ NO  → Local 처리
↓
RFC #1
↓
RFC 결과 처리
↓
DB #2
↓
Response 생성
```

Layer별로 다시 정렬하지 않는다.

---

# 16. Local / private Method

현재 실행 경로에서 실제 호출되는
Local / private Method는 계속 따라간다.

예:

```text
search()
↓
validate()
↓
createRequest()
↓
processResult()
```

Local Method 목록을 미리 만들지 않는다.

현재 Method에서 실제 호출된 Method만 확인한다.

---

# 17. 다른 Business Service

현재 실행 경로에서 실제 호출되는
다른 Business Service / Common Service가 있으면 따라간다.

예:

```text
Service A
↓
Service B
↓
Common Service
```

호출 깊이를 임의로 1단계에서 중단하지 않는다.

Project Root 안의 실제 Source라면
Call Path가 이어지는 동안 계속 분석한다.

---

# 18. 호출 복귀

하위 Method 또는 다른 Service 분석 후
반드시 Caller의 다음 실행 위치로 복귀한다.

예:

```text
Service A
│
├─ Service B
│    │
│    ├─ Mapper
│    └─ return
│
├─ Service B 결과 처리
│
├─ Local Method
│
├─ RFC
│
└─ Response
```

하위 호출을 분석한 뒤
Caller의 후속 Business Logic을 빠뜨리지 않는다.

---

# 19. 중복 / 순환 방지

이미 분석한:

```text
Class + Method
```

는 다시 전체 분석하지 않는다.

순환 호출이면
호출 관계만 기록하고
무한 추적하지 않는다.

이미 확보한 Source 위치와 Evidence는 재사용한다.

---

# 20. Validation / 조건 / 분기

실제 실행 Method를 읽는 과정에서 확인되는:

```text
if
else
switch
Validation
null check
상태 비교
값 비교
예외 조건
```

을 실행 흐름에 포함한다.

조건을 찾기 위해
별도의 Repository 전체 검색을 수행하지 않는다.

Source에 없는 YES / NO 결과를 생성하지 않는다.

---

# 21. 반복 처리

실제 Call Path에 반복문이 존재하면 기록한다.

예:

```text
equipmentList 반복
│
├─ Mapper 호출
├─ 조건
└─ External 호출
```

Runtime 반복 횟수를 Source에서 알 수 없다면
숫자를 추측하지 않는다.

---

# 22. Mapper

실제 실행 경로에서 호출되는 Mapper만 분석한다.

확인:

```text
Mapper Type
Mapper Method
호출 위치
```

Mapper Variable의 Type이
현재 읽은 Java Source에서 확인되면 그대로 사용한다.

필요한 경우에만 현재 Java Source에서
Mapper 선언 위치를 확인한다.

Repository 전체 Mapper 검색을 하지 않는다.

---

# 23. Mapper.java 정책

Mapper Java Interface는
상세 분석 대상이 아니다.

Mapper Type과 Mapper Method가
Service Source에서 명확하면
Mapper.java를 읽지 않고 MyBatis XML로 이동한다.

Mapper Type 또는 Method 연결이 불명확한 경우에만
필요한 최소 범위를 확인한다.

---

# 24. MyBatis 분석 범위

MyBatis는 다음까지만 분석한다.

```text
Mapper Type
↓
Mapper Method
↓
MyBatis XML 파일 위치
↓
Statement ID
↓
STOP
```

SQL 내용은 분석하지 않는다.

---

# 25. MyBatis XML 탐색

Mapper Type을 기준으로
연결되는 MyBatis XML을 찾는다.

우선 검색 범위:

```text
CURRENT_PROJECT/src/main/resources/**
```

가능하면 정확한 namespace를 사용한다.

정확한 XML 파일이 확인되면
다른 XML 후보 탐색을 중단한다.

---

# 26. Statement ID

확정된 XML 안에서
실제 Mapper Method와 연결되는
Statement ID만 확인한다.

예:

```text
Mapper

EquipmentMapper.selectEquipment()

↓

MyBatis XML

EquipmentMapper.xml

↓

Statement ID

selectEquipment
```

Statement ID가 확인되면
해당 Mapper의 MyBatis 분석을 종료한다.

---

# 27. XML / SQL 금지

MyBatis XML은:

```text
XML 위치
Statement ID
```

확인을 위한 범위에서만 검색한다.

하지 않는다.

```text
XML 전체 Read
Statement Body Read
SQL 내용 분석
SELECT 내용 분석
INSERT 내용 분석
UPDATE 내용 분석
DELETE 내용 분석
MERGE 내용 분석
Table 분석
Column 분석
JOIN 분석
WHERE 분석
Parameter 분석
Dynamic SQL 분석
include 내부 추적
resultMap 분석
SQL Result Mapping
Oracle Metadata 조회
```

Statement ID 확인 이후
SQL을 보기 위한 추가 Read를 실행하지 않는다.

---

# 28. DB 호출 표현

실제 Mapper 호출은 문서에서:

```text
DB #1
DB #2
DB #3
```

처럼 실제 실행 위치 기준으로 번호를 부여한다.

같은 Mapper Method가 반복 호출되어도
실행 위치가 다르면 별도 DB 호출로 표현한다.

DB Block에는 다음만 기록한다.

```text
호출 위치
호출 전 조건
Mapper
Mapper Method
MyBatis XML
Statement ID
호출 후 처리
Evidence
```

SQL을 분석하지 않는다.

---

# 29. DB 상세 Block

형식:

```text
┌─ DB #N : {확인 가능한 처리 설명} ───────────
│ 호출 위치
│   {Class.Method}
│
│ 호출 조건
│   {실제 Source 조건}
│
│ Mapper
│   {MapperType.MapperMethod}
│
│ MyBatis XML
│   {Project Root 상대경로}
│
│ Statement ID
│   {Statement ID}
│
│ SQL 상세
│   분석 범위에서 제외
│
│ 호출 후 처리
│   {Caller에서 실제 수행되는 후속 처리}
│
│ Evidence
│   {Project Root 상대경로:Line Range}
└──────────────────────────────────────────────
```

SQL 내용을 추측해서
호출 목적을 작성하지 않는다.

---

# 30. SAP / RFC / 외부 연동

실제 Call Path에서 발견되는 외부 연동만 분석한다.

```text
SAP
RFC
REST / HTTP
SOAP
Message
File
기타 Client / Adapter
```

프로젝트 전체 외부 연동 목록을 만들지 않는다.

---

# 31. 외부 연동 추적

외부 연동이 실제 Project Source를 통해 이어지면
실제 호출 경로를 따라간다.

예:

```text
Service
↓
Common Service
↓
Adapter
↓
RFC Client
```

또는:

```text
Service
↓
Interface Service
↓
HTTP Client
```

Project Root 아래 실제 Source가 존재하는 범위까지 추적한다.

---

# 32. External Dependency Boundary

호출이 JAR / 외부 Dependency 내부로 넘어가면
더 이상 추적하지 않는다.

표현:

```text
════════ External Dependency Boundary ════════

{확인된 호출}

↓

외부 Library 내부 구현
분석하지 않음
```

---

# 33. 외부 연동 상세

외부 연동에서는 Source에서 실제 확인되는 범위까지 기록한다.

가능한 항목:

```text
호출 위치
호출 경로
호출 조건
Function / Endpoint
HTTP Method
URL
Request 생성
Parameter Mapping
Response
Response Mapping
Error 처리
호출 후 처리
Evidence
```

확인되지 않는 항목을 채우기 위해
관련 없는 Source까지 확장하지 않는다.

확인할 수 없으면:

```text
확인되지 않음
```

으로 기록한다.

---

# 34. 외부 연동 순서

외부 연동 번호는 실제 실행 순서 기준이다.

예:

```text
DB #1
↓
RFC #1
↓
DB #2
↓
REST #1
↓
RFC #2
↓
DB #3
```

종류별로 재정렬하지 않는다.

---

# 35. Exception

현재 실제 Call Path를 분석하면서 확인되는
Exception 흐름을 기록한다.

예:

```text
조건
↓
throw BusinessException
```

또는:

```text
try
↓
External Call
↓
catch
↓
Error Mapping
↓
throw
```

Exception을 찾기 위해
Project 전체를 검색하지 않는다.

Framework 일반 동작을 추측하지 않는다.

---

# 36. Transaction

현재 Call Path Source에서:

```text
@Transactional
```

또는 실제 Transaction 설정이 확인되면 기록한다.

Rollback 동작은
Source에서 확인 가능한 범위만 작성한다.

Transaction이 명시적으로 확인되지 않으면
추측하지 않는다.

---

# 37. Response

Backend 분석은 Mapper나 외부 연동에서 끝내지 않는다.

하위 호출이 끝나면 Caller로 복귀하여
최종 Response까지 계속 따라간다.

확인:

```text
하위 호출 결과
↓
후속 처리
↓
값 변환
↓
Response 객체 생성
↓
Service Return
↓
Controller Return
```

실제 Source에서 확인되는 Mapping만 기록한다.

---

# 38. Evidence

중요 판단은 가능한 경우
Source Evidence와 연결한다.

형식:

```text
Project Root 기준 상대경로
Class / Method
Line Range
```

절대경로를 출력하지 않는다.

Line Range를 확인할 수 없으면:

```text
Line Range
확인되지 않음
```

으로 기록한다.

추측하지 않는다.

Evidence를 만들기 위해
이미 분석한 Source를 불필요하게 다시 읽지 않는다.

---

# 39. 분석 Cache

현재 분석 과정에서 확보한 정보는 재사용한다.

재사용 대상:

```text
Class + Method
Source Path
Line Range
Mapper Type + Method
Mapper Type + XML Path
XML + Statement ID
외부 연동 Source
```

같은 정보를 반복 Grep / Read하지 않는다.

---

# 40. 출력 경로

최종 문서는:

```text
docs/analysis/{화면명}/backend/
```

아래에 생성한다.

화면명은 입력받은 값을 그대로 사용한다.

---

# 41. 최종 파일명

최종 파일명은 반드시:

```text
{FE Base Name}-{BE ID}.md
```

형식을 사용한다.

예:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
FE-ACT-010-create-BE-003.md
```

최종 OUTPUT_PATH:

```text
docs/analysis/{화면명}/backend/{FE Base Name}-{BE ID}.md
```

FE Base Name을 다시 생성하지 않는다.

FE Base Name을 분석 결과에 따라 변경하지 않는다.

BE ID는 분석 시작 전에 확정한다.

OUTPUT_PATH도 분석 시작 전에 확정한다.

---

# 42. 최종 문서 구조

다음 구조를 사용한다.

```text
1. 기능 정보
2. 기능 요약
3. 한눈에 보는 실행 흐름
4. 핵심 정보
5. 전체 실행 Tree
6. Business Logic 상세
7. DB 상세
8. 외부 연동 상세
9. Exception / Transaction
10. Response 생성
11. Source Evidence
12. 미확인 항목
13. 분석 경계
```

실제 기능에 존재하지 않는 Block은
억지로 생성하지 않는다.

---

# 43. 한눈에 보는 실행 흐름

단순 Layer 목록을 만들지 않는다.

실제 주요 Business Logic 순서를 표현한다.

예:

```text
Controller
↓
Service
↓
Validation
↓
DB #1
↓
조건
├─ YES → RFC #1
└─ NO → Local 처리
↓
DB #2
↓
Response
```

---

# 44. 전체 실행 Tree

전체 실행 Tree는
실제 실행 순서와 호출 깊이,
Caller 복귀를 표현한다.

예:

```text
Controller
│
└─ Service A
    │
    ├─ [1] Validation
    │
    ├─ [2] Local Method
    │
    ├─ [3] Service B
    │      │
    │      ├─ DB #1
    │      │    ├─ Mapper
    │      │    ├─ XML
    │      │    └─ Statement ID
    │      │
    │      └─ return
    │
    ├─ [4] Service B 결과 처리
    │
    ├─ [5] RFC #1
    │      │
    │      └─ return
    │
    ├─ [6] RFC 결과 처리
    │
    ├─ [7] DB #2
    │
    ├─ [8] Response 생성
    │
    └─ return
```

SQL은 Tree에 넣지 않는다.

---

# 45. 미확인 항목

실제 Source에서 확인하지 못한 값은
추측하지 않는다.

예:

```text
External Base URL
  확인되지 않음

사유
  현재 Project Source에서 실제 Configuration 값 확인 불가
```

SQL을 분석하지 않은 것은
미확인 항목이 아니다.

정책적으로 분석 범위에서 제외한 것이다.

---

# 46. 분석 경계

포함:

```text
Controller
Service / ServiceImpl
Local / private Method
Other Service / Common Service
Validation
조건 / 분기
반복
Mapper
MyBatis XML 위치
Statement ID
SAP / RFC
REST / HTTP
기타 실제 외부 연동
Exception
Transaction
Response
Source Evidence
```

제외:

```text
SQL Body
SQL 상세
Table / Column 분석
Dynamic SQL
SQL Parameter Mapping
SQL Result Mapping
Oracle Metadata
Schema Metadata
관련 없는 Service
관련 없는 Mapper
관련 없는 외부 연동
JAR 내부
Decompiled Source
Project Root 외부 Source
```

---

# 47. Excel 복사 형식

`.claude/references/BE-REFERENCE.md`의
Text Block + ASCII Tree 형식을 유지한다.

Markdown Table을 사용하지 않는다.

각 Block은 가능하면
Excel 한 셀에 독립적으로 복사해도
내용을 이해할 수 있게 작성한다.

---

# 48. Read-Only

Backend 분석은 Read-Only다.

```text
업무 API 실행 금지
실제 RFC 호출 금지
상태 변경 REST 요청 금지
Message Publish 금지
File 외부 전송 금지
Database 변경 작업 금지
```

Source를 분석해서 문서만 생성한다.

---

# 49. 성능 보호 규칙

다음은 금지한다.

```text
전체 Repository 선행 탐색
전체 Java 목록 생성
전체 Service 목록 생성
전체 Mapper 목록 생성
전체 XML 목록 생성
Local Method 사전 목록 생성
외부 연동 사전 목록 생성
Mapper.java 기본 Read
MyBatis XML 전체 Read
Statement Body Read
SQL 분석
Oracle Metadata 조회
Code Index 기본 사용
같은 Symbol 반복 검색
같은 Source 반복 Read
Reference 반복 Read
```

실제 Call Path를 따라가면서
필요한 Source만 확인한다.

---

# 50. 최종 검증

문서 생성 전에 확인한다.

```text
입력 화면명 확인
↓
FE Base Name 확인
↓
BE ID 확인
↓
OUTPUT_PATH 확인
↓
Reference 1회 로드 확인
↓
Controller 확인
↓
실제 Service / 구현체 확인
↓
실제 실행 순서 확인
↓
Local/private Method 확인
↓
다른 Service / Common Service 확인
↓
하위 호출 후 Caller 복귀 확인
↓
실제 Mapper 호출 확인
↓
MyBatis XML 위치 확인
↓
Statement ID 확인
↓
SQL을 분석하지 않았는지 확인
↓
실제 외부 연동 확인
↓
외부 연동 후 Caller 복귀 확인
↓
조건 / 분기 확인
↓
반복 구조 확인
↓
Exception 확인
↓
Transaction 확인
↓
Response까지 연결 확인
↓
Evidence 확인
↓
BE-REFERENCE 형식 적용
↓
OUTPUT_PATH에 문서 생성
```

현재 API에 존재하지 않는 항목은
억지로 생성하지 않는다.

---

# 51. STOP

현재 Backend URL 하나의 분석 문서가 완성되면 STOP 한다.

자동으로 다음 Backend URL을 분석하지 않는다.

자동으로 다음 Backend URL을 추측하지 않는다.

자동으로 다른 화면을 분석하지 않는다.

현재 Call Path와 연결되지 않은:

```text
Service
Mapper
XML
External
Business Logic
```

으로 확장하지 않는다.

현재 Backend URL의 실제 Call Path가
Response까지 완료되고:

```text
docs/analysis/{화면명}/backend/{FE Base Name}-{BE ID}.md
```

파일이 생성되면 종료한다.