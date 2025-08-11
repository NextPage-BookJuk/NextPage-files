# 최소 방어 가이드 (FK/UNIQUE 없이 운영하려면)

## 1) 인덱스는 별도로 추가

* UNIQUE 대신 **일반 인덱스**라도 걸어 성능 확보

```sql
CREATE INDEX idx_user_email ON user(email);
CREATE INDEX idx_mp_meeting_user ON meeting_participant(meeting_id, user_id);
CREATE INDEX idx_mr_meeting_rev ON meeting_review(meeting_id, reviewer_id, reviewee_id);
```

## 2) 애플리케이션 레벨 검증 + 트랜잭션

* 삽입 전 존재 확인 → 같은 트랜잭션에서 처리
* 동시성 방지: `SELECT ... FOR UPDATE` 또는 저장 시나리오를 **단일 트랜잭션**으로 묶기
* 예시(개념):

  * “참여 신청” 서비스:

    * Tx 시작 → `meeting`/`user` 존재 확인
    * `meeting_participant`에 같은 `(meeting_id, user_id)` 있는지 확인(인덱스 사용)
    * 없으면 INSERT → Tx 커밋

## 3) 삭제/수정 시 수동 전파

* `user` 삭제 전: 해당 사용자의 `post/comment/participant/review` 먼저 정리
* `meeting` 삭제 전: `post/participant/review` 먼저 정리
* 실수 줄이려면 **서비스 계층에 전용 메서드**로 캡슐화

  * 예: `deleteMeetingWithChildren(meetingId)`

## 4) 정합성 배치 점검

* 야간 배치로 고아/중복 탐지 쿼리 실행

```sql
-- 고아 예
SELECT p.* FROM post p LEFT JOIN meeting m ON p.meeting_id = m.meeting_id WHERE m.meeting_id IS NULL;

-- 중복 예
SELECT meeting_id, user_id, COUNT(*) FROM meeting_participant GROUP BY 1, 2 HAVING COUNT(*) > 1;
```

## 5) 테스트(통합/동시성) 강화

* 중복 신청, 동시 좋아요, 부모 삭제 시 자식 정리 등 **경계 케이스**를 통합테스트로 고정

---

## 결론

* **중복 문제가 가장 즉각적이고 크지만**, **고아 레코드, 삭제 전파 부재, 성능 저하, 동시성 이슈**도 현실적으로 모두 발생
* FK/UNIQUE 없이 가려면 **인덱스 · 트랜잭션 · 수동 전파 · 배치 점검**으로 방어막을 반드시 세워야 함
