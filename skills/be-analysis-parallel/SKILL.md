---
name: be-analysis-parallel
description: 하나의 FE Action에 연결된 여러 Backend URL의 BE ID와 OUTPUT_PATH를 입력 순서대로 먼저 확정한 후 최대 2개의 Worker에서 Backend 분석을 병렬 실행하고 실제 생성 파일까지 검증한다.
argument-hint: "<화면명> <FE Base Name>"
allowed-tools: Read, Grep, Glob
---

# Backend Analysis Parallel

## 1. 목적

하나의 FE Action에서 호출되는 여러 Backend URL을
독립적으로 병렬 분석한다.

이 Skill은 Backend Business Logic을 직접 분석하지 않는다.

병렬 실행을 위한 Parent 역할만 수행한다.

실제 Backend 분석 규칙은 기존:

```text
.claude/skills/be-analysis/SKILL.md
```

를 기준으로 한다.

문서 표현 형식은 기존:

```text
.claude/references/BE-REFERENCE.md
```

를 기준으로 한다.

기존 두 파일은 수정하지 않는다.

---

# 2. 핵심 원칙

병렬 실행 전에 Parent가 반드시 먼저 확정한다.

```text
화면명
↓
FE Base Name
↓
Backend 목록
↓
기존 BE 파일
↓
각 Backend의 BE ID
↓
각 Backend의 OUTPUT_PATH
↓
Worker 시작
```

Worker가 다음을 결정하면 안 된다.

```text
BE ID
OUTPUT_PATH
파일명
FE Base Name
```

Parent가 먼저 전체 작업의 ID와 경로를 확정한 후
Worker를 시작한다.

---

# 3. 입력

기본 입력:

```text
/be-analysis-parallel <화면명> <FE Base Name>
```

예:

```text
/be-analysis-parallel generator-add FE-ACT-010-create
```

그 다음 Backend 목록을 입력받는다.

예:

```text
POST /generator/create
GET /generator/status
POST /generator/validate
```

또는 처음부터 다음과 같이 전달될 수 있다.

```text
화면명:
generator-add

FE Base Name:
FE-ACT-010-create

Backend:
POST /generator/create
GET /generator/status
POST /generator/validate
```

HTTP Method를 모르는 경우:

```text
UNKNOWN /generator/status
```

을 허용한다.

---

# 4. Backend 입력 형식

Backend 한 건의 형식:

```text
<HTTP Method|UNKNOWN> <Backend URL>
```

예:

```text
POST /generator/create
GET /generator/status
UNKNOWN /generator/check
```

각 줄을 독립 Backend 분석 작업으로 취급한다.

입력 순서를 유지한다.

---

# 5. 입력 검증

Parent는 Worker 실행 전에 다음을 확인한다.

```text
화면명 존재
FE Base Name 존재
Backend 목록 존재
각 Backend URL 존재
각 Backend HTTP Method 또는 UNKNOWN 존재
```

잘못된 항목이 있으면
해당 항목을 임의로 보정하지 않는다.

명확한 입력만 Worker 작업으로 생성한다.

---

# 6. FE Base Name

FE Base Name은 모든 Backend Worker의 Parent Key다.

예:

```text
FE-ACT-010-create
```

최종 Backend 문서:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
FE-ACT-010-create-BE-003.md
```

모든 Worker는 동일한 FE Base Name을 사용한다.

Worker가 FE Base Name을 변경하면 안 된다.

---

# 7. 기존 Backend 문서 확인

Worker를 시작하기 전에 Parent가 다음 위치를 확인한다.

```text
docs/analysis/{화면명}/backend/
```

현재 FE Base Name과 일치하는 파일만 확인한다.

예:

```text
FE Base Name:
FE-ACT-010-create
```

확인 대상:

```text
FE-ACT-010-create-BE-*.md
```

다른 FE Action 파일은
BE 번호 결정에 사용하지 않는다.

---

# 8. BE ID 선확정

Parent는 모든 Backend의 BE ID를
Worker 실행 전에 입력 순서대로 확정한다.

예:

기존 파일:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
```

입력:

```text
POST /backend/a
GET /backend/b
POST /backend/c
```

할당:

```text
Backend #1
  POST /backend/a
  BE-003

Backend #2
  GET /backend/b
  BE-004

Backend #3
  POST /backend/c
  BE-005
```

Worker 실행 후 번호를 결정하면 안 된다.

---

# 9. BE ID 규칙

BE ID:

```text
BE-001
BE-002
BE-003
...
```

3자리 번호를 사용한다.

기존 최대 번호 다음부터 시작한다.

예:

```text
기존:
BE-001
BE-003

신규 첫 번호:
BE-004
```

빈 번호를 재사용하지 않는다.

병렬 Worker끼리 같은 BE ID를 사용할 수 없다.

---

# 10. OUTPUT_PATH 선확정

BE ID를 확정한 즉시
각 Backend의 OUTPUT_PATH를 확정한다.

