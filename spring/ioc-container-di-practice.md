---
description: ioc 컨테이너와 di 구현 연습
---

# Ioc container, di practice

UserService.java

```java

class UserService {
    private final Logger logger;

    public UserService(Logger logger) {
        this.logger = logger;
    }

    public String getUser() {
        logger.log("Getting user...");
        return "{ id: 1, name: 'John Doe' }";
    }
}
```

Logger.java

```java

class Logger {
    public void log(String message) {
        System.out.println("[LOG]: " + message);
    }
}
```

Main.java

```java


// HINT: DIContainer 클래스를 만들어서 의존성을 관리하고, 해결하는 기능을 구현하세요.
// HINT: 클래스(ex. Class<?>)는 인스턴스가 아닙니다.
public class Main {
    public static void main(String[] args) {
        DIContainer container = new DIContainer();

        // 의존성 등록.
        // 의존성을 해결할 때 사용할 클래스를 등록한다.
        container.register("Logger", Logger.class);
        container.register("UserService", UserService.class);

        // 의존성 해결
        // 의존성을 해결할 때 사용할 객체를 만들고, 반환한다.
        // 이미 객체가 만들어진 경우, 재생성하지 않는다.
        // 객체를 생성할 때, 생성자에 필요한 매개변수를 스스로 해결한다.
        UserService userService = (UserService) container.resolve("UserService");
        System.out.println(userService.getUser());
    }
}
```



```java
package nudge;

import java.util.HashMap;

public class DIContainer {
    Map<String, Class<T>> classMapper;

    public DIContainer() {
        classMapper = new HashMap<>();
    }

    public void register(String name, Class<T> targetClass) {
        Class<T> targetInstance;
        if (targetClass == Logger.class) {
            targetInstance = new Logger();

        } else if (targetClass == UserService.class) {
            var logger = (Logger) this.resolve("Logger");
            targetInstance = new targetClass(logger);
        }

        classMapper.put(name, targetInstance);
    }

    public Class<T> resolve(String name) {
        return classMapper.getOrDefault(name, null);
    }

}

```





* 실제 객체를 저장하기 위해서는 Map\<String, Object>를 사용해야합니다.

```java
import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;

public class DIContainer {
    private final Map<String, Class<?>> classMapper;  // 클래스 등록을 저장하는 맵
    private final Map<String, Object> instanceMap;    // 생성된 객체를 저장하는 맵

    public DIContainer() {
        classMapper = new HashMap<>();
        instanceMap = new HashMap<>();
    }

    public void register(String name, Class<?> targetClass) {
        classMapper.put(name, targetClass);
    }

    public Object resolve(String name) {
        // 1. 이미 인스턴스가 있으면 그대로 반환 (싱글턴 개념 적용)
        if (instanceMap.containsKey(name)) {
            return instanceMap.get(name);
        }

        // 2. 등록된 클래스 정보를 가져옴
        Class<?> targetClass = classMapper.get(name);
        if (targetClass == null) {
            throw new IllegalArgumentException("No registered class for name: " + name);
        }

        try {
            // 3. 생성자 정보를 가져옴
            Constructor<?>[] constructors = targetClass.getConstructors();

            // 4. 기본 생성자가 있는 경우
            if (constructors.length == 0) {
                Object instance = targetClass.getDeclaredConstructor().newInstance();
                instanceMap.put(name, instance);
                return instance;
            }

            // 5. 첫 번째 생성자를 사용하여 의존성 주입 (DI)
            Constructor<?> constructor = constructors[0]; // 첫 번째 생성자 선택
            Class<?>[] paramTypes = constructor.getParameterTypes();
            Object[] dependencies = new Object[paramTypes.length];

            // 6. 생성자 파라미터를 순회하며 필요한 의존성을 해결
            for (int i = 0; i < paramTypes.length; i++) {
                dependencies[i] = resolve(paramTypes[i].getSimpleName());  // 재귀적으로 의존성 해결
            }

            // 7. 인스턴스 생성 후 저장
            Object instance = constructor.newInstance(dependencies);
            instanceMap.put(name, instance);
            return instance;
        } catch (Exception e) {
            throw new RuntimeException("Failed to create instance for: " + name, e);
        }
    }
}

```

