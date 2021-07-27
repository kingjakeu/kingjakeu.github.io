---
title: "레디스(Redis) 자료 구조"
date: 2021-07-27 14:00:00 +0900
category: study
lastmod : 2021-07-27 14:00:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

레디스(Redis)는 단순 `Key-Value` 저장소가 아닌, 다양한 종류의 Value을 지원하는 자료 구조 서버이다.  

기존의 Key-Value 저장소는 `String key`와 `String value`가 결합하지만, 레디스에서는 String 뿐만아니라 **더 복잡한 자료 구조**도 지원한다.

- [Redis Strings](#redis-strings)
- [Redis Lists](#redis-lists)
- [Redis Hahses](#redis-hahses)
- [Redis Sets](#redis-sets)
- [Redis Sorted sets](#redis-sorted-sets)
- [Bitmaps](#bitmaps)
- [HyperLogLogs](#hyperloglogs)
  
## 레디스 키(Key)의 특징

레디스에서 키(Key) Binary-Safe하다. 죽, String 부터 JPEG파일 까지 `Byte sequence`로 되어 있다면 모두 키로 활용 가능 하다.

- 매우 긴 키는 좋지 않다. 1024 Byte의 Key는 메모리 측면 뿐만 아니라 Key를 비교하며 찾는 데도 좋지 않다. 큰 값의 존재를 확인 하는 작업이더라도 해싱을 하는 것이 더 좋다.
- 매우 짧은 키 또한 좋지 않다. 짧은 키는 약간의 메모리를 아껴 줄수는 있지만, 관리하고 읽기 좋은 키로 지정하는 것이 좋다.
- 스키마와 함께 지정해 보자. `"Object-type:id"` 형태로 키를 지정하고 `.`, `-`를 이용 하여 단어 수를 늘릴 수 있다.
- Key size의 최대 값은 512MB 이다.

## Redis Strings

- String type은 가장 단순한 자료 형이다. 레디스 키가 String인 점을 이용, `string-string`매핑을 이용하여 연결되는 자료 매핑을 할 수도 있다. HTML 매핑도 가능
- `SET`을 통해 값을 저장할 때, 해당 키에 저장 된 값을 갱신한다. 해당 키에 저장된 값이 String이 아니더라도 String으로 덮어 써버린다.
- 값은 최대 512 MB이며, String으로 될 수 있는 binary data도, JPEG 이미지도 저장 가능하다.

## Redis Lists

- 레디스의 List는 `Linked List` 형태로 구현되어 있다.
- Linked List의 특성상, Head or Tail에 값을 추가 시에 **Constant time**으로 빠르다.
- 하지만 인덱스로 리스트 내의 값을 찾을 경우, **N의 시간**, 즉, 추가할 때보다 더 긴 시간이 필요하다.

### List의 활용

- SNS에서의 회원의 최근 업로드된 글 저장
- Consumer - producer 패턴에서, producer가 항목들을 리스트에 집어 넣고, Consumer가 실행 할 때

### List의 특징

- 이미 다른 데이터 타입으로 선언된 key로 add 불가능하다.
- 스트림을 제외한, 비어 있는 key는 자동으로 없어진다.

## Redis Hahses

- field-value로 구성 되어있는 전형적인 `hash`의 형태
- 메모리가 허용하는 한, 제한없이 field들을 넣을 수가 있다.

## Redis Sets

- 정렬되지 않은 String collection
- Set을 활용한 element 존재 여부, `intersection`, `union`, `difference` 연산도 가능하다.
- `SPOP` 커맨드를 통해 랜덤한 element 추출 또한 가능 하다.
  
## Redis Sorted sets

- Set과 Hash가 섞인 데이터 종류
- Set처럼 unique, element 중복 방지가 된다.
- 하지만 Set과 다르게 `score`을 활용하여, element를 정렬하여 저장한다. score가 같을 경우 `lexicographically` 정렬된다.

## Bitmaps

- Bitmap이 data type으로 정의 되어 있는것은 아니지만, String타입에 **비트 연산이 가능**하다.
- String이 512MB 저장 할 수 있듯이 2^32 bit까지 사용 가능하다.
- 저장할 때, **저장 공간 절약에 큰 장점**이 있다.

## HyperLogLogs

- 확률적 자료 구조로 유니크한 항목들의 개수를 세는데 사용한다.
- 일반적으로 유니크한 항목들의 개수를 셀 때에는 count할 만킁의 메모리가 필요하지만, **HLL에서는 constant 메모리만 사용**한다.
  
---

### Reference

- [https://redis.io/topics/data-types-intro](https://redis.io/topics/data-types-intro)
