---
title: "Schema registry란?"
author: ["조민준"]
date: 2022-07-01T21:42:30+09:00
draft: false
categories: ["DevOps"]
tags: ["MSA", "Kafka"]
ShowToc: true
---

## 1. Schema registry 란?

데이터 관리의 중요한 관점들 중 하나는 schema의 버전 관리이다.
응용프로그램의 시간이 지날수록 schema가 정의되기 시작한 시점부터 schema는 점점 바뀌어가고,
producer와 consumer는 직접적인 관계가 끊어져있기 때문에 운영상에 발생하는 이슈가 있다.

> producer는 consumer가 어떤 메세지를 소비할지 알 수 없다.  
> consumer는 producer가 어떤 메세지를 생산했는지 알 수 없다.

위와 같은 상황에서 producer가 갑자기 다른 schema를 이용해서 메세지를 생산할 경우, consumer는 이 메세지에 대해서 대처하지 못할 수 있다.

이는 구조적인 결합도는 낮지만, 메세지 schema에 대한 의존성이 높기 때문인데,
schema registry는 이를 보완하기 위해 고안되었다.

Confluent Schema registry는 Avro, Json, Protobuf 등의 schema 정보의 history를 subjects를 통해 관리하며, REST API를 통해 **compatibility settings**을 결정하고 현재 버전과 이전 버전간의 호환성을 지원한다.

![schema-registry-kafka.png](../schema-registry/schema-registry-kafka.png)

Schema registry는 kafka boroker와 독립적으로 존재하며, producer와 consumer는 kafka broker와 읽고 쓰는 동안 Schema registry와 동작하며 데이터 모델을 확인할 수 있다.

## 2. Schemas, Subjects and Topics 란?

topic은 kafka의 topic을, schema는 Avro, Json, Protobuf 등으로 정의된 데이터 포맷 구조를 의미한다.

Subject는 Schema registry에 schema가 등록된 이름이며, 여러 버전의 schema가 등록될 수 있다.

따라서 Subject를 통해 계속해서 Schema의 정보를 관리할 수 있고, 새로운 버전의 Schema ID와 버전을 확인할 수 있다.

![schema registry.png](../schema-registry/schema-subject-topic.png)

- kafka topic은 메세지가 포함되어 있으며, 각 메세지는 key - value 쌍으로 되어있으며
  메세지의 key와 value는 Avro, Json, Protobuf 등으로 직렬화할 수 있다.
- Schema는 데이터 포맷의 구조를 정의한다.
- kafka의 topic 이름은 schema의 이름과 의존적이지 않다.
- Schema의 ID는 전역적이다.

## 3. Compatibility settings 란?

schema compatibility checking는 모든 schema를 버전화해서 schema registry compatibility type에 의해서 구현된다.

즉, 아래의 schema 전략에 의한 패턴으로 호환성을 유지하게 된다.

| Compatibility type  | 허가되는 변경                                 | 비교하는 schema | upgrade 순서 |
| ------------------- | --------------------------------------------- | --------------- | ------------ |
| BACKWARD            | - 필드 삭제</br>- Optional 필드 추가          | 마지막 버전     | Consumers    |
| BACKWARD_TRANSITIVE | - 필드 삭제</br>- Optional 필드 추가          | 모든 이전 버전  | Consumers    |
| FORWARD             | - 필드 추가</br>- Optional 필드 삭제          | 마지막 버전     | Producers    |
| FORWARD_TRANSITIVE  | - 필드 추가</br>- Optional 필드 삭제          | 모든 이전 버전  | Producers    |
| FULL                | - Optional 필드 추가</br>- Optional 필드 삭제 | 마지막 버전     | Any order    |
| FULL_TRANSITIVE     | - Optional 필드 추가</br>- Optional 필드 삭제 | 모든 이전 버전  | Any order    |
| NONE                | 모든 변경 허용                                | 비교하지 않음   | Depends      |

- `BACKWARD`: (_default_) consumer가 새로운 스키마를 사용하여 producer가 마지막 버전의 스키마로 생성한 메세지를 읽을 수 있다.
  (새로운 스키마로 이전 스키마 메세지를 읽는다.)
  (새로운 스키마 필드에 default value가 없으면 오류가 발생한다.)
- `BACKWARD_TRANSITIVE`: consumer가 새로운 스키마를 사용하여 producer가 모든 마지막 버전 스키마로 생성한 메세지를 읽을 수 있다.
- `FORWARD`: consumer가 마지막 버전의 스키마를 사용하여 producer가 새로운 스키마로 생성한 메세지를 읽을 수 있다.
  (이전 스키마로 새로운 스키마 데이터를 읽는다.)
  (새로운 스키마에서 필드가 삭제되면, 이전 스키마에 default value가 있어야 한다.)
