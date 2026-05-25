# CoWorks Backend

> 문서 작성부터 서명까지, 한 번에

CoWorks는 대학 내 종이 문서, 수기 서명, 이메일 중심의 행정 문서 처리 과정을 온라인으로 전환하기 위한 전자문서 처리 플랫폼입니다. 이 저장소는 CoWorks 서비스의 백엔드 영역으로, 문서 생성, 작성, 검토, 전자서명, 알림, 폴더 관리 등 핵심 비즈니스 로직과 API를 담당합니다.

## 프로젝트 소개

CoWorks Backend는 PDF 기반 행정 문서를 템플릿으로 등록하고, 해당 템플릿을 바탕으로 문서를 생성한 뒤 작성, 검토, 서명, 완료 단계까지 처리하는 서버 애플리케이션입니다.

문서마다 입력 필드, 테이블 필드, 서명 위치가 달라질 수 있기 때문에 문서 데이터는 PostgreSQL의 `JSONB` 타입으로 유연하게 저장합니다. 사용자, 문서, 역할, 폴더, 알림처럼 관계가 명확한 데이터는 JPA 엔티티로 관리하여 문서 처리 흐름의 정합성을 유지합니다.

## 주요 기능

- **인증 및 사용자 관리**
  - 회원가입, 로그인, JWT 발급
  - 히즈넷 로그인 연동
  - 사용자 검색
  - 직분과 권한에 따른 기능 접근 제어

- **템플릿 관리**
  - PDF 템플릿 업로드
  - PDF 이미지 변환 및 다중 페이지 처리
  - 입력 필드, 테이블 필드, 서명 필드 좌표 정보 저장
  - 템플릿 수정, 삭제, 복제
  - 템플릿 기본 저장 폴더 설정

- **문서 워크플로우**
  - 템플릿 기반 문서 생성
  - 문서 작성 데이터 저장
  - 편집자, 검토자, 서명자 역할 지정
  - 검토 승인 및 반려
  - 전자서명 승인 및 반려
  - 문서 상태 변경 이력 기록

- **공개 서명 링크**
  - 서명 토큰 기반 공개 서명 문서 조회
  - 이메일 링크를 통한 외부 서명 처리
  - 토큰 만료, 사용 여부, 접근 횟수 관리

- **대량 문서 생성**
  - Excel, CSV 파일 업로드
  - 업로드 데이터 검증 및 스테이징 저장
  - 미리보기 후 문서 일괄 생성
  - 미등록 사용자 임시 할당 및 가입 후 자동 연결

- **폴더 기반 문서 관리**
  - 폴더 생성, 수정, 삭제
  - 폴더 트리 조회
  - 문서 폴더 이동
  - 미분류 문서 조회

- **알림 및 메일**
  - 문서 할당, 검토 요청, 서명 요청, 반려 알림 메일 발송
  - SSE 기반 실시간 알림 스트림 제공
  - 읽지 않은 알림 조회 및 읽음 처리

## 문서 상태 흐름

```text
DRAFT
  -> EDITING
  -> READY_FOR_REVIEW
  -> REVIEWING
  -> SIGNING
  -> COMPLETED
```

검토 또는 서명 단계에서 반려되면 `REJECTED` 이력이 기록되고, 문서는 다시 `EDITING` 상태로 돌아가 수정할 수 있습니다.

## 주요 도메인

- `User` - 사용자 정보, 직분, 권한
- `Template` - PDF 템플릿과 좌표 기반 필드 정보
- `Document` - 실제 작성 대상 문서와 문서 데이터
- `DocumentRole` - 문서별 작성자, 편집자, 검토자, 서명자 역할
- `DocumentStatusLog` - 문서 상태 변경 이력
- `Folder` - 문서 분류를 위한 폴더 구조
- `Notification` - 사용자별 알림
- `SigningToken` - 공개 서명 링크 접근 토큰
- `BulkStaging`, `BulkStagingItem` - 대량 문서 생성 임시 데이터

## API 구성

- `/auth` - 인증, 회원가입, 로그인, 사용자 정보 조회
- `/users` - 사용자 검색
- `/templates` - 템플릿 업로드, 조회, 수정, 삭제, 복제
- `/documents` - 문서 생성, 조회, 수정, 삭제, 워크플로우 처리
- `/documents/bulk` - 대량 문서 생성 미리보기, 확정, 취소
- `/folders` - 폴더 및 폴더 내 문서 관리
- `/notifications` - 알림 조회, 읽음 처리, SSE 스트림
- `/pdf` - PDF 이미지 변환
- `/files` - 업로드된 PDF 및 이미지 파일 제공
- `/public/sign` - 토큰 기반 공개 서명

## 기술 스택

- **Java 17**
- **Spring Boot 3.2.3**
- **Spring Web**
- **Spring Data JPA**
- **Spring Security**
- **PostgreSQL**
- **JWT**
- **Thymeleaf Mail Template**
- **PDFBox**
- **iText**
- **Apache POI**
- **Commons CSV**
- **Lombok**

## 프로젝트 구조

```text
src/main/java/com/hiswork/backend/
├── annotation/       # 커스텀 어노테이션
├── aspect/           # 접근 권한 검증 AOP
├── config/           # 보안, CORS, 메일 설정
├── controller/       # REST API 컨트롤러
├── domain/           # JPA 엔티티와 enum
├── dto/              # 요청/응답 DTO
├── exception/        # 예외 처리
├── repository/       # Spring Data JPA Repository
├── service/          # 비즈니스 로직
└── util/             # 인증, JWT 유틸리티
```

## 실행 환경

- Java 17
- PostgreSQL
- Gradle

`application.yml`은 `.env` 파일을 선택적으로 불러오도록 설정되어 있습니다.

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=coworks
DB_USERNAME=postgres
DB_PASSWORD=password

JWT_SECRET_KEY=your-jwt-secret-key

MAIL_USERNAME=your-mail-account
MAIL_PASSWORD=your-mail-password

FRONTEND_URL=http://localhost:5173

HISNET_URL=your-hisnet-url
HISNET_ACCESS_KEY=your-hisnet-access-key
```

## 실행 방법

```bash
./gradlew bootRun
```

Gradle wrapper가 없는 환경에서는 아래 명령어를 사용할 수 있습니다.

```bash
gradle bootRun
```

기본 서버 주소는 `http://localhost:8080`입니다.

## 테스트

```bash
./gradlew test
```

## 파일 저장 경로

업로드된 PDF 템플릿과 변환 이미지는 기본적으로 아래 경로에 저장됩니다.

```text
uploads/
└── pdf-templates/
```
