# 🚀 실무 밀착형 프로젝트: React useEffect와 Axios를 활용한 영화 API 연동

### 📅 작성일: 2026-03-12

---

## 1. 개요
`TMDB` API를 활용하여 현재 상영 중인 영화 데이터를 비동기로 호출하고, 데이터 수신 전후의 로딩 상태(Loading State)를 효율적으로 관리하는 실습을 진행했습니다.

## 2. 핵심 설계 포인트
* **비동기 데이터 페칭**: `Axios`와 `async/await`를 사용하여 영화 데이터를 안정적으로 가져옴.
* **로딩 상태 관리**: `loading` 상태값(`true/false`)을 활용하여 데이터 수신 중 사용자에게 시각적 피드백 제공.
* **안정적인 예외 처리**: `try-catch-finally` 구문을 도입하여 에러 발생 시에도 로딩 상태가 반드시 종료되도록 설계함.

## 3. 트러블슈팅 (Troubleshooting)
### ❌ 문제 상황: [plugin:vite:import-analysis] 에러 및 SyntaxError
* **원인 1**: `App.jsx` 임포트 경로 끝에 보이지 않는 공백(`"Base "`)이 포함되어 파일 참조 실패.
* **원인 2**: `setloading(ture)`와 같이 `true` 예약어에 오타가 발생하여 로직 중단.
* **원인 3**: 파일 내부에 `export default` 선언이 누락되어 모듈 해석 오류 발생.

### ✅ 해결 과정
* **경로 수정**: 임포트 구문의 후행 공백 제거 및 파일명 정합성 확인.
* **구문 교정**: 오타(`ture` -> `true`) 수정 및 `finally` 블록을 사용하여 로딩 종료 보장.
* **모듈 최적화**: 파일 하단에 `export default`를 명시하여 외부 참조 문제 해결.

## 4. 실무 표준 코드 예시
```javascript
const fetchMovies = async () => {
  setLoading(true); // 로딩 시작
  
  // UX 확인을 위한 2초 가상 지연
  await new Promise((resolve) => setTimeout(resolve, 2000));

  try {
    const response = await axios.get("[https://api.themoviedb.org/3/movie/now_playing](https://api.themoviedb.org/3/movie/now_playing)", { 
      params: { 
        api_key: "YOUR_API_KEY",
        language: "ko-KR"
      } 
    });
    setMovies(response.data.results);
  } catch (error) {
    console.error("데이터 호출 에러:", error);
  } finally {
    setLoading(false); // 성공/실패 여부와 상관없이 로딩 종료 (뒷정리)
  }
};
```

---

## 💡 멘토의 한마디
실무에서는 단순히 기능을 구현하는 것보다, **"에러가 났을 때 사용자가 벽을 느끼지 않게 하는 것(로딩 해제)"**이 더 중요합니다. `finally` 구문은 바로 그런 '센스 있는 개발자'가 되기 위한 첫걸음입니다!