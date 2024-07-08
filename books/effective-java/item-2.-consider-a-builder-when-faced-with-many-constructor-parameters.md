---
description: 생성자에 매개변수가 많다면 빌더를 고려하라
---

# Item 2. Consider a Builder When Faced with Many Constructor Parameters

#### 생성자에 매개변수가 많다면 빌더를 고려하라

객체 지향 프로그래밍에서 생성자는 클래스의 인스턴스를 초기화하는 데 중요한 역할을 한다. 그러나 선택적 매개변수가 많아질 경우, 코드의 가독성과 유지 보수성이 떨어질 수 있다. 이 문제를 해결하기 위해 빌더 패턴을 고려하는 것이 좋다. 여기서는 생성자 패턴의 문제점과 빌더 패턴의 장점을 비교하고, 구체적인 예제를 통해 이를 설명하고자 한다.

**점층적 생성자 패턴의 문제점**

점층적 생성자 패턴은 필수 매개변수만을 받는 생성자에서 시작해 선택적 매개변수를 추가한 생성자를 여러 개 제공하는 방식이다. 예를 들어, 식품 포장의 영양 정보를 클래스에 담고자 한다면, 다음과 같은 점층적 생성자 패턴을 사용할 수 있다.

```java
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량) 필수
    private final int servings;     // (회, 총 n회 제공량) 필수
    private final int calories;     // (1회 제공량) 선택
    private final int fat;          // (g/1회 제공량) 선택
    private final int sodium;       // (mg/1회 제공량) 선택
    private final int carbohydrate; // (g/1회 제공량) 선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

위의 코드에서 볼 수 있듯이, 생성자 매개변수가 늘어날수록 생성자 오버로딩이 복잡해지고 가독성이 떨어진다. 특히 매개변수의 순서가 중요해지면서 실수를 유발하기 쉽다.

**자바빈즈 패턴의 문제점**

자바빈즈 패턴에서는 매개변수 없는 생성자를 호출한 후, 각 필드를 설정하기 위해 setter 메서드를 사용하는 방식이다.

```java
public class NutritionFacts {
    private int servingSize = -1;  // 필수; 기본값 없음
    private int servings = -1;     // 필수; 기본값 없음
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }

    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calories = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}
```

자바빈즈 패턴은 점층적 생성자 패턴의 단점을 일부 보완할 수 있지만, 불변성을 보장하지 못하고, 객체가 완전히 생성되기 전까지는 일관성이 깨진 상태에 놓일 수 있다는 문제점이 있다.

**빌더 패턴의 장점**

빌더 패턴은 복잡한 객체를 생성하는 과정을 단순화하고 가독성을 높이는 데 유용하다. 빌더 패턴을 사용하면, 생성자나 자바빈즈 패턴에서 발생하는 문제를 피할 수 있다.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

위 예제에서 볼 수 있듯이, 빌더 패턴은 가독성과 일관성을 모두 확보할 수 있는 방법이다. 필수 매개변수는 빌더의 생성자를 통해 설정하고, 선택 매개변수는 메서드 체이닝을 통해 설정할 수 있다. 이를 통해 복잡한 객체를 쉽게 생성할 수 있으며, 불변성을 유지할 수 있다.

**결론**

생성자나 점층적 생성자 패턴이 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 것이 더 낫다. 빌더 패턴은 코드의 가독성을 높이고, 객체의 일관성과 불변성을 유지할 수 있는 강력한 방법이다. 이를 통해 유지보수성과 확장성을 모두 갖춘 객체를 생성할 수 있다.
