```python
import numpy as np

arr = np.array([1, 2, 3, 4, 5])

matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
])

print(arr * 2)
print(matrix * 2)

# zeros - 0
zeros = np.zeros((3, 4))
print(zeros)

# ones - 1
ones = np.ones((2, 3))

# one matrix
eye = np.eye(4)
print(eye)

linspace = np.linspace(0, 10, 5)
print(linspace)

arange = np.arange(0, 10, 3)
print(arange)

# ---

arr = np.array([1, 2, 3, 4, 5])
print(arr[1:4]) # indexes

matrix = np.array([
    [1, 2, 3],
    [4, 5]
])
print(matrix[1:, :2])

# ---

arr = np.arange(1, 13)
print(arr)

reshaped = arr.reshape((3, 4))
print(reshaped)

# ---

arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])

# horizontal
hstack = np.hstack((arr1, arr2))
print(hstack)

# vertical
vstack = np.vstack((arr1, arr2))
print(vstack)

matrix = np.array([
    [1, 2, 3, 4],
    [5, 6, 7, 8]
])

hsplit = np.hsplit(matrix, 2)
print(hsplit)

vsplit = np.vsplit(matrix, 2)
print(vsplit)

# ---

arr = np.array([1, 2, 3, 4])

print(np.mean(arr))

print(np.std(arr))

print(np.median(arr))

print(np.min(arr), np.max(arr))
```
