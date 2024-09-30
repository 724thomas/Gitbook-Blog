---
description: 추이성 문제 해결하기 - 이펙티브 자바
---

# Handling Transitivity Issues in Java

### 1. 추이성이란 무엇인가?

추이성은 첫 번째 객체가 두 번째 객체와 같고, 두 번째 객체가 세 번째 객체와 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 의미입니다. 이를 코드로 쉽게 설명할 수 있습니다:

```java
@Override
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
           ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

이 조건을 만족하지 않으면, 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가할 때 문제가 발생할 수 있습니다.

### 2. Point 클래스 예제

다음은 2차원 좌표계를 표현하는 간단한 Point 클래스를 예로 들어보겠습니다.

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

이제 이 클래스를 확장하여 점에 색상을 추가해보겠습니다.

### 3. ColorPoint 클래스

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
}
```

여기서 equals 메서드를 어떻게 구현해야 할까요? 그대로 둔다면 Point의 구현이 상속되어 ColorPoint 객체의 색상 정보는 무시한 채 비교를 수행하게 됩니다.

### 4. 대칭성 위배 코드

다음 코드는 equals 규약을 위배한 코드입니다:

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

이 방식은 일반 Point를 ColorPoint와 비교한 결과와 그 반대를 비교한 결과가 다를 수 있습니다.

```java
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);

p.equals(cp)는 true를, cp.equals(p)는 false를 반환합니다.
```

### 5. 추이성 위배 코드

다음은 추이성을 위배한 코드입니다:

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;

    // 여기 일반 Point면 색상을 무시하고 비교한다.
    if (!(o instanceof ColorPoint))
        return o.equals(this);

    // 여기 ColorPoint면 색상까지 비교한다.
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

이 방식은 대칭성을 지켜주지만, 추이성을 깨뜨립니다.

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

p1.equals(p2)와 p2.equals(p3)는 true를 반환하는데, p1.equals(p3)는 false를 반환합니다.
```

### 6. 해결책

사실 이 현상은 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제입니다. 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않습니다.

다음은 잘못된 해결책 코드입니다:

```java
@Override
public boolean equals(Object o) {
    if (o == null || getClass() != o.getClass())
        return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

이 방식은 equals 규약을 지켜주지만, 리스코프 치환 원칙(Liskov Substitution Principle)을 위배하게 됩니다.

### 7. 리스코프 치환 원칙(Liskov Substitution Principle) 해결책

리코프 치환 원칙을 준수하면서 추이성을 유지하는 방법은 다음과 같습니다:

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;
    if (!(o instanceof ColorPoint))
        return o.equals(this);
    ColorPoint cp = (ColorPoint) o;
    return super.equals(cp) && ((ColorPoint) o).color == color;
}
```

이 방식은 추이성을 지키면서도, 리스코프 치환 원칙을 준수합니다.

### 8. 결론

추상 클래스를 상속받는 하위 클래스에서 equals 규약을 지키면서 값을 추가하는 방법은 어렵습니다. 이를 위해서는 다양한 설계 패턴을 활용하여 문제를 해결해야 합니다.
