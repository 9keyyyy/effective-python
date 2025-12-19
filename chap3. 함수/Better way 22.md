# 22 변수 위치 인자를 사용해 시각적인 잡음을 줄여라
## TL;DR

- 가변적 위치 인자는 가변인자(var args) 또는 스타인자(star args)라고 부름
- def 문에서 *args를 사용하면 함수가 가변 위치 인자를 받을 수 있음
- * 연산자를 사용하면 가변인자를 받는 함수에게 시퀀스 원소들을 전달할 수 있음
- 제너레이터에 *연산자를 사용하면 프로그램이 메모리를 모두 소진하고 중단될 수 있음
- args 를 받는 함수에 새로운 위치 기반 인자를 넣으면 버그가 발생 할 수 있음. 이때는 키워드 기반 인자 사용

## 가변적 위치 인자

- 위치 인자 (positional argument)를 가변적으로 받으면 함수 호출을 더 깔끔하게 할 수 있음
- 가변적인 위치인자는 가변인자 (varargs) or 스타인자 (star args) 라고 부르기도 함
    - *args 라고 이름을 붙이는 경우가 많아서 스타 인자라고 부름

### +) 위치 인자란?

- 함수 인자를 받는 방식에는 2가지가 있음
    1. 위치로 매칭 → 위치 인자
    2. 키워드로 매칭 → 키워드 인자
    
    ```python
    def func(a, b):
          print(a, b, sep='-')
    
    func('py', 'thon') # 위치 인자
    func(b='thon', a='py') # 키워드 인자
    func('py', b='thon') # 혼합도 가능, But 위치 인자가 먼저여야함
    func(a='py', 'thon') # SyntaxError
    
    ```
    

### 예시) 디버깅 정보를 로그에 남기는 경우

> Case: 인자수 고정
> 

```python
def log(message, values):
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('내 숫자는 ', [1, 2])
log('안녕 ', [])

>>
내 숫자는: 1, 2
안녕

```

> Case: 가변 인자 사용
> 

```python
def log(message, *values):  # 마지막 인자 이름 앞에 *추가
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('내 숫자는 ', [1, 2])
log('안녕')

>>
내 숫자는: 1, 2
안녕

```

- 로그에 남길 값이 없는 경우엔 생략하고 메세지만 전달하면 됨
- 시각적 잡음이 없음 
- 언패킹 대입문 (Better Way 13.) 과 비슷하게 사용
    - 중간 위치 가능, 2개 이상 사용 불가능
    
    ```python
    # 가능
    def log(message, *values, code):
    ...
    
    # SyntaxError
    def log(message, *values, *code):
    ...
    
    ```
    
- 가변 위치 인자에 시퀀스(list …)를 사용하고 싶은 경우
    - 연산자는 파이썬이 시퀀스의 원소들을 함수의 위치인자로 넘기도록 함
    
    ```python
    favorites = [7, 33, 99]
    log('좋아하는 숫자는', favorites)
    log('좋아하는 숫자는', *favorites)
    
    >>
    좋아하는 숫자는: [7, 33, 99]
    좋아하는 숫자는: 7, 33, 99
    
    ```
    

## 가변적인 위치 인자의 문제점

### 1. 선택적인 위치 인자가 함수에 전달되기 전, 항상 튜플로 변환됨

- 함수를 호출하는 쪽에서 제너레이터 앞에 * 연산자를 사용하면 제너레이터의 모든 원소를 얻기 위해 반복함
- (Better Way 30: 리스트를 반환하기보다는 제너레이터를 사용하라 참고)
- 이로 인해 메모리를 많이 소비하거나 프로그램이 중단될 수 있음

```python
def my_generator():
    for i in range(10):
        yield i

def my_func(*args):
    print(args)

it = my_generator()
my_func(*it)

>>
(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

```

**Then?**

- args를 받는 함수는 인자 목록에서 가변적인 부분에 들어가는 인자의 개수가 처리하기에 적합할 정도로 적다는 것을 알고 있을 때 사용하는 것이 가장 적절함
- args는 여러 리터럴이나 변수 이름을 함께 전달하는 함수 호출에 이상적
- args는 프로그래머의 편의와 가독성을 위한 기능이니 성능을 잘 생각해야함

### 2. 함수에 새로운 위치 인자를 추가되면 해당 함수를 호출하는 모든 코드를 변경해야함

```python
def log(sequence, message, *values):
    if not values:
        print(f'{sequence} - {message}')
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{sequence} - {message}: {values_str}')

log(1, '좋아하는 숫자는', 7, 33)   # 새 코드에서 가변 인자를 사용. 문제 없음
log(1, '안녕')                   # 새 코드에서 가변 인자 없이 메시지만 사용. 문제 없음
log('좋아하는 숫자는', 7, 33)      # 예전 방식 코드는 깨짐

>>
1 - 좋아하는 숫자는: 7, 33
1 - 안녕
좋아하는 숫자는 - 7: 33

```

- 기존 log 함수에 새로운 위치 인자 (sequence) 추가
- 기존 함수를 호출하던 코드들은 모두 동작을 하지 않기에 전부 수정해야함

**Then?**

- 이런 가능성을 없애기 위해선 *args를 받아들이는 함수를 확장할 때는 키워드 인자만 사용해야함 (Better Way 25: 위치로만 인자를 지정하거나 키워드로만 인자를 지정하게 해서 호출을 명확하게 만들어라 참고)
- 더 방어적으로 하기 위해선 타입 어노테이션 (Better Way 90: typing과 정적 분석을 통해 버그를 없애라 참고) 사용해도 됨
