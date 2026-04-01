# 📅 작성일: 2026-04-01
# ★  LangGraph Memory & State Mastery

이 저장소는 LangGraph를 활용하여 에이전트의 **단기 기억(Checkpointer)**, **장기 기억(Store)**, 그리고 효율적인 토큰 관리를 위한 **대화 요약(Summary)** 기법을 실무 수준으로 구현한 가이드를 담고 있습니다.

---

## 1. 비즈니스 로직 및 설계 핵심 포인트

### 🧠 에이전트의 두 가지 기억 장치
실무 인공지능은 사람처럼 두 가지 방식의 기억을 사용합니다.
1. **단기 기억 (Short-term Memory)**: 현재 나누고 있는 대화의 흐름을 기억합니다. `thread_id`를 사용하여 "아까 내가 말한 그거"를 알아듣게 합니다.
2. **장기 기억 (Long-term Memory)**: 대화가 끝나도 사라지지 않는 유저의 선호도, 직업, 고정 규칙 등을 `Store`에 저장하여 다음 대화에서도 활용합니다.

### 🧹 효율적인 토큰 관리 (Summarization)
대화가 길어지면 비용이 비싸지고 속도가 느려집니다. 본 프로젝트는 일정 토큰(`MAX_TOKENS`)이 넘어가면 과거 대화를 요약하고 메모리에서 삭제하여 최적의 성능을 유지합니다.

---

## 2. 주요 기능 구현 (Code Example)

### ① 영화 검색 & 상세 정보 조회 도구
에이전트가 사용할 외부 API(TMDB) 연결 도구입니다.

````python
import os
import requests
from langchain_core.tools import tool

@tool
def search_movies(query: str) -> str:
    """영화 제목으로 검색하여 ID와 기본 정보를 가져옵니다."""
    response = requests.get(
        "[https://api.themoviedb.org/3/search/movie](https://api.themoviedb.org/3/search/movie)",
        headers={"Authorization": f"Bearer {os.getenv('TMDB_API_KEY')}"},
        params={"query": query, "language": "ko-KR"},
    )
    results = response.json().get("results", [])[:5]
    return "\n".join([f"- [{m['id']}] {m['title']} ({m['release_date'][:4]})" for m in results])

@tool
def get_movie_detail(movie_id: str) -> str:
    """영화 ID로 감독과 러닝타임을 조회합니다."""
    url = f"[https://api.themoviedb.org/3/movie/](https://api.themoviedb.org/3/movie/){movie_id}"
    params = {"append_to_response": "credits", "language": "ko-KR"}
    headers = {"Authorization": f"Bearer {os.getenv('TMDB_API_KEY')}"}
    
    data = requests.get(url, headers=headers, params=params).json()
    runtime = data.get("runtime", 0)
    crew = data.get("credits", {}).get("crew", [])
    directors = [m['name'] for m in crew if m['job'] == 'Director']
    
    return f"감독: {', '.join(directors)} / 러닝타임: {runtime}분"

   

 ② 통합 에이전트 그래프 (Memory + Summary) ★
 from typing import Annotated
from langgraph.graph import StateGraph, START, END, MessagesState
from langchain_core.messages import SystemMessage, RemoveMessage, trim_messages

# 1. 상태 정의 (메시지 + 요약본) ★
class CombinedState(MessagesState):
    summary: str

# 2. 요약 로직 함수 ★
def summarize_conversation(state: CombinedState):
    messages = state["messages"]
    # 최근 100토큰만 남기고 나머지는 요약 대상으로 분리
    kept = trim_messages(messages, max_tokens=100, strategy="last", token_counter=llm)
    kept_ids = {msg.id for msg in kept}
    old_messages = [msg for msg in messages if msg.id not in kept_ids]
    
    # 요약문 생성 및 구 메시지 삭제
    summary_text = llm.invoke(f"다음 내용을 요약해줘: {old_messages}").content
    delete_actions = [RemoveMessage(id=msg.id) for msg in old_messages]
    
    return {"summary": summary_text, "messages": delete_actions}

# 3. 그래프 조립 ★
workflow = StateGraph(CombinedState)
workflow.add_node("chatbot", chatbot_with_summary)
workflow.add_node("summarize", summarize_conversation)

workflow.add_edge(START, "chatbot")
workflow.add_conditional_edges(
    "chatbot", 
    lambda state: "summarize" if len(state['messages']) > 10 else END
)
workflow.add_edge("summarize", END)

3. 트러블슈팅 (Troubleshooting)
❓ 문제: MemorySaver를 썼는데 서버를 껐다 켜니 이름 기억을 못 해요!
원인: MemorySaver는 휘발성 메모리에 저장되므로 프로세스가 종료되면 데이터가 날아갑니다.

해결: 실무 환경(Production)에서는 PostgresSaver를 사용하여 DB에 영구적으로 저장해야 합니다.

Python
# 실무용 영구 저장 설정
from langgraph.checkpoint.postgres import PostgresSaver
with PostgresSaver.from_conn_string(DB_URL) as saver:
    app = workflow.compile(checkpointer=saver)
❓ 문제: RemoveMessage를 썼는데 메시지가 안 사라져요.
원인: RemoveMessage는 반드시 삭제하고자 하는 메시지의 id 값을 정확히 전달해야 합니다.

해결: trim_messages를 통해 걸러진 메시지들의 id를 리스트로 만들어 반환하도록 로직을 수정했습니다.