---
name: be-simple-test
description: Backend URL에서 실제 호출되는 실행 경로를 추적하되 MyBatis는 XML 파일과 Statement ID까지만 확인하여 탐색 성능을 측정한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Simple Test v0.2 - MyBatis Minimal

## 1. 목적

Backend URL에서 시작하여
실제 Source 호출 관계만 따라간다.

이번 버전의 핵심 테스트:

MyBatis SQL 분석을 제거하고

Mapper
→ XML 파일
→ Statement ID

까지만 확인한다.

이를 통해 MyBatis XML/SQL 분석이
전체 Backend 분석 시간의 병목인지 확인한다.

---

## 2. 입력

HTTP Method

Backend URL

예:

POST /material/create

또는:

UNKNOWN /material/create

---

## 3. 분석 흐름

실제 호출이 존재하는 만큼 다음을 추적한다.

Backend URL

→ Controller

→ Service

→ ServiceImpl

→ Local/private Method

→ 다른 Business Service

→ Mapper

→ MyBatis XML

→ Statement ID

→ 종료

모든 단계가 반드시 존재할 필요는 없다.

실제 Source에 존재하는 흐름만 분석한다.

---

## 4. 이번 버전에서 하지 않는 것

다음은 수행하지 않는다.

- SQL 내용 Read
- SQL 해석
- SELECT / INSERT / UPDATE / DELETE 분석
- Main Table 분석
- JOIN 분석
- WHERE 분석
- Parameter 분석
- Dynamic SQL 분석
- include 분석
- resultMap 분석
- Oracle Metadata
- BE-REFERENCE
- Markdown 문서 생성
- Write
- Agent
- 병렬 Worker
- Code Index
- Mapper.java 상세 분석
- Dependency/JAR 내부 분석

MyBatis는:

XML 파일

+

Statement ID

확인 즉시 종료한다.

---

## 5. 핵심 규칙

1. Backend URL과 HTTP Method로 Controller를 찾는다.

2. Controller에서 실제 호출되는 Service를 따라간다.

3. 현재 Method에서 실제 호출되는 코드만 추적한다.

4. 같은 Class의 Local/private Method가 실제 호출되면 따라간다.

5. 다른 Business Service가 실제 호출되면 해당 Method를 따라간다.

6. Mapper 호출이 발견되면 Mapper Type과 Mapper Method를 확보한다.

7. Mapper Type으로 MyBatis XML을 찾는다.

8. 해당 XML에서 Mapper Method와 같은 Statement ID가 존재하는지만 확인한다.

9. Statement ID가 확인되면 해당 Statement Body를 읽지 않는다.

10. 이미 분석한 Class + Method + Mapper + XML은 다시 탐색하지 않는다.

---

# Source Boundary

## 6. Java

Java Source:

