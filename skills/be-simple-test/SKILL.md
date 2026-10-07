---
name: be-simple-test
description: Backend URL에서 실제 호출되는 실행 경로를 추적하고 MyBatis XML의 실제 Statement Body까지만 읽어 SQL Read 비용을 측정한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Simple Test v0.3 - MyBatis Statement Read

## 1. 목적

Backend URL에서 시작하여
실제 Source 호출 관계만 따라간다.

기본 흐름:

Backend URL
→ Controller
→ Service
→ ServiceImpl
→ Local/private Method
→ 다른 Business Service
→ Mapper
→ MyBatis XML
→ Statement ID
→ Statement Body Read
→ 종료

이번 버전의 목적은 하나다.

MyBatis Statement Body를 읽는 비용을 측정한다.

SQL 내용은 읽지만
SQL을 분석하거나 해석하지 않는다.

---

## 2. 기준 버전

이전 v0.2 테스트:

MyBatis XML
+
Statement ID

까지만 확인했을 때:

57초

이번 버전은
v0.2에 Statement Body Read만 추가한다.

다른 분석 범위는 늘리지 않는다.

---

## 3. 입력

HTTP Method

Backend URL

예:

POST /material/create

또는:

UNKNOWN /material/create

---

## 4. 이번 버전에서 추가되는 것

v0.2:

Mapper
→ XML
→ Statement ID
→ STOP

v0.3:

Mapper
→ XML
→ Statement ID
→ Statement Body Read
→ STOP

추가 기능은 이것 하나뿐이다.

---

## 5. 이번 버전에서 하지 않는 것

다음은 수행하지 않는다.

- SQL 의미 해석
- SQL 요약
- Main Table 분석
- JOIN 분석
- WHERE 분석
- Parameter 분석
- Dynamic SQL 분석
- include 내부 분석
- resultMap 분석
- Request → SQL Mapping
- Oracle Metadata
- BE-REFERENCE
- Markdown 문서 생성
- Write
- Agent
- 병렬 Worker
- Code Index
- Mapper.java 상세 분석
- Dependency/JAR 내부 분석

Statement Body를 읽은 뒤
SQL에 대한 추가 판단을 하지 않는다.

---

# 핵심 규칙

## 6. 실행 경로

다음 실제 실행 경로를 따라간다.

Controller
→ Service
→ ServiceImpl
→ 실제 Local/private Method
→ 실제 Business Service
→ 실제 Mapper
→ MyBatis XML
→ 실제 Statement
→ Statement Body

모든 단계가 반드시 존재할 필요는 없다.

Source에 실제 존재하는 흐름만 추적한다.

---

## 7. 실제 호출 기준

현재 Method에서
실제로 호출되는 코드만 따라간다.

관련 있어 보인다는 이유로
Source를 추가 탐색하지 않는다.

---

## 8. 중복 분석 금지

이미 확인한:

Class + Method

Mapper Type + Mapper Method

Mapper Type + XML

XML + Statement ID

조합은 다시 탐색하지 않는다.

---

# Source Boundary

## 9. Java

Java Source:

gipms-api-*/src/main/java/**

Controller가 발견된 프로젝트를:

CURRENT_PROJECT

로 사용한다.

---

## 10. Resources

MyBatis:

CURRENT_PROJECT/src/main/resources/**

안에서만 탐색한다.

다른 Backend 프로젝트로
Resources 검색 범위를 확장하지 않는다.

---

## 11. Hard Exclude

탐색하지 않는다.

- docs/**
- **/sample/**
- **/samples/**
- **/test/**
- **/tests/**
- **/target/**
- **/build/**
- **/generated/**
- **/*.jar
- **/*.class
- **/node_modules/**
- **/.git/**
- **/.gradle/**
- **/.m2/**
- **/libs/**
- **/lib/**
- **/BOOT-INF/**
- **/WEB-INF/lib/**

---

# Search

## 12. 탐색 방식

정확한 Symbol

→ Grep

→ 위치 확보

→ 필요한 범위만 Read

→ 다음 Symbol

순서로 진행한다.

---

## 13. 금지

다음은 금지한다.

Repository 전체 구조 파악

전체 Java 목록 수집

전체 XML 목록 수집

전체 Service 목록 수집

전체 Mapper 목록 수집

관련 Source 사전 탐색

전체 파일 기본 Read

---

# Controller

## 14. Controller

Backend URL과 HTTP Method로
Controller를 찾는다.

class-level mapping

+

method-level mapping

을 확인한다.

UNKNOWN이면
Annotation에서 Method를 확인한다.

---

## 15. Controller에서 확보

다음만 확보한다.

Controller Class

Controller Method

Service Type

Service Method

CURRENT_PROJECT

---

# Service

## 16. Service

Controller에서 실제 호출되는
Service만 따라간다.

실제 호출된 Method만 확인한다.

---

## 17. ServiceImpl

정확한 Service 구현체만 찾는다.

실제 호출된 Method 부분만 읽는다.

전체 ServiceImpl 파일을
기본적으로 읽지 않는다.

---

# Local/private

## 18. Local/private Method

현재 실행 경로에서
실제로 호출된 Local/private Method만 따라간다.

동일 Class 안에서
실제 선언이 확인되는 Method만 처리한다.

Local Method 목록을
미리 만들지 않는다.

---

## 19. Local 중복 방지

이미 분석한 Local Method는
다시 읽지 않는다.

순환 호출이면
호출 관계만 기록한다.

---

# Business Service

## 20. 다른 Business Service

현재 Method에서 실제 호출되는
Business Service만 따라간다.

예:

Service A
→ Service B
→ Service C

실제 호출 관계가 있으면
계속 따라간다.

전체 Service 목록을
검색하지 않는다.

---

## 21. Service 중복 방지

이미 분석한:

Class + Method

는 다시 읽지 않는다.

---

# Mapper

## 22. Mapper

실제 실행 경로에서 발견된
Mapper 호출만 처리한다.

확보:

Mapper Variable

Mapper Type

Mapper Method

---

## 23. Mapper Type

현재 읽은 Source에서
Mapper Type이 확인되면 재사용한다.

확인되지 않은 경우에만
현재 ServiceImpl 파일 안에서
Mapper Variable을 찾는다.

Repository 전체에서
Mapper Variable을 검색하지 않는다.

---

## 24. Mapper.java

Mapper Java Interface는
기본적으로 읽지 않는다.

가능하면:

ServiceImpl
→ Mapper Type
→ Mapper Method
→ MyBatis XML

로 바로 이동한다.

---

# MyBatis XML

## 25. XML 탐색

Mapper Type으로
정확한 namespace를 찾는다.

검색 범위:

CURRENT_PROJECT/src/main/resources/**

예:

namespace="...MaterialMapper"

---

## 26. XML 확정

정확한 Mapper XML이 발견되면
다른 XML 탐색을 즉시 중단한다.

Mapper Type:

MaterialMapper

와 연결되는 XML을 한 번 찾으면
같은 Mapper Type에서는 재사용한다.

---

## 27. Statement ID

확정된 XML 안에서만
Mapper Method와 같은 Statement ID를 찾는다.

예:

Mapper Method:

selectMaterial

이면:

id="selectMaterial"

을 찾는다.

---

# Statement Body Read

## 28. 핵심 테스트

Statement ID가 확인되면
해당 Statement Body를 읽는다.

예:

<select id="selectMaterial">

부터:

</select>

까지.

또는:

<insert>
→ </insert>

<update>
→ </update>

<delete>
→ </delete>

까지.

---

## 29. 부분 Read

Statement 시작 위치에서:

약 30줄

만 먼저 Read한다.

종료 Tag가 보이면
즉시 Read를 종료한다.

---

## 30. 추가 Read

30줄 안에 종료 Tag가 없을 때만
다음 범위를 읽는다.

추가 Read 역시
필요한 범위만 수행한다.

전체 XML 파일을 읽지 않는다.

---

## 31. 긴 Statement

Statement가 길어도
종료 Tag를 찾기 위해서만
추가 Read한다.

SQL을 이해하기 위해
범위를 추가 확장하지 않는다.

---

## 32. SQL 분석 금지

Statement Body를 읽은 후
다음을 분석하지 않는다.

SELECT 의미

INSERT 의미

UPDATE 의미

DELETE 의미

Table

JOIN

WHERE

Parameter

Dynamic SQL

Business 의미

---

## 33. Dynamic SQL

다음 Tag가 보여도
분석하지 않는다.

<if>

<choose>

<when>

<otherwise>

<foreach>

단순히 Statement Body의 일부로 읽고
추가 탐색하지 않는다.

---

## 34. include

<include refid="..."/>

가 보여도
refid만 따라가지 않는다.

include Source를 찾지 않는다.

---

## 35. resultMap

resultMap이 보여도
정의로 이동하지 않는다.

---

## 36. Statement Cache

동일:

XML + Statement ID

를 이미 읽었다면
다시 Read하지 않는다.

---

# External

## 37. SAP / RFC / External

실제 실행 흐름에서 발견되면
호출 이름만 기록한다.

내부 Source로 들어가지 않는다.

예:

sapService.send(...)

External Call:
sapService.send

---

# Evidence

## 38. Evidence

분석하면서 이미 확인한 위치를 사용한다.

Evidence 때문에
추가 Grep 또는 Read를 하지 않는다.

형식:

프로젝트 루트 기준 상대경로:라인범위

절대경로는 사용하지 않는다.

---

# Output

## 39. 출력

파일을 생성하지 않는다.

화면에만 출력한다.

형식:

=== BE SIMPLE TEST v0.3 ===

INPUT

Method:
{HTTP_METHOD}

URL:
{BACKEND_URL}


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


EXECUTION FLOW

{CONTROLLER_CLASS}#{CONTROLLER_METHOD}

→ {SERVICE}

→ {실제 Local/private 또는 Business Service}

→ {MAPPER_TYPE}#{MAPPER_METHOD}

→ {XML_PATH}#{STATEMENT_ID}


LOCAL METHODS

실제 추적된 경우만 출력한다.

- {CLASS}#{METHOD}
- Evidence: {PATH:LINES}


BUSINESS SERVICES

실제 추적된 경우만 출력한다.

- {SERVICE}#{METHOD}
- Evidence: {PATH:LINES}


MAPPERS

1.

Mapper:
{MAPPER_TYPE}

Method:
{MAPPER_METHOD}

XML:
{XML_PATH}

Statement:
{STATEMENT_ID}

Statement Lines:
{START_LINE}-{END_LINE}

Evidence:
{PATH:LINES}


STATEMENT BODY

1.

Mapper:
{MAPPER_TYPE}#{MAPPER_METHOD}

Statement:
{STATEMENT_ID}

Read:
YES

Lines:
{START_LINE}-{END_LINE}

SQL Analysis:
SKIPPED


EXTERNAL CALLS

실제 발견된 경우만 출력한다.

- {CALL}


STATUS

COMPLETED

---

# 실패

## 40. 탐색 실패

Source를 찾지 못해도
검색 범위를 무작정 확장하지 않는다.

필요하면 다음을 기록한다.

CONTROLLER_NOT_FOUND

SERVICE_NOT_FOUND

SERVICE_IMPL_NOT_FOUND

MAPPER_TYPE_NOT_FOUND

MAPPER_XML_NOT_FOUND

STATEMENT_NOT_FOUND

STATEMENT_END_NOT_FOUND

---

# STOP

## 41. 종료 조건

실제 Mapper 호출에 대해:

Mapper XML 확인

+

Statement ID 확인

+

Statement Body Read

가 끝나면 종료한다.

SQL 분석을 시작하지 않는다.

include를 따라가지 않는다.

resultMap을 따라가지 않는다.

추가 Source를 탐색하지 않는다.

BE-REFERENCE를 읽지 않는다.

문서를 생성하지 않는다.

다른 Skill을 실행하지 않는다.