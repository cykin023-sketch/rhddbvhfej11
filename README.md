sequenceDiagram
    autonumber
    
    actor User as CRM 사용자
    participant CRM_WEB as CRM 프론트엔드
    participant CRM_AUTH as CRM 인증 서버<br/>(Authorization Server)
    participant REGISTRY as 등기 정보 사이트<br/>(OAuth2 Client)
    participant CRM_API as CRM 리소스 서버<br/>(REST API)

    %% 1단계: OIDC 기반 SSO (사용자 식별 및 화면 제공)
    rect rgb(240, 248, 255)
    Note right of User: [Phase 1] OIDC 기반 로그인 및 지도 팝업
    User->>CRM_WEB: CRM 사이트 로그인
    User->>CRM_WEB: 등기 정보 연동 버튼 클릭
    CRM_WEB->>REGISTRY: 새 탭 이동 (OIDC Login 요청 트리거)
    REGISTRY->>CRM_AUTH: Authorization Code 요청 (scope: openid)
    CRM_AUTH-->>REGISTRY: Authorization Code 반환
    REGISTRY->>CRM_AUTH: Code로 Access/ID Token 교환 요청
    CRM_AUTH-->>REGISTRY: ID Token (사용자 정보) 및 Access Token 발급
    REGISTRY-->>User: 해당 사용자 맞춤 등기 정보 지도 UI 렌더링 (새 탭)
    end

    %% 2단계: 사용자 액션 및 서버 간 토큰 발급
    User->>REGISTRY: 특정 건물 정보 선택 후 '전송' 클릭
    
    rect rgb(255, 240, 245)
    Note right of User: [Phase 2] Client Credentials Flow (서버 대 서버)
    REGISTRY->>CRM_AUTH: API 접근용 토큰 요청<br/>(grant_type=client_credentials, client_id/secret)
    CRM_AUTH-->>REGISTRY: Access Token 발급 (JWT, scope: building:write)
    end

    %% 3단계: 리소스 서버 API 호출 및 비즈니스 로직 처리
    rect rgb(240, 255, 240)
    Note right of User: [Phase 3] 건물 정보 전송 및 CRM 비즈니스 로직
    REGISTRY->>CRM_API: 건물 정보 POST 전송<br/>(Header - Authorization: Bearer <JWT>)
    CRM_API->>CRM_API: JWT 서명, 만료(exp), 발급자(iss), 권한(scope) 자체 검증
    CRM_API->>CRM_API: 수신된 건물 정보 DB 저장 및 기존 고객 정보와 매핑
    CRM_API-->>REGISTRY: 200 OK (전송 성공)
    end
    
    REGISTRY-->>User: 전송 완료 알림창 표시
    User->>CRM_WEB: 기존 CRM 탭으로 돌아가서 화면 확인
    CRM_WEB-->>User: 고객 정보와 조합된 건물 정보 노출
