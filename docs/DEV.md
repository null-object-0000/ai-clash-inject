# AI Clash Inject - Development Guide

**English** | [简体中文](./DEV.zh-CN.md)

## Quick start

### 1. Start the dev server

```bash
npm install
npm run dev
```

Or with Bun:

```bash
bun install
bun dev
```

After the server starts, open: **<http://localhost:5173>**

The dev page provides an injection code generator for building bookmarklets and console scripts.

### 2. Inject into an AI website

#### Bookmarklet (recommended)

1. Visit <http://localhost:5173>
2. Click "Generate Bookmarklet"
3. Drag the generated bookmark to your bookmarks bar
4. Open DeepSeek or another AI website
5. Click the "AI Clash Inject" bookmarklet

```text
javascript:(async function(){if(!window.AIClashInject){const s=document.createElement('script');s.src='http://localhost:5173/standalone.js';await new Promise(r=>{s.onload=r;document.head.appendChild(s);});}})();
```

#### Console manual injection

1. Open DeepSeek or another AI website
2. Press F12 to open DevTools
3. Run in the Console:

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
        console.error('[AI Clash Inject]', '❌ Script load failed', err);
        reject(new Error('Script load failed — is the dev server running?'));
      };
      document.head.appendChild(s);
    });
  }
})().catch(err => console.error('[AI Clash Inject]', '💥 Injection failed:', err));
```

### 3. Test capabilities

After injection, run these in the Console:

```javascript
// Fill the input box
await window.__AI_CLASH.chat.fill('I want to wash my car. The car wash is 50 meters from my house. Should I drive or walk?')

// Send and listen to streaming output
await window.__AI_CLASH.chat.send({
  onConversationId: (conversationId) => {
    console.log('[AI Clash Inject] Got conversation ID:', conversationId);
  },
  onSseChunk: (text, isThink, stage, conversationId) => {
    console.log('[AI Clash Inject] SSE chunk:', text, 'thinking:', isThink, 'stage:', stage, 'conversationId:', conversationId);
  },
  onComplete: (fullText, conversationId) => {
    console.log('[AI Clash Inject] Complete, full response:', fullText, 'conversationId:', conversationId);
  }
})

// Start a new chat
await window.__AI_CLASH.chat.newChat()

// Thinking mode
await window.__AI_CLASH.thinking.getState()
await window.__AI_CLASH.thinking.enable()
await window.__AI_CLASH.thinking.getState()
await window.__AI_CLASH.thinking.disable()
await window.__AI_CLASH.thinking.getState()

// One-shot: enable thinking + search + new chat in a single call
await window.__AI_CLASH.chat.send('I want to wash my car. The car wash is 50 meters from my house. Should I drive or walk?', {
  thinking: true,
  search: true,
  newChat: true
}, {
  onConversationId: (conversationId) => {
    console.log('[AI Clash Inject] Got conversation ID:', conversationId);
  },
  onSseChunk: (text, isThink, stage, conversationId) => {
    console.log('[AI Clash Inject] SSE chunk:', text, 'thinking:', isThink, 'stage:', stage, 'conversationId:', conversationId);
  },
  onComplete: (fullText, conversationId) => {
    console.log('[AI Clash Inject] Complete, full response:', fullText, 'conversationId:', conversationId);
  }
})
```

## Known issues

- Search process output is filtered and not exposed to users.
- Doubao / Yuanbao / Wenxin have built-in search that triggers automatically.
- Doubao's fast (non-thinking, normal) mode treats the first chunk as thinking content.
- When Qwen enters multi-stage thinking, the initial planning stage is not detected (`"mime_type": "plan_cot/post"`).
- Wenxin's deep thinking (X1.1/5.0 has no manual switch; 4.5 Turbo auto-decides) cannot be manually controlled.
