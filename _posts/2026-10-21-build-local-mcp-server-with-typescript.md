---
title: "TypeScript로 30분 만에 만드는 로컬 MCP 서버"
description: "Model Context Protocol의 v1/v2 패키지 구조 차이를 분석하고, 공식 TypeScript SDK v1.32 기반으로 Claude 연동 로컬 MCP 서버를 직접 구현합니다."
date: 2026-10-21 21:00:00 +0900
categories: [AI & Tech, Claude]
tags: [Dev Log, Web Dev]
image:
  path: /assets/img/posts/2026-10-21-build-local-mcp-server-with-typescript/cover.jpg
  alt: "TypeScript 기반 로컬 Model Context Protocol 서버 구축 가이드"
published: true
---

코딩 에이전트에게 내 로컬 프로젝트의 깃 히스토리나 로컬 DB를 안전하게 읽히려면 어떻게 해야 할까. 매번 터미널 명령어를 일일이 프롬프트로 복사해 붙여넣는 방식은 작업 호흡을 끊어먹는다.

해결책은 앤트로픽(Anthropic)이 오픈소스로 공개한 모델 컨텍스트 프로토콜(Model Context Protocol, MCP)이다. MCP는 AI 모델과 로컬 개발 환경(파일, Git, SQLite, API)을 표준화된 JSON-RPC 2.0 프로토콜로 연결하는 개방형 규격이다. 

다만 최근 MCP 생태계는 모놀리식 구조인 기존 **v1 SDK(`@modelcontextprotocol/sdk`)**와 2026년 공개된 모듈형 **v2 규격(`@modelcontextprotocol/server`)**이 공존하고 있어, 패키지 버전과 임포트 경로를 혼동하기 쉽다.

이번 글에서는 MCP SDK v1과 v2의 구조적 차이를 짚고, 현재 로컬 Claude Desktop 및 Claude Code와 가장 안정적으로 호환되는 **공식 TypeScript SDK v1.32 기준**으로 최근 커밋 내역을 조회하는 `git-inspector` MCP 서버를 직접 구현하는 과정을 정리했다.

---

## 1. MCP SDK v1과 v2의 구조적 차이

MCP를 시작할 때 가장 먼저 겪는 혼란은 패키지 이름과 아키텍처의 분기다.

| 비교 항목 | MCP SDK v1 (`@modelcontextprotocol/sdk`) | MCP SDK v2 (`@modelcontextprotocol/server`) |
|---|---|---|
| **패키지 구조** | 단일 모놀리식 패키지 (`1.32.x` 안정화 브랜치) | 기능별 모듈 분할 (`@modelcontextprotocol/server`, `client`) |
| **통신 기본값** | 로컬 프로세스 표준 입출력(`stdio`) 중심 | 무상태(Stateless) HTTP 및 서버리스 웹 중심 |
| **서버 API** | `Server` 클래스 + `setRequestHandler` | 고수준 `McpServer` 빌더 + 프레임워크 미들웨어 |
| **데스크톱 호환성** | **Claude Desktop / Claude Code 로컬 기본 지원** | Cloudflare Workers / Hono 등 클라우드 배포에 특화 |

클라우드 기반 서버리스 배포에는 v2 모듈형 구조가 유리하지만, 로컬 컴퓨터에서 Claude Desktop의 자식 프로세스로 띄워 Git 저장소나 로컬 파일을 읽을 때는 기존 **v1 SDK(`@modelcontextprotocol/sdk@1.x`)**가 가장 안정적으로 검증되어 있다. 이번 실습에서도 이 v1.32 릴리스를 기준으로 작성한다.

---

## 2. MCP가 작동하는 기본 구조

기존의 LLM 도구 연동은 모델 제공자마다 제각각이었다. OpenAI의 Function Calling, LangChain의 Tool 정의, Claude의 Tool Use 문법이 모두 달랐다. MCP는 이 파편화를 표준 프로토콜 하나로 정리했다.

![Model Context Protocol 아키텍처 및 stdio 데이터 흐름](/assets/img/posts/2026-10-21-build-local-mcp-server-with-typescript/01-mcp-architecture-stdio-flow.png)

MCP의 핵심 삼각 구조는 다음과 같다.

1. **Host**: Claude Desktop 앱이나 Claude Code CLI처럼 사용자가 대화하는 프론트엔드 환경이다.
2. **Client**: 호스트 내부에서 MCP 서버와의 연결 수명주기를 관리하고 툴을 호출하는 클라이언트 모듈이다.
3. **Server**: 개발자가 작성한 경량 프로세스다. 표준 입출력(`stdio`)을 통해 클라이언트와 통신하며 실제 도구(Tool)와 리소스(Resource)를 제공한다.

로컬 개발 환경에서는 별도의 웹 서버나 포트 포워딩이 필요 없는 `stdio` 트랜스포트가 기본이다. 호스트가 자식 프로세스로 서버를 실행하고, 표준 입력(`stdin`)과 표준 출력(`stdout`)으로 JSON-RPC 메시지를 주고받는다. 네트워크 포트 충돌이나 방화벽 걱정이 아예 없다.

