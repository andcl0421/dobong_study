# 📅 2026-03-10 실무 밀착형 멘토링: React 이벤트 핸들링 & State 관리

## 📌 오늘의 학습 목표
* React의 **이벤트 핸들링** 방식 이해 (HTML/JS와의 차이점)
* **Hook(useState)**을 이용한 동적 데이터 관리 및 화면 렌더링 원리 파악
* 컴포넌트 분리 시 발생할 수 있는 **경로 및 대소문자 이슈** 해결 (Troubleshooting)
* **Module CSS**를 활용한 관심사 분리(Style 분리) 실습

---

## 1. React 이벤트 핸들링 (Event Handling)

### ✅ 핵심 규칙
1. **카멜 케이스(CamelCase)** 사용: `onclick`이 아니라 `onClick`, `onchange`가 아니라 `onChange`를 사용합니다.
2. **함수 전달**: `onClick={handleClick()}`처럼 호출하는 게 아니라 `onClick={handleClick}`처럼 **함수 이름(참조)**만 전달합니다.
3. **기본 동작 방지**: `return false` 대신 `event.preventDefault()`를 명시적으로 호출합니다.

### 💡 주요 이벤트 예시
* **onClick**: 버튼 클릭, div 클릭 등 모든 클릭 요소
* **onChange**: 입력창(`input`, `textarea`)의 값이 변할 때 (React에서는 실시간으로 발생)
* **onSubmit**: 폼 제출 시 페이지 새로고침 방지용
* **onKeyDown**: 엔터키 처리 등 키보드 입력 시

---

## 2. React Hook: useState

### 🔑 개념 이해
* **Hook**: 함수형 컴포넌트가 '상태'를 기억하게 해주는 특별한 함수 (반드시 `use`로 시작).
* **State**: 컴포넌트 내부에서 변하는 데이터. 이 값이 변하면 React는 화면을 **리렌더링(다시 그리기)** 합니다.

### 🛠️ 실무 표준 작성법
````javascript
// ❌ 잘못된 방식: 직접 변수 수정 (화면 안 바뀜)
count = count + 1; 

// ✅ 올바른 방식: Setter 함수 사용
setCount(count + 1);

// ✨ 더 안전한 방식: 함수형 업데이트 (이전 값을 보장)
setCount(prev => prev + 1);

3. 실무 실습: 컴포넌트 기반 기능 구현
🐾 실습 1: 디뭉이 성장일기 (좋아요 버튼)
컴포넌트를 LikeButton.jsx로 분리하여 독립적인 상태를 관리합니다.

📝 실습 2: 실시간 글자 수 카운터 (TextCounter)
사용자의 입력을 실시간으로 반영하고, 조건부 스타일링을 적용합니다.

// TextCounter.jsx
import React, { useState } from 'react';
import styles from "./TextCounter.module.css";

const TextCounter = () => {
  const [text, setText] = useState("");

  return (
    <div className={styles.container}>
      <h2>글자 수 세기 <span>📝</span></h2>
      <input 
        value={text} 
        onChange={(e) => setText(e.target.value)} 
        className={styles.inputField}
      />
      <p className={`${styles.countText} ${text.length > 10 ? styles.overLimit : ""}`}>
        현재 글자 수: {text.length}자
      </p>
    </div>
  );
};

🚀 오늘의 트러블슈팅 (Troubleshooting)
1. 컴포넌트 호출 시 숫자가 증가하지 않는 문제
문제: 코드는 정상이나 버튼 클릭 시 UI에 반영되지 않음.

원인:

컴포넌트 이름을 소문자로 시작(state2Base)하여 리액트가 HTML 태그로 오인함.

부모 컴포넌트(App.jsx)에서 엉뚱한 경로의 파일을 import 함.

해결: 컴포넌트 명칭을 **대문자(PascalCase)**로 수정하고, App.jsx에서 정확한 상대 경로를 지정함.

2. CSS Import 경로 에러 (../../index.css)
문제: 하위 폴더에서 최상위 CSS를 불러올 때 경로를 찾지 못함.

원인: 폴더 계층 구조에 대한 이해 부족 (../는 한 단계 위).

해결: ../../index.css와 같이 상위 폴더로 두 번 이동하여 경로를 매칭함.

3. 'removeChild' on 'Node' 런타임 에러
문제: 버튼 클릭 시 화면이 멈추며 DOM 조작 에러 발생.

원인: 브라우저의 자동 번역 기능이 리액트가 관리하는 텍스트 노드를 임의로 변경하여 가상 DOM 동기화가 깨짐.

해결: 브라우저 번역 기능을 끄거나, 텍스트와 이모지를 <span> 등으로 감싸 구조를 견고하게 만듦.

💡 설계 핵심 포인트 (Mentor's Note)
관심사의 분리: 스타일은 .module.css로, 로직은 .jsx로, 배치는 Base 파일로 나누어 관리한다.

컴포넌트 독립성: 각 컴포넌트가 사용할 CSS는 해당 컴포넌트 파일 안에서 직접 import 하는 것이 재사용성에 유리하다.

변수명 준수: 실무 표준인 Snake Case나 CamelCase를 상황에 맞춰 명확히 사용한다. (예: handleLike, like_count 등)