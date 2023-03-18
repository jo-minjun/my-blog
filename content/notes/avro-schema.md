---
title: "Avro schema"
author: ["조민준"]
date: 2022-07-02T18:12:21+09:00
draft: false
categories: ["DevOps"]
tags: ["MSA", "Kafka"]
ShowToc: true
---

## 1. Avro 란?

아브로(Avro)는 아파치의 하둡 프로젝트에서 개발된 RPC 및 데이터 직렬화 프레임워크이다.  
schema를 json으로 정의하여 바이너리 포맷으로 직렬화 한다.

## 2. 장점

1. 데이터의 타입을 알 수 있다.
2. 스키마가 직렬화되어 네트워크 통신에 자유롭다.
3. 스키마에 설명이 포함되어 schema 구조를 이해하는데 도움을 준다.
4. 다양한 language를 지원한다. (java, c, c++ 등)
5. default 값을 정의할 수 있다.

## 3. Data Type

| 이름      | 태그                                                   |
| --------- | ------------------------------------------------------ |
| `null`    | no value                                               |
| `boolean` | a binary value                                         |
| `int`     | 32bit signed integer                                   |
| `long`    | 62bit signed integer                                   |
| `float`   | single precision(단정밀도) 32bit floating-point number |
| `double`  | double percision(배정밀도) 64bit floating-point number |
| `bytes`   | sequence of 8-bit unsigned bytes                       |
| `string`  | unicode character sequence                             |

### Enums

name, namespace, aliases, doc, symbols, default 등을 가진다.

```json
{
  "type": "enum",
  "name": "Suit",
  "symbols": ["SPADES", "HEARTS", "DIAMONDS", "CLUBS"]
}
```

### Arrays

items를 가진다.

```json
{
  "type": "array",
  "items": "string",
  "default": []
}
```

### Maps

values를 가진다.

```json
{
  "type": "map",
  "values": "long",
  "default": {}
}
```

### Unions

string, int, boolean 등과 같은 여러개의 서로 다른 타입을 가짐으로써 선택적인 값을 저장할 수 있도록 한다.

## 4. Schema 필드

| 이름      | 태그                                                                  |
| --------- | --------------------------------------------------------------------- |
| name      | json을 지원하는 스키마의 이름                                         |
| namespace | name을 구별하는 패키지                                                |
| doc       | 스키마를 설명하는 doc                                                 |
| aliases   | name의 별칭                                                           |
| fields    | name, doc, type, default, order, aliases 등을 가지는 json type object |

## 5. Avro Schema 예시

```json
{
  "type": "record",
  "name": "LongList",
  "aliases": ["LinkedLongs"], // old name for this
  "fields": [
    { "name": "value", "type": "long" }, // each element has a long
    { "name": "next", "type": ["null", "LongList"] } // optional next element
  ]
}
```

```json
{
   "type" : "record",
   "namespace" : "tutorialspoint",
   "name" : "empdetails ",
   "fields" :
   [
      { "name" : "experience", "type": ["int", "null"] },
      { "name" : "age", "type": "int" }
      {"name": "additional", "type": {"type": "map", "values": "string"}}
   ]
}
```
