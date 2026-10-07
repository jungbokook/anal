코드 기능 분석 프로젝트 가이드

1. 목적

Frontend 화면을 시작점으로 기능을 단계적으로 분석하고
각 분석 결과를 독립된 문서로 생성한다.

전체 시스템을 한 번에 분석하지 않는다.

현재 사용자가 선택한 대상만 분석한다.

⸻

2. 분석 구조

분석은 4개의 독립 단계로 수행한다.

1단계. 화면 기능 분석

입력:

* Frontend URL

분석:

* 실제 화면의 Button, Link, Grid, Tab, Popup
* 화면 진입 시 자동 실행되는 기능
* Action과 연결된 Handler
* 관련 Source 위치

출력:

* 화면 기능 목록 문서
* Action ID (ACT-001, ACT-002 …)

화면 기능을 식별하는 것까지만 수행한다.

⸻

2단계. Frontend 기능 분석

입력:

* 사용자가 선택한 Action ID

분석:

* Event / Handler
* Validation
* Parameter 생성
* 조건 / 분기
* Frontend 비즈니스 로직
* State 처리
* Backend 호출
* HTTP Method / URL
* Request Parameter / Body

출력:

* Frontend 기능 문서
* 발견된 Backend API 정보

Backend 내부 구현은 분석하지 않는다.

⸻

3단계. API 규격 분석

입력:

* 사용자가 선택한 Backend API

분석:

* API 기능
* HTTP Method
* URL
* Header
* Query Parameter
* Path Parameter
* Request Body
* Request Type
* 필수 여부
* Validation
* Response
* Response Type
* HTTP Status
* 오류 Response

API 규격은 가능한 경우 다음 정보를 교차 확인한다.

* 실제 Runtime Request / Response
* Frontend 호출 코드
* Controller
* Request DTO
* Response DTO
* Validation

Controller 이후의 다음 영역은 분석하지 않는다.

* Service
* ServiceImpl
* Mapper
* MyBatis XML
* SQL
* Backend 내부 비즈니스 로직

출력:

* API 규격 문서
* API ID (API-001, API-002 …)

⸻

4단계. Backend 기능 분석

입력:

* 사용자가 선택한 Backend API
* 또는 Backend URL
* 또는 API ID

Backend 분석은 독립된 Backend 분석 Skill의 규칙을 따른다.

CLAUDE.md에서는 Backend Source 탐색 방법을 강제하지 않는다.

Backend 분석에서 필요한 주요 대상은 다음과 같다.

* Controller
* Request / Validation
* Service / ServiceImpl
* Backend 비즈니스 로직
* 조건 / 분기
* 내부 Method
* 다른 Business Service 호출
* 데이터 변환
* 상태 처리
* Mapper
* MyBatis Mapper XML
* Dynamic SQL
* SQL
* 실제 외부 시스템 연동
* SAP / RFC
* 후처리
* Response
* Exception

상세 탐색 순서와 성능 최적화 규칙은
Backend 분석 Skill에서 정의한다.

⸻

3. 단계 독립 원칙

각 단계는 독립적으로 실행한다.

화면 기능 분석
→ 문서 생성
→ 종료

Frontend 기능 분석
→ 문서 생성
→ 종료

API 규격 분석
→ 문서 생성
→ 종료

Backend 기능 분석
→ 문서 생성
→ 종료

현재 단계가 완료되어도
다음 단계로 자동 진행하지 않는다.

다음 분석 대상은 사용자가 직접 선택한다.

⸻

4. 분석 근거

확인되지 않은 내용을 추측하여
사실처럼 작성하지 않는다.

가능한 경우 다음 근거를 사용한다.

1. Runtime
2. Source Code
3. 코드 탐색 결과

Runtime에서 관찰한 값과
Source Code에서 확인한 내용을 구분한다.

코드 이름이나 탐색 결과만으로
비즈니스 로직을 확정하지 않는다.

관련 Source Code를 실제 확인하여 해석한다.

⸻

5. MCP 역할

MCP는 필요한 분석 단계에서만 사용한다.

MCP 사용 자체가 목적이 되어서는 안 된다.

분석 Skill에서 특정 MCP 사용 여부를 별도로 정의한 경우
해당 Skill의 규칙을 우선한다.

Chrome DevTools MCP

Frontend Runtime 확인이 필요한 경우 사용한다.

주요 용도:

* 실제 화면
* DOM / UI Action
* Runtime 동작
* Network
* Request
* Response

주로 화면 기능 분석 및
API 실제 호출 정보 확인에 사용한다.

Code Index MCP

관련 Source 위치나 Symbol 탐색에 사용할 수 있다.

주요 용도:

* Symbol
* Source Path
* Reference
* Caller / Callee
* 호출 관계

