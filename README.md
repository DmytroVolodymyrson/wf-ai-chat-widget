# WheelsFeels Chat Widget - AI Assistant

An n8n-powered AI chat widget for the WheelsFeels website. Provides real-time customer support.

## Overview

This project contains the n8n workflow for a website chat widget that helps WheelsFeels customers find the right vehicle storage and sleeping systems.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHAT WIDGET WORKFLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    ┌──────────────────┐    ┌──────────────┐   │
│  │ Chat Trigger │───▶│ Window Buffer    │───▶│   AI Agent   │   │
│  │ (Hosted Chat)│    │    Memory        │    │ (GPT-4.1-mini)│  │
│  └──────────────┘    └──────────────────┘    └──────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Nodes

| Node | Type | Purpose |
|------|------|---------|
| Chat Trigger | `@n8n/n8n-nodes-langchain.chatTrigger` | Hosted chat widget endpoint |
| AI Agent | `@n8n/n8n-nodes-langchain.agent` | GPT-4.1-mini with customer service prompt |
| OpenAI Chat Model | `@n8n/n8n-nodes-langchain.lmChatOpenAi` | LLM configuration (temp: 0.7) |
| Window Buffer Memory | `@n8n/n8n-nodes-langchain.memoryBufferWindow` | Session-based memory (10 messages) |

## Configuration

### Chat Trigger Settings

- **Mode**: Hosted Chat (n8n-managed widget)
- **Authentication**: None (public)
- **Initial Message**: "Hi there! I'm Anton from WheelsFeels. How can I help you today?"

### CORS Allowed Origins

```
http://localhost:8000
https://stage.wheelsfeels.com
https://wheelsfeels.com
```

### AI Agent System Prompt

The AI acts as "Anton", a friendly customer service specialist who:
- Helps customers find storage drawers, camping systems, and accessories
- Answers questions about products, shipping, warranty, and returns
- Keeps responses concise but helpful
- Escalates complex issues to email/phone support

## Deployment

### Using n8n Hosted Chat

The workflow uses n8n's built-in hosted chat mode. Once active, the chat widget is available at:

```
https://wheelsfeels.app.n8n.cloud/webhook/<webhook-id>/chat
```

### Using @n8n/chat Widget (Custom Integration)

For custom website integration, use the [@n8n/chat](https://www.npmjs.com/package/@n8n/chat) package:

```html
<link href="https://cdn.jsdelivr.net/npm/@n8n/chat/dist/style.css" rel="stylesheet" />
<script type="module">
  import { createChat } from 'https://cdn.jsdelivr.net/npm/@n8n/chat/dist/chat.bundle.es.js';
  createChat({
    webhookUrl: 'https://wheelsfeels.app.n8n.cloud/webhook/<webhook-id>/chat',
    mode: 'window',
    showWelcomeScreen: false,
    initialMessages: ['Hi there! How can I help you today?']
  });
</script>
```

## Future Improvements

- [ ] Add Supabase product lookup tools (vehicle generation → products)
- [ ] Pass product page context via metadata
- [ ] Add Slack monitoring with threading
- [ ] Implement email follow-up workflow for ended chats
- [ ] Store chat transcripts in Supabase

## Related Projects

- [wf-email-followup-automation](https://github.com/DmytroVolodymyrson/wf-email-followup-automation) - Automated AI email follow-ups from Tawk.to live chat

## License

Private project for WheelsFeels.
