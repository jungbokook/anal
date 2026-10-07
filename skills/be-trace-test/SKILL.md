---
name: be-trace-test
description: Backend URL 하나를 대상으로 Controller부터 ServiceImpl, 직접 Mapper 호출, MyBatis XML, SQL까지 최소 탐색하고 단계별 진행 체크포인트를 출력하여 병목 구간을 확인한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Trace Test v0.2 - Checkpoint

## 1. 목적

Backend 분석 최소 경로의 병목 구간을 확인한다.

분석 범위:

Backend URL
→ Controller
→ Service
→ ServiceImpl
→ 직접 Mapper 호출
→ MyBatis XML
→ SQL

새로운 분석 기능은 추가하지 않는다.

v0.1과 동일한 분석 범위를 유지한다.

---

## 2. 입력

HTTP_METHOD

BACKEND_URL

예:

POST /material/create

또는:

UNKNOWN /material/create

---

## 3. 실행 시작

분석 시작 즉시 다음을 출력한다.

[BE-TRACE] START

Method: {HTTP_METHOD}
URL: {BACKEND_URL}

각 단계가 완료되는 즉시
다음 단계로 넘어가기 전에 Checkpoint를 출력한다.

Checkpoint를 마지막에 몰아서 출력하지 않는다.

---

## 4. 이번 버전에서 하지 않는 것

다음은 수행하지 않는다.

- BE-REFERENCE
- Markdown 문서 생성
- Write
- Agent
- 병렬 처리
- Code Index
- Oracle MCP
- Database Metadata
- Local/private Method 내부 추적
- 다른 Service 내부 추적
- SAP
- RFC
- 외부 API 내부 추적
- Exception 상세 분석
- Response 상세 분석
- Validation 상세 분석
- Branch 상세 분석
- resultMap 상세 분석
- include 내부 추적
- Mapper.java 기본 탐색

이번 테스트는 새로운 기능을 추가하는 테스트가 아니다.

---

## 5. Source Boundary

Java:

