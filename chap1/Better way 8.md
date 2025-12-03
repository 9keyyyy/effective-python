# 08 여러 이터레이터에 대해 나란히 루프를 수행하려면 zip을 사용하라
### TL;DR

- 두 리스트를 동시에 이터레이션하는 경우, `zip` generator 로 동시에 두 이터레이션을 감싸는 것이 가독성이 좋음
- 두 이터레이터의 길이가 다를 경우, 주의 필요

### 동작 예시

> Task: Find the longest name

```python
names = ['Cecilia', '남궁민수', '김철수']
counts = [len(n) for n in names]

```

#### 1. 인덱스를 이용

- 인덱스를 이용해 원소를 찾는 과정은 코드를 읽기 어렵게 만든다.
  ```python
  longest_name = None
  max_count = 0
  for i in range(len(names)):
      count = counts[i]
      if count > max_count:
          longest_name = names[i]
          max_count = count
  
  ```
- `enumerate` 를 사용하더라도, 인덱스를 이용해 배열 원소를 가져오는 연산이 불가피하다.
  ```python
  for i, name in enumerate(names):
      count = counts[i]
      if count > max_count:
          longest_name = name
          max_count = count
  
  ```

#### 2. `zip` generator 를 사용

```python
for name, count in zip(names, counts):
    if count > max_count:
        longest_name = name
        max_count = count
```

- 주의) 입력 이터레이터의 길이가 서로 다를 때는 `zip` 이 자신이 감싼 이터레이터 중 어느 하나가 끝날때까지만 튜플을 내놓는다.
  ```python
  names.append('Rosalind')
  for name, count in zip(names, counts):
      print(name)
  
  >>>
  Cecilia
  남궁민수
  김철수

  ```
- `zip` 에 전달한 리스트들의 길이가 같지 않을 경우 `itertools` 내장 모듈에 들어있는 `zip_longest` 사용을 고려하라.
  ```python
  import itertools
  
  for name, count in itertools.zip_longest(names, counts):
      print(f'{name}:{count}')
  
  >>>
  Cecilia: 7
  남궁민수: 4
  김철수: 3
  Rosalind: None
  
  ```
