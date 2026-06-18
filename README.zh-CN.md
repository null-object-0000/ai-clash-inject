# AI Clash Inject

[English](./README.md) | **简体中文**

一套轻量级 DOM 自动化库，用于操控 DeepSeek、豆包、Qwen、元宝等 AI 网页平台。

AI Clash Inject 是 AI Clash 的底层网页控制库，用于抽象不同 AI 网站的输入、发送、响应读取、思考模式、联网搜索和流式输出监听等能力。

## 这是什么？

AI Clash Inject 为不同的 AI 网页平台提供统一的自动化层。

它不调用官方 API，而是将脚本注入 AI 网站，操控其现有网页 UI。因此非常适合浏览器扩展、本地自动化、测试、演示和研究工具。

核心能力：

* 向 AI 对话输入框填入提示词
* 发送消息
* 读取流式响应
* 检测思考/推理输出
* 在支持的平台上控制思考模式
* 在支持的平台上控制搜索模式
* 在可用时获取对话 ID
* 开启新对话
* 为不同 AI 平台构建 provider 适配器

## 使用场景

AI Clash Inject 可用于多种场景：

* 浏览器扩展集成
* Bookmarklet / DevTools 控制台注入
* Puppeteer / Playwright 自动化
* AI 网页平台本地调试
* 远程控制实验
* Provider 适配器开发

## 支持的平台

当前 provider 支持仍在持续演进中。

| 平台     | 普通对话 |  思考 |   搜索 | 思考 + 搜索 | 对话 ID | 流式输出 |
| -------- | -------: | ----: | -----: | ----------: | ------: | -------: |
| DeepSeek |       ✅ |    ✅ |     ✅ |           ✅ |       ✅ |        ✅ |
| 豆包     |       ✅ |    ✅ |    ⚠️ |           ✅ |       ✅ |        ✅ |
| Qwen     |       ✅ |    ✅ |    ⚠️ |           ✅ |       ✅ |        ✅ |
| 元宝     |       ✅ |    ✅ |    ⚠️ |           ✅ |       ✅ |        ✅ |
| 文心一言 |       ✅ |   ⚠️ |    ⚠️ |          ⚠️ |       ✅ |        ✅ |

图例：

* ✅ 稳定或已测试
* 🧪 实验性
* ⚠️ 部分支持
* ➖ 不支持

## 安装

```bash
npm install @ai-clash/inject
```

或使用 Bun：

```bash
bun add @ai-clash/inject
```

> 注意：本包处于活跃开发阶段。API 在首个稳定版本发布前可能发生变化。

## 快速开始

### 从浏览器扩展注入

```ts
import { createInjector } from '@ai-clash/inject';

const injector = createInjector({
  provider: 'deepseek',
  adapter: 'window',
});

await injector.inject();

await injector.call('chat', 'send', {
  onSseChunk: (text, isThinking, stage, conversationId) => {
    console.log('[AI Clash Inject] 数据块：', {
      text,
      isThinking,
      stage,
      conversationId,
    });
  },
  onComplete: (fullText, conversationId) => {
    console.log('[AI Clash Inject] 完成：', {
      fullText,
      conversationId,
    });
  },
});
```

### 通过 script 标签注入

```html
<script src="http://localhost:5173/standalone.js"></script>
<script>
  await window.__AI_CLASH.chat.fill('你好');
  await window.__AI_CLASH.chat.send({
    onComplete: (fullText) => {
      console.log(fullText);
    },
  });
</script>
```

### 使用 Puppeteer / Playwright 注入

```ts
await page.addScriptTag({
  url: 'http://localhost:5173/standalone.js',
});

await page.evaluate(async () => {
  const ai = window.__AI_CLASH;

  await ai.chat.fill('来自 AI Clash Inject 的问候');

  await ai.chat.send({
    onComplete: (fullText) => {
      console.log('[AI Clash Inject] 完成：', fullText);
    },
  });
});
```

## 本地开发

```bash
npm install
npm run dev
```

或使用 Bun：

```bash
bun install
bun dev
```

开发服务器启动后，打开：

```text
http://localhost:5173
```

开发页面提供生成注入脚本和 bookmarklet 的工具。

详细的本地调试说明请参见：

```text
docs/DEV.zh-CN.md
```

## Bookmarklet 使用

在本地开发时，你可以通过 bookmarklet 将独立脚本注入到 AI 网站中：

```text
javascript:(async function(){if(!window.AIClashInject){const s=document.createElement('script');s.src='http://localhost:5173/standalone.js';await new Promise(r=>{s.onload=r;document.head.appendChild(s);});}})();
```

然后打开支持的 AI 网站，点击 bookmarklet 即可。

## Provider 适配器

每个 AI 平台都以 provider 适配器的形式实现。

Provider 适配器负责：

* 定位输入框
* 填入提示词文本
* 触发发送
* 读取流式输出
* 检测思考内容
* 检测对话完成
* 控制功能开关，如思考模式或搜索
* 在可用时提取对话元数据

要添加新的 provider，请在以下目录创建新的适配器：

```text
src/providers/
```

推荐的贡献流程：

```text
1. 复制一个已有的 provider 适配器
2. 替换平台特定的 DOM 选择器和事件逻辑
3. 添加本地调试笔记
4. 测试普通对话、思考、搜索和流式输出
5. 提交 Pull Request
```

## 已知限制

AI Clash Inject 通过网页 UI 操控第三方 AI 网站，因此当这些网站更新其 DOM 结构、路由逻辑或流式响应格式时，适配器可能会失效。

已知限制：

* 部分平台内置搜索功能，不暴露手动搜索开关。
* 部分平台自动决定思考模式。
* 部分平台可能将推理输出拆分为多个内部阶段。
* 搜索过程输出目前会被过滤，不会作为用户可见的回答内容暴露。
* Provider 支持可能因地区、账户、A/B 实验和 UI 版本而有所不同。

## 负责任的使用

请仅在你有权访问的网站和账户上使用本库。遵守每个 AI 平台的服务条款、速率限制和使用政策。

本项目旨在用于浏览器扩展集成、本地自动化、测试和研究。不得用于滥用、垃圾信息、抓取隐私数据或绕过访问控制。

## 贡献

欢迎贡献，尤其欢迎以下方面：

* 新的 provider 适配器
* 修复失效的 DOM 选择器
* 改进流式响应解析
* 调试工具
* 文档改进
* Playwright / Puppeteer 示例

在提交 provider 适配器之前，请包含：

* 目标平台名称
* 支持的功能
* 已知不支持的功能
* 手动测试结果
* 当行为因 UI 版本而异时的截图或说明

## 许可证

MIT