gipms-api-*/src/main/java/**

Resources:

gipms-api-*/src/main/resources/**

이 범위만 사용한다.

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

JAR / Dependency fallback은 금지한다.

---

## 7. 공통 FAST 원칙

항상 다음 순서로 진행한다.

정확한 문자열
→ Grep
→ 위치 확보
→ 필요한 범위만 Read
→ 다음 Symbol 확보
→ 다음 단계

금지:

- 전체 Repository 구조 파악
- 전체 Java 파일 Read
- 전체 XML Read
- 관련 파일 사전 수집
- 관련성이 확인되지 않은 Source 탐색
- 이미 찾은 Symbol 재검색

---

## 8. Cache

한 번 확보한 정보는 다시 찾지 않는다.

유지:

CURRENT_PROJECT

CONTROLLER_FILE
CONTROLLER_CLASS
CONTROLLER_METHOD

SERVICE_TYPE
SERVICE_METHOD

SERVICE_IMPL_FILE
SERVICE_IMPL_CLASS

MAPPER_TYPES
MAPPER_METHODS
MAPPER_XML

같은 Symbol을 다시 Grep하지 않는다.

---

# PHASE 1. Controller

## 9. Controller 탐색

BACKEND_URL에서
식별력이 높은 Mapping 문자열을 사용한다.

Java Source에서 Controller 후보를 찾는다.

후보에서:

class-level mapping

+

method-level mapping

을 결합하여
BACKEND_URL과 일치하는지 확인한다.

HTTP_METHOD가 지정되어 있으면 함께 확인한다.

HTTP_METHOD가 UNKNOWN이면
Mapping Annotation에서 Method를 확정한다.

---

## 10. Controller Read

Controller Method 위치에서
약 60줄만 먼저 Read한다.

Method 종료가 보이지 않을 때만 확장한다.

전체 Controller 파일은 읽지 않는다.

---

## 11. Controller에서 확보

다음 정보를 확보한다.

- Controller Class
- Controller Method
- Service Variable
- Service Type
- Service Method
- CURRENT_PROJECT

Controller가 위치한 gipms-api-* 프로젝트를
CURRENT_PROJECT로 지정한다.

---

## 12. CHECKPOINT 1

Controller가 확정되는 즉시 출력한다.

[BE-TRACE] CHECKPOINT 1/5 - CONTROLLER FOUND

Project:
{CURRENT_PROJECT}

Controller:
{CONTROLLER_CLASS}#{CONTROLLER_METHOD}

Service Call:
{SERVICE_TYPE}#{SERVICE_METHOD}

Checkpoint 출력 후 즉시 Service 탐색으로 이동한다.

---

# PHASE 2. Service / ServiceImpl

## 13. Service 탐색

Controller에서 확보한
정확한 Service Type만 사용한다.

검색 범위:

CURRENT_PROJECT/src/main/java/**

Service 이름을 추측하지 않는다.

전체 Service 목록을 탐색하지 않는다.

---

## 14. Service Method

Controller에서 실제 호출한
Service Method 선언만 확인한다.

Service Interface 전체를 분석하지 않는다.

필요한 정보가 확보되면
즉시 ServiceImpl 탐색으로 이동한다.

---

## 15. ServiceImpl 탐색

정확한:

implements ServiceType

또는 이미 Source에서 확인한
구현 Class 이름을 사용한다.

검색 범위:

CURRENT_PROJECT/src/main/java/**

구현체가 확정되면
다른 후보 탐색을 중단한다.

---

## 16. ServiceImpl Method

Controller에서 호출한
정확한 Service Method만 찾는다.

Method 시작 위치에서
약 80줄을 먼저 Read한다.

Method 종료가 보이지 않을 때만
추가 범위를 Read한다.

전체 ServiceImpl 파일을 읽지 않는다.

---

## 17. CHECKPOINT 2

ServiceImpl Method가 확보되는 즉시 출력한다.

[BE-TRACE] CHECKPOINT 2/5 - SERVICE IMPL FOUND

Service:
{SERVICE_TYPE}#{SERVICE_METHOD}

ServiceImpl:
{SERVICE_IMPL_CLASS}#{SERVICE_METHOD}

Checkpoint 출력 후
즉시 직접 Mapper 호출 확인으로 이동한다.

---

# PHASE 3. Direct Mapper Calls

## 18. Mapper 분석 범위

ServiceImpl의 현재 Method Body에서
직접 호출되는 Mapper만 찾는다.

예:

materialMapper.selectMaterial(...)

materialMapper.insertMaterial(...)

historyMapper.insertHistory(...)

---

## 19. 이번 단계에서 추적하지 않는 호출

다음 호출은 내부로 들어가지 않는다.

Local/private Method:

validateMaterial(...)

calculateValue(...)

다른 Service:

otherService.check(...)

외부 연동:

sapService.send(...)

externalClient.call(...)

이러한 호출은 존재 여부와 이름만 기록할 수 있다.

내부 Source는 탐색하지 않는다.

---

## 20. Mapper 호출에서 확보

각 Mapper 호출에서 다음을 확보한다.

- Mapper Variable
- Mapper Type
- Mapper Method

예:

Variable:
materialMapper

Type:
MaterialMapper

Method:
selectMaterial

---

## 21. Mapper Type 확인

현재 읽은 ServiceImpl Method 범위에서
Mapper Type이 확인되면 그대로 사용한다.

확인되지 않을 때만
현재 ServiceImpl 파일에서
Mapper Variable 이름을 정확히 Grep한다.

예:

materialMapper

Mapper Type을 확보하면
즉시 Grep을 종료한다.

---

## 22. Mapper.java

Mapper Java Interface는
이번 테스트에서 기본적으로 읽지 않는다.

기본 흐름:

ServiceImpl
→ Mapper Type
→ Mapper Method
→ MyBatis XML

Mapper.java를 통한 중간 검증을 하지 않는다.

---

## 23. CHECKPOINT 3

직접 Mapper 호출 목록이 확보되는 즉시 출력한다.

[BE-TRACE] CHECKPOINT 3/5 - DIRECT MAPPER CALLS FOUND

Mapper Call Count:
{COUNT}

Mapper Calls:

1. {MAPPER_TYPE}#{MAPPER_METHOD}
2. {MAPPER_TYPE}#{MAPPER_METHOD}
3. ...

Mapper 호출이 없으면:

Mapper Call Count:
0

으로 출력한다.

Checkpoint 출력 후
즉시 XML 탐색으로 이동한다.

---

# PHASE 4. MyBatis XML

## 24. XML 탐색

각 Mapper Type에 대해
CURRENT_PROJECT의:

src/main/resources/**

에서 namespace를 찾는다.

예:

<mapper namespace="...MaterialMapper">

정확한 XML이 발견되면
해당 Mapper Type에 대한
다른 XML 탐색을 중단한다.

---

## 25. XML Fallback

Mapper Type namespace로 찾지 못한 경우에만
정확한 Mapper Method ID를 사용할 수 있다.

예:

id="selectMaterial"

검색 범위는 여전히:

CURRENT_PROJECT/src/main/resources/**

로 제한한다.

다른 Backend 프로젝트까지 확장하지 않는다.

---

## 26. Statement 탐색

확정된 Mapper XML 안에서
실제 Mapper Method와 연결되는 Statement만 찾는다.

대상:

<select>

<insert>

<update>

<delete>

예:

<select id="selectMaterial">

---

## 27. XML 부분 Read

Statement 시작 위치에서
약 40줄만 먼저 Read한다.

다음 종료 Tag가 확인되면
즉시 해당 Statement Read를 종료한다.

</select>

</insert>

</update>

</delete>

40줄 안에 종료되지 않을 때만
추가 범위를 읽는다.

전체 XML 파일은 읽지 않는다.

---

## 28. CHECKPOINT 4

필요한 XML Statement 위치가 모두 확보되면
즉시 출력한다.

[BE-TRACE] CHECKPOINT 4/5 - MYBATIS XML FOUND

XML Count:
{COUNT}

Statements:

1.
Mapper:
{MAPPER_TYPE}#{MAPPER_METHOD}

XML:
{XML_PATH}

Statement:
{STATEMENT_ID}

2.
...

Checkpoint 출력 후
즉시 SQL 확인으로 이동한다.

---

# PHASE 5. SQL

## 29. SQL 분석 범위

각 Statement에서 다음만 확인한다.

- SQL Type
- Main Table
- JOIN Table
- WHERE
- Parameter
- Dynamic SQL 존재 여부
- include 존재 여부

SQL을 장문으로 설명하지 않는다.

---

## 30. SQL Type

다음 중 하나로 기록한다.

SELECT

INSERT

UPDATE

DELETE

---

## 31. Table

SQL Source에서 직접 확인되는 Table만 기록한다.

추측하지 않는다.

확인:

- Main Table
- JOIN Table

---

## 32. Parameter

SQL에서 직접 사용되는
MyBatis Parameter를 확인한다.

예:

#{materialId}

#{plantCode}

${value}

상세 Request → SQL Mapping은
이번 버전에서 분석하지 않는다.

---

## 33. Dynamic SQL

다음 Tag가 존재하면 기록한다.

<if>

<choose>

<when>

<otherwise>

<foreach>

이번 버전에서는:

Dynamic SQL:
YES

또는:

Dynamic SQL:
NO

정도로만 기록한다.

상세 Branch 분석은 하지 않는다.

---

## 34. include

Statement에:

<include refid="..."/>

가 있으면:

Include:
{refid}

만 기록한다.

include 내부 Source는 추적하지 않는다.

---

## 35. CHECKPOINT 5

SQL 기본 정보가 확보되는 즉시 출력한다.

[BE-TRACE] CHECKPOINT 5/5 - SQL FOUND

SQL Statements:

1.
Mapper:
{MAPPER_TYPE}#{MAPPER_METHOD}

SQL Type:
{SQL_TYPE}

Main Table:
{MAIN_TABLE}

Dynamic SQL:
{YES|NO}

Include:
{REFID|NONE}

2.
...

---

# PHASE 6. Final Result

## 36. 종료 조건

CHECKPOINT 5까지 완료하면
추가 Source 탐색을 하지 않는다.

다음 기능을 분석하지 않는다.

- Local Method 내부
- 다른 Service 내부
- SAP/RFC
- External API
- Response 상세
- Exception 상세
- Validation 상세
- Branch 상세
- resultMap
- include 내부

---

## 37. 최종 결과

파일을 생성하지 않는다.

다음 형식으로 화면에 출력한다.

=== BE TRACE TEST v0.2 ===

HTTP METHOD:
{HTTP_METHOD}

BACKEND URL:
{BACKEND_URL}

CURRENT PROJECT:
{CURRENT_PROJECT}

CONTROLLER

Class:
{CONTROLLER_CLASS}

Method:
{CONTROLLER_METHOD}

Evidence:
{PATH:LINES}

SERVICE

Type:
{SERVICE_TYPE}

Method:
{SERVICE_METHOD}

SERVICE IMPL

Class:
{SERVICE_IMPL_CLASS}

Method:
{SERVICE_METHOD}

Evidence:
{PATH:LINES}

DIRECT MAPPER CALLS

1.

Mapper Type:
{MAPPER_TYPE}

Mapper Method:
{MAPPER_METHOD}

XML:
{XML_PATH}

Statement:
{STATEMENT_ID}

SQL Type:
{SQL_TYPE}

Main Table:
{MAIN_TABLE}

Dynamic SQL:
{YES|NO}

Include:
{REFID|NONE}

Evidence:
{PATH:LINES}

NON-TRACED CALLS

Local Methods:
{NAMES|NONE}

Other Services:
{NAMES|NONE}

External:
{NAMES|NONE}

TRACE

{CONTROLLER_CLASS}#{CONTROLLER_METHOD}
→ {SERVICE_TYPE}#{SERVICE_METHOD}
→ {SERVICE_IMPL_CLASS}#{SERVICE_METHOD}
→ {MAPPER_TYPE}#{MAPPER_METHOD}
→ {STATEMENT_ID}
→ {SQL_TYPE} {MAIN_TABLE}

STATUS:
COMPLETED

---

## 38. Evidence

Evidence는 Source를 탐색하면서
이미 확인한 위치를 사용한다.

Evidence 때문에
추가 Grep이나 Read를 하지 않는다.

형식:

Project Root 기준 상대경로:라인범위

절대경로는 출력하지 않는다.

---

## 39. 탐색 실패

특정 단계에서 Source를 찾지 못하면
검색 범위를 무작정 확장하지 않는다.

다음 중 하나를 기록한다.

CONTROLLER_NOT_FOUND

SERVICE_NOT_FOUND

SERVICE_IMPL_NOT_FOUND

MAPPER_TYPE_NOT_FOUND

MAPPER_XML_NOT_FOUND

STATEMENT_NOT_FOUND

SQL_NOT_FOUND

JAR이나 Dependency로 이동하지 않는다.

---

## 40. Checkpoint 원칙

Checkpoint의 목적은
어느 구간에서 실행이 오래 걸리는지
사용자가 관찰할 수 있게 하는 것이다.

따라서 각 Checkpoint는
해당 단계가 완료되는 즉시 출력한다.

다음 단계까지 기다렸다가
이전 Checkpoint를 함께 출력하지 않는다.

순서:

[BE-TRACE] START

↓

CHECKPOINT 1/5
CONTROLLER FOUND

↓

CHECKPOINT 2/5
SERVICE IMPL FOUND

↓

CHECKPOINT 3/5
DIRECT MAPPER CALLS FOUND

↓

CHECKPOINT 4/5
MYBATIS XML FOUND

↓

CHECKPOINT 5/5
SQL FOUND

↓

FINAL RESULT

↓

종료

---

## 41. 완료

최종 결과 출력 후 즉시 종료한다.

추가 Source 탐색을 하지 않는다.

다음 분석 단계를 자동 실행하지 않는다.