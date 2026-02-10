# 🚀 SQLAlchemy 성능 최적화: N+1 문제와 로딩 전략(Loading Strategies)

**작성일:** 2026-02-10

---

## 1. N+1 문제 (The N+1 Query Problem)
데이터베이스 조회 시 발생하는 대표적인 성능 저하 현상입니다. 
- **정의:** 1번의 쿼리로 $N$개의 데이터를 가져왔을 때, 연관된 데이터를 가져오기 위해 $N$번의 추가 쿼리가 발생하는 상황입니다.
- **예시:** 게시글 10개를 조회($1$번)했는데, 각 게시글의 작성자 이름을 알기 위해 유저 테이블을 10번($N$번) 더 조회하여 총 11번의 통신이 발생하는 경우입니다.



---

## 2. 지연 로딩 (Lazy Loading) vs 즉시 로딩 (Eager Loading)

### 🐢 지연 로딩 (Lazy Loading)
- **특징:** SQLAlchemy의 기본 동작 방식입니다. 데이터가 실제로 '참조'되는 시점에 쿼리를 실행합니다.
- **장점:** 당장 필요 없는 데이터를 가져오지 않아 메모리를 아낄 수 있습니다.
- **단점:** 반복문 안에서 연관 객체에 접근하면 N+1 문제를 일으키는 주범이 됩니다.

### ⚡ 즉시 로딩 (Eager Loading)
- **특징:** 메인 데이터를 가져올 때 연관 데이터까지 '한 번에' 가져옵니다.
- **장점:** 데이터베이스 통신 횟수를 획기적으로 줄여 성능을 최적화합니다.

---

## 3. SQLAlchemy 즉시 로딩 전략 비교

| 전략 | 방식 | 권장 관계 | 특징 |
| :--- | :--- | :--- | :--- |
| **`joinedload`** | `JOIN` 사용 | **N:1**, **1:1** | 한 번의 쿼리로 해결. 다대일 관계에 최적. |
| **`selectinload`** | `IN` 절 사용 | **1:N**, **N:M** | 쿼리는 2번 발생하지만, 일대다 관계에서 데이터 중복(Cartesian Product)을 방지함. |

[Image comparing SQL JOIN vs SELECT IN for fetching related entities]

---

## 4. 실무형 복합 로딩 구현 예시

실무에서는 여러 관계가 얽힌 복잡한 객체 그래프를 한 번에 가져와야 합니다.

````python
from sqlalchemy import select
from sqlalchemy.orm import joinedload, selectinload

# [비즈니스 로직] 게시글을 조회하면서 작성자, 작성자 프로필, 댓글, 댓글 작성자까지 한꺼번에 로드
# 변수명 제안: fetch_post_with_all_relations (함수/변수의 목적을 명확히 함)
stmt = (
    select(Post)
    .options(
        # 1. 연쇄 로딩 (Chained): 게시글 -> 작성자(N:1) -> 프로필(1:1)
        joinedload(Post.user).joinedload(User.profile),

        # 2. 다중 로딩 (Multiple): 게시글 -> 댓글들(1:N)
        # 3. 혼합 로딩: 댓글들 -> 각 댓글의 작성자(N:1)
        selectinload(Post.comments).joinedload(Comment.author)
    )
)

# 비동기 환경이나 엄격한 환경에서는 lazy="raise_on_sql" 설정을 통해 
# 실수로 인한 지연 로딩 발생 시 에러를 던지도록 설계하는 것이 좋습니다.

트러블슈팅 (Troubleshooting)
이슈: 1:N 관계에서 joinedload 사용 시 결과 뻥튀기 현상
현상: 게시글 1개에 댓글 10개가 달린 경우, SQL JOIN 결과 게시글 데이터가 10줄로 중복되어 넘어오는 현상 발생.

원인: SQL의 카테시안 곱(Cartesian Product) 특성으로 인해 발생.

해결: 컬렉션 형태(1:N)의 관계에서는 selectinload를 사용하여 메인 쿼리와 연관 쿼리를 분리(총 2번 실행)함으로써 중복 데이터 전송을 막고 성능을 개선함.

