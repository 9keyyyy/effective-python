# 18 __missing__을 사용해 키에 따라 다른 디폴트 값을 생성하는 방법을 알아두라

### TL;DR

- 딕셔너리의 디폴트 값을 만드는 계산 비용이 높거나 예외가 발생하는 상황에서는 `dict` 의 `setdefault` 메서드를 사용하지 마라
- `defaultdict` 에 전달되는 함수는 인자를 받지 않는다. 따라서 접근에 사용한 키 값에 맞는 디폴트 값을 생성할 수 없다
- 디폴트 키를 만들 때 어떤 키를 사용했는지 반드시 알아야 하는 상황이라면 직접 `dict` 의 하위 클래스와 `__missing__` 메서드를 정의

### When?

- `setdefault` 는 키가 없는 경우를 짧은 코드로 처리 (Better way 16)
- `defaultdict` 는 원소가 없는 경우를 처리 (Better way 17)
- 그러나, `setdefault` 나 `defaultdict` 모두 사용하기 적당하지 않은 경우도 있음

### 예시상황

파일 시스템에 있는 프로필 사진을 관리하는 프로그램에서 프로필 사진의 경로와 열린 파일 핸들을 연관시켜주는 딕셔너리

```python
pictures = {}
path = 'profile.png'

if (handle := pictures.get(path)) is None:
     try:
          handle = open(path, 'a+b')
     except OSError:
          print(f'경로를 열수 없습니다')
          raise
     else:
          pictures[path] = handle

handle.seek(0)
image_data = handle.read()
```

- 파일 핸들이 이미 딕셔너리 안에 있으면, 딕셔너리를 단 한번만 읽음
- 파일 핸들이 없으면 `get` 을 사용해 딕셔너리를 한번 읽고 try/except 블록의 else 절에서 핸들을 딕셔너리에 대입
- 이와 같은 방법은 딕셔너리를 많이 읽고 블록 깊이가 더 깊어짐

### setdefault 사용 예시

```python
try:
     handle = pictures.setdefault(path, open(path, 'a+b'))
except OSError:
     print(f'경로를 열수 없습니다')
     raise
else:
     handle.seek(0)
     image_data = handle.read()

```

- 이와 같은 방법은 파일 핸들을 만드는 내장 함수인 `open` 이 딕셔너리에 경로가 있는지 여부와 관계없이 항상 호출
- 이로 인해 같은 프로그램 상에 존재하던 열린 파일 핸들과 혼동될 수 있는 새로운 파일 핸들이 생길 수도 있dma

### defaultdict 사용 예시

```python
from collections import defaultdict

def open_picture(profile_path):
     try:
          return open(profile_path, 'a+b')
     except OSError:
          print(f'경로를 열수 없습니다')
          raise

pictures = defaultdict(open_picture)
handle = pictures[path]
handle.seek(0)
image_data = handle.read()

```

- `TypeError: open_picture() missing 1 positional argument: 'profile_path'`
- `defaultdict` 생성자에 전달한 함수는 인자를 받을 수 없음

### Solution

`dict` 타입의 하위 클래스를 만들고 `__missing__` 특별 메서드를 구현하면 키가 없는 경우를 처리하는 로직을 커스텀화할 수 있음

```python
class Pictures(dict):
     def __missing__(self, key):
          value = open_pictures(key)
          self[key] = value
          return value

pictures = Pictures()
handle = pictures[path]
handle.seek(0)
image_data = handle.read()
```

- `pictures[path]` 라는 딕셔너리 접근에서 `path` 가 딕셔너리에 없으면 `__missing__` 메서드가 호출됨
- 이 메서드는 키에 해당하는 디폴트값을 생성해 딕셔너리에 넣어준 다음에 그 값을 반환

