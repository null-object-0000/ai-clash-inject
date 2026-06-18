# AI Clash Inject

**English** | [简体中文](./README.zh-CN.md)

A lightweight DOM automation library for controlling AI web platforms such as DeepSeek, Doubao, Qwen, Yuanbao, and more.

AI Clash Inject 是 AI Clash 的底层网页控制库，用于抽象不同 AI 网站的输入、发送、响应读取、思考模式、联网搜索和流式输出监听等能力。

## What is this?

AI Clash Inject provides a unified automation layer for different AI web platforms.

Instead of calling official APIs, it works by injecting scripts into AI websites and controlling their existing web UI. This makes it useful for browser extensions, local automation, testing, demos, and research tools.

Core capabilities:

* Fill prompts into AI chat input boxes
* Send messages
* Read streaming responses
* Detect thinking / reasoning output
* Control thinking mode when supported
* Control search mode when supported
* Get conversation IDs when available
* Start new chats
* Build provider adapters for different AI platforms

## Use cases

AI Clash Inject can be used in multiple scenarios:

* Browser extension integration
* Bookmarklet / DevTools console injection
* Puppeteer / Playwright automation
* Local debugging for AI web platforms
* Remote control experiments
* Provider adapter development

## Supported platforms

Current provider support is still evolving.

| Platform | Normal Chat | Thinking | Search | Thinking + Search | Conversation ID | Streaming |
| -------- | ----------: | -------: | -----: | ----------------: | --------------: | --------: |
| DeepSeek |           ✅ |        ✅ |      ✅ |                 ✅ |               ✅ |         ✅ |
| Doubao   |           ✅ |        ✅ |     ⚠️ |                 ✅ |               ✅ |         ✅ |
| Qwen     |           ✅ |        ✅ |     ⚠️ |                 ✅ |               ✅ |         ✅ |
| Yuanbao  |           ✅ |        ✅ |     ⚠️ |                 ✅ |               ✅ |         ✅ |
| Wenxin   |           ✅ |       ⚠️ |     ⚠️ |                ⚠️ |               ✅ |         ✅ |

Legend:

* ✅ Stable or tested
* 🧪 Experimental
* ⚠️ Partially supported
* ➖ Not supported

## Status

AI Clash Inject is in early development. Provider adapters may break when AI websites update their DOM structure or streaming format.

## Installation

> **Note:** This package has not been published to npm yet. For now, clone this repository and build it locally.

```bash
git clone https://github.com/null-object-0000/ai-clash-inject.git
cd ai-clash-inject
npm install
npm run build
```

Once published, you will be able to install with:

```bash
npm install @ai-clash/inject
```

Or with Bun:

```bash
bun add @ai-clash/inject
```

## Quick start

### Inject from a browser extension

```ts
import { createInjector } from '@ai-clash/inject';

const injector = createInjector({
  provider: 'deepseek',
  adapter: 'window',
});

await injector.inject();

await injector.call('chat', 'fill', 'Hello from AI Clash Inject');

await injector.call('chat', 'send', {
  onSseChunk: (text, isThinking, stage, conversationId) => {
    console.log('[AI Clash Inject] chunk:', {
      text,
      isThinking,
      stage,
      conversationId,
    });
  },
  onComplete: (fullText, conversationId) => {
    console.log('[AI Clash Inject] complete:', {
      fullText,
      conversationId,
    });
  },
});
```

### Inject with a local standalone script

```html
<script src="http://localhost:5173/standalone.js"></script>
<script>
  await window.__AI_CLASH.chat.fill('Hello');
  await window.__AI_CLASH.chat.send({
    onComplete: (fullText) => {
      console.log(fullText);
    },
  });
</script>
```

### Inject with Puppeteer / Playwright

```ts
await page.addScriptTag({
  url: 'http://localhost:5173/standalone.js',
});

await page.evaluate(async () => {
  const ai = window.__AI_CLASH;

  await ai.chat.fill('Hello from AI Clash Inject');

  await ai.chat.send({
    onComplete: (fullText) => {
      console.log('[AI Clash Inject] complete:', fullText);
    },
  });
});
```

## Local development

```bash
npm install
npm run dev
```

Or with Bun:

```bash
bun install
bun dev
```

After the dev server starts, open:

```text
http://localhost:5173
```

The development page provides tools for generating injection scripts and bookmarklets.

For detailed local debugging instructions, see [docs/DEV.md](docs/DEV.md).

## Bookmarklet usage

During local development, you can inject the standalone script into an AI website by using a bookmarklet:

```text
javascript:(async function(){if(!window.AIClashInject){const s=document.createElement('script');s.src='http://localhost:5173/standalone.js';await new Promise(r=>{s.onload=r;document.head.appendChild(s);});}})();
```

Then open a supported AI website and click the bookmarklet.

After injection, `window.AIClashInject` guards against double-loading. The runtime API is exposed as `window.__AI_CLASH` — use that for all chat, thinking, and search operations.

## Provider adapter

Each AI platform is implemented as a provider adapter.

A provider adapter is responsible for:

* Locating the input box
* Filling prompt text
* Triggering send
* Reading streaming output
* Detecting thinking content
* Detecting completion
* Controlling feature switches such as thinking or search
* Extracting conversation metadata when available

To add a new provider, create a new adapter under:

```text
src/providers/
```

Recommended contribution flow:

1. Copy an existing provider adapter.
2. Replace platform-specific DOM selectors and event logic.
3. Add local debugging notes.
4. Test normal chat, thinking, search, and streaming output.
5. Submit a pull request.

## Known limitations

AI Clash Inject controls third-party AI websites through their web UI, so adapters may break when those websites update their DOM structure, routing logic, or streaming response format.

Known limitations:

* Some platforms have built-in search and do not expose a manual search switch.
* Some platforms decide thinking mode automatically.
* Some platforms may split reasoning output into multiple internal stages.
* Search-process output is currently filtered and not exposed as user-facing answer content.
* Provider support may differ across regions, accounts, A/B experiments, and UI versions.

## Responsible usage

Use this library only with websites and accounts you are authorized to access. Respect the terms, rate limits, and usage policies of each AI platform.

This project is intended for browser extension integration, local automation, testing, and research. It is not intended for abuse, spam, scraping private data, or bypassing access controls.

## Contributing

Contributions are welcome, especially:

* New provider adapters
* Fixes for broken DOM selectors
* Better streaming response parsing
* Debugging tools
* Documentation improvements
* Playwright / Puppeteer examples

Before submitting a provider adapter, please include:

* Target platform name
* Supported features
* Known unsupported features
* Manual test result
* Screenshots or notes when behavior depends on UI version

## License

MIT
