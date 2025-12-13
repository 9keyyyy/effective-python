# 16 in을 사용하고 딕셔너리 키가 없을 때 KeyError를 처리하기보다는 get을 사용하라

## TL;DR
- 딕셔너리 키가 없는 경우를 처리하는 방법
  - in 식
  - KeyError 예외
  - get 메서드
  - setdefault 메서드
- 카운터와 같이 기본적인 타입의 값이 들어가는 딕셔너리를 다룰 때, 딕셔너리에 넣을 값을 만드는 비용이 비싸거나 만드는 과정에 예외가 발생할 수 있는 경우 get 메서드를 사용하는 것이 좋음
- 해결하려는 문제에 dict의 setdefault 메서드를 사용하는 방법이 가장 적합해 보인다면 setdefault 대신 defaultdict를 사용하는 것 고려

</br>

### 카운터로 이뤄진 딕셔너리
샌드위치 가게의 고객들이 가장 좋아하는 빵에 대한 투표를 저장한 딕셔너리
```python
counters = {
	'품퍼니켈' : 2,
	'사워도우' : 1,
}
```
- 위와 같은 딕셔너리에서 투표가 일어날 때 카운터를 증가시키려면 먼저 키가 딕셔너리에 존재하는지 확인 필요
- 키가 없으면 디폴트 카운터 값인 0을 딕셔너리에 넣고 그 카운터를 증가

#### 1. if 문과 키가 존재할 때 참을 반환하는 in을 사용하는 방법
```python
key = '밀'

if key in counters:
	count = counters[key]
else:
	count = 0

counters[key] = count + 1
```
- 딕셔너리에서 키를 두 번 읽고, 키에 대한 값을 한 번 대입해야 함 ->  비효율적

#### 2. 존재하지 않는 키에 접근할 때 발생하는 KeyError 예외를 활용하는 방법
```python
try:
	count = counters[key]
except KeyError:
	count = 0

counters[key] = count + 1
```
- 키를 한 번만 읽고 값을 한 번만 대입하면 되므로 in에 비해 효율적

#### 3. 그 외 in식과 KeyError를 사용해 짧은 코드를 작성하는 방법
- <방법 1>
  ```python
  if key not in counters:
  	counters[key] = 0
  counters[key] += 1
  ```
- <방법 2>
  ```python
  if key in counters:
  	counters[key] += 1
  else:
  	counters[key] = 1
  ```
- <방법 3>
  ```python
  try:
  	counters[key] += 1
  except KeyError:
  	counters[key] = 1
  ```
대입을 중복 사용해야 하므로 코드의 가독성이 떨어지므로 비효율적

#### 4. dict 내장 타입의 get 메서드 사용하는 방법
```python
count = counters.get(key,0) 
# get의 두 번재 인자는 딕셔너리에 key가 없을 때 돌려줄 디폴트 값

counters[key] = count + 1
```
- 키를 한 번만 읽고 값을 한 번만 대입
- KeyError 방법에 비해 코드가 짧음
- 가장 효율적이고 가독성을 높일 수 있는 방법

#### 5. collection 내장 모듈의 Counter 클래스

카운터로 이뤄진 딕셔너리를 유지해야 하는 경우에는 collection 내장 모듈의 Counter 클래스를 사용하는 것을 고려해보는 것도 좋음

</br>

### 딕셔너리에 저장된 값이 복잡한 경우
어떤 사람이 어떤 유형의 빵에 투표했는지를 저장한 딕셔너리
```python
votes = {
	'바게트' : ['철수', '순이'],
	'치아바타' : ['하니', '유리'],
}
```

#### 1. in을 사용한 방법
```python
key = '브리오슈'
who = '단이'

if key in votes:
	names = votes[key]
else:
	votes[key] = names = []

names.append(who)
print(votes)

>>>
{'바게트': ['철수', '순이'], '치아바타': ['하니', '유리'], '브리오슈': ['단이']
```
- 딕셔너리에서 키를 두 번 읽고, 키가 없`는 경우 값을 한 번 대입하므로 비효율적
- `votes[key] = names = []`
   a = b = c 와 같은 형태의 이중 대입문은 a와 b가 c 값을 가진다는 의미이다.


#### 2. KeyError 예외를 활용하는 방법
```python
try:
	names = votes[key]
except KeyError:
	votes[key] = names = []

names.append(who)
```
- 키를 한 번만 읽고 값을 한 번만 대입하면 되므로 in을 활용한 방법에 비해 효율적이나, get 메소드 활용에 비해 비효율적

#### 3. get 메소드를 사용하는 방법
```python
names = votes.get(key)
if names is None:
	votes[key] = names = []

names.append(who)
# -----
# 추가적으로 BetterWay10. '대입식을 사용해 반복을 피하라' 에서 소개된 왈러스 연산자를 사용하면 더 짧게 가능
if (names := votes.get(key)) is None:
	votes[key] = names = []

names.append(who)
```
- 키를 한 번만 읽고 값을 한 번만 대입
- KeyError 방법에 비해 코드가 짧음 -> 가장 효율적이고 가독성을 높일 수 있음


#### 4. setdefault 메서드를 활용한 방법
```python
names = votes.setdefault(key, [])
names.append(who)
```
- 딕셔너리에서 키를 사용해 값을 가져오려고 시도
- 키가 없으면 제공받은 디폴트 값을 키에 연관시켜 딕셔너리에 대입한 다음 키에 연관된 값을 반환
- 문제점
  ```python
  data = {}
  key = 'foo'
  value = []
  data.setdefault(key, value)
  print('이전:', data)
  value.append('hello')
  print('이후:', data)
  
  >>>
  이전: {'foo': []}
  이후: {'foo': ['hello']}
  ```
  - get과 대입식을 사용하는 것 보다 짧지만 저자는 set이라는 단어가 위 패턴의 동작을 잘 설명하지 못하는 단어라 가독성이 좋지 않다고 판단
  - 키가 없으면 setdefault에 전달된 디폴트 값이 별도로 복사되지 않고 딕셔너리에 직접 대입
  - 호출할 때마다 리스트를 만들어야 해서 성능 저하의 여지 존재
  - setdefault 대신 defaultdict 메서드를 사용할 것 고려
