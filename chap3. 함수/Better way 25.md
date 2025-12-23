# 25 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들라
# TL;DR

1. **키워드로만** 지정해야 하는 인자를 사용하면 호출하는 쪽에서 특정 인자를 (위치를 사용하지 않고) 반드시 키워드를 사용해 호출하도록 강제할 수 있음
    - 함수 호출의 의도를 명확히 할 수 있음
    - 키워드로만 지정해야 하는 인자는 인자 목록에서 * 다음에 위치
2. **위치로만** 지정해야 하는 인자를 사용하면 호출하는 쪽에서 키워드를 사용해 인자를 지정하지 못하게 만들 수 있고, 이에 따라 함수 구현과 함수 호출 지점 사이의 결합을 줄일 수 있음
    - 위치로만 지정해야 하는 인자는 인자 목록에서 / 앞에 위치
3. 인자 목록에서 **`/`와 * 사이에 있는 파라미터는 키워드를 사용해 전달해도 되고 위치를 기반으로 전달**해도 됨 (파이썬 함수 파라미터의 기본 동작)


## Keyword-Only and Positional-Only Arguments

### positional argument로만 이루어진 함수의 경우

```python
def safe_division(number, divisor,
                  ignore_overflow,        # boolean
                  ignore_zero_division):  # boolean
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else: raise

```

함수 호출시 인자 순서에 따라 예외처리 리턴값이 바뀌는 코드

```python
safe_division(1.0, 10**500, True, False)    # 0
safe_division(1.0, 10**500, False, True)    # OverflowError: int too large to convert to float
safe_division(1.0, 0, False, True)          # inf
safe_division(1.0, 0, True, False)          # ZeroDivisionError: float division by zero

```

bool 변수 위치 혼동시 곤란

### keyword argument

따라서 이런경우 아래처럼 키워드인자를 사용하여 가독성을 높이는 것이 좋음

```python
def safe_division_b(number, divisor,
                    ignore_overflow=False,        # Changed
                    ignore_zero_division=False):  # Changed
                    ...

safe_division_b(1.0, 10**500, ignore_overflow=True)   # 0
safe_division_b(1.0, 0, ignore_zero_division=True)    # inf

```

### keyword-only

아래처럼 `*`를 사용하여 keyword-only argument 명시 가능

```python
def safe_division_c(number, divisor, *,  # Changed, * 뒤로는 반드시 keyword-only
                    ignore_overflow=False,
                    ignore_zero_division=False):
                    ...

# *뒤의 인자들은 positional arguments로 대체 불가
safe_division_c(1.0, 10**500, True, False)
>>>
Traceback ...
TypeError: safe_division_c() takes 2 positional arguments but 4  ̄were given
But keyword arguments and their default values will work as expected (ignoring an exception in one case and raising it in another):

# keyword-only arguments
result = safe_division_c(1.0, 0, ignore_zero_division=True)
print(result)
>>>
float

```

### 문제

keyword-only 인자 외의 인자들에 대한 처리

```python
# number, divisor에 대해 위치와 키워드를 혼용
assert safe_division_c(number=2, divisor=5) == 0.4
assert safe_division_c(divisor=5, number=2) == 0.4
assert safe_division_c(2, divisor=5) == 0.4

# 다른 코드에서 키워드 인자 사용해서 함수 호출했는데 앞의 두 인자 이름이 변경된 경우
def safe_division_c(numerator, denominator, *,  # Changed (number->numerator, divisor->denominator)
                    ignore_overflow=False,
                    ignore_zero_division=False):
                    ...

safe_division_c(number=2, divisor=5)
>>>
Traceback ...
TypeError: safe_division_c() got an unexpected keyword argument  ̄'number'

```

### positional-only arguments

- python 3.8부터 적용 가능
- 아래처럼 `/` 를 사용하여 positional-only argument 명시 가능

```python
def safe_division_d(numerator, denominator, /, *,  # Changed, / 앞으로는 반드시 positional-only
                    ignore_overflow=False,
                    ignore_zero_division=False):
                    ...

# 위치 인자로는 정상 작동
safe_division_d(2, 5)
>>>
0.4

# 키워드 시도시 에러
safe_division_d(numerator=2, denominator=5)
>>>
Traceback ...
TypeError: safe_division_d() got some positional-only arguments  ̄passed as keyword arguments: 'numerator, denominator'

```

safe_division_d에서 위치로만 지정하는 인자(numerator, denominator)명을 바꿔도 키워드인자로 사용할 수 없고 위치로만 인자를 받아오기 때문에 문제가 없어짐

### 키워드 인자와 위치 인자 둘다 사용하기

**`/`, `*`사이에 있는 파라미터**들은 위치인자도 될 수 있고 키워드인자도 될 수 있음

```python
def safe_division_e(numerator, denominator, /,
                    ndigits=10, *,
                    ignore_overflow=False,
                    ignore_zero_division=False):
    try:
        fraction = numerator / denominator
        return round(fraction, ndigits)
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
else: raise

result = safe_division_e(22, 7)
print(result)
result = safe_division_e(22, 7, 5)
print(result)
result = safe_division_e(22, 7, ndigits=2)
print(result)
>>>
3.1428571429
3.14286
3.14

```
