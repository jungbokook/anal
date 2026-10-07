---
name: be-trace-test
description: Backend URL 하나를 대상으로 Controller에서 ServiceImpl, 직접 Mapper 호출, MyBatis XML, SQL까지 최소 탐색으로 추적하고 단계별 체크포인트를 출력한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Trace Test v0.2 - FAST Baseline

## 1. 목적

Backend URL에서 시작하여
최소 Source 탐색으로 다음 경로만 확인한다.

Backend URL
→ Controller
→ Service
→ ServiceImpl
→ 직접 Mapper 호출
→ MyBatis XML
→ SQL

이 Skill은 FAST 탐색 기준 버전이다.

현재 성능 기준:

약 1분 45초

이 Skill에는
추가 Deep 분석 기능을 넣지 않는다.

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

분석 시작 즉시 출력한다.

[BE-TRACE] START

Method: {HTTP_METHOD}
URL: {BACKEND_URL}

각 단계가 완료되는 즉시
Checkpoint를 출력한다.

---

## 4. 수행하지 않는 것

다음은 수행하지 않는다.

- BE-REFERENCE
- Markdown 문서 생성
- Write
- Agent
- 병렬 처리
- Code Index
- Oracle MCP
- Database Metadata
- Local/private Method 탐색
- Local/private Method 내부 추적
- 다른 Service 내부 추적
- SAP 내부 추적
- RFC 내부 추적
- 외부 API 내부 추적
- Exception 상세 분석
- Response 상세 분석
- Validation 상세 분석
- Branch 상세 분석
- resultMap 상세 분석
- include 내부 추적
- Mapper.java 기본 탐색

---

## 5. Source Boundary

Java:

