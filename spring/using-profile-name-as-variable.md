# using profile name as variable

```java
private final String profile = System.getProperty("spring.profiles.active");
```

위 코드를 통해 현재 활성화된 프로필 이름을 profile에 저장할 수 있다.

profile 변수를 통하여 해당 프로필에서만 작동할 수 있는 코드를 만들 수 있다.



```java
if (profile.equals('dev')) {
    executeDevProfileOnlyMethod()
} else {
    executeOtherProfileMethod()
}
```
