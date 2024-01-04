---
description: 정규화
---

# Array vs Linked List

*   Array

    * 연속 메모리
    * 한꺼번에 할당되고, 한꺼번에 해제된다
    * Random Access에 강하다. Array\[index], O(1)
    * 중간 삽입/삭제에 약하다. (단, 배열 끝은 괜찮다) O(n)
    * Cache Miss가 잘 일어나지 않는다
      * Array\[index]를 메모리에서 가져오게되면, Array\[index] 근처 데이터를 Cache에 저장
    * 요소가 사라질 때마다 GC 되지 않는다
      * Array의 특정 요소를 안쓰더라도, 저절로 사라지지 않는다. (한꺼번에 사라진다)


* Linked List
  * 불연속 메모리 - 필요한 만큼 메모리 사용
  * Graph의 일종 (Linked List and Tree) 노드와 Edge로 이루어져있는 것
  * Single Linked List, Double Linked List
    * Single Linked List는 Edge가 한개
    * Double Linked List는 Edge가 두개
  * Randoomo Access에 약하다. O(n)
  * 삽입, 삭제에 효율적. O(1)
  * Cache Miss가 잘 일어남
    * 불연속 메모리이기 때문에, 근처에 없을 가능성이 크다
  * 요소가 사라질때 GC가 일어남
    * 노트 하나가 없어질때마다, 포인터/값이 GC된다.

