# 04 C 스타일 형식 문자열을 str.format과 쓰기보다는 f-문자열을 통한 인터폴레이션을 사용하라
## 형식화(formatting)
미리 정의된 문자열에 데이터 값을 끼워 넣어 사람이 보기 좋은 문자열로 저장하는 과정

### 1. C 스타일 형식 문자열
- %형식화 연산자 사용
- 문제점 1: 오른쪽 tuple 내 데이터 값 순서, 타입 변경 시 오류 발생 가능성
    ```old_way = '%-10s = %.2f' % (key, value)```
- 문제점 2: 형식화 전 값을 변경할 경우 가독성 감소
    ```print('#%d: %-10s = %d' % (i + 1, item.title(), round(count)))```
- 문제점 3: 같은 값을 사용하기 위해 tuple에서 같은 값 여러 번 반복 -> 실수, 오류 가능성 up
- 문제점 4: 위 문제를 해결하기 위한 dictionary 사용 시, 문장의 번잡함 증가
    ``` new_way = '%(key)-10s = %(value).2f' % {'key': key, 'value': value}```
    - 형식 지정자 + 딕셔너리 키 (+ 변수 명)까지 세 번 이상 같은 문자의 반복
    

### 2. 내장 함수 format과 str.format
- python 3부터 도입
- str.format 은 %형식화 연산자 대신 위치 지정자 {} 사용
- index 활용해 인자의 순서 변경 가능 [문제점 1 해결]
    ```formatted = '{1} = {0}'.format(key, value)```
- index 중복 사용 가능 [문제점 3 해결]
    ```formatted = '{0}는 음식을 좋아해. {0}가 요리하는 모습을 봐요.'.format(name)```
- 가독성 문제는 여전히 존재

### 3. f-문자열 
- python 3.6부터 도입
- interploation을 통한 형식 문자열: 값을 직접 문자열 안에 작성
- 키와 값을 불필요하게 중복 지정할 필요 없음 [문제점 4 해결]
    ```f_string = f'{key:<10} = {value:.2f}'```
- 위치 지정자 중괄호 내부에도 **파이썬 식**을 쓸 수 있어 간결함 [문제점 2 해결]
    ```f_string = f'#{i+1}: {item.title():<10s} = {round(count)}'```

### 한 눈에 비교
```
f_string = f'{key:<10} = {value:.2f}'

c_tuple  = '%-10s = %.2f' % (key, value)

str_args = '{:<10} = {:.2f}'.format(key, value)

str_kw   = '{key:<10} = {value:.2f}'.format(key=key, value=value)

c_dict   = '%(key)-10s = %(value).2f' % {'key': key, 'value': value}
```
