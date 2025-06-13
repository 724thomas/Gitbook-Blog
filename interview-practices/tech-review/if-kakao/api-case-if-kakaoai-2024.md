---
description: 추가 배포 없이 API의 case 통일시키기 / if(kakaoAI)2024
---

# Standardizing API Case Handling Without Redeployment

https://www.youtube.com/watch?v=ZE5xgQuvHFQ\&list=PLwe9WEhzDhwGXZ0REjCOYosGkrDkkHZKk\&index=7\&ab\_channel=kakaotech

#### 서비스 스펙

프레임 워크 : Spring

Parser : Jackson

Domain : 한달 주기로 결제를 진행하는 구독 서비스

사용 중인 CASE 종류: 카멜, 스네이크, 케밥

#### 변경하려고 했을 때 만난 문제들

1. API 전달 시 CASE가 다른 경우
   * 최초 요청은 카멜, 전달받는 쪽에서 스네이크를 요구하는 경우
   * 하나의 요청에 두 개의 DTO가 필요하게 됨
2. DTO 내부에 CASE가 다른 DTO를 포함시켜야 하는 경우
   * DTO 내, 외부의 CASE가 섞여있음
   * 받는 쪽에서는 어느 CASE로 파싱을 해야할 지 모르는 경우가 생김
3. Jackson에서 Unknown Property설정의 default false설정으로 인해\
   매핑되지 않는 필드가 있더라도 예외를 던지지 않고 런타임에 도달해야 해당 필드가 null임을 알 수 있음.
4.  MSA 환경에서 무중단 배포를 하기 위해서는:

    1. Callee쪽에서 변경할 CASE의 API를 새로운 버전으로 배포
       1. Caller: v1/api/users
       2. Callee: v1/api/users, v2/api/users (new!)
    2. Caller쪽에서 버전 변경한 API 배포
       1. Caller: v1 → v2/api/users (new!)
       2. Callee: v1/api/users(deprecated), v2/api/users
    3. Callee쪽에서 이전 버전 API 삭제
       1. Caller: v2/api/users
       2. Callee: v2/api/users

    Callee가 많아지면, 대규모 배포하기 위해 서버점검을 걸고 작업을 하는게 문제

#### 처리방안

1. 모든 서버에 CASE와 상관없이 파싱이 가능하도록 해주는 모듈 추가

* 대상이 되는 서버에 모듈을 설정
* Request DTO에는 RequestBodyAdvice 사용
* Response DTO에는 Custom Deserializer 사용한 모듈 설정
  * 케이스와 상관 없이 받아주는 서버 케이스에 맞춰 요청을 변경하는 전략

1. 각 서버의 CASE를 하나로 통합하고, 변경 모듈 삭제

#### Request 처리

