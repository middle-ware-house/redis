# Redis에서 키를 관리하는 법

- 자료구조를 가리키는 key를 의미
    - innerkey가 아님

## 키의 자동 생성과 삭제

- 자동 생성
    - 키가 존재하지 않을때 아이템을 넣으면 자동으로 빈 자료구조 생성하여 넣음
    - 키 작업을 따로 하지 않아도 자동으로 명시한 키 값으로 자료구조 생성
    - 대신, 같은 키로 다른 자료구조가 있다면 에러 반환

- 자동 삭제
    - stream을 제외한 모든 자료구조는 모든 아이템이 삭제되면 자동으로 키도 삭제됨

## 키와 관련된 커맨드

- `EXISTS key keyname`
    - keyname에 해당하는 키 있는지 확인
    - 존재하면 1, 없으면 0
- `KEYS pattern h*llo`
    - 저장된 키를 패턴에 맞게 조회하며, 위 예시의 `h*llo` 의 경우 * 자리에 아무것도 없거나 어떤것이 와도 패턴이 맞는 키를 모두 조회
    - 패턴은 `글롭패턴`을 따른다
- `KEYS`
    - 모든 키를 조회하지만 위험한 커맨드라 사용하면 안된다.
    - 키가 너무 많이 저장되어있다면 레디스는 싱글스레드이기 때문에 하나의 작업이 너무 오래걸리면 뒤의 대기중인 작업에 영향이 간다.
- `SCAN cursor [MATCH pattern] [COUNT count] [TYPE type]`
    - 패턴, 범위, 타입에 맞는 키만 조회
    - 이 또한 범위가 너무 넓거나 많은 키가 조회되게 되면 오래걸리므로 위험하니 적절히 설정해야한다.
- `SORT key ~~~`
    - list, set, sorted set에만 사용 가능
    - 키 내부 value 정렬하여 반환
- `RENAME oldkey newkey` / `RENAMENX oldkey newkey`
    - 모두 키 이름을 바꾸지만 후자는 newkey가 이미 존재하는 키 이름이 아닌 경우에만 동작한다.
- `COPY source dest`
    - source에 지정된 키를 dest 키에 복사
        - dest에 지정한 키가 이미 있다면 에러 반환
    - 옵션
        - REPLACE
            - dest에 지정한 키가 있다 해도 삭제한 뒤에 복사하므로 에러는 발생하지 않음
- `TYPE key`
    - key에 해당하는 자료구조 타입 조회
- `OBJECT <subcommand> [options]`
    - 키에 대한 상세 정보 조회
- `FLUSHALL [ASYNC | SYNC]`
    - 저장된 모든 키 삭제
- `DEL key`
    - 키와 키에 저장된 모든 아이템 삭제, 기본적으로 동기 처리
- `UNLINK key`
    - 키와 키에 저장된 모든 아이템을 삭제하지만, 백그라운드에서 다른 스레드에 의해 처리
    - set이나 sorted set 과 같이 하나의 키에 여러 아이템 저장되는 자료구조에 DEL을 수행하는 것은 위험하기에 UNLINK를 사용하는 것이 좋음
- `EXPIRE key seconds`
    - 키가 만료될 시간을 초 단위로 정의
- `EXPIREAT key unix-time-seconds`
    - 키가 특정 유닉스 타임스탬프에 만료됨
- `EXPIRETIME key`
    - 키가 삭제되는 유닉스 타임스탬프를 초 단위로 조회
- `TTL key`
    - 키가 몇초 뒤에 삭제되는지 조회