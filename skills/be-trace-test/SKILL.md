---
name: be-trace-test
description: Backend URL 하나를 대상으로 Controller부터 ServiceImpl, 다른 Business Service 호출, Mapper, MyBatis XML, SQL까지 필요한 경우에만 추적하여 Backend 실행 흐름과 탐색 성능을 확인한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Trace Test v0.3 - Business Service Trace

## 1. 목적

v0.2의 빠른 Backend 탐색 구조를 유지하면서
실제 다른 Business Service 호출이 발견된 경우에만
해당 Service 내부로 이동한다.

기본 분석 범위:

Backend URL
→ Controller
→ Service
→ ServiceImpl
→ Mapper
→ MyBatis XML
→ SQL

다른 Business Service 호출이 존재하는 경우:

Backend URL
→ Controller
→ Service A
→ ServiceImpl A
→ Service B
→ ServiceImpl B
→ Mapper
→ MyBatis XML
→ SQL

또는:

Service A
→ Service B
→ Service C
→ Mapper

처럼 실제 호출 관계가 이어지는 경우
Business Service 호출은 계속 추적한다.

핵심 원칙:

다른 Service를 찾기 위해
추가 탐색하지 않는다.

현재 읽고 있는 Method에서
실제 Service 호출이 확인된 경우에만 이동한다.

---

## 2. 입력

HTTP_METHOD

BACKEND_URL

예:

POST /material/create

또는:

UNKNOWN /material/create

---

## 3. 실행 시작

분석 시작 즉시 출력한다.

[BE-TRACE] START

Method: {HTTP_METHOD}
URL: {BACKEND_URL}

각 주요 단계가 완료되면
즉시 Checkpoint를 출력한다.

Checkpoint를 마지막에 몰아서 출력하지 않는다.

---

## 4. 이번 버전에서 하지 않는 것

다음은 수행하지 않는다.

- BE-REFERENCE
- Markdown 문서 생성
- Write
- Agent
- 병렬 처리
- Code Index
- Oracle MCP
- Database Metadata
- Local/private Method 탐색
- Local/private Method 내부 추적
- SAP 내부 추적
- RFC 내부 추적
- 외부 API 내부 추적
- Exception 상세 분석
- Response 상세 분석
- Validation 상세 분석
- Branch 상세 분석
- resultMap 상세 분석
- include 내부 추적
- Mapper.java 기본 탐색

이번 버전에서 새로 추가되는 것은:

다른 Business Service 내부 추적

하나뿐이다.

---

## 5. Source Boundary

Java:

gipms-api-*/src/main/java/**

Resources:

gipms-api-*/src/main/resources/**

이 범위만 사용한다.

---

## 6. Hard Exclude

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

JAR / Dependency fallback은 금지한다.

---

## 7. 공통 FAST 원칙

항상 다음 순서로 진행한다.

정확한 문자열
→ Grep
→ 위치 확보
→ 필요한 범위만 Read
→ 다음 Symbol 확보
→ 다음 단계

금지:

- 전체 Repository 구조 파악
- 전체 Java 파일 Read
- 전체 XML Read
- 전체 Service 목록 탐색
- 관련 파일 사전 수집
- 관련성이 확인되지 않은 Source 탐색
- 이미 찾은 Symbol 재검색

---

## 8. 가장 중요한 탐색 원칙

현재 Method를 읽는 과정에서
이미 보이는 호출만 사용한다.

다음 대상을 찾기 위해
별도의 후보 탐색을 하지 않는다.

예:

materialService.create(...)

가 현재 Method Body에 실제 존재하면
해당 Service를 추적할 수 있다.

반대로:

관련 Service가 있을 것이라고 예상하여

MaterialService

MaterialCheckService

MaterialHistoryService

등을 미리 검색하면 안 된다.

---

## 9. Cache

한 번 확보한 정보는 다시 찾지 않는다.

유지:

CURRENT_PROJECT

CONTROLLER_FILE
CONTROLLER_CLASS
CONTROLLER_METHOD

SERVICE_TYPES
SERVICE_METHODS

SERVICE_IMPL_FILES
SERVICE_IMPL_CLASSES

VISITED_SERVICE_METHODS

MAPPER_TYPES
MAPPER_METHODS
MAPPER_XML

같은 Symbol을 다시 Grep하지 않는다.

---

# PHASE 1. Controller

## 10. Controller 탐색

BACKEND_URL에서
식별력이 높은 Mapping 문자열을 사용한다.

Java Source에서 Controller 후보를 찾는다.

후보에서:

class-level mapping

+

method-level mapping

을 결합하여
BACKEND_URL과 일치하는지 확인한다.

HTTP_METHOD가 지정되어 있으면 함께 확인한다.

HTTP_METHOD가 UNKNOWN이면
Mapping Annotation에서 Method를 확정한다.

---

## 11. Controller Read

Controller Method 위치에서
약 60줄만 먼저 Read한다.

Method 종료가 보이지 않을 때만 확장한다.

전체 Controller 파일은 읽지 않는다.

---

## 12. Controller에서 확보

다음 정보를 확보한다.

- Controller Class
- Controller Method
- Service Variable
- Service Type
- Service Method
- CURRENT_PROJECT

Controller가 위치한 gipms-api-* 프로젝트를
CURRENT_PROJECT로 지정한다.

---

## 13. CHECKPOINT 1

Controller가 확정되는 즉시 출력한다.

[BE-TRACE] CHECKPOINT 1/6 - CONTROLLER FOUND

Project:
{CURRENT_PROJECT}

Controller:
{CONTROLLER_CLASS}#{CONTROLLER_METHOD}

Service Call:
{SERVICE_TYPE}#{SERVICE_METHOD}

Checkpoint 출력 후
즉시 Service 탐색으로 이동한다.

---

# PHASE 2. Entry Service / ServiceImpl

## 14. Service 탐색

Controller에서 확보한
정확한 Service Type만 사용한다.

검색 범위:

CURRENT_PROJECT/src/main/java/**

전체 Service 목록을 탐색하지 않는다.

Service 이름을 추측하지 않는다.

---

## 15. Service Method

Controller에서 실제 호출한
Service Method 선언만 확인한다.

Service Interface 전체를 분석하지 않는다.

필요한 정보가 확보되면
즉시 ServiceImpl 탐색으로 이동한다.

---

## 16. ServiceImpl 탐색

정확한:

implements ServiceType

또는 Source에서 이미 확인된
구현 Class 이름을 사용한다.

검색 범위:

CURRENT_PROJECT/src/main/java/**

구현체가 확정되면
다른 후보 탐색을 중단한다.

---

## 17. ServiceImpl Method

Controller에서 호출한
정확한 Service Method만 찾는다.

Method 시작 위치에서
약 80줄을 먼저 Read한다.

Method 종료가 보이지 않을 때만
추가 범위를 Read한다.

전체 ServiceImpl 파일을 읽지 않는다.

---

## 18. CHECKPOINT 2

Entry ServiceImpl Method가 확보되는 즉시 출력한다.

[BE-TRACE] CHECKPOINT 2/6 - ENTRY SERVICE IMPL FOUND

Service:
{SERVICE_TYPE}#{SERVICE_METHOD}

ServiceImpl:
{SERVICE_IMPL_CLASS}#{SERVICE_METHOD}

---

# PHASE 3. Business Service Trace

## 19. Business Service 호출 확인

현재 읽고 있는 ServiceImpl Method Body에서
실제 다른 Service 호출이 보이는지 확인한다.

예:

materialCheckService.validate(...)

historyService.saveHistory(...)

equipmentService.getEquipment(...)

실제 호출 표현식이 존재하는 경우에만
추적 후보로 사용한다.

---

## 20. Service 후보를 찾기 위한 추가 검색 금지

다른 Business Service가 있는지 확인하기 위해:

Repository 전체 Grep

Service 이름 패턴 검색

@Service 전체 검색

*Service.java 검색

*ServiceImpl.java 검색

을 수행하지 않는다.

현재 Method Body에서
실제 호출이 보여야 한다.

---

## 21. Service Type 확보

다른 Service 호출이 발견된 경우:

예:

materialCheckService.validate(...)

먼저 현재 ServiceImpl Source에서
materialCheckService의 Type을 확인한다.

현재 읽은 범위에서 Type이 이미 보이면
그 정보를 그대로 사용한다.

보이지 않을 때만:

현재 ServiceImpl 파일

하나에서 변수명:

materialCheckService

를 정확히 Grep한다.

Repository 전체에서
변수명을 검색하지 않는다.

---

## 22. 다른 Service 탐색

확보한 정확한 Service Type으로만 탐색한다.

예:

MaterialCheckService

검색 범위:

CURRENT_PROJECT/src/main/java/**

정확한 Service Type을 찾는다.

전체 Service 후보를 수집하지 않는다.

---

## 23. 다른 Service Method

실제 호출된 Method만 확인한다.

예:

materialCheckService.validate(...)

이면:

MaterialCheckService#validate

만 확인한다.

Service Interface의 다른 Method는 분석하지 않는다.

---

## 24. 다른 ServiceImpl 탐색

정확한 Service Type을 구현하는
ServiceImpl만 찾는다.

예:

implements MaterialCheckService

구현체가 확인되면
다른 후보 탐색을 중단한다.

---

## 25. 다른 ServiceImpl Method Read

실제 호출된 Method 위치를 찾는다.

초기:

약 80줄

만 Read한다.

Method 종료가 보이지 않을 때만
추가 범위를 읽는다.

전체 ServiceImpl 파일을 읽지 않는다.

---

## 26. Business Service 내부 분석

다른 Service Method 안에서는
다음만 확인한다.

- 직접 Mapper 호출
- 다른 Business Service 호출
- 외부 시스템 호출 이름

Local/private Method는 추적하지 않는다.

Validation 상세 분석도 하지 않는다.

Branch 상세 분석도 하지 않는다.

---

## 27. Service → Service → Service

다른 ServiceImpl Method 안에서
또 다른 Business Service 호출이
실제로 발견되면 동일 규칙으로 추적한다.

예:

Service A

→ Service B

→ Service C

→ Mapper

실제 호출 관계가 이어지는 동안
Business Service는 계속 따라갈 수 있다.

---

## 28. Service 추적 깊이

고정 Depth 제한을 두지 않는다.

실제 Service 호출 관계가 이어지는 동안
계속 추적한다.

단:

이미 분석한:

Service Type + Method

조합은 다시 분석하지 않는다.

VISITED_SERVICE_METHODS에 기록한다.

---

## 29. 순환 Service 호출 방지

예:

Service A#methodA

→ Service B#methodB

→ Service A#methodA

구조가 있더라도

이미 VISITED_SERVICE_METHODS에 존재하면
다시 Source를 읽지 않는다.

호출 관계만 기록하고 종료한다.

---

## 30. 동일 Service 다른 Method

같은 Service Type이라도
다른 Method가 실제 호출되면
별도의 실행 단계로 취급할 수 있다.

예:

MaterialService#create

→ MaterialService#validate

단:

Local/private Method는 이번 버전에서
추적 대상이 아니다.

실제 주입된 Business Service 호출인 경우에만
Service Trace 대상으로 처리한다.

---

## 31. Mapper 호출 수집

다음 위치에서 발견된 Mapper 호출을 모두 수집한다.

- Entry ServiceImpl Method
- 추적된 다른 Business ServiceImpl Method

중복:

Mapper Type + Mapper Method

조합은 한 번만 XML/SQL 분석한다.

---

## 32. 외부 시스템 호출

현재 Method에서 다음과 같은 호출이 발견되어도:

sapService.send(...)

rfcClient.execute(...)

externalClient.call(...)

이번 버전에서는 내부로 들어가지 않는다.

호출 이름만 기록한다.

예:

External:
sapService.send

---

## 33. CHECKPOINT 3

Business Service 추적이 완료되면
즉시 출력한다.

[BE-TRACE] CHECKPOINT 3/6 - BUSINESS SERVICES TRACED

Additional Service Count:
{COUNT}

Service Flow:

{ENTRY_SERVICE}#{METHOD}
→ {SERVICE_B}#{METHOD}
→ {SERVICE_C}#{METHOD}

다른 Service 호출이 없으면:

Additional Service Count:
0

Service Flow:

{ENTRY_SERVICE}#{METHOD}

으로 출력한다.

---

# PHASE 4. Mapper Calls

## 34. Mapper 호출 대상

Entry ServiceImpl과
추적된 Business ServiceImpl에서
실제 발견된 Mapper 호출만 사용한다.

Mapper를 별도로 찾기 위한
후보 검색을 하지 않는다.

---

## 35. Mapper 호출에서 확보

각 Mapper 호출에서:

- Mapper Variable
- Mapper Type
- Mapper Method

를 확보한다.

예:

Variable:
materialMapper

Type:
MaterialMapper

Method:
selectMaterial

---

## 36. Mapper Type 확인

현재 읽은 Method 범위에서
Mapper Type이 이미 확인되면 그대로 사용한다.

보이지 않을 때만:

해당 ServiceImpl 파일

하나에서 Mapper Variable을
정확히 Grep한다.

Repository 전체에서
Mapper Variable을 검색하지 않는다.

---

## 37. Mapper.java

Mapper Java Interface는
기본적으로 읽지 않는다.

기본 흐름:

ServiceImpl
→ Mapper Type
→ Mapper Method
→ MyBatis XML

Mapper.java를 통한
중간 검증을 하지 않는다.

---

## 38. CHECKPOINT 4

Mapper 호출 목록이 확보되면
즉시 출력한다.

[BE-TRACE] CHECKPOINT 4/6 - DIRECT MAPPER CALLS FOUND

Mapper Call Count:
{COUNT}

Mapper Calls:

1. {MAPPER_TYPE}#{MAPPER_METHOD}
2. {MAPPER_TYPE}#{MAPPER_METHOD}
3. ...

Mapper 호출이 없으면:

Mapper Call Count:
0

으로 출력한다.

---

# PHASE 5. MyBatis XML

## 39. XML 탐색

각 Mapper Type에 대해
CURRENT_PROJECT의:

src/main/resources/**

에서 namespace를 찾는다.

예:

<mapper namespace="...MaterialMapper">

정확한 XML이 발견되면
해당 Mapper Type의 다른 XML 탐색을 중단한다.

---

## 40. Mapper XML Cache

동일 Mapper Type에 대해
XML을 이미 찾았다면
다시 namespace 검색하지 않는다.

예:

MaterialMapper#selectMaterial

MaterialMapper#insertMaterial

MaterialMapper#updateMaterial

세 Method가 있어도:

MaterialMapper XML

은 한 번만 찾는다.

이후 같은 XML에서
각 Statement ID만 확인한다.

---

## 41. XML Fallback

Mapper Type namespace로 찾지 못한 경우에만
정확한 Mapper Method ID를 사용한다.

예:

id="selectMaterial"

검색 범위:

CURRENT_PROJECT/src/main/resources/**

다른 Backend 프로젝트로
검색 범위를 확장하지 않는다.

---

## 42. Statement 탐색

확정된 Mapper XML에서
실제 Mapper Method와 연결되는 Statement만 찾는다.

대상:

<select>

<insert>

<update>

<delete>

예:

<select id="selectMaterial">

---

## 43. XML 부분 Read

Statement 시작 위치에서
약 40줄만 먼저 Read한다.

다음 종료 Tag가 확인되면
즉시 해당 Statement Read를 종료한다.

</select>

</insert>

</update>

</delete>

40줄 안에 종료되지 않을 때만
추가 범위를 읽는다.

전체 XML 파일은 읽지 않는다.

---

## 44. CHECKPOINT 5

필요한 XML Statement가 확보되면
즉시 출력한다.

[BE-TRACE] CHECKPOINT 5/6 - MYBATIS XML FOUND

XML Count:
{COUNT}

Statements:

1.

Mapper:
{MAPPER_TYPE}#{MAPPER_METHOD}

XML:
{XML_PATH}

Statement:
{STATEMENT_ID}

2.
...

---

# PHASE 6. SQL

## 45. SQL 분석 범위

각 Statement에서 다음만 확인한다.

- SQL Type
- Main Table
- JOIN Table
- WHERE
- Parameter
- Dynamic SQL 존재 여부
- include 존재 여부

SQL을 장문으로 설명하지 않는다.

---

## 46. SQL Type

다음 중 하나로 기록한다.

SELECT

INSERT

UPDATE

DELETE

---

## 47. Table

SQL Source에서
직접 확인되는 Table만 기록한다.

확인:

- Main Table
- JOIN Table

추측하지 않는다.

---

## 48. Parameter

SQL에서 직접 사용되는
MyBatis Parameter를 확인한다.

예:

#{materialId}

#{plantCode}

${value}

상세 Request → SQL Mapping은
이번 버전에서 분석하지 않는다.

---

## 49. Dynamic SQL

다음 Tag가 존재하면 기록한다.

<if>

<choose>

<when>

<otherwise>

<foreach>

이번 버전에서는:

Dynamic SQL:
YES

또는:

Dynamic SQL:
NO

정도로 기록한다.

상세 Branch 분석은 하지 않는다.

---

## 50. include

Statement에:

<include refid="..."/>

가 있으면:

Include:
{refid}

만 기록한다.

include 내부 Source는 추적하지 않는다.

---

## 51. CHECKPOINT 6

SQL 기본 정보가 확보되면
즉시 출력한다.

[BE-TRACE] CHECKPOINT 6/6 - SQL FOUND

SQL Statements:

1.

Mapper:
{MAPPER_TYPE}#{MAPPER_METHOD}

SQL Type:
{SQL_TYPE}

Main Table:
{MAIN_TABLE}

Dynamic SQL:
{YES|NO}

Include:
{REFID|NONE}

2.
...

---

# PHASE 7. Final Result

## 52. 종료 조건

CHECKPOINT 6까지 완료하면
추가 Source 탐색을 하지 않는다.

다음은 분석하지 않는다.

- Local/private Method
- SAP/RFC 내부
- External API 내부
- Response 상세
- Exception 상세
- Validation 상세
- Branch 상세
- resultMap
- include 내부

---

## 53. 최종 결과

파일을 생성하지 않는다.

화면에 다음 형식으로 출력한다.

=== BE TRACE TEST v0.3 ===

HTTP METHOD:
{HTTP_METHOD}

BACKEND URL:
{BACKEND_URL}

CURRENT PROJECT:
{CURRENT_PROJECT}

CONTROLLER

Class:
{CONTROLLER_CLASS}

Method:
{CONTROLLER_METHOD}

Evidence:
{PATH:LINES}

ENTRY SERVICE

Type:
{SERVICE_TYPE}

Method:
{SERVICE_METHOD}

SERVICE IMPL

Class:
{SERVICE_IMPL_CLASS}

Method:
{SERVICE_METHOD}

Evidence:
{PATH:LINES}

BUSINESS SERVICE TRACE

Additional Service Count:
{COUNT}

Flow:

{ENTRY_SERVICE}#{METHOD}
→ {SERVICE_B}#{METHOD}
→ {SERVICE_C}#{METHOD}

Services:

1.

Service:
{SERVICE_TYPE}

Method:
{SERVICE_METHOD}

Implementation:
{SERVICE_IMPL_CLASS}

Evidence:
{PATH:LINES}

2.
...

DIRECT MAPPER CALLS

1.

Called From:
{SERVICE_TYPE}#{SERVICE_METHOD}

Mapper Type:
{MAPPER_TYPE}

Mapper Method:
{MAPPER_METHOD}

XML:
{XML_PATH}

Statement:
{STATEMENT_ID}

SQL Type:
{SQL_TYPE}

Main Table:
{MAIN_TABLE}

Dynamic SQL:
{YES|NO}

Include:
{REFID|NONE}

Evidence:
{PATH:LINES}

EXTERNAL CALLS

{CALLS|NONE}

TRACE

{CONTROLLER_CLASS}#{CONTROLLER_METHOD}

→ {ENTRY_SERVICE}#{METHOD}

→ {SERVICE_B}#{METHOD}

→ {SERVICE_C}#{METHOD}

→ {MAPPER_TYPE}#{MAPPER_METHOD}

→ {STATEMENT_ID}

→ {SQL_TYPE} {MAIN_TABLE}

STATUS:
COMPLETED

---

## 54. Evidence

Evidence는 Source 탐색 과정에서
이미 확인한 위치를 사용한다.

Evidence 때문에
추가 Grep이나 Read를 하지 않는다.

형식:

Project Root 기준 상대경로:라인범위

절대경로는 출력하지 않는다.

---

## 55. 탐색 실패

특정 단계에서 Source를 찾지 못하면
검색 범위를 무작정 확장하지 않는다.

다음 중 하나를 기록한다.

CONTROLLER_NOT_FOUND

SERVICE_NOT_FOUND

SERVICE_IMPL_NOT_FOUND

BUSINESS_SERVICE_NOT_FOUND

BUSINESS_SERVICE_IMPL_NOT_FOUND

MAPPER_TYPE_NOT_FOUND

MAPPER_XML_NOT_FOUND

STATEMENT_NOT_FOUND

SQL_NOT_FOUND

JAR이나 Dependency로 이동하지 않는다.

---

## 56. Business Service 탐색 실패

호출은 존재하지만
Service Type 또는 구현체를 찾지 못한 경우:

검색 범위를 다른 프로젝트까지
무작정 확장하지 않는다.

확인된 호출만 기록하고
해당 Branch 추적을 종료한다.

다른 정상 Branch 분석은 계속한다.

---

## 57. Checkpoint 순서

[BE-TRACE] START

↓

CHECKPOINT 1/6
CONTROLLER FOUND

↓

CHECKPOINT 2/6
ENTRY SERVICE IMPL FOUND

↓

CHECKPOINT 3/6
BUSINESS SERVICES TRACED

↓

CHECKPOINT 4/6
DIRECT MAPPER CALLS FOUND

↓

CHECKPOINT 5/6
MYBATIS XML FOUND

↓

CHECKPOINT 6/6
SQL FOUND

↓

FINAL RESULT

↓

종료

---

## 58. 핵심 성능 원칙

다른 Business Service 추적 기능 때문에
평소 탐색량이 증가하면 안 된다.

다른 Service 호출이 없는 URL에서는
v0.2와 거의 동일한 탐색 흐름을 유지한다.

즉:

현재 Method에서
다른 Service 호출 없음

이면:

추가 Service 검색 0회

이어야 한다.

다른 Service 호출이 있을 때만:

호출된 Service Type 확인

→ 정확한 Service 탐색

→ 정확한 ServiceImpl 탐색

→ 호출된 Method 부분 Read

를 수행한다.

---

## 59. 금지되는 패턴

다음 방식으로 분석하지 않는다.

ServiceImpl 발견

→ 관련 Service 전부 검색

→ 관련 Impl 전부 검색

→ 관련 Mapper 전부 검색

이 방식은 금지한다.

반드시 실제 호출 기준으로만 진행한다.

올바른 방식:

현재 Method

→ 실제 otherService.method() 발견

→ otherService Type 확인

→ 해당 Service만 탐색

→ 해당 Method만 Read

---

## 60. 완료

최종 결과 출력 후 즉시 종료한다.

추가 Source 탐색을 하지 않는다.

다음 분석 단계를 자동 실행하지 않는다.