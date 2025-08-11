# 📑 BookJuk API 명세서 — DDL 정합 패치 v1

> 본 문서는 **현재 DDL(2025-08-11 기준)** 과의 정합성을 맞추기 위해 필요한 변경 사항만 반영한 **패치 버전**입니다. 기존 명세에서 **바뀐 섹션만** 전체 형태로 재기술했습니다. (그 외 엔드포인트/로직은 종전 명세 유지)

---

## 0. 용어/스키마 정합성 가이드

* 주소 컬럼: `region`(시/도), `city`(시/구/군), `detailAddress`(선택)
* 사용자 프로필: `nickname`, `email`, `profileImage`, `introduction`, `preferredGenre`(단일 문자열)
* 참여/출석 상태는 **동일 컬럼**(`meeting_participant.status`)에서 관리

  * 허용값: `PENDING`, `APPROVED`, `REJECTED`, `CANCELED`, `EXPIRED`, `ATTENDED`, `ABSENT`
* 파생 값(서버 계산):

  * `currentParticipants` = meeting\_participant 에서 `status='APPROVED'` 카운트
  * `receivedLikes` = meeting\_review 에서 `reviewee_id = user_id` 카운트
* 제약: DB는 UNIQUE/FK가 없으므로 **애플리케이션 트랜잭션 내 중복 검사** 필요

---

## 1. 회원 (User) — 마이페이지 관련 필드 정합화

### 1-4. 내 정보 조회 (변경 사항만)

* **GET** `/api/users/me`

**Response (예시)**

```json
{
  "userId": 1,
  "nickname": "책읽는호랑이",
  "email": "book@user.com",
  "profileImage": "http://.../profile.jpg",
  "introduction": "책을 사랑합니다.",
  "preferredGenre": "소설",
  "createdAt": "2025-08-07T16:42:00",
  "receivedLikes": 7,           
  "joinedMeetings": 3           
}
```

> `preferredGenres`(배열) → `preferredGenre`(문자열)로 정합화.

---

## 2. 모임 (Meeting) — 주소/필터 정합화

### 2-1. 모임 생성 (Request/Response 변경)

* **POST** `/api/meetings`
* **권한:** 로그인 (호스트)
* **설명:** 이미지/주소/책 필수, **장르는 선택**(DDL 상 NULL 허용)

**Request**

```json
{
  "title": "8월 미드나잇 독서모임",
  "description": "밤에 모여 책 읽어요.",
  "imageUrl": "http://.../meet1.jpg",
  "bookTitle": "달까지 가자",
  "bookAuthor": "장류진",
  "genre": "소설",                
  "meetingTime": "2025-08-15T19:30:00",
  "region": "서울특별시",        
  "city": "강남구",              
  "detailAddress": "독서카페 2층",
  "maxParticipants": 8
}
```

**Response (성공)**

```json
{
  "meetingId": 10,
  "title": "8월 미드나잊 독서모임"
}
```

### 2-2. 모임 리스트 조회 (필터/정렬 파라미터 변경)

* **GET** `/api/meetings?region=서울특별시&city=강남구&genre=소설&status=RECRUITING&sort=latest&page=1&size=6`
* **설명:** 6개씩 카드, 필터(`region`, `city`, `genre`, `status`), 정렬(`latest`/`deadline`/`popular`)

**Response (예시)**

```json
{
  "content": [
    {
      "meetingId": 10,
      "host": {
        "userId": 1,
        "nickname": "책읽는호랑이",
        "profileImage": "http://.../profile1.jpg",
        "introduction": "책을 사랑합니다."
      },
      "title": "8월 미드나잊 독서모임",
      "description": "조용한 사람들을 찾는 모임입니다.",
      "imageUrl": "http://.../meet1.jpg",
      "bookTitle": "돈의 속성",
      "bookAuthor": "김승호",
      "genre": "소설",
      "meetingTime": "2025-08-15T19:30:00",
      "region": "서울특별시",
      "city": "강남구",
      "detailAddress": "독서카페 2층",
      "maxParticipants": 8,
      "currentParticipants": 6,
      "status": "RECRUITING",
      "createdAt": "2025-08-14T19:30:00",
      "updatedAt": "2025-08-14T20:30:00"
    }
  ],
  "page": 1,
  "size": 6,
  "totalElements": 32
}
```

