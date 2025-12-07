# 10 대입식을 사용해 반복을 피하라
## TL;DR

- 대입식에서는 왈러스 연산자(:=)로 대입과 평가를 동시에 하여 코드의 중복을 줄이자
- Python에 없는 switch/case 문이나 do/while 루프 기능을 대입식으로 대체해보자

### 대입식 `:=`
- 왈러스(Walrus): 바다코끼리 :=
- 대입문이 쓰일 수 없는 위치에서 변수에 값을 대입할 수 있음
    - 변수의 값을 평가하는 위치에서 변수에 값을 대입 = 대입&평가
- Python 3.8~

### Example 1-1: 변수의 반복

```python
fresh_fruit = {
'apple': 10,
'banana': 8,
'lemon': 5,
}

count = fresh_fruit.get('lemon', 0)
if count:
    make_lemonade(count)
else:
    out_of_stock()

```

if문에서만 쓰이는 count 변수, 꼭 따로 선언해야 할까?

- if 앞에서 변수 정의 → 실제보다 더 중요해 보임

```python
(fresh_fruit.get('lemon') and make_lemonade(fresh_fruit['lemon'])) or out_of_stock()

```
→ 가독성 좋지않음

```python
if count := fresh_fruit.get('lemon', 0):
    make_lemonade(count)
else:
    out_of_stock()

```
→ count가 if 문에서만 의미가 있다는 점이 명확 → 가독성 good!

<br/>

### Example 1-2. 변수의 반복 - 값을 비교하는 경우

```python
if (count := fresh_fruit.get('apple', 0)) >= 4:
    make_cider(count)
else:
    out_of_stock()

```

if 문에서 값을 비교하는 경우, **대입식을 괄호로 둘러싸줘야 함**

<br/>

### Example 1-3. 변수의 반복 - 대입 + 비교 + 함수 호출

```python
pieces = 0
# count = fresh_fruit.get('banana', 0)
if (count := fresh_fruit.get('banana', 0)) >= 2:
    pieces = slice_bananas(count)

try:
    smoothies = make_smoothies(pieces)
except OutOfBananas:
    out_of_stock()

```

- count 변수 강조 X
- if문 다음에도 중요한 변수(pieces) 명확
<br/>

### Example 2. switch/case 문 흉내내기

- Python에는 switch/case 문이 없음 → 왈러스 연산자로 비슷하게 구현 가능
- 만들 수 있는 가장 좋은 주스 제공: 바나나 > 사과 > 레몬

```python
count = fresh_fruit.get('banana', 0)
if count >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
else:
    count = fresh_fruit.get('apple', 0)
    if count >= 4:
        to_enjoy = make_cider(count)
    else:
        count = fresh_fruit.get('lemon', 0)
        if count:
            to_enjoy = make_lemonade(count)
        else:
            to_enjoy = 'Nothing'

```

지저분 & 가독성 안좋음

```python
if (count := fresh_fruit.get('banana', 0)) >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
elif (count := fresh_fruit.get('apple', 0)) >= 4:
    to_enjoy = make_cider(count)
elif count := fresh_fruit.get('lemon', 0):
    to_enjoy = make_lemonade(count)
else:
    to_enjoy = 'Nothing'

```

원래 코드보다 들여쓰기와 내포가 줄어듦
<br/>

### Example 3. do while 문 흉내내기

- Python에는 do while 문이 없음 → 왈러스 연산자로 비슷하게 구현 가능
- 과일 바구니에 있는 과일을 모두 주스로 만들고 병에 담기

```python
bottles = []
fresh_fruit = pick_fruit()
while fresh_fruit:
    for fruit, count in fresh_fruit.items():
        batch = make_juice(fruit, count)
        bottles.extend(batch)
    fresh_fruit = pick_fruit()

```

`fresh_fruit = pick_fruit()` 호출 반복

```python
bottles = []
while True:                     # Loop
    fresh_fruit = pick_fruit()
    if not fresh_fruit:         # And a half
        break
    for fruit, count in fresh_fruit.items():
        batch = make_juice(fruit, count)
        bottles.extend(batch)

```

코드 반복은 없앴으나, while 루프의 유용성이 줄어듦

```python
bottles = []
while fresh_fruit := pick_fruit():
    for fruit, count in fresh_fruit.items():
        batch = make_juice(fruit, count)
        bottles.extend(batch)

```

코드 반복 X, 무한 루프 X
