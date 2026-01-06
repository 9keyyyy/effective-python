# 36 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라
## TL;DR

- itertools를 이용해서 이터레이터(iterator)나 제너레이터(generator)를 다룰 수 있음
- itertools 함수는 크게 3가지 범주로 나눌 수 있음
    1. 여러 이터레이터 연결
    2. 이터레이터 필터링
    3. 이터레이터의 원소 조합

---

## itertools

```python
import itertools

```

- 이터레이터나 제너레이터를 다룰 때 유용하게 사용 할 수 있는 파이썬 내장 모듈
- `help(itertools)` 를 통해 itertools 문서 확인 가능
        

## 1. 여러 이터레이터 연결

### chain(p, q, …) 

```python
it = itertools.chain([1, 2, 3], [4, 5, 6])
print(list(it))

>>>
[1, 2, 3, 4, 5, 6]

```

- *p,q, … : iterables*
- iterables의 모든 이터러블이 소진될 때 까지 진행되는 이터레이터 생성 → 이터러블들을 하나로 합치는 이터레이터

### repeat(elem [,n]) 

```python
it = itertools.repeat('안녕', 3)
print(list(it))

>>>
['안녕', '안녕', '안녕']

```

- elem : 반복하고 싶은 객체
- n : 반복 횟수, 지정되지 않았다면 무한번
- elem 을 n 번 생성하는 이터레이터 → 한값을 계속 반복해서 내놓고 싶을 때 사용

### cycle(p) 

```python
it = itertools.cycle([1, 2])
result = [next(it) for _ in range (10)]
print(result)

>>>
[1, 2, 1, 2, 1, 2, 1, 2, 1, 2]

```

- p : *iterable*
- itertools.**cycle**(*iterable*)
- 이터레이터가 내놓는 원소들을 계속 반복하고 싶을 때

### tee(it, n) 

```python
it1, it2, it3 = itertools.tee(['하나', '둘'], 3)
print(list(it1))
print(list(it2))
print(list(it3))

>>>
['하나', '둘']
['하나', '둘']
['하나', '둘']

```

- it: 단일 iterable
- n = 2: 반복 횟수
- iterable 에서 n 개의 독립 이터레이터를 반환 → n 개 만큼의 이터레이터를 병렬적으로 만들고 싶을 때
- 이 함수로 만들어진 각 이터레이터를 소비하는 속도가 같지 않으면, 처리가 덜 된 이터레이터의 원소를 큐에 담아줘야하므로 메모리 사용량이 늘어남!
    
    ```python
    original_iterable = [1, 2, 3, 4, 5]
    iter1, iter2 = tee(original_iterable)
    
    # iter1을 한 번 더 소비
    next(iter1)
    
    # iter2를 끝까지 소비
    for item in iter2:
        print(item)
    
    # iter1을 나머지 부분까지 소비
    for item in iter1:
        print(item)
    
    ```
    
    - 이 경우 iter1은 iter2가 다 소비 되기 전까지 계속 해서 원본 이터러블 원소를 가지고 있어야함

### zip_longest(p, q, … , fillvalue) 

```python
keys = ['하나', '둘', '셋']
values = [1, 2]

normal = list(zip(keys, values))
print('zip:', normal)

it = itertools.zip_longest(keys, values, fillvalue='없음')
longest = list(it)
print('zip_longest:', longest)

>>>
zip: [('하나', 1), ('둘', 2)]
zip_longest: [('하나', 1), ('둘', 2), ('셋', '없음')]

```

- p, q, … : iterables
- fillvalue = None : 누락된 값을 채울 객체
- zip 내장함수 (Better Way 8 참고)의 변종으로 이터러블들의 각각의 요소를 집계하는 이터레이터 생성
- 가장 긴 이터러블이 소진될 때 까지 이터레이션 반복
- 여러 이터러블 중 짧은 쪽 이터러블의 원소를 다 사용한 경우 fillvalue로 지정한 값을 채워넣어줌.

## 2. 이터레이터에서 원소 거르기 (필터링)

### islice(seq, [start,] stop [, step])

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

first_five = itertools.islice(values, 5)
print('앞에서 다섯 개:', list(first_five))

middle_odds = itertools.islice(values, 2, 8, 2)
print('중간의 홀수들:', list(middle_odds))

>>>
앞에서 다섯 개: [1, 2, 3, 4, 5]
중간의 홀수들: [3, 5, 7]

```

- seq : interable
- start : 시작 index, 지정되어 있지 않다면 0
- stop : 종료 index, stop - 1 까지
- step : 지정되어 있지 않다면 1
- 원소 인덱스를 이용해 슬라이싱
- 동작은 시퀀스 슬라이싱이니 스트라이딩(Better Way 11, 12)과 비슷

### takewhile(predicate, seq)

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
less_than_seven = lambda x: x < 7
it = itertools.takewhile(less_than_seven, values)
print(list(it))

>>>
[1, 2, 3, 4, 5, 6]

```

- predicate : 술어
- seq : interable
- 이터러블에서 주어진 술어(predicate)가 false 를 반환하는 첫 원소가 나타날 때 까지 원소를 돌려줌 → 즉 술어가 Ture 반환하는 동안 계속 돌려줌

