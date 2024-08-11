---
description: 옵저버 패턴
---

# Ch2. Observer Pattern

### **기상 모니터링 애플리케이션 알아보기**

기상 모니터링 애플리케이션은 실시간으로 기상 데이터를 수집하고 이를 사용자에게 시각적으로 제공하는 시스템입니다. 예를 들어, 온도, 습도, 기압 등의 데이터를 수집하여 화면에 표시하거나, 이 데이터를 기반으로 날씨 예보를 제공할 수 있습니다.

이 시스템에서는 `WeatherData`라는 클래스를 사용해 데이터를 수집하고, `DisplayElement`라는 인터페이스를 구현한 여러 디스플레이 요소들이 `WeatherData`로부터 갱신된 데이터를 받아 화면에 표시하게 됩니다. 이때 옵저버 패턴을 사용하여 `WeatherData` 객체가 상태를 변경할 때마다 관련된 디스플레이 객체들이 자동으로 갱신될 수 있도록 합니다.

<details>

<summary>WeatherData 클래스</summary>

`WeatherData` 클래스는 기상 데이터를 저장하고 관리하는 역할을 합니다. 이 클래스는 옵저버 패턴의 **Subject** 역할을 수행하며, 옵저버 패턴의 주체로서 상태 변화가 발생할 때 등록된 옵저버들에게 변경된 상태를 알리는 역할을 합니다.

```java
import java.util.ArrayList;

public class WeatherData implements Subject {
    private ArrayList<Observer> observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList<>();
    }

    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
    }

    @Override
    public void removeObserver(Observer o) {
        observers.remove(o);
    }

    @Override
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

    // 기타 WeatherData 클래스 메서드
}
```

`WeatherData` 클래스는 옵저버들을 관리하기 위해 `ArrayList`를 사용하며, 기온, 습도, 기압 등의 데이터를 저장합니다. `setMeasurements()` 메서드를 호출할 때마다 `measurementsChanged()` 메서드를 통해 옵저버들에게 데이터 변경을 알립니다.

</details>

## **구현 목표**

이 애플리케이션의 목표는 다음과 같습니다:

1. **기상 데이터를 수집**하여 실시간으로 업데이트합니다.
2. **옵저버 패턴을 사용**하여 데이터 변경 시 자동으로 관련 디스플레이 요소들이 갱신되도록 합니다.
3. **디스플레이 요소들을 독립적으로 설계**하여 필요에 따라 쉽게 추가하거나 제거할 수 있도록 합니다.
4. **유연한 구조**를 통해 시스템이 확장 가능하도록 합니다.
5.

## **기상 스테이션용 코드 추가하기**

이제 기상 스테이션을 구현하기 위한 코드 작업을 진행합니다. 이 부분에서는 옵저버 패턴의 **Observer** 역할을 하는 다양한 디스플레이 요소를 구현하고, 이를 `WeatherData` 클래스와 연결합니다.

<details>

<summary>옵저버 인터페이스와 디스플레이 인터페이스</summary>



```java
public interface Observer {
    void update(float temp, float humidity, float pressure);
}

public interface DisplayElement {
    void display();
}
```

`Observer` 인터페이스는 기상 데이터를 수신하기 위한 `update()` 메서드를 정의하며, `DisplayElement` 인터페이스는 데이터를 화면에 표시하는 `display()` 메서드를 정의합니다.

</details>



## **옵저버 패턴 이해하기**

옵저버 패턴(Observer Pattern)은 객체 간의 일대다 관계를 정의하여 한 객체의 상태가 변경될 때 그와 연관된 다른 객체들이 자동으로 알림을 받고, 갱신될 수 있도록 하는 디자인 패턴입니다. 이 패턴은 특히 변화가 자주 일어나는 시스템에서 유용하며, 느슨한 결합(Loose Coupling)을 통해 유연하고 확장 가능한 시스템을 설계할 수 있습니다.

