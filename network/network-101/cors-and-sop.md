---
description: CORS & SOP
---

# CORS & SOP

## Origin

출처는 URL의 세 가지 구성 요소, 즉 스킴(scheme), 호스트(host), 그리고 포트(port)를 기준으로 정의됩니다. 동일한 출처라 함은 이 세 가지 요소가 모두 동일한 경우를 말합니다.

예를 들어:

* `http://example.com:80`와 `http://example.com`는 동일한 출처입니다.
* `http://example.com:80`와 `https://example.com`는 다른 출처입니다(스킴이 다름).
* `http://example.com:80`와 `http://example.com:8080`는 다른 출처입니다(포트가 다름).
* `http://example.com`와 `http://sub.example.com`는 다른 출처입니다(호스트가 다름).

## SOP

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

동일 출처 정책(Same-Origin Policy, SOP)은 웹 브라우저의 중요한 보안 정책으로, 한 출처에서 불러온 스크립트가 다른 출처의 리소스에 접근하는 것을 제한합니다. 이를 통해 악의적인 사이트가 사용자의 데이터를 탈취하거나, 사용자의 권한을 오용하는 것을 방지합니다. 즉, 동일 출처(Same-Origin) 서버에 있는 리소스는 자유로이 가져올수 있지만, 다른 출처(Cross-Origin) 서버에 있는 이미지나 유튜브 영상 같은 리소스는 상호작용이 불가능합니다.





## SOP의 작동 방식

동일 출처 정책은 클라이언트 측에서 실행되는 자바스크립트가 다른 출처의 자원에 접근하는 것을 제한합니다. 이를 통해 악의적인 스크립트가 사용자의 민감한 정보를 접근하거나 조작하는 것을 방지합니다.

**주요 제한 사항**

* **DOM 조작**: 한 출처에서 불러온 스크립트는 다른 출처의 DOM(Document Object Model)을 조작할 수 없습니다.
* **AJAX 요청**: 한 출처에서 불러온 스크립트는 다른 출처에 AJAX 요청을 보낼 수 없습니다.
* **쿠키 및 로컬 저장소**: 다른 출처의 쿠키나 로컬 저장소 데이터에 접근할 수 없습니다.



## 출처 비교와 차단은 브라우저에 구현된 스펙

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

서버는 리소스 요청에 의한 응답은 말끔히 해주었고, 잘못이 없습니다. 하지만 브라우저가 이 응답을 분석해서 동일 출처가 아니면, 시뻘건 에러를 내뿜는 것입니다. (사실 서버가 헤더 정보를 덜 줘서 그런것) 그래서 브라우저에는 에러가 뜨지만, 정작 서버 쪽에는 정상적으로 응답을 했다고 하기 때문에 난항은 겪는 것입니다. 즉, 응답 데이터는 멀쩡하지만 브라우저 단에서 받을수 없도록 차단을 한 것입니다.

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>



## CORS

다른 출처의 리소스 공유에 대한 허용/비허용 정책입니다. 개발을 하다 보면 기능상 어쩔 수 없이 다른 출처 간의 상호작용을 해야 하는 케이스도 있으며, 또한 실무적으로 다른 회사의 서버 API를 이용해야 하는 상황도 존재합니다. 따라서 이와 같은 예외 사항을 두기 위해 CORS 정책을 허용하는 리소스에 한해 다른 출처라도 받아들인다는 것입니다.



<figure><img src="../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

## 해결방법

서버에서 Access-Control-Allow-Origin 헤더에 허용할 출처를 기재해서 클라이언트에 응답하면 해결됩니다. 백엔드 개발자가 해결해야합니다.
