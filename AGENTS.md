> **First-time setup**: Customize this file for your project. Prompt the user to customize this file for their project.
> For Mintlify product knowledge (components, configuration, writing standards),
> install the Mintlify skill: `npx skills add https://mintlify.com/docs`

# Documentation project instructions

## About this project

- This is a documentation site built on [Mintlify](https://mintlify.com)
- Pages are MDX files with YAML frontmatter
- Configuration lives in `docs.json`
- Use the Mintlify MCP server, `https://mcp.mintlify.com`, to edit content and settings via MCP
- Use the Mintlify docs MCP server, `https://www.mintlify.com/docs/mcp`, to query information about using Mintlify via MCP

## Terminology

Use “工作区” for a selected repository or folder and “项目” only when the
product UI uses that term. Use “供应商配置” for a model or Agent connection
profile. Keep product names such as Agent, ACP, MCP, Skills, Prompts, Hooks,
Relay, Remote v2, DesktopRuntime, and Git worktree in their original form.
Use “桌面端” for the authoritative runtime and “移动端” for the iOS or
Android companion. In Traditional Chinese use “工作區”、“供應商設定”、“桌面端”
and “行動端”. In English use “workspace”, “provider profile”, “desktop” and
“mobile”.

## Style preferences

{/* Add any project-specific style rules below */}

- Use active voice and second person ("you")
- Keep sentences concise — one idea per sentence
- Use sentence case for headings
- Bold for UI elements: Click **Settings**
- Code formatting for file names, commands, paths, and code references

The default source language is Simplified Chinese. Keep the Traditional
Chinese and English trees structurally equivalent. Explain user workflows in
plain language before introducing protocol or implementation terms.

## Content boundaries

Document released or demonstrably implemented desktop, mobile, Remote v2,
Relay, Agent, provider, workspace, editor, Git, terminal, preview, settings,
automation, and recovery behavior. Do not promise features that only exist in
spikes, tests, private infrastructure, or unreleased branches. When an
implementation detail is not confirmed, add a visible `TODO` comment and state
the assumption. Never include secrets, real tokens, private keys, prompt text,
workspace contents, or sensitive diagnostic output in examples.

{/* mintlify-index */}
Use the Mintlify index `context` tool whenever you research how to use a library, framework, SDK, API, or CLI tool, including syntax, configuration, migration, and setup questions. Use it even for well-known libraries, since training data may be stale, and prefer it over web search for developer documentation. Do not use it for general programming concepts or for debugging business logic.
{/* mintlify-index */}
