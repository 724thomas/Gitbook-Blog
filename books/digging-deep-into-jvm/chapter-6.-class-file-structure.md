# Chapter 6. Class file structure

## 2. 플랫폼 독립을 향한 초석

플랫폼 독립성은, 자바 가상 머신이 다양한 플랫폼을 지원하고, 모든 가상 머신이 동일한 프로그램 저장 형식(바이트코드)을 지원한다는 것.

다만, 자바 가상 머신은 가상 머신에서 다른 언어를 실행할 가능성을 염두해둔 "언어 독립성"도 보장하려고 노력중.

언어가 어떻든, 가상 머신을 통해 최종 "클래스 파일"로 저장합니다.

<figure><img src="/broken/files/elzw6jjYwhKynNMy19Bf" alt=""><figcaption></figcaption></figure>



## 3. 클래스 파일의 구조

모든 클래스 파일은 각각 하나의 클래스 또는 인터페이스를 정의함. 하지만, 클래스나 인터페이스를 꼭 파일에 담아둘 필요는 없는데, 동적으로 생성하여 클래스 로더에 직접 제공할 수 있기 떄문.

클래스 파일은 바이틀르 하나의 단위로 하는 이진 스트림 집합체(0,1)이다.

* 각 데이터 항목이 정해진 순서에 맞게, 구분 기호 없이 조밀하게 나열됨.
* 파일 전체가 낭비되는 공간 없이 프로그램 실행을 위한 데이터로 채워진다.
* 1 바이트가 넘는 데이터 항목은 바이트 단위로 분할됨.
* <자바 가상 머신 명세>에 따르면, 데이터를 저장하는 데는 의사 구조(pseudo structure)를 이용함.

<details>

<summary>unsigned number 부호없는 숫자</summary>

* 기본 데이터 타입을 표현.
* u1, u2, u4, u8은 각각 1, 2, 4, 8 바이트를 뜻함.
* 숫자, 인덱스 참조, 수량 값을 기술하거나 UTF-8로 인코딩된 문자열 값을 구성

</details>

<details>

<summary>테이블</summary>

* 여러개의 부호 없는 숫자나 또 다른 테이블로 구성된 복합 데이터 타입을 표현.
* 구분이 쉽도록 테이블 이름은 \_info로 끝남.
* 테이블은 계층적으로 구성된 복합 구조의 데이터를 설명하는데 사용.
* 클래스 파일 전체는 본질적으로 테이블이다.

<figure><img src="/broken/files/bxw5nx0g5tXUWlrVqYrx" alt=""><figcaption></figcaption></figure>

</details>

* 같은 타입의 데이터를 여러개 표현할때, 개수가 정해지지 않았다면, \*\_count 형태의 항목들을 통해 개수를 알려줌. {개수 + 개수만큼의 데이터 타입} 형태를 해당 타입의 "컬렉션"이라고 함
* 클래스 구조는 XML 같은 언어를 이용하지 않고, 구분자가 없기떄문에 데이터 항목은 데이터가 저장되는 바이트 순서, 의미, 길이, 순서가 엄격하게 제한됨.
* 클래스 파일의 바이트 순서는 빅 엔디언.

<details>

