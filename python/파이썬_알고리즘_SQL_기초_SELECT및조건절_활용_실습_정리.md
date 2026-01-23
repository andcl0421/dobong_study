#### [2026-01-23] SQL 기초 문제 풀이 및 데이터베이스 핵심 정리

---

### 1. 오늘 학습 요약 (핵심 정리)
- SQL의 기본 구조인 `SELECT`, `FROM`, `WHERE` 절의 역할을 이해하고 실습함
- 비교 연산자(`>`, `=`), 논리 연산자(`AND`, `OR`, `NOT`), 범위(`BETWEEN`), 포함(`IN`), 패턴 매칭(`LIKE`), NULL 처리(`IS NULL`) 등 다양한 필터링 기법을 익힘
- 데이터베이스 설계의 기초인 키(PK, FK)의 개념과 관계(1:N, M:N)의 차이를 학습함

---

### 2. 배운 개념 정리
- **기본 키(Primary Key)**: 테이블의 각 행을 유일하게 식별하는 값으로, 중복되거나 NULL일 수 없음
- **외래 키(Foreign Key)**: 다른 테이블의 기본 키를 참조하여 테이블 간의 관계를 정의함
- **M:N 관계**: 두 테이블이 서로 여러 행과 관련 있는 경우로, 실무에서는 '중계 테이블'을 만들어 해결함
- **데이터 무결성**: 데이터의 정확성과 일관성을 유지하기 위한 규칙(개체, 참조, 도메인 무결성)

---

### 3. 오늘 작성한 코드 리뷰 (SQL 실습)

```sql
-- 1. 인구가 800만 이상인 도시 조회
SELECT c.name, c.population 
FROM city c 
WHERE c.population >= 8000000;

-- 2. 한국(KOR) 도시 조회
SELECT c.name, c.countrycode 
FROM city c 
WHERE c.countrycode = 'KOR';

-- 3. 유럽(Europe) 대륙 나라 조회
SELECT n.name, n.region 
FROM country n 
WHERE n.continent = 'Europe';

-- 4. 'San'으로 시작하는 도시 조회 (패턴 매칭)
SELECT c.name 
FROM city c 
WHERE c.name LIKE 'San%';

-- 5. 1900년 이후 독립한 나라 조회
SELECT n.name, n.indepyear 
FROM country n 
WHERE n.indepyear > 1900;

-- 6. 인구 100만~200만 사이 한국 도시 조회
SELECT c.name 
FROM city c 
WHERE c.countrycode = 'KOR' 
  AND c.population BETWEEN 1000000 AND 2000000;

-- 7. 한/일/중 인구 500만 이상 도시 조회
SELECT c.name, c.countrycode, c.population 
FROM city c 
WHERE c.countrycode IN ('KOR', 'JPN', 'CHN') 
  AND c.population >= 5000000;

-- 8. 'A'로 시작하고 'a'로 끝나는 도시 조회
SELECT c.name 
FROM city c 
WHERE c.name LIKE 'A%a';

-- 9. 동남아시아를 제외한 아시아 나라 조회
SELECT n.name, n.region 
FROM country n 
WHERE n.continent = 'Asia' 
  AND n.region != 'Southeast Asia';

-- 10. 오세아니아 대륙 중 기대수명 데이터가 없는(NULL) 나라 조회
SELECT n.name, n.lifeexpectancy, n.continent 
FROM country n 
WHERE n.continent = 'Oceania' 
  AND n.lifeexpectancy IS NULL;
```

---

### 4. 실수/에러와 해결 과정
- **문제**: `WHERE` 절 조건 뒤에 불필요한 컬럼명(`c.name`)을 덧붙여 문법 에러 발생
- **원인**: `WHERE`는 필터링 조건만 적는 곳인데 조회 결과에 나올 컬럼을 중복해서 적음
- **해결**: `SELECT`에는 '보여줄 것', `WHERE`에는 '찾을 기준'만 명확히 구분하여 작성함

---

### 5. 실무/RAG 관점의 참견
- **RAG 구현 시**: 사용자가 자연어로 "한국의 대도시 알려줘"라고 물으면, 시스템은 `population`과 `countrycode` 컬럼을 매핑하여 SQL을 생성해야 하므로 정확한 컬럼명 이해가 필수적임
- **데이터 정제**: `IS NULL`을 활용해 결측치를 찾아내는 과정은 데이터 분석 및 AI 학습 데이터 전처리에서 매우 중요한 단계임