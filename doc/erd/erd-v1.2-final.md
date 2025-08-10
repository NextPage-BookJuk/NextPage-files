```sql
-- =================================================================
--  1. 데이터베이스 생성 및 선택
-- =================================================================
CREATE DATABASE IF NOT EXISTS `book_juk`
    DEFAULT CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;
USE `book_juk`;

-- =================================================================
--  독서모임 서비스 DDL (리팩토링 ver. / FK, 인덱스/제약조건 보강)
-- =================================================================
-- 주의:
-- 테이블 간의 데이터 정합성은 애플리케이션 코드 레벨에서 반드시 검증해야 합니다.
-- (예: INSERT 전 meeting_id, user_id 등의 존재 여부를 코드로 직접 확인)
-- 데이터 양이 많아질 경우, 인덱스가 부족하면 검색 성능이 저하될 수 있습니다.
-- MariaDB/MySQL 버전에 따라 CHECK 제약의 동작이 다를 수 있어, 값 제한은 ENUM 권장.
-- =================================================================

-- -----------------------------------------------------
-- Table `user`  (사용자 정보)
-- -----------------------------------------------------
CREATE TABLE `user` (
                        `user_id`         BIGINT NOT NULL AUTO_INCREMENT COMMENT '유저 고유 식별자',
                        `nickname`        VARCHAR(50) NOT NULL COMMENT '닉네임 (중복 허용)',
                        `email`           VARCHAR(100) NOT NULL UNIQUE COMMENT '이메일 (로그인 ID)',
                        `password`        VARCHAR(255) NOT NULL COMMENT '비밀번호 (해시화하여 저장)',
                        `preferred_genre` VARCHAR(100) NULL COMMENT '선호 장르',
                        `profile_image`   VARCHAR(255) NULL DEFAULT 'default_profile.jpg' COMMENT '프로필 이미지 URL',
                        `introduction`    TEXT NULL COMMENT '자기소개',
                        `created_at`      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '회원 가입 시점',
                        `updated_at`      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '회원 정보 수정 시점',
                        PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='사용자 정보';

-- -----------------------------------------------------
-- Table `meeting`  (독서 모임)
-- -----------------------------------------------------
CREATE TABLE `meeting` (
                           `meeting_id`       BIGINT NOT NULL AUTO_INCREMENT COMMENT '모임 고유 식별자',
                           `host_id`          BIGINT NOT NULL COMMENT '방장(주최자)의 user_id',
                           `title`            VARCHAR(255) NOT NULL COMMENT '모임 제목',
                           `description`      TEXT NULL COMMENT '모임 상세 설명',
                           `image_url`        VARCHAR(255) NULL COMMENT '모임 대표 이미지 URL',
                           `book_title`       VARCHAR(255) NOT NULL COMMENT '선정 도서 제목',
                           `book_author`      VARCHAR(100) NOT NULL COMMENT '선정 도서 저자',
                           `genre`            VARCHAR(100) NULL COMMENT '모임 장르',
                           `meeting_time`     DATETIME NOT NULL COMMENT '모임 시간',
                           `location`         VARCHAR(255) NOT NULL COMMENT '모임 장소',
                           `max_participants` INT NOT NULL COMMENT '최대 참여 인원',
                           `status`           ENUM('RECRUITING','COMPLETED','CANCELLED')
                               NOT NULL DEFAULT 'RECRUITING' COMMENT '모임 상태',
                           `created_at`       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '모임 생성 시점',
                           `updated_at`       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '모임 정보 수정 시점',
                           PRIMARY KEY (`meeting_id`),
                           KEY `idx_meeting_host_id` (`host_id`),
                           KEY `idx_meeting_time` (`meeting_time`),
                           CONSTRAINT `fk_meeting_host`
                               FOREIGN KEY (`host_id`) REFERENCES `user`(`user_id`)
                                   ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='독서 모임';

-- -----------------------------------------------------
-- Table `meeting_participant`  (모임 참여/승인/출석 관리)
-- -----------------------------------------------------
CREATE TABLE `meeting_participant` (
                                       `id`         BIGINT NOT NULL AUTO_INCREMENT COMMENT '참여 고유 식별자',
                                       `meeting_id` BIGINT NOT NULL COMMENT '참여 모임의 id (meeting.meeting_id 참조)',
                                       `user_id`    BIGINT NOT NULL COMMENT '참여 사용자의 user_id (user.user_id 참조)',
                                       `role`       ENUM('HOST','PARTICIPANT') NOT NULL COMMENT '역할: HOST(주최자), PARTICIPANT(참여자)',
                                       `status`     ENUM('PENDING','APPROVED','REJECTED','ATTENDED','ABSENT')
                                           NOT NULL DEFAULT 'PENDING' COMMENT '참여 상태',
                                       `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '신청 시점',
                                       `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '상태 변경 시점',
                                       PRIMARY KEY (`id`),
                                       UNIQUE KEY `uq_meeting_user` (`meeting_id`, `user_id`), -- 모임당 1인 1참여 보장
                                       KEY `idx_mp_user_id` (`user_id`),
                                       CONSTRAINT `fk_mp_meeting`
                                           FOREIGN KEY (`meeting_id`) REFERENCES `meeting`(`meeting_id`)
                                               ON DELETE CASCADE ON UPDATE CASCADE,
                                       CONSTRAINT `fk_mp_user`
                                           FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`)
                                               ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='모임 참여자 (신청부터 출석까지 관리)';

