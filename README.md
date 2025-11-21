# **개인 개발 리포트 (How Do I Look — Backend Curation API)**

## 1. **프로젝트 개요**

**프로젝트명**: **How Do I Look**

**프로젝트 목적**: 본 프로젝트는 사용자가 등록한 스타일(Style)에 대해 다른 사용자들이 큐레이팅(Curating)을 남기고, 이를 안전하게 생성(Create)·조회(Read)·수정(Update)·삭제(Delete) 할 수 있도록 지원하는 백엔드 시스템을 구축하는 것을 목적으로 합니다.
더불어 본 프로젝트는 팀 단위 협업을 통해 진행되었으며, 특히 데이터 유효성 검증, 관계형 데이터 모델링, 안정적인 API 설계를 하는 것에 중점을 두었습니다.

**핵심 기능**:

- Style 기반 Curation 작성 기능 제공
- Curation 데이터 조회, 페이지네이션, 검색 기능
- 비밀번호 기반의 수정·삭제 인증 로직
- Superstruct 기반 요청 데이터 유효성 검증
- 전역 에러 핸들링 및 일관된 응답 구조 제공

## 2. **담당한 작업**

**역할**: Curation API 개발

**기여 내용**:

- Curation CRUD API 전체 구현

  - POST /styles/:styleId/curations
  - GET /styles/:styleId/curations (페이지네이션 + 검색)
  - PUT /curations/:curationId
  - DELETE /curations/:curationId

- Superstruct로 모든 요청값 검증

  - page/pageSize 범위 제한
  - nickname, content 글자 수 제한
  - 점수 필드(0~10) 범위 검증
  - optional vs nullable 정확한 처리

- 전역 에러 핸들링 설계
  - StructError → 400
  - ForbiddenError → 403
  - NotFoundError → 404
  - Prisma KnownErrors → 400/404/500
  - 그 외 예상치 못한 에러 → 500

## 3. **기술적 성과**

**사용 기술 스택**:

- **Backend**: Node.js / Express
- **Database**: PostgreSQL / Prisma ORM
- **협업 도구**: Discord, GitHub, Notion, Google Docs

**주요 성과**:

- 유효성 검증 체계 확립

  - Superstruct를 사용해 API 진입 단계에서 모든 잘못된 요청을 차단
  - page/pageSize upper bound 설정 → 비정상 요청 방어
  - update API에서 optional + nullable 조합으로 PUT method 의미에 중점을 두고자 했음

- 안정적인 에러 처리

  - 비밀번호 틀림 → ForbiddenError(403)
  - 없는 ID 조회 → 404
  - Prisma KnownError(P2025, P2002) 직접 대응
  - JSON 파싱 오류까지 400으로 통일 처리

- 검색 + 페이지네이션 기능 완성

  - searchBy가 동적으로 key가 되는 구조를 설계

- 관계형 데이터 모델 최적화
  - Comment·Style과의 관계를 명확히 하고 select 절을 최소화해 불필요한 데이터 반환을 줄임

## 4. **문제점 및 해결 과정**

**1.문제점**: Superstruct refine 시그니처 에러

- **상황**: refiner is not a function, Expected number but received null 같은 에러 발생

- **원인**: Superstruct v2에서는 refine(struct, name, fn) 순서가 달라져 있었음. 공식 문서와 다른 버전의 예시를 보고 작성하여 오류 발생

- **해결**: refine 함수 시그니처를 v2 기준으로 수정

**2.문제점**: Github, 브랜치 전략 이해도 부족으로 rebase loof, merge hell 발생

- **상황**: 2주간의 커밋이 쌓인 상태로 develop 브랜치에 rebase 시도, 이후 다시 feature 브랜치에 rebase 재시도로 인한 rebase loof, merge hell

- **원인**: 브랜치 전략의 이해도 부족과 진행 과정 중 성급한 판단으로 인해 feature 브랜치에 rebase 실행

- **해결**: 팀원들과 빠르게 의논하여 제시된 다양한 방법들을 순차적으로 실행. 기존 커밋 히스토리를 포기하더라도 새로운 브랜치를 만들어 소모 시간을 빠르게 단축시키기로 함. 이후 성공적으로 rebase, push, PR, merge에 성공함.

## 5. 협업 및 피드백

- 팀원들과 데일리 스크럼을 통해 진행 상황을 공유하고 문제를 빠르게 논의
- 팀원들과 코드 리뷰를 통해 받은 피드백을 적극 반영
- Prisma 관계 설계, Superstruct 사용 방식 등에 대한 상호 의견 교환으로 품질 향상

**배운 점**:

- optional과 nullable의 차이를 이해하게 됨
- 협업 시 브랜치 전략의 중요성을 체감
- 협업을 통해 같이 성장해 나가는 모습을 보고 느끼며 팀워크의 의미를 십분 체감할 수 있었음

## 6. 코드 품질 및 최적화

**단일 책임 원칙 준수**:

- controller / struct / error / router 파일 분리
- 로직의 관심사를 명확히 구분하여 유지보수 용이

**일관된 응답 형식**:

- 모든 에러 메시지, 상태코드를 통일
- 비정상 요청에 대한 빠른 차단으로 코드 가독성 증가

**일관된 응답 형식**:

- select로 필요한 필드만 가져오도록 최적화
- count + findMany를 Promise.all로 병렬 처리

## 7. 향후 개선 사항 및 제안

- **비밀번호 해싱 적용**: 현재는 평문 저장이므로 bcrypt 적용 필요
- **인증 시스템 확장**: 현재는 비밀번호 기반 접근 제어 → 향후 JWT 기반 사용자 인증 고려
