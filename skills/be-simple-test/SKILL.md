---
name: be-simple-test
description: Backend URL에서 시작하여 실제 호출되는 Backend 실행 경로만 끝까지 추적하는 최소 규칙 성능 테스트.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Simple Test v0.1

## 목적

Backend URL에서 시작하여
실제 Source의 호출 관계만 따라가며
Backend 실행 흐름을 끝까지 확인한다.

복잡한 단계별 탐색 절차를 사용하지 않는다.

입력:

HTTP Method + Backend URL

출력:

실제 Backend 실행 흐름

---

## 분석 범위

다음 흐름을 실제 호출이 존재하는 만큼 추적한다.

Controller
→ Service
→ ServiceImpl
→ Local/private Method
→ 다른 Business Service
→ Mapper
→ MyBatis XML
→ SQL
→ SAP/RFC/외부 연동
→ Return / Exception

모든 단계가 반드시 존재할 필요는 없다.

실제 Source에 존재하는 흐름만 분석한다.

---

## 핵심 규칙

1. Backend URL과 HTTP Method로 Controller를 찾는다.

2. Controller에서 실제 호출되는 Service부터 실행 흐름을 따라간다.

3. 현재 Method에서 실제 호출되는 코드만 추적한다.

4. 같은 클래스의 Local/private Method가 실제 호출되면 따라간다.

5. 다른 Business Service가 실제 호출되면 해당 Method를 따라간다.

6. Mapper 호출은 Mapper.java 전체를 분석하지 말고 가능한 경우 MyBatis XML의 실제 Statement로 바로 이동한다.

7. MyBatis XML에서는 실제 호출된 Statement와 SQL만 확인한다.

8. SAP/RFC/외부 연동은 실제 호출이 확인된 경우에만 추적한다.

9. 이미 분석한 Class + Method는 다시 읽지 않는다.

10. 실제 호출 관계가 확인되지 않은 Source는 탐색하지 않는다.

---

## 탐색 원칙

검색 범위는 좁게 유지하고
호출 깊이는 실제 실행 경로 끝까지 따라간다.

정확한 Symbol을 우선 사용한다.

필요한 위치를 찾으면
Method 또는 Statement 주변만 읽는다.

전체 파일을 기본적으로 읽지 않는다.

이미 확보한:

- 파일
- Class
- Method
- Mapper
- XML

정보는 다시 검색하지 않는다.

---

## Source 범위

Java:

gipms-api-*/src/main/java/**

Resources:

gipms-api-*/src/main/resources/**

Controller가 발견된 프로젝트를 기준으로
관련 Source를 우선 추적한다.

---

## 제외

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

Dependency/JAR 내부로 이동하지 않는다.

Oracle Metadata를 조회하지 않는다.

Code Index를 사용하지 않는다.

Agent를 사용하지 않는다.

병렬 Worker를 사용하지 않는다.

---

## MyBatis

Service에서 실제 Mapper 호출이 확인되면:

Mapper Type
+
Mapper Method

를 기준으로 MyBatis XML을 찾는다.

실제 Statement만 읽는다.

다음을 확인한다.

- SELECT / INSERT / UPDATE / DELETE
- Main Table
- JOIN
- WHERE
- Parameter
- Dynamic SQL
- include

관련 없는 Statement는 분석하지 않는다.

---

## Local/private Method

현재 실행 경로에서
실제로 호출되는 Local/private Method만 따라간다.

관련 있어 보인다는 이유로
다른 Local Method를 탐색하지 않는다.

---

## 다른 Service

현재 Method에서
실제로 호출되는 다른 Business Service만 따라간다.

Service 목록을 미리 수집하지 않는다.

실제 호출된 Method만 분석한다.

Service 간 호출이 계속되면
실제 호출 관계를 계속 따라간다.

예:

Service A
→ Service B
→ Service C
→ Mapper

또는:

Service A
→ Service B
→ Service A의 다른 Method

실제 Source에 존재하면 그대로 추적한다.

---

## SAP / RFC / 외부 연동

실제 실행 경로에서 발견된 경우에만 확인한다.

호출이 없으면 찾지 않는다.

Dependency/JAR 내부 구현은 분석하지 않는다.

프로젝트 Source에서 확인 가능한 범위까지만 추적한다.

---

## Evidence

분석하면서 확인한 Source 위치를 재사용한다.

Evidence를 만들기 위해
Source를 다시 검색하지 않는다.

형식:

프로젝트 루트 기준 상대경로:라인범위

절대경로를 출력하지 않는다.

---

## 출력

파일을 생성하지 않는다.

화면에만 출력한다.

형식:

=== BE SIMPLE TEST ===

INPUT

Method:
{HTTP_METHOD}

URL:
{BACKEND_URL}


EXECUTION FLOW

Controller
{CLASS}#{METHOD}

↓

Service
{CLASS}#{METHOD}

↓

{실제 호출 순서}

↓

Mapper
{MAPPER}#{METHOD}

↓

SQL
{SQL_TYPE} {TABLE}

↓

Return
{RETURN}


DETAIL

1. {실행 단계}
   - 처리:
   - 호출:
   - 조건:
   - Evidence:

2. {실행 단계}
   - 처리:
   - 호출:
   - 조건:
   - Evidence:

...


MAPPER / SQL

- Mapper:
- Statement:
- SQL Type:
- Main Table:
- JOIN:
- WHERE:
- Parameter:
- Dynamic SQL:
- Include:
- Evidence:


EXTERNAL

실제 호출이 있는 경우만 출력한다.

- Type:
- Call:
- 처리:
- Evidence:


EXCEPTION

실제 확인된 예외 흐름만 출력한다.


STATUS

COMPLETED

---

## STOP

실제 실행 흐름 추적이 끝나면 즉시 종료한다.

추가 후보 Source를 찾지 않는다.

문서를 생성하지 않는다.

BE-REFERENCE를 읽지 않는다.

다른 Skill을 실행하지 않는다.