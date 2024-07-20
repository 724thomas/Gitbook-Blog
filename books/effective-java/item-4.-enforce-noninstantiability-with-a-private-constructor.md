---
description: 인스턴스화를 막으려거든 private 생성자를 사용해라
---

# Item 4. Enforce Noninstantiability with a Private Constructor

자바는 객체 지향 프로그래밍 언어로, 객체 지향의 주요 개념 중 하나인 인스턴스화(객체 생성)가 중요하다. 하지만 때로는 클래스가 인스턴스화되지 않도록 해야 하는 경우가 있다. 이는 주로 유틸리티 클래스와 같이 인스턴스 생성이 불필요하거나 불가능해야 하는 경우에 해당한다. **이펙티브 자바**의 아이템 4에서는 이러한 상황에서 어떻게 private 생성자를 활용할 수 있는지에 대해 설명하고 있다.



### 인스턴스화를 막아야 하는 이유

클래스의 인스턴스화를 막아야 하는 이유는 다양하다. 그 중에서도 대표적인 이유는 다음과 같다.

### 유틸리티 클래스의 경우

많은 유틸리티 클래스는 여러 유용한 정적 메서드들로 구성되어 있다. 이러한 클래스는 상태를 가지지 않으며 인스턴스를 생성할 필요가 없다. 인스턴스를 생성하는 것은 의미가 없고, 이를 막는 것이 더 좋다.

```java
public class MathUtils {
    // private 생성자
    private MathUtils() {
        throw new AssertionError(); // 인스턴스화 방지
    }

    public static int add(int a, int b) {
        return a + b;
    }

    public static int subtract(int a, int b) {
        return a - b;
    }
}
```

위의 예제에서 `MathUtils` 클래스는 유틸리티 메서드만을 포함하고 있으며, 인스턴스화되지 않도록 `private` 생성자를 정의하고 있다. 유틸리티 클래스는 주로 공통적으로 사용되는 기능을 제공하며, 이러한 클래스는 상태를 가지지 않으므로 객체로 만들 필요가 없다. 이러한 방식으로 코드를 구성하면 코드의 재사용성이 높아지고, 중복을 피할 수 있다.



### 상속을 방지하기 위해

때로는 특정 클래스가 더 이상 상속되지 않도록 하고 싶을 때 인스턴스화를 막음으로써 그 목적을 달성할 수 있다. 예를 들어, 상수만을 담고 있는 클래스의 경우 상속을 허용하지 않는 것이 바람직하다.

```java
public class Constants {
    // private 생성자
    private Constants() {
        throw new AssertionError(); // 인스턴스화 방지
    }

    public static final String APP_NAME = "MyApp";
    public static final int VERSION = 1;
}
```

이 예제에서 `Constants` 클래스는 앱의 상수 값을 정의하고 있다. 이 클래스는 인스턴스를 생성할 필요가 없으며, 상속할 필요도 없다. 따라서 `private` 생성자를 사용하여 인스턴스화와 상속을 모두 방지한다.



### 설계 명확성

클래스 설계 상 인스턴스화가 의미가 없는 경우 이를 명확히 하기 위해 인스턴스화를 막을 수 있다. 이는 코드의 가독성과 유지보수성을 높이는 데 기여한다. 설계 명확성을 높이기 위해 인스턴스화를 방지하는 예시를 살펴보자.

```java
public class StringUtils {
    // private 생성자
    private StringUtils() {
        throw new AssertionError(); // 인스턴스화 방지
    }

    public static boolean isEmpty(String str) {
        return str == null || str.isEmpty();
    }

    public static String reverse(String str) {
        if (str == null) {
            return null;
        }
        return new StringBuilder(str).reverse().toString();
    }
}
```

`StringUtils` 클래스는 문자열과 관련된 유틸리티 메서드를 제공하며, 인스턴스화가 불필요하다. 따라서 `private` 생성자를 사용하여 인스턴스화를 방지하고, 이를 통해 클래스의 목적을 명확히 한다.



### 인스턴스화를 막는 방법: private 생성자

인스턴스화를 막기 위해 가장 간단하고 효과적인 방법은 `private` 생성자를 사용하는 것이다. 이렇게 하면 클래스 외부에서 해당 클래스의 인스턴스를 생성할 수 없다. 다음은 이를 구현하는 간단한 예시다.