---

## 3. 프로젝트 초기화 및 필수 의존성 설치

Node.js 18 이상 환경에서 새 디렉터리를 만들고 TypeScript 개발 환경을 세팅한다. 설치 시 패키지 이름을 정확히 명시한다.

```bash
mkdir local-git-mcp
cd local-git-mcp
npm init -y

# 공식 v1.x 안정화 패키지 및 Zod 설치
npm install @modelcontextprotocol/sdk@^1.32.0 zod
npm install -D typescript @types/node tsx
npx tsc --init
```

`package.json`에 ES 모듈(`"type": "module"`)과 빌드 스크립트를 추가한다.

```json
{
  "name": "local-git-mcp",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "local-git-mcp": "./dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "inspect": "npx @modelcontextprotocol/inspector node dist/index.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.32.0",
    "zod": "^3.23.8"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "tsx": "^4.19.0",
    "typescript": "^5.6.0"
  }
}
```

설치 명령을 실행하면 npm은 v1.x 최신 안정 패치인 `1.32.1` 버전을 `package-lock.json`에 일관되게 기록한다. 설치 명령어, `package.json`, lockfile이 모두 v1.32.x 안정 브랜치에 고정되므로 환경에 따른 SDK 버전 불일치가 방지된다.

`tsconfig.json`에서는 NodeNext 모듈 해석 방식과 Node.js 내장 모듈 타입 정의를 지정한다.

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "types": ["node"]
  },
  "include": ["src/**/*"]
}
```

---

## 4. MCP 서버 코드 작성: git-inspector

`src/index.ts` 파일을 생성하고 서버를 작성한다. 임포트 경로는 v1.x 규격에 따라 `@modelcontextprotocol/sdk/server/index.js`와 `@modelcontextprotocol/sdk/server/stdio.js`를 사용한다. Git 명령어를 실행할 때는 셸 인젝션 위험이 있는 `exec` 대신 인자를 배열로 분리하는 `execFile`을 사용한다.

```typescript
#!/usr/bin/env node
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import { execFile } from "node:child_process";
import { promisify } from "node:util";
import { z } from "zod";

const execFileAsync = promisify(execFile);

// 1. MCP 서버 인스턴스 초기화 (v1.x Server 클래스)
const server = new Server(
  {
    name: "local-git-mcp",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// 2. 도구 입력 스키마 정의 (Zod)
const GetRecentCommitsSchema = z.object({
  count: z.number().int().min(1).max(20).default(5),
  cwd: z.string().optional(),
});

// 3. 제공 가능한 도구 목록 노출 (tools/list)
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "get_recent_commits",
        description: "현재 Git 저장소의 최근 커밋 해시와 메시지, 작성자 정보를 가져온다.",
        inputSchema: {
          type: "object",
          properties: {
            count: {
              type: "number",
              description: "조회할 커밋 개수 (1~20, 기본값: 5)",
              default: 5,
            },
            cwd: {
              type: "string",
              description: "Git 저장소 작업 경로 (생략 시 현재 작업 디렉터리)",
            },
          },
        },
      },
      {
        name: "get_repo_status",
        description: "Git 저장소의 현재 브랜치와 변경/스테이징 상태를 조회한다.",
        inputSchema: {
          type: "object",
          properties: {
            cwd: {
              type: "string",
              description: "Git 저장소 작업 경로",
            },
          },
        },
      },
    ],
  };
});

// 4. 도구 실행 핸들러 구현 (tools/call)
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;
  const targetDir = (args?.cwd as string) || process.cwd();

  try {
    if (name === "get_recent_commits") {
      const parsed = GetRecentCommitsSchema.parse(args || {});
      const { stdout } = await execFileAsync(
        "git",
        ["log", `-${parsed.count}`, "--oneline", "--decorate"],
        { cwd: targetDir }
      );

      return {
        content: [
          {
            type: "text",
            text: stdout.trim() || "커밋 내역이 없습니다.",
          },
        ],
      };
    }

    if (name === "get_repo_status") {
      const { stdout } = await execFileAsync(
        "git",
        ["status", "--short", "--branch"],
        { cwd: targetDir }
      );

      return {
        content: [
          {
            type: "text",
            text: stdout.trim() || "작업 트리가 깨끗합니다.",
          },
        ],
      };
    }

    throw new Error(`알 수 없는 도구 요청: ${name}`);
  } catch (error: any) {
    return {
      isError: true,
      content: [
        {
          type: "text",
          text: `Git 실행 실패: ${error.message}`,
        },
      ],
    };
  }
});

// 5. Stdio 트랜스포트 연결
async function run() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Local Git MCP Server running on stdio");
}

run().catch((error) => {
  console.error("Fatal error running MCP server:", error);
  process.exit(1);
});
```

코드가 간결하다. 복잡한 네트워크 통신 없이 `ListToolsRequestSchema`로 도구의 이름과 스키마를 선언하고, `CallToolRequestSchema`에서 실제 로직을 분기해 결과 텍스트를 돌려주면 끝이다.

---

## 5. MCP Inspector로 실시간 디버깅

서버를 작성한 뒤 Claude 데스크톱에 바로 붙이기보다, 앤트로픽이 제공하는 웹 기반 디버거인 `@modelcontextprotocol/inspector`로 먼저 검증하는 것이 훨씬 빠르다.

```bash
npm run build
npm run inspect
```

명령어를 실행하면 로컬 웹 브라우저(`http://localhost:5173` 또는 자동 할당 포트)가 열리며 인터랙티브 디버깅 콘솔이 뜬다.

