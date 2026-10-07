---
name: be-simple-test
description: Backend URL에서 실제 호출되는 실행 경로를 추적하고 Mapper는 MyBatis XML 파일 위치까지만 확인한다. XML 내용과 SQL은 읽지 않는다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Simple Test v0.2 - XML Location Only

## 1. 목적

Backend URL에서 시작하여
실제 Source 호출 관계만 따라가며
Backend 실행 흐름을 확인한다.

기본 흐름:

Backend URL
→ Controller
→ Service
→ ServiceImpl
→ Local/private Method
→ 다른 Business Service
→ Mapper
→ MyBatis XML 파일 위치

핵심 원칙:

검색 범위는 좁게 유지하고
실제 호출 깊이는 끝까지 따라간다.

MyBatis는 XML 파일 위치만 확인한다.

XML 파일 내용은 절대 읽지 않는다.

---

## 2. 입력

HTTP Method

Backend URL

예:

POST /material/create

또는:

UNKNOWN /material/create

---

## 3. 분석 범위

실제 Source에 존재하는 경우 다음을 추적한다.

Controller

Service

ServiceImpl

Local/private Method

다른 Business Service

Mapper

MyBatis XML 파일 위치

모든 단계가 반드시 존재할 필요는 없다.

실제 호출되는 흐름만 분석한다.

---

## 4. 하지 않는 것

다음은 수행하지 않는다.

- MyBatis XML Read
- Statement ID 검색
- Statement Body 검색
- SQL 분석
- SQL Query 확인
- Table 분석
- JOIN 분석
- WHERE 분석
- Parameter 분석
- Dynamic SQL 분석
- include 추적
- resultMap 추적
- Oracle Metadata
- Database Metadata
- Response 상세 분석
- Exception 상세 분석
- BE-REFERENCE
- Markdown 문서 생성
- Write
- Agent
- 병렬 Worker
- Code Index
- Mapper.java 상세 분석
- Dependency/JAR 내부 분석
- 관련 Source 사전 수집

---

# Source Boundary

## 5. Java

Java Source:

gipms-api-*/src/main/java/**

Controller가 발견된 프로젝트를:

CURRENT_PROJECT

로 사용한다.

---

## 6. Resources

MyBatis XML 파일 위치 검색 범위:

CURRENT_PROJECT/src/main/resources/**

다른 Backend 프로젝트의 Resources까지
검색 범위를 확장하지 않는다.

중요:

resources 안의 XML은
파일 위치 확인 용도로만 검색한다.

XML 내용을 Read하지 않는다.

---

## 7. Hard Exclude

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

Dependency/JAR 내부로 이동하지 않는다.

---

# Search

## 8. 탐색 방식

항상:

정확한 Symbol
→ Grep
→ 위치 확보
→ 필요한 Java 부분만 Read
→ 다음 Symbol

순서로 진행한다.

MyBatis XML은 예외다.

XML은:

Mapper Type
→ Grep
→ XML 파일 경로 확보
→ STOP

으로 처리한다.

XML에 Read를 실행하지 않는다.

---

## 9. 금지

다음 방식은 사용하지 않는다.

Repository 전체 구조 파악

전체 Java 파일 목록 수집

전체 XML 파일 목록 수집

전체 Service 목록 수집

전체 Mapper 목록 수집

관련 Source 사전 탐색

관련 있어 보인다는 이유의 Source 탐색

전체 파일 기본 Read

XML 파일 Read

---

## 10. Cache

이미 확보한 정보는 재사용한다.

다음 조합은 다시 분석하지 않는다.

Class + Method

Mapper Type + Mapper Method

Mapper Type + XML Path

같은 Symbol을 반복 Grep하지 않는다.

같은 Mapper Type의 XML 위치를
반복 검색하지 않는다.

---

# Controller

## 11. Controller 탐색

Backend URL에서
식별력이 높은 Mapping 문자열로
Controller를 찾는다.

확인:

class-level mapping

+

method-level mapping

+

HTTP Method

HTTP Method가 UNKNOWN이면
Mapping Annotation에서 확인한다.

---

## 12. Controller 분석

실제 요청을 처리하는
Controller Method만 읽는다.

전체 Controller 파일을
기본적으로 읽지 않는다.

---

## 13. Controller에서 확보

다음을 확보한다.

Controller Class

Controller Method

Service Type

Service Method

CURRENT_PROJECT

---

# Service

## 14. Service

Controller에서 실제 호출되는
Service만 따라간다.

전체 Service 목록을 찾지 않는다.

실제 호출된 Method만 확인한다.

---

## 15. ServiceImpl

정확한 Service 구현체만 찾는다.

실제 호출된 Method 부분만 읽는다.

전체 ServiceImpl 파일을
기본적으로 읽지 않는다.

---

# Execution Flow

## 16. 실제 호출 기준

현재 Method에서
실제로 호출되는 코드만 추적한다.

관련 있어 보인다는 이유로
다른 Source를 찾지 않는다.

---

# Local/private Method

## 17. Local/private Method

현재 실행 경로에서
실제로 호출되는 Local/private Method만 따라간다.

동일 Class 안에서
실제 선언이 확인되는 Method만 처리한다.

Local Method 목록을
미리 만들지 않는다.

---

## 18. Local 중복 방지

이미 분석한:

Class + Method

는 다시 읽지 않는다.

순환 호출이면
호출 관계만 기록한다.

---

# Business Service

## 19. 다른 Business Service

현재 Method에서 실제 호출되는
다른 Business Service만 따라간다.

예:

Service A
→ Service B
→ Service C

실제 호출 관계가 있으면
계속 따라간다.

전체 Service 목록을
미리 검색하지 않는다.

---

## 20. Service 왕복

실제 Source에 다음과 같은 흐름이 있으면
그대로 따라간다.

Service A
→ Service B
→ Service A의 다른 Method
→ Service C

이미 분석한 동일:

Class + Method

만 다시 읽지 않는다.

---

# Mapper

## 21. Mapper

실제 실행 경로에서 발견된
Mapper 호출만 처리한다.

확보:

Mapper Variable

Mapper Type

Mapper Method

---

## 22. Mapper Type

현재 읽은 Java Source에서
Mapper Type이 확인되면 그대로 사용한다.

확인되지 않은 경우에만
현재 ServiceImpl Java 파일 안에서
Mapper Variable 선언을 찾는다.

Repository 전체에서
Mapper Variable을 검색하지 않는다.

---

## 23. Mapper.java

Mapper Java Interface는
기본적으로 읽지 않는다.

가능하면:

ServiceImpl
→ Mapper Type
→ Mapper Method
→ MyBatis XML 파일 위치

로 바로 이동한다.

Mapper Method 상세 확인을 위해
Mapper.java를 Read하지 않는다.

---

# MyBatis

## 24. MyBatis 목표

MyBatis에서는 오직:

XML 파일 위치

만 확인한다.

Statement ID를 확인하지 않는다.

SQL을 확인하지 않는다.

---

## 25. XML 위치 탐색

Mapper Type을 이용하여
현재 Backend 프로젝트의 resources에서
연결되는 XML 파일을 찾는다.

검색 범위:

CURRENT_PROJECT/src/main/resources/**

가능하면 Mapper Type의
정확한 namespace 문자열을 Grep한다.

예:

Mapper Type:

MaterialMapper

검색:

namespace="...MaterialMapper"

---

## 26. XML 발견 즉시 종료

Mapper Type과 연결되는
정확한 XML 파일 경로가 발견되면:

XML_PATH

만 저장한다.

그 즉시 해당 Mapper의
MyBatis 탐색을 종료한다.

---

## 27. XML 절대 Read 금지

중요:

발견된 MyBatis XML에
Read를 실행하지 않는다.

XML 파일을 열지 않는다.

Statement를 찾지 않는다.

Statement ID를 찾지 않는다.

Mapper Method 이름을
XML 내부에서 검색하지 않는다.

SQL을 찾지 않는다.

---

## 28. XML Grep 제한

XML에 대한 Grep은
Mapper Type과 연결되는 XML 파일 위치를
찾기 위한 1회 검색만 허용한다.

XML 경로가 확인된 이후에는
해당 XML을 대상으로
추가 Grep을 실행하지 않는다.

즉:

Mapper Type
→ XML Path 확인
→ STOP

이다.

---

## 29. XML 경로 Cache

동일 Mapper Type이 다시 등장하면
이미 확보한 XML_PATH를 재사용한다.

XML을 다시 검색하지 않는다.

---

# External

## 30. SAP / RFC / External

실제 실행 경로에서
외부 호출이 발견되면 이름만 기록한다.

예:

sapService.send(...)

rfcClient.execute(...)

externalClient.call(...)

이번 테스트에서는
외부 연동 내부 Source로
추가 이동하지 않는다.

---

# Evidence

## 31. Evidence

분석하면서 이미 확인한 Source 위치를 사용한다.

Evidence를 만들기 위해
추가 Grep 또는 Read를 하지 않는다.

형식:

프로젝트 루트 기준 상대경로:라인범위

절대경로는 출력하지 않는다.

MyBatis Evidence는:

XML 파일 경로

까지만 사용한다.

XML Line Number는 요구하지 않는다.

---

# Output

## 32. 결과

파일을 생성하지 않는다.

화면에만 출력한다.

형식:

=== BE SIMPLE TEST v0.2 - XML LOCATION ONLY ===

INPUT

Method:
{HTTP_METHOD}

URL:
{BACKEND_URL}


EXECUTION FLOW

{CONTROLLER_CLASS}#{CONTROLLER_METHOD}

↓

{SERVICE_TYPE}#{SERVICE_METHOD}

↓

{실제 Local/private Method}

↓

{실제 다른 Business Service}

↓

{MAPPER_TYPE}#{MAPPER_METHOD}

↓

MyBatis XML
{XML_PATH}


CONTROLLER

Class:
{CONTROLLER_CLASS}

Method:
{CONTROLLER_METHOD}

Evidence:
{PATH:LINES}


SERVICE FLOW

1.

Class:
{CLASS}

Method:
{METHOD}

Calls:
{CALLS}

Evidence:
{PATH:LINES}


LOCAL METHODS

실제 존재하는 경우만 출력한다.

1.

Class:
{CLASS}

Method:
{METHOD}

Called From:
{CALLER}

Evidence:
{PATH:LINES}


BUSINESS SERVICES

실제 존재하는 경우만 출력한다.

1.

Service:
{SERVICE}

Method:
{METHOD}

Called From:
{CALLER}

Evidence:
{PATH:LINES}


MAPPERS

1.

Mapper:
{MAPPER_TYPE}

Method:
{MAPPER_METHOD}

XML:
{XML_PATH}

XML Read:
NO

Statement ID:
NOT CHECKED

SQL:
NOT ANALYZED


EXTERNAL CALLS

실제 확인된 경우만 출력한다.

- {CALL}

없으면:

NONE


STATUS

COMPLETED

---

# Failure

## 33. 탐색 실패

특정 Source를 찾지 못해도
검색 범위를 무작정 확장하지 않는다.

필요하면 기록한다.

CONTROLLER_NOT_FOUND

SERVICE_NOT_FOUND

SERVICE_IMPL_NOT_FOUND

MAPPER_TYPE_NOT_FOUND

MAPPER_XML_NOT_FOUND

---

# STOP

## 34. 종료

실제 실행 흐름에 대해:

Controller

Service

Local/private Method

Business Service

Mapper

MyBatis XML 파일 위치

확인이 끝나면 즉시 종료한다.

XML을 Read하지 않는다.

Statement ID를 찾지 않는다.

SQL을 읽지 않는다.

include를 따라가지 않는다.

resultMap을 따라가지 않는다.

추가 후보 Source를 탐색하지 않는다.

BE-REFERENCE를 읽지 않는다.

문서를 생성하지 않는다.

다른 Skill을 실행하지 않는다.