# Union Find

```python
def union(x,y):
    x = find(x)
    y = find(y)
    if x < y:
        parents[y] = x
    else:
        parents[x] = y

def find(x):
    if x != parents[x]:
        parents[x] = find(parents[x])
    return parents[x]
```
