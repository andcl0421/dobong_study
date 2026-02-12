#  TIL (Today I Learned): 2026-02-12

##  API 설계 및 데이터베이스 구조 최적화

### 1.  비즈니스 로직 및 설계 핵심 포인트
* **Request/Response 모델 분리**: 보안을 위해 입력용(`UserCreate`)과 출력용(`UserResponse`) 스키마를 분리하여 비밀번호 노출을 원천 차단했습니다.
* **DB 참조 무결성 (CASCADE)**: 부모(User, Product) 삭제 시 자식(Wishlist) 데이터가 자동으로 정리되도록 `ondelete="CASCADE"`를 설정했습니다.
* **성능 최적화 (Indexing)**: 검색 빈도가 높은 `nickname` 필드에 인덱스를 추가하여 조회 속도를 개선했습니다.
* **데이터 타입 표준화**: 시간 기록을 위해 `int`나 `str` 대신 계산과 정렬에 유리한 `DateTime` 타입을 채택했습니다.

---

###  트러블슈팅 (Troubleshooting)

#### 1) 405 Method Not Allowed
- **문제**: 서버는 `POST`를 기다리는데 포스트맨에서 `GET`으로 요청함.
- **해결**: HTTP 메서드를 `POST`로 일치시켜 해결.

#### 2) JSON Decode Error (Expecting ':' delimiter)
- **문제**: JSON 작성 시 쉼표(`,`)를 누락하거나 문법이 틀림.
- **해결**: JSON 문법(쌍따옴표, 콤마 위치)을 교정하여 데이터 파싱 성공.

#### 3) Not Authenticated (401 Unauthorized)
- **문제**: 인증이 필요한 API에 토큰 없이 접근함.
- **해결**: 포스트맨 Authorization 탭에 `Bearer Token`을 추가하여 인증 통과.

---

### 💻 실무형 예시 코드 (Final Implementation)

#### [Database Model: User & WishList]
```python
from sqlalchemy.orm import Mapped, mapped_column, relationship
from sqlalchemy import String, ForeignKey, func
from datetime import datetime
from database import Base

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    email: Mapped[str] = mapped_column(String(100), unique=True, nullable=False)
    password: Mapped[str] = mapped_column(String(255), nullable=False) # 해싱 대비 길이 확보
    nickname: Mapped[str] = mapped_column(String(50), index=True, nullable=False)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())

    # 자식 객체 관리 (delete-orphan: 부모와 관계 끊어지면 자동 삭제)
    wishlists: Mapped[list["WishList"]] = relationship(
        "WishList", back_populates="user", cascade="all, delete-orphan"
    )

class WishList(Base):
    __tablename__ = "wishlist"

    user_id: Mapped[int] = mapped_column(ForeignKey("users.id", ondelete="CASCADE"), primary_key=True)
    product_id: Mapped[int] = mapped_column(ForeignKey("products.id", ondelete="CASCADE"), primary_key=True)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())

    user: Mapped["User"] = relationship("User", back_populates="wishlists")
```

#### [API Schemas]
```python
from pydantic import BaseModel, EmailStr, ConfigDict
from datetime import datetime

class UserCreate(BaseModel):
    email: EmailStr
    password: str
    nickname: str

class UserResponse(BaseModel):
    id: int
    email: str
    created_at: datetime
    model_config = ConfigDict(from_attributes=True)
```

---