```java
public class UtilityClass {
    // private 생성자
    private UtilityClass() {
        throw new AssertionError(); // 인스턴스화 방지
    }
    
    // 유틸리티 메서드
    public static String getGreeting() {
        return "Hello, World!";
    }
}
```

위 코드에서 `UtilityClass`는 유틸리티 메서드만을 포함하고 있으며, `private` 생성자를 통해 인스턴스화가 불가능하게 되어 있다. `throw new AssertionError();` 구문은 실수로라도 생성자가 호출될 경우 예외를 던져 경고를 발생시킨다.



### 싱글톤 패턴과의 차이점

싱글톤 패턴은 클래스의 인스턴스가 하나만 존재하도록 보장하는 패턴이다. 반면, `private` 생성자를 사용한 유틸리티 클래스는 아예 인스턴스화를 막는 것이 목적이다. 두 패턴 모두 인스턴스 관리를 위해 사용되지만, 목적과 구현 방식에서 차이가 있다.

```java
// 싱글톤 패턴 예제
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
        // private 생성자
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}

// 유틸리티 클래스 예제
public class UtilityClass {
    private UtilityClass() {
        throw new AssertionError();
    }

    public static void utilityMethod() {
        // 유틸리티 메서드
    }
}
```

위 예제에서 싱글톤 패턴은 인스턴스를 하나만 생성하도록 보장하며, 유틸리티 클래스는 아예 인스턴스를 생성할 수 없도록 하고 있다.



### 실세계 예제: Lombok과 자바 스프링부트를 이용한 유틸리티 클래스

Lombok을 사용하면 코드 작성이 더욱 간편해진다. Lombok의 `@UtilityClass` 어노테이션을 사용하면 유틸리티 클래스 작성을 더욱 쉽게 할 수 있다. 이를 자바 스프링부트와 함께 사용하는 예제를 통해 살펴보자.

#### Lombok 사용 예제

```java
import lombok.experimental.UtilityClass;

@UtilityClass
public class StringUtils {
    public String toUpperCase(String input) {
        return input == null ? null : input.toUpperCase();
    }
}
```

위 예제에서는 Lombok의 `@UtilityClass`를 사용해 간단히 유틸리티 클래스를 만들었다. 이 어노테이션을 사용하면 Lombok이 자동으로 `private` 생성자를 추가하여 인스턴스화를 방지해준다.

#### Spring Boot 서비스 클래스와 결합

스프링부트와 결합된 예제에서는 유틸리티 클래스가 서비스 레이어에서 어떻게 사용될 수 있는지 보여준다.

```java
import org.springframework.stereotype.Service;

@Service
public class GreetingService {

    public String getGreetingMessage(String name) {
        return StringUtils.toUpperCase("Hello, " + name);
    }
}
```

위 예제에서 `GreetingService`는 `StringUtils` 유틸리티 클래스를 사용하여 인풋 문자열을 대문자로 변환하고 있다. 이로 인해 유틸리티 클래스의 유용성이 더욱 부각된다.

#### Lombok을 사용하지 않는 유틸리티 클래스

Lombok을 사용하지 않고 유틸리티 클래스를 작성하는 경우, `private` 생성자를 수동으로 추가해야 한다.

```java
public class ManualUtilityClass {

    // private 생성자
    private ManualUtilityClass() {
        throw new AssertionError();
    }

    public static String reverseString(String input) {
        if (input == null) {
            return null;
        }
        return new StringBuilder(input).reverse().toString();
    }
}
```

#### Lombok의 추가 기능: @UtilityClass의 이점

Lombok의 `@UtilityClass` 어노테이션은 단순히 `private` 생성자를 추가하는 것 외에도 다음과 같은 이점을 제공한다:

1. **자동 정적 메서드 변환**: 클래스 내의 모든 메서드를 자동으로 정적 메서드로 변환한다.
2. **가독성 향상**: 코드의 가독성을 높여 유지보수성을 향상시킨다.
3. **불필요한 코드 제거**: 불필요한 코드를 줄여 생산성을 높인다.

```java
@UtilityClass
public class AdvancedStringUtils {

    public String toLowerCase(String input) {
        return input == null ? null : input.toLowerCase();
    }

    public boolean isEmpty(String input) {
        return input == null || input.isEmpty();
    }
}
```

