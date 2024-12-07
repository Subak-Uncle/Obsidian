### Product Requirements Document (PRD) - 부트박스(BOOTBOX)

---

## 1. 서비스 개요

**부트박스(BOOTBOX)**는 IT 분야에 입문하거나 보다 전문적인 역량을 기르고자 하는 사용자들에게 다양한 부트캠프 프로그램을 추천하고 비교할 수 있는 플랫폼입니다. 

사용자의 특성과 선호도를 반영해 맞춤형 부트캠프를 추천하며, 부트캠프별 비교와 평가를 통해 사용자에게 최적의 선택을 돕습니다.

### 핵심 서비스

1. **맞춤형 부트캠프 추천**
2. **부트캠프 비교 기능**
3. **부트캠프 리스트 제공 및 검색/필터링**
4. **간편한 부트캠프 등록**
5. **광고 및 홍보 배너 제공**

---

## 2. 타겟 사용자 및 페르소나
### 타겟층

- 20~30대 
- 직무 전환을 준비하는 사람들
- 대학생 및 취업준비생
- 학원 홍보 담당자

### 페르소나 정의

**1. 비전공 직무 전환자 (30대 마케팅 매니저 김민수)**

- 목표: 안정적인 IT 업계로 직무 전환
- 기능 요구사항: 포트폴리오 제공, 유연한 학습 일정, 커리어 상담

**2. 개발자 취업을 준비하는 전공자 (20대 컴퓨터 공학 전공 대학생 이지훈)**

- 목표: 실무 경험을 쌓고, 최신 기술 습득
- 기능 요구사항: 실무 프로젝트 기회, 최신 기술 교육, 커리어 상담

**3. 비전공자 취업준비생 (20대 문예창작과 졸업생 박지영)**

- 목표: 신입 개발자로 취업
- 기능 요구사항: 기초 코딩 교육, 체계적인 교육 과정, 커리어 상담

**4. 학원 홍보 담당자 (32세 학원 홍보 담당자 김민재)**

- 목표: 학원 교육과정 홍보 및 수강생 모집
- 기능 요구사항: 부트캠프 등록, 홍보 배너, 등록 절차 간소화

---

## 3. 주요 기능 요구사항

### 3.1 사용자 맞춤형 부트캠프 추천 기능

- **직무 MBTI 테스트**: 사용자 성향 및 직무 적합도 분석을 통해 추천
  - 로그인 후 사이트 초기 진입 시 MBTI 설문 제안
  - 성향에 맞춘 직무별 부트캠프 추천 결과 제공
- **부트캠프 검색 필터**: 기간, 비용, 지역, 지원금, 선발 절차 등의 필터링 옵션
- **선호도 기반 추천**: 사용자의 강의 선호도를 설문으로 입력받아 추천

### 3.2 부트캠프 비교 기능

- **부트캠프 리스트 제공**: 목록 형태로 부트캠프를 제공하고 비교 가능
  - 부트캠프별 기본 정보, 수강비, 커리큘럼 등을 포함한 상세 페이지 제공

### 3.3 부트캠프 리스트 및 검색/필터 기능

- **검색 및 필터링**: 사용자에게 상세한 부트캠프 정보 제공을 위해 다양한 필터링 및 검색 기능 제공
- **정렬 옵션**: 평점순, 등록순, 비용순 등 다양한 정렬 옵션 제공

### 3.4 간편 부트캠프 등록 기능

- **부트캠프 등록**: 학원 홍보 담당자가 새로운 부트캠프 과정을 쉽게 등록할 수 있음
  - 등록 양식 간소화 및 필요한 정보만 기입할 수 있는 기능
	  - 등록 양식을 직접 컨트롤 할 수 있음
  - 부트캠프 정보 파일을 업로드 시 자동 기입
- **홍보 배너 등록**: 학원 홍보 담당자가 광고 배너 요청을 통해 추가 홍보 가능

### 3.5 메인 화면 및 사용자 경험 향상

- **메인 페이지**: 광고 배너, 내비게이션, 직무 MBTI 테스트, 추천 기능 배치
- **사용자 경험 강화**: 간편 회원가입 및 로그인 기능 제공
- **직무 설문조사**: 설문조사를 통해 사용자에게 맞는 부트캠프를 추천하는 페이지

---

## 4. 기술 요구사항

### 프론트엔드
- React, Next.js를 통한 사용자 인터페이스 구현
- Zustand를 통한 전역 상태 관리 및 사용자 데이터 관리
- 직무 MBTI 및 선호도 설문조사 기능을 위한 설문 API 연동

### 백엔드
- Spring Boot 기반 API 서버 구축
- JWT 인증 방식으로 로그인 및 세션 관리
- 데이터베이스: PostgreSQL을 사용하여 부트캠프 및 사용자 데이터 저장
- Redis 캐시 시스템을 통해 데이터 로드 최적화

### 데이터 및 인프라
- AWS 인프라를 활용한 서버 배포 및 S3를 통한 이미지 관리
- ElasticSearch를 통한 부트캠프 리스트 검색 최적화
- GitHub Actions를 사용한 CI/CD 파이프라인 구축

---

## 5. 개발 로드맵

1. **1단계 (MVP 개발)**
   - 사용자 맞춤형 부트캠프 추천, 부트캠프 리스트 제공, 간편 로그인/회원가입
2. **2단계 (추천 및 비교 기능 고도화)**
   - 직무 MBTI 및 설문조사 기반 추천 기능, 부트캠프 비교 기능 강화
3. **3단계 (학원 홍보 및 관리 기능 추가)**
   - 학원 광고 배너 등록, 트래픽 분석 기능 추가
4. **4단계 (챗봇 및 추천 알고리즘 고도화)**
   - 챗봇 연동 및 AI 기반 추천 시스템 추가

---
