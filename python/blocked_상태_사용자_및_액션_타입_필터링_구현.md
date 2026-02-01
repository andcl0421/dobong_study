## 1️⃣ 내가 푼 문제

중첩된 사용자 액션 데이터에서  
- `status`가 `'blocked'`인 액션이 **하나라도 존재하는 사용자(user_id)** 를 찾고  
- 전체 데이터를 통틀어 **blocked 상태인 action의 type을 중복 없이 추출**하는 문제

### 📌 문제 예시 데이터

```python
users = [
    {
        "user_id": "U001",
        "actions": [
            {"type": "login", "status": "success"},
            {"type": "comment", "status": "success"}
        ]
    },
    {
        "user_id": "U002",
        "actions": [
            {"type": "login", "status": "blocked"},
            {"type": "purchase", "status": "success"}
        ]
    },
    {
        "user_id": "U003",
        "actions": [
            {"type": "comment", "status": "success"}
        ]
    },
    {
        "user_id": "U004",
        "actions": [
            {"type": "purchase", "status": "blocked"},
            {"type": "login", "status": "blocked"}
        ]
    },
    {
        "user_id": "U005",
        "actions": [
            {"type": "login", "status": "success"},
            {"type": "purchase", "status": "blocked"}
        ]
    }
]
```

---

## 2️⃣ 풀이 과정

1. 전체 `users` 리스트를 순회한다.
2. 각 사용자별로 `actions` 리스트를 다시 순회한다.
3. 각 action에서 `status` 값을 확인한다.
4. `status == 'blocked'`인 경우:
   - 해당 사용자의 `user_id`를 결과 리스트에 추가 (중복 방지)
   - 해당 action의 `type`을 type 리스트에 추가 (중복 방지)
5. 의미가 다른 결과(user_id / action type)는 **서로 다른 리스트**로 관리한다.

---

## 3️⃣ 내 코드

```python
blocked_user_ids = []
blocked_action_types = []

for user in users:
    user_id = user['user_id']
    actions = user['actions']

    for action in actions:
        action_type = action['type']
        status = action['status']

        if status == 'blocked':
            if user_id not in blocked_user_ids:
                blocked_user_ids.append(user_id)
            if action_type not in blocked_action_types:
                blocked_action_types.append(action_type)

print(blocked_user_ids, blocked_action_types)
```

---

## 4️⃣ 실무 관점에서의 주의사항

- **서로 다른 의미의 데이터는 반드시 분리된 자료구조로 관리**
  - user_id와 action type을 하나의 리스트에 섞으면 재사용성과 가독성이 크게 떨어짐
- 파이썬 **내장 함수 이름(`type`, `list`, `dict` 등)** 을 변수명으로 사용하지 말 것
- 중복 제거가 필요한 경우:
  - 데이터 규모가 크다면 `set` 사용 고려
- 입력 데이터(`users`)와 결과 데이터 변수명을 명확히 구분해야 실수 방지 가능

---

## 5️⃣ 배운 점

- 중첩 리스트 구조에서는 **반복 순서와 조건 위치가 핵심**
- 요구사항이 여러 개인 경우, 결과도 **여러 개의 리스트/구조로 나누는 것이 정답**
- 로직이 맞아도 **데이터 구조 설계가 틀리면 오답**
- 실무 데이터 처리에서는 “동작”보다 **의미와 구조**가 더 중요함
