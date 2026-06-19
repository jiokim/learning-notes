# Claude Certified Architect

**출처:** [Claude Certified Architect – Foundations Certification Exam Guide](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F8lsy243ftffjjy1cx9lm3o2bw%2Fpublic%2F1773274827%2FClaude+Certified+Architect+%E2%80%93+Foundations+Certification+Exam+Guide.pdf) (Anthropic, PBC · Confidential NTK, v0.1)  
**목적:** 커리큘럼을 가이드로 삼아 AI/Claude 개념을 학습  
**상태:** 진행 중

## 도메인 (5개)

| 도메인 | 비중 | 핵심 주제 |
|---|---|---|
| [D1. Agentic Architecture & Orchestration](domain-01-agentic-architecture.md) | 27% | 에이전트 루프(observe→think→act→feedback), 멀티 에이전트 오케스트레이션 패턴, 서브 에이전트, 부분 장애 시 폴백 |
| [D2. Tool Design & MCP Integration](domain-02-tool-design-mcp.md) | 18% | 도구 description 설계, 구조화된 에러 응답, MCP 서버 구성(tools/resources/prompts) |
| [D3. Claude Code Configuration & Workflows](domain-03-claude-code-workflows.md) | 20% | CLAUDE.md 계층 구성, `.claude/rules/`, 커스텀 스킬, Plan 모드, CI/CD 통합 |
| [D4. Prompt Engineering & Structured Output](domain-04-prompt-engineering.md) | 20% | 프롬프트 설계 원칙, tool use + JSON Schema 구조화 출력, Message Batches API |
| [D5. Context Management & Reliability](domain-05-context-management.md) | 15% | 컨텍스트 우선순위, 에스컬레이션 설계, 에러 전파 |

## 학습 일정 (도메인당 1주)

- [ ] Week 1 — D1. Agentic Architecture & Orchestration
- [ ] Week 2 — D2. Tool Design & MCP Integration
- [ ] Week 3 — D3. Claude Code Configuration & Workflows
- [ ] Week 4 — D4. Prompt Engineering & Structured Output
- [ ] Week 5 — D5. Context Management & Reliability

## Anthropic Academy 코스 ↔ 도메인 매핑

(anthropic.skilljar.com, 무료)

| 코스 | 도메인 |
|---|---|
| [Claude 101](https://anthropic.skilljar.com/claude-101) | D4, D5 |
| [AI Fluency: Framework & Foundations](https://anthropic.skilljar.com/ai-fluency-framework-foundations) | D5 |
| [AI Capabilities and Limitations](https://anthropic.skilljar.com/ai-capabilities-and-limitations) | D5 |
| [Building with the Claude API](https://anthropic.skilljar.com/claude-with-the-anthropic-api) | D2, D4 |
| [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action) | D3 |
| [Introduction to Agent Skills](https://anthropic.skilljar.com/introduction-to-agent-skills) | D3 |
| [Introduction to Subagents](https://anthropic.skilljar.com/introduction-to-subagents) | D1 |
| [Introduction to MCP](https://anthropic.skilljar.com/introduction-to-model-context-protocol) | D2 |
| [MCP: Advanced Topics](https://anthropic.skilljar.com/model-context-protocol-advanced-topics) | D2 |
| [Introduction to Claude Cowork](https://anthropic.skilljar.com/introduction-to-claude-cowork) | D3 |
| [Claude with Amazon Bedrock](https://anthropic.skilljar.com/claude-in-amazon-bedrock) | D2 |
| [Claude with Google Vertex AI](https://anthropic.skilljar.com/claude-with-google-vertex) | D2 |

## 참고 자료

- [Claude 공식 문서](https://docs.anthropic.com/)
- [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook)
- [MCP 공식 명세](https://modelcontextprotocol.io/)
- [Anthropic Academy](https://anthropic.skilljar.com/)