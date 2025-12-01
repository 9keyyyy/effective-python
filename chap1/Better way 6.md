# 06 인덱스를 사용하는 대신 대입을 사용해 데이터를 언패킹하라
## TL;DR

1. 파이썬은 한 문장 안에서 여러 값을 대입할 수 있는 언패킹이라는 특별한 문법을 제공
2. 파이썬 언패킹은 일반화돼 있으므로 모든 이터러블에 적용 가능 + 여러 계층으로 내포된 경우에도 언패킹을 적용 가능
3. 인덱스를 사용해 시퀀스 내부에 접근하는 대신 언패킹을 사용해 시각적인 잡음을 줄이고 코드를 더 명확하게 만드는게 좋음

## 튜플 소개

```
snack_calories = {
	'감자칩': 140,
	'팝콘': 80,
	'땅콩': 190,
}
items = tuple(snack_calories.items())
print(items)

>>>
(('감자칩', 140), ('팝콘', 80), ('땅콩', 190))

```

```
item = ('호박엿', '식혜')
first = item[0]
second = item[1]
print(first, '&', second)

>>>
호박엿 & 식혜

```

```
pair = ('약과', '호박엿')
pair[0] = '타래과'

>>>
Traceback ...
TypeError: 'tuple' object does not support item assignment

```

## 언패킹 소개

```
item = ('호박엿', '식혜')
first, second = item  # 언패킹
print(first, '&', second)

>>>
호박엿 & 식혜

```
- 언패킹은 튜플 인덱스를 사용하는 것보다 시각적인 잡음이 적음
- 리스트, 시퀀스, 이터러블(iterable) 안에 여러 계층으로 이터러블이 들어간 경우 등 다양한 패턴을 언패킹 구문에 사용할 수 있음
- 아래과 같이 코드를 작성하는 것을 추천하지는 않지만, 가능은 함
  ```
  favorite_snacks = {
  	'짭조름한 과자': ('프레즐', 100),
  	'달콤한 과자': ('쿠키', 180),
  	'채소': ('당근', 20),
  }
  
  ((type1, (name1, cals1)),
   (type2, (name2, cals2)),
   (type3, (name3, cals3))) = favorite_snacks.items()
  
  print(f'Favorite {type1} is {name1} with {cals1} 칼로리')
  print(f'Favorite {type2} is {name2} with {cals2} 칼로리')
  print(f'Favorite {type3} is {name3} with {cals3} 칼로리')
  
  >>>
  Favorite 짭조름한 과자 is 프레즐 with 100 칼로리
  Favorite 달콤한 과자 is 쿠키 with 180 칼로리
  Favorite 채소 is 당근 with 20 칼로리
  
  ```


#### 안좋은 예시

```
def bubble_sort(a):
	for _ in range(len(a)):
		for i in range(1, len(a)):
			if a[i] < a[i-1]:
				temp = a[i]
				a[i] = a[i-1]
				a[i-1] = temp

names = ['프레즐', '당근', '쑥갓', '베이컨']
bubble_sort(names)
print(names)

>>>
['당근', '베이컨', '쑥갓', '프레즐']

```

#### 언패킹 예시

```
def bubble_sort(a):
	for _ in range(len(a)):
		for i in range(1, len(a)):
			if a[i] < a[i-1]:
				a[i-1], a[i] = a[i], a[i-1]  # 맞바꾸기

names = ['프레즐', '당근', '쑥갓', '베이컨']
bubble_sort(names)
print(names)

>>>
['당근', '베이컨', '쑥갓', '프레즐']

```

#### for 루프 또는 그와 비슷한 다른 요소(컴프리헨션(comprehension)이나 제너레이터 식)의 대상인 리스트의 원소를 언패킹
- 먼저 언패킹을 사용하지 않고 간식이 들어 있는 list에 대해 이터레이션(iteration)하는 코드 -> snacks 구조 내부의 깊숙한 곳에 있는 데이터를 인덱스로 찾으려면 코드가 길어짐
  ```
  snacks = [('베이컨', 350), ('도넛', 240), ('머핀', 190)]
  for i in range(len(snacks)):
  	item = snacks[i]
  	name = item[0]
  	calories = item[1]
  	print(f'#{i+1}: {name}은 {calories} 칼로리이다.')
  
  >>>
  #1: 베이컨은 350 칼로리이다.
  #2: 도넛은 240 칼로리이다.
  #3: 머핀은 190 칼로리이다.
  
  ```
- enumerate 내장 함수(chap 7)와 언패킹을 사용
  ```
  for rank, (name, calories) in enumerate(snacks, 1):
  	print(f'#{rank}: {name}은 {calories} 칼로리이다.')
  
  >>>
  #1: 베이컨은 350 칼로리이다.
  #2: 도넛은 240 칼로리이다.
  #3: 머핀은 190 칼로리이다.
  ```