- `FORWARD_TRANSITIVE`: consumer가 모든 마지막 버전의 스키마를 사용하여 producer가 새로운 스키마로 생성한 메세지를 읽을 수 있다.
- `FULL`: `BACKWARD` 와 `FORWARD` 를 모두 만족한다.
- `FULL_TRANSITIVE`: `BACKWARD_TRANSITIVE` 와 `FORWARD_TRANSITIVE` 를 모두 만족한다.
- `NONE`: schema compatibility checks are disabled

## 4. Rest API Interface Reference - Schemas, Subjects

Schema Registry REST API 서버는 사용된 API의 버전과 데이터의 직렬화 포맷을 표시하기 위해 요청과 응답에 content type을 사용한다.

현재는 직렬화 포맷은 JSON만을 지원하고 API의 버전은 v1만 사용할 수 있다.
하지만 나중 버전의 호환을 위해서 반드시 content type을 사용해야한다.

추천하는 content type은 `application/vnd.schemaregistry.v1+json`이다.
v1은 API의 버전이고, json은 직렬화 포맷이다.

모든 API endpoint는 다음과 같은 error message 포맷을 사용한다.

```json
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/vnd.schemaregistry.v1+json

{
    "error_code": 422,
    "message": "schema may not be empty"
}
```

> 추가적으로 요청시 json은 string 형태로 전달해야 한다.

### Schemas 관련

#### `GET /schemas/ids/{int: id}`

입력한 id를 이용하여 스키마 정보를 요청한다.

> **Parameters:**  
> id (int) - 전역적으로 unique한 스키마 id

**Response JSON Object:**
schema (string) - id로 구분한 schema string

**Status Codes:**
404 Not Found - schema not found
500 Internal Server Error - Error in the backend datastore

**Example request:**

```bash
GET /schemas/ids/1 HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  "schema": "{\"type\": \"string\"}"
}
```

#### `GET /schemas/types/`

Schema Registry에 저장된 스키마 타입을 요청한다.

> **Response JSON Object:**
> schema (string) - Schema Registry에서 현재 사용가능한 스키마 타입

**Status Codes:**
404 Not Found - schema not found
500 Internal Server Error - Error in the backend datastore

**Example request:**

```bash
GET /schemas/types HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  ["JSON", "PROTOBUF", "AVRO"]
}
```

#### `GET /schemas/ids/{int: id}/versions`

Schema Registry에 저장된 스키마 타입을 요청한다.

> **Parameters:**  
> id (int) - 전역적으로 unique한 스키마 id

**Response JSON Object:**
subject - subject의 이름
version - return된 subject의 버전

**Status Codes:**
404 Not Found - schema not found
500 Internal Server Error - Error in the backend datastore

**Example request:**

```bash
GET /schemas/ids/1/versions HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

[{"subject":"test-subject1","version":1}]
```

### Subjects 관련

> subject resource는 Schema Registry에 저장된 모든 subject 목록을 제공한다.  
> subject는 스키마가 저장된 이름을 나타낸다.  
> 만약 Kafka에 Schema Registry를 사용하고 있다면, subject는 topic에 대한 key 또는 value 스키마를 등록하고 있는지에 따라 `<topic>-key` 또는 `<topic>-value`를 참조한다.

#### `GET /subjects`

Schema Registry에 저장된 subject의 목록을 요청한다.

> **Parameters:**  
> subject (string) - subject의 이름  
> deleted (boolean) - default는 false이다. `?deleted=true`로 요청하면 soft delete된 subject 목록을 함께 return 한다.

**Response JSON Object:**
name (string) - subject

**Status Codes:**
500 Internal Server Error - Error in the backend datastore

**Example request:**

```bash
GET /subjects HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

["subject1", "subject2"]
```

#### `GET /subjects/(string: subject)/versions`

Schema Registry에 저장된 subject의 버전 목록을 요청한다.

> **Parameters:**  
> subject (string) - subject의 이름

**Response JSON Object:**
version (int) - subject 아래에 저장된 스키마의 버전

**Status Codes:**
404 Not Found - schema not found
500 Internal Server Error - Error in the backend datastore

**Example request:**

```bash
GET /subjects/test/versions HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

[
  1, 2, 3, 4
]
```

#### `DELETE /subjects/(string: subject)`

등록된 특정 subject를 삭제한다.
이 API는 topic을 재사용하거나 개발 환경에서만 사용하는 것이 권장된다.

