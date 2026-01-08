모듈
# 모듈(Module)

## 1. 모듈(Module)

### 1-1. 모듈이란?
모듈(Module)이란, 특정 기능을 하는 코드를 파이썬 파일(.py) 단위로 작성한 것을 말한다.  
모듈을 이용하면 다른 파이썬 파일에 작성된 기능을 가져다 사용할 수 있다.

**모듈의 장점**
- 코드를 새로 작성하지 않고, 다른 모듈에 있는 코드를 불러와서 재사용할 수 있다.
- 기능 단위로 코드가 분리되어 유지보수가 쉬워진다.

### 1-2. import
모듈을 불러올 때는 `import 모듈이름`(.py 파일의 이름)와 같은 형식으로 작성한다.  
모듈 내의 기능을 사용할 때는 `모듈이름.함수명`과 같은 형식으로 작성한다.

from random import * 는 파이썬에서 random 모듈 안에 있는 모든 함수와 변수들을 한꺼번에 가져온다는 뜻입니다.
shuffle -> 섞는다라는 의미 
import random -> 랜덤 관련 기능들이 들어 있는 도구 상자를 가져오는 것
random 모듈 안에 명단을 섞는 함수 (그래서 명단랜덤섞기라고 생각할수도 있음)
def는 “함수를 만든다”는 뜻

## 2. 자주 사용하는 모듈
파이썬은 다양한 기능이 있는 표준 모듈을 제공한다.  
따라서, 기능을 직접 만들 필요 없이 이미 만들어진 모듈을 불러오는 것만으로 편리하게 사용 가능하다.

### 2-1. math 모듈
math 모듈은 수학 연산을 다루는 기능을 제공한다.
(import math)

# math.sqrt(숫자): 양수 제곱근 반환
print(f"sqrt(2): {math.sqrt(2)}")

# math.pow(숫자, n): 숫자의 n제곱 반환
print(f"2의 3제곱: {math.pow(2, 3)}")

# math.ceil(숫자): 올림값 반환
print(f"3.2의 올림: {math.ceil(3.2)}")
print(f"3.8의 올림: {math.ceil(3.8)}")

# math.floor(숫자): 내림값 반환
print(f"3.2의 내림: {math.floor(3.2)}")
print(f"3.8의 내림: {math.floor(3.8)}")

# math.pi: 원주율
print(f"원주율(pi): {math.pi}")

### 2-2. random 모듈
random 모듈은 난수(무작위 수)를 다루는 기능을 제공한다.

### 2-3. collections 모듈
collections 모듈은 특수한 용도의 컨테이너 자료형을 제공한다.

### 2-4. itertools 모듈
itertools 모듈은 효율적인 반복 작업을 위한 도구(순열, 조합 등)를 제공한다.


## 오늘의 실습
- 랜덤으로 조편성하기 

``` python
from random import shuffle # shuffle 함수를 불러옴


def make_groups(names) : # def 함수이름(재료):
    shuffle(names) #리스트를 무작위로 섞음 
    name_list = {} # 조번호 와 조원 리스트를 저장할 곳 
    group_number = 1 # 조번호를 카운트하는 변수
   
    for i in range(0,len(names),4):
        member = names[i:i+4] # 리스트 슬라이싱으로 그룹 만듬 i번째부터 i+3번째까지 =4명
        name_list[group_number] = member # 딕셔너리에 조 번호 (키),조원 리스트 (값) 저장
    
        group_number +=1
    
    for num,member in name_list.items() :
        print(f'{num}조 : {','.join(member)}')

    return name_list

make_groups(names_list) 
```
s

