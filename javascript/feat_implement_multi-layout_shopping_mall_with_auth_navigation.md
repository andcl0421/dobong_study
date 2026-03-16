# 🛒 React 쇼핑몰 레이아웃 및 인증 실습

### 📅 작성일: 2026-03-16

---

### 🚀 1. 실습 목표
- `react-router-dom`을 이용한 다중 레이아웃(`Layout`, `SimpleLayout`) 구성
- `/layout-prob/` 공통 경로를 기반으로 한 중첩 라우팅 구현
- `useNavigate`와 `useState`를 활용한 로그인/회원가입 로직 완성

---

### 💻 2. 핵심 구현 코드

#### 📂 Router 설정 (`src/router/index.jsx`)
- 홈/상품은 `Layout`(Header+Footer) 적용
- 로그인/회원가입은 `SimpleLayout`(Header만) 적용

````javascript
const router = createBrowserRouter([
  {
    path: "/layout-prob",
    element: <Layout />,
    children: [
      { path: "", element: <Home /> },
      { path: "products", element: <Products /> },
      { path: "cart", element: <Cart /> },
    ],
  },
  {
    path: "/layout-prob",
    element: <SimpleLayout />,
    children: [
      { path: "login", element: <Login /> },
      { path: "register", element: <Register /> },
    ],
  },
]);

📂 로그인 로직 (src/pages/Login.jsx)
onSubmit 핸들러와 e.preventDefault()를 사용한 폼 전송 제어

성공 시 replace: true 옵션으로 뒤로가기 방지

JavaScript
const handle_login = (e) => {
  e.preventDefault();
  if (user_id === "admin" && user_pw === "1234") {
    alert("로그인 성공!");
    navigate("/layout-prob/", { replace: true });
  } else {
    set_error_msg("아이디 또는 비밀번호가 틀렸습니다.");
  }
};

3. 실무 밀착형 설계 포인트
중첩 라우팅과 레이아웃 분리

페이지 성격에 따라 다른 레이아웃을 적용하여 코드의 재사용성을 높였습니다.

제어 컴포넌트 (Controlled Component)

input의 value를 리액트 state와 동기화하여 실시간 데이터 검증이 가능하게 구현했습니다.

사용자 경험 (UX) 개선

form 태그를 활용해 엔터 키 로그인을 지원하고, 가입 완료 후 자동으로 로그인 페이지로 이동시켜 흐름을 끊지 않았습니다.

⚠️ 4. Troubleshooting (트러블슈팅)
문제: ReferenceError: navigate is not defined

원인: useNavigate 훅을 선언(const navigate = useNavigate();)하지 않고 사용함.

해결: 컴포넌트 상단에 훅 선언문을 추가하여 해결.

문제: 로그인 버튼 클릭 시 페이지가 새로고침되며 데이터가 유실됨.

원인: form 태그의 기본 제출 동작(GET/POST)이 발생함.

해결: e.preventDefault()를 호출하여 브라우저의 기본 동작을 차단함.