기상 모니터링 애플리케이션에서 옵저버 패턴을 적용하면, `WeatherData` 클래스는 데이터를 수집하고 관리하며, 온도, 습도, 기압이 변경될 때마다 관련 디스플레이 객체들에게 자동으로 알림을 보냅니다. 디스플레이 객체들은 이러한 변화를 수신하여 자신의 상태를 업데이트하고, 변경된 데이터를 사용자에게 시각적으로 제공합니다.

옵저버 패턴을 구현하려면 주체(Subject), 옵저버(Observer), 그리고 이 둘 간의 느슨한 결합을 관리하는 구조가 필요합니다.

***

## **옵저버 패턴의 작동 원리**

옵저버 패턴의 작동 원리는 매우 직관적입니다. 주체(Subject)는 옵저버들을 관리하고, 상태가 변할 때 이들에게 알림을 보내는 역할을 합니다. 옵저버(Observer)는 주체의 상태가 변할 때 이를 수신하고, 그에 따라 적절한 작업을 수행합니다.

기상 모니터링 애플리케이션에서는 `WeatherData` 클래스가 주체 역할을 하며, 다양한 디스플레이 요소들이 옵저버 역할을 합니다. 이 구조를 통해, `WeatherData`에서 기상 정보가 변경되면 자동으로 디스플레이 요소들이 갱신됩니다.

<details>

<summary>WeatherData 클래스</summary>

먼저, 옵저버 패턴의 주체(Subject) 역할을 하는 `WeatherData` 클래스를 작성합니다:

```java
java코드 복사import java.util.ArrayList;
import java.util.List;

public class WeatherData implements Subject {
    private List<Observer> observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList<>();
    }

    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
    }

    @Override
    public void removeObserver(Observer o) {
        observers.remove(o);
    }

    @Override
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

    // 기타 WeatherData 메서드
}
```

위 코드에서 `WeatherData` 클래스는 주체(Subject)로서 옵저버들을 관리합니다. 옵저버들은 리스트(`ArrayList`)에 저장되며, `notifyObservers()` 메서드에서 상태가 변경될 때 모든 옵저버들에게 변경된 데이터를 알립니다.

옵저버들은 `update()` 메서드를 통해 새로운 데이터를 수신하며, 이를 바탕으로 자신의 상태를 갱신합니다.

</details>

***

## **옵저버 패턴의 정의**

옵저버 패턴은 다음과 같은 정의를 가집니다:

* **주체(Subject):** 상태를 관리하고, 옵저버를 등록 및 삭제하며, 상태 변경 시 옵저버들에게 알림을 보냅니다.
* **옵저버(Observer):** 주체의 상태 변경을 감지하여, 그에 따라 자신의 상태를 업데이트합니다.
* **느슨한 결합(Loose Coupling):** 주체와 옵저버 간의 결합을 최소화하여, 시스템의 유연성을 높이고, 확장성을 확보합니다.

이 패턴을 적용하면, 주체가 옵저버의 구체적인 구현을 알 필요가 없으므로 주체와 옵저버는 서로 독립적으로 변경될 수 있습니다. 이를 통해 시스템이 더 유연하고, 유지보수가 용이하게 됩니다.

***



## **옵저버 패턴의 구조**

옵저버 패턴의 구조는 크게 주체(Subject)와 옵저버(Observer)로 나뉘며, 이 두 요소가 상호작용하는 방식으로 구성됩니다.&#x20;

옵저버 패턴의 주요 인터페이스와 클래스를 정의하는 예제 코드

<details>

<summary>Subject 인터페이스</summary>

주체(Subject) 인터페이스는 옵저버를 등록, 삭제, 알림을 보내는 메서드를 정의합니다:

```java
public interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}
```

</details>

<details>

<summary>Observer 인터페이스</summary>

옵저버(Observer) 인터페이스는 주체로부터 상태 변경을 통지받을 때 호출되는 `update()` 메서드를 정의합니다:

```java
public interface Observer {
    void update(float temp, float humidity, float pressure);
}
```

</details>

