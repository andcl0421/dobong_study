# 2026-01-16 학습 기록: TMDB API를 활용한 데이터 핸들링

## 1. 오늘 학습 요약
- API를 통해 외부 데이터를 가져오고, 이를 사용자 요구사항에 맞게 가공(정렬, 필터링)하는 실전 프로세스를 학습함.
- 특히 대량의 데이터를 수집하기 위한 반복 요청과 효율적인 데이터 정렬 방식을 중점적으로 다룸.

---

## 2. 배운 개념 정리
### (1) API 페이지네이션 (Pagination)
- **정의:** 한 번에 모든 데이터를 주지 않고 페이지 단위로 나누어 제공하는 방식.
- **중요성:** 서버의 부하를 줄이기 위해 대부분의 실무 API가 채택함. 100개 이상의 데이터를 얻으려면 `page` 파라미터를 바꿔가며 여러 번 요청해야 함.

### (2) Lambda (이름 없는 함수)
- **정의:** `lambda 인자: 표현식` 형태의 간단한 일회성 함수.
- **주의점:** 복잡한 로직보다는 `sort`의 `key`처럼 간단한 기준을 제시할 때 사용함.

### (3) List Comprehension (리스트 컴프리헨션)
- **정의:** 리스트를 생성하는 짧고 간결한 파이썬 문법.
- **장점:** 코드가 간결해지고 실행 속도가 빠름.

---

## 3. 오늘 작성한 코드리뷰
```python
import requests
from pprint import pprint

# API 호출을 위한 기본 설정
URL = "[https://api.themoviedb.org/3/movie/now_playing](https://api.themoviedb.org/3/movie/now_playing)"
headers = {"Authorization": f"Bearer {API_KEY}"}
params = {'language': 'ko-kr'}

all_movies = []

try:
    # 1. 100개 이상의 데이터를 위해 여러 페이지를 반복 요청
    for page_num in range(1, 7):
        params['page'] = page_num # 페이지 번호 갱신
        response = requests.get(URL, headers=headers, params=params)
        response.raise_for_status() # 404 등 에러 발생 시 예외 처리
        
        data = response.json()
        # extend를 사용하여 리스트 안에 알맹이들만 합침
        all_movies.extend(data.get('results', []))

    # 2. 평점(vote_average) 기준 내림차순 정렬
    # reverse=True를 주어야 높은 순서대로 정렬됨
    all_movies.sort(key=lambda x: x['vote_average'], reverse=True)

    # 3. 리스트 컴프리헨션으로 제목만 추출 (가공 단계)
    movie_title_list = [movie["title"] for movie in all_movies]

    print(f"수집된 영화 수: {len(movie_title_list)}")
    pprint(movie_title_list[:10]) # 상위 10개 확인

except Exception as e:
    print(f"실행 중 에러 발생: {e}")
```

---

## 4. 실수/에러와 해결 과정
- **SyntaxError (Incomplete input):** `try`만 쓰고 `except`를 쓰지 않아 문장이 덜 끝난 상태로 인식됨 -> `except` 구문을 추가하여 해결.
- **404 Client Error:** URL 주소 끝에 `Popular`라고 대문자를 써서 발생 -> API 명세서대로 소문자 `popular`로 수정하여 해결.
- **데이터 누락:** `requests.get`이 `for`문 밖에 있어 마지막 페이지만 수집됨 -> 요청 로직을 반복문 안으로 넣어 모든 페이지를 수집함.

---

## 5. 실무/RAG 관점의 참견
- **데이터 품질 관리:** RAG(검색 증강 생성) 시스템에서 API 데이터를 사용할 때, 무조건 많이 가져오기보다 오늘처럼 '평점순'으로 정렬하여 신뢰도가 높은 데이터를 먼저 AI에게 제공하는 것이 답변의 질을 결정함.
- **안전한 호출:** 실무에서는 `.get()` 메서드를 사용하여 특정 키가 없을 때 프로그램이 멈추는 것을 방지하는 것이 필수임.