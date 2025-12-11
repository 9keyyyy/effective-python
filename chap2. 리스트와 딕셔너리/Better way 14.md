# 14 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라
## TL;DR

- sort의 key 값을 통해 리스트에서 원소 대신 비교에 사용할 객체를 반환할 수 있음
- key에서 튜플을 반환해 여러 정렬 기준 사용 가능 (숫자의 경우 부호를 바꿔 정렬 순서 변경 가능)
- 부호를 바꿀 수 없다면 sort를 여러 번 호출 (우선순위가 점점 높아지는 순서로 호출)


### list.sort()의 기본 동작

'자연스러운 순서'를 판단할 수 있는 타입(정수, 문자열, 부동소수점 등)을 오름차순으로 정렬

```python
numbers = [93, 86, 11, 68, 70]
numbers.sort()

>>>
[11, 68, 70, 86, 93]
```

### '자연스러운 순서'가 없다면, `sort` 의 `key` 파라미터 사용

`lambda` 함수를 통해 원소 애트리뷰트에 접근, 인덱스로 접근 가능

```python
class Tool:
    def __init__(self, name, weight):
        self.name = name
        self.weight = weight

    def __repr__(self):
        return f'Tool({self.name!r}, {self.weight})'

tools = [
    Tool('수준계', 3.5),
    Tool('해머', 1.25),
    Tool('스크류드라이버', 0.5),
    Tool('끌', 0.25),
]

tools.sort(key=lambda x: x.name)
print('정렬: ', tools)

>>>
정렬:  [Tool('끌', 0.25), Tool('수준계', 3.5), Tool('스크류드라이버', 0.5), Tool('해머', 1.25)]
```
```
tools.sort(key=lambda x: x.weight)
print('무게순 정렬:', tools)

>>>
무게순 정렬: [Tool('끌', 0.25), Tool('스크류드라이버', 0.5), Tool('해머', 1.25), Tool('수준계', 3.5)]
```

### tuple 타입을 이용하여 여러 기준으로 정렬

- tuple을 비교하면, tuple의 각 위치를 iteration하여, 각 인덱스에 해당하는 원소를 비교
  ```python
  drill = (4, '드릴')
  sander = (4, '연마기')
  assert drill[0] == sander[0] # 무게가 같다
  assert drill[1] < sander[1]  # 알파벳순으로 볼 때 더 작다
  assert drill < sander        # 그러므로 드릴이 더 먼저다
  ```
- 우선순위에 따라 key 함수에 튜플 형태로 반환
  ```python
  power_tools = [
      Tool('드릴', 4),
      Tool('원형 톱', 5),
      Tool('착암기', 40),
      Tool('연마기', 4),
  ]
  
  power_tools.sort(key=lambda x: (x.weight, x.name))
  print(power_tools)
  
  >>>
  [Tool('드릴', 4), Tool('연마기', 4), Tool('원형 톱', 5), Tool('착암기', 40)]
  ```
- 오름차순 / 내림차순은 순서가 동일하게 적용
  ```python
  power_tools.sort(key=lambda x: (x.weight, x.name),
                   reverse=True) # 모든 비교 기준을 내림차순으로 만든다
  print(power_tools)
  
  >>>
  [Tool('착암기', 40), Tool('원형 톱', 5), Tool('연마기', 4), Tool('드릴', 4)]
  ```
- 숫자 값은 `-` 연산자로 정렬 방향 반전 가능
  ```python
  power_tools.sort(key=lambda x: (-x.weight, x.name))
  print(power_tools)
  
  >>>
  [Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]
  ```

### `sort`를 여러 번 호출하여 여러 기준으로 정렬하기

`sort`를 여러 번 호출해도 이전 순서를 기본적으로 유지

```python
power_tools.sort(key=lambda x: x.name)   # name 기준 오름차순
power_tools.sort(key=lambda x: x.weight, # weight 기준 내림차순
                 reverse=True)

>>>
[Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]
```

> 우선순위 역순으로 호출해야 원하는 정렬 순서 얻을 수 있음에 주의
>