### dropwhile(predicate, seq) 

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
less_than_seven = lambda x: x < 7
it = itertools.dropwhile(less_than_seven, values)
print(list(it))

>>>
[7, 8, 9, 10]

```

- predicate : 술어
- seq : interable
- takewhile 의 반대
- 이터레이터에서 주어진 술어(predicate)가 false 를 반환하는 첫 원소가 나타날 때 까지 원소를 건너 뜀 → 즉 술어가 True 를 반환하는 동안 건너띔

### filterfalse(predicate, seq) 

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens = lambda x: x % 2 == 0

filter_result = filter(evens, values)
print('Filter:', list(filter_result))

filter_false_result = itertools.filterfalse(evens, values)
print('Filter false:', list(filter_false_result))

>>>
Filter: [2, 4, 6, 8, 10]
Filter false: [1, 3, 5, 7, 9]

```

- predicate : 술어
- seq : interable
- filter 내장 함수의 반대
- 주어진 이터레이터에서 술어가 false를 반환하는 모든 원소를 돌려줌

## 3. 이터레이터에서 원소 조합 만들기

### accumulate(p [,func, *, initial])

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
sum_reduce = itertools.accumulate(values)
print('합계:', list(sum_reduce))

def sum_modulo_20(first, second):
    output = first + second
    return output % 20

modulo_reduce = itertools.accumulate(values, sum_modulo_20)
print('20으로 나눈 나머지의 합계:', list(modulo_reduce))

>>>
합계: [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]
20으로 나눈 나머지의 합계: [1, 3, 6, 10, 15, 1, 8, 16, 5, 15]

```

- p : interable
- func(acc, cur) = operator.add: 이항 함수(파라미터가 2개)
- inital = None (default)
- 이항 함수를 반복 적용하면서 이터레이터 원소의 값을 하나로 줄어줌 
- 원본 이터레이터의 각 원소에 대해 누적된 값을 내놓는 이터레이터 반환
- 이 결과는 한번에 한단계식 결과를 내놓는 점을 제외하고는 functools 내장 모듈에 있는 reduce 함수와 결과가 동일

### product(p,q, [repeat]) —>

```python
single = itertools.product([1, 2], repeat=2)
print('리스트 한 개:', list(single))

multiple = itertools.product([1, 2], ['a', 'b'])
print('리스트 두 개:', list(multiple))

>>>
리스트 한 개: [(1, 1), (1, 2), (2, 1), (2, 2)]
리스트 두 개: [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]

```

- p, q, … : iterables
- repeat = 1 : 자기 자신과의 곱을 하고 싶은 경우 반복 횟수
    - `product(A, repeat=4) == product(A, A, A, A)`
- 하나 이상의 이터레이터에 들어 있는 아이템들의 데카르트 곱 (cartesian product)를 반환
- 리스트 컴프리헨션을 깊이 내포시키는 대신 이 함수를 사용하면 편함

### permutations(p, r) 

```python
it = itertools.permutations([1, 2, 3, 4], 2)
print(list(it))

>>>
[(1, 2), (1, 3), (1, 4), (2, 1), (2, 3), (2, 4), (3, 1), (3, 2), (3, 4), (4, 1), (4, 2), (4, 3)]

```

- p : iterable
- r : 생성할 순열의 길이, 지정되지 않았다면 p의 길이
- 이터레이터가 내놓는 원소들로부터 만들어낸 길이 r인 순열을 돌려줌
- 원본 이터레이터의 원소가 모두 다 다르다고 가정하고 순열을 만듬
    ```python
    print(list(itertools.permutations([1,2,2])))
    
    >>>
    [(1, 2, 2), (1, 2, 2), (2, 1, 2), (2, 2, 1), (2, 1, 2), (2, 2, 1)]
    
    ```
- 만들어지는 원소의 순서는 원본 이터레이터에서의 원소 인덱스의 오름차순으로 결정

### combinations(p, r)

```python
it = itertools.combinations([1, 2, 3, 4], 2)
print(list(it))

>>>
[(1, 2), (1, 3), (1, 4), (2, 3), (2, 4), (3, 4)]

```

- p : iterable
- r : 생성할 조합의 길이
- 이터레이터가 내놓는 원소들로부터 만들어낸 길이 r인 조합을 돌려줌
- 마찬가지로 원본 이터레이터의 원소가 모두 다 다르다고 가정하고 순열을 만들고, 순서는 원본 이터레이터에서의 원소 인덱스에 의해 결정

### combinations_with_replacement(p, r) —>

```python
it = itertools.combinations_with_replacement([1, 2, 3, 4], 2)
print(list(it))

>>>
[(1, 1), (1, 2), (1, 3), (1, 4), (2, 2), (2, 3), (2, 4), (3, 3), (3, 4), (4, 4)]

```

- p : iterable
- r : 생성할 조합의 길이
- combinations와 같지만 원소의 반복을 허용 → 중복 조합
- 마찬가지로 원본 이터레이터의 원소가 모두 다 다르다고 가정하고 순열을 만들고, 순서는 원본 이터레이터에서의 원소 인덱스에 의해 결정
