# 🛒 상품 관리 및 위시리스트 시스템 API 설계

**작성일:** 2026-02-10

---

## 1. 프로젝트 개요
비전공자 신입 개발자로서 데이터베이스의 **N:M(다대다) 관계**를 이해하고, SQLAlchemy의 **Association Object**와 **Association Proxy**를 활용하여 실무형 위시리스트 기능을 설계하고 구현했습니다.

---

## 2. 데이터 모델 정의 (Entities)

### 👤 User (사용자)
- **역할:** 시스템의 일반 엔티티로, 상품을 찜할 수 있는 주체입니다.
- **주요 필드:** `user_id(PK)`, `nickname`

### 📦 Product (상품)
- **역할:** 시스템에서 관리되는 상품 데이터입니다.
- **주요 필드:** `product_id(PK)`, `product_name`

### ❤️ Wishlist (연결 모델)
- **역할:** User와 Product를 잇는 다리 역할을 하며, **"언제 찜했는지"**에 대한 추가 정보를 저장합니다.
- **주요 필드:** `user_id(FK)`, `product_id(FK)`, `created_at`

---

## 3. 비즈니스 로직 및 설계 핵심 포인트

### 🛠 Association Proxy 기술 적용
단순히 연결 테이블을 거쳐가는 것이 아니라, `user.wishlist_items.append(product)` 처럼 파이썬의 리스트를 다루듯 직관적으로 코드를 짤 수 있게 설계했습니다.



### 🔍 API 엔드포인트 명세
1. **[POST] /users**: 사용자 생성
2. **[GET] /users**: 모든 사용자 목록 조회
3. **[POST] /users/{user_id}/wishlist/{product_id}**: 위시리스트 추가 (중복 체크 로직 포함)
4. **[GET] /users/{user_id}/wishlist**: 특정 사용자의 찜 목록 조회 (찜한 시점 포함)
5. **[DELETE] /users/{user_id}/wishlist/{product_id}**: 위시리스트 삭제 (찜 취소)

---

## 4. 핵심 구현 코드 (SQLAlchemy)

````python
from datetime import datetime
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.orm import relationship, declarative_base
from sqlalchemy.ext.associationproxy import association_proxy

Base = declarative_base()

class Wishlist(Base):
    __tablename__ = 'wishlists'
    
    # 변수명: fk_user_id (실무형 명명 규칙 제안)
    user_id = Column(Integer, ForeignKey('users.user_id'), primary_key=True)
    product_id = Column(Integer, ForeignKey('products.product_id'), primary_key=True)
    created_at = Column(DateTime, default=datetime.now)

    user = relationship("User", back_populates="wishlist_entries")
    product = relationship("Product")

class User(Base):
    __tablename__ = 'users'
    
    user_id = Column(Integer, primary_key=True)
    nickname = Column(String(50), nullable=False)
    
    wishlist_entries = relationship("Wishlist", back_populates="user", cascade="all, delete-orphan")
    
    # 지름길(Proxy) 설정
    wishlist_items = association_proxy('wishlist_entries', 'product', 
                                     creator=lambda p: Wishlist(product=p))

class Product(Base):
    __tablename__ = 'products'
    
    product_id = Column(Integer, primary_key=True)
    product_name = Column(String(100), nullable=False)