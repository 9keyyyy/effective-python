# 19 함수가 여러 값을 반환하는 경우 절대로 네 값 이상을 언패킹하지 말라
## TL; DR

- 함수가 여러 값을 반환하거나 언패킹할때 값이나 변수를 최대 3개까지만 사용해야 함
- 만약에 더 많이 필요할 경우 작은 클래스를 반환하거나 namedtuple 인스턴스를 반환하는 것이 좋음


### 예시 1: 언패킹 기본 예시


- 언패킹 구문을 이용하면 한번에 두 개 이상의 값을 assign 할 수 있음 
- 이를 통해 함수가 여러 개의 값을 반환하게 하고 unpacking을 할 수 있음

가장 큰 값과 작은 값을 반환하는 함수 예시

```python
def get_stats(numbers):
    minimum = min(numbers)
    maximum = max(numbers)
    return minimum, maximum

lengths = [63, 73, 72, 50, 67, 66, 71, 61, 72, 70]

minimum, maximum = get_stats(lengths)

print(f'최소: {minimum}, 최대: {maximum}')

>>>
최소: 60, 최대: 73

```

get_stats 함수는 원소가 두개인 튜플을 반환하고 전체 코드는 해당 튜플을 두 변수에 대입해서 언패킹함

### 예시 2: 별 표식 (*) 이용 예시

별 표식을 사용하면 더 많은 값을 반환 받을 수 있음
list의 값을 sort하고 scale 시키는 함수

```python
def get_avg_ratio(numbers):
    average = sum(numbers) / len(numbers)
    scaled = [x / average for x in numbers]
    scaled.sort(reverse=True)
    return scaled

longest, *middle, shortest = get_avg_ratio(lengths)

print(f'최대 길이:  {longest:>4.0%}')
print(f'최소 길이: {shortest:>4.0%}')

>>>
최대 길이: 108%
최소 길이: 89%

```

만약 필요 개체수가 많아진다면 어떻게 될까?

### 예시 3: 많은 값을 반환하는 예시 (문제 예시)

최소, 최대, 평균, 중앙값(medean), list 원소 갯수 반환

```python
def get_stats(numbers):
    minimum = min(numbers)
    maximum = max(numbers)

    count = len(numbers)
    average = sum(numbers) / count

    sorted_numbers = sorted(numbers)
    middle = count // 2
    if count % 2 == 0:
        lower = sorted_numbers[middle - 1]
        upper = sorted_numbers[middle]
        median = (lower + upper) / 2
    else:
        median = sorted_numbers[middle]

    return minimum, maximum, average, median, count

minimum, maximum, average, median, count = get_stats(lengths)

print(f'최소 길이: {minimum}, 최대 길이: {maximum}')
print(f'평균: {average}, 중앙값: {median}, 개수: {count}')

>>>
최소 길이 : 60, 최대길이: 73
평균: 67.5, 중앙값: 68.5, 개수:10

```

#### 문제점

1. 모든 반환 값이 수(number)이기 때문에 순서를 혼동하기 쉬움. 이런 실수는 나중에 찾기 힘듬
2. 함수를 호출하는 부분과 언패킹하는 부분이 길어서, 여러가지 방법으로 줄을 바꿀수 있음
    (아래 4 코드는 다 PEP8 스타일)
    ```python
    minimum, maximum, average, median, count = get_stats(
        lengths)
    
    minimum, maximum, average, median, count = \\
        get_stats(lengths)
    
    (minimum, maximum, average,
        median, count) = get_stats(lengths)
    
    (minimum, maximum, average, median, count
        ) = get_stats(lengths)
    
    ```

## 결론

함수가 여러 값을 반환하거나 언패킹할때 값이나 변수를 네 개 이상 사용하면 안됨 (최대 3개)
더 많은 값을 언패킹해야 한다면 작은 class를 만들어서 사용하거나 namedtuple을 사용해야 함

### named tuple 예시
namedtuple: collection 모듈에 있는 객체로 tuple에 추가적인 형식을 더할수 있음
```python
from collections import namedtuple

# Define a namedtuple type
Person = namedtuple('Person', ['name', 'age'])

# Create a namedtuple instance
john = Person(name='John Doe', age=30)

# Access fields
print(john.name)  # Output: John Doe
print(john.age)   # Output: 30

# Indexable like a regular tuple
print(john[0])    # Output: John Doe
``` python
```