형식:

```text
docs/analysis/{화면명}/backend/{FE Base Name}-{BE ID}.md
```

예:

```text
Backend #1

Method:
POST

URL:
/backend/a

BE ID:
BE-003

OUTPUT_PATH:
docs/analysis/generator-add/backend/FE-ACT-010-create-BE-003.md
```

모든 OUTPUT_PATH를 확정한 후에만
Worker 실행을 시작한다.

---

# 11. 작업 계획

Worker 실행 직전 Parent는 내부적으로
전체 작업 계획을 확정한다.

예:

```text
TASK #1
Method       POST
URL          /backend/a
BE ID        BE-003
OUTPUT_PATH  .../FE-ACT-010-create-BE-003.md

TASK #2
Method       GET
URL          /backend/b
BE ID        BE-004
OUTPUT_PATH  .../FE-ACT-010-create-BE-004.md

TASK #3
Method       POST
URL          /backend/c
BE ID        BE-005
OUTPUT_PATH  .../FE-ACT-010-create-BE-005.md
```

이 계획은 Worker 실행 중 변경하지 않는다.

---

# 12. Worker 수

동시에 실행하는 Worker는 최대:

```text
2
```

개다.

Backend가 1개이면:

```text
Worker 1개
```

Backend가 2개이면:

```text
Worker 2개 병렬
```

Backend가 3개 이상이면:

```text
Batch #1
  Worker #1
  Worker #2

완료 후

Batch #2
  Worker #3
  Worker #4

완료 후

Batch #3
  ...
```

형태로 처리한다.

---

# 13. Batch 규칙

예를 들어 Backend가 5개이면:

```text
Batch #1
├─ TASK #1
└─ TASK #2

Batch #2
├─ TASK #3
└─ TASK #4

Batch #3
└─ TASK #5
```

현재 Batch의 Worker가 완료된 후
다음 Batch를 시작한다.

최대 동시 Worker 수 2를 넘지 않는다.

---

# 14. Worker 역할

각 Worker는 Backend 하나만 분석한다.

Worker 입력:

```text
화면명
FE Base Name
BE ID
HTTP Method
Backend URL
OUTPUT_PATH
```

Worker는 다른 Backend를 분석하지 않는다.

---

# 15. Worker 분석 규칙

Worker의 Backend 분석 규칙은 반드시:

```text
.claude/skills/be-analysis/SKILL.md
```

를 기준으로 한다.

문서 형식은:

```text
.claude/references/BE-REFERENCE.md
```

를 기준으로 한다.

병렬 Skill에서 별도의 Backend 분석 규칙을 새로 만들지 않는다.

단일 `be-analysis`와 병렬 분석의 결과 품질이
달라지지 않아야 한다.

---

# 16. Worker의 BE ID 결정 금지

일반 `be-analysis` Skill에는
BE ID를 자동 결정하는 규칙이 존재할 수 있다.

병렬 실행에서는 Parent가 전달한:

```text
BE ID
OUTPUT_PATH
```

가 우선한다.

Worker는 기존 Backend 파일을 다시 확인하여
새 번호를 선택하지 않는다.

Worker는 Parent가 전달한 BE ID를 그대로 사용한다.

---

# 17. Worker의 OUTPUT_PATH 변경 금지

Worker는 Parent가 전달한 OUTPUT_PATH에만
최종 문서를 생성한다.

금지:

```text
새 BE ID 생성

다른 파일명 생성

FE Base Name 변경

다른 backend 폴더 선택

기존 번호 재계산
```

---

# 18. Worker 분석 범위

각 Worker의 기본 실행 흐름:

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
외부 연동
↓
Caller 복귀
↓
Response
↓
문서 생성
```

세부 규칙은
`be-analysis/SKILL.md`를 따른다.

---

# 19. MyBatis

병렬 Worker도 단일 BE 분석과 동일하다.

```text
Mapper Type
↓
Mapper Method
↓
MyBatis XML 위치
↓
Statement ID
↓
STOP
```

분석하지 않는다.

```text
Statement Body
SQL
Table
Column
JOIN
WHERE
Dynamic SQL
SQL Parameter Mapping
SQL Result Mapping
Oracle Metadata
```

---

# 20. Reference

각 Worker의 최종 문서는:

```text
.claude/references/BE-REFERENCE.md
```

형식을 따른다.

Reference Sample은 Evidence가 아니다.

실제 Source만 Evidence로 사용한다.

---

# 21. Worker 독립성

Worker끼리 다음 정보를 공유하여
분석 결과를 추측하지 않는다.

```text
Controller 추측
Service 추측
Mapper 추측
XML 추측
Statement ID 추측
External 추측
Response 추측
```

각 Worker는 자신의 Backend URL을
실제 Source 기준으로 독립 분석한다.

단:

```text
화면명
FE Base Name
BE ID
OUTPUT_PATH
```

는 Parent가 확정한 값을 사용한다.

---

# 22. Worker 간 파일 충돌 방지

Parent가 OUTPUT_PATH를 선확정하므로
Worker끼리 같은 파일을 작성하면 안 된다.

예:

```text
Worker #1
→ FE-ACT-010-create-BE-003.md

