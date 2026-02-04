
`feat: FastAPI 기반 분리 설계 구조를 적용한 게시판 CRUD 핵심 로직 구현`

**상세 내용:**
- MVC 패턴을 응용한 Router-Service-Repository-Schema 레이어 분리 설계
- Pydantic을 활용한 입력(PostCreate) 및 출력(PostDetailResponse) 데이터 규격화
- 유효성 검사 로직(HTTPException) 및 비즈니스 로직 분리
- 메모리 기반 저장소를 활용한 게시글 저장 및 관리 기능 구현

---

##  프로젝트 개요
FastAPI를 활용하여 MVC(Model-View-Controller) 패턴의 장점을 도입한 **게시판 CRUD API 시스템**입니다. 단순히 기능을 만드는 것에 그치지 않고, 데이터의 흐름을 명확히 제어하기 위해 비즈니스 로직과 데이터베이스 접근 로직을 분리하여 설계했습니다.

##  아키텍처 핵심 설계 포인트
실무에서는 코드를 한 파일에 몰아넣지 않고, 역할에 따라 '방'을 나누는 것이 중요합니다.

1. **Router (`post_routers.py`)**: 
   - **문지기 역할**: 클라이언트의 요청(HTTP Request)을 가장 먼저 받아 적절한 서비스로 연결합니다.
2. **Schema (`post.py`)**: 
   - **데이터 규격**: 주고받는 데이터의 '형식'을 정의합니다. 보안을 위해 입력용(`PostCreate`)과 출력용(`PostDetailResponse`)을 엄격히 분리했습니다.
3. **Service (`post_service.py`)**: 
   - **비즈니스 브레인**: 데이터가 비어있는지 검증하거나, 핵심 비즈니스 규칙을 결정합니다.
4. **Repository (`post_repository.py`)**: 
   - **창고 관리자**: 실제 데이터베이스(메모리 리스트)에 접근하여 데이터를 넣고 빼는 일만 전담합니다.

---

##  주요 비즈니스 로직 및 예시 코드

### 1. 스키마 설계 (Data Models)
실무에서는 단순히 `id`라고 하기보다 객체의 정체성을 살려 `post_id`라고 부르는 것이 혼선을 줄여줍니다.

````python
from pydantic import BaseModel
from typing import Optional

# [Entity] 실제 데이터의 원형
class Post:
    def __init__(self, title: str, content: str):
        self.post_id = None  # 생성 시 자동 부여 예정
        self.title = title
        self.content = content

# [Schema] 데이터 입력용 (클라이언트 -> 서버)
class PostCreate(BaseModel):
    title: str
    content: str

# [Schema] 데이터 출력용 (서버 -> 클라이언트)
class PostDetailResponse(BaseModel):
    post_id: int
    title: str
    content: str

2. 서비스 로직 (Business Logic)

from fastapi import HTTPException, status

class PostService:
    def create_post(self, data: PostCreate):
        # [비즈니스 검증] 제목이나 내용이 공백인 경우를 차단
        if not data.title.strip():
            raise HTTPException(
                status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
                detail="제목은 필수 입력사항입니다. 공백은 허용되지 않습니다."
            )
        
        # 새로운 게시글 객체 생성 (Entity화)
        new_post_entity = Post(title=data.title, content=data.content)
        
        # 저장소(Repository)에 저장을 위임
        return post_repository.save(new_post_entity)

트러블슈팅 (Troubleshooting)
Issue: Postman 테스트 중 422 Unprocessable Entity 에러 발생
문제 상황: 제목(title)을 빈 값으로 보냈을 때 서버 에러가 아닌 422 에러가 발생하며 요청이 거절됨.

원인 파악: PostService에서 data.title.strip() 검증을 통해 값이 비어있을 경우 의도적으로 예외를 던지도록 설계했기 때문임.

해결 방법: 이는 시스템 버그가 아니라, 잘못된 데이터 유입을 막는 정상적인 검증 과정임. 클라이언트에게 정확한 가이드를 주기 위해 HTTPException을 사용하여 상태 코드와 상세 메시지를 전달하도록 처리함.