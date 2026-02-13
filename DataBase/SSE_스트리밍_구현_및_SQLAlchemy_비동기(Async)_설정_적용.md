#  [2026-02-13] 실시간 데이터 처리(SSE)와 비동기(Async) DB 설계

## 1. 실시간 데이터의 마법: SSE (Server-Sent Events)
일반적인 API가 "손님이 주문하고 음식이 나올 때까지 벨이 울리기를 기다리는 방식"이라면, **SSE**는 "주방장이 요리 과정마다 한 마디씩 진행 상황을 알려주는 방식"입니다.

* **비즈니스 로직 핵심**:
    * **단방향 통신**: 서버가 클라이언트에게 일방적으로 데이터를 밀어줍니다.
    * **가벼움**: WebSocket보다 구현이 간단하며, 일반 HTTP 프로토콜을 사용합니다.
    * **자동 재연결**: 연결이 끊기면 브라우저가 똑똑하게 다시 연결을 시도합니다.
* **주요 용도**: ChatGPT 같은 LLM 스트리밍 답변, 실시간 배달 위치 알림, 주식 시세 업데이트.



---

## 2. 서버의 효율 극대화: Sync vs Async
서버가 일을 처리하는 '태도'의 차이입니다.

* **동기(Sync - `def`)**: 한 번에 한 가지 일만 합니다. 점원이 커피가 나올 때까지 다른 손님을 받지 않고 서 있는 것과 같습니다. (Thread Pool 활용)
* **비동기(Async - `async def`)**: 기다리는 시간에 다른 일을 합니다. 점원이 주문을 받고, 커피가 나오는 동안 다음 손님을 받는 것과 같습니다. (Event Loop 활용)
* **주의사항**: `async def` 안에서는 반드시 비동기 전용 라이브러리(예: `asyncio.sleep`, `httpx`)를 사용해야 서버가 멈추지 않습니다.

---

## 3. DB 성능의 핵심: Connection Pool
매번 DB에 새로 접속하는 것은 큰 비용(시간)이 듭니다.
* **설계 포인트**: 미리 DB로 가는 '고속도로 차선(Connection)'을 5~10개 정도 뚫어놓고, 요청이 오면 비어있는 차선을 빌려줬다가 작업이 끝나면 회수하는 방식입니다.
* **설정 예시**: `pool_size=5`, `max_overflow=10` 설정을 통해 동시 접속자가 늘어날 때 유연하게 대응합니다.

---

## 4.  트러블슈팅 (Troubleshooting)

### 이슈: 비동기(Async) 환경에서 Lazy Loading 에러
* **상세 내용**: 비동기 엔드포인트에서 게시글을 가져온 뒤, `post.comments`를 참조하려고 할 때 `MissingGreenlet` 에러가 발생하며 데이터 조회가 실패함.
* **원인**: 비동기 방식에서는 속성에 접근할 때 자동으로 쿼리를 날리는 '지연 로딩(Lazy Loading)'을 지원하지 않음. (속성 접근 시 `await`를 쓸 수 없기 때문)
* **해결책 (Eager Loading)**: `selectinload`를 사용하여 쿼리 시점에 연관 데이터를 미리 한꺼번에 가져오도록 수정.

````python
# 수정 전 (에러 발생)
post = await db.get(Post, 1)
print(post.comments) # Error! -> MissingGreenlet 발생

# 수정 후 (정상 작동)
stmt = select(Post).where(Post.id == 1).options(selectinload(Post.comments))
result = await db.execute(stmt)
post_data = result.scalar_one()
print(post_data.comments) # 이미 메모리에 로드되어 있으므로 OK!

5.  실무형 예시 코드

import asyncio
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from sqlalchemy.orm import selectinload
from fastapi.responses import StreamingResponse

router = APIRouter(prefix="/api/v1", tags=["Real-time & Async"])

# [실무 제안] 변수명은 snake_case를 사용하며, 역할이 명확한 이름을 사용합니다.
# token_generator: 스트리밍 데이터를 생성하는 생성기 역할임을 명시

@router.get("/chat/stream")
async def stream_ai_response(user_input: str):
    """
    AI 답변을 한 글자씩 실시간으로 전송하는 SSE 엔드포인트
    """
    async def token_generator():
        mock_response = f"입력하신 '{user_input}'에 대한 AI 답변입니다."
        for token in mock_response:
            # SSE 형식 표준 준수 (data: {message}\n\n)
            yield f"data: {token}\n\n"
            await asyncio.sleep(0.05)
        yield "data: [DONE]\n\n"

    return StreamingResponse(token_generator(), media_type="text/event-stream")

@router.get("/posts/{post_id}")
async def get_post_with_comments(
    post_id: int, 
    db_session: AsyncSession = Depends(get_async_db)
):
    """
    비동기 DB 세션을 사용하여 게시글과 댓글을 즉시 로딩(Eager Loading)으로 조회
    """
    # 멘토 제안: scalars().one_or_none()을 조합하여 객체 하나를 안전하게 추출합니다.
    query_stmt = (
        select(Post)
        .where(Post.id == post_id)
        .options(selectinload(Post.comments))
    )
    result = await db_session.execute(query_stmt)
    target_post = result.scalar_one_or_none()
    
    return target_post