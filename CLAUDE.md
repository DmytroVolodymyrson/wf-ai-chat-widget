# WheelsFeels Chat Widget Project

## Workflow Info

- **Workflow Name**: Website Chat Widget - AI Assistant
- **Workflow ID**: `DKfVfqHubRSwIGYT`
- **n8n Instance**: wheelsfeels.app.n8n.cloud

## MCP Tools

Use the `n8n-wheelsfeels` MCP server to manage the workflow:

```bash
# Get workflow
mcp__n8n-wheelsfeels__n8n_get_workflow(id="DKfVfqHubRSwIGYT")

# Update workflow
mcp__n8n-wheelsfeels__n8n_update_full_workflow(...)

# List all workflows
mcp__n8n-wheelsfeels__n8n_list_workflows()
```

## Development Notes

### Current State (v1)

Basic implementation with:
- Chat Trigger (hosted mode)
- AI Agent with basic system prompt
- Window Buffer Memory (10 messages)
- GPT-4.1-mini model

### Planned Enhancements

1. **Product Lookup Tools**: Add Supabase tools to query vehicle generations and products (similar to Tawk.to follow-up workflow)

2. **Page Context**: Pass current product page URL via `metadata` option to give AI context about what the customer is viewing

3. **Slack Monitoring**: Send chat logs to Slack with threading by session

4. **Follow-up Emails**: Trigger email follow-ups when chat sessions end

## Related Files

- `workflow.json` - Exported n8n workflow (safe to share, contains no API keys)

## Security

This is a PUBLIC repository. Do not commit:
- `.mcp.json` (contains API keys)
- `.env` files
- Any credential files
