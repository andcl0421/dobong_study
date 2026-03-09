# 📅 2026-03-09 실습: React Styling & Advanced Rendering

### 📌 비즈니스 로직 및 설계 핵심 포인트
1. **스타일 격리(Isolation)**: `Module CSS`를 통해 컴포넌트 간 클래스명 충돌을 방지하고 독립적인 스타일 스코프 유지.
2. **유틸리티 퍼스트(Utility-First)**: `Tailwind CSS`를 도입하여 별도의 CSS 파일 관리 없이 HTML 수준에서 신속한 UI 레이아웃 구축.
3. **선언적 렌더링**: `map()` 메서드와 조건부 연산자(`&&`, `? :`)를 결합하여 데이터 상태에 따라 UI를 동적으로 제어.
4. **리스트 식별성(Identification)**: 고유 `key` 값을 부여하여 React Virtual DOM의 효율적인 업데이트 최적화.

---

### 🎨 1. Styling: Module CSS & Tailwind
#### [Module CSS 적용]
* 파일명 규칙: `컴포넌트명.module.css`
* 접근 방식: 객체 표기법(`styles.className`) 및 케밥 케이스 대응(`styles["class-name"]`).

#### [Tailwind CSS v4.0]
* **설치**: `npm install tailwindcss @tailwindcss/vite`
* **설정**: `vite.config.js`에 플러그인 추가 및 `index.css`에 `@import "tailwindcss";` 선언.

---

### 🔄 2. 조건부 렌더링 (Conditional Rendering)
| 방식 | 사용 사례 | 코드 예시 |
| :--- | :--- | :--- |
| **삼항 연산자** | A 또는 B를 반드시 보여줘야 할 때 | `{isLogin ? <LogoutBtn /> : <LoginBtn />}` |
| **논리 연산자(&&)** | 조건이 참일 때만 특정 UI를 노출할 때 | `{isAdmin && <AdminPanel />}` |
| **Class 조합** | 상태에 따라 스타일을 변경할 때 | `className={isDone ? styles.done : ""}` |

---

### 📋 3. 리스트 렌더링 (List Rendering)
* **메커니즘**: `array.map()`을 사용하여 배열 데이터를 JSX 엘리먼트 배열로 변환.
* **Key Prop**: 리스트 항목에는 반드시 고유 식별자(ID)를 `key`로 전달해야 함 (Index 사용 지양).

````javascript
{members.map((member) => (
  <div key={member.id}>
    <span>{member.name}</span>
    {member.role === "admin" ? <AdminBadge /> : <UserBadge />}
  </div>
))}

🚀 Troubleshooting
1. Uncaught ReferenceError: el is not defined
현상: 부모 컴포넌트에서 자식 컴포넌트 호출 시 정의되지 않은 el 변수 참조 시도.

원인: 자식 컴포넌트 내부 map 함수에서 사용하는 지역 변수를 부모의 Scope에서 사용함.

해결: 자식 컴포넌트가 스스로 데이터를 처리하도록 설계하거나, 부모에서 명확한 데이터를 Props로 전달하여 해결.

2. JSX Expression Parsing Error
현상: 삼항 연산자 로직이 화면에 텍스트로 그대로 출력됨.

원인: JSX 내부에 자바스크립트 로직을 작성하면서 중괄호({ })를 누락함.

해결: 로직 전체를 { }로 감싸 자바스크립트 엔진이 해석하도록 수정.

3. Multiple Root Elements Error
현상: return 문 내부에 여러 태그가 나열되어 빨간 줄 발생.

원인: JSX는 반드시 단일 Root Element를 반환해야 함.

해결: Fragment(<> ... </>) 또는 <div>로 전체 요소를 래핑하여 해결.