> **Parameters:**  
> subject (string) - subject의 이름  
> permanent (boolean) - `?permanent=true`를 추가하여 hard delete를 표시한다.

**Response JSON Object:**
version (int) - subject 아래에 저장된 스키마의 버전

**Status Codes:**
404 Not Found - schema not found
500 Internal Server Error - Error in the backend datastore

**Example request:**

```bash
DELETE /subjects/test HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

[
  1, 2, 3, 4
]
```

#### `GET /subjects/(string: subject)/versions/(versionId: version)`

등록된 특정 subject를 요청한다.

> **Parameters:**  
> subject (string) - subject의 이름  
> version (versionId) - return될 스키마의 버전이다. [1, 2^31 - 1] 또는 latest가 유효한 값이다.

**Response JSON Object:**
subject (string) - subject의 이름
id (int) - 전역적으로 unique한 shema의 id
version (int) - subject 아래에 저장된 return될 스키마의 버전
schemaType (string) - schema의 format (default: AVRO)
schema (string) - schema의 내용

**Status Codes:**
404 Not Found - schema not found
422 Unprocessable Entity - Invalid version
500 Internal Server Error - Error in the backend datastore

**Example request:**

```bash
GET /subjects/test/versions/1 HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  "name": "test",
  "version": 1,
  "schema": "{\"type\": \"string\"}"
}
```

#### `GET /subjects/(string: subject)/versions/(versionId: version)/schema`

등록된 특정 subject를 요청한다. unescaped schema만 return 된다.?

> **Parameters:**  
> subject (string) - subject의 이름  
> version (versionId) - return될 스키마의 버전이다. [1, 2^31 - 1] 또는 latest가 유효한 값이다.

**Response JSON Object:**
schema (string) - schema의 내용

**Status Codes:**
404 Not Found - subject not found or version not found
422 Unprocessable Entity - Invalid version
500 Internal Server Error - Error in the backend datastore

>

**Example request:**

```bash
GET /subjects/test/versions/1/schema HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{"type": "string"}
```

#### `POST /subjects/(string: subject)/versions`

등록된 특정 subject에 새로운 스키마를 등록한다.
만약 성공적으로 등록되면, unique한 스키마의 id가 return 된다.
동일한 스키마가 다른 subject에 등록되면 동일한 id가 return 된다. 그라나 스키마의 버전은 subject에 따라 다를 수 있다.

> **Parameters:**  
> subject (string) - subject의 이름  
> normalize (boolean) - `?normalize=true`를 추가하여 normalize 상태를 표시한다.?

**Request JSON Object:**
schema (string) - schema의 내용
schemaType - 스키마의 포맷 (default: AVRO)
references - 스키마의 이름 지정 (optional)

**Response JSON Object:**
subject (string) - subject의 이름
id (int) - 전역적으로 unique한 스키마의 id
version (int) - return 되는 스키마의 버전
schema (string) - schema의 내용

**Status Codes:**
409 Conflic - Incompatible schema
422 Unprocessable Entity - Invalid version
500 Internal Server Error - Error in the backend datastore or Operation timed out or Error while forwarding the request to the primary

**Example request:**

```json
POST /subjects/test/versions HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json

{
  "schema":
    "{
       \"type\": \"record\",
       \"name\": \"test\",
       \"fields\":
         [
           {
             \"type\": \"string\",
             \"name\": \"field1\"
           },
           {
             \"type\": \"com.acme.Referenced\",
             \"name\": \"int\"
           }
          ]
     }",
  "schemaType": "AVRO",
  "references": [
    {
       "name": "com.acme.Referenced",
       "subject":  "childSubject",
       "version": 1
    }
  ]
}
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{"id":1}
```

#### `POST /subjects/(string: subject)`

특정 subject에 schema가 이미 등록됐는지 확인한다.
만약 존재하면 전역적으로 unique한 id와 schema를 return 한다.

> **Parameters:**  
> subject (string) - subject의 이름  
> normalize (boolean) - `?normalize=true`를 추가하여 normalize 상태를 표시한다.

**Request JSON Object:**
schema (string) - schema의 내용
schemaType - 스키마의 포맷 (default: AVRO)
references - 스키마의 이름 지정 (optional)

**Response JSON Object:**
subject (string) - subject의 이름
id (int) - 전역적으로 unique한 스키마의 id
version (int) - return 되는 스키마의 버전
schema (string) - schema의 내용

**Status Codes:**
404 Not Found - Subject not found
500 Internal Server Error - Internal server error

**Example request:**

