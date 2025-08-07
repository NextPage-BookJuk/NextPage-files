```bash
bookjuk/
├── build.gradle or pom.xml         # 빌드 스크립트 (Gradle 또는 Maven)
├── .gitignore                      # Git 버전 관리에서 제외할 파일 목록
├── README.md                       # 프로젝트 설명 및 안내 문서
│
├── docs/                           # 프로젝트 관련 문서 폴더
│   ├── api.md                      # API 명세서
│   ├── erd.png                     # ERD (데이터베이스 관계도)
│   └── til.md                      # 팀원들의 Today I Learned 기록
│
│
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/bookjuk/
    │   │       ├── BookJukApplication.java # Spring Boot 메인 애플리케이션
    │   │       │
    │   │       ├── config/         # ✏️ 전역 설정 (Security, JPA, Cors 등)
    │   │       ├── domain/         # 📦 도메인 엔티티 (User, Meeting, Participant 등)
    │   │       ├── dto/            # 📮 데이터 전송 객체 (Request/Response)
    │   │       ├── repository/     # 🗄️ 데이터 접근 계층 (JPA Repository, QueryDSL)
    │   │       ├── service/        # ✨ 비즈니스 로직 처리 계층
    │   │       ├── controller/     # 🎮 API 엔드포인트 및 웹 요청 처리
    │   │       ├── exception/      # 🚨 전역 및 커스텀 예외 처리
    │   │       └── util/           # 🛠️ 공통 유틸리티 클래스 (날짜, 파일 처리 등)
    │   │
    │   └── resources/
    │       ├── application.yml     # 애플리케이션 환경설정 파일
    │       ├── static/             # 정적 리소스 (CSS, JavaScript, 이미지 등)
    │       └── templates/          # 서버 사이드 뷰 템플릿 (Thymeleaf, JSP 등)
```
