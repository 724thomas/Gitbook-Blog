---
description: Comparable을 구현할지 고려하라
---

# Item14. Consider Implementing Comparable

## 1. Comparable 인터페이스란?

`Comparable` 인터페이스는 자바에서 객체의 자연 순서를 정의하기 위해 사용된다. 이 인터페이스는 클래스의 인스턴스 간에 순서를 비교할 수 있도록 해준다. 기본적으로 정렬이 필요한 클래스가 `Comparable` 인터페이스를 구현하게 되면, 그 클래스의 인스턴스들끼리 자연스러운 순서로 정렬할 수 있게 된다.

`Comparable` 인터페이스는 다음과 같이 단 하나의 메서드를 가지고 있다:

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

이 메서드는 두 객체를 비교하고, 비교 결과를 나타내는 정수를 반환한다. 일반적으로 이 정수는 다음과 같은 의미를 가진다:

* 음수 (-1): 현재 객체가 비교 대상보다 작음
* 0: 현재 객체가 비교 대상과 같음
* 양수 (1): 현재 객체가 비교 대상보다 큼

## 2. Comparable 구현의 중요성

`Comparable` 인터페이스를 구현함으로써 얻게 되는 주요 이점은 다음과 같다:

### 2.1 컬렉션 정렬의 용이성

자바의 표준 라이브러리에서 제공하는 `Collections.sort()` 메서드나 `Arrays.sort()` 메서드를 이용해 쉽게 정렬할 수 있다. 이 메서드들은 기본적으로 `Comparable`을 구현한 객체를 정렬하는 데 사용된다.

```java
List<String> words = Arrays.asList("banana", "apple", "cherry");
Collections.sort(words);
System.out.println(words); // [apple, banana, cherry]
```

### 2.2 TreeMap과 TreeSet의 사용

`TreeMap`과 `TreeSet`은 내부적으로 이진 트리 구조를 사용하여 데이터를 저장하고, 이 데이터를 정렬된 상태로 유지한다. 이러한 컬렉션에 객체를 저장할 때도 `Comparable`이 요구된다.

```java
Set<Person> people = new TreeSet<>();
people.add(new Person("John"));
people.add(new Person("Alice"));
people.add(new Person("Bob"));

System.out.println(people); // [Alice, Bob, John]
```

### 2.3 표준 정렬 규칙 준수

`Comparable`을 구현함으로써 해당 객체의 표준 정렬 규칙을 정의하게 된다. 이는 개발자가 명시적으로 정의하지 않아도, 특정 객체에 대한 일반적인 정렬 규칙을 제공할 수 있게 해준다.

## 3. Comparable 구현의 예제: Lombok과 자바 스프링부트 사용

여기서는 `Person` 클래스를 예로 들어 `Comparable`을 구현하는 방법을 설명한다. 이 예제에서는 Lombok을 사용하여 코드의 간결성을 유지하면서, 자바 스프링부트를 활용하여 프로젝트에 적용해보겠다.

### 3.1 Person 클래스 작성

`Person` 클래스는 이름과 나이를 가지며, 이름을 기준으로 정렬하도록 `Comparable`을 구현한다.

```java
import lombok.Data;

@Data
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    @Override
    public int compareTo(Person other) {
        return this.name.compareTo(other.name);
    }
}
```

### 3.2 PersonController 클래스 작성

이제 스프링부트에서 `Person` 객체를 관리하는 `Controller`를 만들어보자.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

@RestController
@RequestMapping("/persons")
public class PersonController {

    @GetMapping("/sorted")
    public List<Person> getSortedPersons() {
        List<Person> persons = new ArrayList<>();
        persons.add(new Person("John", 30));
        persons.add(new Person("Alice", 25));
        persons.add(new Person("Bob", 28));

        Collections.sort(persons);
        return persons;
    }
}
```

위의 `PersonController`는 `/persons/sorted` 경로로 요청이 들어오면, 이름 순서로 정렬된 `Person` 목록을 반환한다.

### 3.3 테스트

스프링부트를 실행하고, `localhost:8080/persons/sorted`로 접속하면 정렬된 `Person` 객체들의 리스트를 확인할 수 있다.

## 4. Comparable 구현 시 주의사항

`Comparable` 인터페이스를 구현할 때 몇 가지 주의해야 할 점이 있다.

### 4.1 일관된 equals와 compareTo

`compareTo` 메서드를 구현할 때, 이 메서드의 결과가 `equals` 메서드의 결과와 일치해야 한다. 즉, `compareTo`에서 0을 반환하는 두 객체는 `equals`에서도 `true`를 반환해야 한다.

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;
    Person person = (Person) obj;
    return Objects.equals(name, person.name);
}

@Override
public int compareTo(Person other) {
    return this.name.compareTo(other.name);
}
```

### 4.2 성능 고려

`compareTo` 메서드는 객체 간의 비교 작업을 수행하기 때문에 성능이 중요한 경우 이 메서드를 효율적으로 구현하는 것이 필요하다. 특히 복잡한 비교 작업이나 자주 호출되는 경우 성능 저하를 유발할 수 있다.

### 4.3 Comparator와의 차이

`Comparable`과 `Comparator`는 모두 객체를 비교하는 데 사용되지만, `Comparable`은 객체 자체에 내장된 비교 메서드를 의미하는 반면, `Comparator`는 외부에서 객체를 비교할 수 있는 별도의 클래스를 의미한다. 특정 상황에 따라 `Comparator`를 사용하면 유연한 비교가 가능하다.

```java
java코드 복사import java.util.Comparator;

public class AgeComparator implements Comparator<Person> {
    @Override
    public int compare(Person p1, Person p2) {
        return Integer.compare(p1.getAge(), p2.getAge());
    }
}
```

이처럼 `Comparator`를 사용하면 다양한 기준으로 객체를 비교할 수 있어, 더 많은 유연성을 제공한다.

## 5. 결론

`Comparable` 인터페이스는 자바에서 객체의 정렬을 위해 매우 중요한 역할을 한다. 이 인터페이스를 구현하면, 컬렉션 정렬, `TreeMap`, `TreeSet` 등을 손쉽게 사용할 수 있고, 객체 간의 자연스러운 순서를 정의할 수 있다. 그러나 구현 시 `equals`와의 일관성, 성능 최적화 등을 고려해야 한다.
