# 30 리스트를 반환하기보다는 제너레이터를 사용하라

## TL;DR
- 제너레이터를 활용하면 결과를 리스트에 합쳐서 반환하는 것보다 깔끔
- 제너레이터가 반환하는 이터레이터는 제너레이터 함수의 본문에서 yield가 반환하는 값들로 이뤄진 집합을 만들어냄
- 제너레이터를 사용하면 작업 메모리에 모든 입출력을 저장할 필요가 없어 입력이 아주 커도 출력 시퀀스 만들 수 있음



## 리스트를 반환하기 vs. 제너레이터 사용하기

### 예시1: 문자열에서 찾은 단어 인덱스 반환

**1) 리스트 반환하기**

```python
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index + 1)
    return result

address = '컴퓨터(영어: Computer, 문화어: 콤퓨터, 순화어:전산기)는 진공관'
result = index_words(address)
print(result[:10])

>>>
[0, 8, 18, 23, 28, 38]

```

문제점
- **코드의 핵심을 알아보기 힘듦**
    - 반환할 리스트와 상호작용하는 부분이 많아 함수의 핵심 기능이 강조되지 않음
        - 리스트에 추가할 값(`index + 1`)이 중요한데, 메서드 호출(`result.append`)이 두드러져 보임
        - 결과 리스트를 만드는 줄, 반환하는 줄이 필요
- **반환하기 전에 리스트에 모든 결과를 다 저장해야 함**
    - 입력이 매우 크면 프로그램이 메모리 소진으로 중단될 수 있음

**2) 제너레이터 사용하기**

```python
def index_words_iter(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1

it = index_words_iter(address)
print(next(it))
print(next(it))

>>>
0
8

```


- 제너레이터는 `yield` 식을 사용하는 함수에 의해 생성됨
- 위 함수가 호출되면 제너레이터 함수가 실제 실행되지 않고 즉시 이터레이터를 반환
- 이터레이터는`next` 내장 함수를 호출할 때마다 제너레이터 함수를 다음 `yield` 식까지 진행시킴
- 제너레이터가 `yield`에 전달하는 값은 이터레이터에 의해 호출하는 쪽에 반환됨
- 리스트가 필요하다면?
    - 제너레이터가 반환하는 이터레이터를 `list()`에 넘기면 간단하게 리스트로 변환 가능(Better Way 32 참조)
    - `result = list(index_words_iter(address)) print(result[:10]) >>> [0, 8, 18, 23, 28, 38]`

      
#### 주의
- 제너레이터가 반환하는 이터레이터에는 상태가 있기 때문에, 호출하는 쪽에서 재사용 불가(Better way 31 참조)


### 예시2: 파일을 한 번에 한 줄씩 읽어 한 단어씩 출력

**제너레이터 사용**

```python
def index_file(handle):
    offset = 0
    for line in handle:
        if line:
            yield offset
        for letter in line:
            offset += 1
            if letter == ' ':
                yield offset

```

### 결론
- 결과로 시퀀스를 생성하는 함수를 만들 때, 리스트를 반환하기보다는 **제너레이터 사용을 추천**
- 리스트 반환 방식의 문제점
    - 반환할 리스트와 상호작용하는 부분이 많아 코드의 핵심을 알아보기 힘듦
    - 반환하기 전에 리스트에 모든 결과를 다 저장해야 함
        - 입력이 매우 크면 프로그램이 메모리 소진으로 중단될 수 있음
- **제너레이터(generator)**
    - 동작
        - 제너레이터는 `yield` 식을 사용하는 함수에 의해 생성됨
        - 함수가 호출되면 제너레이터 함수가 실제 실행되지 않고 즉시 이터레이터(iterator)를 반환
        - 이터레이터는 제너레이터 함수의 본문에서 `yield`가 반환하는 값들로 이뤄진 집합을 생성함
        - 이터레이터는`next` 내장 함수를 호출할 때마다 제너레이터 함수를 다음 `yield` 식까지 진행시킴
        - 제너레이터가 `yield`에 전달하는 값은 이터레이터에 의해 호출하는 쪽에 반환됨
    - 장점
        - 가독성이 높고 깔끔함
        - 작업 메모리에 모든 입출력을 저장할 필요가 없음 -> 입력이 아주 커도 출력 시퀀스 생성 가능
    - 주의사항
        - 제너레이터가 반환하는 이터레이터에는 상태가 있기 때문에, 호출하는 쪽에서 재사용 불가

- 함수의 작업 메모리가 입력 중 가장 긴 줄의 길이로 제한되므로 메모리 소진 위험 감소
- 실행 결과(`islice` 함수: Better Way 36 참조)
    - `import itertools with open('address.txt', 'r', encoding='utf-8') as f: it = index_file(f) results = itertools.islice(it, 0, 10) print(list(results)) >>> [0, 8, 18, 23, 28, 38]`
