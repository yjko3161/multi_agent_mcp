# 🤖 Multi-Agent Distributed Coding Team Setup Guide
> **윈도우 PC 4대와 Antigravity, MCP를 활용한 분산형 AI 개발 팀 구축 가이드**

이 프로젝트는 다중 에이전트(Multi-Agent)가 물리적으로 분리된 여러 대의 PC에서 협업하며, 깃허브(GitHub)를 통해 작업을 관리하고 코드를 검수하는 환경을 구축하는 것을 목표로 합니다.

## 🏗 시스템 아키텍처 (4인 팀 구성)

| 에이전트 | 역할 (Role) | 주요 환경 | 주요 작업 |
| :--- | :--- | :--- | :--- |
| **PM** | 기획 및 관리 | 메인 PC (Antigravity) | GitHub Issues 생성 및 전체 일정 관리 |
| **Developer** | 코드 구현 | 서브 PC 1 | 로컬 파일 시스템 제어 및 코드 작성(Push) |
| **QA/Tester** | 실행 및 검증 | 서브 PC 2 | 터미널 제어, 코드 실행 및 PR 테스트 결과 보고 |
| **Reviewer** | 최종 검수 | 메인 PC / Cloud | PR 코드 리뷰, 성능 점검 및 최종 Merge 승인 |

## 🛠 필수 도구 세팅 (윈도우 공통)
- **Git & Node.js & Python:** 모든 PC에 설치 및 환경 변수(PATH) 등록 필수.
- **OpenSSH Server:** 서브 PC들의 제어를 위해 윈도우 기능에서 활성화.
- **Tailscale (선택):** 여러 대의 PC를 하나의 가상 로컬 네트워크로 안전하게 통합.

## 🚀 환경 구축 단계

### 1. GitHub 권한 설정
- [GitHub Personal Access Token (PAT)](https://github.com/settings/tokens) 발급.
- 권한: `Contents`, `Issues`, `Pull Requests` (Read & Write).

### 2. Antigravity MCP 구성 (`mcp_config.json`)
메인 PC의 `%AppData%\Antigravity\config\mcp_config.json`에 아래와 같이 서브 PC들의 자원을 등록합니다.

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "YOUR_TOKEN_HERE" }
    },
    "coder_pc": {
      "command": "ssh",
      "args": ["user@SubPC1_IP", "npx", "-y", "@modelcontextprotocol/server-filesystem", "C:/Workspace/Project"]
    },
    "qa_pc": {
      "command": "ssh",
      "args": ["user@SubPC2_IP", "npx", "-y", "@modelcontextprotocol/server-terminal"]
    }
  }
}
