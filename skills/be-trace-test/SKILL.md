---
name: be-trace-test
description: Backend URL 하나를 대상으로 Controller부터 ServiceImpl, Local/private Method, Mapper 호출, MyBatis XML, SQL까지 최소 탐색으로 추적하여 Backend 실행 흐름과 탐색 성능을 확인한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Trace Test v0.3 - Local Method Trace

## 1. 목적

v0.2의 빠른 Backend 탐색 구조를 유지하면서
ServiceImpl 내부의 Local/private Method 호출 추적만 추가한다.

분석 범위:

Backend URL
→ Controller
→ Service
→ ServiceImpl
→ Local/private Method
→ Local/private Method
→ 직접 Mapper 호출
→ MyBatis XML
→ SQL

이번 버전의 핵심은:

ServiceImpl의 시작 Method에서 호출되는
같은 Class 내부 Method를 끝까지 추적하는 것이다.

단:

다른 Service 내부로는 들어가지 않는다.

SAP/RFC/외부 시스템 내부로도 들어가지 않는다.

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
- 다른 Service 내부 추적
- SAP 내부 추적
- RFC 내부 추적
- 외부 API 내부 추적
- Exception 상세 분석
- Response 상세 분석
- Validation 상세 분석
- 전체 Branch 상세 분석
- resultMap 상세 분석
- include 내부 추적
- Mapper.java 기본 탐색

이번 버전에서 새로 추가되는 것은:

Local/private Method 추적

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
- 관련 파일 사전 수집
- 관련성이 확인되지 않은 Source 탐색
- 이미 찾은 Symbol 재검색

---

## 8. Cache

한 번 확보한 정보는 다시 찾지 않는다.

유지:

CURRENT_PROJECT

CONTROLLER_FILE
CONTROLLER_CLASS
CONTROLLER_METHOD

SERVICE_TYPE
SERVICE_METHOD

SERVICE_IMPL_FILE
SERVICE_IMPL_CLASS

LOCAL_METHODS
VISITED_LOCAL_METHODS

MAPPER_TYPES
MAPPER_METHODS
MAPPER_XML

같은 Symbol을 다시 Grep하지 않는다.

---

# PHASE 1. Controller

## 9. Controller 탐색

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

## 10. Controller Read

Controller Method 위치에서
약 60줄만 먼저 Read한다.

Method 종료가 보이지 않을 때만 확장한다.

전체 Controller 파일은 읽지 않는다.

---

## 11. Controller에서 확보

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

## 12. CHECKPOINT 1

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

# PHASE 2. Service / ServiceImpl

## 13. Service 탐색

Controller에서 확보한
정확한 Service Type만 사용한다.

검색 범위:

