![](https://github.com/middle-ware-house/redis/assets/89020004/4155d5a2-47d6-4b8e-bcd2-bdb73c748634)

전체적인 자료구조 개요

# Redis의 자료구조

## string [”key” → “value”]

### 특징

- 키와 실제 데이터가 1:1 로 연결되는 유일한 자료구조

### value에 저장 가능한 데이터

- 최대 512MB의 문자열 데이터 저장 가능
- binary-safe 하기에 JPEG 이미지, 바이트 값, HTTP 응답값 등 또한 저장 가능
- 숫자 또한 저장 가능
    - `INCR`(1 증가), `INCRBY`(지정하는 숫자만큼 덧셈 가능)
    - 비슷한 원리로 `DECR`, `DECRBY` 도 가능
    - 숫자를 원자적으로 조작 가능 → 여러 클라이언트가 경쟁 상태를 발생시키지 않음 (중복 처리 없음)

### 기본 사용

- `SET k v` → O(1)
    - k 라는 key에 v 라는 value 저장
- `GET k` -> O(1)
    - k 라는 key에 해당하는 value 가져옴
- 숫자의 경우
    - `INCR k` , `DECR k`
    - `INCRBY k 30` , `DECRBY k 30`
        - key가 k인 숫자value 30 증가, 감소
    - `MGET k1 k2 k3`
        - k1, k2, k3 에 해당하는 value 가져옴
    - `MSET k1 10 k2 20 k3 30`
        - k1 = 10, k2 = 20, k3 = 30 으로 값 지정

### 활용 예시

- 다른 자료구조를 사용하고 싶은데 해당 자료구조에는 어려 데이터를 못 담을때
    - Java 기반 어플리케이션에서 Object를 json으로 문자열화 하여 저장 가능
- string 여러 값을 한 번에 가져오고 싶은 경우
    - `MSET`, `MGET` 명령어로 한 번에 여러 value 가져옴

## list [”key” → [”v1”, “v2”, “v3”, “v4”]]

### 특징

- LinkedList 기반이지만 index로 접근 가능
    - 레디스의 주요 list 관련 환경 변수
        - list-max-ziplist-entries 512 : 엔트리 수가 512 이하면 짚 리스트, 512보다 많으면 LinkedList
        - list-max-ziplist-value 64 : 값의 크기(바이트)가 64 이하면 짚 리스트, 64보다 크면 LinkedList
    - 위 두 조건 중 하나라도 만족하면 LinkedList로 전환하여 저장
        - ziplist는 메모리를 절약할 수 있는 자료구조
- 하나의 list에 42억여 개의 아이템 저장 가능
- 인덱스 접근 가능
- 스택과 큐로 사용 가능 (MQ의 queue와 같이 사용 가능)
- 맨 왼쪽 = head / 맨 오른쪽 = tail
    - 위 둘이 모두 있기에 순방향, 역방향 탐색 모두 가능

### value에 저장 가능한 데이터

- string의 묶음 자료구조이므로 원소 하나하나는 string의 value와 같음

### 기본 사용

- L을 R로 바꾸면 왼쪽부터가 아닌 오른쪽부터 연산을 시도한다.
- `LPUSH listkey element` O(1)
- `LRANGE 0 -1` O(1)
    - 0 ~ -1 인덱스의 요소를 가져옴
        - 즉, 가장 좌측부터 처음부터 끝까지 다 가져오게 됨
- `LPOP listkey` O(1)
    - 가장 왼쪽 요소 가져오고 list에서 삭제
- `LTRIM listkey 0 1` O(1)
    - 지정한 인덱스 범위 밖의 모든 요소 삭제 (0 ~ 1 번째 인덱스 제외 모두 삭제)
- `LINSERT listkey BEFORE B E` O(n)
    - B앞에 E를 저장 (B는 요소 값 중 하나)
- `LSET listkey 1 F`
    - 인덱스 1의 값을 F로 수정
- `LINDEX listkey F`
    - F의 인덱스 조회

### 활용 예시

- 리스트에 최대 개수를 지정해 저장하고 싶은 경우
    - LPUSH 이후 LTRIM 으로 첫 인덱스부터 최대 개수 인덱스까지 명령을 수행하면 된다.
    - 깃허브의 기여도 곡선 그래프
    
    ![image](https://github.com/middle-ware-house/redis/assets/89020004/36519109-024a-4b33-aafc-434cf4e8a452)
    

## hash [”key” → {”innerkey” → “value”}]

### 특징

- 동적으로 다양한 필드 추가 가능

### value에 저장가능한 데이터

- innerkey, value 모두 문자열로 저장 (innerkey는 임의로 이해를 돕기 위해 지정한 용어다.)

### 기본 사용

- `HSET key innerkey value` → O(1)
- `HGET key innerkey` → O(1)
- `HMGET key innerkey1 innerkey2`
- `HGETALL key`
    - 결과 예시 (다음과 같이 innerkey, value를 순서대로 줄을 나누어 가져올 수 있다.)
        - innerkey1
        - value1
        - innerkey2
        - value2

### 활용 예시

- 객체를 표현하기에 적합하여 관계형 데이터베이스의 테이블 데이터로 변환해야할때
- 하나의 키에 해당하는 여러 데이터 저장해야 할때 (innerkey가 동적일때 포함)

## set [”key” → [”A”, “D”, “C”, “B”]] (순서 없음)

### 특징

- 중복 허용하지 않으며 순서 없이 여러 데이터 저장
- 교집합, 합집합, 차집합 등의 집한 연산 커맨드 제공

### value에 저장가능한 데이터

- 문자열 여러개

### 기본 사용

- `SADD setkey A`
- `SADD setkey A A B B C D`
    - set이 비어있다는 가정하에 4개만 저장됨
- `SMEMBERS setkey`
    - 전체 아이템 가져옴
- `SREM setkey A`
    - A 삭제
- `SPOP setkey`
    - 랜덤으로 하나의 아이템 가져오고 set에서 삭제
- `SINTER setkey1 setkey2`
- `SUNION setkey1 setkey2`
- `SDIFF setkey1 setkey2`

### 활용 예시

- 중복 없이 저장하고 싶을때
    - 참고) 중복 없는 개수 세는 것은 hyperloglog 자료구조도 고려해야함

## sorted set [”key” → {”memberA” ↔ 100, “memberB ↔ 150”}]

### 특징

- 스코어에 의해 “오름차순 정렬”됨
    - 내림차순 정렬로 조회도 가능
- 같은 스코어의 경우 데이터(member)의 사전순으로 정렬
- 인덱스를 통해 접근하기에 용이
    - list는 LinkedList 기반이기 때문에 O(n) 이지만, sorted set의 경우 O(log n) 이다.
- zset 이라고도 불림
    - 따라서 명령어 맨 앞이 Z 로 시작 (S면 set이랑 겹침)
- 초기에는 zip list를 사용하며, 문자열 데이터가 하나라도 65kb 이상이거나 데이터 개수가 128개를 초과하게 되면 skip list 사용

### value에 저장 가능한 데이터

- string과 score(숫자) 를 하나의 쌍으로 여러 쌍 저장

### 기본 사용

- `ZADD zsetkey 100 memberA 150 memberB`
    - 옵션 목록
        - XX: 이미 존재한다면 스코어 업데이트
        - NX: 없다면 삽입, 있다면 업데이트는 하지 않음
        - LT: 업데이트하고자 하는 숫자가 기존 스코어보다 작을 때에만 업데이트, 존재하지 않으면 삽입
        - GT: 업데이트하고자 하는 숫자가 기존 스코어보다 클 때에만 업데이트, 존재하지 않으면 삽입
- `ZRANGE zsetkey 1 3`
    - 범위 조회 (1번째 부터 3번째 까지 3개의 데이터 조회)
        - `ZRANGE zsetkey 0 -1` 은 모두 조회한다
    - 옵션
        - WITHSCORES: 숫자 데이터도 같이 조회
- `ZRANGE zsetkey 100 150 BYSCORE`
    - 숫자 데이터를 통한 범위 조회
        - +inf, -inf 를 통해 양수, 음수의 최댓값을 사용할 수 있다.
        - 예를 들면 150 이상의 모든 데이터를 조회하고자 한다면
            - `ZRANGE zsetkey 200 +inf BYSCORE`
    - 옵션
        - WITHSCORES: 숫자 데이터도 같이 조회
        - REV: 순서 거꾸로 조회
- `ZRANGE zsetkey (b (d BYLEX`
    - 사전 순으로 string 데이터에 범위에 해당하는 값 조회
        - 무조건 ( 혹은 [ 를 먼저 적어야 한다
            - ( 를 포함하는 데이터의 경우 [ 사용
        - 가장 처음 문자는 - , 마지막 문자는 + 로 사용 가능
            - `ZRANGE zsetkey - + BYLEX` 는 모두 조회

### 활용 예시

- 랭킹 시스템에 정말 자주 사용된다.
- 하나의 숫자 기준을 통해 정렬하여 저장하고 싶을때

## bitmap [”key” → 00100010 00100010]

### 특징

- string 자료구조에 bit 연산을 수행할 수 있도록 확장된 형태
- 2^32 의 비트를 가지고 있는 비트맵
- 저장공간을 획기적으로 줄일 수 있음
    - 40억건의 유저의 y/n 와 같은 간단한 데이터를 512MB 안에 저장 가능

### value에 저장 가능한 데이터

- 1, 0 와 같이 비트를 저장

### 기본 사용

- `SETBIT bitmapkey 2 1`
- `GETBIT bitmapkey 2`
- `BITFIELD bitmapkey SET u1 6 1 SET u1 10 1 SET u1 14 1`
    - 여러 비트를 set
- `BITCOUNT bitmapkey`
    - 비트맵에 존재하는 1의 개수 조회

### 활용 예시

- T/F 혹은 Y/N 등으로 두가지 상태만 갖는 데이터 저장하고 개수 세야 할때
    - 특히 2^32 가 안되며 DB PK가 1씩 증가하는 숫자일때
    - 전체 유저 중 설문조사를 참여한 인원 수를 세고 싶을때
        - 0, 1 로 불참/참여 를 셀 수 있음

## Hyperloglog [”key” → (v1, v3, v2, v5, v4)]

### 특징

- 대량 데이터에 중복되지 않는 고유한 값을 집계하기 쉬움
- 최대 12KB의 크기를 가짐 → 메모리 효율적
- 추정되는 오차는 0.81%로 비교적 정확하게 데이터 개수 추정 가능
- 최대 2^64 개의 아이템 저장 가능

### value에 저장 가능한 데이터

- string (숫자 가능)

### 기본 사용

- `PFADD key 123`
- `PFCOUNT key`

### 활용 예시

- 중복없이 데이터를 세고 싶은데 오차가 1% 미만은 괜찮은 경우
    - 예를 들면 좋아요를 사용자별로 한 번만 누를 수 있고 개수가 10k 와 같이 표시되는 경우

## geospatial [”key” → {“innerkey” → (127.0016, 37.60013)}]

### 특징

- 내부적으로 데이터는 sorted set으로 저장됨
- 하나의 자료구조 안에 키 값은 중복 안됨

### value에 저장 가능한 데이터

- 위도 경도 (double)

### 기본 사용

- `GEOADD key 127.0016, 37.60013 innerkey`
- `GEOPOS key innerkey`
- `GEODIST key innerkey1 innerkey2 KM`
    - KM == kilometer
- `GEOSEARCH key FROMMEMBER innerkey BYRADIUS 100 KM`
    - 옵션
        - BYRADIUS: 반경 거리 기준
        - BYBOX: 직사각형 거리를 기준
        - ASC, DESC: 각각 거리 가까운 순서, 먼 순서로 정렬

### 활용 예시

- 좌표 저장과 거리 계산, 주변 장소 검색을 효율적으로 하고 싶은 경우

## stream

### 특징

- 메시지 브로커로 사용할때 사용
- 소비자 그룹 개념이 있음
- 데이터 분산 처리 가능
- 데이터를 계속 추가하는 방식
    - 실시간 이벤트 혹은 로그성 데이터 저장을 위해 사용 가능
- 카프카에서 영향을 받아 만들어짐

* 7장에서 자세히 다룬다고 함

## zip list & skip list

- zip list 및 skip list 자료구조 설명 레퍼런스

[ZSET과 LIST](https://songhayoung.github.io/2021/06/04/Redis/zset-vs-list/#Quick-List)

- skip list 기반 sorted set 동작 원리 레퍼런스

[Sorted Set을 활용한 최근 본 상품 기능 구현](https://miintto.github.io/docs/recently-viewed-items)