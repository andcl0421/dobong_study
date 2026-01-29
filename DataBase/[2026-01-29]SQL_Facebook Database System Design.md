#  페이스북(Facebook) 서비스 데이터베이스 설계

본 프로젝트는 SNS의 핵심 기능을 데이터 관점에서 분석하고, 효율적인 데이터 저장 및 조회를 위한 
관계형 데이터베이스(RDBMS) 설계 실습 결과물입니다.

##  프로젝트 개요
- **목표**: 대규모 사용자의 상호작용이 발생하는 SNS 환경에서의 데이터 무결성 및 성능 최적화 설계
- **핵심 주제**: 사용자 관리, 팔로우 시스템, 게시물 및 해시태그 검색 최적화

---

##  데이터베이스 모델링 상세 설명

### 1. 사용자 및 팔로우 (User & Friendship)
- **자기 참조(Self-referencing) 설계**: `friendship` 테이블은 동일한 `user` 테이블을 `follower_id`와 `following_id`로 각각 참조합니다.
- **데이터 무결성**: 모든 팔로우 관계는 실제 가입된 사용자 사이에서만 성립되도록 외래키(FK)를 설정했습니다.

### 2. 게시물 및 콘텐츠 (Post & Comment)
- **1:N 관계**: 한 명의 사용자가 여러 게시물과 댓글을 작성할 수 있도록 1대다 관계로 설계되었습니다.
- **추적 기능**: `created_at` 필드를 통해 게시물과 댓글의 생성 시간을 정확히 기록합니다.

### 3. 해시태그 검색 최적화 (Hashtag Optimization)
- **다대다(M:N) 해소**: 게시물과 태그 사이의 복잡한 관계를 `post_hashtag`라는 중간 테이블로 연결했습니다.
- **성능 이점**: 게시물 본문을 일일이 읽지 않고도 특정 태그가 달린 글을 빠르게 조회(Index 효과)할 수 있습니다.

##  ERD (Entity Relationship Diagram)
- 최종 설계도: (../images/facebook_erd.png) 참조

## 💻 SQL 구현 코드 (DDL)

```sql
/* 2026-01-29 최종 설계 반영 */

-- 사용자 정보 테이블
CREATE TABLE user (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    nickname VARCHAR(50) NOT NULL,
    birth_date DATE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 팔로우 관계 테이블 (자기 참조)
CREATE TABLE friendship (
    friendship_id INT PRIMARY KEY AUTO_INCREMENT,
    follower_id INT NOT NULL,
    following_id INT NOT NULL,
    status VARCHAR(20) DEFAULT 'active',
    FOREIGN KEY (follower_id) REFERENCES user(user_id) ON DELETE CASCADE,
    FOREIGN KEY (following_id) REFERENCES user(user_id) ON DELETE CASCADE
);

-- 게시물 테이블
CREATE TABLE post (
    post_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    image_url VARCHAR(255),
    content TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES user(user_id) ON DELETE CASCADE
);

-- 댓글 테이블
CREATE TABLE comment (
    comment_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    post_id INT NOT NULL,
    content TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES user(user_id) ON DELETE CASCADE,
    FOREIGN KEY (post_id) REFERENCES post(post_id) ON DELETE CASCADE
);

-- 해시태그 마스터 테이블
CREATE TABLE hashtag (
    hashtag_id INT PRIMARY KEY AUTO_INCREMENT,
    tag_name VARCHAR(50) NOT NULL UNIQUE
);

-- 게시물-태그 연결 테이블 (M:N 해소)
CREATE TABLE post_hashtag (
    post_id INT NOT NULL,
    hashtag_id INT NOT NULL,
    PRIMARY KEY (post_id, hashtag_id),
    FOREIGN KEY (post_id) REFERENCES post(post_id) ON DELETE CASCADE,
    FOREIGN KEY (hashtag_id) REFERENCES hashtag(hashtag_id) ON DELETE CASCADE
);
```

## 🎓 설계 리뷰 및 배운 점
- 실무 수준의 **Naming Convention**(`snake_case`)을 적용하여 협업 가능성을 높였습니다.
- `ON DELETE CASCADE` 옵션을 통해 데이터 삭제 시 연관 데이터가 함께 정리되는 자동화 로직을 반영했습니다.
- 복잡한 SNS 관계를 시각화하고 물리적인 테이블로 변환하는 과정을 통해 **RDBMS의 구조적 이해**를 높였습니다.