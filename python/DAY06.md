### Python 기초 학습 정리
  
## 리스트 + 조건문 + 함수의 retur 개념
문자열 템플릿 (Template String)

템플릿 문자열은 미리 만들어진 문자열 형식(틀)에  
값만 채워 넣어서 사용하는 방식
format() 사용 예제
```python
template = "안녕하세요, {name}님!"
result = template.format(name="철수")
print(result)
```
- 출력결과 안녕하세요, 철수님!

## 딕셔너리의 items()
-items()는 딕셔너리에서 (key, value) 쌍을 꺼내는 메서드

## 함수와 return 
함수는 특정 로직을 묶어서 재사용하기 위해 사용한다.
return은 함수의 결과값을 호출한 곳으로 돌려주는 역할
``` python
def add(a, b):
    return a + b

result = add(3, 5)
print(result)
```
출력결과 '8'

## 오늘 배운 핵심 요약

- 문자열 템플릿: format(), f-string
- 딕셔너리 반복: items()
- 리스트 + 조건문으로 데이터 필터링
- 리스트(list), 딕셔너리(dict)가 중첩된 구조에서 데이터 꺼내는 방법
- for문을 몇 번 써야 하는지 판단하는 기준
- 이중 for문이 필요한 경우와 필요 없는 경우 구분
- 조건문과 반복문을 활용한 실전 문제 풀이
- 함수(function)의 기본 개념과 사용하는 이유
- 함수의 return은 결과를 돌려준다
- 실무에서는 코드 길이보다 가독성과 구조 이해가 더 중요함