# 📑 BookJuk API 명세서 (최종)

> 모든 엔드포인트는 `/api` 하위에 구성되어 있습니다.\
> (예시: `/api/users`, `/api/meetings`, ...)

---

## 1. 회원 (User)

### 1-1. 회원가입

- **POST** `/api/users/register`
- **설명:** 이메일 중복 불가, 닉네임 중복 허용
- **권한:** 비로그인

**Request**

```json
{
  "nickname": "책읽는호랑이",
  "email": "book@user.com",
  "password": "securePwd1",
  "preferredGenre": "소설",
  "profileImage": "http://.../profile.jpg", // optional
  "introduction": "책을 사랑합니다."         // optional
}
```

**Response (성공)**

```json
{
  "userId": 1,
  "nickname": "책읽는호랑이",
  "email": "book@user.com",
  "createdAt": "2025-08-07T16:42:00"
}
```

---

### 1-2. 이메일 중복확인

- **GET** `/api/users/check-email?email={email}`
- **설명:** 회원가입 전 중복 체크
- **Response**

```json
{ "available": true }
```

---

### 1-3. 로그인

- **POST** `/api/users/login`
- **설명:** JWT/세션 기반\
  **Request**

```json
{
  "email": "book@user.com",
  "password": "securePwd1"
}
```

**Response**

```json
{
  "token": "JWT or session value",
  "user": {
    "userId": 1,
    "nickname": "책읽는호랑이"
  }
}
```

---

### 1-4. 내 정보 조회

- **GET** `/api/users/me`
- **권한:** 로그인\
  **Response**

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

---

### 1-5. 내 정보 수정

- **PUT** `/api/users/me`
- **설명:** 이메일은 수정 불가\
  **Request**

```json
{
  "nickname": "책읽는호랑이2",
  "profileImage": "http://.../profile2.jpg",
  "introduction": "소개 변경",
  "preferredGenre": "에세이"
}
```

**Response**

```json
{ "success": true }
```

---

## 2. 모임 (Meeting)

### 2-1. 모임 생성

- **POST** `/api/meetings`
- **권한:** 로그인 (호스트)
- **설명:** 최대 2\~12인, 이미지/주소/장르/책 필수 **Request**

```json
{
  "title": "8월 미드나잇 독서모임",
  "description": "밤에 모여 책 읽어요.",
  "imageUrl": "http://.../meet1.jpg",
  "bookTitle": "달까지 가자",
  "bookAuthor": "장류진",
  "genre": "소설",
  "meetingTime": "2025-08-15T19:30:00",
  "location": "서울시 강남구 독서카페 2층",
  "maxParticipants": 8
}
```

**Response**

```json
{
  "meetingId": 10,
  "title": "8월 미드나잇 독서모임"
}
```

---

### 2-2. 모임 리스트 조회 (검색/필터/정렬)

- **GET** `/api/meetings?region=강남구&genre=소설&status=RECRUITING&sort=latest&page=1&size=6`
- **설명:** 6개씩 카드, 필터(지역/장르/상태), 정렬(마감임박/최신/인기순) **Response**

```json
{
  "content": [
    {
      "meetingId": 10,
      "title": "8월 미드나잇 독서모임",
      "imageUrl": "http://.../meet1.jpg",
      "region": "강남구",
      "genre": "소설",
      "meetingTime": "2025-08-15T19:30:00",
      "status": "RECRUITING",
      "currentParticipants": 6,
      "maxParticipants": 8,
      "host": { "userId": 1, "nickname": "책읽는호랑이" }
    },
    ...
  ],
  "page": 1,
  "size": 6,
  "totalElements": 32
}
```

---

### 2-3. 모임 상세 조회

- **GET** `/api/meetings/{meetingId}` **Response**

```json
{
  "meetingId": 10,
  "title": "8월 미드나잇 독서모임",
  "description": "밤에 모여 책 읽어요.",
  "imageUrl": "http://.../meet1.jpg",
  "bookTitle": "달까지 가자",
  "bookAuthor": "장류진",
  "genre": "소설",
  "meetingTime": "2025-08-15T19:30:00",
  "location": "서울시 강남구 독서카페 2층",
  "maxParticipants": 8,
  "status": "RECRUITING",
  "createdAt": "2025-08-01T09:30:00",
  "updatedAt": "2025-08-01T09:35:00",
  "host": { "userId": 1, "nickname": "책읽는호랑이" },
  "participants": [
    { "userId": 1, "nickname": "책읽는호랑이", "role": "HOST", "status": "APPROVED" },
    { "userId": 2, "nickname": "책벌레", "role": "PARTICIPANT", "status": "APPROVED" }
  ],
  "postCount": 3
}
```

---

### 2-4. 모임 정보 수정

- **PUT** `/api/meetings/{meetingId}`
- **권한:** 호스트만 가능 **Request**

```json
{
  "title": "모임명 변경",
  "description": "소개 수정",
  "imageUrl": "http://.../meet2.jpg",
  "meetingTime": "2025-08-16T20:00:00",
  "location": "서울시 강남구 북카페 3층",
  "maxParticipants": 10
}
```

