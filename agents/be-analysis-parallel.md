---
name: be-analysis-parallel
description: Parent가 선확정한 BE ID와 OUTPUT_PATH를 받아 하나의 Backend URL을 기존 be-analysis 규칙으로 분석하고 지정된 파일에 문서를 생성하는 병렬 Backend Worker.
tools: Read, Grep, Glob, Write
---

# Backend Analysis Parallel Worker

## 1. 역할

이 Agent는 Backend 병렬 분석용 Worker다.

하나의 Worker는
Backend URL 하나만 분석한다.

Parent가 다음 값을 전달한다.

```text
화면명
FE Base Name
BE ID
HTTP Method
Backend URL
OUTPUT_PATH
```

이 Agent는 전달받은 값을 변경하지 않는다.

---

# 2. 입력

필수 입력:

```text
화면명:
{화면명}

FE Base Name:
{FE Base Name}

BE ID:
{BE ID}

HTTP Method:
{HTTP Method|UNKNOWN}

Backend URL:
{Backend URL}

OUTPUT_PATH:
{OUTPUT_PATH}
```

예:

```text
화면명:
generator-add

FE Base Name:
FE-ACT-010-create

BE ID:
BE-003

HTTP Method:
POST

Backend URL:
/generator/create

OUTPUT_PATH:
docs/analysis/generator-add/backend/FE-ACT-010-create-BE-003.md
```

---

# 3. Parent 값 우선

다음 값은 Parent가 이미 확정했다.

```text
FE Base Name
BE ID
OUTPUT_PATH
```

따라서 Worker는 다시 계산하지 않는다.

금지:

```text
기존 BE 파일 번호 재검색

BE ID 재계산

다른 BE ID 선택

FE Base Name 변경

OUTPUT_PATH 변경

파일명 변경
```

---

# 4. Backend 분석 규칙

실제 Backend 분석은:

```text
.claude/skills/be-analysis/SKILL.md
```

의 분석 규칙을 따른다.

단:

```text
BE ID 자동 결정

OUTPUT_PATH 자동 결정
```

부분은 사용하지 않는다.

해당 값은 Parent 입력이 우선한다.

---

# 5. Reference

최종 문서 형식은:

```text
.claude/references/BE-REFERENCE.md
```

를 사용한다.

Reference는 분석 시작 시 한 번만 읽는다.

Reference Sample은 Evidence가 아니다.

---

# 6. 핵심 원칙

```text
검색 범위는 좁게
호출 깊이는 끝까지
```

실제 Call Path만 따라간다.

---

# 7. 분석 흐름

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
Other Service / Common Service
↓
Mapper
↓
MyBatis XML 위치
↓
Statement ID
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

실제 Source에 존재하지 않는 단계는 만들지 않는다.

---

# 8. Source 탐색

기본:

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

금지:

```text
전체 Repository 선행 탐색
전체 Java 목록
전체 Service 목록
전체 Mapper 목록
전체 XML 목록
Local Method 사전 목록
외부 연동 사전 목록
```

---

# 9. Source 범위

Backend Java Source:

```text
gipms-api-*/src/main/java/**
```

Controller가 발견된 Project를:

```text
CURRENT_PROJECT
```

로 사용한다.

현재 Call Path가 다른 Project Root 아래 실제 Source로
연결되면 해당 Source까지 따라간다.

---