<details>

<summary>DisplayElement 인터페이스</summary>

디스플레이 요소를 위한 `DisplayElement` 인터페이스는 데이터를 화면에 표시하는 `display()` 메서드를 정의합니다:

```java
public interface DisplayElement {
    void display();
}
```

</details>

<details>

<summary>CurrentConditionsDisplay 클래스</summary>

이제 옵저버 인터페이스를 구현한 `CurrentConditionsDisplay` 클래스를 작성해 보겠습니다. 이 클래스는 주체의 상태가 변경될 때 업데이트되며, 현재 조건을 화면에 출력합니다.

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
    private float temperature;
    private float humidity;
    private Subject weatherData;

    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    @Override
    public void display() {
        System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity");
    }
}
```

이 클래스는 주체 `WeatherData`에 옵저버로 등록되어, 기상 데이터가 변경될 때마다 `update()` 메서드가 호출되어 상태를 갱신하고, 그 상태를 `display()` 메서드를 통해 화면에 출력합니다.

</details>

<details>

<summary> <strong>WeatherStation 클래스</strong></summary>

마지막으로, `WeatherStation` 클래스를 통해 모든 요소들을 통합하여 기상 모니터링 애플리케이션을 완성합니다:

```java
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

`WeatherStation` 클래스에서는 `WeatherData` 객체를 생성하고, `CurrentConditionsDisplay` 객체를 옵저버로 등록합니다. 그 후, 기상 데이터가 변경될 때마다 `setMeasurements()` 메서드를 통해 새로운 데이터를 설정하고, 자동으로 디스플레이가 업데이트되는 것을 확인할 수 있습니다.

</details>



## **느슨한 결합의 위력**

옵저버 패턴의 가장 큰 장점 중 하나는 \*\*느슨한 결합(Loose Coupling)\*\*을 제공한다는 점입니다. 느슨한 결합이란, 객체들 간의 의존성을 최소화하여, 객체가 서로에 대해 구체적인 정보를 갖지 않도록 하는 것을 의미합니다. 이렇게 하면 한 객체가 변경되더라도 다른 객체에 미치는 영향이 최소화됩니다.

기상 모니터링 애플리케이션에서 `WeatherData` 클래스는 옵저버가 누구인지, 어떻게 구현되었는지 알 필요가 없습니다. 그저 옵저버가 `Observer` 인터페이스를 구현하고 있다는 사실만 알고 있으며, 옵저버에게 데이터를 전달할 때 `update()` 메서드를 호출하기만 하면 됩니다. 이로 인해 `WeatherData` 클래스는 새로운 디스플레이 요소가 추가되거나 기존 요소가 변경되더라도 코드 수정 없이 계속해서 동작할 수 있습니다.

<details>

<summary>느슨한 결합이 어떻게 구현되는지 보여주는 예시</summary>

```java
public interface Observer {
    void update(float temp, float humidity, float pressure);
}

public interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}

public class WeatherData implements Subject {
    private List<Observer> observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList<>();
    }

    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
    }

    @Override
    public void removeObserver(Observer o) {
        observers.remove(o);
    }

    @Override
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
```



</details>



### **기상 스테이션 설계하기**

이제 기상 스테이션의 구체적인 설계를 진행해 보겠습니다. 기상 스테이션은 여러 디스플레이 요소로 구성되며, 각 요소는 `WeatherData` 객체로부터 기상 정보를 받아 이를 화면에 표시합니다. 여기서는 `CurrentConditionsDisplay` 외에도 `StatisticsDisplay`와 `ForecastDisplay`라는 추가적인 디스플레이 요소를 설계합니다.

<details>

<summary><strong>StatisticsDisplay 클래스</strong></summary>

`StatisticsDisplay` 클래스는 온도에 대한 통계를 계산하여 화면에 출력합니다. 이 클래스는 `Observer`와 `DisplayElement` 인터페이스를 구현합니다.

