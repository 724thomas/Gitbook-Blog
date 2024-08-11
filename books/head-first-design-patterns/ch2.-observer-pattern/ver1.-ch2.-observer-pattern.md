---
description: Ch2. 옵저버 패턴
---

# Ver1. Ch2. Observer Pattern

## 옵저버 패턴

디자인 패턴은 소프트웨어 개발에서 반복적으로 등장하는 문제를 해결하기 위한 체계적인 방법론을 제공합니다. 이 중 옵저버 패턴(Observer Pattern)은 객체 간의 일대다 의존성을 정의하여, 한 객체의 상태가 변할 때 그 상태 변화를 다른 객체들에게 자동으로 통지하는 디자인 패턴입니다.

### 옵저버 패턴의 개념

옵저버 패턴은 주로 이벤트 기반 시스템에서 사용됩니다. 예를 들어, GUI 애플리케이션에서 버튼 클릭 이벤트를 처리할 때 버튼 객체는 주제(Subject)이고, 그 버튼을 감시하는 여러 이벤트 리스너들은 옵저버(Observer)입니다. 버튼 클릭 시 모든 등록된 리스너들이 호출되어 각기 정의된 동작을 수행하게 됩니다.

#### 구성 요소

옵저버 패턴은 세 가지 주요 구성 요소로 이루어져 있습니다:

1. **Subject(주제)**: 상태를 관리하며, 상태 변경 시 옵저버들에게 알립니다.
2. **Observer(옵저버)**: 주제의 상태 변화를 감지하고, 그에 따라 행동을 수행합니다.
3. **ConcreteSubject(구체적인 주제)**: 주제의 구체적인 구현체로, 옵저버들을 관리하고 상태를 제공합니다.
4. **ConcreteObserver(구체적인 옵저버)**: 옵저버의 구체적인 구현체로, 주제의 상태 변경을 통지받아 자신의 상태를 갱신합니다.

#### 옵저버 패턴의 구조

다음은 옵저버 패턴의 구조를 나타내는 UML 다이어그램입니다.

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

### 옵저버 패턴의 예제 코드

옵저버 패턴의 작동 방식을 이해하기 위해 간단한 예제 코드를 살펴보겠습니다. 이 예제에서는 날씨 데이터를 관찰하는 WeatherData 객체와, 날씨 정보를 표시하는 Display 객체를 구현합니다.

```java
import java.util.ArrayList;

interface Observer {
    void update(float temp, float humidity, float pressure);
}

interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}

class WeatherData implements Subject {
    private ArrayList<Observer> observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList<>();
    }

    public void registerObserver(Observer o) {
        observers.add(o);
    }

    public void removeObserver(Observer o) {
        int i = observers.indexOf(o);
        if (i >= 0) {
            observers.remove(i);
        }
    }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(temperature, humidity, pressure);
        }
    }

    public void measurementsChanged() {
        notifyObservers();
    }

    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }
}

class CurrentConditionsDisplay implements Observer {
    private float temperature;
    private float humidity;
    private Subject weatherData;

    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    public void display() {
        System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity");
    }
}

public class WeatherStation {
    public static void main(String[] args) {
        WeatherData weatherData = new WeatherData();

        CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay(weatherData);

        weatherData.setMeasurements(80, 65, 30.4f);
        weatherData.setMeasurements(82, 70, 29.2f);
        weatherData.setMeasurements(78, 90, 29.2f);
    }
}
```

### 옵저버 패턴의 장단점

#### 장점

* **느슨한 결합**: Subject와 Observer가 느슨하게 결합되어 있어, 서로 독립적으로 변경될 수 있습니다.
* **확장성**: 새로운 Observer를 추가하더라도 Subject를 변경할 필요가 없습니다.

#### 단점

* **복잡성 증가**: Observer가 많아질수록 시스템의 복잡성이 증가할 수 있습니다.
* **성능 저하**: 모든 Observer에게 알림을 보내는 과정에서 성능이 저하될 수 있습니다.

### 결론

옵저버 패턴은 이벤트 중심의 애플리케이션에서 유용하게 사용되는 패턴입니다. 이 패턴을 활용하면 객체 간의 의존성을 효과적으로 관리할 수 있으며, 시스템의 유연성과 확장성을 높일 수 있습니다.
