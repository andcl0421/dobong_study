# 📅 2026-03-11 (수) 실습 통합 정리

## 📂 프로젝트: React-Advanced-State-Logic
> **핵심 요약**: State 관리의 기초부터 심화(이벤트 애니메이션, 로컬 스토리지 저장, 컴포넌트 설계)까지 단계별 구현 실습


## 💡 실습 단계별 핵심 포인트

### 1. 이모지 우수수 효과 (Loop & Animation)
단순한 하트 표시를 넘어, `for` 문으로 여러 객체를 생성하고 `Math.random()`으로 위치와 시간을 다르게 주어 풍성한 시각 효과를 연출했습니다.

### 2. 브라우저 창고 이용하기 (JSON 변환)
객체 배열인 `foods`를 문자열로 포장(`stringify`)해서 저장하고, 다시 꺼낼 때 데이터를 푸는(`parse`) 과정을 통해 새로고침해도 데이터가 유지되는 기능을 구현했습니다.

### 3. 컴포넌트 조립 (Component Composition)
하나의 거대한 파일을 만드는 대신, `Container`, `Input`, `Display`로 역할을 나누어 관리하는 실무 지향적 설계를 적용했습니다.

---

## 💻 주요 구현 코드 (Key Highlights)

### [이모지 쏟아지는 로직]
````javascript
const showerEmoji = (type) => {
  const newItems = [];
  for (let i = 0; i < 20; i++) {
    newItems.push({
      id: Math.random(),
      char: type === "like" ? "❤️" : "👎",
      left: Math.random() * 100 + "vw",
      delay: Math.random() * 1 + "s", // 우수수 느낌의 핵심!
    });
  }
  setEmojis((prev) => [...prev, ...newItems]);
};

[데이터 기억하기 (LocalStorage)]
JavaScript
// 저장: 데이터가 바뀔 때마다 실행
useEffect(() => {
  localStorage.setItem("my-list", JSON.stringify(data));
}, [data]);
🚀 Troubleshooting (오늘의 해결 과정)
❓ 문제 현상
이모지가 하나만 떨어지거나, 새로고침 시 공들여 추가한 음식 목록이 자꾸 초기화됨.

부모-자식 컴포넌트 분리 시 데이터가 전달되지 않아 화면이 멈춤.

✅ 해결 과정
반복문 도입: for 문을 사용해 한 번에 여러 이모지 객체를 생성하도록 수정.

포장/해체 작업: 로컬 스토리지는 문자열만 받으므로 JSON 함수를 사용해 데이터 형태를 변환하여 해결.

Props 연결: 부모의 함수를 자식에게 props로 정확히 전달하여 데이터 흐름 연결.

# 📅 실습 통합 정리

## 📂 실습 내용: React 심화 및 컴포넌트 설계
- **Food List**: 데이터 불변성(`[...]`)을 활용한 리스트 관리
- **Emoji Shower**: `for`문과 `animation`을 조합한 역동적 UI 구현
- **Persistence**: `localStorage`를 이용해 새로고침해도 데이터 유지하기
- **Architecture**: `MessageContainer` 부모에서 자식으로 데이터 전달(Props) 및 상태 끌어올리기
- **Styling**: `Tailwind CSS`를 이용한 실무형 디자인 적용

---

## 🚀 오늘의 트러블슈팅 (Troubleshooting)
- **문제**: 터미널에서 커밋 메시지에 소괄호`()`나 특수기호를 넣었을 때 `syntax error` 발생.
- **원인**: Bash 셸에서 특수 문자를 명령어로 오인하여 발생한 문제.
- **해결**: 복잡한 커밋 메시지는 VS Code의 GUI 툴을 사용하거나, 터미널에서는 특수문자를 지우고 큰따옴표`""`로 감싸서 해결함.