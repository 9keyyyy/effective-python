# 29 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라
## TL;DR

- 대입식 `:=`을 컴프리헨션이나 제너레이터의 **조건** 부분에 사용
- 컴프리헨션이나 제너레이터 내부에서 재사용 가능

## 대입식을 컴프리헨션에 사용하는 경우

- **같은 계산을 여러 위치에서 공유하는 경우**

```python
stock = {
    "못": 125,
    "나사못": 35,
    "나비너트": 8,
    "와셔": 24,
}

order = ["나사못", "나비너트", "클립"]

def get_batches(count, size):
    return count // size

```

```python
found = {name: get_batches(stock.get(name, 0), 8)
         for name in order
         if get_batches(stock.get(name, 0), 8)}
print(found)

>>>
{'나사못': 4, '나비너트': 1}

```

- 가독성 나쁨
- 두 식을 동시에 변경해야 하므로 실수할 가능성 높음

```python
found = {name: batches for name in order
         if (batches := get_batches(stock.get(name, 0), 8))}
print(found)

>>>
{'나사못': 4, '나비너트': 1}

```

- 왈러스 연산자 `:=`를 이용, 조건문에 사용
- order 마다 get_batches를 한 번만 호출해서 변수에 저장하므로 불필요한 연산량 감소

## 대입식을 조건 부분에서만 사용하는 게 좋은 이유

- 값에 사용은 가능하나, 조건에 해당 변수 사용 시 컴프리헨션 평가 순서로 인해 오류 발생

```python
result = {name: (tenth := count // 10)
          for name, count in stock.items() if tenth > 0}

```

- 값에 사용할 경우, 루프 밖으로 해당 변수 누출

```python
half = [(last := count // 2) for count in stock.values()]
print(f'{half}의 마지막 원소는 {last}')

>>>
[62, 17, 4, 12]의 마지막 원소는 12

```

- 조건문에 사용할 경우, 변수 누출 X

```python
half = [count // 2 for count in stock.values()]
print(half)  # 작동함
print(count) # 루프 변수가 누출되지 않기 때문에 예외가 발생함

>>>
[62, 17, 4, 12]
NameError: name 'count' is not defined.

```

## 제너레이터도 똑같은 방식으로 작동
```python
order = ['나사못', '나비너트', '클립']

found = ((name, batches) for name in order
         if (batches := get_batches(stock.get(name, 0), 8)))
print(next(found))
print(next(found))

>>>
('나사못', 4)
('나비너트', 1)

```
