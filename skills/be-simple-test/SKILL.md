---
name: be-simple-test
description: Backend URL에서 실제 호출되는 실행 경로를 추적하고 Mapper는 MyBatis XML 파일 위치와 Statement ID까지만 확인하며 Response와 Exception 흐름을 분석한다.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Grep, Read
---

# BE Simple Test v0.3 - Response / Exception

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
→ MyBatis XML 위치
→ Statement ID
→ Response
→ Exception

핵심 원칙:

검색 범위는 좁게 유지하고
실제 호출 깊이는 끝까지 따라간다.

---

## 2. MyBatis 정책

MyBatis는 다음까지만 확인한다.

Mapper Type
→ Mapper Method
→ XML 파일 위치
→ Statement ID

여기서 종료한다.

다음은 분석하지 않는다.

- SQL Body
- SQL Query
- SELECT 내용
- INSERT 내용
- UPDATE 내용
- DELETE 내용
- Main Table
- JOIN
- WHERE
- Parameter
- Dynamic SQL
- include 내부
- resultMap
- Oracle Metadata

쿼리를 확인하기 위해
Statement Body를 Read하지 않는다.

---

## 3. 입력

HTTP Method

Backend URL

예:

POST /material/create

또는:

UNKNOWN /material/create

---

## 4. 분석 범위

실제 Source에 존재하는 경우 다음을 추적한다.

Controller

Service

ServiceImpl

Local/private Method

다른 Business Service

Mapper

MyBatis XML 위치

Statement ID

Response

Exception

SAP/RFC/External 호출 존재 여부

모든 단계가 반드시 존재할 필요는 없다.

실제 호출되는 흐름만 분석한다.

---

## 5. 하지 않는 것

다음은 수행하지 않는다.

- SQL 분석
- SQL Body Read
- Oracle Metadata
- Database Metadata
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

## 6. Java

Java Source:

