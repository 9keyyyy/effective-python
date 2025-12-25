# 27 map과 filter 대신 컴프리헨션을 사용하라
## TL;DR

- 리스트 컴프리헨션은 `lambda` 식을 사용하지 않기 때문에 `map`과 `filter`를 사용하는 것보다 명확
- 딕셔너리와 집합도 컴프리헨션으로 생성할 수 있음

## 리스트 컴프리헨션

파이썬은 다른 시퀀스나 이터러블로부터 리스트를 뽑을 수 있는 구문을 제공 -> 리스트 컴프리헨션 (comprehension)

### for 문
리스트에 있는 모든 원소의 제곱을 계산한다고 할 때, 가장 기본적인 방법은 `for` 루프를 활용하는 것

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
squares = []
for x in a:
    squares.append(x**2)

print(squares)

>>>
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

```

### 리스트 컴프리헨션
간단한 코드로 동일한 결과를 얻을 수 있음

```python
squared = [x**2 for x in a] # List Comprehension
print(squares)

>>>
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

```

### map

리스트 컴프리헨션 대신 `map`을 사용할 수도 있지만, 이 경우 `lambda function`을 정의해야 하기 때문에 시각적으로 좋지 않음

```python
alt = map(lambda x: x**2, a)

```

### 리스트 컴프리헨션 - 필터링
- 리스트 컴프리헨션을 사용하면 `map`보다 입력값을 필터링하기 쉬움 
- ex. 짝수의 제곱만 계산하고자 할 때, 리스트 컴프리헨션을 사용하면 뒤에 조건식만 추가하면 됨
  ```python
  even_squares = [x**2 for x in a if x % 2 == 0]
  
  print(even_squares)
  ```
- map을 사용하는 경우, `filter` 내장 함수를 함께 사용해야 하는데 가독성이 떨어짐
  ```python
  alt = map(lambda x: x**2, filter(lambda x: x % 2 == 0, a))
  assert even_squares == list(alt)
  
  ```

## 딕셔너리 컴프리헨션 & 집합 컴프리헨션

딕셔너리와 집합에도 동등한 컴프리헨션 사용 가능

```python
even_squares_dict = {x: x**2 for x in a if x % 2 ==0}
threes_cubed_set = {x**3 for x in a if x % 3 == 0}
print(even_squares_dict)
print(threes_cubed_set)

>>>
{2: 4, 4: 16, 6: 36, 8: 64, 10: 100}
{216, 729, 27}

```

`map`과 `filter`를 사용하면 여러줄로 나뉘고 코드도 길어지므로 복잡함
```python
alt_dict = dict(map(lambda x: (x, x**2),
            filter(lambda x:  x % 2 == 0, a))
alt_set = set(map(lambda x: x**3,
            filter(lambda x: x % 3 == 0, a))

```
