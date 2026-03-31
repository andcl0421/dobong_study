# 📅 2026-03-31 

## 🎯 학습 목표
- **LangGraph**의 기본 구조(Nodes, Edges) 이해
- **State(상태)**를 활용한 데이터 관리 및 `Annotated` 활용법 숙지
- **Conditional Edges**를 이용한 `if-else` 형태의 루프(Loop) 구조 설계
- `stream()`을 활용한 에이전트 실행 과정 실시간 모니터링

---

## 🏗️ 설계 핵심 포인트 (Business Logic)

### 1. State: 공용 서류 가방
단순한 변수 선언이 아니라, 에이전트 내의 모든 노드가 공유하는 **메모리 공간**입니다. 
- `TypedDict`를 사용해 데이터 규격을 정의합니다.
- `Annotated[list, add_messages]`를 사용하면 대화 기록을 덮어쓰지 않고 차곡차곡 쌓을 수 있습니다.

### 2. Workflow (Builder): 설계도와 조립
- `StateGraph` 객체(Builder)를 생성하여 전체 흐름을 정의합니다.
- **Nodes**: 실제 일을 하는 부서 (함수 단위)
- **Edges**: 업무 전달 경로 (일방통행 또는 조건부)

### 3. Conditional Edges: 지능형 라우팅
일반적인 `if-else` 문처럼 작동하며, 이전 단계의 결과(`State`)를 보고 다음 행선지를 결정합니다. 이번 실습에서는 '품질 미흡 시 재작성' 루프를 만드는 핵심 역할을 했습니다.

---

## 💻 실습 최종 코드 (글 작성 및 자가 수정 에이전트)

````python
from typing import Annotated, TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI

# 1. State 정의 (서류 가방 설계)
class WritingState(TypedDict):
    topic: str
    draft: str
    feedback: str
    passed: bool
    attempts: int

# 2. 노드 함수 정의
def write_content(state: WritingState):
    """주제를 바탕으로 글을 작성하거나 피드백을 반영해 수정합니다."""
    current_attempts = state.get("attempts", 0) + 1
    # 실무 Tip: 피드백 유무에 따라 프롬프트를 가변적으로 구성
    return {
        "draft": f"[{state['topic']}]에 대한 작성글 (시도 {current_attempts}회차)", 
        "attempts": current_attempts
    }

def review_content(state: WritingState):
    """작성된 글을 검토하여 통과 여부와 피드백을 결정합니다."""
    # 실제 환경에서는 LLM이 'PASS' 혹은 'FAIL'을 판단하도록 구성
    is_passed = state["attempts"] >= 2  # 예시로 2회차에 통과하도록 설정
    return {
        "passed": is_passed, 
        "feedback": "내용을 조금 더 보충해주세요." if not is_passed else ""
    }

# 3. 라우팅 함수 (if-else 갈림길)
def route_by_result(state: WritingState):
    if state["passed"] or state["attempts"] >= 3:
        return "END"
    return "write"

# 4. 그래프 조립 (Builder 활용)
builder = StateGraph(WritingState)

builder.add_node("write", write_content)
builder.add_node("review", review_content)

builder.add_edge(START, "write")
builder.add_edge("write", "review")

# 조건부 엣지 설정
builder.add_conditional_edges(
    "review",
    route_by_result,
    {
        "END": END,
        "write": "write"
    }
)

writing_graph = builder.compile()

# 5. 실행 (Stream 모드)
inputs = {"topic": "AI가 교육에 미치는 영향", "attempts": 0}
for chunk in writing_graph.stream(inputs):
    print(chunk)


    Troubleshooting (오늘의 해결 과정)
❓ 이슈: 왜 AI의 대답이 자꾸 덮어씌워질까?
원인: State 정의 시 단순히 list로만 선언하면 새로운 데이터가 들어올 때 이전 데이터를 삭제하고 덮어쓰는 기본 동작 때문임.

해결: Annotated[list, add_messages]를 사용하여 리스트에 새로운 내용이 **추가(Append)**되도록 규칙을 지정함.

❓ 이슈: add_conditional_edges에서 길을 잃는 문제
원인: 라우팅 함수가 리턴하는 '글자'와 add_node로 등록한 '방 이름'이 일치하지 않음.

해결: 리턴값과 노드 이름을 정확히 매칭하고, strip().lower()를 사용해 공백이나 대소문자 오타로 인한 버그를 방지함.