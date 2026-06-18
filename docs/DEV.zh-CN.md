# AI Clash Inject - 开发指南

[English](./DEV.md) | **简体中文**

## 快速开始

### 1. 启动开发服务器

```bash
npm install
npm run dev
```

或使用 Bun：

```bash
bun install
bun dev
```

服务器启动后访问：**<http://localhost:5173>**

开发页面提供注入代码生成器，可用于构建 bookmarklet 和控制台脚本。

### 2. 注入到 AI 网站

#### Bookmarklet（推荐）

1. 访问 <http://localhost:5173>
2. 点击"生成 Bookmarklet"
3. 将生成的书签拖到书签栏
4. 打开 DeepSeek 等 AI 网站
5. 点击书签栏的"AI Clash Inject"

注入后，`window.AIClashInject` 用于防重复加载。运行时 API 暴露在 `window.__AI_CLASH` 上。以下示例均通过 `window.__AI_CLASH` 调用。

```text
javascript:(async function(){if(!window.AIClashInject){const s=document.createElement('script');s.src='http://localhost:5173/standalone.js';await new Promise(r=>{s.onload=r;document.head.appendChild(s);});}})();
```

#### Console 手动注入

1. 打开 DeepSeek 等 AI 网站
2. 按 F12 打开开发者工具
3. 在 Console 中运行：

```javascript
(async ()=>{
  if(!window.AIClashInject){
    const s=document.createElement('script');
    s.src='http://localhost:5173/standalone.js';
    await new Promise((resolve, reject) => {
      s.onload = () => {
        resolve();
      };
      s.onerror = (err) => {
        console.error('[AI Clash Inject]', '❌ 脚本加载失败', err);
        reject(new Error('脚本加载失败，请检查开发服务器是否启动'));
      };
      document.head.appendChild(s);
    });
  }
})().catch(err => console.error('[AI Clash Inject]', '💥 注入失败:', err));
```

### 3. 测试能力

注入成功后，在 Console 中运行：

```javascript
// 填充输入框
await window.__AI_CLASH.chat.fill('我想去洗车，汽车店距离我家50米，你说我应该开车去还是走过去？')

// 发送消息并监听流式输出
await window.__AI_CLASH.chat.send({
  onConversationId: (conversationId) => {
    console.log('[AI Clash Inject] 获取到会话 ID:', conversationId);
  },
  onSseChunk: (text, isThink, stage, conversationId) => {
    console.log('[AI Clash Inject] 收到 SSE chunk:', text, '思考模式:', isThink, '阶段:', stage, '会话 ID:', conversationId);
  },
  onComplete: (fullText, conversationId) => {
    console.log('[AI Clash Inject] 完成，完整回复:', fullText, '会话 ID:', conversationId);
  }
})

// 开始新对话
await window.__AI_CLASH.chat.newChat()

// 思考模式
await window.__AI_CLASH.thinking.getState()
await window.__AI_CLASH.thinking.enable()
await window.__AI_CLASH.thinking.getState()
await window.__AI_CLASH.thinking.disable()
await window.__AI_CLASH.thinking.getState()

// 一站式：同时开启思考 + 搜索 + 新对话
await window.__AI_CLASH.chat.send('我想去洗车，汽车店距离我家50米，你说我应该开车去还是走过去？', {
  thinking: true,
  search: true,
  newChat: true
}, {
  onConversationId: (conversationId) => {
    console.log('[AI Clash Inject] 获取到会话 ID:', conversationId);
  },
  onSseChunk: (text, isThink, stage, conversationId) => {
    console.log('[AI Clash Inject] 收到 SSE chunk:', text, '思考模式:', isThink, '阶段:', stage, '会话 ID:', conversationId);
  },
  onComplete: (fullText, conversationId) => {
    console.log('[AI Clash Inject] 完成，完整回复:', fullText, '会话 ID:', conversationId);
  }
})
```

## 已知问题

- 搜索过程的产出会过滤掉，暂不输出给用户。
- 豆包/腾讯元宝/文心一言是内置联网搜索，自动触发。
- 豆包的快速（非思考、普通模式）模式会将首包算作 think 内容。
- 千问若触发多阶段思考，目前初始阶段思考不会被监测到（`"mime_type": "plan_cot/post"`）。
- 文心一言的深度思考（X1.1/5.0 无手动开关，4.5 Turbo 自动决策）不支持手动控制。
