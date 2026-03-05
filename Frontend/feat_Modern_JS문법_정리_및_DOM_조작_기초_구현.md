# 📅 작성일: 2026-03-05

## 🚀 실무 밀착형 모던 자바스크립트 & DOM 핵심 정리

오늘 수업에서는 자바스크립트의 현대적 문법(ES6+)과 브라우저를 내 마음대로 조종하는 DOM(Document Object Model)에 대해 학습했습니다. 단순히 화면을 만드는 것을 넘어, 데이터가 어떻게 흐르고 사용자의 반응에 어떻게 대응하는지 그 '설계 원리'를 파악하는 것이 핵심입니다.

---

## 💡 비즈니스 로직 및 설계 핵심 포인트

1.  **가독성과 생산성 (ES6+):** - `단축 프로퍼티`나 `구조 분해 할당`은 단순히 타이핑을 줄이는 것이 아니라, 객체의 구조를 한눈에 파악하게 하여 협업 시 코드 리뷰 효율을 높입니다.
2.  **데이터 안정성 (Optional Chaining & Nullish):** - 서버에서 데이터를 받아올 때(API 통신), 예상치 못한 `null`이나 `undefined`로 인해 서비스가 멈추는(런타임 에러) 현상을 방지하는 '방어적 프로그래밍'의 필수 도구입니다.
3.  **효율적인 UI 업데이트 (Event Delegation):** - 수백 개의 리스트 아이템에 일일이 이벤트를 거는 대신, 부모 요소 하나에 위임함으로써 메모리 사용량을 줄이고 동적으로 추가되는 요소에도 대응할 수 있는 설계를 구축합니다.

---

## 🛠 주요 학습 내용 & 예시 코드

### 1. 모던 자바스크립트 (ES6+) 필수 문법
비전공자 눈높이 설명: "복잡한 주소지 대신 별명을 쓰고, 짐을 일일이 풀지 않고 한 번에 던져주는 스마트한 방법들입니다."

```javascript
// [1] 단축 프로퍼티 & 계산된 속성명
const user_role = "admin";
const user_name = "Jun";

const user_profile = {
  user_name, // 변수명과 key가 같으면 생략!
  [`${user_role}_id`]: 1004, // 변수값으로 key 이름을 동적으로 생성
};

// [2] 펼침 연산자 (Spread) - "기존 데이터를 보존하며 복사하기"
const original_list = [1, 2, 3];
const updated_list = [...original_list, 4]; // [1, 2, 3, 4]

// [3] 구조 분해 할당 - "필요한 것만 쏙쏙 뽑아 쓰기"
const employee = { name: "철수", age: 30, job: "Dev" };
const { name: emp_name, job: emp_job } = employee; 

// [4] 옵셔널 체이닝 & Nullish 병합 - "에러 방지 콤보"
const api_response = { data: { user: null } };
const city = api_response?.data?.user?.address ?? "Unknown City";
```

### 2. DOM (Document Object Model) 조작
비전공자 눈높이 설명: "HTML이라는 설계도를 자바스크립트가 읽을 수 있는 '나무 모양 객체(Tree)'로 바꾼 것입니다. 우리는 이 나무의 가지를 치거나 새 잎을 달 수 있습니다."



```javascript
// 요소 선택 및 스타일 제어
const main_title = document.querySelector("#title");
main_title.textContent = "반갑습니다, 개발자님!";
main_title.style.color = "royalblue";

// 클래스 조작 (실무 권장 방식)
const login_modal = document.querySelector(".modal");
login_modal.classList.add("is-active"); // CSS와 역할을 분리하세요!
```

---

## ⚠️ 트러블슈팅 (Troubleshooting)

### 이슈: `addEventListener` 내부에서 `event.target`과 `event.currentTarget`의 혼동
- **현상**: 부모 요소에 이벤트를 걸었는데, 자식 요소를 클릭하니 예상치 못한 요소가 잡힘.
- **원인**: **이벤트 버블링(Bubbling)** 때문. 클릭한 실제 지점(`target`)과 이벤트 리스너가 달린 지점(`currentTarget`)이 다르기 때문임.
- **해결**: 
  - 실제 클릭된 가장 안쪽 요소를 찾을 때는 `event.target` 사용.
  - 내가 이벤트를 직접 걸어준(부모) 요소를 고정해서 쓰고 싶을 때는 `event.currentTarget` 사용.
- **교훈**: 이벤트 위임 구현 시 반드시 두 속성의 차이를 구분하여 조건문(`if (event.target.tagName === 'LI')`)을 작성해야 함.

---

## 🔍 코드 리뷰: 더 나은 변수명 제안 (Naming Convention)

| 기존 변수명 | 제안하는 변수명 (Snake Case) | 이유 |
| :--- | :--- | :--- |
| `obj` | `user_info_map` | 단순히 객체임을 명시하기보다 담긴 데이터의 성격을 표현합니다. |
| `arr1`, `arr2` | `original_items`, `cloned_items` | 배열의 선언 순서가 아닌 역할 차이를 명시합니다. |
| `btn` | `submit_order_button` | 어떤 기능을 수행하는 버튼인지 명확히 합니다. |
| `data` | `raw_api_response` | 가공 전 데이터임을 명시하여 혼동을 방지합니다. |

---

