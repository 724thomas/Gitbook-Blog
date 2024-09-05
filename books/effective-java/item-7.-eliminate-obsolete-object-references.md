---
description: Item 7. 다 쓴 객체 참조를 해제하라
---

# Item 7. Eliminate Obsolete Object References

### **메모리 누수의 정의와 중요성**

메모리 누수(memory leak)는 장기적으로 애플리케이션의 성능을 저하시키고, 심각한 경우 시스템을 다운시킬 수 있습니다. 자바는 가비지 컬렉터를 통해 메모리 관리를 자동으로 하지만, 개발자가 다 쓴 객체 참조를 적절히 해제하지 않으면 여전히 메모리 누수가 발생할 수 있습니다.

메모리 누수는 더 이상 필요하지 않은 객체가 가비지 컬렉터에 의해 회수되지 않고 메모리에 남아 있는 상태를 말합니다. 이는 사용되지 않는 객체가 계속 메모리를 차지하고 있어 새로운 객체 생성을 방해하는 상황을 초래합니다. 메모리 누수는 애플리케이션의 메모리 사용량을 증가시켜 성능을 저하시키고, 심각한 경우 메모리 부족으로 인해 프로그램이 비정상적으로 종료될 수 있습니다.



### **예시: 메모리 누수의 영향**

한 대형 전자상거래 사이트는 고객 세션을 메모리에 저장했지만, 세션이 만료된 후에도 세션 객체를 적절히 제거하지 않았습니다. 이로 인해 메모리 사용량이 점점 증가하여 서버가 자주 다운되는 문제가 발생했습니다. 이를 해결하기 위해 세션 만료 시 객체 참조를 명시적으로 해제하는 코드를 추가하여 문제를 해결했습니다.



### **메모리 누수가 발생하는 이유**

자바에서 메모리 누수가 발생하는 주요 원인은 다음과 같습니다:

* **컬렉션 클래스 사용 시 다 쓴 객체를 제거하지 않음**: 컬렉션(예: ArrayList, HashMap)에 추가된 객체는 명시적으로 제거하지 않으면 참조가 계속 유지됩니다.
* **캐시 사용 시 참조 해제 실패**: 캐시(Cache)에 저장된 객체가 필요 없을 때 적절히 제거하지 않으면 메모리 누수가 발생합니다.
* **리소스 누수**: 파일, 소켓 등의 리소스를 사용한 후 닫지 않으면 리소스가 회수되지 않습니다.
* **내부 클래스와 익명 클래스 사용 시**: 내부 클래스와 익명 클래스는 외부 클래스의 인스턴스를 암묵적으로 참조합니다. 이를 명시적으로 해제하지 않으면 메모리 누수가 발생할 수 있습니다.



### **예시: 컬렉션 클래스에서의 메모리 누수**

```java
public class MemoryLeakExample {
    public static void main(String[] args) {
        List<Object> list = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            list.add(new Object());
        }
        // 명시적으로 제거하지 않으면 메모리 누수 발생
    }
}
```



### **자바에서 메모리 누수를 방지하는 방법**

**명시적 null 처리**

컬렉션에서 다 쓴 객체를 제거할 때는 명시적으로 null 처리를 해줘야 합니다. 예를 들어:

```java
List<Object> list = new ArrayList<>();
Object obj = new Object();
list.add(obj);

// 다 쓴 객체 참조 해제
obj = null;
list.remove(0);
```



**try-with-resources와 finalize**

try-with-resources는 자바 7부터 도입된 기능으로, 리소스를 자동으로 닫아줍니다. 이를 사용하면 리소스 누수를 방지할 수 있습니다.

```java
try (BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
    // 파일 읽기
} catch (IOException e) {
    e.printStackTrace();
}
```



**WeakReference와 SoftReference**

메모리 누수를 방지하기 위해 WeakReference와 SoftReference를 사용할 수 있습니다. 이는 가비지 컬렉터가 해당 객체를 더 적극적으로 회수할 수 있도록 합니다.

```java
Map<String, WeakReference<Object>> map = new HashMap<>();
Object value = new Object();
WeakReference<Object> weakRef = new WeakReference<>(value);
map.put("key", weakRef);

// 필요 없을 때
value = null;
```

WeakReference는 객체가 약하게 참조되도록 하여, 필요 없을 때 가비지 컬렉터에 의해 회수될 수 있게 합니다.



HashMap과 WeakHashMap

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

HashMap을  사용하면 GC 돌리고 나서도 2개가 남아있습니다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

