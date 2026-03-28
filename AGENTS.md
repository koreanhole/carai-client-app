# Carai 프로젝트 소프트웨어 설계 및 에이전트 가이드

본 문서는 Carai(차량 문서 및 점검 스캐너 앱)의 비즈니스 로직, 상세 기능, 사용성(Flow), 서비스 목표 및 소프트웨어 설계 원칙을 정의한 가이드라인입니다. 모든 개발자 및 AI 에이전트는 본 문서를 바탕으로 프로젝트를 이해하고 코드를 작성해야 합니다.

## 1. 서비스 목표 (Service Goals)
- **페이퍼리스(Paperless) 작업 환경 구축:** 기존 정비소의 종이 기반 점검표 및 문서를 디지털화하여 문서 관리 및 전송의 효율성을 높입니다.
- **AI 기반 점검 자동화 및 보조:** 차량 점검 시 OCR 스캔 및 AI 시각 분석을 통해 결함 및 마모 패턴을 자동으로 분석하고 신뢰할 수 있는 데이터를 제공합니다.
- **고객 신뢰도 향상:** 투명한 점검 리포트 생성 및 고객 서명, 손쉬운 이메일/출력 공유를 통해 정비소와 고객 간의 신뢰를 확고히 구축합니다.

## 2. 비즈니스 로직 및 핵심 도메인 (Business Logic)
- **역할 기반 접근 제어 (RBAC):** 사용자는 '정비사(Mechanic)' 또는 '오너(Owner)' 등의 역할을 가집니다. 오너는 팀 관리 기능을 추가로 이용할 수 있습니다.
- **문서 및 이미지 처리:** 모바일 기기의 카메라를 이용해 문서를 스캔하고, OCR을 통해 텍스트를 추출하며, 차량 부품 이미지를 캡처해 서버로 안전하게 전송합니다.
- **AI 분석 파이프라인:** 전송된 이미지는 서버 측 AI 모델을 거쳐 결함 상태(Defects, Wear patterns) 및 차량 컴포넌트 건강 점수(Health Score)로 분석되어 반환됩니다.
- **데이터 불변성 및 로컬 캐싱:** `Freezed`를 사용해 앱 내 상태 데이터를 불변(Immutable)으로 관리하며, `Hive`를 통해 인증 토큰 및 일부 로컬 데이터를 캐싱해 오프라인 환경에 대비합니다.

## 3. 상세 기능 설명 (Feature Descriptions)

### 3.1. 인증 및 사용자 관리 (Auth)
- **SMS 본인 인증:** 전화번호와 SMS 인증 코드를 통한 간편하고 안전한 로그인 및 회원가입 체계 구축.
- **마이페이지 (My Page):** 사용자 프로필 관리 및 권한(정비사/오너)에 따른 맞춤 설정 제공. 오너의 경우 팀원 관리 기능 접근 가능.

### 3.2. 정비사 대시보드 (Mechanic Dashboard)
- **작업 목록 관리:** 당일 할당된 점검 차량 목록 및 상태(진행 중, 완료 등) 한눈에 파악.
- **신규 점검 생성:** 번호판 스캔(Plate Number OCR Scan)을 통해 즉시 신규 차량 점검 세션 시작 가능.

### 3.3. 점검 상세 입력 (Inspection Details)
- **부품별 상태 평가:** 엔진 오일(Engine Oil), 에어 필터(Air Filter), 드라이브 벨트(Drive Belt), 냉각수 호스(Coolant Hoses) 등 주요 부품 상태를 'Good', 'Rec(권장)', 'Bad'로 분류하여 평가.
- **사진 및 메모 첨부:** 문제가 있는 부품에 대해 증거 사진을 촬영하고 상세 정비 메모를 추가하여 기록 상세화.

### 3.4. AI 점검 결과 (AI Inspection Results)
- **AI 분석 진행 상태 모니터링:** 시각적 데이터를 기반으로 한 AI 분석 진행률(Progress) 실시간 표시.
- **종합 점수 및 컴포넌트 상태:** 차량의 전반적인 건강 점수(Health Score)와 개별 부품별 위험/주의 상태를 직관적인 차트 및 지표로 시각화.

