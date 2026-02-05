# [2026-02-05] FastAPI & SQLAlchemy 계층형 게시판 구현

## 1. 프로젝트 개요
본 프로젝트는 **SQLAlchemy ORM**과 **PostgreSQL**을 연동하여, 비전공자 신입 개발자가 실무에서 즉시 활용 가능한 **계층형 아키텍처(Layered Architecture)**를 적용한 게시판 CRUD 시스템입니다.

### 🏗 설계 핵심 포인트 (Architecture)


* **Router**: 클라이언트의 요청(Request)을 접수하고 최종 응답(Response)을 반환합니다.
* **Service**: 비즈니스 로직의 중심지로, **트랜잭션(Transaction)의 경계**를 설정합니다.
* **Repository**: 데이터베이스와의 직접적인 통신(SQL 실행)을 담당하여 데이터 접근 로직을 캡슐화합니다.
* **Model**: 데이터베이스 테이블 구조를 파이썬 클래스로 정의합니다.
* **Schema**: Pydantic을 이용해 데이터 입출력 형식을 검증하고 API 스펙을 관리합니다.

---

## 2. 계층별 상세 구현

### ① 데이터베이스 설정 (`database.py`)
- `load_dotenv`를 통한 환경 변수 관리 및 세션 의존성 주입(`get_db`)을 구현했습니다.

````python
import os
from dotenv import load_dotenv
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

load_dotenv()
DATABASE_URL = os.getenv("DATABASE_URL")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

class Base(DeclarativeBase):
    pass

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

② SQLAlchemy 모델 (models/post.py)
Mapped와 mapped_column을 사용하여 정적 타입 힌팅을 지원하는 2.0 스타일로 정의했습니다.


from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy import String, Text
from database import Base

class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    title: Mapped[str] = mapped_column(String(50), nullable=False)
    content: Mapped[str] = mapped_column(Text, nullable=False)

  ③ Pydantic 스키마 (schemas/post.py)
ConfigDict(from_attributes=True) 설정을 통해 ORM 객체를 Pydantic 모델로 자동 변환합니다.

from pydantic import BaseModel, ConfigDict

class PostCreate(BaseModel):
    title: str
    content: str

class PostListResponse(BaseModel):
    id: int
    title: str
    model_config = ConfigDict(from_attributes=True)

class PostDetailResponse(BaseModel):
    id: int
    title: str
    content: str
    model_config = ConfigDict(from_attributes=True)


④ 데이터 접근 계층 (repositories/post_repository.py)
SQL 쿼리 로직을 서비스 계층과 분리하여 데이터 조작에만 집중합니다.

from sqlalchemy.orm import Session
from sqlalchemy import select
from mysite4.models.post import Post
from mysite4.schemas.post import PostCreate

class PostRepository:
    def save(self, db: Session, new_post: Post):
        db.add(new_post)
        return new_post

    def find_all(self, db: Session):
        return db.scalars(select(Post)).all()

    def find_by_id(self, db: Session, id: int):
        return db.get(Post, id)

    def update(self, db: Session, post: Post, data: PostCreate):
        post.title = data.title
        post.content = data.content
        return post

    def delete(self, db: Session, post: Post):
        db.delete(post)

post_repository = PostRepository()


⑤ 비즈니스 로직 계층 (services/post_service.py)
**트랜잭션(Transaction)**의 원자성을 보장하기 위해 commit과 refresh를 관리합니다.

from sqlalchemy.orm import Session
from fastapi import HTTPException, status
from mysite4.repositories.post_repository import post_repository
from mysite4.models.post import Post

class PostService:
    def create_post(self, db: Session, data: PostCreate):
        new_post = Post(title=data.title, content=data.content)
        post_repository.save(db, new_post)
        db.commit() 
        db.refresh(new_post)
        return new_post

    def read_posts(self, db: Session):
        return post_repository.find_all(db)

    def read_post_by_id(self, db: Session, id: int):
        post = post_repository.find_by_id(db, id)
        if not post:
            raise HTTPException(status.HTTP_404_NOT_FOUND, "존재하지 않는 게시글입니다.")
        return post

    def update_post(self, db: Session, id: int, data: PostCreate):
        post = self.read_post_by_id(db, id)
        updated_post = post_repository.update(db, post, data)
        db.commit()
        db.refresh(updated_post)
        return updated_post

    def delete_post(self, db: Session, id: int):
        post = self.read_post_by_id(db, id)
        post_repository.delete(db, post)
        db.commit()

post_service = PostService()


⑥ 라우터 계층 (routers/post_router.py)
Depends(get_db)를 통한 세션 주입으로 API 엔드포인트를 구현했습니다.

from fastapi import APIRouter, Depends, status
from sqlalchemy.orm import Session
from database import get_db
from mysite4.services.post_service import post_service
from mysite4.schemas.post import PostCreate, PostDetailResponse, PostListResponse

router = APIRouter(prefix="/posts-db", tags=["posts"])

@router.post("", response_model=PostDetailResponse, status_code=status.HTTP_201_CREATED)
def create_post(data: PostCreate, db: Session = Depends(get_db)):
    return post_service.create_post(db, data)

@router.get("", response_model=list[PostListResponse])
def read_posts(db: Session = Depends(get_db)):
    return post_service.read_posts(db)

@router.get("/{id}", response_model=PostDetailResponse)
def read_post(id: int, db: Session = Depends(get_db)):
    return post_service.read_post_by_id(db, id)

@router.put("/{id}", response_model=PostDetailResponse)
def update_post(id: int, data: PostCreate, db: Session = Depends(get_db)):
    return post_service.update_post(db, id, data)

@router.delete("/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int, db: Session = Depends(get_db)):
    post_service.delete_post(db, id)


트러블슈팅 (Troubleshooting)
이슈 1: 데이터베이스 테이블 미생성
문제: Base.metadata.create_all(bind=engine)을 실행했음에도 PostgreSQL에 테이블이 생성되지 않음.

원인: SQLAlchemy의 Base 객체가 해당 모델 클래스를 인지하기 위해서는 실행 시점에 모델 파일이 import 되어 있어야 함.

해결: main.py 상단에 from mysite4.models.post import Post를 추가하여 해결함.

이슈 2: Pydantic 응답 시 dict 변환 에러
문제: DB에서 조회한 SQLAlchemy 객체를 Pydantic 모델을 통해 반환하려 할 때 데이터 형식이 맞지 않아 에러 발생.

해결: Pydantic 스키마 정의부에 model_config = ConfigDict(from_attributes=True)를 추가하여 ORM 객체의 속성을 자동으로 읽어오도록 설정함.