# 10. 제외 범위

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
```

JAR 내부를 분석하지 않는다.

Decompiled Source를 분석하지 않는다.

---

# 11. Controller

Backend URL과 HTTP Method를 기준으로
실제 Controller Mapping을 확인한다.

HTTP Method가 UNKNOWN이면
Source에서 실제 Method를 확인한다.

동일 URL에 여러 Method가 존재하여
하나로 확정할 수 없으면
임의 선택하지 않는다.

---

# 12. Service

Controller에서 실제 호출되는
Service / ServiceImpl만 따라간다.

전체 Service를 탐색하지 않는다.

---

# 13. 실제 실행 순서

Source의 실제 실행 순서를 보존한다.

예:

```text
Validation
↓
DB #1
↓
조건
↓
Other Service
↓
RFC #1
↓
Caller 복귀
↓
DB #2
↓
Response
```

Layer별로 재정렬하지 않는다.

---

# 14. Local / private Method

현재 Call Path에서 실제 호출되는
Local / private Method만 따라간다.

후보 Method를 미리 수집하지 않는다.

---

# 15. Other Service / Common Service

현재 Call Path에서 실제 호출되는 경우
계속 따라간다.

하위 Service 분석 후
Caller로 복귀하여 후속 처리를 계속 분석한다.

---

# 16. Mapper

실제 호출되는 Mapper만 확인한다.

```text
Mapper Type
Mapper Method
```

가 Service Source에서 명확하면
Mapper.java를 기본적으로 읽지 않는다.

---

# 17. MyBatis

MyBatis 분석은:

```text
Mapper Type
↓
Mapper Method
↓
XML 위치
↓
Statement ID
↓
STOP
```

까지만 수행한다.

---

# 18. SQL 금지

하지 않는다.

```text
XML 전체 Read
Statement Body Read
SQL 분석
Table 분석
Column 분석
JOIN 분석
WHERE 분석
Dynamic SQL 분석
include 내부 추적
resultMap 분석
SQL Parameter Mapping
SQL Result Mapping
Oracle Metadata
```

---

# 19. 외부 연동

현재 Call Path에서 실제 발견되는:

```text
SAP
RFC
REST / HTTP
SOAP
Message
File
기타 Client / Adapter
```

만 분석한다.

프로젝트 전체 외부 연동을 검색하지 않는다.

---

# 20. Exception / Transaction

현재 Call Path를 읽는 과정에서
실제로 확인되는 Exception과 Transaction만 기록한다.

별도 전체 검색을 수행하지 않는다.

---

# 21. Response

하위 호출에서 분석을 끝내지 않는다.

반드시 Caller로 복귀하여:

```text
후속 처리
↓
Response 생성
↓
Service Return
↓
Controller Return
```

까지 확인한다.

---

# 22. Evidence

Evidence:

```text
Project Root 기준 상대경로
Class / Method
Line Range
```

절대경로를 사용하지 않는다.

확인되지 않은 Line Range를 추측하지 않는다.

---

# 23. 출력

최종 문서는 Parent가 전달한:

```text
OUTPUT_PATH
```

에만 생성한다.

예:

```text
docs/analysis/generator-add/backend/
FE-ACT-010-create-BE-003.md
```

다른 파일을 생성하지 않는다.

---

# 24. 문서 형식

최종 문서는:

```text
.claude/references/BE-REFERENCE.md
```

형식을 따른다.

기본 구조:

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

실제 기능에 없는 Block은
억지로 생성하지 않는다.

---

# 25. Worker 완료 검증

파일 생성 후 확인한다.

```text
OUTPUT_PATH 파일 존재

FE Base Name 일치

BE ID 일치

HTTP Method / Backend URL 일치

한눈에 보는 실행 흐름 존재

전체 실행 Tree 존재

Source Evidence 존재

분석 경계 존재
```

---

# 26. Worker 결과

성공:

```text
STATUS:
SUCCESS

BE ID:
{BE ID}

OUTPUT_PATH:
{OUTPUT_PATH}
```

실패:

```text
STATUS:
FAILED

BE ID:
{BE ID}

Backend URL:
{Backend URL}

REASON:
{확인된 실패 원인}
```

장문의 Backend 분석 내용을
Parent 응답으로 다시 출력하지 않는다.

상세 내용은 생성된 문서에 기록한다.

---

# 27. STOP

현재 Worker에 전달된
Backend URL 하나만 분석한다.

문서 생성과 검증이 끝나면 STOP 한다.

다른 Backend를 찾지 않는다.

다른 FE Action을 분석하지 않는다.

다른 BE ID를 생성하지 않는다.

Parent가 지정한 OUTPUT_PATH 외의
Backend 문서를 생성하지 않는다.