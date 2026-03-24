# 📅 2026-03-24 (화) - LangChain Advanced: Structured Output & Custom Tools

## 🚀 실무 설계 핵심 포인트 (Business Logic)

### 1. Structured Output: AI의 답변을 데이터로 변환
- **문제 의식**: LLM의 자유로운 텍스트 응답은 실제 서비스 코드(Python/JS)에서 제어하기 어렵습니다.
- **해결책**: `Pydantic`과 `with_structured_output()`을 결합하여 응답을 **객체(Object)** 형태로 강제합니다.
- **실무 활용**: 감성 분석 결과 저장, 이력서 정보 추출, 대화 내용 요약 및 DB 저장 등에 필수적입니다.

### 2. Tool & Function Calling: AI에게 '손과 발' 달아주기
- **개념**: LLM은 똑똑하지만 외부 실시간 정보(날씨, 뉴스, DB)를 모릅니다. `@tool`은 LLM이 필요할 때 호출할 수 있는 **함수 목록**입니다.
- **작동 원리**: LLM이 스스로 판단하여 `tool_calls`를 생성하면, 개발자가 해당 함수를 실행하고 결과를 다시 LLM에게 전달하는 **Agent Loop**가 형성됩니다.

### 3. Enum & Field: AI의 '가드레일'
- `Enum`을 사용하여 AI가 정해진 카테고리(예: junior, mid, senior) 내에서만 답변하도록 제한하여 데이터 무결성을 보장합니다.
- `Field(description=...)`은 AI에게 주는 **'힌트'**입니다. 설명이 구체적일수록 AI의 도구 선택 정확도가 올라갑니다.

---

## 🛠️ 주요 실습 및 트러블슈팅 (Troubleshooting)

### [실습 1] AI 이력서 분석기 및 상담 요약봇
- **상황**: 비정형 텍스트(이력서, 채팅로그)에서 핵심 정보만 뽑아내야 함.
- **해결**: `ResumeAnalysis`와 `ConsultingSummary` 모델을 설계하여 복잡한 대화 내용을 JSON 형태로 정제.
- **포인트**: `RunnableWithMessageHistory`를 통해 대화 맥락을 유지한 뒤, 마지막에만 구조화된 요약을 실행하는 2단계 프로세스 구축.

### [실습 2] TMDB & Tavily 커스텀 도구 제작
- **상황**: AI가 최신 영화 정보나 실시간 웹 검색 결과를 답변에 반영해야 함.
- **해결**: API Key를 환경변수(`.env`)로 관리하고, `requests` 라이브러리를 활용해 외부 데이터를 가져오는 `@tool` 구현.
- **Troubleshooting**: 
  - **문제**: AI가 도구를 써야 할 때 안 쓰고 추측해서 답변함.
  - **원인**: 도구 설명(docstring)이 부족하거나 모델의 `temperature`가 높음.
  - **해결**: `temperature=0`으로 설정하고, 시스템 프롬프트에 **Few-shot(예시)**을 추가하여 도구 사용 시점을 명확히 교육함.

---

## 💻 핵심 코드 요약

### 1. Structured Output 정의 (Pydantic)
````python
class ResumeAnalysis(BaseModel):
    name: str = Field(description="지원자 이름")
    level: Level = Field(description="경력 레벨 (junior/mid/senior)")
    tech_stack: list[str] = Field(description="보유 기술")

# 체인 연결
structured_llm = llm.with_structured_output(ResumeAnalysis)
analysis_chain = prompt | structured_llm

2. Custom Tool 정의 (@tool)

@tool
def search_movies(query: str) -> str:
    """영화를 검색합니다. 제목이나 키워드를 입력하세요. 예: '인셉션'"""
    # ... API 호출 로직 ...
    return movie_data

# LLM에 도구 바인딩
llm_with_tools = llm.bind_tools([search_movies])