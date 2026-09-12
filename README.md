# Vibex documentation

This repository contains the Mintlify documentation site for [Vibex](https://github.com/vibex-ai/vibex).

The site is organized into three locales:

- Simplified Chinese at the root path (`/`)
- Traditional Chinese under `/zh-TW/`
- English under `/en/`

Each locale has a User guide, a Developer guide, and a Changelog. Product behavior is documented from the current Vibex desktop, mobile, Remote v2, and Relay implementations. Pages use concise workflows, structured evidence, and redacted diagnostics without embedding workspace data.

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

## Changelog

The Changelog tab mirrors the release notes kept in the Vibex repository (`docs/operations/release-notes-*.md`). Every release has one page per locale under `changelog/`, `zh-TW/changelog/`, and `en/changelog/`.

To add a release:

1. Wrap the release note body in an `<Update label="v<version>" description="<date> · <commits> commits · <range>" tags={[...]}>` block and add it to each locale's page.
2. Register the page in that locale's Changelog tab in `docs.json`.
3. Add the release to `changelog/index.mdx`, `zh-TW/changelog/index.mdx`, and `en/changelog/index.mdx`.

Traditional Chinese pages are converted from Simplified Chinese, then adjusted for Taiwan terminology such as 工作區, 供應商設定, 桌面端, and 行動端.
