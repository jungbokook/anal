---
name: be-simple-test
description: Backend URL에서 시작해 실제 호출되는 코드만 끝까지 따라가고 MyBatis는 XML 위치와 Statement ID까지만 확인하는 최소 Backend 분석.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Simple Test v0.4 - Minimal Trace

## 목적

Backend URL에서 시작해 실제 실행 경로만 추적한다.

검색 범위는 좁게 유지하고
실제 호출 깊이는 끝까지 따라간다.

복잡한 단계별 분석을 하지 않는다.

---

## 입력

HTTP Method

Backend URL

예:

POST /material/create

UNKNOWN /material/create

---

## 분석 규칙

1. URL + HTTP Method로 Controller를 찾는다.

2. Controller에서 실제 호출되는 Method만 따라간다.

3. 현재 Method에서 실제 호출되는 Local/private Method와 다른 Business Service Method도 실제 호출이면 계속 따라간다.

4. 이미 확인한 `Class#Method`는 다시 읽지 않는다.

5. Mapper 호출이 나오면 `Mapper Type + Mapper Method`를 확인하고 MyBatis XML 위치와 Statement ID까지만 찾는다. SQL Body는 읽지 않는다.

6. 실행 경로를 읽으면서 이미 보이는 조건문, Validation, 값 변경, Return, Throw, Catch는 별도 검색 없이 같이 기록한다.

7. SAP/RFC/외부 연동이 실제 호출되면 프로젝트 Source에서 확인 가능한 실제 호출 경로만 따라간다. JAR/Dependency 내부는 보지 않는다.

8. 관련 있어 보인다는 이유만으로 Source를 찾지 않는다. 실제 호출 관계가 확인된 Source만 탐색한다.

9. Evidence는 분석 중 이미 확인한 위치를 사용한다. Evidence 때문에 다시 Grep/Read하지 않는다.

10. 실제 실행 경로가 끝나면 즉시 종료한다.

---

## 탐색 방식

가능하면 다음 방식만 사용한다.

정확한 Symbol
→ Grep
→ 필요한 부분만 Read
→ 다음 실제 호출 Symbol

전체 파일을 기본적으로 읽지 않는다.

전체 Repository 구조를 조사하지 않는다.

Service/Mapper/Local Method 후보 목록을 미리 만들지 않는다.

---

## Source 범위

Java:

gipms-api-*/src/main/java/**

MyBatis:

Controller가 발견된 현재 Backend 프로젝트의

src/main/resources/**

---

## 제외

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

Oracle Metadata를 조회하지 않는다.

Code Index를 사용하지 않는다.

Agent를 사용하지 않는다.

병렬 Worker를 사용하지 않는다.

---

## MyBatis

Mapper 호출이 실제 발견되면:

Mapper Type
+
Mapper Method

를 확인한다.

그 다음 현재 Backend 프로젝트의 resources에서
Mapper Type과 연결되는 MyBatis XML을 찾는다.

XML이 확인되면
해당 Mapper Method와 연결되는 Statement ID만 확인한다.

예:

MaterialMapper#selectMaterial

→

gipms-api-xxx/src/main/resources/.../MaterialMapper.xml

→

id="selectMaterial"

여기서 종료한다.

하지 않는다:

- Statement Body Read
- SQL 분석
- Table 분석
- JOIN 분석
- WHERE 분석
- Parameter 분석
- Dynamic SQL 분석
- include 추적
- resultMap 추적

---

## 실행 흐름 기록

Source를 읽으면서 이미 보이는 정보는
추가 탐색 없이 실행 흐름에 포함한다.

예:

조건 확인
→ 값 설정
→ Local Method
→ 다른 Service
→ Mapper
→ Return

또는:

조건 확인
→ Throw

조건/Return/Throw를 찾기 위한
별도 Repository 검색은 하지 않는다.

---

## External

SAP / RFC / 외부 연동이
실제 실행 경로에 존재하는 경우만 추적한다.

프로젝트 Source 안에서
실제 호출되는 Method를 따라간다.

프로젝트 Source 밖:

JAR

Library

Dependency

내부로는 이동하지 않는다.

Source에서 더 이상 확인할 수 없으면:

EXTERNAL BOUNDARY

로 기록하고 종료한다.

---

## Evidence

형식:

프로젝트 루트 기준 상대경로:라인범위

절대경로는 출력하지 않는다.

이미 분석하면서 확인한 위치만 사용한다.

---

## 출력

파일을 생성하지 않는다.

화면에 다음 형식으로 출력한다.

=== BE SIMPLE TEST v0.4 ===

INPUT

Method:
{HTTP_METHOD}

URL:
{BACKEND_URL}


EXECUTION FLOW

{실제 실행 순서}

예:

Controller#method

→ Service#method

→ 조건 확인

→ LocalMethod

→ BusinessService#method

→ Mapper#method

→ MyBatis XML#statementId

→ Return


DETAIL

1. {Class#Method}
   - 처리:
   - 실제 호출:
   - 조건/상태 변경:
   - Return/Throw:
   - Evidence:

2. {Class#Method}
   - 처리:
   - 실제 호출:
   - 조건/상태 변경:
   - Return/Throw:
   - Evidence:


MYBATIS

1.
- Mapper: {MAPPER_TYPE}#{MAPPER_METHOD}
- XML: {XML_PATH}
- Statement ID: {STATEMENT_ID}
- SQL: NOT ANALYZED


EXTERNAL

실제 호출이 있는 경우만 출력한다.

- 호출:
- 연결 흐름:
- Boundary:
- Evidence:


STATUS

COMPLETED

---

## STOP

실제 호출 경로 추적이 끝나면 즉시 종료한다.

추가 후보 Source를 찾지 않는다.

SQL을 읽지 않는다.

BE-REFERENCE를 읽지 않는다.

문서를 생성하지 않는다.

다른 Skill을 실행하지 않는다.