단, Code Index 사용을 모든 분석에 강제하지 않는다.

분석 Skill에서 더 빠른 Source 탐색 방법을 정의한 경우
해당 방법을 사용한다.

Code Index 결과만으로
Business Logic을 확정하지 않는다.

⸻

6. Backend Database 분석 원칙

Backend 분석의 기본 Database 근거는
실제 Source에서 확인되는 MyBatis SQL이다.

기본 Source 흐름:

Controller
→ Service
→ ServiceImpl
→ Mapper 호출
→ MyBatis Mapper XML
→ SQL

SQL에서 확인 가능한 다음 정보를 분석할 수 있다.

* SELECT / INSERT / UPDATE / DELETE
* Table
* JOIN
* WHERE
* Parameter
* Dynamic SQL
* include
* resultMap

Database Metadata 조회는
Backend 분석의 기본 필수 단계가 아니다.

별도 분석 Skill 또는 사용자의 명시적 요청이 없는 경우
Database Metadata 조회를 수행하지 않는다.

⸻

7. 분석 범위

현재 선택된 기능과
직접 관련된 범위만 탐색한다.

전체 Repository를 불필요하게 탐색하지 않는다.

현재 단계보다 이후 영역을 미리 분석하지 않는다.

한 단계에서 다음 단계의 대상이 발견되더라도
필요한 식별 정보만 기록하고
상세 분석하지 않는다.

⸻

8. Source 탐색 기본 원칙

Source 탐색은 가능한 한
현재 분석 대상과 직접 연결된 범위에서 수행한다.

다음과 같은 불필요한 전체 탐색은 피한다.

* 전체 Repository 무차별 검색
* 관련성이 확인되지 않은 Class 탐색
* 관련성이 확인되지 않은 Service 탐색
* 관련성이 확인되지 않은 Mapper 탐색
* Dependency 내부 구현 탐색
* JAR 내부 Source 탐색

정확한 Class, Method, Symbol 또는 호출 관계를 확인한 경우
해당 대상을 우선적으로 탐색한다.

상세 탐색 전략은 각 분석 Skill에서 정의한다.

⸻

9. 문서 저장 위치

분석 결과는 화면 단위로 관리한다.

Project Root:

docs/analysis/{화면명}/

기본 구조:

docs/analysis/{화면명}/
├─ SCREEN-{화면명}.md
├─ frontend/
├─ api/
└─ backend/

화면 기능 분석

docs/analysis/{화면명}/SCREEN-{화면명}.md

Frontend 기능 분석

Frontend 분석 Skill에서 정의한
Action 기반 파일명 규칙을 사용한다.

API 규격 분석

API 분석 Skill에서 정의한
API 기반 파일명 규칙을 사용한다.

Backend 기능 분석

Backend 분석 Skill에서 정의한
BE 기반 파일명 규칙을 사용한다.

파일명 상세 규칙은 각 단계의 Skill에서 관리한다.

CLAUDE.md에서 특정 분석 Skill의
세부 파일명 생성 로직을 중복 정의하지 않는다.

필요한 디렉터리가 존재하지 않는 경우 생성한다.

현재 분석 단계에 필요한
디렉터리와 문서만 생성한다.

기존 분석 문서가 존재하는 경우
임의로 덮어쓰지 않는다.

⸻

10. 단계 간 연결

Frontend URL
→ 화면 기능 문서
→ 종료

사용자가 Action 선택
→ Frontend 기능 문서
→ 종료

사용자가 API 선택
→ API 규격 문서
→ 종료

사용자가 Backend 분석 대상 선택
→ Backend 기능 문서
→ 종료

Frontend 분석에서 Backend URL이 발견되어도
Backend 분석을 자동 실행하지 않는다.

API 분석에서 Backend 구현 위치가 발견되어도
Backend 분석을 자동 실행하지 않는다.

⸻

11. Backend 분석 재구축 원칙

Backend 분석은 성능과 정확성을 검증하면서
독립적으로 구축한다.

CLAUDE.md에서는 다음을 강제하지 않는다.

* Agent 사용
* 병렬 분석
* Code Index 사용
* Reference 사용
* Validator 사용
* Database Metadata 조회
* 특정 검색 횟수
* 특정 Backend 탐색 구현

이러한 실행 전략은
Backend 분석 Skill에서 관리한다.

따라서 Backend 분석 방식을 변경하더라도
Frontend / API 분석 구조에는 영향을 주지 않는다.

⸻

12. 핵심 원칙

대상 선택
→ 필요한 범위만 탐색
→ 실제 근거 확인
→ 분석
→ 독립 문서 생성
→ 종료
→ 사용자가 다음 대상 선택