```java
public class StatisticsDisplay implements Observer, DisplayElement {
    private float maxTemp = 0.0f;
    private float minTemp = 200;
    private float tempSum= 0.0f;
    private int numReadings;
    private Subject weatherData;

    public StatisticsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temp, float humidity, float pressure) {
        tempSum += temp;
        numReadings++;

        if (temp > maxTemp) {
            maxTemp = temp;
        }

        if (temp < minTemp) {
            minTemp = temp;
        }

        display();
    }

    @Override
    public void display() {
        System.out.println("Avg/Max/Min temperature = " + (tempSum / numReadings)
            + "/" + maxTemp + "/" + minTemp);
    }
}
```

`StatisticsDisplay` 클래스는 `WeatherData`로부터 기상 데이터를 받아 평균, 최고, 최저 온도를 계산하고 이를 출력합니다. 이 클래스도 역시 `WeatherData`와 느슨하게 결합되어 있으며, 변경이 필요할 경우 독립적으로 수정할 수 있습니다.



</details>

<details>

<summary><code>ForecastDisplay</code> 클래스</summary>

기압 데이터를 기반으로 간단한 날씨 예측을 제공합니다. 이 클래스도 `Observer`와 `DisplayElement` 인터페이스를 구현합니다.

```java
public class ForecastDisplay implements Observer, DisplayElement {
    private float currentPressure = 29.92f;  
    private float lastPressure;
    private Subject weatherData;

    public ForecastDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temp, float humidity, float pressure) {
        lastPressure = currentPressure;
        currentPressure = pressure;

        display();
    }

    @Override
    public void display() {
        System.out.print("Forecast: ");
        if (currentPressure > lastPressure) {
            System.out.println("Improving weather on the way!");
        } else if (currentPressure == lastPressure) {
            System.out.println("More of the same");
        } else if (currentPressure < lastPressure) {
            System.out.println("Watch out for cooler, rainy weather");
        }
    }
}
```

`ForecastDisplay` 클래스는 기압을 사용하여 단순한 날씨 예측을 제공합니다. 기압이 상승하면 날씨가 좋아질 것으로, 하락하면 나빠질 것으로 예측합니다. 이 클래스 또한 `WeatherData`와 느슨하게 결합되어 있으며, 쉽게 확장하거나 수정할 수 있습니다.



</details>



### **기상 스테이션 구현하기**

이제 설계한 모든 구성 요소들을 통합하여 기상 스테이션을 구현하겠습니다. 이 단계에서는 `WeatherData` 클래스와 다양한 디스플레이 요소들을 연결하여, 실제로 기상 데이터를 받아오는 애플리케이션을 완성합니다.

<details>

<summary><code>WeatherStation</code> 클래스</summary>

이 클래스는 `WeatherData` 객체를 생성하고, 여러 디스플레이 요소를 옵저버로 등록하며, 기상 데이터가 변경될 때마다 디스플레이가 자동으로 업데이트되도록 설정합니다.

```java
public class WeatherStation {
    public static void main(String[] args) {
        // WeatherData 객체 생성
        WeatherData weatherData = new WeatherData();

        // 디스플레이 요소 생성 및 옵저버로 등록
        CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay(weatherData);
        StatisticsDisplay statisticsDisplay = new StatisticsDisplay(weatherData);
        ForecastDisplay forecastDisplay = new ForecastDisplay(weatherData);

        // 기상 데이터 설정 및 갱신
        weatherData.setMeasurements(80, 65, 30.4f);
        weatherData.setMeasurements(82, 70, 29.2f);
        weatherData.setMeasurements(78, 90, 29.2f);
    }
}
```

#### **코드 설명**

* **WeatherData 객체 생성:** `WeatherData` 클래스는 기상 데이터를 관리하고, 옵저버들에게 알림을 보내는 주체(Subject) 역할을 합니다.

<!---->

* **디스플레이 요소 생성 및 등록:** `CurrentConditionsDisplay`, `StatisticsDisplay`, `ForecastDisplay`와 같은 디스플레이 클래스들은 옵저버(Observer) 역할을 합니다. 이들은 생성될 때 `WeatherData` 객체에 자신을 옵저버로 등록합니다.

