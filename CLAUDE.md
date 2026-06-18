# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

`@ai-clash/inject` is a lightweight DOM automation library that injects into AI chat platform pages (DeepSeek, 豆包, 通义千问, Qwen, 元宝, 文心一言, MiMo/LongCat) to programmatically control them — fill prompts, send messages, toggle thinking/search modes, and capture streaming responses via SSE interception.

## Build & dev commands

```bash
npm run build              # Full build: ESM + UMD lib + standalone IIFE
npm run build:esm          # Library only (ESM + UMD)
npm run build:standalone   # Standalone IIFE only
npm run dev                # Watch standalone + serve on :5173 (requires concurrently, http-server)
npm run dev:build-only     # Watch standalone only (no server)
npm run serve              # Serve dist/ on :5173
npm run typecheck          # tsc --noEmit
```

There are no tests yet.

## Architecture

```
src/
├── index.ts                  # Public API barrel export
├── core/
│   ├── types.ts              # All type definitions (ProviderConfig, Capabilities, InjectorOptions, etc.)
│   ├── injector.ts           # Main injector: createInjector() + SSE interception + adapter setup
│   ├── dom-utils.ts          # DOM helpers: findElement, waitForElement, simulateRealClick, fuzzy class selectors
│   └── incremental-utils.ts  # Full→incremental text extraction (IncrementalHelper) for APIs that return complete content each time
├── providers/
│   ├── index.ts              # PROVIDERS registry + getProviderConfig() / getProviderIds()
│   ├── deepseek.ts           # chat.deepseek.com
│   ├── doubao.ts             # doubao.com
│   ├── qianwen.ts            # qianwen.com (通义千问)
│   ├── qwen.ts               # chat.qwen.ai (international)
│   ├── longcat.ts            # longcat.chat / tiangong.cn
│   ├── yuanbao.ts            # yuanbao.tencent.com
│   ├── wenxin.ts             # yiyan.baidu.com
│   └── mimo.ts               # aistudio.xiaomimimo.com (Xiaomi MiMo)
└── standalone/
    └── entry.ts              # Auto-inject entry: detects provider from domain, exposes window.__AI_CLASH
```

### Key concepts

**Provider = a config object** (`ProviderConfig` in `types.ts`) describing one AI platform: DOM selectors for chat input/send/new-chat buttons, toggle actions for thinking/search modes, SSE URL pattern + `parseLine` function, auth/login detection, and conversation ID extraction.

**Injector = `createInjector(options)`** — the main factory. Given a provider ID and adapter type, it builds `Capabilities` (chat, auth, thinking, search) and wires them to the chosen adapter. Call `.inject()` to activate, `.eject()` to clean up.

**Adapters** decouple capability execution from the communication channel:
- `window` — exposes `window.__AI_CLASH` with direct method access + CustomEvent RPC
- `extension` — Chrome `chrome.runtime.onMessage` listener
- `ws` — WebSocket client with JSON-RPC
- `broadcast` — BroadcastChannel messaging

**SSE interception** is four-pronged (in `injector.ts`): monkey-patches `fetch`, `XMLHttpRequest`, `TextDecoder.prototype.decode`, and `ReadableStream.prototype.getReader` to capture streaming chat responses without relying on any single transport. Each provider's `sse.parseLine` receives raw SSE data lines and returns `{text, isThink, done}` chunks.

**DOM selectors** support a custom `>>` pseudo-syntax for text-content matching (e.g., `.btn >> 深度思考` finds an element with class `btn` containing that text), and `*` wildcard class selectors (e.g., `.avatar__*` matches dynamic hash suffixes).

**Standalone build** (`standalone/entry.ts`) auto-detects the current domain, creates an injector, and exposes `window.__AI_CLASH`. It also listens for `postMessage` RPC calls from content scripts (for Chrome extension ISOLATED world communication), forwarding SSE chunks back.

### Provider `sse.parseLine` contract

Each provider's SSE config has:
- `urlPattern` — regex tested against request URLs to identify chat completion requests
- `detectionKeywords` — strings used to heuristically decide whether a TextDecoder/ReadableStream is carrying chat SSE data (avoids parsing every stream on the page)
- `parseLine(line: string)` — receives one trimmed SSE line; returns `{text, isThink, done, conversationId?} | null`
  - `isThink: true` → thinking/reasoning content, `false` → final response, `null` → stream-end signal
  - Return `null` for lines that should be ignored

### Adding a new provider

1. Create `src/providers/<name>.ts` exporting a `ProviderConfig` object
2. Register it in `src/providers/index.ts` (add to `PROVIDERS` record and export the const)
3. Add domain mapping in `src/standalone/entry.ts` `detectProviderFromDomain()`
4. Add example SSE capture in `examples/<name>/` if needed