<summary> 빅 엔디언 (&#x3C;->리틀 엔디언)</summary>

가장 큰 단위의 바이트가 가장 낮은 주소에, 가장 작은 단위의 바이트가 가장 높은 주소에 저장.

</details>



## 3.1 매직 넘버와 클래스 파일의 버전

* 1\~4번째 바이트: 모든 클래스 파일의 처음 4바이트는 가상 머신이 허용하는 클래스 파인인지 여부를 빠르게 확인하는 매직 넘버로 시작함. 0xCAFEBABE. (클래스 파일뿐만 아니라, GIF, JPEG 같은 파일에도 파일 헤더에 매직 넘버 등장)
* 5\~6번째 바이트: 마이너 버전
* 7\~8번째 바이트: 메이저 버전
* 45번쨰 바이트 부터 자바 버전 번호가 나옵니다.

<figure><img src="/broken/files/ZsDVBt0EijWgyre7oMcA" alt=""><figcaption></figcaption></figure>

## 3.2 상수 풀

버전 번호 다음으로는 상수 풀 항목입니다. 상수 풀은 클래스 파일의 자원 창고라고 하며, 클래스 파일 구조에서 다른 클래스와 가장 많이 연관된 부분입니다. 차지하는 공간도 대체로 가장 큽니다.

상수 풀에 들어 있는 상수의 수는 고정적이지 않으므로, 상수 풀 항목들에 앞서 항목 개수를 알려주는 u2 타입 데이터가 필요. 관례상 개수는 1부터 시작.

<figure><img src="/broken/files/fsaoHo4XSZwLIM5hRVuR" alt=""><figcaption></figcaption></figure>

위 그림에서 TestClass 클래스의 상수 풀 크기는 16진수로 0x0013이고, 10진수 19에 해당. 다시 말해, 상수 풀에는 상수가 18개 존재하며, 인덱스 범위는 1\~18까지다.

* 0번째 상수를 비운 이유: '상수 풀 항목을 참조하지 않음'을 표현해야하는 특수한 경우에 인덱스를 0으로 설정하도록 함.
* 클래스파일에서 상수풀만 개수를 1부터 시작.
* 상수풀에 담기는 유형은 리터럴과 심벌 참조.

<details>

<summary>리터럴</summary>

자바 언어 수준에서 이야기하는 상수(final로 선언된 문자열이나 상수)와 비슷한 개념

리터럴은 말 그대로 **직접적으로 값이 들어 있는 상수**입니다.

<figure><img src="/broken/files/zqPC8X2kSz8dPYarnAQP" alt=""><figcaption></figcaption></figure>

</details>

<details>

<summary> 심벌 참조</summary>

컴파일과 관련된 개념. 아래 유형의 상수들을 포함

* 모듈에서 export하거나 import 하는 패키지
* 클래스와 인터페이스의 완전한 이름
* 필드 이름과 서술자
* 메서드 이름과 서술자
* 메서드 핸들과 메서드 타입(method handle, method type, invoke dynamic)
* 동적으로 계산되는 호출 사이트와 동적으로 계산되는 상수

심벌 참조는 리터럴과 달리, **다른 클래스, 메서드, 필드 등의 "위치나 이름"에 대한 참조 정보**입니다.\
**JVM이 실행 시에 연결(linking)해서 실제 메모리 주소로 변환**합니다.

<figure><img src="/broken/files/sHJMczIeiAGR28v2oroa" alt=""><figcaption></figcaption></figure>

</details>



자바 코드를 javac로 컴파일 할때는 링크 단계가 없다. 필드와 메서드가 메모리에서 어떤 구조로 표현되는가에 대한 정보는 클래스 파일에 저장되지 않음. (자바에서 링크는 가상 머신이 클래스 파일을 로드할 때 동적으로 이루어짐.)

* 가상 머신은 클래스 파일을 로드할 때 상수 풀에서 해당 심벌 참조들을 가져옴. 그 뒤, 클래스가 생성되거나 구동할 때 해석하여 실제 메모리 주소로 변환.
* 상수 풀 안의 상수 각각이 모두 테이블입니다. JDK 21 기준으로 총 17가지 상수 타입이 존재합니다.\
  (초기에 11가지 테이블 구조 + 동적언어 지원을 위한 4가지 + 모듈 시스템을 지원하기 위한 2가지)
* 공통적으로 u1 타입의 플래그 비트로 시작하며, 현재 상수가 속한 상수 타입을 나타냄.&#x20;

<details>

<summary>상수 풀의 항목 타입</summary>

<figure><img src="/broken/files/5bBgCYWI46UeNBu1qnIX" alt=""><figcaption></figcaption></figure>



</details>



상수풀 테이블 구조 예시:

예시1: 값이 10인 상수 타입은 CONSTANT\_Methodref\_info 입니다.

같은 클래스의 메서드를 가리키는 심벌 참조이며 구조는 아래와 같습니다.

<figure><img src="/broken/files/8AqR6HaEuczfsVcwyukv" alt=""><figcaption></figcaption></figure>

<figure><img src="/broken/files/DBBqYtwQstVFEhQAwjA5" alt=""><figcaption></figcaption></figure>

* tag: 플래그 비트. 상수 타입을 구분하는 용도
* index: 상수 풀에서의 인덱스로 위에서는 각각 2와 3입니다. 전체 의미를 파악하기 위해서는 2번쨰, 3번째 상수도 확인해봐야 함.

#### 인덱스2(2번째 상수): 값이 07인 두번째 상수의 플래그 비트 타입은 CONSTANT\_Class\_info입니다.

이 타입은 클래스나 인터페이스를 가리키는 심벌 참조이며 구조는 아래와 같습니다.

<figure><img src="/broken/files/VDmCPQp4MjaHewAHnrH6" alt=""><figcaption></figcaption></figure>

name\_index의 값은 4 이므로, 네번쨰 상수를 보면 해당 메서드가 정의된 클래스의 이름을 알 수 있습니다. 하나씩 추적해서 조합하면, 첫번째 상수의 의미는 Object 클래스의 기본 인스턴스 생성자임을 알 수 있습니다.

<figure><img src="/broken/files/gP32bEsgSz5c8VfMdX6Z" alt=""><figcaption></figcaption></figure>

그림의 분석 결과

* \#4: 클래스 이름
* \#5: 메서드 이름
* \#6: 메서드의 타입



표 6-3에서 확인해보면 CONSTANT\_utf8\_info 타입 상수이며 구조는 아래와 같습니다.

<figure><img src="/broken/files/PVaEwtY872Ydz7CVEb9A" alt=""><figcaption></figcaption></figure>

* length: UTF-8 축약 인코딩된 문자열이 몇 바이트인지 나타냄. \
  바로 이어서 길이만큼 데이터가 문자열의 실제 데이터.

현재 예시에서 이름이 되는 문자열의 길이(오프셋: 0x00000018)는 0x0010, 즉 16바이트임.

이어지는 16바이트는 "java/lang/Object"이다.

<figure><img src="/broken/files/TdT2h8rHyZcDNobu9o7y" alt=""><figcaption></figcaption></figure>



그 외에 나머지 12개의 상수 풀의 상수는 javap로 출력한 내용을 참조.

<details>

<summary>상수 풀의 내용 - (javap)</summary>

<div data-full-width="true"><figure><img src="/broken/files/OUdFq2JaBrkURswiSHo7" alt=""><figcaption></figcaption></figure></div>

* I, ()V, \<init>, LineNumberTable은 필드 테이블, 메서드 테이블, 속성 테이블에서 참조됨.

</details>

<details>

<summary>상수 풀의 17개 데이터 타입의 일반적인 구조</summary>

<figure><img src="/broken/files/jSdk4iJd2yrdvEgHbZ5i" alt=""><figcaption></figcaption></figure>

<div data-full-width="true"><figure><img src="/broken/files/ZNrT8sJMWq8ZsxUeWWTz" alt=""><figcaption></figcaption></figure></div>



</details>



## 3.3 접근 플래그

상수 풀듸 다음 2바이트는 현재 클래스의 접근 정보를 식별하는 접근 플래그입니다.\
현재 클래스 파일이 표현하는 대상이 클래스/인터페이스/public/abstract인지, 클래스의 경우 final인지 등의 정보가 담김니다.

<details>

<summary>접근 플래그의 종류와 의미</summary>

<figure><img src="/broken/files/X1xyftrnGYCgk3ePqvaZ" alt=""><figcaption></figcaption></figure>



</details>



## 3.4 클래스, 부모 클래그, 인터페이스 인덱스

<figure><img src="/broken/files/xKTh1oMDY6uD4rMLzJun" alt=""><figcaption></figcaption></figure>

이어서 현재 클래스 인덱스, 부모 클래스 인덱스, 인터페이스 인덱스 컬렉션(interfaces)이 나옵니다. 파일의 상속 관계를 규정.

클래스 인덱스, 부모 클래스 인덱스

* u2타입
* 현재 클래스와 부모 클래스의 완전한 이름을 결정하는데 사용.
* 최상위 클래스 java.lang.Object를 제외한 모든 자바 클래스 인덱스는 값이 0이 될 수 없음.



인터페이스 인덱스 컬렉션

* u2 타입 데이터들의 묶음
* 현재 클래스가 구현한 인터페이스들을 기술.
* 컬렉션의 첫 항목은 테이블 크기를 뜻함. 0이면 아무런 인터페이스도 구현하지 않았다는 뜻.



## 3.5 필드 테이블

인터페이스나 클래스 안에 선언된 변수들을 설명하는데 사용됩니다. 여기서 필드란, 클래스 변수와 인스턴스 변수를 뜻함. (메서드 안에 선언된 지역 변수는 필드가 아님)

필드에 포함되는 정보

* public, private protected: 필드에 접근 범위 제한
* static: 인스턴스 변수와 클래스 변수의 구분
* final: 불변 여부
* volatile: 휘발성
* transient: 직렬화 시 포함 여부
* 데이터 타입: 기본 타입, 객체, 배열
* 필드 이름

위 정보중 modifier는 각각 true/false로 나타낼 수 있음. 반면 필드 이름과 타입은 상수 풀에 정의된 상수를 참조해야함&#x20;

<details>

<summary>필드 테이블 구조</summary>

<figure><img src="/broken/files/nkqlTqynin2CJ3JU40qe" alt=""><figcaption></figcaption></figure>



</details>

<details>

<summary>AccessFlag</summary>

<figure><img src="/broken/files/aVRpha4Dd8HyCdgUdeBV" alt=""><figcaption></figcaption></figure>

데이터 타입은 u2.

</details>

<details>

<summary>name_index, descriptor_index</summary>

필드의 단순 이름과 필드 및 메서드 서술자 참조를 가리킴.

필드의 단순 이름

* 메서드나 필드의 이름을 참조할 때 이용하며 타입과 매개 변수 정보가 생략된 형태



메서드 서술자 참조

* 필드의 경우, 데이터 타입까지.
* 메서드의 경우, 매개 변수 목록과 반환값까지 기술.

<figure><img src="/broken/files/YRFvpS10eMxDKUPMOZoC" alt=""><figcaption></figcaption></figure>

</details>



필드 테이블 구조 예시:

코드를 컴파일해 생성한 TestClass.class 파일의 경우 테이블 컬렉션이 0x000000B9부터 시작.

<figure><img src="/broken/files/7lt5vt59u2mMKl9XqVTE" alt=""><figcaption></figcaption></figure>

* 0x0001: 필드 개수를 뜻하는 field\_count. 필드 테이블에는 데이터가 단 하나라는 뜻.
* 0x0002: accessFlag 이며, ACC\_PRIVATE플래그만 true.
* 0x000B: 필드 이름을 가르키고, 상수풀 11번째 상수를 확인하면 CONSTANT-Utf8\_info 타입의 문자열이고, 값은 "m".
* 0x000C: 필드 서술자이며, 상수풀에서 찾아보면 문자열 "I".

조합하면 private int m임을 알 수 있다.

* 만약 final static int m = 123; 이라면, 123을 가리키는 ConstantValue 속성이 등장했을것
* 내부 클래스는 외부 클래스를 가리킬 수단으로, 소스코드에서는 존재하지 않는 필드가 등장할 수도 있음



## 3.6 메서드 테이블

<details>

<summary>메서드 테이블 구조</summary>

<figure><img src="/broken/files/RCa90zTDKhaKxQsvOXTU" alt=""><figcaption></figcaption></figure>

</details>

필드 테이블과 유사하지만, volatile과 acc\_transient가 사라지고, synchronized, native, strictfp, abstract 키워드에 대응하는 플래그가 추가되었습니다.&#x20;

<details>

<summary>메서드 접근 플래그 표</summary>

<figure><img src="/broken/files/YicGOWZEteQRt9asLb5W" alt=""><figcaption></figcaption></figure>



</details>

* 메서드 정의는 접근 플래그, 이름 인덱스, 서술자 인덱스만으로 명확하게 표현 가능.
* 메서드 본문의 코드는 javac 컴파일러에 의해 바이트코드 명령어로 변환된 후, 메서드 속성 테이블 컬렉션의 "Code" 속성에 따로 저장됨.

<details>

<summary>메서드 테이블 구조 예시:</summary>

<figure><img src="/broken/files/zX5hQ4wmrUrBqYdpsvNh" alt=""><figcaption></figcaption></figure>

* 0x0001: 접근 플래그. ACC\_PUBLIC true
* 0x0005: 이름 인덱스. 값 = \<init>
* 0x0006: 서술자 인덱스. 값 = ()V
* 0x0001: 속성 테이블 카운터. 속성이 1개 존재
* 0x000D: 속성 테이블 인덱스. 값 =  "Code"

</details>



## 3.7 속성 테이블

클래스 파일, 필드 테이블, 메서드 테이블, Code 속성, 레코드 구성요소(record\_component\_info)는 모두 특정 시나리오에서 특정한 정보를 설명하기 위해 고유한 속성 테이블을 포함할 수 있음.

* 다른 데이터 항목들과 다르게, 속성 테이블 컬렉션은 제약이 상대적으로 느슨하며, 순서에도 엄격하지 않음
* 기존 속성 이름과 중복되지 않는 한, 자체 제작한 컴파일러가 새로운 속성 정보를 속성 테이블에 추가할 수 있도록 허용
* 자바 가상 머신이 인식하지 못하는 속성은 무시.&#x20;

<details>

<summary>자바 명세에 사전 정의된 속성</summary>

<figure><img src="/broken/files/CDiREDgI2fQfIwcl2HgX" alt=""><figcaption></figcaption></figure>

<figure><img src="/broken/files/s2YbAyvOFtFFcY05w2EZ" alt=""><figcaption></figcaption></figure>

</details>

* 속성 이름은 모두 CONSTANT\_Utf8 타입 상수를 참조해 표현.
* 속성 값은 u4 타입으로 나타냄.

속성값 자체는 사용자 정의할 수 있다. 다만 속성 테이블이 만족해야하는 공통 구조는 아래와 같음

<details>

<summary>속성 테이블 구조</summary>

<figure><img src="/broken/files/A94HxJQ2CDOZXu18wUD8" alt=""><figcaption></figcaption></figure>



</details>



### 코드 속성

자바 프로그램의 메서드 본문 코드는 자바 컴파일러에 의해 최종적으로 바이트코드 명령어로 변환된 후 Code 속성에 저장됨. (Code 속성은 메서드 테이블의 속성 컬렉션에 위치하지만, 모든 메서드 테이블에 포함되는 것은 아니다. 인터페이스 / 추상클래스의 추상 메서드는 code에 없음).&#x20;

<details>

<summary>Code속성 테이블 구조</summary>

<figure><img src="/broken/files/Tt6vTO6dskQLIxgwYdrB" alt=""><figcaption></figcaption></figure>

* attribute\_name\_index:&#x20;
  * CONSTANT\_Utf8\_info 타입 상수를 가리키는 인덱스.&#x20;
  * 값은Code로 고정되어있음.
* attribute\_length:&#x20;
  * 이 속성의 값이 차지하는 길이. &#x20;
  * 크기: 테이블전체  길이에서 6바이트를 뺀 만큼.(attribute\_name\_index와 attribute\_length까지가 6바이트를 차지함으로)
* max\_stack:&#x20;
  * 피연산자 스택의 최대 깊이.&#x20;
  * 가상 머신은 이 값만큼의 피연산자 스택을 스택 프레임에 할당
* max\_locals:&#x20;
  * 지역 변수 테이블에 피룡한 저장소 공간을 뜻함. 즉, 이 지역 변수들이 차지하는 변수 슬롯의 개수.
  * 피연산자 스택의 깊이와 변수 슬롯의 개수를 필요 이상으로 크게 잡으면 메모리가 낭비되기 때문에, 자바 가상 머신은 사용을 마친 변수 슬롯을 재사용함.
* code\_length
  * 자바 소스 코드가 컴파일되어 생성된 바이트코드 명령어들을 저장하는데 사용됨.
  * code\_length는 바이트 코드의 길이 (u4 타입, 이지만, 값이 65535를 넘을 수 없으로사실상 u2타입에 해당)
* code
  * 자바 소스 코드가 컴파일되어 생성된 바이트코드 명령어들을 저장하는데 사용됨.
  * 바이트 코드 명령어들이 순서대로 저장되는 바이트 스트림.

</details>

<details>

<summary>속성 테이블 예시:</summary>

<figure><img src="/broken/files/OixTBvAfXyJPpoX3cp7B" alt=""><figcaption></figcaption></figure>

* 0x0001: 피연산자 스택의 최대 깊이
* 0x0001: 지역변수 테이블의 용량
* 0x0000 0005: 바이트코드 영역이 차지하는 공간의 길이
* 2A B7 00 01 B1: 해당 공간 길이 5개의 바이트. 명령어도 변환됨.



명령어로 변환되는 과정:

* 2A: 바이트코드 명령어 테이블에서 0x2A에 해당하는 명령어를 찾음
* B7: 바이트코드 명령어 테이블에서 0xB7에 해당하는 명령어가 invoke special을 찾아냄.
* 00 01: invoke 명령어에 딸린 매개 변수를 읽는다. 0x0001에 해당하는 상수를 찾아보면 \<init>() 메서드를 가리키는 심벌 참조임.
* B1: 바이트코드 명령어 테이블에서 0xB1은 return

결과는 아래와 같다.

<figure><img src="/broken/files/YrwAU14AzXPvtrnI4e1J" alt=""><figcaption></figcaption></figure>

</details>



### Exceptions 속성

* 테이블의 code 속성과 같은 맥락. ( ≠ 예외 테이블)
* 메서드에서 throw될 수 있는 검사 예외들을 나열하는 기능. (메서드 설명에서 throws 키워드 뒤에 나오는 예외들)

<details>

<summary>Exceptions 속성구조</summary>

<figure><img src="/broken/files/HsVElb0mJ3soYTXbPEFU" alt=""><figcaption></figcaption></figure>

* number\_of\_exceptions:해당 메서드가 던질 수 있다고 명시한 검사 예외의 수
* exception\_index\_table: 상수 풀의 CONSTANT\_Class\_info 타입 상수를 가리키는 인덱스. 검사 예외의 클래스를 알려줌

</details>



### LineNumberTable 속성

* 자바 소스코드의 줄 번호와 바이트코드의 줄 번호(바이트코드 오프셋) 사이의 대응 관계를 설명하는 속성
* 프로그램을 실행하는 데 꼭 필요한 속성은 아니지만, 클래스 파일에 기본적으로 생성됨.
* 해당 속성이 없으면, 프로그램에서 예외가 발생했을 때 오류를 일으킨 코드의 줄 번호가 스택 추적 정보에 나타자이 않음.

<details>

<summary>LineNumberTable 속성구조</summary>

<figure><img src="/broken/files/Doo31R9Q7ZOiweG9ytOx" alt=""><figcaption></figcaption></figure>

* line\_number\_table : u2 타입인 start\_poc와 line\_number 항목으로 구성
* start\_pc: 바이트코드의 줄 번호
* line\_number: 자바 소스의 줄 번호

</details>



### LocalVariableTable, LocalVariableTypeTable 속성

LocalVariableTable

* 스택 프레임에 있는 지역 변수 테이블 안에 변수와 자바 소스 코드에 정의된 변수 사이의 관계를 설명하는 속성
* 이 속성을 새성하지 않으면, 다른 사람이 이 메서드를 참조할 때 매개 변수 이름을 알 수 없음

<details>

<summary>LocalVariableTable 속성 구조</summary>

<figure><img src="/broken/files/pMLaZlPgEXXoK6dXiEx9" alt=""><figcaption></figcaption></figure>

* local\_variable\_table: 소스 코드에서 스택 프레임과 지역 변수 사이의 관계를 나타냄
* start\_pc: 지역 변수의 수명 주기가 시작되는 바이트코드의 오프셋.
* length: 유효 범위의 길이 (start\_pc와의 조합으로 지역변수의 유효 범위 표현)
* name\_index: 상수 풀 안의 CONSTANT\_Utf8\_info 타입 상수를 가리키는 인덱스. 지역 변수 이름을 나타냄
* descriptor\_index:상수 풀 안의 CONSTANT\_Utf8\_info 타입 상수를 가리키는 인덱스. 지역 변수의 서술자를 나타냄
* index: 스택 프레임의 지역 변수 테이블에서 해당 지역 변수를 담고 있는 변수 슬롯의 위치

</details>

LocalVariableTypeTable

* 제네릭이 도입되면서 자매 속성으로 추가됨.

<details>

<summary>LocalVariableTypeTable 속성 구조</summary>

<figure><img src="/broken/files/Jyts7dnOdpv3komJpgON" alt=""><figcaption></figcaption></figure>

<figure><img src="/broken/files/52dNhTrt9Q1BmPtRBtTx" alt=""><figcaption></figcaption></figure>

</details>



### SourceFile, SourceDebugExtention 속성

SourceFile

* 클래스 파일을 생성한 자바 소스 파일 이름이 기록.

<details>

<summary>SourceFile 속성 구조</summary>

<figure><img src="/broken/files/CmjVLnHIy3iGGFpHsLPf" alt=""><figcaption></figcaption></figure>

</details>

SourceDebugExtention

* 컴파일러에 의해 또는 동적으로 생성된 클래스에 개발자를 위한 사용자 정보를 쉽게 추가할 수 있도록 설계된 속성.

<details>

<summary>SourceDebugExtention 속성 구조</summary>

<figure><img src="/broken/files/Hmiek5S3kZmmz373F3X6" alt=""><figcaption></figcaption></figure>

</details>



### ConstantValue 속성

* 정적 변수에 값을 자동으로 할당하도록 가상 머신에 알립니다.
* static 키워드로 선언된 변수(클래스 변수)에만 이 속성이 붙습니다.

<details>

<summary>ConstantValue 속성 구조</summary>

<figure><img src="/broken/files/gY9J6M1AGE0MhAz4WA8k" alt=""><figcaption></figcaption></figure>

</details>



### InnerClasses 속성

* 내부 클래스와 호스트 클래스 사이의 연결 관계를 기록합니다. 내부 클래스를 정의하면 컴파일러가 InnerClasses 속성을 자동으로 생성합니다.

<details>

<summary>InnerClasses 속성 구조</summary>

<figure><img src="/broken/files/hbyMBGCx4jDtSaxmhvPw" alt=""><figcaption></figcaption></figure>

</details>



### Deprecated, Synthetic 속성

Deprecated

* 클래스, 필드 또는 메서드를 프로그램 작성자가 폐기 대상으로 지정했음을 나타냄. (@deprecated)

Synthetic

* 컴파일러가 추가한 필드나 메서드임을 나타냄.
* JDK5부터는 접근 플래그에 ACC\_SYNTHETIC을 설정하여 컴파일러가 자동 생성해 추가한 필드과 메서드를 식별할 수 있음

<details>

<summary>Deprecated, Synthetic 속성 구조</summary>

<figure><img src="/broken/files/shVUKF5iytpotfHQnQnz" alt=""><figcaption></figcaption></figure>

</details>



### StackMapTable 속성

* JDK 6 때 클래스 파일 명세에 추가됨.
* Code 속성의 attributes 테이블에 자리하는 복잡한 가변 길이 속성.
* 가상 머신이 클래스를 로드할 때 바이트코드 검증 단계에서 타입 검증기가 활용함

<details>

<summary>StackMapTable 속성 구조</summary>

<figure><img src="/broken/files/KTm9zEpVOGeZ0Vo8prE8" alt=""><figcaption></figcaption></figure>

</details>



### Signature 속성

* JDK 5때 제네릭을 지원하기 위해 추가됨
* 클래스의 속성 테이블, 필드 테이블, 메서드 테이블에 선택적으로 등장할 수 있으며, 길이는 일정함
* 클래스, 인터페이스, 초기화 메서드, 기타 클래스 멤버가 타입 변수나 매개 변수화 타입을 포함할 경우 제네릭 시그니처 정보를 담기 위해 이용.
* 자바 언어가 제네릭을 소거법으로 구현했기때문에 이 속성이 필요함.

<details>

<summary>Signature 속성 구조</summary>

<figure><img src="/broken/files/h39IdN4ozRqj5K84BAEN" alt=""><figcaption></figcaption></figure>

</details>

<details>

<summary>소거법 (by gpt)</summary>

**컴파일 시점에는 제네릭 타입 정보를 활용하지만, 런타임에는 그 정보를 지워버리는 방식**입니다.\
즉, **제네릭 타입 정보는 컴파일 후 `.class` 파일에는 남지 않아요.**

#### 🔍 예를 들어:

```java
java복사편집List<String> list1 = new ArrayList<>();
List<Integer> list2 = new ArrayList<>();
```

이 두 개는 자바 코드에서는 분명히 타입이 다르지만,\
**컴파일된 이후에는 JVM 입장에서는 둘 다 그냥 `List`입니다.**\
제네릭 정보(`String`, `Integer`)는 지워지고,\
**Object로 변환**돼서 다뤄져요.

***

### ✅ 왜 소거법을 쓰나요?

* **하위 호환성 때문**이에요.
  * 자바 1.5 이전에는 제네릭이 없었고, 모든 컬렉션은 `Object`를 사용했어요.
  * 기존 코드와 호환되게 하기 위해, **JVM 수준에서는 제네릭을 도입하지 않고 컴파일러 수준에서만 처리**하게 만들었어요.

***

### ✅ 그래서 `Signature` 속성이 왜 필요하냐면?

> **JVM은 제네릭 타입 정보를 모릅니다.**\
> 하지만!\
> **우리가 IDE에서 타입 힌트를 보고, 리플렉션으로 타입을 확인하려면 정보가 있어야 하죠?**

그래서 컴파일러는 **제네릭 타입 정보를 `Signature` 속성이라는 곳에 따로 기록**해 둡니다.

</details>



### BootstrapMethods 속성

* JDK 7떄 추가됨. 복잡한 가변 길이 속성으로, 클래스 파일의 속성 테이블에 위치.
* invokedynamic 명령어가 참조하는 부트스트랩 메서드 한정자가 담김.

<details>

<summary>BootstrapMethods 속성 구조</summary>

<figure><img src="/broken/files/eIXSuHKZNyJZMPXIG2GK" alt=""><figcaption></figcaption></figure>

</details>



### MethodParameters 속성

* 메서드 테이블에서 사용되는 가변 길이 속성.
* 메서드가 받는 매개 변수 각각의 이름과 정보를 기록.

<details>

<summary>MethodParameters 속성 구조</summary>

<figure><img src="/broken/files/OHlGEtEoMn3mD257Bejj" alt=""><figcaption></figcaption></figure>

</details>



### 모듈화 관련 속성

* 모듈 관련 기능을 지원하기 위해 클래스 파일 형식도 확장하여 Module, ModulePackages, ModuleMainClass 속성을 추가.
* 모듈 이름, 버전, 플래그 정보와, 모듈에 정의된 requirements, exports, opens, uses, provides 요구 사항의 내용을 모두 담음.

<details>

<summary>Module, ModulePackages, ModuleMainClass속성 구조</summary>

<figure><img src="/broken/files/lxS08SSaDePPJ2Q3HUKH" alt=""><figcaption></figcaption></figure>

<figure><img src="/broken/files/68eQ12aXqAILa2XWObiL" alt=""><figcaption></figcaption></figure>



<figure><img src="/broken/files/OMGD81lpyouYKRECNJZg" alt=""><figcaption></figcaption></figure>



<figure><img src="/broken/files/iol2oThqGnhHaz314Sfj" alt=""><figcaption></figcaption></figure>

</details>



### 런타임 애너테이션 관련 속성

* 애너테이션 정보를 담기 위한 새로운 네가지 속성
* RuntimeVisibleAnnotations, RuntimeInvisitibleAnnotations, RuntimeVisibleParameterAnnotations, RuntimeInvisibleParameterAnnotations.

<details>

<summary>RuntimeVisibleAnnotations, annotation 속성 구조</summary>

<figure><img src="/broken/files/LsZdSggnvyITnTjO52f3" alt=""><figcaption></figcaption></figure>

<figure><img src="/broken/files/oihU0DrWI2UD9RRn7BDL" alt=""><figcaption></figcaption></figure>

</details>



### Record 속성

* 불변 객체를 쉽게 생성할 수 있도록 해주는 클래스.
* JDK 16부터 도입

<details>

<summary>Record 속성 구조</summary>

<figure><img src="/broken/files/D2CVyVfzTexmyNbByx0P" alt=""><figcaption></figcaption></figure>

</details>



### PermittedSubclasses 속성

* 봉인된 클래스(봉인된 인터페이스)를 지원하기 위한 속성

<details>

<summary>PermittedSubclasses 속성 구조</summary>

<figure><img src="/broken/files/PSUFgFupFag8YTdH1fjl" alt=""><figcaption></figcaption></figure>

</details>





## 4. 바이트코드 명령어 소개

자바 가상 머신의 명령어는 특정 작업을 뜻하는 바이트 길이의 숫자인 연산 코드(opcode)와 해당 작업에 필요한 0개 이상의 피연산자로 이루어짐. 피연산자는 피연산자 스택에 저장됨.



바이트 코드 명령어 집합은, 고유한 특징과 장단점이 있는 명령어 집합 아키텍처입니다.

단점:

* 연산 코드 길이가 1바이트로 제한되기 떄문에 최대 256개의 연산 코드만 표현 가능
* 클래스 파일 구조에서는 컴파일된 코드에 들어있는 피연산자의 길이 정렬을 허용하지 않음\
  -> 따라서,1바이트가 넘는 데이터를 처리할때는 가상  머샤니이 런타임에 해당 바이트들을 특정 구조로 재구성해야함.\
  -> 결론적으로, 바이트코드를 해석하고 실행하는 속도가 조금 느려짐.

<details>

<summary>"피연산자의 길이 정렬을 허용하지 않음" - gpt</summary>

`"피연산자의 길이 정렬을 허용하지 않는다"`는 말은, **JVM 바이트코드 내의 데이터가 CPU처럼 정렬(alignment)되어 있지 않다**는 의미



### ✅ 먼저, "길이 정렬(alignment)"이란?

\*\*정렬(alignment)\*\*은 메모리에 데이터를 배치할 때, 특정 크기(단위)에 맞춰 위치를 정렬하는 거예요.

#### 예시:

* `int`는 4바이트니까 **4의 배수 주소**에 두는 식.
* 이렇게 하면 CPU가 데이터를 더 빠르게 읽을 수 있음 (하드웨어 성능 최적화 때문).

> 이런 정렬은 **C, C++처럼 하드웨어에 가까운 언어에서 흔히 사용**돼요.

***

### ✅ 그런데 JVM은?

JVM의 `.class` 파일(즉, 바이트코드)은 플랫폼 독립적이에요.

> 그래서 **데이터 정렬(alignment)을 고려하지 않고**,\
> 그냥 **1바이트 단위로 바이트코드를 순서대로 나열**해요.

***

### ✅ 그래서 "피연산자의 길이 정렬을 허용하지 않는다"는 뜻은?

#### 👉 정리하면:

> **JVM의 클래스 파일은 C 언어나 하드웨어처럼 4바이트, 8바이트 단위로 데이터를 정렬하지 않고**,\
> 바이트코드를 **그냥 연속적으로 바이트 단위로 붙여 놓는다.**

#### 예시:

```java
int x = 100;
```

* JVM에서는 이 `100`이라는 상수도 상수 풀에서 `u1`, `u2`, `u4` 단위로 나뉘어져 있지만,
* 물리적으로는 **딱 붙어서 1바이트씩 순서대로 나열**됨.
* 중간에 **패딩(padding)** 을 넣어서 정렬하지 않음.

***

### ✅ 왜 그렇게 설계했나?

* JVM의 목적: **"Write once, run anywhere"**
* CPU나 OS마다 **정렬 방식이 다를 수 있기 때문에**, 정렬을 고려하지 않음.
* 대신 **JVM이 런타임에 해석하거나 JIT 컴파일할 때** 그걸 다 처리함.

</details>

장점:

* 피연산자 길이 정렬을 포기하여, 패딩과 공백을 없앨 수 있음.
* 연산 코드들이 바이트 하나로 표현되기 때문에 컴파일된 결과물이 짧고 간결.

<details>

<summary>예외 처리를 고려하지 않았을때, 자바 가상 머신이 해석하는 기본적인 실행모들의 의사 코드.</summary>

<figure><img src="/broken/files/NAsszOVTDMPgidP5YQqx" alt=""><figcaption></figcaption></figure>

</details>



## 4.1 바이트코드와 데이터 타입들

자바 가상 머신 명령어 집합을 보면 대다수 명령어 자체에 해당 연산에 필요한 데이터의 타입 정보가 포함되어 있습니다.\
(예를 들어, iload 명령어는 지역 변수 테이블에서 피연산자 스택으로 int 타입 데이터를, float 명령어는 float 타입 데이터를 읽어들임.)

데이터 타입과 관련된 대부분의 바이트 코드 명령어는 연산 코드 이름이 전용 데이터 타입을 뜻하는 문자로 시작됩니다.  (iload는 int 타입 데이터 연산을 뜻함. arraylength나 goto 같은 경우는 예외)

1바이트 크기의 연산 코드에 데이터 타입 정보까지 포함시키기엔 어려워서, 자주 쓰이는 연산과 데이터 타입 조합에만 전용 명령어를 배정했고, 그 외 타입은 별도 지시문을 이용해서 지원되는 타입으로 변환해 사용합니다.&#x20;

<details>

<summary>자바 가상 머신이 제공하는 데이터 타입 관련 명령어</summary>

<figure><img src="/broken/files/T9HbaJRI18k7IIyqYYtm" alt=""><figcaption></figcaption></figure>

</details>



### 로드와 스토어 명령어

* 스택 프레임의 지역 변수 테이블과 피연산자 스택 사이에서 데이터를 주고 받는데 사용됨.
* 데이터를 담는 역할의 피연산자 스택과 지역 변수 테이블이 해당 명령어들로 조작됩니다.

<details>

<summary>로드와 스토어 명령어들</summary>

<figure><img src="/broken/files/CwZwETAkzVLBSabNzBlR" alt=""><figcaption></figcaption></figure>

</details>



### 산술 명령어

* 피연산자 스택의 값 두개를 이용해 특정한 산술 연산을 수행하고, 결과값을 다시 피연산자 스택의 맨 위에 저장합니다.
* 정수 데이터를 다루는 부류와, 부동 소수점 데이터를 다루는 부류로 구분됨.

<details>

<summary>산술 명령어</summary>

<figure><img src="/broken/files/XLdXFxoBRHHW8Hlg2eZ8" alt=""><figcaption></figcaption></figure>

</details>



**정수 데이터:**

데이터를 다루다 0으로 나누는 경우네는 ArithmeticException을 던져야한다는 것은 규정했지만, 오버플로가 나는 경우 어떤 결과를 내야하는지는 명시되지 않습니다.

**부동 소수점 데이터:**

자바 가상 머신은 IEEE754가 정의한 비정규화된 부동 소수점 수와 점진적 언더플로 연산 규칙을 완벽하게 지원해야합니다.

반올림 모드 규칙:

* 모든 연산 결과를 적절한 정밀도로 반올림.
* 정확하지 않은 결과는 표현 가능한 가장 가까운 값으로 반올림.
* 표현 가능한 두 값이 '수학적으로 정확한 값'과 차이가 똑같다면, 최하위 비트가 0인 값을 우선시.



### 형 변환 명령어

숫자 타입 데이터를 다른 숫자 타입으로 변환합니다.

* 데이터 타입의 표현 범위가 넓어지는 경우: 자바 가상 머신이 알아서 수행합니다.
* 표현 범위가 축소되는 경우: 형 변환 명령어를 반드시 명시해야합니다. (변환 값이 부정확해지기 때문에)



### 객체 생성과 접근 명령어

<details>

<summary>객체 생성과 접근 명령어</summary>

<figure><img src="/broken/files/H7vhqudwkSpoFBNuE9X4" alt=""><figcaption></figcaption></figure>

</details>



### 피연산자 스택 관리 명령어

<details>

<summary>피연산자 스택 관리 명령어</summary>

<figure><img src="/broken/files/mAhPwBzg4Y02GCI7zob5" alt=""><figcaption></figcaption></figure>

</details>



### 제어 전이 명령어

프로그램의 실행 흐름을 조건에 따라 지정한 위치의 명령어로 이동시킵니다.

<details>

<summary>제어 전이 명령어</summary>

<figure><img src="/broken/files/kruovlUgGnCUozbfnNS0" alt=""><figcaption></figcaption></figure>

</details>

* int 타입용 조건 분기 명령어와, 참조 타입용 조건 분기 명령어가 따로 있음
* null 값 확인용 명령어도 별도로 제공
* boolean, byte, char, short 타입 조건분기에는 모두 int 타입용 명령어를 사용
* long, float, double 타입의 조건 분기에는 각 타입 전용의 비교 연산 명령어를 먼저 실행



### 메서드 호출과 반환 명령어

<details>

<summary>메서드 호출과 반환 명령어</summary>

<figure><img src="/broken/files/hta29DW48YR0XN7pbXSc" alt=""><figcaption></figcaption></figure>

<figure><img src="/broken/files/UU22oWgyxZIEZiE2XXnY" alt=""><figcaption></figcaption></figure>

</details>



### 예외 처리 명령어

* throw 문으로 예외를 명시적으로 던진은 작업은 athrow 명령어로 구현됨.
* 자바 가상 머신은 예외 처리를 바이트코드 명령어 대신 예외 테이블을 이용해 구현합니다.



### 동기화 명령어

메서드 수준 동기화

* 메서드 수준 동기화는 바이트코드 명령어가 아니라 메서드 호출과 반환 명령어로 구현됨.
* 상수풀-메서드 테이블에 있는 ACC\_SYNCHRONIZED 접근 플래그를 확인하여 알 수 있음



명령어 블록 동기화

* minotirenter와 monitorexit 명령어가 사용됨.



## 5. 설계는 공개, 구현은 비공개

자바 가상 머신 명세는 공통된 프로그램 저장 형식(클래스 파일 형식과 바이트코드 명령어 집합)을 정의합니다. 이는 어떤 하드웨어나 운영체제를 사용하든 상관없이 지켜져야하는 약속입니다. &#x20;

다만, 어떻게 구현을 했는냐는 <자바 가상 머신 명세>에 따라 구현되지 않아도 됩니다.\
가상 머신 구현자는 확장성을 활용하여, 고성능, 적은 메모리 소비, 훌륭한 이식성을 갖춘 가상 머신을 구현할 수 있습니다. 어떤 측면에 집중할지는 구현 목표에 따라 다르지만 주로 다음 두 가지로 귀결됩니다.

* 로딩 시 또는 런타임에 자바 가상 머신 코드를 다른 가상 머신용 명령어 집합으로 변환.
* 로딩 시 또는 런타임에 자바 가상 머신 코드를 호스트 CPU의 네이티브 명령어 집합으로 변환(JIT 컴파일러의 코드 생성 기술)

