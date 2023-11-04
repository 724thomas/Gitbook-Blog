# Sorting Array

정렬: x번째 원소 기준

```python
arr.sort(key = lambda x: x[n])
```



정렬: x\[0]번째 원소로 정렬 후,  x\[1]번째 원소로 정렬

```python
arr = sorted(arr, key = lambda x: (x[0], x[1]))
arr = sorted(sorted(arr, key = lambda x : x[0]), key = lambda x : x[1])
```

정렬: 길이로 정렬

```python
arr = sorted(arr, key=len)
```
