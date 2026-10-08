
---
name: be-analysis-parallel
description: 하나의 FE Action에 연결된 여러 Backend URL의 BE ID와 OUTPUT_PATH를 먼저 확정하고, be-analysis-worker Subagent를 최대 2개씩 실제 병렬 실행하여 Backend 분석 문서를 생성한다.
argument-hint: "<화면명> <FE Base Name>"
allowed-tools: Read, Glob, Agent
---

# Backend Analysis Parallel

## 1. 목적

하나의 FE Action에 연결된 여러 Backend URL을
독립 Subagent에서 병렬 분석한다.

이 Skill은 Backend Source를 직접 분석하지 않는다.

Main의 역할:

```text
입력 수집
↓
기존 BE 파일 확인
↓
모든 BE ID 선확정
↓
모든 OUTPUT_PATH 선확정
↓
Worker Batch 구성
↓
Subagent 최대 2개 병렬 실행
↓
Batch 완료 대기
↓
다음 Batch 실행
↓
생성 파일 검증
↓
결과 출력
```

실제 Backend Source 분석은 반드시:

```text
be-analysis-worker
```

Subagent가 담당한다.

---

# 2. 기존 분석 엔진 보호

다음 파일은 병렬 실행 과정에서 수정하지 않는다.

```text
.claude/skills/be-analysis/SKILL.md
.claude/references/BE-REFERENCE.md
```

현재 정상 동작하는 단일 Backend 분석 규칙을
병렬 처리 때문에 변경하지 않는다.

Markdown 제목 검증도
기존 분석 엔진의 규칙을 사용한다.

---

# 3. 입력

호출:

```text
/be-analysis-parallel <화면명> <FE Base Name>
```

예:

```text
/be-analysis-parallel generator-add FE-ACT-010-create

POST /generator/create
GET /generator/status
POST /generator/validate
GET /generator/result
```

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

입력 순서를 유지한다.

---

# 4. Main 역할

Main은 다음만 수행한다.

```text
화면명 확인

FE Base Name 확인

Backend 목록 파싱

기존 BE 파일 확인

BE ID 결정

OUTPUT_PATH 결정

Batch 구성

be-analysis-worker 실행

Worker 완료 대기

생성 파일 존재 확인

Worker Markdown 제목 검증 결과 확인

최종 결과 출력
```

Main은 Backend Source 분석을 직접 하지 않는다.

Main에서 다음을 수행하지 않는다.

```text
Controller 검색
Service 검색
ServiceImpl 분석
Local Method 분석
Mapper 검색
MyBatis XML 분석
Statement ID 검색
RFC 분석
REST 분석
Business Logic 분석
Response 분석
```

위 작업은 Worker 전용이다.

---

# 5. 기존 Backend 파일 확인

다음 위치만 확인한다.

```text
docs/analysis/{화면명}/backend/
```

현재 FE Base Name과 일치하는 파일:

```text
{FE Base Name}-BE-*.md
```

만 확인한다.

예:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
```

다른 FE Base Name의 파일은 무시한다.

---

# 6. BE ID 선확정

모든 Worker를 시작하기 전에
전체 Backend의 BE ID를 먼저 확정한다.

예:

기존:

```text
FE-ACT-010-create-BE-001.md
FE-ACT-010-create-BE-002.md
```

신규 입력:

```text
POST /backend/a
GET /backend/b
POST /backend/c
GET /backend/d
```

선확정:

```text
/backend/a
→ BE-003

/backend/b
→ BE-004

/backend/c
→ BE-005

/backend/d
→ BE-006
```

Worker가 BE ID를 결정하면 안 된다.

---

# 7. BE 번호 규칙

형식:

```text
BE-001
BE-002
BE-003
...
```

기존 최대 번호 다음부터 시작한다.

중간 번호가 비어 있어도 재사용하지 않는다.

예:

```text
BE-001
BE-003
```

이면 다음 번호는:

```text
BE-004
```

이다.

---

# 8. OUTPUT_PATH 선확정

모든 Backend에 대해 Worker 실행 전에
OUTPUT_PATH를 확정한다.

형식:

```text
docs/analysis/{화면명}/backend/{FE Base Name}-{BE ID}.md
```

예:

```text
TASK #1

HTTP Method
POST

Backend URL
/backend/a

BE ID
BE-003

OUTPUT_PATH
docs/analysis/generator-add/backend/FE-ACT-010-create-BE-003.md
```

Worker가 OUTPUT_PATH를 변경하면 안 된다.

---

# 9. 전체 Task 생성

입력 Backend를 순서대로 Task로 만든다.

예:

```text
TASK #1
POST /backend/a
BE-003
.../FE-ACT-010-create-BE-003.md

TASK #2
GET /backend/b
BE-004
.../FE-ACT-010-create-BE-004.md

TASK #3
POST /backend/c
BE-005
.../FE-ACT-010-create-BE-005.md

