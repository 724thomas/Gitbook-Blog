---
description: 카카오 계정 캐시 전환기 / if(kakao)2022
---

# Kakao Account Cache Migration / if(kakao)2022

[컨퍼런스 영상](https://youtu.be/ssSbfF8Otgc?si=WxKcLaBOMrZ_Gsoh)

### 해결한 기술적인 이슈 요약

* client side sharding 선택과 Jedis
  * Backend 서버가 어떤 샤드로 요청 보낼지 선택 : client side sharding
  * 샤딩이 되어 있는 DB/캐시 가 자동으로 어떤 샤드로 가면 될지 알려줌 : server side sharding
    * 샤드 3개인데 서버는 4개 이상 띄워짐 1\~2개는 샤드로 보내는거 도와줌
* mget과 multi key 문제
  * mget 커맨드를 활용하기 위해 여러 Jedis 클라이언트가 필요했고, 이러한 기능이 제공되지 않아서 직접 구현
  * TreeMap 을 활용한 이유는 해시 링에서 탐색하는 과정을 편하게 구현 가능하기 때문
* 커넥션 반복 get/release 문제
  * 반복문에서 반복해서 커넥션을 가져왔다가 반납하는 문제
* Redis로 전환과 카나리 배포
  * 안정성을 위해 카나리 배포를 사용할 수 있음
* 커넥션풀 문제 및 버전업
  * 라이브러리 버전 이슈를 모니터링으로 확인하고 버전업
