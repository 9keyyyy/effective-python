## PEP8이란?
Python Enhancement Proposal (PEP) #8
- 파이썬 코드를 어떤 형식으로 작성할지 알려주는 스타일 가이드([영문 버전](https://peps.python.org/pep-0008/), [한글 버전](https://wikidocs.net/7896))
- 일관된 스타일로 코드를 작성하면, 1) 코드를 더 쉽게 읽을 수 있고 2) 쉽게 협력할 수 있고 3) 나중에 코드를 수정하기 쉽고 4) 흔히 저지르기 쉬운 실수를 피할 수 있음.
- 파이썬 언어가 개선됨에 따라 가이드도 변경됨.

## 꼭 따라야 하는 규칙
### 공백
- 탭 대신 스페이스 사용
- 중요한 들여쓰기는 4칸 스페이스[^1]
- 라인의 길이는 79개 문자 이하로[^2]
- 긴 식을 여러 줄에 나눠서 쓸 때는 일반적인 들여쓰기보다 4칸 스페이스 더 들여쓰기
- 파일 안에서 각 함수와 클래스 사이에는 빈 줄을 두 줄 넣기
- 클래스 안에서 메서드와 메서드 사이에는 빈 줄을 한 줄 넣기
- 딕셔너리는 `key: value`와 같은 식으로 스페이스 넣기
- 변수를 대입할 때 `=` 전후에 스페이스 하나씩 넣기
- 타입 표기를 하는 경우 `변수이름: 타입`과 같은 식으로 스페이스 넣기


### 명명 규약
- 함수, 변수, attribute는 `lowercase_underscore`
- 클래스는 `CapitalizeedWord`
- 보호되어야 하는 인스턴스 attribute는 `_lowercase_underscore`처럼 밑줄로 시작
- 비공개(클래스 내부에서만 쓰는) 인스턴스 attribute는 `__lowercase_underscore`처럼 밑줄 두 개로 시작
- 모듈 수준의 상수는 `ALL_CAPS`
- 클래스 내부의 인스턴스 메서드[^3]는 `self`가 첫 번째 호출 객체 인자로 들어가야 함
- 클래스 메서드[^3]는 클래스를 가리키는 첫 번째 인자로 `cls` 사용


### 식과 문
- 긍정적인 식을 부정하지(`if not a is b`) 말고 부정을 내부에 넣기(`if a is not b`)
- 빈 컨테이너나 시퀀스를 검사할 때는 `if len(something) == 0` 대신 `if not something`
- 비어있지 않은지 검사할 때는 `if len(something) > 0` 대신 `if something`
- 한 줄짜리 `if`, `for`, `while`, `except` 복합문을 사용하지 말고 여러 줄에 나눠 배치
- 식을 한 줄 안에 다 쓸수 없는 경우에는 괄호와 줄바꿈, 들여쓰기를 추가해서 읽기 쉽게
- 여러 줄에 걸쳐 식을 쓸 때는 `\`보다는 괄호 활용


### 임포트
- import 문은 항상 파일 맨 앞에
- 모듈을 임포트할 때는 절대 경로 사용 `import foo` -> `from bar import foo`
- 상대 경로를 사용해야 하는 경우에는 상대 경로임을 명시 `from . import foo`
- 임포트는 1) 표준 라이브러리, 2) 서드파티, 3) 만든 모듈 순으로 섹션을 나누고, 각 섹션 내에서는 알파벳 순서로 임포트

[^1]: VS Code는 기본적으로 탭을 사용하면 4칸 스페이스로 자동으로 변환
[^2]: 한글 한 글자는 문자 두 개로 계산
[^3]: 파이썬에는 static, instance, class 세 가지 메서드가 존재. static (@staticmethod)은 인스턴스 정의 없이 사용 가능하며, 독립적으로 동작하는 메서드. instance는 클래스를 인스턴스화 했을 때만 사용 가능. class (@classmethod)는 static과 달리 클래스 속성이나 다른 메서드에 접근 가능하지만 여전히 인스턴스 정의 없이 사용 가능.