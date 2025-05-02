# 05 복잡한 식을 쓰는 대신 도우미 함수를 작성하라
## TL;DR
- 파이썬 문법을 사용하면 복잡한 로직도 간결하게 작성할 수 있음
	- 그러나 가독성이 나빠질 수 있음
- 코드를 짧게 만들기보다는 가독성을 중시하고, Don't Repeat Yourself (DRY) 원칙을 따를 것
	- 불(boolean) 연산자 `or` 나 `and`를 식에 사용하는 것보다 `if/else` 식을 쓰는 것이 가독성이 좋음
	- 복잡한 식이나 반복 사용하는 로직은 꼭 도우미 함수(helper function)로 옮길 것

## 예시: URL 질의 문자열(query string) 구문 분석(parsing)
- 파라미터별 상태
	- 어떤 파라미터는 여러 값이, 다른 파라미터에는 하나의 값이 있을 수 있음
	- 어떤 파라미터는 이름은 있지만 값이 없고, 다른 파라미터는 아예 존재하지 않을 수도 있음
```python
from urllib.parse import parse_qs

my_values = parse_qs('빨강=5&파랑=0&초록=',
                     keep_blank_values=True)
print(repr(my_values))

# Output
# {'빨강': ['5'], '파랑': ['0'], '초록': ['']}
```
- 결과 딕셔너리에 `get` 메서드를 사용하는 경우
	- 파라미터별 상태에 따라 다른 값이 반환됨
```python
print('빨강:', my_values.get('빨강'))
print('초록:', my_values.get('초록'))
print('투명도:', my_values.get('투명도'))

# Output
# 빨강: ['5']
# 초록: ['']
# 투명도: None
```

### 이슈: 디폴트 값 대입 및 정수 변환
- 기대 결과
	- 파라미터가 없거나 비어있는 경우 0을 디폴트 값으로 대입
	- 모든 파라미터 값을 정수로 변환해서 즉시 수식에 활용
- 고려 방법
	- 방법 1: `if` 식(expression)
	- 방법 2: `if/else` 조건식(=삼항 조건식)
	- 방법 3: 완전한 `if/else` 문(statement)
	- 방법 4: 도우미 함수

### 방법 1. `if` 식(expression)
```python
# 질의 문자열이 '빨강=5&파랑=0&초록='인 경우
red = my_values.get('빨강', [''])[0] or 0
green = my_values.get('초록', [''])[0] or 0
opacity = my_values.get('투명도', [''])[0] or 0

# Output
# 빨강: '5'
# 초록: 0
# 투명도: 0
```
- 딕셔너리 `get` 메서드 (Better Way 16 참조)
	- 딕셔너리 내에 키가 없으면 두 번째 인자를 반환
- 따라서 위 코드는 아래와 같이 작동함
	- `'빨강'`
		- 딕셔너리 내에 키가 있으므로 `get` 메서드가 `['5']`를 반환
		- `or` 식의 좌측 값인 `['5'][0]`, 즉 `'5'`는 `True`로 평가되므로 `red`에 `'5'`를 대입
	- `'초록'`
		- 딕셔너리 내에 키가 있으므로 `get` 메서드가 `['']`를 반환
		- `or` 식의 좌측 값인 `[''][0]`, 즉 빈 문자열 `''`는 `False`로 평가되므로 `green`에 `0`을 대입
	- `'투명도'`
		- 딕셔너리 내에 키가 없으므로 `get` 메서드가 디폴트 값인 `['']`를 반환
		- 이후는 `'초록'`과 같은 방식으로 처리됨
- 방법 1의 단점
	- 가독성이 떨어짐
	- 파라미터 정수 변환 같은 추가 작업을 하기에는 더욱 불편함
```python
red = int(my_values.get('빨강', [''])[0] or 0)
```

### 방법 2. `if/else` 조건식(=삼항 조건식)
- (아주 복잡한 경우가 아니므로) `if/else` 조건식이 코드를 방법 1보다 명확하게 해줌
```python
red_str = my_values.get('빨강', [''])
red = int(red_str[0]) if red_str[0] else 0
```

### 방법 3. 완전한 `if/else` 문(statement)
- 방법 2보다 더 명확한 구조
```python
green_str = my_values.get('초록', [''])
if green_str[0]:
    green = int(green_str[0])
else:
    green = 0
```

### 방법 4. 도우미 함수
- 위 로직을 반복 적용하려면 꼭 도우미 함수를 작성해야 함
- 도우미 함수를 써서 작성한 코드가 훨씬 명확함
```python
def get_first_int(values, key, default=0):
    found = values.get(key, [''])
    if found[0]:
        return int(found[0])
    return default

green = get_first_int(my_values, '초록')
```