gipms-api-*/src/main/java/**

Controller가 발견된 프로젝트를:

CURRENT_PROJECT

로 사용한다.

---

## 7. Resources

MyBatis 탐색 범위:

CURRENT_PROJECT/src/main/resources/**

다른 Backend 프로젝트의 Resources까지
검색 범위를 확장하지 않는다.

---

## 8. Hard Exclude

다음은 탐색하지 않는다.

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

# FAST Search

## 9. 기본 탐색 방식

항상:

정확한 문자열

→ Grep

→ 위치 확보

→ 필요한 부분만 Read

→ 다음 Symbol

순서로 진행한다.

전체 파일을 기본적으로 읽지 않는다.

---

## 10. 금지

다음 방식으로 탐색하지 않는다.

Repository 전체 구조 파악

전체 Java 파일 목록 수집

전체 XML 파일 목록 수집

전체 Service 목록 수집

전체 Mapper 목록 수집

관련 있어 보이는 Source 사전 탐색

---

# Controller

## 11. Controller 탐색

Backend URL에서
식별력이 높은 Mapping 문자열로
Controller 후보를 찾는다.

class-level mapping

+

method-level mapping

+

HTTP Method

를 확인한다.

HTTP Method가 UNKNOWN이면
Annotation에서 확인한다.

---

## 12. Controller에서 확보

다음만 확보한다.

Controller Class

Controller Method

Service Type

Service Method

CURRENT_PROJECT

---

# Service

## 13. Service

Controller에서 실제 호출된
Service만 추적한다.

전체 Service 목록을 찾지 않는다.

실제 호출된 Method만 확인한다.

---

## 14. ServiceImpl

정확한 Service Type의
구현체만 찾는다.

실제 호출된 Method 부분만 읽는다.

전체 ServiceImpl 파일을
기본적으로 읽지 않는다.

---

# Execution Trace

## 15. 실제 호출 기준

ServiceImpl Method에서
실제로 호출되는 대상만 추적한다.

관련 있어 보인다는 이유로
다른 Source를 찾지 않는다.

---

## 16. Local/private Method

현재 실행 경로에서
실제로 호출된 Local/private Method만 따라간다.

동일 Class 안에서 확인 가능한 경우에만
Local Method로 처리한다.

관련 Local Method 목록을
미리 만들지 않는다.

---

## 17. 다른 Business Service

현재 Method에서 실제 호출되는
다른 Business Service만 추적한다.

예:

Service A

→ Service B

→ Service C

실제 호출 관계가 있으면 따라간다.

전체 Service 목록을
미리 검색하지 않는다.

---

## 18. 중복 방지

이미 분석한:

Class + Method

조합은 다시 읽지 않는다.

순환 호출이 발생하면
호출 관계만 기록하고
재분석하지 않는다.

---

# Mapper

## 19. Mapper 호출

실제 실행 경로에서 발견된
Mapper 호출만 처리한다.

예:

materialMapper.selectMaterial(...)

확보:

Mapper Variable

Mapper Type

Mapper Method

---

## 20. Mapper Type

현재 읽은 Source에서
Mapper Type이 확인되면 그대로 사용한다.

보이지 않을 때만
현재 ServiceImpl 파일 안에서
Mapper Variable을 찾는다.

Repository 전체에서
Mapper Variable을 검색하지 않는다.

---

## 21. Mapper.java

Mapper.java는
기본적으로 읽지 않는다.

가능하면:

ServiceImpl

→ Mapper Type

→ Mapper Method

→ MyBatis XML

로 바로 이동한다.

---

# MyBatis Minimal

## 22. 가장 중요한 규칙

MyBatis에서 SQL을 분석하지 않는다.

이번 테스트에서는:

Mapper Type

→ XML

→ Statement ID

까지만 확인한다.

---

## 23. XML 탐색 범위

XML 검색은 반드시:

CURRENT_PROJECT/src/main/resources/**

범위로 제한한다.

다른 프로젝트까지 검색하지 않는다.

---

## 24. Namespace 우선

Mapper Type을 알고 있으면
정확한 namespace 문자열로 XML을 찾는다.

예:

MaterialMapper

를 알고 있다면:

namespace="...MaterialMapper"

를 찾는다.

정확한 XML이 발견되면
XML 검색을 즉시 중단한다.

---

## 25. XML Cache

Mapper Type과 XML 관계를
한 번 확인하면 재사용한다.

예:

MaterialMapper

→ material/MaterialMapper.xml

을 한 번 찾았다면

MaterialMapper의 다른 Method 때문에
namespace를 다시 검색하지 않는다.

---

## 26. Statement ID

XML 파일이 확정되면
해당 XML 안에서만
정확한 Statement ID를 찾는다.

예:

Mapper Method:

selectMaterial

이면:

id="selectMaterial"

만 찾는다.

---

## 27. Statement Body 금지

다음이 확인되면:

<select id="selectMaterial">

또는:

<insert id="insertMaterial">

또는:

<update id="updateMaterial">

또는:

<delete id="deleteMaterial">

Statement가 존재한다고 판단한다.

그 아래 SQL Body를 읽지 않는다.

---

## 28. XML 전체 Read 금지

MyBatis XML 전체를 읽지 않는다.

Statement 주변 40줄 Read도 하지 않는다.

Grep 결과로 Statement ID가 확인되면
그것으로 종료한다.

---

## 29. include 금지

다음을 분석하지 않는다.

<include>

<sql>

<resultMap>

<if>

<choose>

<foreach>

이번 테스트에서는
존재 여부조차 확인할 필요 없다.

---

## 30. XML Fallback

Mapper Type namespace로
XML을 찾지 못한 경우에만
Mapper Method ID를 사용할 수 있다.

검색:

CURRENT_PROJECT/src/main/resources/**

에서:

id="{MAPPER_METHOD}"

를 찾는다.

후보가 여러 개 나오면
Mapper namespace와 일치하는 XML만 선택한다.

후보 XML 전체를 읽지 않는다.

---

# External

## 31. SAP / RFC / External

실제 실행 경로에서 발견되면
호출 이름만 기록한다.

이번 테스트에서는
외부 연동 내부로 들어가지 않는다.

예:

sapService.send(...)

이면:

External Call:
sapService.send

만 기록한다.

---

# Evidence

## 32. Evidence

분석 중 이미 확인한 위치만 사용한다.

Evidence를 위해
추가 Grep 또는 Read를 하지 않는다.

형식:

프로젝트 루트 기준 상대경로:라인범위

절대경로는 출력하지 않는다.

---

# Output

## 33. 결과

파일을 생성하지 않는다.

화면에만 출력한다.

형식:

=== BE SIMPLE TEST v0.2 ===

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

→ {실제 Local/private 또는 Business Service 호출}

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

Evidence:
{PATH:LINE}


EXTERNAL CALLS

실제 발견된 경우만 출력한다.

- {CALL}


MYBATIS SQL ANALYSIS

SKIPPED


STATUS

COMPLETED

---

# 실패

## 34. 탐색 실패

특정 Source를 찾지 못해도
검색 범위를 무작정 확장하지 않는다.

필요한 경우 다음을 출력한다.

CONTROLLER_NOT_FOUND

SERVICE_NOT_FOUND

SERVICE_IMPL_NOT_FOUND

MAPPER_TYPE_NOT_FOUND

MAPPER_XML_NOT_FOUND

STATEMENT_NOT_FOUND

---

# STOP

## 35. 종료

모든 실제 Mapper에 대해:

XML

+

Statement ID

가 확인되면 종료한다.

SQL Body를 읽지 않는다.

추가 Source를 탐색하지 않는다.

BE-REFERENCE를 읽지 않는다.

문서를 생성하지 않는다.

다른 Skill을 실행하지 않는다.