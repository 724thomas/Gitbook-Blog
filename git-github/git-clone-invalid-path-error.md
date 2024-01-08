# git clone, invalid path error

깃허브에서 repo를 클론하게 될때, 운영체제의 차이로 발생하는 문제다.

`:`, `*`, `?`, `<`, `>`, `|`

또는

`CON`, `PRN`, `AUX`, `NUL`, etc

문자열이 windows에선는 지원하지 않는다.

해결책 1.

![](../.gitbook/assets/image.png)

git clone한 directory에서 git bash를 열고

```bash
git config core.protectNTFS false
git checkout -f HEAD
```

를 입력하게 되면, 특수문자를 허용하게 되고, 클론이 된다.



해결책 2.&#x20;

나같은 경우 파일경로에 AUX라는 폴더 이름이 있어서 문제가 생겼다.

깃레포에 들어가서, 해당 폴더를 AUXX로 바꿔주고 클론을 한 후에,

작업이 끝나고 다시 AUX로 바꿔서 푸시를 했다...





