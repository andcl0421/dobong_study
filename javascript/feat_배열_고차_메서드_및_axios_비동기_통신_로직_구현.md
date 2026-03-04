# 📚 [2026-03-04] 자바스크립트 심화: 고차 함수부터 비동기 통신까지

## 🎯 학습 목표
- 배열 고차 메서드(`filter`, `map`, `reduce`)를 활용한 데이터 가공 숙달
- 콜백 함수와 비동기 처리(`async/await`)의 이해
- `axios`를 이용한 외부 API 통신 실습 및 예외 처리

---

## 💡 주요 학습 내용

### 1. 배열 고차 메서드 (Array Higher-Order Methods)
- **`filter()`**: 조건에 맞는 요소만 골라내어 '새로운 배열'을 만듭니다. (원본 보존)
- **`map()`**: 각 요소를 원하는 형태로 변환하여 '새로운 배열'을 만듭니다.
- **`reduce(acc, cur)`**: 여러 개의 데이터를 하나의 값(합계 등)으로 응축합니다. `acc`는 누적 바구니, `cur`은 현재 아이템입니다.

### 2. 비동기 처리 & Axios (Asynchronous JS)
- **비동기(Asynchronous)**: 빨래를 돌려놓고 설거지를 하는 것처럼, 작업이 끝날 때까지 기다리지 않고 다음 일을 하는 방식입니다.
- **`async / await`**: 비동기 코드를 마치 순서대로 실행되는 코드처럼 읽기 쉽게 만들어주는 문법입니다.
- **`axios`**: 브라우저와 서버 간에 데이터를 주고받기 위해 사용하는 실무 표준 라이브러리입니다.

### 3. 예외 처리 (Error Handling)
- **`try...catch`**: 에러가 발생할 것 같은 코드를 `try`에 넣고, 에러 발생 시 `catch`에서 안전하게 처리하여 프로그램이 멈추지 않게 합니다.

---

## 🛠️ 비즈니스 로직 및 설계 핵심 포인트 (실무 예시)

````javascript
// 주문 목록에서 완료된 항목의 총 매출 계산 로직
const calculate_completed_revenue = async (api_url) => {
    try {
        // 1. axios를 이용한 데이터 수신
        const response = await axios.get(api_url);
        const orders = response.data;

        // 2. 비즈니스 로직: 완료(completed) 상태인 주문의 total 합산
        const total_revenue = orders
            .filter((order) => order.status === 'completed')
            .reduce((acc, cur) => acc + cur.total, 0);

        console.log(`오늘의 총 매출: ${total_revenue}원`);
    } catch (error) {
        console.error("데이터를 가져오는 중 오류가 발생했습니다:", error.message);
    }
};

코드 리뷰: 더 나은 변수명 제안 (Naming Convention)
total_order (X) → total_revenue 또는 completed_total_sum (O)

이유: 단순한 주문 목록이 아니라 '금액의 합계'임을 명확히 나타내야 합니다.

post_item (O) → 배열 posts를 순회할 때 단수형인 post 또는 명확한 명칭인 post_item을 권장합니다.

⚠️ Troubleshooting (오늘의 삽질)
이슈: reduce 연산 결과가 NaN으로 출력됨
현상: 장바구니 문제의 로직(cur.quantity * cur.price)을 주문 합계 문제에 그대로 적용했으나 결과값이 숫자가 아니라고 나옴.

원인: 데이터 구조(Schema)가 달랐음. orders 배열에는 quantity 속성이 없고 이미 계산된 total 속성만 존재했기 때문임.

해결: 현재 다루고 있는 객체의 속성명을 정확히 확인하여 acc + cur.total로 수정함.

교훈: 코드를 복사하기 전 반드시 console.log()를 통해 데이터의 구조를 먼저 파악하는 습관이 중요함을 깨달음.

📦 모듈화 (Module)
export와 import를 사용하여 기능을 별도 파일로 분리했습니다. 
이는 대규모 프로젝트에서 코드의 재사용성을 높이고 유지보수를 쉽게 만들어주는 실무 핵심 기술입니다.