### 3.5. 서명 및 리포트 내보내기 (Signatures & Export)
- **디지털 서명 수집:** 점검 완료 시 정비사(Inspector) 서명과 고객(Customer) 서명 입력란 제공.
- **리포트 공유 및 저장:** 완성된 점검 리포트를 Print(출력) 하거나 Email(이메일)로 전송. 외부 시스템으로 PDF 형식으로 Export 지원.

## 4. 사용성 플로우 (Usability Flow)
1. **[로그인]** 앱 실행 후 SMS 인증을 통해 시스템에 로그인합니다.
2. **[대시보드 진입]** 정비사 대시보드에서 점검할 차량을 선택하거나 새 차량의 번호판을 스캔하여 등록합니다.
3. **[점검 수행]** 차량의 후드(Under Hood)를 열고 각 부품을 점검하며 상태를 체크하고, 필요 시 현장 사진과 메모를 남깁니다.
4. **[AI 분석]** 수집된 데이터 및 이미지를 기반으로 AI 시각 검사(AI Inspection) 및 OCR 스캔을 요청합니다.
5. **[결과 확인]** AI가 도출한 결함 및 마모 패턴 분석 결과(건강 점수 등)를 검토합니다.
6. **[최종 요약]** 종합 점검 요약(General Inspection Notes)을 작성합니다.
7. **[서명 및 공유]** 정비사 서명을 기입하고 고객에게 결과를 설명한 후 고객 서명을 받습니다. 이후 최종 리포트를 이메일이나 출력본으로 공유(Export)합니다.

## 5. 소프트웨어 설계 및 아키텍처 원칙 (Software Design Principles)

### 5.1. Feature-first Clean Architecture
- 코드는 기능(Feature)별로 분리(`lib/features/`)되며, 각 기능 내에서 `data`, `domain`, `presentation` 레이어로 나뉩니다.
- **Domain:** 비즈니스 로직, Entity, Repository 인터페이스. (프레임워크 독립적 설계)
- **Data:** API 통신(Dio), 로컬 DB(Hive), Repository 구현체 및 DTO.
- **Presentation:** UI(Widgets) 화면 구성 및 상태 관리(Riverpod).

### 5.2. UI 및 상태 관리 (Presentation Layer)
- **Atomic Design:** `design_system` 폴더 내에 Foundations, Atoms, Molecules, Organisms 구조로 UI 컴포넌트를 설계하여 UI 재사용성과 일관성을 극대화합니다.
- **Riverpod 상태 관리:** UI와 비즈니스 로직을 완벽히 분리하며, ViewModel(Notifier) 내부에서는 `BuildContext` 사용을 엄격히 금지합니다.
- **GoRouter 기반 타입 세이프 라우팅:** `go_router_builder`를 사용하여 모든 화면 이동 및 파라미터 전달을 타입 안전(Type-safe)하게 처리합니다.

### 5.3. 에러 처리 및 데이터 모델링
- **함수형 에러 핸들링:** 예외(Exception)를 throw하는 대신, `fpdart`의 `Either<Failure, T>`를 반환하여 에러 처리를 명시적으로 강제합니다.
- **불변 데이터:** 모든 데이터 클래스(Entity, DTO, State)는 `Freezed` 패키지를 사용하여 불변(Immutable) 객체로 구성합니다. `dynamic` 타입 사용은 철저히 금지됩니다.

## 6. 에이전트 및 개발자 행동 지침 (Agent/Developer Rules)
- 본 앱의 코드를 수정하거나 기능을 추가할 때, **`README.md`와 이 `AGENTS.md` 파일에 명시된 아키텍처 및 코딩 규칙을 최우선으로 준수**해야 합니다.
- **모든 명령어는 `fvm`을 통해 실행**합니다. (예: `fvm flutter run`, `fvm dart run build_runner build --delete-conflicting-outputs`)
- 코드 수정 후에는 반드시 컴파일 및 빌드 러너를 실행하여 생성된 코드의 정합성을 확인하고 에러가 없음을 검증해야 합니다.