![Image](https://github.com/user-attachments/assets/7e96d127-216f-4c90-b74c-98d1538a0aab)

* Filter 사용: 아직 어느 Controller에 매핑됐는지 모르는 상태. Target class에 맞춘 parsing불가.
* Intercepter 사용: Target class는 알 수 있지만, Request를 DispatcherServlet에서 받은 그대로 돌려주기 때문에 변경 불가.
  * 인터셉터에서는 기존의 Request를 변조할 수 없음
  * 변조하려면 Byte 코드를 조작해야 하는데, 위험성이 존재함
  * 스프링에서 지원하는 인터페이스 또한 없기에 불가능.
*   AOP 사용

    * SpringMVC에서 사용할 수 있는 RequestBodyAdvice를 사용
    * Controller를 찾아, RequestBody에 Value를 매핑하기 직전 적용됨.

    ```java
    public interface RequestBodyAdvice {
    	boolean supports(..);
    	HttpInputMessage beforeBodyRead(..); <-body를 읽기전에, 작업
    	Object afterBodyRead(..); }
    ```
* RequestBodyAdvice 사용 전략, **`beforeBodyRead`** 구현
  1. 스네이크로 먼저 변환하고, Error가 잡히면 Camel로 변경 → **`실패`**
     * Unknwon Property=FALSE 설정 때문에 에러로 잡히는 것이 아니라 null 값이 설정
  2. 어노테이션 활용 → **`성공`**
     * Snake 변환 시 @JsonNamingProerty, @JsonProperty 사용
     * 타겟 클래스에 두 가지의 어노테이션이 존재하는지 확인하고 Snake 변경 여부 판단
* Edge Cases 처리:
  * Json의 Key만 변경해야하는 경우
    * JsonNode로 Key만 뽑아내서 convert
  * 여러개의 Generic이 중첩되는 경우
    * Generic 사이에 case가 다른 경우 오류라고 가정. 가장 InnerDto를 기준으로 parsing 작업 처리.’
    *   "Generic 사이에 case가 다른 경우 오류라고 **가정**"하는 이유는?

        #### 현실적으로 모든 중첩 Generic의 케이스를 정확히 감지하고 처리하는 건 **불가능에 가깝습니다**

        * `ApiResponse<PageResponse<YourDto>>`처럼 중첩 구조에서,
        * `ApiResponse`는 camelCase, `PageResponse`는 kebab-case, `YourDto`는 snake\_case라면?
        * 각 레벨마다 서로 다른 변환 전략을 적용해야 하고, JSON 파싱 시 중간 구조(Wrapper)를 지나서 내부 DTO까지 **정확하게 매핑하기 위해선** 각 타입을 다 추적해야 함.
        * 그래서 그냥 InnerDto를 기준으로 parsing작업을 처리해서, 나머지 필드는 무시.
          * OuterDto: Kebab → CamelToSnake: null값
          * InnerDto: Camel → CamelToSnake: 정상적으로 변환
  * Map과 같이 타입이 두개 이상 가진 제너릭일 경우 제외
    * 클래스의 클래스를 재귀적으로 탐색해야함
      * 타입이 두개 이상인 클래스는 Map 제외하고는 없었음.
      * Map의 Key값을 실제 변수명으로 사용할 때가 있어 Map을 제외함.

#### Response 처리

* 서버에서 필요한 응답값을 외부에 요청하고, 받을때 Wrapper class 사용중.
* Jackson에서 특정 클래스에 Custom한 Deserializer를 적용하는 기능을 사용하기로 **`결정`**
  * Override된 deserialize 메서드 사용
  * RequestBodyAdvice에 적용한 로직을 그대로 적용
    * 기준 Generic 추출
    * Map타입 제외
    * 어노테이션을 통한 Case 판단
*   Trouble Shooting

    * Jackson 내부에서 Deserializer는 map에 캐싱되어 재사용됨 (동시성 문제)

    ```java
    @Override
    public JsonDeserializer<?> createContextual(
    		DeserializationContext ctxt,
    		BeanProperty property
    ){
    		var type = property != null?
    		property.getType() : ctxt.getContextualType();
    		
    		//return this; 일 경우 이전에 캐싱된 type으로 파싱되기에 error발생
    		return new CustomDeserializer(type); //따라서 원하는 타입으로 인스턴스를 따로 생성함
    }
    ```

#### 결론

![Image](https://github.com/user-attachments/assets/beec5c22-bbe2-42a6-aa1a-fa20186d63b3)

외부에서 어떤 케이스든, 두 가지 모듈이 적용된 상태에서는 원하는 형태의 케이스로 변환할 수 있음

* Caller와 Callee를 반복적으로 배포할 필요가 없어짐
* 배포 순서 문제도 없어짐
* 의도치 않은 케이스가 있는지는 로그를 통해 일정기간 모니터링

결과:

* 총 900개 가량의 DTO를 서비스 중단 없이 일원화
* CASE 또한 프로토콜의 일부로, 복잡성을 높이는 원인이 될 수 있음
* 팀 내 코딩 컨벤션이 있었기 때문에 가능했던 작업
  * 정규표현식을 이용해 case를 변경했기 때문에 숫자나 영어 대문자가 있었으면 의도하지 않은 결과가 나올 수 있었음
  * Naming, Wrapper class rule이 있어서 ApiResponse Dto에 커스텀 Deserializer를 일괄적으로 처리할 수 있었음.
* 팀 내 코딩 컨벤션은 쉬운 변경을 위한 기초가 됨

제한사항:

* 변환 과정에서 **리플렉션을 사용**하기 떄문에 속도에 민감하면 적용 고려 필요
* RequestBodyAdvice는 MVC에만 적용이 가능하기 때문에 Webflux에는 적용 불가
