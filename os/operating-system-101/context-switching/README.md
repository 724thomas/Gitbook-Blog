# 컨텍스트 스위칭

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

컨텍스트 스위칭(context switching)은 앞서 설명한 PCB를 교환하는 과정을 말합니다. 한 프로세스에 할당된 시간이 끝나거나 인터럽트에 의해 발생합니다. 컴퓨터는 많은 프로그램을 동시에 실행하는 것처럼 보이지만 어떠한 시점에서 실행되고 있는 프로세스는 단 한 개이며, 많은 프로세스가 동시에 구동되는 것처럼 보이는 것은 다른 프로세스와의 컨텍스트 스위칭이 아주 빠른 속도로 실행되기 때문입니다.

참고: 현대 컴퓨터는 멀티코어의 CPU를 가지기 때문에 한 시점에 한 개의 프로그램이라는 설명은 틀린 설명입니다. 하지만 컨텍스트 스위칭을 설명할 때는 싱글코어를 기준으로 설명.



{% hint style="info" %}
**스위칭시 캐시미스**

컨텍스트 스위칭이 일어날 때 프로세스가 가지고 있는 메모리 주소가 그대로 있으면 잘못된 주소 변환이 일어나므로 캐시 클리어 과정을 한다.
{% endhint %}