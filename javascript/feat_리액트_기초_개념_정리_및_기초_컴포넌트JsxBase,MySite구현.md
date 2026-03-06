# 📚 [TIL] React 기초: SPA, Virtual DOM, JSX 및 Props 핵심 정리

**작성일:** 2026-03-06  

---

## 1. 왜 리액트(React)인가?
리액트는 웹페이지를 **'레고 블록(Component)'** 조립하듯 만들게 해주는 도구입니다.

* **컴포넌트 기반:** 화면을 독립된 조각으로 나눠 재사용 가능
* **거대한 생태계:** 수많은 라이브러리와 커뮤니티로 문제 해결이 쉬움
* **높은 수요:** 취업 시장에서 매우 유리한 기술 스택

---

## 2. SPA (Single Page Application)
하나의 HTML 파일(`index.html`)만 사용하여, 페이지 새로고침 없이 부드럽게 화면을 전환하는 방식입니다.

| 구분 | SPA (React, Gmail) | MPA (Naver News, 위키백과) |
| :--- | :--- | :--- |
| **화면 전환** | 새로고침 없이 부분 업데이트 | 전체를 새로 불러옴 (깜빡임 발생) |
| **속도** | 초기 로딩은 느리나 이후 매우 빠름 | 매 페이지마다 로딩 시간 소요 |
| **장점** | 모바일 앱 같은 자연스러운 사용감 | 검색 엔진 최적화(SEO)에 유리함 |

---

## 3. Virtual DOM (가상 DOM)
성능 최적화의 핵심입니다. 실제 화면을 고치기 전에 메모리에서 미리 연습해보는 **'중간 계층'**입니다.

* **원리:** 데이터 변경 → 가상 DOM 생성 → 이전 가상 DOM과 비교(Diffing) → 바뀐 부분만 실제 DOM에 **한 번에 반영(Batch Update)**
* **장점:** 화면 전체를 다시 그리는 비효율을 줄여 성능 향상

---

## 4. JSX (JavaScript XML)
자바스크립트 안에서 HTML처럼 코드를 짤 수 있는 문법입니다.

### ⚠️ JSX 필수 규칙 (실무 주의사항)
1.  **최상위 태그는 반드시 1개:** 여러 태그를 반환할 땐 빈 태그 `<> ... </>` (Fragment)로 감싸야 함
2.  **모든 태그 닫기:** `<input />`, `<br />` 처럼 Self-closing 필수
3.  **CamelCase 속성:** `class` → `className`, `onclick` → `onClick`
4.  **Style은 객체로:** `style={{ backgroundColor: 'red' }}` 형식 (중괄호 2개 필수!)

---

## 5. 컴포넌트 (Component)
재사용 가능한 UI 블록입니다.

* **작성 규칙:** 파일명과 함수명은 반드시 **PascalCase**(첫 글자 대문자)로 작성
* **실무 팁:** `rafce` 스니펫을 사용하여 빠르게 화살표 함수 구조를 생성함

````javascript
// src/components/WelcomeBox.jsx
const WelcomeBox = () => {
  const user_status = "온라인"; // Snake Case 변수명 권장
  return <div>환영합니다! 현재 {user_status} 상태입니다.</div>;
};
export default WelcomeBox;

6. Props (Properties)
부모 컴포넌트가 자식에게 전달하는 **'읽기 전용 데이터'**입니다.

전달: <Child name="철수" age={25} />

수령: 구조 분해 할당을 사용하는 것이 실무 표준

주의: 자식은 받은 Props를 절대 직접 수정할 수 없음 (Immutable)

// 구조 분해 할당 + 기본값 설정 예시
const UserProfile = ({ user_name = "익명", user_age = 0 }) => {
  return (
    <div>
      <h2>이름: {user_name}</h2>
      <p>나이: {user_age}</p>
    </div>
  );
};

🚩 오늘의 트러블슈팅 (Troubleshooting)
1) [vite:import-analysis] 에러
문제: 경로 시작에 .을 빼먹어 파일을 못 찾음.

해결: import Article from "./Article" 처럼 상대 경로(./)를 명확히 표시함.

2) export 'default' 누락 에러
문제: 파일을 불러왔으나 정작 해당 파일에서 export default로 내보내지 않음.

해결: 파일 하단에 export default 컴포넌트명;을 추가하여 짝을 맞춤.