```json
POST /subjects/test HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json

{
      "schema":
         "{
                \"type\": \"record\",
                \"name\": \"test\",
                \"fields\":
                  [
                    {
                      \"type\": \"string\",
                      \"name\": \"field1\"
                    },
                    {
                      \"type\": \"int\",
                      \"name\": \"field2\"
                    }
                  ]
              }"
    }
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
      "subject": "test",
      "id": 1
      "version": 3
      "schema":
         "{
                \"type\": \"record\",
                \"name\": \"test\",
                \"fields\":
                  [
                    {
                      \"type\": \"string\",
                      \"name\": \"field1\"
                    },
                    {
                      \"type\": \"int\",
                      \"name\": \"field2\"
                    }
                  ]
              }"
    }
```

#### `DELETE /subjects/(string: subject)/versions/(versionId: version)`

특정 subject에 등록된 schema의 버전을 삭제한다.
이 API는 호환성 목적으로 이전에 등록한 스키마를 삭제하거나 이전에 등록한 스키마를 다시 등록해야 하는 개발 환경이나 극단적인 상황에서만 사용하는 것이 좋다.

> **Parameters:**  
> subject (string) - subject의 이름  
> version (versionId) - 삭제될 schema 버전을 표시한다. [1, 2^31 - 1] 또는 latest가 될 수 있다.  
> permanent (boolean) - `?permanent=true`를 추가하여 hard delete를 표시한다.

**Response JSON Object:**
int - 삭제된 schema의 버전

**Status Codes:**
404 Not Found - Subject not found or Version not found
422 Unprocessable Entity - Invalid version
500 Internal Server Error - Error in the backend data store

**Example request:**

```bash
DELETE /subjects/test/versions/1 HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```bash
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

1
```

#### `GET /subjects/(string: subject)/versions/{versionId: version}/referencedby`

주어진 subject와 버전에 대한 schema의 id의 목록을 요청한다.

> **Parameters:**  
> subject (string) - subject의 이름  
> version (versionId) - return 되는 schema 버전을 표시한다. [1, 2^31 - 1] 또는 latest가 될 수 있다.

**Request JSON Array of Objects:**
id (int) - 전역적으로 unique한 스키마의 id

**Status Codes:**
404 Not Found - Subject not found
500 Internal Server Error - Error in the backend data store

**Example request:**

```bash
GET /subjects/test/versions/1/referencedby HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

[
  1, 2, 3, 4
]
```

## 5. REST API Interface Reference - Compatibility, Config

### Compatibility

compatibility resource는 모든 버전 또는 특정 버전에 대해 사용자가 스키마의 호환성을 검사할 수 있도록 한다.

#### `POST /compatibility/subjects/(string: subject)/versions/(versionId: version)`

input 스키마에 대해서 특정 스키마의 버전에 대한 호환성을 검사한다.

> **Parameters:**
> subject (string) - 호환성을 테스트할 subject의 이름  
> version (versionId) - 호환성을 테스트할 대상의 schema 버전을 표시한다. [1, 2^31 - 1] 또는 latest가 될 수 있다.  
> verbose (boolean) - `?verbose=true`를 추가하여 호환성 테스트에 실패하는 이유를 출력한다.

**Request JSON Object:**
schema - 스키마의 내용
schemaType - 스키마의 포맷 (default: AVRO)
references - 참조된 스키마의 이름을 지정한다. (optional)

**Response JSON Object:**
is_compatible (boolean) - 호환가능 여부

---

**Status Codes:**
404 Not Found - Subject not found or Version not found
422 Unprocessable Entity - Invalid schema or Invalid version
500 Internal Server Error - Error in the backend data store

**Example request:**

```json
POST /compatibility/subjects/test/versions/latest HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json

{
  "schema":
    "{
       \"type\": \"record\",
       \"name\": \"test\",
       \"fields\":
         [
           {
             \"type\": \"string\",
             \"name\": \"field1\"
           },
           {
             \"type\": \"int\",
             \"name\": \"field2\"
           }
         ]
     }"
}
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  "is_compatible": true
}
```

#### `POST /compatibility/subjects/(string: subject)/versions`

호환성 전략에 따라 subject의 하나의 버전 또는 여러 버전의 호환성을 확인한다.

> **Parameters:**
> subject (string) - 호환성을 테스트할 subject의 이름
> verbose (boolean) - **`?verbose=true`**를 추가하여 호환성 테스트에 실패하는 이유를 출력한다.

**Request JSON Object:**
schema - 스키마의 내용
schemaType - 스키마의 포맷 (default: AVRO)
references - 참조된 스키마의 이름을 지정한다. (optional)

**Response JSON Object:**
is_compatible (boolean) - 호환가능 여부

---

**Status Codes:**
404 Not Found - Subject not found
422 Unprocessable Entity - Invalid schema
500 Internal Server Error - Error in the backend data store

**Example request:**

```json
POST /compatibility/subjects/test/versions
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json

