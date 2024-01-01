# Binary Search

```python
def sol1(k, arr):
    left = 0
    right = len(arr)

    while left < right:
        mid = (left + right) // 2
        if arr[mid] >= k: # >=
            right = mid
        else:
            left = mid + 1

    return len(arr) - left
# arr의 원소중 k보다 크거나 같은 원소의 개수. (k보다 작은 숫자 + 1의 인덱스)
    
def sol2(k, arr):
    left = 0
    right = len(arr)

    while left < right:
        mid = (left + right) // 2
        if arr[mid] > k: # >
            right = mid
        else:
            left = mid + 1

    return len(arr) - left
# arr의 원소중 k보다 큰 원소의 개수 (k보다 큰 다음 수의 인덱스)
```
