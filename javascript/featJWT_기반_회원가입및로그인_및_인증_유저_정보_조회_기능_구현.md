

## 📅 작성일: 2026-03-18

## 🚀 프로젝트명: React + FastAPI를 활용한 Full-Stack JWT 인증 시스템
기존의 Todo 서비스에 사용자 보안을 위한 **JWT(JSON Web Token)** 인증 체계를 도입했습니다. 회원가입부터 로그인 상태 유지, 그리고 권한에 따른 페이지 접근 제어(Protected Route)까지 실무 표준 방식으로 구현했습니다.

---

## 1. 📂 저장소 판별 및 키워드 (Keywords)
* **Backend**: FastAPI, PyJWT, bcrypt, SQLAlchemy, PostgreSQL
* **Frontend**: React, Zustand, Axios Interceptor, React Router Dom, Tailwind CSS
* **Auth**: JWT, Hashing, LocalStorage, Protected Route

---

## 2. 💡 실무형 설계 핵심 포인트 (Business Logic)

### [Backend] 철저한 보안 통제
* **비밀번호 보호**: `bcrypt`를 사용하여 사용자 비밀번호를 암호화된 '해시' 값으로 저장합니다. (DB가 해킹당해도 비밀번호는 안전합니다!)
* **토큰 발행**: 로그인 성공 시 서버는 `SECRET_KEY`로 서명된 유일한 **JWT**를 발행하여 클라이언트에 전달합니다.

### [Frontend] 효율적인 상태 관리와 자동화
* **Zustand Store**: 로그인 정보를 전역에서 관리하여 어떤 페이지에서도 유저 이름을 띄울 수 있게 설계했습니다.
* **Axios Interceptor**: 모든 API 요청마다 토큰을 일일이 넣는 노가다를 방지하기 위해, 요청 직전에 토큰을 자동으로 끼워 넣는 '자동 수문장' 로직을 적용했습니다.

---

## 3. 💻 실무 표준 예시 코드 (Direct Implementation)

### ① Axios 공통 인스턴스 (`src/api.js`)
변수명은 실무 표준인 `Snake Case`를 적용하여 가독성을 높였습니다.

````javascript
import axios from "axios";

// 1. 기본 주소가 설정된 통신 도구(인스턴스) 만들기
const api_client = axios.create({
  baseURL: "http://localhost:8000",
});

// 2. 요청 인터셉터: 편지(요청)를 보내기 전, 봉투에 '토큰' 스탬프를 자동으로 찍어줌
api_client.interceptors.request.use((config) => {
  const access_token = localStorage.getItem("token");
  
  if (access_token) {
    // 실무 약속: Bearer 뒤에 한 칸 띄우고 토큰을 넣습니다.
    config.headers.Authorization = `Bearer ${access_token}`;
  }
  return config;
});

export default api_client;

② 보호된 라우트 (src/components/auth_prac/ProtectedRoute.jsx)
로그인하지 않은 사용자가 마이페이지에 들어오려고 할 때 로그인 페이지로 튕겨내는 '경비원' 컴포넌트입니다.

import { Navigate, Outlet } from "react-router-dom";
import use_auth_store from "../../store/use_auth_store";

const ProtectedRoute = () => {
  const is_authenticated = use_auth_store((state) => state.access_token);

  // 토큰이 없으면 로그인 페이지로 강제 이동 (replace로 뒤로가기 방지)
  if (!is_authenticated) {
    return <Navigate to="/auth/login" replace />;
  }

  // 토큰이 있으면 가려져 있던 자식 페이지(Outlet)를 보여줌
  return <Outlet />;
};

export default ProtectedRoute;

🛠️ 트러블슈팅 (Troubleshooting)
1. 새로고침 시 유저 정보 실종 사건
현상: 로그인을 잘 했는데, 새로고침만 하면 "누구세요?"라며 로그아웃이 됨.

원인: 리액트의 변수(Zustand)는 새로고침하면 초기화되기 때문입니다.

해결: App.jsx가 켜질 때 useEffect를 사용하여 브라우저 금고(localStorage)에 있는 토큰을 꺼내 서버에 "이 토큰 누구 거야?"라고 물어보고 정보를 복구하는 check_auth 기능을 구현했습니다.

2. 회원가입 시 중복 이메일 처리
현상: 이미 가입된 이메일로 또 가입하면 서버가 죽음.

원인: DB 모델 설정에서 unique=True로 설정된 이메일에 중복 값이 들어오려 함.

해결: FastAPI에서 existing_user를 먼저 조회하고, 값이 있다면 HTTPException(409) 에러를 명확히 던져서 프런트엔드에서 "이미 가입된 이메일입니다"라고 안내하도록 수정했습니다.