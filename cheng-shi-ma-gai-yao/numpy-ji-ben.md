# Numpy基本

### 定義多維矩陣

{% code lineNumbers="true" %}
```python
import numpy as np
x = np.array([[1,2,3,4],[5,6,7,8],[9,10,11,12]])
print(x)
print(x.shape)
```
{% endcode %}

```
[[ 1  2  3  4]
 [ 5  6  7  8]
 [ 9 10 11 12]]
(3, 4)
```

### 線性運算

{% code lineNumbers="true" %}
```python
y=x*2
print(y)
print(x+y)
```
{% endcode %}

```
[[ 2  4  6  8]
 [10 12 14 16]
 [18 20 22 24]]
[[ 3  6  9 12]
 [15 18 21 24]
 [27 30 33 36]]
```

### 取元素值

{% code lineNumbers="true" %}
```python
print(x[1,2])
```
{% endcode %}

```
7
```

### 找出矩陣當中的最大值及其引數

{% code lineNumbers="true" %}
```python
print(np.max(x))
print(np.unravel_index(np.argmax(x, axis=None), x.shape))
```
{% endcode %}

```
12
(2, 3)
```
