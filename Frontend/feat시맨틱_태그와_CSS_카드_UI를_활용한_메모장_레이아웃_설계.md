# 📅 작성일: 2026-02-25

## 1. 비즈니스 로직 및 설계 핵심 포인트
이 프로젝트는 사용자가 일상의 기록을 체계적으로 관리할 수 있는 **'카드형 메모장 UI'**를 구현하는 데 초점을 맞췄습니다.

* **구조적 설계 (Semantic Markup):** `header`, `main`, `section` 등 의미 있는 태그를 사용하여 데이터의 위계를 명확히 했습니다. 이는 웹 접근성을 높이는 실무의 기본 소양입니다.
* **시각적 요소 (Visual Hierarchy):** 제목(`h2`) 하단에 라인을 추가하고 날짜를 우측 정렬(`text-align: right`)하여 정보의 중요도를 시각적으로 구분했습니다.
* **중앙 집중식 레이아웃:** `margin: 20px auto`를 활용해 콘텐츠를 화면 중앙에 배치함으로써 사용자의 시선이 메모에 집중될 수 있도록 설계했습니다.



---

## 2. 실무 밀착형 코드 예시

### 📝 index.html
```html
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>나만의 메모장</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <header class="header_section">
      <h1>나만의 메모장</h1>
    </header>

    <main>
      <section class="memo_card">
        <h2 class="memo_title">장보기 목록</h2>
        <div class="memo_content">계란, 닭가슴살, 대파, 양파</div>
        <div class="memo_date">2026-02-25</div>
      </section>

      <section class="memo_card">
        <h2 class="memo_title">주말 계획</h2>
        <div class="memo_content">토요일 오전 휴식, 일요일 오전 러닝</div>
        <div class="memo_date">2026-02-25</div>
      </section>
    </main>

    <footer class="footer_info">간단한 메모 웹 페이지</footer>
  </body>
</html>
```

### 🎨 style.css
```css
/* 전체 초기화 및 박스 모델 설정 */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box; /* 패딩이 넓이를 침범하지 않게 방어 */
}

body {
    background-color: aliceblue;
    font-family: 'Pretendard', sans-serif; /* 실무에서 자주 쓰는 폰트 예시 */
}

.header_section {
    background-color: orange;
    text-align: center;
    padding: 20px;
    color: white;
}

.memo_card {
    width: 600px;
    margin: 20px auto; /* 상하 여백 20px, 좌우 자동(가운데 정렬) */
    padding: 20px;
    border-radius: 10px;
    background-color: white;
    min-height: 150px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1); /* 카드 느낌을 주는 그림자 */
}

.memo_title {
    border-bottom: 2px solid #f0f0f0;
    padding-bottom: 8px;
    margin-bottom: 12px;
    font-weight: 700; /* 제목 강조 */
}

.memo_date {
    text-align: right;
    color: #888888;
    font-size: 14px;
    margin-top: 20px;
}

footer {
    margin-top: 100px;
    text-align: center;
    padding-bottom: 30px;
    color: #999999;
}
```

---

## 3. 코드 리뷰 및 네이밍 제언
* **Naming Convention:** 클래스명을 지을 때 하이픈(`-`)보다는 실무 표준인 **Snake Case(`_`)**를 사용하여 `.memo_card`, `.memo_title`과 같이 작성하는 것을 추천합니다. 이는 데이터베이스 컬럼명과 통일감을 주어 협업 시 유리합니다.
* **유지보수성:** `* { box-sizing: border-box; }` 설정을 통해 추후 패딩 값이 변하더라도 레이아웃 전체 폭이 틀어지지 않도록 방어 코드를 작성했습니다.

---

## 4. 🛠️ 트러블슈팅 (Troubleshooting)

**에러 상황:**
메모 섹션 사이에 간격을 조절하려고 `margin`을 넣었으나, 예상과 다르게 레이아웃이 겹치거나 여백이 부족해 보이는 현상.

**원인 분석:**
`margin-top`과 `margin-bottom`이 만날 때 발생하는 '마진 상쇄(Margin Collapsing)' 현상 혹은 `box-sizing` 미설정으로 인한 크기 계산 오류입니다. (중학생 설명: 상자 겉에 투명한 보호막이 서로 겹쳐서 여백이 하나처럼 보이는 현상이에요!)

**해결 과정:**
1.  **조치 사항:** `box-sizing: border-box`를 최상단에 선언하여 크기 계산을 고정했습니다.
2.  **조치 사항:** `margin: 20px auto`로 상하 여백을 통일하여 요소 간의 간격을 일정하게 확보했습니다.

---

