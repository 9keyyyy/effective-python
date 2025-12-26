# 28 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라
## TL;DR
- 컴프리헨션은 여러 수준의 루프 지원 + 각 수준마다 여러 조건 지원
- 제어 하위 식이 세 개 이상인 컴포넌트는 피하자
   - 루프 2개, 조건문 2개, 루프 1개+조건문 1개 정도가 적정선
 

## 컴프리헨션 내부에 제어 하위식을 사용하는 방법

### 루프 2개: 행렬을 하나의 list로 펼치는 것

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat =  [x for row in matrix for x in row]
print(flat)

```

```bash
>>>
[1, 2, 3, 4, 5, 6, 7, 8, 9]

```

- 루프는 왼쪽에서 오른쪽 순으로 해설
- 즉, 아래와 같은 의미

```python
flat = []
for row in matrix:
   for x in row:
      flat.append(x)

```

### 루프 2개: 행렬 원소의 제곱

```python
sqaured = [[x**2 for x in row] for row in matrix]
print(sqaured)

```

```bash
>>>
[1, 4, 9], [16, 25, 36], [49, 64, 81]]

```

### 조건문 2개

- if가 연달아 붙으면 and 조건으로 간주

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
b = [x for x in a if x > 4 if x % 2 == 0]
c = [x for x in a if x > 4 and x %2 == 0]

```

## 제어 하위식이 너무 많을 때 (bad cases)

### 루프 3개

- 이런 경우엔 줄바꿈 해서 나타낼 것

```python
my_list = [
   [[1, 2, 3], [4, 5, 6]],
   ...
]
flat = [x for sublist1 in my_lists
       for sublist2 in sublist1
       for x in sublist2]

```

- 하지만 다중 for문을 쓰는 것이 가독성 측면에서 더 좋다
- extend로 인해, for 문 한개를 줄임

```python
flat = []
for sublist1 in my_lists:
    for sublist2 in sublist1:
        flat.extend(sublist2)

```

### 수준별 조건문

- 합계가 10보다 큰 행 중 내부에 3으로 나누어 떨어지는 값만 남기고 싶을 경우 -> 가독성이 매우 안좋음

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
filtered = [[x for x in row if x % 3 == 0]
            for row in matrix if sum(row) >= 10]

```

```bash
>>>
[[6], [9]]

```
