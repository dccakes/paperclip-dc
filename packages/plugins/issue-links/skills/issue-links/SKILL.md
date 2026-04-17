---
name: issue-links
description: >
  Read and write local filesystem path and GitHub PR URL fields on issues.
  Use when you need to record where an issue's code lives on disk or link
  it to its pull request.
---

# Issue Links Plugin

The Issue Links plugin stores two extra fields on each issue:

- **Local path** — absolute filesystem path to the relevant directory or file (e.g. `/Users/me/projects/repo`)
- **GitHub PR URL** — full URL of the pull request (e.g. `https://github.com/org/repo/pull/123`)

Both fields are readable and writable by agents via the plugin tool API.

## Calling Plugin Tools

Plugin tools are invoked via a single endpoint. All standard `PAPERCLIP_*` env vars are available.

```
POST $PAPERCLIP_API_URL/api/plugins/tools/execute
Authorization: Bearer $PAPERCLIP_API_KEY
Content-Type: application/json

{
  "tool": "<namespaced-tool-name>",
  "parameters": { ... },
  "runContext": {
    "agentId": "$PAPERCLIP_AGENT_ID",
    "runId": "$PAPERCLIP_RUN_ID",
    "companyId": "$PAPERCLIP_COMPANY_ID",
    "projectId": null
  }
}
```

A successful response contains `{ "result": { "content": "...", "data": { ... } } }`.
An error response contains `{ "result": { "error": "..." } }`.

## Available Tools

### `paperclip-issue-links:issue-links.set-local-path`

Set or clear the local filesystem path for an issue.

```json
{
  "tool": "paperclip-issue-links:issue-links.set-local-path",
  "parameters": {
    "issueId": "<issue-id>",
    "value": "/absolute/path/to/repo"
  },
  "runContext": { "agentId": "...", "runId": "...", "companyId": "...", "projectId": null }
}
```

Pass `"value": ""` to clear the field. The response data includes `{ issueId, localPath }`.

### `paperclip-issue-links:issue-links.set-github-pr-url`

Set or clear the GitHub PR URL for an issue.

```json
{
  "tool": "paperclip-issue-links:issue-links.set-github-pr-url",
  "parameters": {
    "issueId": "<issue-id>",
    "value": "https://github.com/org/repo/pull/123"
  },
  "runContext": { "agentId": "...", "runId": "...", "companyId": "...", "projectId": null }
}
```

Pass `"value": ""` to clear the field. The response data includes `{ issueId, githubPrUrl }`.

## When to Use

- After creating or starting work on an issue: call `set-local-path` with the path to the relevant repo/directory.
- After opening a pull request: call `set-github-pr-url` with the PR URL.
- Both calls log an activity entry on the issue automatically.
