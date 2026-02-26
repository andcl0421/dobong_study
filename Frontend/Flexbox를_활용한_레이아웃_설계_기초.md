
# 📅 작성일: 2026-02-26

## 📝 실습 내용: Flexbox를 활용한 레이아웃 설계 기초
비전공자 신입 개발자로서 웹 화면 배치의 핵심 기술인 **Flexbox**를 집중적으로 실습했습니다. 요소들을 수평, 수직으로 자유롭게 배치하고, 박스 안에 박스를 넣는 중첩 구조를 통해 복합적인 UI 레이아웃을 구성하는 방법을 익혔습니다.

---

## 🏗️ 비즈니스 로직 및 설계 핵심 포인트

### 1. Flexbox 배치의 3대 요소
* **정렬의 시작 (`display: flex`)**: 부모 박스에 선언하여 자식들을 유연하게 움직일 수 있는 상태로 만듭니다.
* **가로 정렬 (`justify-content`)**: `center`(중앙), `between`(양 끝), `flex-end`(오른쪽 정렬) 등을 통해 주축의 흐름을 제어합니다.
* **세로 정렬 (`align-items`)**: `center`를 활용해 높이가 큰 부모 안에서도 자식을 세로 중앙에 딱 맞춥니다.

### 2. 레이아웃 중첩 전략
* `.big_box`라는 큰 틀 안에 여러 개의 `.flex` 컨테이너를 넣어 3x3 격자 구조를 설계했습니다.
* `flex-direction: column`을 사용하여 상하 구조의 레이아웃을 잡는 실무 패턴을 적용했습니다.

---

## 💻 실습 완성 코드 (Refactored)

실무 표준 명명 규칙인 **Snake Case**를 적용하고, 구조를 더 명확하게 개선한 코드입니다.

### [index.html]
````html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flexbox 레이아웃 실습</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="box_item flex justify_center align_center">content</div>

    <hr>

    <div class="flex justify_between">
        <div class="box_item"></div>
        <div class="flex">
            <div class="box_item"></div>
            <div class="box_item"></div>
        </div>
    </div>

    <hr>

    <div class="big_box flex flex_column justify_center align_center">
        <div class="flex">
            <div class="box_item"></div>
            <div class="box_item"></div>
            <div class="box_item"></div>
        </div>
        <div class="flex">
            <div class="box_item"></div>
            <div class="box_item"></div>
            <div class="box_item"></div>
        </div>
        <div class="flex">
            <div class="box_item"></div>
            <div class="box_item"></div>
            <div class="box_item"></div>
        </div>
    </div>
</body>
</html>

/* 공통 박스 스타일 */
.box_item {
    height: 100px;
    width: 100px;
    border: 1px solid #333;
    padding: 20px;
    margin: 10px;
}

/* 대형 컨테이너 */
.big_box {
    height: 600px;
    width: 600px;
    border: 5px solid black;
    margin: 100px auto 0;
}

/* 유틸리티 클래스: 실무에서 재사용성을 높이는 방식 */
.flex { 
    display: flex; 
}

.flex_column { 
    flex-direction: column; 
}

.justify_center { 
    justify-content: center; 
}

.justify_between { 
    justify-content: space-between; 
}

.align_center { 
    align-items: center; 
}

/* 간격 조절 */
hr { 
    margin: 50px 0; 
}

.space { 
    height: 5vh; 
}

⚠️ Troubleshooting (오늘의 트러블슈팅)
1. Position 속성과 Absolute의 이해
문제: position: absolute를 사용했을 때 자식 상자가 화면에서 사라지거나 위치가 엉뚱한 곳으로 잡힘.

원인: 부모 상자에 position: relative 기준점이 없었고, 자식 상자에 배경색이 없어 투명하게 보였음.

해결: 부모에게 relative를 부여하여 기준을 잡고, 자식에게 background-color와 top/left 값을 주어 위치를 시각적으로 확인함.

2. 클래스 명명 불일치
문제: .just-center라는 이름에 flex-end가 설정되어 있어 코드의 가독성이 떨어짐.

해결: 클래스명은 스타일의 목적을 정직하게 반영해야 하므로 .justify_end 등으로 수정 제안함.