{
  "schema":
    "{
       \"type\": \"record\",
       \"name\": \"test\",
       \"fields\":
         [
           {
             \"type\": \"string\",
             \"name\": \"field1\"
           },
           {
             \"type\": \"int\",
             \"name\": \"field2\"
           }
         ]
     }"
}
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  "is_compatible": true
}
```

### Config

#### `PUT /config`

전역적 호환성 전략을 변경한다.

> **Request JSON Object:**  
> compatibility (string) - 새롭게 변경된 호환성 전략을 표시한다.

---

**Status Codes:**
422 Unprocessable Entity - Invalid compatibility level
500 Internal Server Error - Error in the backend data store or Error while forwarding request to the primary

**Example request:**

```json
PUT /config HTTP/1.1
Host: kafkaproxy.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json

{
  "compatibility": "FULL"
}
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  "compatibility": "FULL"
}
```

#### `**GET /config**`

전역적 호환성 전략을 요청한다.

> **Response JSON Object:**  
> compatibility (string) - 현재 호환성 전략을 표시한다.

---

**Status Codes:**
500 Internal Server Error - Error in the backend data store

**Example request:**

```bash
GET /config HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  "compatibilityLevel": "FULL"
}
```

#### `PUT /config/(string: subject)`

특정 subject에 대한 호환성 전략을 변경한다.

> **Parameters:**
> subject (string) - subject의 이름

**Request JSON Object:**
compatibility (string) - 새롭게 변경된 호환성 전략을 표시한다.

---

**Status Codes:**
422 Unprocessable Entity - Invalid compatibility level
500 Internal Server Error - Error in the backend data store or Error while forwarding request to the primary

**Example request:**

```json
PUT /config/test HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json

{
  "compatibility": "FULL"
}
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  "compatibility": "FULL"
}
```

#### `GET /config/(string: subject)`

특정 subject에 대한 호환성 전략을 요청한다.

> **Parameters:**  
> subject (string) - subject의 이름  
> defaultToGlobal (boolean) - **`?defaultToBlobal=false`**를 추가하여 호환성을 표시한다.  
> **`?defaultToBlobal=true`**를 추가하면 호환성 검사에 필요한 사항을 표시한다.

**Response JSON Object:**
compatibility (string) - 현재 호환성 전략을 표시한다.

---

**Status Codes:**
404 Not Found - Subject not found
500 Internal Server Error - Error in the backend data store

**Example request:**

```bash
GET /config/test HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
   "compatibilityLevel": "FULL"
}
```

#### `DELETE /config/(string: subject)`

특정 subject에 대한 호환성 전략을 삭제하고, 전역적 호환성 전략을 사용한다.

> **Parameters:**  
> subject (string) - subject의 이름

---

**Status Codes:**
404 Not Found - Subject not found
500 Internal Server Error - Error in the backend data store

**Example request:**

```bash
DELETE /config/test HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

**Example response:**

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
   "compatibility": "NONE"
}
```

## 6. Subject Name Strategy

subject 이름을 만들어 내기 위한 subject naming strategy가 있다.

| Strategy                         | 설명                                                                                                                                                                                                                                                                                     |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TopicNameStrategy</br> (default) | - topic 이름으로부터 subject의 이름을 만든다.</br>- 항상 하나의 topic에 있는 메세지는 같은 schema를 가지는 것을 보장한다.                                                                                                                                                                |
| RecordNameStrategy               | - record 이름으로부터 subject의 이름을 만들고, subject 아래에 서로 다른 schema를 가질 수 있는 논리적으로 관련된 그룹화를 제공한다.</br>- 하나의 topic에 여러개의 schema를 가지는 것을 허용한다.</br>- 이 전략은 메세지가 서로 다른 데이터 구조를 가질때 유용하게 사용할 수 있다.         |
| TopicRecordNameStrategy          | - topic과 record 이름으로부터 subject의 이름을 만들고, subject 아래에 서로 다른 schema를 가질 수 있는 논리적으로 관련된 그룹화를 제공한다.</br>- 하나의 topic에 여러개의 schema를 가지는 것을 허용한다.</br>- 이 전략은 메세지가 서로 다른 데이터 구조를 가질때 유용하게 사용할 수 있다. |

<참고>

[https://docs.confluent.io/platform/current/schema-registry/serdes-develop/index.html#](https://docs.confluent.io/platform/current/schema-registry/serdes-develop/index.html#)