![Model Context Protocol Inspector 실행 화면](/assets/img/posts/2026-10-21-build-local-mcp-server-with-typescript/02-mcp-inspector-tool-execution.png)

*(위 이미지는 `@modelcontextprotocol/inspector`의 인터페이스 레이아웃과 도구 호출 데이터 구조를 독자가 직관적으로 이해할 수 있도록 재구성한 아키텍처 다이어그램입니다).*

인스펙터 화면에서는 세 가지를 즉시 확인할 수 있다.

1. **Connected Tools**: 서버가 노출한 `get_recent_commits`와 `get_repo_status`가 정상적으로 목록에 나타나는지 확인한다.
2. **Interactive Run**: 매개변수 `{ "count": 3 }`을 입력하고 `Execute Tool` 버튼을 누르면 Git 커밋 로그가 즉시 반환된다.
3. **JSON-RPC Traffic**: 하단 로그 창에서 클라이언트와 서버가 주고받은 원본 JSON-RPC 페이로드를 실시간으로 모니터링할 수 있다.

---

## 6. Claude Desktop 설정 파일에 등록하기

인스펙터 검증이 끝났다면 운영 환경에 붙여본다. Claude Desktop 설정 파일 경로는 운영체제별로 다음과 같다.

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

설정 파일의 `mcpServers` 항목에 서버 실행 명령과 절대 경로를 지정한다.

```json
{
  "mcpServers": {
    "local-git": {
      "command": "node",
      "args": [
        "C:/projects/local-git-mcp/dist/index.js"
      ]
    }
  }
}
```

설정 후 Claude Desktop을 완전히 재시작하면 채팅 입력창 우측 하단에 망치 모양의 툴 아이콘이 활성화된다. "최근 작업한 커밋 3개만 요약해줘"라고 입력하면 Claude가 백그라운드에서 `local-git` 서버의 `get_recent_commits` 도구를 호출해 커밋 목록을 직접 읽고 분석 결과를 출력한다.

---

## 7. 개발자가 반드시 기억해야 할 3대 함정

로컬 MCP 서버를 개발할 때 흔히 마주치는 삽질 포인트가 있다.

### 1) stdio에서 `console.log()`는 금기다

서버가 표준 출력(`stdout`)을 통해 JSON-RPC 메시지를 전송하고 있을 때, 무심코 코드 중간에 `console.log("debug data")`를 넣으면 JSON 파서가 깨지며 통신이 영구 중단된다. 모든 디버깅 출력은 반드시 표준 에러(`console.error()`)로 보내야 한다.

### 2) 비동기 예외 누수와 좀비 프로세스

자식 프로세스(`git` CLI)가 실패했을 때 예외를 잡지 않고 그대로 던지면 Node.js 프로세스가 비정상 종료된다. `CallToolRequestSchema` 핸들러 내부에서 반드시 `try/catch`로 에러를 감싸고 `isError: true` 플래그와 함께 구조화된 메시지를 반환해야 호스트가 다운되지 않는다.

### 3) 셸 명령어 인젝션 방지

`exec("git log " + args.count)` 형태로 문자열을 직접 결합하면 악의적인 인자가 주입되었을 때 시스템 명령어가 실행될 위험이 있다. 반드시 `execFile`을 사용해 인자를 배열로 격리하고 Zod로 데이터 타입을 강제해야 안전하다.

---

## 8. 정리: 언제 스킬을 쓰고 언제 MCP를 써야 할까?

Claude 생태계에는 프로젝트 스킬(Skills)과 MCP 서버가 공존한다. 무엇을 골라야 할지 헷갈린다면 다음 기준으로 판단하면 된다.

| 비교 항목 | Claude Code Custom Skill | Model Context Protocol (MCP) |
|---|---|---|
| **동작 방식** | 마크다운(`SKILL.md`) 기반 컨텍스트 로딩 | 독립 Node/Python 프로세스 (JSON-RPC) |
| **적합한 작업** | 코딩 컨벤션, 작업 가이드, 가벼운 셸 스크립트 | 데이터베이스 조회, 외부 API 연동, 복잡한 로컬 도구 |
| **재사용성** | 프로젝트 저장소 내부 종속 | Claude Desktop, Claude Code, Cursor 전반 공유 |
| **구현 비용** | 파일 1개 작성으로 즉시 완료 | 패키지 빌드 및 프로세스 관리 필요 |

단순한 가이드나 프롬프트 규칙은 `SKILL.md`로 끝내고, 로컬 Git·데이터베이스·하드웨어처럼 실제 런타임 시스템과 연동해야 하는 작업은 MCP 서버로 분리하는 것이 가장 깔끔한 아키텍처다.
