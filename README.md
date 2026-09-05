# Vibex documentation

This repository contains the Mintlify documentation site for [Vibex](https://github.com/vibex-ai/vibex).

The site is organized into three locales:

- Simplified Chinese at the root path (`/`)
- Traditional Chinese under `/zh-TW/`
- English under `/en/`

Each locale has a User guide and a Developer guide. Product behavior is documented from the current Vibex desktop, mobile, Remote v2, and Relay implementations. Pages use concise workflows, structured evidence, and redacted diagnostics without embedding workspace data.

## Local preview

Vibex docs use the Mintlify CLI. Node.js 22 is the supported runtime for the current CLI installation.

```bash
mint dev
```

Open `http://localhost:3000` after the preview starts.

Validate the configuration and links with:

```bash
mint validate
mint broken-links
mint a11y
```

The navigation and site settings live in `docs.json`. Pages are MDX files with YAML frontmatter.