위 예제에서는 `@UtilityClass` 어노테이션을 통해 `toLowerCase`와 `isEmpty` 메서드를 정적 메서드로 자동 변환하고 있다. 이는 개발자가 별도로 `static` 키워드를 추가할 필요 없이 유틸리티 메서드를 작성할 수 있게 한다.

#### 스프링부트와 유틸리티 클래스의 통합

스프링부트 프로젝트에서 유틸리티 클래스는 다양한 방식으로 유용하게 사용될 수 있다. 예를 들어, 공통적인 문자열 처리, 날짜 처리, 데이터 변환 등의 작업에 유틸리티 클래스를 사용할 수 있다. 스프링부트와의 통합 예제를 더 살펴보자.

**예제: 날짜 유틸리티 클래스**

```java
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import lombok.experimental.UtilityClass;

@UtilityClass
public class DateUtils {
    private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");

    public String formatDate(LocalDate date) {
        return date == null ? null : date.format(formatter);
    }

    public LocalDate parseDate(String dateString) {
        return dateString == null ? null : LocalDate.parse(dateString, formatter);
    }
}
```

위 예제에서는 날짜 형식 변환을 위한 유틸리티 클래스를 작성하였다. `formatDate` 메서드는 `LocalDate` 객체를 문자열로 변환하고, `parseDate` 메서드는 문자열을 `LocalDate` 객체로 변환한다.

**예제: 스프링부트 서비스 클래스에서의 사용**

```java
import org.springframework.stereotype.Service;
import java.time.LocalDate;

@Service
public class DateService {

    public String getFormattedDate(LocalDate date) {
        return DateUtils.formatDate(date);
    }

    public LocalDate parseDateString(String dateString) {
        return DateUtils.parseDate(dateString);
    }
}
```

위 예제에서 `DateService` 클래스는 `DateUtils` 유틸리티 클래스를 사용하여 날짜 형식 변환 작업을 수행한다. 이렇게 함으로써 코드의 재사용성을 높이고, 중복을 줄일 수 있다.

#### 예제: 데이터 변환 유틸리티 클래스

데이터 변환 작업은 애플리케이션에서 흔히 발생하는 작업이다. 이를 유틸리티 클래스로 구현하여 코드의 중복을 줄이고, 재사용성을 높일 수 있다.

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.experimental.UtilityClass;

@UtilityClass
public class JsonUtils {
    private static final ObjectMapper objectMapper = new ObjectMapper();

    public String toJson(Object object) throws JsonProcessingException {
        return objectMapper.writeValueAsString(object);
    }

    public <T> T fromJson(String jsonString, Class<T> valueType) throws JsonProcessingException {
        return objectMapper.readValue(jsonString, valueType);
    }
}
```

위 예제에서는 JSON 변환 작업을 위한 유틸리티 클래스를 작성하였다. `toJson` 메서드는 객체를 JSON 문자열로 변환하고, `fromJson` 메서드는 JSON 문자열을 객체로 변환한다.

**예제: 스프링부트 컨트롤러에서의 사용**

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class JsonController {

    @GetMapping("/toJson")
    public String convertToJson(@RequestParam String data) {
        try {
            return JsonUtils.toJson(data);
        } catch (JsonProcessingException e) {
            return "Error converting to JSON";
        }
    }

    @GetMapping("/fromJson")
    public String convertFromJson(@RequestParam String jsonData) {
        try {
            return JsonUtils.fromJson(jsonData, String.class);
        } catch (JsonProcessingException e) {
            return "Error converting from JSON";
        }
    }
}
```

위 예제에서 `JsonController` 클래스는 `JsonUtils` 유틸리티 클래스를 사용하여 JSON 변환 작업을 수행한다. 이를 통해 컨트롤러의 코드를 간결하게 유지하고, 코드의 재사용성을 높일 수 있다.

### 결론

유틸리티 클래스와 같이 인스턴스화가 불필요한 클래스는 `private` 생성자를 사용하여 인스턴스화를 막아야 한다. 이는 클래스의 설계 명확성을 높이고, 불필요한 객체 생성을 방지하여 자원을 절약할 수 있는 효과적인 방법이다. Lombok을 사용하면 이를 더욱 간편하게 구현할 수 있으며, 스프링부트와 결합하여 실제 프로젝트에서 유용하게 활용할 수 있다. 이러한 패턴을 통해 더욱 견고하고 유지보수하기 쉬운 코드를 작성할 수 있다.
