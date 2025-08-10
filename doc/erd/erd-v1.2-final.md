
```sql
-- =================================================================
--  1. 데이터베이스 생성 및 선택 (이 부분을 추가하세요!)
-- =================================================================
CREATE DATABASE IF NOT EXISTS `book_juk_db` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE `book_juk_db`;

-- =================================================================
--  독서모임 서비스 DDL (리팩토링 ver. / 외래 키, 인덱스/제약조건 일부 추가)
-- =================================================================
-- 주의:
-- 테이블 간의 데이터 정합성은 애플리케이션 코드 레벨에서 반드시 검증해야 합니다.
-- (예: INSERT 전 meeting_id, user_id 등의 존재 여부를 코드로 직접 확인)
-- 데이터 양이 많아질 경우, 인덱스가 없어 검색 성능이 저하될 수 있습니다.
-- =================================================================

-- -----------------------------------------------------
-- Table `user`
-- 사용자 정보를 관리합니다.
-- -----------------------------------------------------
CREATE TABLE `user` (
                        `user_id`        BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '유저 고유 식별자',
                        `nickname`       VARCHAR(50) NOT NULL COMMENT '닉네임 (중복 허용)',   -- username → nickname 통일
                        `email`          VARCHAR(100) NOT NULL UNIQUE COMMENT '이메일 (로그인 ID)',
                        `password`       VARCHAR(255) NOT NULL COMMENT '비밀번호 (해시화하여 저장)',
                        `preferred_genre` VARCHAR(100) NULL COMMENT '선호 장르',
                        `profile_image`  VARCHAR(255) NULL DEFAULT 'default_profile.jpg' COMMENT '프로필 이미지 URL',
                        `introduction`   TEXT NULL COMMENT '자기소개',
                        `created_at`     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '회원 가입 시점',
                        `updated_at`     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '회원 정보 수정 시점'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='사용자 정보';

-- -----------------------------------------------------
-- Table `meeting`
-- 독서 모임 정보를 관리합니다.
-- -----------------------------------------------------
CREATE TABLE `meeting` (
                           `meeting_id`       BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '모임 고유 식별자',
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
                           `status`           VARCHAR(20) NOT NULL DEFAULT 'RECRUITING' COMMENT '모임 상태: RECRUITING(모집중), COMPLETED(종료), CANCELLED(취소)',
                           `created_at`       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '모임 생성 시점',
                           `updated_at`       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '모임 정보 수정 시점',
                           FOREIGN KEY (`host_id`) REFERENCES `user`(`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='독서 모임';

-- -----------------------------------------------------
-- Table `meeting_participant`
-- 사용자의 모임 신청, 승인, 출석 상태를 모두 관리합니다.
-- -----------------------------------------------------
CREATE TABLE `meeting_participant` (
                                       `id`         BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '참여 고유 식별자',
                                       `meeting_id` BIGINT NOT NULL COMMENT '참여 모임의 id (meeting.id 참조)',
                                       `user_id`    BIGINT NOT NULL COMMENT '참여 사용자의 user_id (user.user_id 참조)',
                                       `role`       VARCHAR(20) NOT NULL COMMENT 'HOST(주최자), PARTICIPANT(참여자)',
                                       `status`     VARCHAR(20) NOT NULL DEFAULT 'PENDING' COMMENT '참여 상태: PENDING(신청), APPROVED(승인), REJECTED(거절), ATTENDED(출석), ABSENT(불참)',
                                       `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '신청 시점',
                                       `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '상태 변경 시점',
                                       UNIQUE KEY uq_meeting_user (`meeting_id`, `user_id`),  -- 모임당 1인 1참여 보장
                                       FOREIGN KEY (`meeting_id`) REFERENCES `meeting`(`meeting_id`),
                                       FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='모임 참여자 (신청부터 출석까지 관리)';

-- -----------------------------------------------------
-- Table `meeting_review`
-- 모임 종료 후, 멤버 간의 후기(좋아요)를 기록합니다.
-- -----------------------------------------------------
CREATE TABLE `meeting_review` (
    `id`           BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '후기(좋아요) 고유 식별자',
    `meeting_id`   BIGINT NOT NULL COMMENT '관련 모임의 id (meeting.id 참조)',
    `reviewer_id`  BIGINT NOT NULL COMMENT '리뷰(좋아요)를 남긴 사용자의 user_id (user.user_id 참조)',
    `reviewee_id`  BIGINT NOT NULL COMMENT '리뷰(좋아요)를 받은 사용자의 user_id (user.user_id 참조)',
    `created_at`   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '리뷰 작성 시간',
    UNIQUE KEY uq_review (`meeting_id`, `reviewer_id`, `reviewee_id`),  -- 중복 좋아요 방지!
    FOREIGN KEY (`meeting_id`) REFERENCES `meeting`(`meeting_id`),
    FOREIGN KEY (`reviewer_id`) REFERENCES `user`(`user_id`),
    FOREIGN KEY (`reviewee_id`) REFERENCES `user`(`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='모임 후기 (멤버 간 좋아요 평가)';

-- -----------------------------------------------------
-- Table `post`
-- 각 모임의 전용 게시판에 작성되는 게시글을 관리합니다.
-- -----------------------------------------------------
CREATE TABLE `post` (
    `post_id`         BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '게시글 고유 식별자',
    `meeting_id`      BIGINT NOT NULL COMMENT '소속된 모임의 id (meeting.id 참조)',
    `user_id`         BIGINT NOT NULL COMMENT '작성자의 user_id (user.user_id 참조)',
    `title`           VARCHAR(200) NOT NULL COMMENT '게시글 제목',
    `content`         TEXT NOT NULL COMMENT '게시글 내용',
    `image_url`       VARCHAR(255) NULL COMMENT '이미지 URL',
    `created_at`      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '작성일',
    `updated_at`      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '수정일',
    FOREIGN KEY (`meeting_id`) REFERENCES `meeting`(`meeting_id`),
    FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='모임별 게시판의 게시글';

-- -----------------------------------------------------
-- Table `comment`
-- 게시글에 대한 댓글을 관리합니다.
-- -----------------------------------------------------
CREATE TABLE `comment` (
    `id`         BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '댓글 고유 식별자',
    `post_id`    BIGINT NOT NULL COMMENT '원본 게시글의 id (post.id 참조)',
    `user_id`    BIGINT NOT NULL COMMENT '작성자의 user_id (user.user_id 참조)',
    `content`    TEXT NOT NULL COMMENT '댓글 내용',
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '작성일',
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '수정일',
    FOREIGN KEY (`post_id`) REFERENCES `post`(`post_id`),
    FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='게시글의 댓글';
```
