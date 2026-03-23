# 📅 2026-03-23 | LangChain 핵심 개념 및 프롬프트 엔지니어링 실습

## 🚀 비즈니스 로직 및 설계 핵심 포인트

### 1. LangChain: LLM 앱의 '조립 키트'
단순히 OpenAI API를 호출하는 것을 넘어, **프롬프트 + 모델 + 출력 파서**를 하나의 사슬(Chain)처럼 엮는 법을 학습했습니다.
- **LCEL(LangChain Expression Language):** `|` (파이프) 연산자를 사용하여 데이터가 흐르는 길을 만드는 방식입니다.
- **표준 인터페이스:** `invoke()` 메서드 하나로 모든 구성 요소를 실행할 수 있는 일관성을 배웠습니다.

### 2. 프롬프트 엔지니어링의 전략적 접근
AI에게 단순히 질문하는 것이 아니라, **'일 잘하는 비서'**로 만드는 기술을 익혔습니다.
- **Few-shot:** 예시를 줘서 AI가 답변 형식을 완벽히 복사하게 함 (감정 분석기 실습).
- **Role Prompting:** CEO, CTO 등 페르소나를 부여해 관점의 차이를 만듦.
- **CoT(Chain of Thought):** "단계별로 생각하라"고 지시하여 복잡한 논리 문제를 해결함.

---

## 🛠️ 주요 구현 코드 (Example)

### 말투 변환기 및 감정 분석 체인
변수 주입형 템플릿과 Few-shot 기법을 적용한 실무형 코드 예시입니다.

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 1. 모델 및 파서 준비
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
parser = StrOutputParser()

# 2. 감정 분석 체인 (Few-shot 적용)
sentiment_prompt = ChatPromptTemplate.from_messages([
    ("system", "너는 텍스트 감정 분석 전문가야. 아래 형식을 반드시 지켜줘."),
    ("human", "이 제품 최악이에요!"),
    ("ai", "감정: 부정\n강도: 강함\n근거: '최악' 표현 사용"),
    ("human", "{input_text}")
])

chain = sentiment_prompt | llm | parser
print(chain.invoke({"input_text": "배송은 느리지만 물건은 좋네요."}))
```

---

## 🔍 코드 리뷰 및 더 나은 방향 제안 (Naming Convention)

실무에서 더 직관적인 코드를 위해 다음과 같은 변수명을 제안하고 반영했습니다.

| 기존 변수명 | 제안 변수명 | 이유 |
| :--- | :--- | :--- |
| `prompt_few` | `sentiment_analysis_prompt` | 프롬프트의 '기법'보다 '목적'이 드러나야 유지보수가 쉽습니다. |
| `res` | `ai_response` | 단순히 결과라는 뜻보다 AI의 응답임을 명확히 합니다. |
| `styles` | `target_dialects` | 말투 변환의 대상임을 구체적으로 명시합니다. |

---

## ⚠️ 트러블슈팅 (Troubleshooting)

### 1. 패키지 이름 충돌 (Shadowing)
- **문제:** 폴더 이름을 `langchain`으로 지었더니 `uv add langchain` 실행 시 자기 자신을 참조하는 에러 발생.
- **원인:** 로컬 폴더명이 설치하려는 라이브러리 명칭과 같으면 파이프라인이 꼬임.
- **해결:** `pyproject.toml`에서 `name`을 `langchain-practice`로 수정 후 재설치하여 해결.

### 2. 가상환경 경로 불일치 경고
- **문제:** `warning: VIRTUAL_ENV... does not match` 발생.
- **해결:** 명령어 뒤에 `--active` 옵션을 붙여 현재 활성화된 가상환경을 강제로 지정하여 해결.

