---
description: 오브젝트와 의존관계
---

# Object and relation 1.1.\~1.3.

#### 초기 구현

UserDao는 JDBC API를 직접 사용하여 사용자 정보를 등록(add)하고 조회(get)합니다.

문제점:

* DB 연결 생성 코드가 add, get 메서드에 중복됨
* DB 종류, 접속 방식, 드라이버 변경 시 여러 메서드를 동시에 수정해야 함
* 데이터 접근 로직과 DB 연결 로직이 하나의 클래스에 섞여 있음

변경에 취약한 구조



#### 관심사 분리

관심사를 분리할 수 있는 구조이고,

* 사용자 데이터를 SQL로 다루는 책임
* DB 커넥션을 생성하는 책임

DB 커넥션 생성부분을 별도의 메서드로 추출.



#### 상속을 통한 확장

getConnection()을 추상 메서드로 만들고, UserDao를 상속한 NUserDao, DUserDao에서 DB 커넥션 생성 방식을 따로 구현.

이로 인해:

* UserDao는 수정 불필요
* DB 연결 방식만 서브클래스로 확장이 가능

But,

* 동일한 DB 커넥션 로직을 재사용하기 어려움

상속이 아닌 다른 확장 방식이 필요.



#### 클래스 분리

별도의 클래스 SimpleConnectionMaker로 DB 커넥션 생성 책임을 분리하여 UserDao가 해당 클래스를 필드로 사용.

But,

* UserDao가 특정한 구현 클래스에 의존하게 됨
* DB 연결 방식을 바꾸려면 UserDao를 수정해야함.

결국, 독립성은 확보되지 않음



#### 인터페이스 도입

ConnectionMaker 인터페이스를 도입하여 독립성을 확보

* UserDao는 ConnectionMaker 인터페이스에만 의존
* NConnectionMaker, DConnectionMaker에서 구현

UserDao 입장에서는 DB 연결이 어떻게 되는지 모르게 됨.