TASK #4
GET /backend/d
BE-006
.../FE-ACT-010-create-BE-006.md
```

Task 정보는 Worker 실행 후 변경하지 않는다.

---

# 10. 병렬 실행 원칙

동시에 실행할 Worker 수:

```text
MAX_WORKERS = 2
```

Backend가 2개 이상이면
반드시 독립적인 `be-analysis-worker` Subagent 2개를 사용한다.

Main이 두 Backend를 직접 순차 분석하면 안 된다.

하나의 Worker에게 Backend 2개를 전달하면 안 된다.

각 Worker는 Backend 하나만 담당한다.

---

# 11. 실제 Subagent 실행

각 Task는 반드시:

```text
be-analysis-worker
```

Agent로 실행한다.

Batch에 Task가 2개 있으면
두 Agent 호출을 같은 Batch에서 시작한다.

개념:

```text
main
│
├─ be-analysis-worker
│    └─ TASK #1
│
└─ be-analysis-worker
     └─ TASK #2
```

두 Worker는 서로 독립적이다.

---

# 12. 동시 실행 강제

Batch에 Task가 2개 있으면:

```text
Worker #1 시작
Worker #2 시작
```

을 독립 Subagent 작업으로 시작한 뒤
두 결과를 기다린다.

금지:

```text
Worker #1 시작
↓
Worker #1 완료 대기
↓
Worker #2 시작
```

위 방식은 병렬이 아니므로 사용하지 않는다.

원하는 방식:

```text
Worker #1 ───────────────┐
                         ├─ 둘 다 완료
Worker #2 ───────────────┘
```

---

# 13. Batch 처리

Backend가 6개이면:

```text
Batch #1
├─ TASK #1
└─ TASK #2

        ↓ 둘 다 완료

Batch #2
├─ TASK #3
└─ TASK #4

        ↓ 둘 다 완료

Batch #3
├─ TASK #5
└─ TASK #6
```

현재 Batch가 완료되기 전에
다음 Batch를 시작하지 않는다.

---

# 14. 홀수 Task

Backend가 5개이면:

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

마지막 Batch는 Worker 하나만 실행한다.

---

# 15. Worker 입력

각 `be-analysis-worker`에 다음 값을 전달한다.

```text
화면명:
{화면명}

FE Base Name:
{FE Base Name}

BE ID:
{BE ID}

HTTP Method:
{HTTP Method}

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

Worker에게 추가로 다음 완료 조건을 전달한다.

```text
지정 OUTPUT_PATH에 문서를 생성한 후
파일을 다시 Read하여 Markdown 제목을 검증한다.

H1 / H2 / H3 제목 검증이 통과한 경우에만
HEADING_CHECK: PASS를 반환한다.

제목 검증이 실패하면
STATUS: FAILED를 반환한다.
```

---

# 16. Worker Agent 지정

Task 실행 시 일반 Agent에게 맡기지 않는다.

반드시:

```text
be-analysis-worker
```

를 사용한다.

Worker에게 현재 Task 하나의 입력만 전달한다.

---

# 17. Worker 분석 규칙

Worker는 기존:

```text
.claude/skills/be-analysis/SKILL.md
```

의 Backend 분석 규칙을 따른다.

단 다음 항목은 Parent 값이 우선한다.

```text
FE Base Name
BE ID
OUTPUT_PATH
```

Worker가 위 값을 재계산하지 않는다.

Markdown 제목 검증은
해당 Skill의 저장 후 검증 규칙을 따른다.

---

# 18. Reference

Worker 문서 형식:

```text
.claude/references/BE-REFERENCE.md
```

Reference는 Worker가 분석 시작 시 한 번만 읽는다.

Main은 Reference를 읽을 필요가 없다.

---

# 19. MyBatis 정책

Worker는 기존 단일 BE 정책을 그대로 따른다.

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

분석 금지:

```text
Statement Body
SQL
Table
Column
JOIN
WHERE
Dynamic SQL
include 내부
resultMap
Oracle Metadata
```

---

# 20. Worker 완료 조건

Worker 완료:

```text
Backend URL 분석
↓
Response까지 추적
↓
BE-REFERENCE 적용
↓
Markdown H1 / H2 / H3 적용
↓
지정 OUTPUT_PATH에 문서 생성
↓
OUTPUT_PATH 다시 Read
↓
Markdown 제목 검증
↓
누락 시 제목만 보완
↓
최종 제목 검증 통과
↓
SUCCESS 및 HEADING_CHECK: PASS 반환
```

제목 검증이 통과하지 않은 Worker는
SUCCESS를 반환할 수 없다.

---

# 21. Batch 완료 조건

Batch의 모든 Worker가:

```text
SUCCESS
```

또는:

```text
FAILED
```

상태가 될 때까지 기다린다.

그 후 다음 Batch를 시작한다.

한 Worker가 먼저 끝났다고
그 자리에 다음 Task를 바로 넣지 않는다.

반드시 2개 단위 Batch 경계를 유지한다.

---

# 22. 실패 처리

예:

