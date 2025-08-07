# 데이터 흐름 및 엔터티 도출 (DB 설계 전초)

---

## 1. 서비스 플로우 기반 데이터 흐름 요약

```
1. [회원가입/로그인]
   - user(회원) 데이터 생성/조회 (이메일 중복 체크, 비밀번호 암호화)
2. [모임 생성]
   - meeting(모임) 데이터 생성, host(user_id) 참조, meeting_image(이미지) 관리
3. [모임 탐색/상세]
   - meeting(모임), meeting_status, genre, region 데이터 다수 조회
4. [모임 신청/승인/취소/만료]
   - meeting_participant(모임-사용자 매핑) 생성/상태변경(PENDING, APPROVED, REJECTED, EXPIRED, CANCELED)
   - participant_role(HOST/PARTICIPANT)
5. [모임 게시판]
   - post(게시글), comment(댓글) 생성/조회 (참여자/호스트만 작성, 권한 체크)
6. [오프라인 출석]
   - attendance(출석) 생성/수정 (참여/불참, 당일만 가능)
7. [모임 종료/좋아요]
   - meeting 상태 COMPLETED 변경
   - like(좋아요) 생성(본인 제외 참여자/호스트, 1회, 중복 불가)
8. [마이페이지]
   - user(회원) + user_profile(프로필) + 활동 통계(좋아요 수/모임 수 등) 조회
```

---

## 2. 엔터티 후보 및 관계 구조 (ERD 전 단계)

- **User** (회원)
- **Meeting** (모임)
- **MeetingParticipant** (모임 참여/신청/승인/역할)
- **Like** (좋아요)
- **Post** (모임 게시판 글)
- **Comment** (게시판 댓글)
- **Attendance** (모임 출석)
- (확장) **MeetingImage** (모임 이미지 별도 관리)
- (확장) **Genre, Region, MeetingStatus** (공통 코드 테이블)

### 엔터티 간 주요 관계

- User 1\:N Meeting (host)
- User N\:M Meeting (through MeetingParticipant)
- Meeting 1\:N Post, Post 1\:N Comment
- MeetingParticipant 1\:N Like (본인 제외)
- MeetingParticipant 1:1 Attendance (참여/불참)
- User 1\:N Like (받은/남긴)
- Meeting 1\:N MeetingImage

---

## 3. 핵심 엔터티별 주요 속성(컬럼) 도출 예시

```
### User
- user_id (PK) : bigint
- nickname : varchar(30) (중복 허용)
- email : varchar(100) (중복 불가, 로그인 ID)
- password : varchar(255) (암호화)
- profile_img_url : varchar(255)
- intro : text
- preferred_genre : varchar(50)
- created_at, updated_at : timestamp

### Meeting
- meeting_id (PK) : bigint
- title : varchar(255)
- host_id : bigint (User FK)
- book_title : varchar(255)
- book_author : varchar(100)
- genre : varchar(50)
- meeting_time : datetime
- region : varchar(50) (구 단위)
- detail_address : varchar(255)
- max_participants : int (2~12)
- image_url : varchar(255)
- status : enum(RECRUITING, COMPLETED, CANCELLED)
- created_at, updated_at : timestamp

### MeetingParticipant
- id (PK) : bigint
- meeting_id (FK) : bigint
- user_id (FK) : bigint
- role : enum(HOST, PARTICIPANT)
- status : enum(PENDING, APPROVED, REJECTED, EXPIRED, CANCELED)
- joined_at : datetime
- is_attended : boolean
- attended_at : datetime

### Like
- like_id (PK) : bigint
- meeting_id (FK) : bigint
- from_user_id (FK) : bigint (좋아요 남긴 사람)
- to_user_id (FK) : bigint (좋아요 받은 사람)
- created_at : timestamp
- UNIQUE (meeting_id, from_user_id, to_user_id)

### Post
- post_id (PK) : bigint
- meeting_id (FK) : bigint
- user_id (FK) : bigint (작성자)
- title : varchar(200)
- content : text
- image_url : varchar(255)
- created_at, updated_at : timestamp

### Comment
- comment_id (PK) : bigint
- post_id (FK) : bigint
- user_id (FK) : bigint (작성자)
- content : text
- created_at, updated_at : timestamp

### Attendance
- attendance_id (PK) : bigint
- meeting_id (FK) : bigint
- user_id (FK) : bigint
- status : enum(PRESENT, ABSENT, NO_RESPONSE)
- checked_at : datetime


---

