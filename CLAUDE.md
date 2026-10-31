# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A personal C++ knowledge base built with [Quartz](https://quartz.jzhao.xyz/) v5 and deployed to GitHub Pages at https://nrss33.github.io/Cpp/. The actual content is Markdown notes in `content/`, authored in Obsidian.

## 同步规则

每次在本工作区改完东西（笔记、配置、说明等）后，都要立刻同步到网站：先 `npx quartz build` 校验，再 `git add -A && git commit && git push` 推到 `main`（`publish.bat` 或 `npx quartz sync` 均可）。推送会自动触发 GitHub Actions 部署到 https://nrss33.github.io/Cpp/，无需手动操作。

每次更新笔记（新增/删除笔记、新增分类、改动内容结构等）后，也要同步更新 `CLAUDE.md` 里对应的描述（`content/` 笔记清单、内容约定等），保持项目文档与实际内容一致。

## Commands

- `npm ci` — install dependencies (Node >= 22, npm >= 10.9.2; `engine-strict` is on)
- `npx quartz build` — build the static site into `public/`
- `npx quartz build --serve` — build and serve locally at http://localhost:8080
- `publish.bat` — Windows helper: `git add -A && git commit && git push` to the `main` branch, which triggers the CI deploy (also achievable with `npx quartz sync` per the README)
- `npm run check` — `tsc --noEmit` + `prettier --check`
- `npm run format` — `prettier --write`
- `npm test` — run tests via `tsx --test`

## Architecture

- **Quartz v5** static site generator. Framework code in `quartz/` is vendored from upstream — do not edit it. User overrides go in `.quartz/` (gitignored).
- **`content/`** — the only directory you author. Open this folder in Obsidian to edit notes. Notes are organized by功能分类:
  - `content/index.md` — homepage (title `C++ 笔记`), links to the notes below.
  - `content/language_basics.md` — 语言基础（按功能分类，目前含「命名空间」「引用」「内联函数」三节）。
- **`quartz.config.yaml`** — all site configuration (title, locale, theme, plugins, layout) is declared here. Plugins are `@quartz-community/*` / `@quartz-themes/*` npm packages that the `prebuild` script (`install-plugins`) auto-installs into `.quartz/plugins/`.
- **`public/`** — build output (gitignored). Never edit directly.
- **Deployment** — pushing to the `main` branch triggers `.github/workflows/deploy.yml`, which runs `npm ci` + `npx quartz build` and deploys `public/` to GitHub Pages. No separate deploy branch is used.

## Content conventions (`content/*.md`)

- Frontmatter uses `title`, `tags`, `created`.
- Notes are organized by功能分类: each category is one Markdown file (e.g. `content/language_basics.md`). Each file mirrors `Linux.md`: a `## 目录` TOC with unnumbered `### 分类` groups + `[[#主题|主题]]` links, then numbered `## N. 分类` sections with `### 主题` sub-topics.
- Each topic keeps it short — only 作用 and 用法:
  - `**作用**：…` one line for what it's for
  - `**用法**：` a few bullets (定义 / 访问 / 引入 …)
  - one fenced `cpp` code block, plus an optional `> 附:` note for edge cases
- Cross-links use Obsidian wikilinks: `[[note#anchor]]` from `index.md`. **Write the anchor as verbatim heading text** (spaces/parens/case exactly as the heading) — Quartz slugifies anchors at build time and Obsidian matches headings verbatim, so verbatim anchors resolve in both.
- **Adding a topic touches three places:** (1) a new `### 主题` section in the category file, (2) a `[[#主题|主题]]` link in that file's `## 目录`, (3) a `[[分类文件#分类|N. 分类]]` link in `index.md`. Then run `publish.bat` to deploy.
- The build's `ignorePatterns` excludes `private`, `templates`, and `.obsidian`; `.claudian/` and `.playwright-mcp/` are also gitignored.
