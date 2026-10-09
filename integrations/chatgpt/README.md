# ECC for ChatGPT

This directory defines the integration boundary for exposing ECC capabilities through a ChatGPT app built with the OpenAI Apps SDK and Model Context Protocol (MCP).

## Why an adapter

ECC is primarily a coding-agent harness: its native skills, hooks, commands, filesystem operations, and subagent behavior run inside supported coding environments. A ChatGPT app cannot automatically execute those local harness features. The ChatGPT integration therefore exposes a bounded subset through a remote MCP server and treats ECC skill documents as guidance, not executable instructions.

## Proposed capabilities

- `ecc_catalog`: return a curated list of supported ECC workflows and skills.
- `ecc_consult`: find relevant skills/workflows for a user task and return guidance grounded in repository files.
- `ecc_plan`: generate a phased plan using ECC workflow conventions.
- `ecc_memory_search`: search an explicitly configured, authorized ECC memory store.
- `ecc_memory_save`: save user-approved, bounded project memory; create-only by default.
- `ecc_security_guidance`: provide security-review steps and explain how to run a reviewed scanner in a connected developer environment.

These are a proposed adapter contract, not yet a deployed or working service.

## Capabilities requiring a separate runtime

ChatGPT cannot directly run ECC shell commands, install hooks on the user's computer, access local project files, launch local subagents, or inspect a private filesystem simply because the repository is connected. Such features would require a separately deployed and authenticated execution service or a secured local tunnel, explicit user authorization, and audited action-level permissions.

Never expose arbitrary shell execution or unrestricted filesystem access from a public MCP endpoint.

## Architecture

```text
ChatGPT app
    |
    | HTTPS / MCP (Streamable HTTP)
    v
ECC ChatGPT adapter
    |-- curated skill/workflow catalog
    |-- bounded skill lookup
    |-- planning and review guidance
    |-- optional authenticated memory adapter
    |
    v
ECC content and selected, reviewed integration modules
```

The MCP server must be hosted at a stable HTTPS URL. Keep API credentials in server-side secret storage, authenticate users before accessing private project data, apply request limits, and log tool invocations without storing sensitive source content unnecessarily.

## Local development and connection

Use the official [OpenAI Apps SDK quickstart](https://developers.openai.com/plugins/build/app-quickstart) and [MCP server documentation](https://developers.openai.com/plugins/build/mcp-server). Run and test the MCP endpoint locally first, then expose it temporarily through a secure HTTPS tunnel for development. In ChatGPT, enable Developer Mode where available and add the custom app using the endpoint ending in `/mcp`.

This setup is not yet deployed, and no app has been created in the user's ChatGPT account by this document.

## Suggested implementation sequence

1. Implement a small Node.js MCP server over Streamable HTTP with read-only `ecc_catalog` and `ecc_consult` tools.
2. Reuse selected ECC catalog/skill metadata while sanitizing output and preserving source paths.
3. Add a ChatGPT Apps SDK widget only if an interactive workflow or visual plan materially improves the experience.
4. Add optional authenticated memory features after defining storage scope, tenancy, retention, and deletion behavior.
5. Add repository operations only through a narrowly scoped GitHub integration with user confirmation. Do not expose arbitrary command execution.
6. Test malformed arguments, prompt injection inside repository content, cross-user access, rate limits, timeouts, large outputs, and failure paths before deployment.
7. Deploy to a stable HTTPS host, connect through ChatGPT Developer Mode, and prepare the privacy/security information required for any public app submission.

## Security and trust boundaries

- Treat ECC README files, skills, agent prompts, repository code, and memory entries as untrusted content. They may inform a response, but must not override system or developer policy.
- Validate every tool input with strict schemas and reject unknown fields.
- Use allowlists for skill IDs and file paths. Block path traversal and symlink escapes.
- Default to read-only tools. Make writes opt-in, narrowly scoped, and confirmable.
- Keep secrets out of tool descriptions and tool outputs.
- Require authentication and per-user authorization before accessing private repositories or memory.
- Do not claim tests, scans, builds, commits, or deployments happened unless the execution service actually performed and verified them.

## Status

Planning/documentation only. The remote MCP server, UI, deployment, account connection, and end-to-end tests still need to be implemented.