**Response**

```json
{ "success": true }
```

---

### 2-5. 모임 상태 변경

- **PATCH** `/api/meetings/{meetingId}/status`
- **권한:** 호스트만 가능\
  **Request**

```json
{
  "status": "COMPLETED" // 또는 "CANCELLED"
}
```

**Response**

```json
{ "success": true }
```

---

## 3. 모임 참여 (신청/관리)

### 3-1. 모임 참여 신청

- **POST** `/api/meetings/{meetingId}/participants`
- **권한:** 로그인(중복/본인 등 여러 조건 체크) **Request**

```json
{ }
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

---

### 3-2. 내 신청/참여 목록 조회

- **GET** `/api/meetings/my-participations` **Response**

```json
[
  {
    "meetingId": 10,
    "title": "8월 미드나잇 독서모임",
    "status": "APPROVED",
    "role": "PARTICIPANT"
  },
  ...
]
```

---

### 3-3. 신청자 승인/거절/만료/취소

- **PATCH** `/api/meetings/{meetingId}/participants/{participantId}`
- **권한:** 호스트(승인/거절/만료), 본인(취소) **Request**

```json
{
  "status": "APPROVED" // "REJECTED", "CANCELED", "EXPIRED" 등
}
```

**Response**

```json
{ "success": true }
```

---

## 4. 출석 (Attendance)

### 4-1. 출석/불참 체크

- **PATCH** `/api/meetings/{meetingId}/participants/{participantId}/attendance` **Request**

```json
{
  "status": "ATTENDED" // "ABSENT"
}
```

**Response**

```json
{ "success": true }
```

---

## 5. 모임 후기/좋아요 (Review/Like)

### 5-1. 후기(좋아요) 남기기

- **POST** `/api/meetings/{meetingId}/reviews`
- **권한:** 종료된 모임에서 본인 제외 1회, 취소/중복 불가 **Request**

```json
{
  "toUserId": 2
}
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

### 5-2. 내가 남긴/받은 좋아요 내역

- **GET** `/api/users/me/reviews` **Response**

```json
{
  "given": [
    { "meetingId": 10, "toUserId": 2, "toNickname": "책벌레" }
  ],
  "received": [
    { "meetingId": 10, "fromUserId": 3, "fromNickname": "독서광" }
  ]
}
```

---

## 6. 마이페이지

### 6-1. 내 모임/참여 리스트

- **GET** `/api/users/me/meetings` **Response**

```json
[
  {
    "meetingId": 10,
    "title": "8월 미드나잇 독서모임",
    "role": "HOST",
    "status": "COMPLETED"
  }
]
```

---

## 7. 모임 게시판 (Post/Comment)

### 7-1. 게시글 목록 조회

- **GET** `/api/meetings/{meetingId}/posts?page=1&size=10` **Response**

```json
{
  "content": [
    {
      "postId": 101,
      "title": "이번 모임 준비물",
      "userId": 2,
      "nickname": "책벌레",
      "createdAt": "2025-08-02T21:00:00",
      "commentCount": 3
    }
  ],
  "page": 1,
  "size": 10
}
```

---

### 7-2. 게시글 상세 조회

- **GET** `/api/meetings/{meetingId}/posts/{postId}` **Response**

```json
{
  "postId": 101,
  "meetingId": 10,
  "userId": 2,
  "nickname": "책벌레",
  "title": "이번 모임 준비물",
  "content": "간식, 책, 물 챙겨오세요.",
  "imageUrl": "http://.../post.jpg",
  "createdAt": "2025-08-02T21:00:00",
  "updatedAt": "2025-08-02T21:00:00",
  "comments": [
    {
      "commentId": 201,
      "userId": 3,
      "nickname": "독서광",
      "content": "감사합니다!",
      "createdAt": "2025-08-03T10:00:00"
    }
  ]
}
```

---

### 7-3. 게시글 등록

- **POST** `/api/meetings/{meetingId}/posts`
- **권한:** 참여자/호스트만 가능 **Request**

```json
{
  "title": "모임 준비사항",
  "content": "물, 책, 간식 준비",
  "imageUrl": "http://.../post2.jpg"
}
```

**Response**

```json
{ "postId": 102 }
```

---

### 7-4. 게시글 수정/삭제

- **PUT** `/api/meetings/{meetingId}/posts/{postId}`
- **DELETE** `/api/meetings/{meetingId}/posts/{postId}`
- **권한:** 작성자/호스트만\
  **Request** (수정)

```json
{
  "title": "제목 수정",
  "content": "내용 수정"
}
```

**Response**

```json
{ "success": true }
```

---

### 7-5. 댓글 등록/수정/삭제

- **POST** `/api/meetings/{meetingId}/posts/{postId}/comments`
- **PUT** `/api/meetings/{meetingId}/posts/{postId}/comments/{commentId}`
- **DELETE** `/api/meetings/{meetingId}/posts/{postId}/comments/{commentId}` **Request**

```json
{ "content": "댓글 내용" }
```

**Response**

```json
{ "commentId": 201 }
```

---



