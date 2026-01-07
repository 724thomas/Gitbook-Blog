---
description: 프록시 패턴
---

# Proxy Pattern

## 정의

원본 객체를 대리하여 처리하게 함으로써 로직의 흐름을 제어하는 행동 패턴입니다. 대상 객체의 메서드를 직접 실행하는 것이 아닌, 대상 객체에 접근하기 전에 프록시(Proxy) 객체의 메서드를 접근한 후 추가적인 로직을 먼저 처리한 뒤 대상 객체에 접근합니다.

<figure><img src="/broken/files/3pDmfyZsI2JO8QayyZSU" alt=""><figcaption></figcaption></figure>



## 사용처

대상 클래스가 민감한 정보를 가지고 있거나, 인스턴스화 하기에 무겁거나, 추가 기능을 가미하고 싶은데, 원본 객체를 수정할 수 없는 상황일 때를 극복하기 위해 사용됩니다.

* 보안 (보호 프록시)
* 캐싱: 내부에 캐시를 유지하여 캐시미스가 나는 경우에만 대상 객체 접근 (캐싱 프록시)
* 데이터 유효성 검사:
* 지연 초기화: 대상 객체의 생성 비용이 비쌀때 (가상 프록시)

<details>

<summary>Example</summary>

```java
interface Image {
    void display();
}

// RealSubject: 실제로 무거운 객체
class RealImage implements Image {
    private String filename;

    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();  // 무거운 작업 (예: 이미지 파일 읽기)
    }

    private void loadFromDisk() {
        System.out.println("Loading " + filename);
    }

    @Override
    public void display() {
        System.out.println("Displaying " + filename);
    }
}

// Proxy: 실제 객체를 대리하는 프록시 객체
class ProxyImage implements Image {
    private RealImage realImage;
    private String filename;

    public ProxyImage(String filename) {
        this.filename = filename;
    }

    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);  // 실제로 필요할 때만 생성
        }
        realImage.display();
    }
}

// Client: 프록시를 사용하는 클라이언트 코드
public class ProxyPatternDemo {
    public static void main(String[] args) {
        Image image = new ProxyImage("test_image.jpg");

        // 처음 호출: 실제 객체를 생성하고 이미지를 로딩
        image.display();
        System.out.println("");

        // 두 번째 호출: 이미 생성된 객체를 사용
        image.display();
    }
}
```

</details>

* 로깅 (로깅 프록시)
* 원격 객체: 원격 위치에 있는 객체를 가져와서 로컬 처럼 보이게 할 수 있습니다. (원격 프록시)

<details>

<summary>Example</summary>

```java
interface RemoteService {
    void getchData();
}

// RealSubject: 원격 객체로 가정 (실제로는 네트워크 상에 있을 수 있음)
class RealRemoteService implements RemoteService {
    @Override
    public void fetchData() {
        System.out.println("Fetching data from a remote service...");
    }
}

// Proxy: 원격 객체와의 통신을 추상화한 프록시 객체
class RemoteServiceProxy implements RemoteService {
    private RealRemoteService realService;

    @Override
    public void fetchData() {
        if (realService == null) {
            System.out.println("Establishing connection to remote service...");
            realService = new RealRemoteService();  // 실제로 원격 객체에 연결
        }
        realService.fetchData();
    }
}

// Client: 프록시를 통해 원격 객체를 사용하는 클라이언트 코드
public class RemoteProxyDemo {
    public static void main(String[] args) {
        RemoteService service = new RemoteServiceProxy();

        // 첫 번째 호출: 원격 서비스와의 연결을 설정하고 데이터를 가져옴
        service.fetchData();
        System.out.println("");

        // 두 번째 호출: 이미 연결된 서비스에서 데이터를 가져옴
        service.fetchData();
    }
}
```

</details>



## 구조

<figure><img src="/broken/files/f4IIyok0YYxrEsQUtHWY" alt=""><figcaption></figcaption></figure>

다른 객체에 대한 접근을 제어하는 개체.

* Subject: Proxy와 RealSubject를 하나로 묶는 인터페이스 (다형성)
* RealSubject: 대상 객체
* Proxy: 대상 객체 대리자



## 장점

* OCP 준수: 변경에 닫혀있고 확장에 열려있습니다.
* SRP 준수: 대상 객체는 본인의 기능에만 집중할 수 있고, 부가 기능은 프록시 객체가 처리합니다

## 단점

* 코드의 복잡성
* 추상 레이어가 하나 추가되어서 성능저하가 생길 수 있습니다.
