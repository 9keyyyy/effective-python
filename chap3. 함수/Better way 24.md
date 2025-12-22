# 24 None과 독스트링을 사용해 동적인 디폴트 인자를 지정하라
## **TL;DR**

- `None`과 `docstring`을 사용해 동적인 디폴트인자를 지정할 것
- 디폴트인자는 **함수가 정의된 모듈이 로드되는 시점**에 정해짐

## When?

정의한 함수의 디폴트인자로, **가변(mutable) 변수**를 넣어야 할 때

→ **함수의 디폴트인자로 동적인 값을 사용해야 할 때**


###  가변/불변

- **가변 타입(Mutable) 예시 : 리스트(list), 딕셔너리(dict), 집합(set)**
    - 함수가 호출될 때마다 기본 인자로 사용된 가변 객체는 동일한 메모리 주소를 참조하므로, 한 곳에서 변경이 일어나면 다른 모든 곳에 영향을 줌
    - 여러 함수나 스레드에서 동일한 가변 객체에 접근할 때는 동시성 문제를 고려해야 함
- **불변 타입(Immutable) 예시: 정수(int), 부동소수점(float), 문자열(str), 튜플(tuple), 불리언(bool)**
    - 생성된 후에는 변경할 수 없으며 값을 변경하려고 시도하면 새로운 객체가 생성되고, 원래 객체는 변경되지 않음

## **문제상황 예시**

### **동적 타입의 함수가 디폴트인자로 사용된 상황**

- 로그 메시지를 시간과 함께 나타내어야하는 상황(=함수의 호출시간 출력)
  ```python
  from time import sleep
  from datetime import datetime
  
  def log(message, when=datetime.now()):
  	print(f'{when}: {message}')
  
  log('안녕!')
  sleep(0.1)
  log('두번째 안녕!')
  
  >>>
  2024-02-14 20:39:27.216318: 안녕!
  2024-02-14 20:39:27.216318: 두번째 안녕!
  
  ```
- 위 파이썬 스크립트 파일을 실행하면 `__main__`모듈로서 실행되는데 해당 모듈이 로드되는 시점에 `log`함수가 정의
- 그 정의시점에 `datetime.now()`함수가 “단 한번만” 실행이 되고 해당 값이 디폴트값으로 고정
- **그래서 함수를 시간 차이를 두고 두번 호출하여도 로그 시간이 변하지 않음**

### **가변(mutable) 변수 list가 디폴트인자으로 사용된 상황**

디폴트값이 빈 리스트인 `target_list`에 특정 원소를 추가하려는 상황

```python
def append_to_list(element, target_list=[]):
    target_list.append(element)
    return target_list

print(append_to_list(1))
print(append_to_list(2))

>>>
[1]
[1,2] # 기대 출력 : [2]

```

- 첫번째 문제상황과 동일하게 스크립트 실행시점에 “단 한번만” target_list의 디폴트인자가 빈 리스트 [] 가 생성
- 이 인자는 mutable하기 때문에 디폴트인자를 사용하는 **함수의 호출 때마다 공유**

## 해결 방안

1. **디폴트인자로 None을 사용** → 디폴트인자를 사용한다는 “사실”만 넘기자
2. **부족한 가독성을 docstring을 통해 해결**

### **동적 타입의 함수가 디폴트인자로 사용된 상황**

- 아래와 같은 방법은 함수 정의시점에 디폴트인자가 None으로 고정
- 디폴트인자를 사용할 시 None이 사용되어 디폴트인자 사용 여부가 함수내부로 전달된고 함수 호출마다 `datetime.now` 함수가 실행

```python
def log(message, when=None):
	"""메시지와 타임스탬프를 로그에 남긴다.

	Args:
		message: 출력할 메시지
		when: 메시지가 발생한 시각(datetime).
			디폴트 값은 현재시간이다.

	"""

	if when is None:
		when = datetime.now()
	print(f'{when}: {message}')

```

### **가변(mutable) 변수 list가 디폴트인자으로 사용된 상황**

```python
def append_to_list(element, target_list=None):
	"""타겟 리스트에 입력 원소를 추가한다.

	Args:
		element: 추가할 원소
		target_list: 원소를 추가할 리스트
			디폴트 값은 빈리스트([])이다.

	Returns:
		원소를 추가된 타겟 리스트를 리턴한다.

	"""
    if target_list is None:
        target_list = []
    target_list.append(element)
    return target_list

print(append_to_list(1))
print(append_to_list(2))

>>>
[1]
[2]

```
