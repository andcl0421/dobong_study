

## 🚀 실전 Flexbox 레이아웃 실습
비전공자 신입 개발자가 실무에서 가장 많이 접하는 **'랜딩 페이지'**의 핵심 섹션들을 Flexbox만으로 구현한 프로젝트입니다. 

---

## 💡 섹션별 설계 핵심 포인트 (Business Logic)

### 1. 상단 헤더 (Header)
* **목적**: 브랜드 로고와 내비게이션 메뉴를 사용자 눈에 가장 잘 띄게 배치합니다.
* **구현**: `justify-content: space-between`을 사용하여 로고는 왼쪽, 메뉴는 오른쪽에 고정했습니다.

### 2. 히어로 섹션 (Hero Section)
* **목적**: 서비스의 첫인상을 결정하는 핵심 메시지를 화면 정중앙에 배치합니다.
* **구현**: `flex-direction: column`으로 축을 세로로 돌린 뒤, `justify-content: center`와 `align-items: center`를 조합해 가로/세로 완전 중앙 정렬을 구현했습니다.

### 3. 서비스 소개 및 요금제 (Cards)
* **목적**: 여러 정보를 카드 형태로 나열하여 가독성을 높입니다.
* **구현**: 
    * `display: flex`와 `gap: 20px`를 활용해 일정한 간격을 유지했습니다.
    * 요금제 버튼에 `margin-top: auto`를 사용하여, 카드 높이가 달라도 버튼 위치가 하단에 일치하도록 정렬했습니다.

---

## ⚠️ 트러블슈팅 (Troubleshooting)

### 이슈: 카드 간격이 깨지는 현상
* **원인**: 각 카드의 `margin`을 수동으로 계산하다 보니 브라우저 크기에 따라 배치가 어긋나는 문제 발생.
* **해결**: 개별 마진 대신 부모 컨테이너에 `gap` 속성을 적용하여 카드 사이의 간격을 시스템적으로 제어했습니다.
* **학습**: `gap`을 사용하면 첫 번째와 마지막 요소의 여백을 따로 계산할 필요가 없어 실무에서 훨씬 효율적입니다.

---

## 📝 코드 리뷰 및 Naming 제안

비전공자 신입 개발자분들이 실무에서 더 높은 평가를 받을 수 있도록 변수명을 개선했습니다.

| 기존 이름 | 개선된 이름 (제안) | 이유 |
| :--- | :--- | :--- |
| `.introduction-cards` | `.service-list` | 콘텐츠의 성격을 더 직관적으로 나타냄 |
| `.focus` | `.card--highlighted` | 단순히 스타일을 넘어 '강조됨'이라는 상태를 의미함 |
| `.profile-content` | `.profile-info` | 텍스트 정보임을 명확히 함 |

---

## 💻 실습 코드 예시 (Refactored)

````html
<div class="pricing-cards">
  <div class="pricing-card">
    <h3 class="pricing-card__title">Basic</h3>
    <div class="pricing-card__price">무료</div>
    <ul class="pricing-card__features">
      <li>기본 기능</li>
      <li>5GB 저장공간</li>
    </ul>
    <button class="btn btn--outline">선택하기</button>
  </div>
</div>