CURRENT_PROJECT/src/main/java/**

Service 이름을 추측하지 않는다.

전체 Service 목록을 탐색하지 않는다.

---

## 14. Service Method

Controller에서 실제 호출한
Service Method 선언만 확인한다.

Service Interface 전체를 분석하지 않는다.

필요한 정보가 확보되면
즉시 ServiceImpl 탐색으로 이동한다.

---

## 15. ServiceImpl 탐색

정확한:

implements ServiceType

또는 이미 Source에서 확인한
구현 Class 이름을 사용한다.

검색 범위:

CURRENT_PROJECT/src/main/java/**

구현체가 확정되면
다른 후보 탐색을 중단한다.

---

## 16. ServiceImpl Method

Controller에서 호출한
정확한 Service Method만 찾는다.

Method 시작 위치에서
약 80줄을 먼저 Read한다.

Method 종료가 보이지 않을 때만
추가 범위를 Read한다.

전체 ServiceImpl 파일을 읽지 않는다.

---

## 17. CHECKPOINT 2

ServiceImpl Method가 확보되는 즉시 출력한다.

[BE-TRACE] CHECKPOINT 2/6 - SERVICE IMPL FOUND

Service:
{SERVICE_TYPE}#{SERVICE_METHOD}

ServiceImpl:
{SERVICE_IMPL_CLASS}#{SERVICE_METHOD}

Checkpoint 출력 후
Local/private Method 탐색으로 이동한다.

---

# PHASE 3. Local/private Method Trace

## 18. Local Method 정의

현재 ServiceImpl Class 내부에 선언되어 있고
현재 분석 Method 또는 다른 Local Method에서
직접 호출되는 Method를 Local Method로 본다.

예:

public void createMaterial(...) {

    validateMaterial(...);

    String code = makeMaterialCode(...);

    saveMaterial(...);
}

위의:

validateMaterial
makeMaterialCode
saveMaterial

가 같은 ServiceImpl Class에 선언되어 있다면
Local Method 후보이다.

---

## 19. Local Method 후보 추출

현재 분석 중인 Method Body에서
Method Call을 확인한다.

단순히 호출된 모든 Method를
Local Method라고 판단하지 않는다.

같은 ServiceImpl 파일 안에
실제 Method 선언이 존재하는 경우에만
Local Method로 확정한다.

---

## 20. Local Method 확인 범위

Local Method 확인은:

SERVICE_IMPL_FILE

하나에서만 수행한다.

다른 Java 파일을 검색하지 않는다.

Repository 전체에서
Method 이름을 검색하지 않는다.

---

## 21. Local Method 탐색

Local Method 후보가 있으면
SERVICE_IMPL_FILE 안에서
정확한 Method 선언을 찾는다.

Method 위치가 확인되면
해당 Method 주변만 Read한다.

초기 범위:

약 60줄

Method 종료가 보이지 않을 때만
추가 Read한다.

전체 ServiceImpl 파일 Read는 하지 않는다.

---

## 22. Recursive Local Trace

Local Method 안에서
다른 Local Method 호출이 발견되면
동일한 규칙으로 계속 추적한다.

예:

mainMethod()

→ validate()

→ validatePlant()

→ checkPlantCode()

같은 ServiceImpl Class 내부라면
끝까지 따라간다.

---

## 23. Local Method 깊이

고정 Depth 제한을 두지 않는다.

같은 ServiceImpl Class 내부에서
실제 호출 관계가 이어지는 동안 추적한다.

단:

이미 분석한 Method는 다시 분석하지 않는다.

VISITED_LOCAL_METHODS에 등록한다.

---

## 24. 순환 호출 방지

예:

methodA()
→ methodB()
→ methodA()

와 같은 구조가 있더라도
이미 VISITED_LOCAL_METHODS에 존재하는 Method는
다시 Read하지 않는다.

출력에는 호출 관계만 기록할 수 있다.

무한 반복 탐색을 하지 않는다.

---

## 25. Local Method에서 확인할 것

각 Local Method에서는 다음만 확인한다.

- Method 이름
- 호출 관계
- 직접 Mapper 호출
- 추가 Local Method 호출
- 다른 Service 호출 이름
- 외부 연동 호출 이름

이번 단계에서는
비즈니스 로직을 상세 해석하지 않는다.

---

## 26. Local Method 안의 Mapper

Local Method 안에서
Mapper 호출이 발견되면
DIRECT MAPPER CALLS에 포함한다.

예:

mainMethod()

→ validate()

→ saveHistory()

→ historyMapper.insertHistory()

이면:

historyMapper.insertHistory()

도 Mapper 분석 대상이다.

---

## 27. Local Method 안의 다른 Service

예:

materialCheckService.check(...)

가 발견되더라도
이번 버전에서는 내부로 들어가지 않는다.

다음에만 기록한다.

Other Service:
materialCheckService.check

---

## 28. Local Method 안의 외부 연동

예:

sapService.send(...)

rfcClient.execute(...)

externalClient.call(...)

등이 발견되어도
이번 버전에서는 내부 추적하지 않는다.

다음에만 기록한다.

External:
sapService.send

---

## 29. Local Trace 종료 조건

현재 Method에서 시작하여
연결된 모든 Local Method를 확인했고

새로운 Local Method가 더 이상 발견되지 않으면
Local Trace를 종료한다.

---

## 30. CHECKPOINT 3

Local/private Method 추적이 완료되면
즉시 출력한다.

[BE-TRACE] CHECKPOINT 3/6 - LOCAL METHODS TRACED

Local Method Count:
{COUNT}

Local Methods:

1. {METHOD}
2. {METHOD}
3. ...

호출 흐름:

{SERVICE_METHOD}
→ {LOCAL_METHOD}
→ {LOCAL_METHOD}

Local Method가 없으면:

Local Method Count:
0

Local Methods:
NONE

으로 출력한다.

---

# PHASE 4. Direct Mapper Calls

## 31. Mapper 분석 대상

다음 위치에서 발견된
모든 직접 Mapper 호출을 합친다.

1. ServiceImpl 시작 Method

2. 추적된 Local/private Method

중복 Mapper 호출은
같은 Mapper Type + Method 기준으로
한 번만 분석한다.

---

## 32. Mapper 호출에서 확보

각 Mapper 호출에서 다음을 확보한다.

- Mapper Variable
- Mapper Type
- Mapper Method

예:

Variable:
materialMapper

Type:
MaterialMapper

Method:
selectMaterial

---

## 33. Mapper Type 확인

현재 읽은 Method 범위에서
Mapper Type이 확인되면 그대로 사용한다.

확인되지 않을 때만
SERVICE_IMPL_FILE에서
Mapper Variable 이름을 정확히 Grep한다.

Mapper Type을 확보하면
즉시 탐색을 종료한다.

---

## 34. Mapper.java

Mapper Java Interface는
기본적으로 읽지 않는다.

기본 흐름:

ServiceImpl / Local Method
→ Mapper Type
→ Mapper Method
→ MyBatis XML

Mapper.java를 통한
중간 검증은 하지 않는다.

---

## 35. CHECKPOINT 4

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

## 36. XML 탐색

각 Mapper Type에 대해
CURRENT_PROJECT의:

src/main/resources/**

에서 namespace를 찾는다.

예:

<mapper namespace="...MaterialMapper">

정확한 XML이 발견되면
해당 Mapper Type에 대한
다른 XML 탐색을 중단한다.

---

## 37. Mapper XML Cache

동일 Mapper Type에 대해
XML을 이미 찾았다면
다시 namespace 검색을 하지 않는다.

예:

MaterialMapper#selectMaterial

MaterialMapper#insertMaterial

MaterialMapper#updateMaterial

세 Method가 있더라도

MaterialMapper XML은
한 번만 찾는다.

그 XML 안에서
각 Statement ID만 찾는다.

---

## 38. XML Fallback

Mapper Type namespace로 찾지 못한 경우에만
정확한 Mapper Method ID를 사용할 수 있다.

예:

id="selectMaterial"

검색 범위는:

CURRENT_PROJECT/src/main/resources/**

로 제한한다.

다른 Backend 프로젝트까지
검색 범위를 확장하지 않는다.

---

## 39. Statement 탐색

확정된 Mapper XML 안에서
실제 Mapper Method와 연결되는 Statement만 찾는다.

대상:

<select>

<insert>

<update>

<delete>

예:

<select id="selectMaterial">

---

## 40. XML 부분 Read

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

## 41. CHECKPOINT 5

필요한 XML Statement가 모두 확보되면
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

## 42. SQL 분석 범위

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

## 43. SQL Type

다음 중 하나로 기록한다.

SELECT

INSERT

UPDATE

DELETE

---

## 44. Table

SQL Source에서
직접 확인되는 Table만 기록한다.

확인:

- Main Table
- JOIN Table

Table을 추측하지 않는다.

---

## 45. Parameter

SQL에서 직접 사용되는
MyBatis Parameter를 확인한다.

예:

#{materialId}

#{plantCode}

${value}

Request부터 SQL까지의
상세 Parameter Mapping은
이번 버전에서 분석하지 않는다.

---

## 46. Dynamic SQL

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

정도로만 기록한다.

상세 Branch 분석은 하지 않는다.

---

## 47. include

Statement에:

<include refid="..."/>

가 있으면:

Include:
{refid}

만 기록한다.

include 내부 Source는
이번 버전에서 추적하지 않는다.

---

## 48. CHECKPOINT 6

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

## 49. 종료 조건

CHECKPOINT 6까지 완료하면
추가 Source 탐색을 하지 않는다.

다음은 분석하지 않는다.

- 다른 Service 내부
- SAP/RFC 내부
- External API 내부
- Response 상세
- Exception 상세
- Validation 상세
- Branch 상세
- resultMap
- include 내부

---

## 50. 최종 결과

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

SERVICE

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

LOCAL METHOD TRACE

Local Method Count:
{COUNT}

Flow:

{SERVICE_METHOD}
→ {LOCAL_METHOD}
→ {LOCAL_METHOD}

Local Methods:

1.
Method:
{METHOD}

Evidence:
{PATH:LINES}

2.
Method:
{METHOD}

Evidence:
{PATH:LINES}

DIRECT MAPPER CALLS

1.

Called From:
{SERVICE_METHOD|LOCAL_METHOD}

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

NON-TRACED CALLS

Other Services:
{NAMES|NONE}

External:
{NAMES|NONE}

TRACE

{CONTROLLER_CLASS}#{CONTROLLER_METHOD}
→ {SERVICE_TYPE}#{SERVICE_METHOD}
→ {SERVICE_IMPL_CLASS}#{SERVICE_METHOD}

Local:

{SERVICE_METHOD}
→ {LOCAL_METHOD}
→ {LOCAL_METHOD}

Mapper:

{CALLING_METHOD}
→ {MAPPER_TYPE}#{MAPPER_METHOD}
→ {STATEMENT_ID}
→ {SQL_TYPE} {MAIN_TABLE}

STATUS:
COMPLETED

---

## 51. Evidence

Evidence는 Source 탐색 과정에서
이미 확인한 위치를 사용한다.

Evidence를 만들기 위해
추가 Grep이나 Read를 하지 않는다.

형식:

Project Root 기준 상대경로:라인범위

절대경로는 출력하지 않는다.

---

## 52. 탐색 실패

특정 단계에서 Source를 찾지 못하면
검색 범위를 무작정 확장하지 않는다.

다음 중 하나를 기록한다.

CONTROLLER_NOT_FOUND

SERVICE_NOT_FOUND

SERVICE_IMPL_NOT_FOUND

LOCAL_METHOD_NOT_FOUND

MAPPER_TYPE_NOT_FOUND

MAPPER_XML_NOT_FOUND

STATEMENT_NOT_FOUND

SQL_NOT_FOUND

JAR이나 Dependency로 이동하지 않는다.

---

## 53. Local Method 탐색 실패 규칙

호출 이름은 보이지만
SERVICE_IMPL_FILE 안에서
실제 Method 선언을 찾지 못한 경우

Local Method라고 확정하지 않는다.

다른 파일에서 같은 이름을 검색하지 않는다.

NON-TRACED CALLS에
필요한 경우 이름만 기록한다.

---

## 54. Checkpoint 원칙

Checkpoint는
각 단계가 완료되는 즉시 출력한다.

순서:

[BE-TRACE] START

↓

CHECKPOINT 1/6
CONTROLLER FOUND

↓

CHECKPOINT 2/6
SERVICE IMPL FOUND

↓

CHECKPOINT 3/6
LOCAL METHODS TRACED

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

## 55. 핵심 성능 원칙

Local Method 추적을 추가했다고 해서
Repository 검색 범위를 넓히지 않는다.

Local Method 탐색 범위는 항상:

SERVICE_IMPL_FILE

하나로 제한한다.

Local Method마다
전체 파일을 다시 읽지 않는다.

이미 읽은 Source 범위는 재사용한다.

이미 방문한 Local Method는
다시 분석하지 않는다.

Mapper Type과 Mapper XML도
한 번 찾은 결과를 재사용한다.

---

## 56. 완료

최종 결과 출력 후 즉시 종료한다.

추가 Source 탐색을 하지 않는다.

다음 분석 단계를 자동 실행하지 않는다.