Worker #2
→ FE-ACT-010-create-BE-004.md
```

동일 OUTPUT_PATH가 발견되면
Worker를 시작하지 않는다.

---

# 23. Worker 완료 조건

Worker 하나의 완료 조건:

```text
Controller 확인
↓
실제 Call Path 분석
↓
Response까지 연결
↓
BE-REFERENCE 적용
↓
Parent가 지정한 OUTPUT_PATH에 파일 생성
```

파일 생성까지 완료되어야
Worker 완료로 판단한다.

---

# 24. Parent 결과 검증

각 Batch 완료 후 Parent는
해당 Worker의 OUTPUT_PATH가 실제 생성되었는지 확인한다.

검증:

```text
파일 존재 여부
파일명 일치
BE ID 일치
FE Base Name 일치
```

분석 내용을 Parent가 다시 전체 분석하지 않는다.

---

# 25. 문서 최소 검증

생성된 문서에서 최소한 다음 항목의 존재 여부를 확인한다.

```text
기능 정보
한눈에 보는 실행 흐름
전체 실행 Tree
Source Evidence
분석 경계
```

현재 Backend 기능에 DB 또는 외부 연동이 없으면
해당 Block이 없는 것은 실패가 아니다.

---

# 26. 실패 처리

Worker가 실패하면
다른 성공 Worker의 결과를 삭제하지 않는다.

예:

```text
TASK #1
SUCCESS

TASK #2
FAILED

TASK #3
SUCCESS
```

결과:

```text
BE-003.md 유지
BE-005.md 유지

TASK #2 실패 보고
```

실패한 Task 때문에
전체 작업을 처음부터 다시 실행하지 않는다.

---

# 27. 실패 Worker 자동 대체 금지

실패한 Backend를
다른 URL이나 Method로 임의 변경하지 않는다.

실패 원인을 기록한다.

예:

```text
Controller Mapping 확인 실패

동일 URL 다중 Method로 확정 불가

Source 확인 실패

문서 생성 실패
```

---

# 28. 성공 파일 재작성 금지

이미 성공한 Worker의 문서를
다음 Batch에서 다시 생성하지 않는다.

성공한 Task는 완료 상태로 유지한다.

---

# 29. Parent의 Source 분석 금지

Parent는 Backend Business Logic을
직접 분석하지 않는다.

Parent 역할:

```text
입력 관리

BE ID 결정

OUTPUT_PATH 결정

Batch 구성

Worker 실행

완료 확인

파일 검증

최종 결과 보고
```

실제 Source 분석은 Worker가 담당한다.

---

# 30. 기존 단일 Skill 보호

다음 파일은 병렬 실행을 위해 수정하지 않는다.

```text
.claude/skills/be-analysis/SKILL.md
.claude/references/BE-REFERENCE.md
```

현재 단일 분석이 정상 동작하는 상태를 유지한다.

---

# 31. 최종 결과

모든 Worker가 끝나면 Parent는
간단한 실행 결과를 출력한다.

예:

```text
Backend 병렬 분석 완료

화면명
  generator-add

FE Base Name
  FE-ACT-010-create

총 Backend
  3

성공
  3

실패
  0

생성 파일

BE-003
  docs/analysis/generator-add/backend/
  FE-ACT-010-create-BE-003.md

BE-004
  docs/analysis/generator-add/backend/
  FE-ACT-010-create-BE-004.md

BE-005
  docs/analysis/generator-add/backend/
  FE-ACT-010-create-BE-005.md
```

Backend 분석 내용을 Parent 화면에 다시 출력하지 않는다.

상세 분석 내용은 생성된 Markdown 문서를 기준으로 한다.

---

# 32. 최종 검증

종료 전에 확인한다.

```text
화면명 확인
↓
FE Base Name 확인
↓
Backend 입력 순서 확인
↓
기존 BE 번호 확인
↓
모든 BE ID 선확정
↓
모든 OUTPUT_PATH 선확정
↓
OUTPUT_PATH 중복 없음
↓
최대 Worker 2개 준수
↓
모든 Batch 완료
↓
각 OUTPUT_PATH 파일 존재 확인
↓
성공 / 실패 결과 정리
```

---

# 33. STOP

입력된 Backend 목록의
모든 Task가:

```text
SUCCESS
```

또는:

```text
FAILED
```

상태로 확정되면 STOP 한다.

자동으로 다른 Backend URL을 찾지 않는다.

자동으로 다른 FE Action을 분석하지 않는다.

자동으로 다음 화면을 분석하지 않는다.

실패한 Backend를 임의로 변경해서
재분석하지 않는다.

현재 입력된 Backend 목록만 처리하고 종료한다.