2026-02-24

# 📚 HTML5 시맨틱 구조화 및 실무 폼 인터페이스 실습 정리

웹 표준을 준수하는 **시맨틱 마크업(Semantic Markup)**으로 페이지의 뼈대를 설계하고, 실제 비즈니스 로직이 포함된 **데이터 입력 시스템(Form Interface)**을 구현한 실습 기록입니다.

---

## 🛠️ 실무 밀착형 학습 포인트 및 로직

### 1. 시맨틱 레이아웃 설계 (Structure)
* **영역의 명확화**: `<div>`로만 구성하던 방식에서 벗어나 `<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`를 사용하여 브라우저와 검색 엔진이 페이지 구조를 한눈에 파악하도록 설계했습니다.
* **계층적 콘텐츠**: 공지사항이나 게시글은 `<article>`로, 연관된 주제는 `<section>`으로 구분하여 정보의 우선순위를 명확히 했습니다.

### 2. 비즈니스 폼 설계 (UI/UX)
* **메모장 시스템**: 제목(`text`)과 내용(`textarea`)을 입력받고 `<button>`을 통해 전송하는 기본 입력 구조를 실습했습니다.
* **숙소 예약 로직**: 
    * `date` 타입을 사용하여 체크인/체크아웃 날짜를 선택받습니다.
    * `number` 타입의 `min` 속성을 사용하여 성인 인원이 최소 1명 이상 입력되도록 제어했습니다.
    * `radio` 버튼을 통해 여러 객실 타입 중 단 하나만 선택할 수 있는 배타적 선택 로직을 구현했습니다.

### 3. 멀티미디어 및 인터랙션
* **미디어 제어**: `<audio>`와 `<video>` 태그에 `controls`, `autoplay`, `loop` 속성을 부여하여 사용자 친화적인 미디어 재생 환경을 구축했습니다.
* **링크 활용**: 외부 링크는 `target="_blank"`를 사용하여 새 창에서 열리도록 하고, 내부 앵커(`id` 속성)를 통해 페이지 내 특정 위치로 즉시 이동하는 기능을 구현했습니다.

---

## ⚠️ 트러블슈팅 (Troubleshooting)

### 1. 라디오 버튼 중복 선택 및 그룹화 오류
* **현상**: 객실 타입을 선택할 때 여러 개가 동시에 체크되는 문제 발생.
* **원인**: 각 라디오 버튼의 `name` 속성이 서로 다르게 설정되어 별개의 항목으로 인식됨.
* **해결**: 관련 있는 라디오 버튼들의 `name` 값을 `room_category` 등으로 통일하여 하나의 그룹으로 묶어 단일 선택이 가능하게 수정함.

### 2. 리소스 로드 실패 (Broken Image)
* **현상**: 이미지 태그(`<img>`)를 사용했으나 화면에 이미지가 보이지 않고 대체 텍스트(`alt`)만 출력됨.
* **원인**: 로컬 이미지 경로(`images.webp`) 설정 오류 또는 파일 부재.
* **해결**: 상대 경로의 개념을 이해하고 파일 위치를 재설정함. 또한, 이미지 로드 실패 시에도 사용자가 정보를 알 수 있도록 상세한 `alt` 속성을 작성함.

---

## 💻 실무형 예시 코드 (Refactored)

실무 표준인 **Snake Case** 명명 규칙을 적용하고, 비즈니스 로직을 강화한 최종 완성 코드입니다.

```html
<section id="reservation_form_section">
  <h3>숙소 예약하기</h3>
  <form action="/submit_reservation" method="POST">
    <div>
      <label for="check_in_date">체크인 : </label>
      <input type="date" name="check_in_date" id="check_in_date" required />
    </div>

    <div>
      <label for="adult_occupancy">성인 : </label>
      <input type="number" name="adult_occupancy" id="adult_occupancy" min="1" value="1" />
    </div>

    <fieldset>
      <legend>선호 객실 타입</legend>
      <label>
        <input type="radio" name="room_category" value="standard" checked /> 스탠다드
      </label>
      <label>
        <input type="radio" name="room_category" value="deluxe" /> 디럭스
      </label>
      <label>
        <input type="radio" name="room_category" value="suite" /> 스위트
      </label>
    </fieldset>

    <div>
      <label for="special_request_memo">요청사항 : </label>
      <textarea
        name="special_request_memo"
        id="special_request_memo"
        placeholder="특별히 요청하고 싶은 내용을 적어주세요."
      ></textarea>
    </div>

    <button type="submit">예약하기</button>
  </form>
</section>
```

---

## 📌 Commit Message

### **feat: 시맨틱 레이아웃 설계 및 숙소 예약 비즈니스 폼 구현**

**상세 내용:**
- `<header>`, `<main>`, `<aside>` 등 시맨틱 태그를 통한 웹 표준 구조화
- `input` 타입(date, number, radio) 및 `textarea`를 활용한 데이터 수집 폼 구축
- 라디오 버튼의 `name` 그룹화 및 `label` 연결을 통한 UX 및 접근성 개선
- `audio`, `video` 멀티미디어 태그 활용 및 미디어 제어 옵션 적용