<!---->

* **기상 데이터 설정 및 갱신:** `setMeasurements()` 메서드를 호출하여 기상 데이터를 갱신할 때마다, `WeatherData` 객체는 등록된 모든 옵저버들에게 알림을 보내고, 각 디스플레이 요소는 자신의 화면을 갱신합니다.

</details>

***

### **Subject 인터페이스 구현하기**

`WeatherData` 클래스는 `Subject` 인터페이스를 구현하여, 옵저버를 등록, 제거하고, 상태가 변경될 때 옵저버들에게 알림을 보내는 기능을 제공합니다. 이 인터페이스는 옵저버 패턴에서 주체가 가져야 할 주요 메서드를 정의합니다.

<details>

<summary>Subject 인터페이스 코드</summary>

```java
public interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}
```



</details>

<details>

<summary>Observer 인터페이스 코드</summary>

옵저버 인터페이스도 간단히 정리하겠습니다. 각 옵저버는 이 인터페이스를 구현하며, `update()` 메서드를 통해 주체로부터 상태 변경을 통지받습니다.

```java
public interface Observer {
    void update(float temp, float humidity, float pressure);
}
```



</details>



***

### **디스플레이 요소 구현하기**

#### **Implementing Display Elements**

이제 다양한 디스플레이 요소들을 구현해 보겠습니다. 각 디스플레이 클래스는 옵저버로서 `WeatherData` 객체로부터 기상 데이터를 받아 화면에 표시하는 역할을 합니다. 앞서 설명한 `CurrentConditionsDisplay`, `StatisticsDisplay`, `ForecastDisplay` 클래스를 다시 정리하여, 모든 코드를 포함합니다.

<details>

<summary>CurrentConditionsDisplay 클래스</summary>

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
    private float temperature;
    private float humidity;
    private Subject weatherData;

    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    @Override
    public void display() {
        System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity");
    }
}
```

`CurrentConditionsDisplay`는 현재 온도와 습도를 화면에 표시하는 역할을 합니다.



</details>

<details>

<summary>StatisticsDisplay 클래스</summary>

```java
public class StatisticsDisplay implements Observer, DisplayElement {
    private float maxTemp = 0.0f;
    private float minTemp = 200;
    private float tempSum = 0.0f;
    private int numReadings;
    private Subject weatherData;

    public StatisticsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temperature, float humidity, float pressure) {
        tempSum += temperature;
        numReadings++;

        if (temperature > maxTemp) {
            maxTemp = temperature;
        }

        if (temperature < minTemp) {
            minTemp = temperature;
        }

        display();
    }

    @Override
    public void display() {
        System.out.println("Avg/Max/Min temperature = " + (tempSum / numReadings)
            + "/" + maxTemp + "/" + minTemp);
    }
}
```

`StatisticsDisplay`는 온도의 평균, 최고, 최저 값을 계산하여 화면에 표시하는 역할을 합니다.



</details>

<details>

<summary>ForecastDisplay 클래스</summary>

```java
public class ForecastDisplay implements Observer, DisplayElement {
    private float currentPressure = 29.92f;  
    private float lastPressure;
    private Subject weatherData;

    public ForecastDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temperature, float humidity, float pressure) {
        lastPressure = currentPressure;
        currentPressure = pressure;

        display();
    }

    @Override
    public void display() {
        System.out.print("Forecast: ");
        if (currentPressure > lastPressure) {
            System.out.println("Improving weather on the way!");
        } else if (currentPressure == lastPressure) {
            System.out.println("More of the same");
        } else if (currentPressure < lastPressure) {
            System.out.println("Watch out for cooler, rainy weather");
        }
    }
}
```

`ForecastDisplay`는 기압 데이터를 기반으로 날씨를 예측하여 화면에 표시하는 역할을 합니다.

</details>

####

