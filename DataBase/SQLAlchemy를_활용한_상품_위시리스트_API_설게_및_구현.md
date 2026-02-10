1. 비즈니스 로직 및 설계 핵심 포인트
다대다(N:M) 관계의 구현: User와 Product는 서로 여러 개를 가질 수 있습니다. 이 사이를 Wishlist라는 연결 모델이 이어주며, '찜한 시간'이라는 추가 비즈니스 데이터를 관리합니다.

Association Proxy 활용: 개발자가 연결 모델을 직접 생성하는 번거로움을 줄이고, 객체 지향적인 코드를 작성할 수 있도록 프록시 패턴을 적용했습니다.

중복 체크 로직: 동일한 유저가 동일한 상품을 중복해서 찜하지 않도록 API 레벨에서 유효성 검사를 수행합니다.

from datetime import datetime
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.orm import relationship, declarative_base, sessionmaker
from sqlalchemy.ext.associationproxy import association_proxy
from sqlalchemy import create_engine

Base = declarative_base()

# 1. 연결 모델 (Wishlist)
class Wishlist(Base):
    __tablename__ = 'wishlists'
    
    # 변수명 제안: fk_user_id (외래키임을 명확히 표시)
    user_id = Column(Integer, ForeignKey('users.user_id'), primary_key=True)
    product_id = Column(Integer, ForeignKey('products.product_id'), primary_key=True)
    created_at = Column(DateTime, default=datetime.now)

    # 양방향 관계 설정
    user = relationship("User", back_populates="wishlist_entries")
    product = relationship("Product")

# 2. 사용자 모델
class User(Base):
    __tablename__ = 'users'
    
    user_id = Column(Integer, primary_key=True)
    nickname = Column(String(50), nullable=False)
    
    # 연결 모델로의 관계
    wishlist_entries = relationship("Wishlist", back_populates="user", cascade="all, delete-orphan")
    
    # ★ Association Proxy: 중간 모델을 건너뛰고 바로 Product 객체에 접근
    # user.wishlist_items.append(some_product) 로 사용 가능
    wishlist_items = association_proxy('wishlist_entries', 'product', 
                                     creator=lambda p: Wishlist(product=p))

# 3. 상품 모델
class Product(Base):
    __tablename__ = 'products'
    
    product_id = Column(Integer, primary_key=True)
    product_name = Column(String(100), nullable=False)

# --- API 비즈니스 로직 예시 ---

def add_to_wishlist(db_session, target_user, target_product):
    """
    위시리스트 추가 로직 (중복 확인 포함)
    """
    # 1. 이미 찜했는지 확인
    if target_product in target_user.wishlist_items:
        return {"message": "이미 위시리스트에 있는 상품입니다."}, 400
    
    # 2. Proxy를 이용한 간결한 추가
    target_user.wishlist_items.append(target_product)
    db_session.commit()
    return {"message": "위시리스트 추가 완료"}, 201

def get_user_wishlist(target_user):
    """
    찜한 상품 정보와 시점(created_at) 포함 조회
    """
    result = []
    for entry in target_user.wishlist_entries:
        result.append({
            "product_name": entry.product.product_name,
            "wish_at": entry.created_at.strftime("%Y-%m-%d %H:%M:%S")
        })
    return result


3. 트러블슈팅 (Troubleshooting)
문제 상황: user.wishlist_items.append(product) 실행 시 Wishlist 모델의 created_at이 누락되거나 에러 발생.

원인 파악: Association Proxy 설정 시 creator 인자가 지정되지 않아 중간 모델을 어떻게 생성해야 할지 엔진이 판단하지 못함.

해결 방법: association_proxy의 creator 매개변수에 람다 함수(lambda p: Wishlist(product=p))를 전달하여 연결 객체 생성 방식을 명시함.

재발 방지: 복잡한 N:M 관계에서는 단순히 Secondary 설정보다 Association Object를 사용하고 프록시를 설정하는 것이 데이터 확장성에 유리함을 인지함.