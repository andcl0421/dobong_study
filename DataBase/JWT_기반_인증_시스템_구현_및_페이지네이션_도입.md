#  [2026-02-11] 학습 기록: JWT 인증 및 페이지네이션 최적화

##  1. 비즈니스 로직 및 설계 핵심 포인트

오늘 구현한 아키텍처는 **보안**과 **성능**을 동시에 고려한 실무형 설계입니다.

* **비밀번호 보안 (Password Hashing):** 사용자 비밀번호는 `bcrypt`를 이용해 단방향 해싱하여 저장합니다. 이는 데이터베이스가 유출되더라도 원본 비밀번호를 복구할 수 없게 만드는 보안의 기본입니다.
* **토큰 기반 인증 (JWT):** 서버에 세션을 저장하지 않는 **무상태(Stateless)** 방식을 채택하여 서버 확장성을 높였습니다. 
* **데이터 최적화 (Pagination):** 대량의 데이터를 조회할 때 서버와 클라이언트의 부하를 줄이기 위해 `fastapi-pagination`을 적용, 데이터를 일정 단위로 나누어 제공합니다.
* **레이어드 아키텍처 (Layered Architecture):** `Router`(입구), `Service`(비즈니스 로직), `Repository`(DB 접근)로 역할을 분리하여 유지보수성을 극대화했습니다.

---

## 🛠️ 기술적 구현 상세

### 📦 Pagination (데이터 효율적 조회)
기존의 리스트 반환 방식에서 `Page` 객체 반환 방식으로 변경하여 메타데이터(현재 페이지, 전체 개수 등)를 포함한 응답을 제공합니다.



```python
# [변경 후] 리포지토리 로직 (user_repository.py)
from fastapi_pagination.ext.sqlalchemy import paginate

def get_users(db: Session):
    stmt = select(User).options(selectinload(User.profile))
    return paginate(db, stmt)  # Page[UserResponse] 형태로 자동 변환 및 반환
```

### 🔐 인증 및 인가 (Authentication & Authorization)
* **인증(AuthN):** 사용자가 누구인지 확인 (로그인)
* **인가(AuthZ):** 인증된 사용자가 특정 권한을 가졌는지 확인 (관리자 여부 등)

**JWT (JSON Web Token) 구조:**
1.  **Header:** 토큰 타입 및 암호화 알고리즘
2.  **Payload:** 사용자 식별자(`sub`), 만료 시간(`exp`) 등 (Base64 인코딩)
3.  **Signature:** 서버의 `SECRET_KEY`를 이용한 위변조 방지 검증용



---

## 🌐 실시간 통신 방식 비교 (Real-time Communication)

| 방식 | 특징 | 사용 예시 |
| :--- | :--- | :--- |
| **Polling** | 주기적으로 서버에 요청 | 실시간성이 낮아도 되는 정적 데이터 |
| **Long Polling** | 이벤트 발생 전까지 서버 응답 보류 | 실시간 알림, 메시지 도착 확인 |
| **SSE** | 서버에서 클라이언트로 단방향 스트리밍 | 뉴스 피드, 주가 알림 |
| **WebSocket** | 양방향 지속 연결 (전이중 통신) | 실시간 채팅, 온라인 게임 |

---

## ⚠️ Troubleshooting (트러블슈팅)

* **문제:** JWT의 `Payload`가 암호화되는 줄 알고 민감한 개인정보를 넣으려 함.
* **해결:** `Payload`는 누구나 디코딩할 수 있는 Base64 인코딩 형태이므로, 보안을 위해 사용자 ID(`sub`)와 만료 시간(`exp`) 등 최소한의 식별 정보만 담도록 수정함.
* **문제:** `response_model` 설정 시 SQLAlchemy 객체가 Pydantic 모델로 변환되지 않아 에러 발생.
* **해결:** Pydantic 모델의 `model_config`에 `from_attributes=True` 설정을 추가하여 ORM 객체와의 호환성을 확보함.

---

## 📤 깃허브 커밋 가이드

### 1. 커밋 메시지 (Commit Message)

**제목:** `feat: JWT 기반 인증 시스템 구현 및 페이지네이션 도입`

**상세 내용:**
- bcrypt 라이브러리를 통한 비밀번호 단방향 해싱 로직 구현
- PyJWT를 활용한 액세스 토큰 생성 및 검증(AuthService) 추가
- fastapi-pagination 적용을 통한 사용자 목록 조회 API 성능 최적화
- OAuth2PasswordBearer를 이용한 인증 의존성(get_current_user) 수립
- Router-Service-Repository 레이어 분리를 통한 아키텍처 정립