gipms-api-*/src/main/java/**

Controller가 발견된 프로젝트를:

CURRENT_PROJECT

로 사용한다.

---

## 7. Resources

MyBatis 탐색 범위:

CURRENT_PROJECT/src/main/resources/**

다른 Backend 프로젝트의 Resources까지
검색 범위를 확장하지 않는다.

---

## 8. Hard Exclude

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

## 9. 탐색 방식

항상:

정확한 Symbol
→ Grep
→ 위치 확보
→ 필요한 부분만 Read
→ 다음 Symbol

순서로 진행한다.

---

## 10. 금지

다음 방식은 사용하지 않는다.

Repository 전체 구조 파악

전체 Java 파일 목록 수집

전체 XML 파일 목록 수집

전체 Service 목록 수집

전체 Mapper 목록 수집

관련 Source 사전 탐색

관련 있어 보인다는 이유의 Source 탐색

전체 파일 기본 Read

---

## 11. Cache

이미 확보한 정보는 재사용한다.

다음 조합은 다시 분석하지 않는다.

Class + Method

Mapper Type + Mapper Method

Mapper Type + XML

XML + Statement ID

같은 Symbol을 반복 Grep하지 않는다.

---

# Controller

## 12. Controller 탐색

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

## 13. Controller 분석

실제 요청을 처리하는
Controller Method만 읽는다.

전체 Controller 파일을
기본적으로 읽지 않는다.

---

## 14. Controller에서 확보

다음을 확보한다.

Controller Class

Controller Method

Service Type

Service Method

Request 전달 방식

Response 방식

CURRENT_PROJECT

---

# Service

## 15. Service

Controller에서 실제 호출되는
Service만 따라간다.

전체 Service 목록을 찾지 않는다.

실제 호출된 Method만 확인한다.

---

## 16. ServiceImpl

정확한 Service 구현체만 찾는다.

실제 호출된 Method 부분만 읽는다.

전체 ServiceImpl 파일을
기본적으로 읽지 않는다.

---

# Execution Flow

## 17. 실제 호출 기준

현재 Method에서
실제로 호출되는 코드만 추적한다.

관련 있어 보인다는 이유로
다른 Source를 찾지 않는다.

---

# Local/private Method

## 18. Local/private Method

현재 실행 경로에서
실제로 호출되는 Local/private Method만 따라간다.

동일 Class 안에서
실제 선언이 확인되는 Method만 처리한다.

Local Method 목록을
미리 만들지 않는다.

---

## 19. Local 중복 방지

이미 분석한:

Class + Method

는 다시 읽지 않는다.

순환 호출이면
호출 관계만 기록한다.

---

# Business Service

## 20. 다른 Business Service

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

## 21. Service 왕복

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

## 22. Mapper

실제 실행 경로에서 발견된
Mapper 호출만 처리한다.

확보:

Mapper Variable

Mapper Type

Mapper Method

---

## 23. Mapper Type

현재 읽은 Source에서
Mapper Type이 확인되면 그대로 사용한다.

확인되지 않은 경우에만
현재 ServiceImpl 파일 안에서
Mapper Variable을 찾는다.

Repository 전체에서
Mapper Variable을 검색하지 않는다.

---

## 24. Mapper.java

Mapper Java Interface는
기본적으로 읽지 않는다.

가능하면:

ServiceImpl
→ Mapper Type
→ Mapper Method
→ MyBatis XML

로 바로 이동한다.

---

# MyBatis

## 25. MyBatis 목표

MyBatis에서는 오직:

XML 파일 위치

+

Statement ID

만 확인한다.

---

## 26. XML 탐색

Mapper Type으로
정확한 namespace를 찾는다.

검색 범위:

CURRENT_PROJECT/src/main/resources/**

예:

namespace="...MaterialMapper"

---

## 27. XML 확정

정확한 Mapper XML이 발견되면
다른 XML 탐색을 즉시 중단한다.

동일 Mapper Type의 XML은
다시 검색하지 않는다.

---

## 28. Statement ID

확정된 XML 안에서만
Mapper Method와 동일한 Statement ID를 찾는다.

예:

Mapper Method:

selectMaterial

이면:

id="selectMaterial"

을 확인한다.

---

## 29. Statement Body 금지

다음이 확인되면:

<select id="selectMaterial">

또는:

<insert id="insertMaterial">

또는:

<update id="updateMaterial">

또는:

<delete id="deleteMaterial">

Statement가 존재한다고 판단한다.

여기서 MyBatis 분석을 종료한다.

Statement Body를 Read하지 않는다.

---

## 30. XML 전체 Read 금지

MyBatis XML 전체를 읽지 않는다.

Statement 주변 Read도 하지 않는다.

Grep 결과로:

XML 위치

+

Statement ID

가 확인되면 충분하다.

---

# Response

## 31. Response 분석 목적

실제 Backend 실행 결과가
Controller까지 어떻게 반환되는지 확인한다.

Response를 분석하기 위해
새로운 Source 탐색 범위를 만들지 않는다.

이미 읽고 있는 실행 경로를 사용한다.

---

## 32. Service Return

Service / ServiceImpl / Local Method에서
실제로 확인되는 Return을 기록한다.

예:

return result;

return response;

return list;

return count;

return null;

void

---

## 33. Return 전달

다음과 같은 실제 흐름이 있으면 기록한다.

ServiceImpl

return result

↓

Controller

result = service.method(...)

↓

Controller Response

return result

---

## 34. Controller Response

Controller에서 실제 확인되는
Response 형태를 기록한다.

예:

return result;

return ResponseEntity.ok(result);

return response;

void

Model / Map / DTO 반환

실제 Source에 보이는 형태만 기록한다.

---

## 35. Response DTO

Response Type이
현재 읽은 Source에서 명확하면 이름만 기록한다.

예:

MaterialResponse

List<MaterialResponse>

Map<String, Object>

ResponseEntity<MaterialResponse>

DTO 내부 필드 분석을 위해
별도 Source로 이동하지 않는다.

이번 버전에서는
Response DTO 상세 분석을 하지 않는다.

---

# Exception

## 36. Exception 분석 목적

실제 실행 경로에서
직접 확인되는 예외 흐름만 기록한다.

Exception을 찾기 위해
Repository 전체를 검색하지 않는다.

---

## 37. 직접 Throw

현재 읽은 Method에서:

throw

가 실제 확인되면 기록한다.

예:

if (material == null) {
    throw new BusinessException(...);
}

기록:

Condition:
material == null

Exception:
BusinessException

---

## 38. Catch

현재 실행 경로에서:

try / catch

가 실제 확인되면
해당 처리만 기록한다.

예:

catch (Exception e) {
    throw new BusinessException(...);
}

기록:

Catch:
Exception

Throw:
BusinessException

---

## 39. Exception Class 내부 금지

BusinessException

CustomException

RuntimeException

등이 보여도
Exception Class 정의를 찾아가지 않는다.

현재 Source에서 확인되는 정보만 사용한다.

---

## 40. Global Exception Handler

이번 버전에서는:

@ControllerAdvice

@ExceptionHandler

GlobalExceptionHandler

등을 별도로 찾지 않는다.

실제 실행 경로 밖의
전역 예외 처리는 분석하지 않는다.

---

## 41. Validation과 Exception

실제 조건과 Throw가 연결되어 있으면
간단히 기록한다.

예:

materialId == null

→ BusinessException

별도의 Validation 분석을 위해
추가 Source를 찾지 않는다.

---

# External

## 42. SAP / RFC / External

실제 실행 경로에서
외부 호출이 발견되면 기록한다.

예:

sapService.send(...)

rfcClient.execute(...)

externalClient.call(...)

이번 버전에서는
호출 이름까지만 기록한다.

외부 연동 내부 Source로
추가 이동하지 않는다.

---

# Evidence

## 43. Evidence

분석하면서 이미 확인한 Source 위치를 사용한다.

Evidence를 만들기 위해
추가 Grep 또는 Read를 하지 않는다.

형식:

프로젝트 루트 기준 상대경로:라인범위

절대경로는 출력하지 않는다.

---

# Output

## 44. 결과

파일을 생성하지 않는다.

화면에만 출력한다.

형식:

=== BE SIMPLE TEST v0.3 ===

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

MyBatis
{XML_PATH}#{STATEMENT_ID}

↓

{RETURN}

↓

{CONTROLLER_RESPONSE}


CONTROLLER

Class:
{CONTROLLER_CLASS}

Method:
{CONTROLLER_METHOD}

Response:
{RESPONSE}

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

Return:
{RETURN}

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

Return:
{RETURN}

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

Return:
{RETURN}

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

Statement ID:
{STATEMENT_ID}

SQL:
NOT ANALYZED

Evidence:
{PATH:LINE}


RESPONSE

Service Return:
{RETURN}

Controller Response:
{RESPONSE}

Response Type:
{TYPE}

Evidence:
{PATH:LINES}


EXCEPTION

실제 확인된 경우만 출력한다.

1.

Method:
{METHOD}

Condition:
{CONDITION}

Throw:
{EXCEPTION}

Evidence:
{PATH:LINES}

확인된 Exception이 없으면:

NONE


EXTERNAL CALLS

실제 확인된 경우만 출력한다.

- {CALL}

없으면:

NONE


STATUS

COMPLETED

---

# Failure

## 45. 탐색 실패

특정 Source를 찾지 못해도
검색 범위를 무작정 확장하지 않는다.

필요하면 기록한다.

CONTROLLER_NOT_FOUND

SERVICE_NOT_FOUND

SERVICE_IMPL_NOT_FOUND

MAPPER_TYPE_NOT_FOUND

MAPPER_XML_NOT_FOUND

STATEMENT_NOT_FOUND

---

# STOP

## 46. 종료

실제 실행 흐름에 대해:

Controller

Service

Local/private Method

Business Service

Mapper

MyBatis XML 위치

Statement ID

Response

Exception

확인이 끝나면 즉시 종료한다.

SQL을 읽지 않는다.

Statement Body를 읽지 않는다.

include를 따라가지 않는다.

resultMap을 따라가지 않는다.

Exception Class를 찾아가지 않는다.

Global Exception Handler를 찾지 않는다.

추가 후보 Source를 탐색하지 않는다.

BE-REFERENCE를 읽지 않는다.

문서를 생성하지 않는다.

다른 Skill을 실행하지 않는다.