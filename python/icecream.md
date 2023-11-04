---
description: Never use print() to debug again..!
---

# Icecream

print()를 사용하여 디버깅을 하지말고 ic()를 사용하자.

```python
$ pip install icrcream
```

장점:

* 표현, 변수명, 값을 다 보여준다.
* print() 보다 60% 더 빨리 타이핑 가능.
* 자료구조가 예쁘게 출력된다ㅑ.
* 출력이 강조 된다. (Syntax Highlighted)



```python
print(foo('123')) 또는
print("foo('123')", foo('123'))을 입력해서 입력값, 출력값을 나타낼때,

from icecream import ic
if(foo(123))를 하게되면

ic| foo(123): 456 라는 결과값이 나온다.

비슷하게, 
d = {'key': {1: 'one'}}
ic(d['key'][1])

ic| d['key'][1]: 'one'


class klass():
    attr = 'yep'
ic(klass.attr)

ic| klass.attr: 'yep'


def foo():
    ic()
    first()

    if expression:
        ic()
        second()
    else:
        ic()
        third()

ic| example.py:4 in foo()
ic| example.py:11 in foo()
```





{% embed url="https://github.com/gruns/icecream" %}