### 2-3. 모임 상세 조회 (주소/프로필 정합)

* **GET** `/api/meetings/{meetingId}`

**Response (예시)**

```json
{
  "meetingId": 10,
  "title": "8월 미드나잊 독서모임",
  "description": "조용한 사람들을 찾는 모임입니다.",
  "imageUrl": "http://.../meet1.jpg",
  "bookTitle": "돈의 속성",
  "bookAuthor": "김승호",
  "genre": "소설",
  "meetingTime": "2025-08-15T19:30:00",
  "region": "서울특별시",
  "city": "강남구",
  "detailAddress": "독서카페 2층",
  "maxParticipants": 8,
  "currentParticipants": 6,
  "status": "RECRUITING",
  "createdAt": "2025-08-14T19:30:00",
  "updatedAt": "2025-08-14T20:30:00",
  "host": {
    "userId": 1,
    "nickname": "책읽는호랑이",
    "profileImage": "http://.../profile1.jpg",
    "introduction": "책을 사랑합니다."
  },
  "participants": [
    { "userId": 1, "nickname": "책읽는호랑이", "role": "HOST", "status": "APPROVED", "profileImage": "http://.../profile1.jpg" },
    { "userId": 2, "nickname": "책벌레", "role": "PARTICIPANT", "status": "PENDING", "profileImage": "http://.../profile2.jpg" }
  ]
}
```

### 2-4. 모임 정보 수정 (주소 필드 반영)

* **PUT** `/api/meetings/{meetingId}`
* **권한:** 호스트만

**Request (예시)**

```json
{
  "title": "모임명 변경",
  "description": "소개 수정",
  "imageUrl": "http://.../meet2.jpg",
  "meetingTime": "2025-08-16T20:00:00",
  "region": "서울특별시",
  "city": "서초구",
  "detailAddress": "북카페 3층",
  "maxParticipants": 10
}
```

**Response**

```json
{ "success": true }
```

---

## 3. 모임 참여/출석 — 단일 status 컬럼 사용 명시

### 3-1. 모임 참여 신청 (변경 없음, 주석 보강)

* **POST** `/api/meetings/{meetingId}/participants`
* **권한:** 로그인 (중복/본인/정원/진행상태 등 검증은 서비스 단에서 트랜잭션으로 보장)

**Request**

```json
{}
```

**Response**

```json
{
  "participantId": 15,
  "meetingId": 10,
  "userId": 3,
  "role": "PARTICIPANT",
  "status": "PENDING"
}
```

### 3-3. 신청자 상태 변경 (승인/거절/만료/취소)

* **PATCH** `/api/meetings/{meetingId}/participants/{participantId}`
* **권한:** 호스트(승인/거절/만료), 본인(취소)

**Request**

```json
{ "status": "APPROVED" }
```

**Response**

```json
{ "success": true }
```

### 4-1. 출석/불참 체크 (status 컬럼 변경으로 규정)

* **PATCH** `/api/meetings/{meetingId}/participants/{participantId}/attendance`
* **설명:** 출석/불참은 `meeting_participant.status` 값을 변경 (`ATTENDED`/`ABSENT`)

**Request**

```json
{ "status": "ATTENDED" }
```

**Response**

```json
{ "success": true }
```

---

## 5. 후기/좋아요 — 스키마 적합성 유지 (변경 없음, 주석 보강)

### 5-1. 후기(좋아요) 남기기

* **POST** `/api/meetings/{meetingId}/reviews`
* **권한:** 종료된 모임에서 본인 제외 1회, 취소/중복 불가

**Request**

```json
{ "toUserId": 2 }
```

**Response**

```json
{
  "reviewId": 33,
  "fromUserId": 1,
  "toUserId": 2,
  "createdAt": "2025-08-20T21:00:00"
}
```

