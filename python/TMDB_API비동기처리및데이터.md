#  TMDB API 비동기 수집 및 Git 트러블슈팅

## 1. 오늘 학습 요약 (핵심 정리)
- 동기(Sync) 방식의 한계를 극복하기 위해 `asyncio`와 `aiohttp`를 이용한 비동기 데이터 수집을 학습함.
- 여러 API 호출을 동시에 처리하여 전체 실행 시간을 획기적으로 단축함.
- Git 협업 과정에서 발생하는 경로 오류 및 병합 충돌 해결 능력을 배양함.

## 2. 배운 개념 정리
### (1) 비동기 프로그래밍 (Asynchronous)
- **정의:** 한 작업이 끝나기를 기다리지 않고 다음 작업을 동시에 진행하는 방식.
- **왜 중요한지:** 영화 20개의 상세 정보를 하나씩 가져오면(동기) 20초가 걸리지만, 한꺼번에 가져오면(비동기) 약 1~2초 내에 완료 가능함.
- **주의점:** `async def`, `await` 키워드를 정확한 위치에 사용해야 하며, `aiohttp`와 같은 전용 라이브러리가 필요함.

### (2) 세마포어 (Semaphore)
- **정의:** 동시에 실행할 수 있는 작업의 수를 제한하는 장치.
- **용도:** 서버에 한꺼번에 너무 많은 요청을 보내서 차단당하는 것을 방지함 (실무 필수 기술).

## 3. 오늘 작성한 코드리뷰
```python
import asyncio
import aiohttp
import json

# [비동기 일꾼 함수]
async def fetch_with_semaphore(semaphore, session, url):
    # 세마포어를 통해 동시 실행 개수 조절
    async with semaphore:
        async with session.get(url) as response:
            # 서버 응답 상태 확인 및 텍스트 데이터 반환
            print(f"요청 완료: {url} (상태: {response.status})")
            return await response.text()

# [비동기 감독 함수]
async def main():
    # 1. 영화 목록 가져오기 (기존 requests 함수 활용)
    movie_data = get_now_playing_movies()
    movies = movie_data.get('results', [])
    
    # 2. 비동기 환경 설정
    semaphore = asyncio.Semaphore(5) # 한 번에 5개씩만!
    headers = {"Authorization": f"Bearer {API_KEY}"}
    
    async with aiohttp.ClientSession(headers=headers) as session:
        tasks = []
        for movie in movies:
            url = f"[https://api.themoviedb.org/3/movie/](https://api.themoviedb.org/3/movie/){movie['id']}?language=ko-KR"
            # 작업 예약 (Task 생성)
            tasks.append(fetch_with_semaphore(semaphore, session, url))
        
        # 3. 예약된 모든 작업을 동시에 실행
        responses = await asyncio.gather(*tasks)
        
        # 4. 결과 가공 (JSON 문자열 -> 딕셔너리 변환)
        final_list = []
        for res_text in responses:
            detail = json.loads(res_text)
            final_list.append({
                'title': detail.get('title'),
                'vote_average': detail.get('vote_average'),
                'revenue': detail.get('revenue')
            })
        
        pprint(final_list)

# 비동기 실행 시동 걸기
if __name__ == "__main__":
    asyncio.run(main())
```

## 4. 실수/에러와 해결 과정
- **문제 1: Git 충돌 에러**
  - 원인: 로컬 변경사항이 있는 상태에서 `git pull` 시도.
  - 해결: `git reset --hard HEAD`로 로컬 변경사항을 정리하거나 `git checkout`으로 특정 파일 복구 후 성공.
- **문제 2: 비동기 함수 들여쓰기 오류**
  - 원인: `main` 함수가 `fetch_with_semaphore` 함수 내부로 중첩되어 정의됨.
  - 해결: 각 함수의 범위를 독립적으로 분리(왼쪽 벽에 붙임)하여 구조를 바로잡음.

## 5. 실무/RAG 관점의 참견 적용해보기
- **실무 관점:** 실제 대규모 서비스에서는 수백 개의 API를 호출해야 하므로 지금 배운 비동기 로직이 기본 베이스가 됩니다. 세마포어 숫자를 적절히 조절하여 서버 부하와 속도 사이의 균형을 맞추는 것이 핵심입니다.
- **RAG 관점:** RAG 시스템에서 여러 문서를 검색(Retrieval)하거나 대규모 임베딩 작업을 할 때도 이 비동기 패턴을 적용하면 사용자 응답 속도를 크게 향상시킬 수 있습니다.