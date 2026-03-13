# 📅 2026-03-13 (금) - 풀스택 연동: React & FastAPI 통신 구현

## 📌 학습 내용: 클라이언트-서버 모델 이해 및 API 연동
단순히 화면을 만드는 것을 넘어, 실제 백엔드 서버(FastAPI)에서 데이터를 가져오고(`GET`), 저장하고(`POST`), 수정하고(`PATCH`), 삭제하는(`DELETE`) 전체 흐름을 학습했습니다.

## 🚀 주요 구현 포인트
### 1. Axios를 활용한 비동기 통신
- `async/await`를 사용하여 서버의 응답이 올 때까지 기다린 후 화면을 업데이트하는 비동기 처리 로직을 구현했습니다.
- `useEffect` 훅을 사용해 앱이 실행되자마자 서버의 최신 목록을 가져오도록 설계했습니다.

### 2. HTTP Method의 실무적 활용
- **GET**: 서버로부터 할 일 목록 수신 (`axios.get`)
- **POST**: 새로운 할 일 데이터 생성 (`axios.post`)
- **PATCH**: 완료 상태(`done`) 등 일부 데이터만 수정 (`axios.patch`)
- **DELETE**: 특정 데이터 삭제 (`axios.delete`)

### 3. 상태 동기화 (State Synchronization)
- 서버에서 작업(추가, 삭제, 수정)이 성공적으로 완료된 후, 다시 `fetch_todos()`를 호출하여 서버의 최신 데이터와 리액트의 상태를 일치시켰습니다.

## 💻 백엔드-프론트엔드 연결 핵심 코드
````javascript
// 1. 서버에서 데이터 가져오기 (Read)
const fetch_todos = async () => {
  const response = await axios.get("http://localhost:8000/todos");
  set_todos(response.data);
};

// 2. 서버에 데이터 수정 요청 (Update - Toggle)
const handle_toggle = async (todo) => {
  // PATCH를 통해 변경하고 싶은 필드(done)만 전송
  await axios.patch(`http://localhost:8000/todos/${todo.id}`, { 
    done: !todo.done 
  });
  fetch_todos(); // 변경 후 서버 데이터 다시 불러오기
};

🔍 설계 핵심 포인트 (Mentor's Note)
CORS 이슈: FastAPI에서 리액트(Port 3000 등)의 접근을 허용하기 위해 CORSMiddleware 설정이 필수적임을 이해했습니다.

RESTful API: 주소 체계와 Method를 통해 어떤 동작을 할지 명확히 구분하는 설계 방식을 익혔습니다.

🛠️ 트러블슈팅 (Troubleshooting)
문제 상황
리액트에서 데이터를 바꿨는데 새로고침하면 원래대로 돌아오거나, 화면에 취소선이 즉시 반영되지 않는 문제 발생.

해결 과정
단순히 리액트 상태(state)만 바꾸는 것이 아니라, **서버에 업데이트 요청(PATCH)**을 보낸 뒤 다시 목록을 받아오는(fetch) 과정을 통해 데이터의 영속성을 확보함.

서버와 클라이언트 간의 데이터 필드명(예: done, text)이 일치해야 정확한 통신이 이루어짐을 확인함.