---

## 6. 마이페이지 API — 스키마 정합화

### 6-1. 마이페이지 정보 조회 (필드/타입 수정)

* **GET** `/api/mypage`
* **권한:** 로그인

**Response (성공 200)**

```json
{
  "status": true,
  "message": "마이페이지 정보 조회를 성공했습니다.",
  "timestamp": "2025-08-07T12:12:12.2987169",
  "data": {
    "profile": {
      "nickname": "책벌레123",
      "email": "user@example.com",
      "profileImage": "https://example.com/profile/123.jpg",
      "introduction": "안녕하세요! 추리소설을 좋아합니다.",
      "preferredGenre": "추리"
    },
    "statistics": {
      "receivedLikes": 15,
      "participatedMeetings": 8
    },
    "meetings": [
      {
        "title": "추리소설 읽기 모임",
        "date": "2024-08-15T19:00:00Z",
        "status": "completed",
        "role": "participant",
        "book": { "title": "셜록 홈즈", "author": "아서 코난 도일" }
      }
    ]
  }
}
```

**오류**

* 401 Unauthorized
* 404 USER\_NOT\_FOUND

### 6-2. 마이페이지 프로필 수정 (필드명/응답 문구 수정)

* **PUT** `/api/mypage/profile`
* **Headers:** `Authorization: Bearer <token>`; `Content-Type: multipart/form-data`
* **Form fields:**

  * `profileImage` (file, optional)
  * `nickname` (string, optional)
  * `introduction` (string, optional)
  * `preferredGenre` (string, optional)

**Response (성공 200)**

```json
{
  "status": true,
  "message": "마이페이지 프로필을 수정했습니다.",
  "timestamp": "2025-08-07T12:12:12.2987169",
  "data": {
    "nickname": "새로운닉네임",
    "profileImage": "https://example.com/profile/123_new.jpg",
    "introduction": "새로운 자기소개입니다.",
    "preferredGenre": "추리"
  }
}
```

**오류**

* 400 INVALID\_INPUT
* 401 Unauthorized

---

## 7. 모임 게시판 (Post/Comment) — 권한 주석 보강

### 7-1. 게시글 목록 조회 (변경 없음)

* **GET** `/api/meetings/{meetingId}/posts?page=1&size=10`

### 7-2. 게시글 상세 조회 (변경 없음)

* **GET** `/api/meetings/{meetingId}/posts/{postId}`

### 7-3. 게시글 등록 (권한 명시)

* **POST** `/api/meetings/{meetingId}/posts`
* **권한:** **해당 모임의 HOST 또는 APPROVED 상태의 PARTICIPANT만 가능**

**Request**

```json
{ "title": "모임 준비사항", "content": "물, 책, 간식 준비", "imageUrl": "http://.../post2.jpg" }
```

**Response**

```json
{ "postId": 102 }
```

### 7-4. 게시글 수정/삭제 (변경 없음, 권한 주석)

* **PUT** `/api/meetings/{meetingId}/posts/{postId}`
* **DELETE** `/api/meetings/{meetingId}/posts/{postId}`
* **권한:** 작성자 또는 호스트

### 7-5. 댓글 등록/수정/삭제 (변경 없음)

* **POST** `/api/meetings/{meetingId}/posts/{postId}/comments`
* **PUT** `/api/meetings/{meetingId}/posts/{postId}/comments/{commentId}`
* **DELETE** `/api/meetings/{meetingId}/posts/{postId}/comments/{commentId}`

**Request**

```json
{ "content": "댓글 내용" }
```

**Response**

```json
{ "commentId": 201 }
```

---

## 8. 운영/검증 주의사항 (DDL 제약 부재 대응)

* 이메일 중복 체크: DB UNIQUE 미사용 → **동일 트랜잭션 내 중복 재검증**
* 참여 신청: `(meeting_id, user_id)` 중복 방지 검증 → 인덱스 활용 + 트랜잭션 재확인
* 삭제 전 참조 데이터 정리: FK 미사용이므로 서비스 계층에서 순서 보장
