# 07 range보다는 enumerate를 사용하라
## TL; DR

- range 보다는 enumerate를 사용

## range 함수

- range 함수: 정수 집합을 iteration하는 루프가 필요할 때 유용

```python
from random import randint

random_bits = 0
for i in range(32):
    if randint(0, 1):
        random_bits |= 1 << i

print(bin(random_bits))

>>>
0b1100100010010110101110111100001
```

- iteration 할 대상 데이터 구조가 있으면 이 시퀀스에 대해 바로 루프를 돌 수 있음
- 리스트를 이터레이션 하면서 리스트의 몇 번째 원소를 처리 중인지 알아야 할 때가 있음

```python
favor list = ['바닐라', '초콜릿', '피칸', '딸기']
for i in range(len(flavor_list)):
    flavor = flavor_list[i]
    print (f'{i+1}: {flavor}')

>>>
1: 바닐라
2: 초콜릿
3: 피칸
4: 딸기
```

## range 보다는 enumerate를 사용하라

- range를 사용할 경우 다소 투박한 느낌. list의 길이를 알아야 하고, index를 사용해 배열 원소에 접근해야 함
- enumerate 내장 함수 사용
- enumerate는 루프 인덱스와 iterator의 다음 값으로 이루어진 쌍을 넘겨줌
  ```python
  it = enumerate(flavor_list)
  print(next(it))
  print(next(it))
  
  >>> 
  (0, '바닐라')
  (1, '초콜릿')
  ```
- enumerate가 넘겨주는 각 쌍을 for문과 활용하면 간결하게 언패킹이 가능
- enumerate의 두 번째 파라미터로 어디부터 수를 세기 시작할지 지정 가능
  ```python
  for i, flavor in enumerate(flavor_list, 2):
      print(f'{i+1}: {flavor}')
  
  >>>
  3: 피칸
  4: 딸기
  ```
