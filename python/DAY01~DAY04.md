# dobong_study
도봉캠퍼스 교육내용 정리 
## Pre-study Summary (Day01~Day04)
### 1. 변수
값을 저장하는 이름표
나중에 다시 사용하기 위해 데이터를 담아둠
```python
age = 18
name = "jun" 
```


할당은
변수에 값을 넣는 것
= 기호 사용
person : 딕셔너리 객체를 담고 있는 변수
= : 할당 연산자
person = {'name': 'jun', 'age': 18}

### 2.자료형 (Data Type)

딕셔너리 (Dictionary)
-이름표(key) + 값(value) 형태로 데이터 저장
-중괄호 {} 사용
-의미가 중요한 데이터에 적합 / key (앞) value (뒤)

리스트 (List)
-순서가 있는 값들의 모음이며 값에 이름이 없다
-인덱스(위치)로 접근 대괄호 [] 사용

### 3. 조건문 (if / else)

- 조건문이란, 조건이 True / False 인지에 따라 실행 흐름 결정
- 조건은 boolean (True . False)

```python
if money >= 1000:
    print("과자 사먹을거야")
else:
    print("못 사먹어")
```
-들여쓰기는 필수

### 4. 반복문 (while)
while문이란?

-조건이 True인 동안 계속 반복
-조건이 False가 되면 종료
while 조건:
    실행할 코드

``` python 

count = 0

while count < 5:
    print(count)
    count += 1
```

- 무한 루프 주의
```python
 while True:
    print("계속 실행됨")
```

 입력과 함께 사용하는 while문

``` python
password = ""

while password != "1234":
    password = input("비밀번호 입력: ")

print("로그인 성공")
```
- 조건문 + while문 함께 사용
``` python 
money = 5000

while money > 0:
    print("과자를 샀다")
    money -= 1000

print("돈이 없다")
```

-break (반복문 중단)
``` python 
while True:
    answer = input("종료하려면 q 입력: ")

    if answer == "q":
        break
```
- continue (다음 반복으로 이동)
``` python 
count = 0

while count < 5:
    count += 1
    if count == 3:
        continue
    print(count)
```
### 전체 정리

-변수 = 값 저장
-할당 = 값 넣기
-딕셔너리 = key:value
-리스트 = 순서 있는 값
-print() = 출력
-input() = 입력 (문자열)
-조건문 = if / elif / else
-반복문 = while
-들여쓰기 = 문법 그 자체 

# Python for문 정리

---

##  for문이란?

- 정해진 횟수만큼 반복할 때 사용하는 반복문
- 리스트, 문자열, range()와 함께 사용
- 하나씩 꺼내서 처리하는 구조

---

## 기본 구조

for 변수 in 반복가능한_데이터:
    실행할 코드

-리스트와 for문
``` python
fruits = ['사과', '바나나', '딸기']

for fruit in fruits:
    print(fruit)
```






