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

##  전체 프로세스 워크플로우 (Overall Workflow)

이 시스템은 4가지 역할이 유기적으로 연결되어 작동합니다.

1. **PM (기획 및 관리) - [Main PC]**
   - **역할:** 사용자 요구사항 분석, GitHub Issue 생성, 전체 진행 상황 조율.
   - **도구:** `github` 서버 (MCP)를 통해 이슈 및 프로젝트 보드 관리.

2. **Developer (개발) - [Sub PC 1]**
   - **역할:** 실제 코드를 작성하고 수정.
   - **도구:** `coder_pc` 서버 (MCP/SSH)를 통해 원격 파일 시스템에 접근하여 코드 수정 및 Push.

3. **QA/Tester (테스트) - [Sub PC 2]**
   - **역할:** 작성된 코드를 실행하고 테스트.
   - **도구:** `qa_pc` 서버 (MCP/SSH)를 통해 터미널 명령어를 실행하여 테스트 수행 및 로그 확인.

4. **Reviewer (검토) - [Main PC]**
   - **역할:** 코드 리뷰 및 PR 승인.
   - **도구:** PR 내용을 분석하고 최종 Merge 결정.

##  기술적 연결 구조 (Technical Architecture)

`mcp_config.json`이 이 구조의 핵심 인프라(허브) 역할을 합니다.
**Main PC(Antigravity)**는 이 설정 파일을 통해 다른 에이전트(PC)들에게 명령을 내립니다.

- **`coder_pc` 정의:** SSH를 통해 개발 PC의 파일 시스템 제어 권한 획득.
- **`qa_pc` 정의:** SSH를 통해 테스트 PC의 터미널 제어 권한 획득.
- **`github` 정의:** GitHub API를 통해 협업 및 이슈 관리.
