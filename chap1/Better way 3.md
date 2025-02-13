# 03 bytes와 str의 차이를 알아두라
## 두 가지 문자열 종류, bytes(8비트 값 순열)와 str(유니코드 코드 포인트[^1] 순열)

bytes 예시
```python
a = b'h\x65llo'
print(list(a))
print(a)
```
```bash
>>>
[104, 101, 108, 108, 111]
b'hello'
```

str 예시
```python
a = 'a\u0300 propos'
print(list(a))
print(a)
```
```bash
>>>
['a', '`', ' ', 'p', 'r', 'o', 'p', 'o', 's']
à propo
```

- str -> bytes: encode 메서드 사용하여 이진 인코딩 방식 부여
- bytes -> str: decode 메서드 사용하여 텍스트 인코딩 방식 부여
- 대부분의 경우 UTF-8이 system default encoding이나 반드시 그러한 것은 아님 (아래 코드로 확인해 보자)
   - ```python3 -c 'import local; print(locale.getpreferredencoding())'``` 

## 문자열 처리를 위한 (형변환) 도우미 함수

유니코드 샌드위치: 
- bytes와 str을 혼용하면 중간 코드에서 헷갈릴 수 있으므로 문자열 처리 구조 가장 앞과 가장 마지막에 아래 두 도우미 함수 이용
- 이를 통해 다양한 인코딩 방식에 대해 통일된 문자열 처리 코드 사용 가능

str로 변환 보장
```python
def to_str(bytes_or_str):
    if isinstance(bytes_or_str, bytes):
    	  value = bytes_or_str.decode("utf-8)
    else:
    	  value = bytes_or_str
    return value
    
print(repr(to_str(b"foo")))
print(repr(to_str("bar")))
print(repr(to_str(b"\xed\x95\x9c")))
```
```bash
>>>
"foo"
"bar"
"한"
```

bytes로 변환 보장
```python
def to_bytes(bytes_or_str):
    if isinstance(bytes_or_str, str):
    	value = bytes_or_str.encode("utf-8")
    else:
    	value = bytes_or_str
    return value

print(repr(to_bytes(b"foo")))
print(repr(to_bytes("bar")))
print(repr(to_bytes("한글")))
```
```bash
>>>
b'foo'
b'bar'
b'\xed\x95\x9c\xea\xb8\x80'
```

## 연산자 불호환
- 같은 형 끼리는 +와 comparator 사용 가능하나 bytes와 str를 한 수식 내 operand로 섞어 쓰는 것 불가
- format string의 경우 bytes 내 str을 박는 것은 안되나 str 내 bytes를 박을 순 있고, 이러한 경우 아래와 같이 출력[^2]
```python
print('red %s' % b'blue')
```
```bash
red b'blue'
```

## Bytes의 파일 입출력 모드는 'rb', 'wb'
```python
with open('data.bin', 'w') as f:
   f.wrtie(b'....')
```
```bash
Traceback...
TypeError:...
```

단, 'r', 'w'로 쓰고 싶다면 'encoding'을 명시하면 된다.
```python
with open('data.bin', 'r', encoding='cp1252') as f:
   data = f.read()
```

## 유니코드 파일 입출력시 encoding 명시 추천
System default encoding을 신뢰하는 것은 향후 오류를 야기할 수 있으므로 encoding은 항상 명시적으로 주는 것이 좋음

[^1]: 코드 포인트(예: U+1F600)는 정말로 코드이며, 인코딩은 이러한 코드 포인트를 특정 문자(예: 'A')로 변환하는 것
[^2]: 이는 bytes 인스턴스가 ```__repr__``` 메서드를 갖고 있기 때문
