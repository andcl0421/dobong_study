# 2026-01-22 학습 요약

## 1. 오늘 학습 요약 (핵심 정리)
- 외부 API(네이버 뉴스)와 LLM(OpenAI)을 결합하여 실시간 정보를 기반으로 답변하는 **AI 에이전트**의 기본 구조를 학습했습니다.
- 단순한 질의응답을 넘어 AI가 스스로 판단하여 특정 도구를 꺼내 쓰는 **Function Calling** 기술을 실전 코드에 적용해 보았습니다.

## 2. 배운 개념 정리
- **Function Calling**: LLM이 사용자의 질문에 답하기 위해 외부 함수(API 등)가 필요하다고 판단하면, 해당 함수의 이름과 인자값을 생성해 주는 기술입니다.
- **API Key 관리**: `.env` 파일을 통해 보안이 중요한 API 키를 관리하고 `os.getenv()`로 안전하게 불러오는 방법을 익혔습니다.
- **RAG(검색 증강 생성)**: 모델이 학습하지 않은 최신 데이터를 외부에서 검색해와서 답변의 정확도를 높이는 개념을 실습했습니다.

## 3. 오늘 작성한 코드 리뷰
```python
import os
import requests
import json
from openai import OpenAI

# 1. 네이버 뉴스 API 호출 함수 정의
def naver_get_news(search_query):
    # AI가 결정한 search_query를 인자로 받아 네이버 뉴스를 검색함
    NAVER_URL = '[https://openapi.naver.com/v1/search/news.json](https://openapi.naver.com/v1/search/news.json)'
    headers = {
        'X-Naver-Client-Id': os.getenv('NAVER_CLIENT_ID'), 
        'X-Naver-Client-Secret': os.getenv('NAVER_CLIENT_SECRET') 
    }
    params = {'query': search_query, 'display': 3}
    
    response = requests.get(NAVER_URL, params=params, headers=headers)
    return response.json().get('items', []) # 뉴스 리스트만 추출해서 반환

# 2. 에이전트 실행 로직
def run_conversation():
    # 사용자의 요청을 담은 메시지 리스트
    messages = [{"role": "user", "content": "요즘 삼성전자 관련 뉴스 좀 요약해줘."}]

    # AI에게 사용 가능한 도구(tools) 정보를 함께 전달
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=messages,
        tools=tools, # 미리 정의한 도구 설명서
        tool_choice="auto"
    )
    
    # AI가 함수 호출이 필요하다고 판단한 경우(tool_calls) 실행
    # ... (생략) 함수 실행 결과를 다시 LLM에 넣어 최종 답변 생성
```
> **주의점**: `naver_get_news` 함수 내부에서 `input()`을 사용하면 자동화 흐름이 끊기므로, 매개변수(`query`)를 직접 활용하는 것이 실무적입니다.

## 4. 실수/에러와 해결 과정
- **AuthenticationError (401)**: OpenAI 클라이언트를 초기화할 때 네이버 API 키를 넣어서 발생했습니다. → `os.getenv`로 각 서비스에 맞는 정확한 환경 변수를 지목하여 해결했습니다.
- **NameError**: 변수를 정의하기 전에 호출하여 발생했습니다. → `load_dotenv()` 이후 변수에 값을 담는 순서를 명확히 정렬했습니다.
- **SyntaxError**: 인자 사이의 콤마(`,`) 누락으로 발생했습니다. → 코드의 구조를 다시 살피고 문법 규칙을 준수했습니다.

## 5. 실무/RAG 관점의 참견
- 실무에서는 네이버 뉴스 결과에 포함된 `<b>`, `&quot;` 같은 HTML 태그를 정규표현식으로 제거(Cleaning)한 뒤 AI에게 전달해야 답변의 질이 훨씬 좋아집니다.
- 검색 결과가 없을 경우를 대비해 예외 처리를 꼼꼼히 하는 것이 중요하며, 향후 뉴스 본문까지 크롤링하여 요약하는 방식으로 확장하면 더 강력한 RAG 서비스를 만들 수 있습니다.