---
name: be-trace-test
description: Backend URL 하나를 대상으로 Controller에서 ServiceImpl, 직접 Mapper 호출, MyBatis XML, SQL까지 최소 탐색으로 추적하여 순수 Backend 탐색 성능을 측정한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Trace Test v0.1

## 1. 목적

Backend 분석을 처음부터 다시 구축하기 위한
최소 성능 테스트 Skill이다.

이번 버전의 분석 범위는 다음뿐이다.

Backend URL
→ Controller
→ Controller Method
→ Service
→ ServiceImpl
→ 직접 Mapper 호출
→ MyBatis XML
→ SQL

정확성과 속도를 먼저 검증한다.

---

## 2. 입력

HTTP_METHOD

BACKEND_URL

예:

POST /material/create

또는:

UNKNOWN /material/create

---

## 3. 이번 버전에서 하지 않는 것

다음은 분석하지 않는다.

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
- 전체 Validation 분석
- 전체 Branch 분석
- resultMap 상세 분석
- 관련 있어 보이는 Class 탐색

이번 테스트에서는 탐색 범위를 절대 확장하지 않는다.

---

## 4. Source 범위

Backend Java:

gipms-api-*/src/main/java/**

MyBatis Resource:

gipms-api-*/src/main/resources/**

이 범위만 사용한다.

---

## 5. 제외 범위

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

Dependency/JAR fallback을 하지 않는다.

---

## 6. 핵심 탐색 원칙

항상 다음 순서를 사용한다.

정확한 문자열 검색
→ 위치 확인
→ 필요한 부분만 Read
→ 다음 Symbol 확보
→ 즉시 다음 단계

전체 파일을 먼저 읽지 않는다.

Repository 전체 구조를 먼저 파악하지 않는다.

관련 파일 목록을 미리 수집하지 않는다.

---

## 7. 중복 탐색 금지

한 번 찾은 정보는 재사용한다.

내부적으로 다음 정보를 유지한다.

CURRENT_PROJECT
CONTROLLER_FILE
CONTROLLER_CLASS
CONTROLLER_METHOD
SERVICE_TYPE
SERVICE_METHOD
SERVICE_IMPL_FILE
MAPPER_TYPES
MAPPER_METHODS
MAPPER_XML

같은 Symbol을 다시 Grep하지 않는다.

---

## 8. Controller 탐색

BACKEND_URL에서
가장 식별력이 높은 Path를 사용한다.

예:

/api/material/create

이면 우선:

create

또는:

/material/create

와 같이 Mapping을 식별할 수 있는 문자열을 사용한다.

검색 대상은 Java Source만 사용한다.

가능하면 Controller 파일을 우선한다.

---

## 9. Controller 확정

후보 Source에서:

- class-level RequestMapping
- method-level Mapping
- HTTP Method

를 확인한다.

조합 결과가 BACKEND_URL과 일치하는
Controller Method를 확정한다.

HTTP_METHOD가 UNKNOWN이면
Mapping Annotation으로 Method를 확인한다.

---

## 10. Controller 부분 Read

Controller Method 위치를 찾은 후
Method 주변만 읽는다.

초기 범위:

약 60줄

Method 종료가 보이지 않을 때만
추가 Read한다.

전체 Controller 파일 Read를 기본으로 하지 않는다.

---

## 11. Controller에서 확보할 정보

다음만 확보한다.

Controller Class
Controller Method
Service Variable
Service Type
Service Method

Controller의 상세 비즈니스 설명은 작성하지 않는다.

---

## 12. CURRENT_PROJECT

Controller가 위치한:

gipms-api-*

프로젝트를 CURRENT_PROJECT로 지정한다.

이후 기본 탐색은 CURRENT_PROJECT 안에서만 수행한다.

---

## 13. Service 탐색

Controller에서 확인한
정확한 Service Type을 사용한다.

CURRENT_PROJECT의:

src/main/java/**

에서 정확한 Type을 찾는다.

Service 전체를 검색하지 않는다.

---

## 14. Service Method

Controller에서 실제 호출한
Service Method 선언만 확인한다.

Service Interface 전체 분석은 하지 않는다.

Service Interface에 추가 설명이 필요하지 않으면
즉시 ServiceImpl 탐색으로 넘어간다.

---

## 15. ServiceImpl 탐색

정확한:

implements ServiceType

또는 확인된 구현 Class 이름을 사용한다.

CURRENT_PROJECT 안에서 찾는다.

구현체가 확정되면
다른 후보 탐색을 즉시 중단한다.

---

## 16. ServiceImpl Method

Controller에서 호출한
정확한 Service Method만 찾는다.

Method 시작 위치를 Grep한다.

초기:

약 80줄

만 Read한다.

Method 종료가 보이지 않을 때만
추가 범위를 읽는다.

---

## 17. 이번 테스트의 ServiceImpl 분석 범위

ServiceImpl Method Body에서
직접 호출되는 Mapper만 찾는다.

예:

materialMapper.selectMaterial(...)
materialMapper.insertMaterial(...)
historyMapper.insertHistory(...)

다음은 이번 버전에서 내부 추적하지 않는다.

validateMaterial(...)
calculateValue(...)
otherService.check(...)
sapService.send(...)

단, 이런 호출이 존재했다는 사실은
결과에 이름만 기록할 수 있다.

---

## 18. Mapper 호출 확보

각 직접 Mapper 호출에서:

Mapper Variable
Mapper Type
Mapper Method

를 확보한다.

예:

Variable:
materialMapper

Type:
MaterialMapper

Method:
selectMaterial

---

## 19. Mapper Type 확인

현재 읽은 ServiceImpl 범위에서
Mapper Type이 이미 보이면 그대로 사용한다.

보이지 않을 때만
현재 ServiceImpl 파일에서
Mapper Variable 이름을 정확히 Grep한다.

Type을 확보하면 즉시 중단한다.

---

## 20. Mapper.java

이번 FAST 테스트에서는
Mapper Java Interface를 기본적으로 읽지 않는다.

기본 흐름:

ServiceImpl
→ Mapper Type
→ Mapper Method
→ MyBatis XML

Mapper Java 확인은 하지 않는다.

---

## 21. MyBatis XML 탐색

CURRENT_PROJECT의:

src/main/resources/**

에서 Mapper Type과 연결되는 namespace를 찾는다.

예:

<mapper namespace="...MaterialMapper">

정확한 XML이 발견되면
다른 XML 탐색을 중단한다.

---

## 22. XML Statement 탐색

확정된 Mapper XML에서
실제 Mapper Method의 Statement만 찾는다.

예:

<select id="selectMaterial">

또는:

<insert id="insertMaterial">

---

## 23. XML 부분 Read

Statement 시작 위치에서:

약 40줄

만 먼저 읽는다.

다음 종료 Tag가 확인되면 종료한다.

</select>
</insert>
</update>
</delete>

40줄 안에 종료되지 않을 때만
추가 범위를 읽는다.

전체 XML 파일 Read는 하지 않는다.

---

## 24. SQL 분석

이번 테스트에서는 다음만 확인한다.

SQL Type
Main Table
JOIN Table
WHERE
Parameter
Dynamic SQL 존재 여부
include 존재 여부

SQL을 장문으로 설명하지 않는다.

---

## 25. Dynamic SQL

다음 Tag가 있으면
존재 여부와 핵심 조건만 기록한다.

<if>
<choose>
<when>
<otherwise>
<foreach>

상세 실행 트리는 아직 만들지 않는다.

---

## 26. include

다음과 같은 include가 있으면:

<include refid="..."/>

이번 버전에서는:

INCLUDE: refid

만 기록한다.

include 내부를 추가 탐색하지 않는다.

---

## 27. 종료 조건

다음까지 확보하면 즉시 종료한다.

Controller
Service
ServiceImpl
직접 Mapper Calls
Mapper XML
SQL Statements

이후 추가 탐색하지 않는다.

---

## 28. 결과 출력

파일을 생성하지 않는다.

화면에 다음 형식으로만 출력한다.

=== BE TRACE TEST ===

HTTP METHOD:
BACKEND URL:

CURRENT PROJECT:

CONTROLLER
- Class:
- Method:
- Evidence:

SERVICE
- Type:
- Method:

SERVICE IMPL
- Class:
- Method:
- Evidence:

DIRECT MAPPER CALLS

1.
- Mapper Type:
- Mapper Method:
- XML:
- Statement:
- SQL Type:
- Main Table:
- Evidence:

2.
- Mapper Type:
- Mapper Method:
- XML:
- Statement:
- SQL Type:
- Main Table:
- Evidence:

NON-TRACED CALLS
- Local Method:
- Other Service:
- External:

TRACE

Controller#method
→ Service#method
→ ServiceImpl#method
→ Mapper#method
→ XML Statement
→ SQL

STATUS:
COMPLETED

---

## 29. Evidence

Evidence는 Source를 읽는 과정에서 확인한
경로와 Line만 사용한다.

Evidence를 만들기 위해
Source를 다시 검색하지 않는다.

경로는 Project Root 기준 상대경로를 사용한다.

절대경로는 출력하지 않는다.

---

## 30. 탐색 실패

특정 Source를 찾지 못하면
검색 범위를 계속 넓히지 않는다.

다음과 같이 기록한다.

CONTROLLER_NOT_FOUND
SERVICE_NOT_FOUND
SERVICE_IMPL_NOT_FOUND
MAPPER_TYPE_NOT_FOUND
MAPPER_XML_NOT_FOUND
STATEMENT_NOT_FOUND

찾지 못한 이유를 확인하기 위해
JAR이나 Dependency까지 탐색하지 않는다.

---

## 31. 완료

결과 출력 후 즉시 종료한다.

추가 분석을 제안하거나
다음 단계를 자동 실행하지 않는다.