-- -----------------------------------------------------
-- Table `meeting_review`  (모임 종료 후 멤버 간 좋아요/후기)
-- -----------------------------------------------------
CREATE TABLE `meeting_review` (
                                  `id`          BIGINT NOT NULL AUTO_INCREMENT COMMENT '후기(좋아요) 고유 식별자',
                                  `meeting_id`  BIGINT NOT NULL COMMENT '관련 모임의 id (meeting.meeting_id 참조)',
                                  `reviewer_id` BIGINT NOT NULL COMMENT '리뷰(좋아요)를 남긴 사용자 (user.user_id 참조)',
                                  `reviewee_id` BIGINT NOT NULL COMMENT '리뷰(좋아요)를 받은 사용자 (user.user_id 참조)',
                                  `created_at`  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '리뷰 작성 시간',
                                  PRIMARY KEY (`id`),
                                  UNIQUE KEY `uq_review` (`meeting_id`, `reviewer_id`, `reviewee_id`), -- 중복 좋아요 방지
                                  KEY `idx_mr_meeting_id` (`meeting_id`),
                                  KEY `idx_mr_reviewer_id` (`reviewer_id`),
                                  KEY `idx_mr_reviewee_id` (`reviewee_id`),
                                  CONSTRAINT `fk_mr_meeting`
                                      FOREIGN KEY (`meeting_id`) REFERENCES `meeting`(`meeting_id`)
                                          ON DELETE CASCADE ON UPDATE CASCADE,
                                  CONSTRAINT `fk_mr_reviewer`
                                      FOREIGN KEY (`reviewer_id`) REFERENCES `user`(`user_id`)
                                          ON DELETE CASCADE ON UPDATE CASCADE,
                                  CONSTRAINT `fk_mr_reviewee`
                                      FOREIGN KEY (`reviewee_id`) REFERENCES `user`(`user_id`)
                                          ON DELETE CASCADE ON UPDATE CASCADE,
    -- 자기 자신에게 좋아요 금지 (DB 버전에 따라 CHECK 동작 여부 상이)
                                  CHECK (`reviewer_id` <> `reviewee_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='모임 후기 (멤버 간 좋아요 평가)';

-- -----------------------------------------------------
-- Table `post`  (모임 전용 게시판의 게시글)
-- -----------------------------------------------------
CREATE TABLE `post` (
                        `post_id`    BIGINT NOT NULL AUTO_INCREMENT COMMENT '게시글 고유 식별자',
                        `meeting_id` BIGINT NOT NULL COMMENT '소속된 모임의 id (meeting.meeting_id 참조)',
                        `user_id`    BIGINT NOT NULL COMMENT '작성자의 user_id (user.user_id 참조)',
                        `title`      VARCHAR(200) NOT NULL COMMENT '게시글 제목',
                        `content`    TEXT NOT NULL COMMENT '게시글 내용',
                        `image_url`  VARCHAR(255) NULL COMMENT '이미지 URL',
                        `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '작성일',
                        `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '수정일',
                        PRIMARY KEY (`post_id`),
                        KEY `idx_post_meeting_id` (`meeting_id`),
                        KEY `idx_post_user_id` (`user_id`),
                        KEY `idx_post_created_at` (`created_at`),
                        CONSTRAINT `fk_post_meeting`
                            FOREIGN KEY (`meeting_id`) REFERENCES `meeting`(`meeting_id`)
                                ON DELETE CASCADE ON UPDATE CASCADE,
                        CONSTRAINT `fk_post_user`
                            FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`)
                                ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='모임별 게시판의 게시글';

-- -----------------------------------------------------
-- Table `comment`  (게시글 댓글)
-- -----------------------------------------------------
CREATE TABLE `comment` (
                           `id`         BIGINT NOT NULL AUTO_INCREMENT COMMENT '댓글 고유 식별자',
                           `post_id`    BIGINT NOT NULL COMMENT '원본 게시글의 id (post.post_id 참조)',
                           `user_id`    BIGINT NOT NULL COMMENT '작성자의 user_id (user.user_id 참조)',
                           `content`    TEXT NOT NULL COMMENT '댓글 내용',
                           `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '작성일',
                           `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '수정일',
                           PRIMARY KEY (`id`),
                           KEY `idx_comment_post_id` (`post_id`),
                           KEY `idx_comment_user_id` (`user_id`),
                           KEY `idx_comment_created_at` (`created_at`),
                           CONSTRAINT `fk_comment_post`
                               FOREIGN KEY (`post_id`) REFERENCES `post`(`post_id`)
                                   ON DELETE CASCADE ON UPDATE CASCADE,
                           CONSTRAINT `fk_comment_user`
                               FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`)
                                   ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='게시글의 댓글';
```
