# Counting Sort

**한정된 범위**를 갖는 값을 갖는 요소들을 정렬하고 싶을때 사용.

* O(n+k) -> k가 작다면,  O(n)
* 조건들:
  * 제한된 범위의 정수 데이터: 데이터가 작은 범위일때. 예, 0\~100
  * 데이터의 분포가 균일: 각 숫자가 비슷한 빈도로 등장
  * 추가 메모리 사용 가능: 숫자의 빈도를 저장하는 배열이 추가로 필요
  * 정수 값: 소수점이나 문자열에 적합
* 애니메이션 설명: [https://www.cs.miami.edu/home/burt/learning/Csc517.091/workbook/countingsort.html](https://www.cs.miami.edu/home/burt/learning/Csc517.091/workbook/countingsort.html)
