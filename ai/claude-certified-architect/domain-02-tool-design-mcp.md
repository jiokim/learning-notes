# D2. Tool Design & MCP Integration (18%)

## 핵심 개념

### 2.1 명확한 description과 경계를 가진 도구 인터페이스 설계
- 도구 description은 LLM이 어떤 도구를 쓸지 판단하는 주된 근거 — 최소한의 description은 비슷한 도구 사이의 선택을 불안정하게 만듦
- 입력 형식, 예시 쿼리, 엣지 케이스, 경계 설명을 description에 포함해야 신뢰도가 올라감
- 모호하거나 중복된 description(예: `analyze_content` vs `analyze_document`)은 미라우팅을 유발
- 시스템 프롬프트의 키워드 민감 지시문이 잘 쓰인 description을 무시하고 의도치 않은 도구 연결을 만들 수 있음
- 해법: 기능이 겹치는 도구는 이름·description을 명확히 구분(`extract_web_results` 등으로 specific화)하거나, 범용 도구를 입출력 계약이 분명한 여러 전용 도구로 분리

### 2.2 MCP 도구의 구조화된 에러 응답
- MCP `isError` 플래그로 도구 실패를 에이전트에 전달
- 에러 종류 구분: transient(타임아웃·서비스 불가) / validation(잘못된 입력) / business(정책 위반) / permission
- `errorCategory`, `isRetryable` boolean, 사람이 읽을 수 있는 설명을 구조화된 메타데이터로 반환해야 적절한 복구 판단이 가능
- 균일한 "Operation failed" 같은 응답은 에이전트가 복구 결정을 내릴 정보를 빼앗음
- 서브에이전트는 transient 실패를 자체적으로 복구 시도하고, 해결 불가한 에러만 부분 결과·시도 내역과 함께 코디네이터에 전파
- 접근 실패(재시도 판단 필요)와 정상적인 빈 결과(성공)는 반드시 구분

### 2.3 에이전트별 도구 분배와 tool_choice 설정
- 너무 많은 도구(예: 4~5개 대신 18개)를 주면 선택 복잡도가 늘어 도구 선택 신뢰도가 떨어짐
- 전문 분야 밖 도구를 가진 에이전트는 그 도구를 오용하는 경향(예: synthesis 에이전트가 웹 검색을 시도)
- Scoped tool access: 각 에이전트에게 역할에 필요한 도구만, 고빈도 요구가 있으면 제한적 cross-role 도구를 추가
- `tool_choice` 옵션: `"auto"`(텍스트 반환도 가능) / `"any"`(반드시 도구 호출, 어떤 도구든) / 특정 도구 강제 지정(`{"type": "tool", "name": "..."}`)

### 2.4 MCP 서버를 Claude Code/에이전트 워크플로우에 통합
- MCP 서버 스코프: 프로젝트 레벨(`.mcp.json`, 팀 공유 도구) vs 사용자 레벨(`~/.claude.json`, 개인/실험용 서버)
- `.mcp.json`의 환경변수 확장(예: `${GITHUB_TOKEN}`)으로 시크릿을 커밋하지 않고 인증 관리
- 연결 시점에 구성된 모든 MCP 서버의 도구가 동시에 discover되어 에이전트에 노출됨
- MCP resources로 콘텐츠 카탈로그(이슈 요약, 문서 구조, DB 스키마)를 노출하면 탐색적 도구 호출을 줄일 수 있음
- 표준 통합(예: Jira)은 커스텀 서버보다 기존 커뮤니티 MCP 서버를 우선 고려, 커스텀 서버는 팀 특화 워크플로우에만

### 2.5 빌트인 도구(Read/Write/Edit/Bash/Grep/Glob) 선택과 활용
- Grep: 함수명·에러 메시지·import 등 콘텐츠 검색 / Glob: 파일명·확장자 패턴으로 경로 매칭
- Read/Write는 전체 파일 작업, Edit는 고유한 텍스트 매칭 기반의 부분 수정
- Edit가 비고유 매칭으로 실패하면 Read + Write를 대체 수단으로 사용
- 코드베이스 이해는 Grep으로 진입점을 찾고 Read로 import/흐름을 추적하는 식으로 점진적으로 — 처음부터 전체를 다 읽지 않음