WeakHashMap을 사용하면 GC를 돌리면 1개가 남아있습니다.



### **자바 스프링부트에서 예시**

Lombok의 `@Cleanup` 어노테이션은 try-with-resources와 유사하게 리소스를 자동으로 닫아줍니다.

```java
import lombok.Cleanup;
import java.io.*;

public class Example {
    public void readFile() throws IOException {
        @Cleanup BufferedReader br = new BufferedReader(new FileReader("test.txt"));
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    }
}
```

이 예시에서는 `@Cleanup` 어노테이션을 사용하여 BufferedReader 객체를 자동으로 닫아줍니다.



**예시: 캐시에서의 메모리 누수 방지**

```java
import java.util.Map;
import java.util.WeakHashMap;

public class Cache {
    private final Map<String, Object> cache = new WeakHashMap<>();

    public void addToCache(String key, Object value) {
        cache.put(key, value);
    }

    public Object getFromCache(String key) {
        return cache.get(key);
    }
}
```

WeakHashMap을 사용하여 캐시에서 메모리 누수를 방지할 수 있습니다. 필요 없어진 키와 값은 가비지 컬렉터에 의해 자동으로 제거됩니다.



**예시: 이벤트 리스너에서의 메모리 누수 방지**

```java
import java.util.ArrayList;
import java.util.List;

public class EventSource {
    private final List<EventListener> listeners = new ArrayList<>();

    public void addListener(EventListener listener) {
        listeners.add(listener);
    }

    public void removeListener(EventListener listener) {
        listeners.remove(listener);
    }
}

interface EventListener {
    void onEvent();
}
```

이벤트 리스너를 더 이상 사용하지 않을 때는 명시적으로 제거하여 메모리 누수를 방지합니다.

###

### **사례**

1. 대형 전자상거래 사이트의 메모리 누수 해결

한 대형 전자상거래 사이트는 고객 세션을 메모리에 저장했지만, 세션이 만료된 후에도 세션 객체를 적절히 제거하지 않아 메모리 누수가 발생했습니다. 이를 해결하기 위해 세션 만료 시 객체 참조를 명시적으로 해제하는 코드를 추가하여 문제를 해결했습니다.

```java
public class SessionManager {
    private Map<String, HttpSession> sessions = new HashMap<>();

    public void addSession(String sessionId, HttpSession session) {
        sessions.put(sessionId, session);
    }

    public void removeSession(String sessionId) {
        sessions.remove(sessionId);
    }

    public void expireSessions() {
        for (String sessionId : sessions.keySet()) {
            HttpSession session = sessions.get(sessionId);
            if (session != null && session.isExpired()) {
                sessions.remove(sessionId);
            }
        }
    }
}
```

2. 게임 서버의 메모리 누수 해결

한 게임 서버는 게임 중 발생하는 이벤트를 메모리에 저장하고 있었지만, 오래된 이벤트를 적절히 제거하지 않아 메모리 누수가 발생했습니다. 이를 해결하기 위해 일정 시간이 지난 이벤트를 자동으로 제거하는 코드를 추가하여 문제를 해결했습니다.

```java
import java.util.Iterator;
import java.util.LinkedList;
import java.util.List;

public class EventManager {
    private List<Event> events = new LinkedList<>();

    public void addEvent(Event event) {
        events.add(event);
    }

    public void removeOldEvents(long maxAge) {
        Iterator<Event> iterator = events.iterator();
        long currentTime = System.currentTimeMillis();
        while (iterator.hasNext()) {
            Event event = iterator.next();
            if (currentTime - event.getTimestamp() > maxAge) {
                iterator.remove();
            }
        }
    }
}

class Event {
    private long timestamp;

    public Event() {
        this.timestamp = System.currentTimeMillis();
    }

    public long getTimestamp() {
        return timestamp;
    }
}
```

위 코드에서는 `removeOldEvents` 메서드를 통해 일정 시간이 지난 이벤트를 자동으로 제거하여 메모리 누수를 방지했습니다.



### **결론**

메모리 누수는 성능 저하와 시스템 불안정을 초래할 수 있는 중요한 문제입니다. 다 쓴 객체 참조를 적절히 해제함으로써 메모리 누수를 방지할 수 있습니다. 명시적 null 처리, try-with-resources, WeakReference와 SoftReference를 활용하는 방법을 이해하고 적절히 사용해야 합니다. Lombok과 같은 라이브러리는 이러한 작업을 더욱 간편하게 만들어 줄 수 있습니다.
