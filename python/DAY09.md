### 알고리즘 문제 풀이

중첩 for 반복문

조건문 (if)

🔹 코드
popular_titles = []

status = api_response['status']
data = api_response['data']
users = data['users']

for user in users:
    posts = user['posts']
    for post in posts:
        title = post['title']
        likes = post['likes']
        if likes >= 50:
            popular_titles.append(title)

print(popular_titles)

2️⃣ 학생 성적 데이터 평균 계산
🔹 문제

각 학생의 grades 리스트에서

80점 이상 점수만 추출

해당 점수들의 평균이 90점 이상인 학생 이름만 리스트에 저장

🔹 사용 개념

리스트 순회

조건부 합계 및 개수 계산

평균 계산

중첩 반복문

🔹 코드
top_students = []

for student in students:
    total = 0
    count = 0
    name = student['name']
    grades = student['grades']

    for grade in grades:
        if grade >= 80:
            total += grade
            count += 1

    grand_total = total / count
    if grand_total >= 90:
        top_students.append(name)

print(top_students)

3️⃣ 장바구니 최종 결제 금액 계산 함수
🔹 문제

장바구니 상품 총액 계산

회원 등급별 할인 적용

VIP → 15%

GOLD → 10%

할인 후 금액이 50,000원 미만이면 배송비 3,000원 추가

🔹 사용 개념

함수 정의

반복문

조건 분기

실무형 계산 로직

🔹 코드
def calculate_checkout_price(cart_items, membership_grade):
    total = 0

    for cart in cart_items:
        price = cart['price']
        quantity = cart['quantity']
        total += price * quantity

    if membership_grade == 'VIP':
        total *= 0.85
    elif membership_grade == 'GOLD':
        total *= 0.9

    if total < 50000:
        total += 3000

    return int(total)


my_cart = [
    {"price": 25000, "quantity": 1},
    {"price": 15000, "quantity": 2}
]

print(calculate_checkout_price(my_cart, "GOLD"))  # 52500

✅ 오늘 학습 정리

중첩 데이터 구조(list + dict) 탐색 능력 향상

조건에 따른 데이터 필터링

평균 계산 로직 이해

실무에서 자주 쓰이는 결제 로직 구현

변수명, 들여쓰기, 조건 분기 연습