```text
Batch #1

TASK #1
SUCCESS

TASK #2
FAILED
```

두 Task가 모두 종료 상태가 되었으므로
Batch #2를 시작할 수 있다.

실패 Task를 자동 재실행하지 않는다.

실패 Task 때문에 성공 파일을 삭제하지 않는다.

Markdown 제목 검증 실패도
해당 Task의 FAILED 사유로 처리한다.

---

# 23. 생성 파일 검증

Worker 완료 후 Main은
지정 OUTPUT_PATH의 파일 존재 여부를 확인한다.

확인:

```text
파일 존재
파일명
FE Base Name
BE ID
Worker STATUS
Worker HEADING_CHECK
```

Main은 다음 조건을 모두 만족한 경우에만
해당 Task를 최종 SUCCESS로 인정한다.

```text
Worker STATUS: SUCCESS
↓
Worker HEADING_CHECK: PASS
↓
지정 OUTPUT_PATH 파일 존재
↓
파일명 일치
↓
FE Base Name 일치
↓
BE ID 일치
↓
최종 SUCCESS
```

Worker가 SUCCESS를 반환하더라도
HEADING_CHECK가 누락되거나 PASS가 아니면
최종 SUCCESS로 인정하지 않는다.

이 경우 해당 Task는
검증 실패로 처리하고 사유를 기록한다.

Main이 Backend Source를 다시 분석해서
Worker 결과를 검증하지 않는다.

Main이 Reference를 다시 읽지 않는다.

Main이 제목 검증을 위해
Backend 문서를 재작성하지 않는다.

---

# 24. 성공 파일 보호

이미 SUCCESS인 파일은
다음 Batch에서 다시 분석하지 않는다.

다음 Worker가 기존 성공 파일을 수정하면 안 된다.

Markdown 제목 검증을 이유로
다른 Task의 파일을 변경하지 않는다.

---

# 25. 화면 표시 성공 조건

병렬 Batch가 실행되는 동안
실제 실행 구조는 논리적으로 다음과 같아야 한다.

```text
main
│
├─ be-analysis-worker
│    └─ Backend #1
│
└─ be-analysis-worker
     └─ Backend #2
```

Main이 직접 두 Backend를 분석하고 있다면
이 Skill의 의도대로 실행된 것이 아니다.

---

# 26. Main Context 보호

Worker는 분석 상세 내용을
Main에게 전부 반환하지 않는다.

Worker 반환은 최소화한다.

성공:

```text
STATUS: SUCCESS
BE ID: BE-003
OUTPUT_PATH: ...
HEADING_CHECK: PASS
```

실패:

```text
STATUS: FAILED
BE ID: BE-003
OUTPUT_PATH: ...
REASON: ...
```

Backend 상세 내용은 Markdown 파일에 저장한다.

Main은 Worker의 검증 결과만 사용한다.

---

# 27. 최종 결과

모든 Batch가 끝나면 Main은 요약만 출력한다.

예:

```text
Backend 병렬 분석 완료

화면명
generator-add

FE Base Name
FE-ACT-010-create

총 Backend
4

성공
4

실패
0

생성 파일

BE-003
docs/analysis/generator-add/backend/FE-ACT-010-create-BE-003.md

BE-004
docs/analysis/generator-add/backend/FE-ACT-010-create-BE-004.md

BE-005
docs/analysis/generator-add/backend/FE-ACT-010-create-BE-005.md

BE-006
docs/analysis/generator-add/backend/FE-ACT-010-create-BE-006.md
```

성공 건수에는
Markdown 제목 검증까지 통과한 Task만 포함한다.

검증에 실패한 Task는
실패 건수에 포함하고 사유를 표시한다.

---

# 28. 최종 검증

종료 전 확인:

```text
화면명 확인
↓
FE Base Name 확인
↓
Backend 목록 확인
↓
BE ID 전체 선확정
↓
OUTPUT_PATH 전체 선확정
↓
중복 OUTPUT_PATH 없음
↓
Batch 최대 2개 Worker
↓
실제 be-analysis-worker 사용
↓
Batch 단위 완료 대기
↓
모든 Task 상태 확인
↓
Worker HEADING_CHECK 확인
↓
SUCCESS Task의 HEADING_CHECK: PASS 확인
↓
생성 파일 존재 확인
↓
최종 성공 / 실패 건수 확인
↓
최종 결과 출력
```

Main은 분석 내용 자체를 검증하지 않는다.

Worker가 저장 후 검증을 담당하고
Main은 Worker의 검증 상태를 확인한다.

---

# 29. STOP

입력된 Backend 목록만 처리한다.

자동으로 다른 Backend를 찾지 않는다.

자동으로 다른 FE Action을 분석하지 않는다.

자동으로 다음 화면을 분석하지 않는다.

모든 Task가:

```text
SUCCESS
```

또는:

```text
FAILED
```

가 되고, 각 SUCCESS Task의 Markdown 제목 검증 결과까지 확인하면 STOP 한다.
