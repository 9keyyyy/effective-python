# 09 for나 while 루프 뒤에 else 블록을 사용하지 말라
## TL;DR

- 파이썬에서는 for나 while 루프 바로 뒤에 else를 붙히는 문법을 제공
    - 루프 도중 break을 만나지 않은 경우 실행
- 하지만 헷갈리니 사용하지 말자

## 루프이후 else 문법

- 루프 (for/while) 블록이후 바로 다음에 else 블록을 추가 가능
- 앞 루프가 정상적으로 끝까지 완료되는 경우에 else 블록이 실행
    - try/except/else, try/finally, if/else 에서의 쓰임과 다름
      -> 루프가 정상적으로 완료되지 않으면 이 블록을 실행하라는 뜻으로 가정하기 쉽지만, 완전히 반대로 동작

### 동작 예시

else 블록 실행

```python
for i in range(3):
    print('Loop', i)
else:
    print('Else block!)```
>>>
Loop 0
Loop 1
Loop 2
Else block!

```

else 블록 미실행

```python
for i in range(3):
    print('Loop', i)
    if i == 1:
        break
else:
    print('Else block!)```
>>>
Loop 0
Loop 1

```

루프 조건 자체가 미성립이어도 else 블록이 실행

```python
for x in []:
    print('이 줄은 실행되지 않음')
else:
    print('For Else block!)```
>>>
For Else block!

```

```python
while False:
    print('이 줄은 실행되지 않음')
else:
    print('While Else block!)```
>>>
While Else block!

```

## 위 문법이 사용되는 곳
- 루프를 통해 검색을 하는 경우에 유용
- 대표적 예시는 두 수가 coprime(서로의 공약수가 1만 있음)하는 것을 검사하는 코드

```python
a, b = 4, 9
for i in range(2, min(a, b) +1):
    print('검사 중', i)
    if a % i ==0 and b % i == 0:
        print('Not Coprime')
        break
else:
    print("Coprime")
>>>
검사 중 2
검사 중 3
검사 중 4
Coprime

```

## 문제 및 대안

- 어떻게 동작하는지 명확하지 않음
- 아래처럼 도우미 함수를 작성하는것이 더 적절함
- 아래 함수들이 루프 뒤 else를 바로 붙히는 것 보다 훨씬 더 명확함

#### 방법 1. 조건을 찾으면 바로 함수를 반환/그대로 끝나면 default 값 반환

```python
def coprime(a, b):
    for i in range(2,min(a,b)+1):
        if a % i == 0 and b % i == 0:
            return False
    return True
assert coprime(4, 9)
assert not coprime(3, 6)
```

#### 방법 2. 루프 내에서 원하는 대상을 찾았는지 나타내는 결과 변수 도입
```python
def coprime_alternate(a, b):
    is_comprime = True
    for i in range(2,min(a,b)+1):
        if a % i== 0 and b % i == 0:
            is_coprime = False
            break
    return is_coprime
assert coprime_alternate(4, 9)
assert not coprime_alternate(3, 6)
```