gipms-api-*/src/main/java/**

Resources:

gipms-api-*/src/main/resources/**

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

## 7. FAST 원칙

항상:

정확한 문자열
→ Grep
→ 위치 확인
→ 필요한 부분만 Read
→ 다음 Symbol 확보
→ 즉시 다음 단계

순서로 진행한다.

금지:

- 전체 Repository 구조 탐색
- 전체 Java 파일 Read
- 전체 XML Read
- 관련 파일 사전 수집
- 관련성이 확인되지 않은 Source 탐색
- 이미 찾은 Symbol 재검색

---

## 8. Cache

한 번 확보한 정보는 재사용한다.

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

UNKNOWN이면
Mapping Annotation에서 HTTP Method를 확정한다.

---

## 10. Controller Read

Controller Method 위치에서
약 60줄만 먼저 Read한다.

Method 종료가 보이지 않을 때만 확장한다.

전체 Controller 파일을 읽지 않는다.

---

## 11. Controller에서 확보

다음만 확보한다.

- Controller Class
- Controller Method
- Service Variable
- Service Type
- Service Method
- CURRENT_PROJECT

Controller가 위치한:

gipms-api-*

를 CURRENT_PROJECT로 지정한다.

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

---

# PHASE 2. Service / ServiceImpl

## 13. Service 탐색

Controller에서 확인한
정확한 Service Type을 사용한다.

검색 범위:

CURRENT_PROJECT/src/main/java/**

전체 Service 목록을 탐색하지 않는다.

---

## 14. Service Method

Controller에서 실제 호출한
Service Method 선언만 확인한다.

Service Interface 전체를 분석하지 않는다.

---

## 15. ServiceImpl 탐색

정확한:

implements ServiceType

또는 이미 확인된
구현 Class 이름을 사용한다.

CURRENT_PROJECT 안에서만 찾는다.

구현체가 확정되면
다른 후보 탐색을 중단한다.

---

## 16. ServiceImpl Method

정확한 Service Method만 찾는다.

Method 시작 위치에서:

약 80줄

만 먼저 Read한다.

Method 종료가 보이지 않을 때만
추가 Read한다.

전체 ServiceImpl 파일은 읽지 않는다.

---

## 17. CHECKPOINT 2

ServiceImpl Method가 확보되는 즉시 출력한다.

[BE-TRACE] CHECKPOINT 2/5 - SERVICE IMPL FOUND

Service:
{SERVICE_TYPE}#{SERVICE_METHOD}

ServiceImpl:
{SERVICE_IMPL_CLASS}#{SERVICE_METHOD}

ServiceImpl File:
{SERVICE_IMPL_FILE}

---

# PHASE 3. Direct Mapper

## 18. Mapper 분석 범위

ServiceImpl Method Body에서
직접 호출되는 Mapper만 찾는다.

예:

materialMapper.selectMaterial(...)

materialMapper.insertMaterial(...)

historyMapper.insertHistory(...)

---

## 19. 추적하지 않는 호출

다음은 내부 추적하지 않는다.

Local/private Method

다른 Business Service

SAP/RFC

External API

호출이 보여도
내부 Source로 이동하지 않는다.

---

## 20. Mapper 호출에서 확보

각 직접 Mapper 호출에서:

Mapper Variable

Mapper Type

Mapper Method

를 확보한다.

---

## 21. Mapper Type

현재 읽은 범위에서
Mapper Type이 보이면 그대로 사용한다.

보이지 않을 때만
현재 ServiceImpl 파일에서
Mapper Variable 이름을 정확히 Grep한다.

---

## 22. Mapper.java

Mapper Java Interface는
기본적으로 읽지 않는다.

흐름:

ServiceImpl
→ Mapper Type
→ Mapper Method
→ MyBatis XML

---

## 23. CHECKPOINT 3

Mapper 호출 목록이 확보되는 즉시 출력한다.

[BE-TRACE] CHECKPOINT 3/5 - DIRECT MAPPER CALLS FOUND

Mapper Call Count:
{COUNT}

Mapper Calls:

1. {MAPPER_TYPE}#{MAPPER_METHOD}
2. {MAPPER_TYPE}#{MAPPER_METHOD}

Mapper 호출이 없으면:

Mapper Call Count:
0

---

# PHASE 4. MyBatis XML

## 24. XML 탐색

CURRENT_PROJECT의:

src/main/resources/**

에서 Mapper Type과 연결되는
namespace를 찾는다.

예:

<mapper namespace="...MaterialMapper">

정확한 XML이 발견되면
다른 XML 탐색을 중단한다.

---

## 25. Mapper XML Cache

동일 Mapper Type의 XML을
이미 찾았다면 다시 검색하지 않는다.

같은 XML 안에서
다른 Statement ID만 확인한다.

---

## 26. XML Fallback

Mapper Type namespace로 찾지 못한 경우에만
Mapper Method ID를 사용할 수 있다.

검색 범위는:

CURRENT_PROJECT/src/main/resources/**

로 제한한다.

---

## 27. Statement

실제 Mapper Method와 연결되는
Statement만 찾는다.

대상:

<select>
<insert>
<update>
<delete>

---

## 28. XML 부분 Read

Statement 시작 위치에서
약 40줄만 먼저 Read한다.

다음 종료 Tag가 보이면
즉시 종료한다.

</select>
</insert>
</update>
</delete>

필요할 때만 추가 Read한다.

전체 XML은 읽지 않는다.

---

## 29. CHECKPOINT 4

XML Statement가 확보되면 출력한다.

[BE-TRACE] CHECKPOINT 4/5 - MYBATIS XML FOUND

Statements:

1.

Mapper:
{MAPPER_TYPE}#{MAPPER_METHOD}

XML:
{XML_PATH}

Statement:
{STATEMENT_ID}

---

# PHASE 5. SQL

## 30. SQL 분석

다음만 확인한다.

- SQL Type
- Main Table
- JOIN Table
- WHERE
- Parameter
- Dynamic SQL 존재 여부
- include 존재 여부

---

## 31. SQL Type

SELECT

INSERT

UPDATE

DELETE

중 하나로 기록한다.

---

## 32. Table

Source에서 직접 확인되는 Table만 기록한다.

- Main Table
- JOIN Table

추측하지 않는다.

---

## 33. Parameter

SQL에서 직접 사용되는
MyBatis Parameter만 확인한다.

예:

#{materialId}

#{plantCode}

${value}

---

## 34. Dynamic SQL

다음 Tag가 존재하면:

<if>
<choose>
<when>
<otherwise>
<foreach>

Dynamic SQL:
YES

없으면:

Dynamic SQL:
NO

로 기록한다.

---

## 35. include

<include refid="..."/>

가 있으면:

Include:
{refid}

만 기록한다.

내부는 추적하지 않는다.

---

## 36. CHECKPOINT 5

SQL 기본 정보가 확보되면 출력한다.

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

---

# PHASE 6. Final

## 37. 최종 출력

파일은 생성하지 않는다.

다음 형식으로 출력한다.

=== BE TRACE TEST v0.2 FAST ===

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

File:
{SERVICE_IMPL_FILE}

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

Evidence:
{PATH:LINES}

TRACE

{CONTROLLER_CLASS}#{CONTROLLER_METHOD}
→ {SERVICE_TYPE}#{SERVICE_METHOD}
→ {SERVICE_IMPL_CLASS}#{SERVICE_METHOD}
→ {MAPPER_TYPE}#{MAPPER_METHOD}
→ {STATEMENT_ID}
→ {SQL_TYPE} {MAIN_TABLE}

DEEP TRACE INPUT

SERVICE_IMPL_FILE:
{SERVICE_IMPL_FILE}

SERVICE_IMPL_CLASS:
{SERVICE_IMPL_CLASS}

SERVICE_METHOD:
{SERVICE_METHOD}

STATUS:
COMPLETED

---

## 38. Evidence

Evidence는 Source 탐색 중
이미 확인한 위치를 사용한다.

Evidence 때문에
추가 검색하지 않는다.

Project Root 기준 상대경로를 사용한다.

절대경로는 출력하지 않는다.

---

## 39. 실패

찾지 못하면 검색 범위를 확장하지 않는다.

CONTROLLER_NOT_FOUND

SERVICE_NOT_FOUND

SERVICE_IMPL_NOT_FOUND

MAPPER_TYPE_NOT_FOUND

MAPPER_XML_NOT_FOUND

STATEMENT_NOT_FOUND

SQL_NOT_FOUND

---

## 40. 완료

최종 결과 출력 후 즉시 종료한다.

추가 Source 탐색을 하지 않는다.

다음 분석을 자동 실행하지 않는다.