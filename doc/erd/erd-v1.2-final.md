```sql
-- =================================================================
--  1. 데이터베이스 생성 및 선택
-- =================================================================
CREATE DATABASE IF NOT EXISTS bookjuk DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE bookjuk;

-- =================================================================
--  독서모임 서비스 DDL (FK, UNIQUE 제거 버전)
-- =================================================================

-- -----------------------------------------------------
-- Table `user`
-- -----------------------------------------------------
CREATE TABLE `user` (
    `user_id`        BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '유저 고유 식별자',
    `nickname`       VARCHAR(50) NOT NULL COMMENT '닉네임 (중복 허용)',
    `email`          VARCHAR(100) NOT NULL COMMENT '이메일 (로그인 ID)',
    `password`       VARCHAR(255) NOT NULL COMMENT '비밀번호 (해시화하여 저장)',
    `preferred_genre` VARCHAR(100) NULL COMMENT '선호 장르',
    `profile_image`  VARCHAR(255) NULL DEFAULT 'default_profile.jpg' COMMENT '프로필 이미지 URL',
    `introduction`   TEXT NULL COMMENT '자기소개',
    `created_at`     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '회원 가입 시점',
    `updated_at`     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '회원 정보 수정 시점'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='사용자 정보';

-- -----------------------------------------------------
-- Table `meeting`
-- -----------------------------------------------------
CREATE TABLE `meeting` (
    `meeting_id`       BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '모임 고유 식별자',
    `host_id`          BIGINT NOT NULL COMMENT '방장(주최자)의 user_id',
    `title`            VARCHAR(255) NOT NULL COMMENT '모임 제목',
    `description`      TEXT NULL COMMENT '모임 상세 설명',
    `image_url`        VARCHAR(255) NULL COMMENT '모임 대표 이미지 URL',
    `book_title`       VARCHAR(255) NOT NULL COMMENT '선정 도서 제목',
    `book_author`      VARCHAR(100) NOT NULL COMMENT '선정 도서 저자',
    `genre`            VARCHAR(100) NOT NULL COMMENT '모임 장르',
    `meeting_time`     DATETIME NOT NULL COMMENT '모임 시간',
    `region`           VARCHAR(20) NOT NULL COMMENT '시/도',
    `city`             VARCHAR(30) NOT NULL COMMENT '시/군',
    `district`         VARCHAR(30) NOT NULL COMMENT '구/군',
    `detail_address`   VARCHAR(255) NULL COMMENT '상세주소(선택)',
    `max_participants` INT NOT NULL COMMENT '최대 참여 인원',
    `status`           VARCHAR(20) NOT NULL DEFAULT 'RECRUITING' COMMENT '모임 상태',
    `created_at`       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '모임 생성 시점',
    `updated_at`       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '모임 정보 수정 시점'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='독서 모임';

-- -----------------------------------------------------
-- Table `meeting_participant`
-- -----------------------------------------------------
CREATE TABLE `meeting_participant` (
    `id`         BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '참여 고유 식별자',
    `meeting_id` BIGINT NOT NULL COMMENT '참여 모임의 id',
    `user_id`    BIGINT NOT NULL COMMENT '참여 사용자의 user_id',
    `role`       VARCHAR(20) NOT NULL COMMENT 'HOST, PARTICIPANT',
    `status`     VARCHAR(20) NOT NULL DEFAULT 'PENDING' COMMENT '참여 상태',
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '신청 시점',
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '상태 변경 시점'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='모임 참여자';

-- -----------------------------------------------------
-- Table `meeting_review`
-- -----------------------------------------------------
CREATE TABLE `meeting_review` (
    `id`           BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '후기(좋아요) 고유 식별자',
    `meeting_id`   BIGINT NOT NULL COMMENT '관련 모임의 id',
    `reviewer_id`  BIGINT NOT NULL COMMENT '리뷰 작성자',
    `reviewee_id`  BIGINT NOT NULL COMMENT '리뷰 받은 사용자',
    `created_at`   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '리뷰 작성 시간'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='모임 후기';

-- -----------------------------------------------------
-- Table `post`
-- -----------------------------------------------------
CREATE TABLE `post` (
    `post_id`         BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '게시글 고유 식별자',
    `meeting_id`      BIGINT NOT NULL COMMENT '소속된 모임의 id',
    `user_id`         BIGINT NOT NULL COMMENT '작성자의 user_id',
    `title`           VARCHAR(200) NOT NULL COMMENT '게시글 제목',
    `content`         TEXT NOT NULL COMMENT '게시글 내용',
    `image_url`       VARCHAR(255) NULL COMMENT '이미지 URL',
    `created_at`      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '작성일',
    `updated_at`      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '수정일'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='모임별 게시판의 게시글';

-- -----------------------------------------------------
-- Table `comment`
-- -----------------------------------------------------
CREATE TABLE `comment` (
    `id`         BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '댓글 고유 식별자',
    `post_id`    BIGINT NOT NULL COMMENT '원본 게시글의 id',
    `user_id`    BIGINT NOT NULL COMMENT '작성자의 user_id',
    `content`    TEXT NOT NULL COMMENT '댓글 내용',
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '작성일',
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '수정일'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='게시글의 댓글';

-- 접근 권한 확인, 중복 방지용
CREATE INDEX idx_user_email                 ON `user`(email);
CREATE INDEX idx_meeting_host               ON `meeting`(host_id);
CREATE INDEX idx_mp_meeting_user_status     ON `meeting_participant`(meeting_id, user_id, status);
CREATE INDEX idx_post_meeting_created       ON `post`(meeting_id, created_at);
CREATE INDEX idx_comment_post_created       ON `comment`(post_id, created_at);
```
