# using profile name as variable

```java
private final String profile = System.getProperty("spring.profiles.active");
```

위 코드를 통해 현재 활성화된 프로필 이름을 profile에 저장할 수 있다.

profile 변수를 통하여 해당 프로필에서만 작동할 수 있는 코드를 만들 수 있다.

예시:

```java
if (profile.equals('dev')) {
    executeDevProfileOnlyMethod()
} else {
    executeOtherProfileMethod()
}
```

참고: [https://velog.io/@tjswjd031/spring.profiles.active%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%B6%84%EA%B8%B0%EC%B2%98%EB%A6%AC](https://velog.io/@tjswjd031/spring.profiles.active%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%B6%84%EA%B8%B0%EC%B2%98%EB%A6%AC)
