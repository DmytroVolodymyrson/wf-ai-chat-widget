# WheelsFeels Chat Widget - AI Assistant

An n8n-powered AI chat widget for the WheelsFeels website that helps customers find the right vehicle storage and sleeping systems.

## Features

- **Product Lookup**: Queries Supabase database to find products matching customer's vehicle
- **Smart Vehicle Matching**: Strips trim levels (Wilderness, TRD Pro, etc.) for accurate generation matching
- **Markdown Links**: Returns clean clickable links instead of raw URLs
- **Year Validation**: Asks for vehicle year when not provided
- **Multilingual**: Responds in customer's language
- **Lead Collection**: Collects name and email for follow-up

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         CHAT WIDGET WORKFLOW                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐    ┌──────────────┐    ┌────────────────────────────┐ │
│  │ Chat Trigger │───▶│   AI Agent   │───▶│   Product Lookup Tool      │ │
│  │              │    │   (Anton)    │    │   ├─ Find Vehicle Gen      │ │
│  └──────────────┘    └──────────────┘    │   └─ Search Products       │ │
│         │                   │            └────────────────────────────┘ │
│         │                   │                                           │
│         ▼                   ▼                                           │
│  ┌──────────────┐    ┌──────────────┐                                   │
│  │ Window Buffer│    │ OpenAI Chat  │                                   │
│  │    Memory    │    │   (GPT-4.1)  │                                   │
│  └──────────────┘    └──────────────┘                                   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## Configuration

### Chat Trigger Settings

- **Mode**: Hosted Chat (n8n-managed widget)
- **Authentication**: None (public)
- **Initial Message**: "Hi there! I'm Anton from WheelsFeels. How can I help you today?"

### Allowed Origins

```
http://localhost:8000
https://stage.wheelsfeels.com
https://wheelsfeels.com
```

### AI Agent (Anton)

The AI assistant:
- Helps customers find storage drawers, camping systems, and accessories
- Provides product links in Markdown format
- Asks for vehicle year when not provided
- Collects name and email for follow-up
- Escalates returns/refunds and collaboration inquiries to help@wheelsfeels.com

## Evaluation System

The workflow includes a built-in evaluation system using n8n's evaluation nodes:
- **19 test cases** covering product lookups, support questions, and conversation handling
- **AI-based scoring** (1-5) using semantic matching
- **Google Sheets integration** for test data and results

See [CLAUDE.md](CLAUDE.md) for detailed evaluation documentation.

## Deployment

### Using @n8n/chat Widget

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

- [ ] Pass product page context via metadata
- [ ] Add Slack monitoring with threading
- [ ] Implement email follow-up workflow for ended chats
- [ ] Store chat transcripts in Supabase

## Related Projects

- [wf-email-followup-automation](https://github.com/DmytroVolodymyrson/wf-email-followup-automation) - Automated AI email follow-ups from Tawk.to live chat

## License

Private project for WheelsFeels.
