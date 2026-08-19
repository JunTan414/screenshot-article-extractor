# Screenshot Article Extractor · 截图文章提取器

An agent skill that turns an ordered series of article screenshots into one faithful, structured Markdown document. Designed to be **tool-agnostic**: it works in any agent host with multimodal vision (WorkBuddy, Claude Code, Codex, Cursor, ...), with zero OCR dependencies.

一个把「按顺序粘贴的多张文章截图」提取为「单篇结构化 Markdown」的 Agent Skill。流程与宿主无关，任何具备多模态读图能力的 Agent 环境都能运行，零 OCR 依赖。

## Features / 功能

- **Paste order is the only source of truth / 顺序即权威** — assemble strictly in paste order; never reorder, dedupe, or skip pages. File names are never used as a sorting basis.
- **Readability gate / 可读性门禁** — every page is checked for clarity, completeness, obstruction, and confidence before assembly. Any failure → report and STOP. Never guess, never deliver a partial article.
- **Structure reconstruction / 结构化还原** — headings, bold, blockquotes, lists, and tables are rebuilt from visual cues; wording is preserved exactly.
- **Tool-agnostic / 工具无关** — binds to no specific tool name and no OCR library; it only requires the host's native multimodal image-reading ability.
- **Single-file output / 单文件输出** — one article → one `.md` file, named after the article title.

## How it works / 工作流程

1. **Inventory by paste order** — number images 1..N exactly as pasted.
2. **Read every image (read-only)** — extract content into working notes; write nothing yet.
3. **Readability gate** — check every page (clarity / clipped chars / obstruction / confidence). Any failure → error + stop, ask for better images, restart from step 1.
4. **Assemble** — concatenate strictly in order, rebuild structure from visual cues.
5. **Name and save** — filename from the article title; if no target directory was given, **must ask the user** and wait for the answer.
6. **Present** — show the file card and reply with the path + a one-line summary.

## Install / 安装

### Claude Code

```text
/plugin marketplace add JunTan414/screenshot-article-extractor
```

Then: `Browse and install plugins` → `screenshot-article-extractor` → `Install now`.
Or directly: `/plugin install screenshot-article-extractor@screenshot-article-extractor`

### WorkBuddy

```bash
git clone https://github.com/JunTan414/screenshot-article-extractor.git
cp -r screenshot-article-extractor/skills/screenshot-article-extractor ~/.workbuddy/skills/
```

On Windows the user-level skills directory is `C:\Users\<you>\.workbuddy\skills\`.

### Other hosts / 其它宿主

Copy the folder `skills/screenshot-article-extractor/` into your host's skills directory. The host must have multimodal vision — run the porting checklist below before migrating.

## Usage / 使用示例

Paste the screenshots in reading order (any file names are fine — names are never used for ordering) and say:

- "提取文章文字，保存成 md"
- "整理这篇文章到 <目录>"

If you do **not** specify a target directory, the skill will ask you for it first and only then save the file.

## Porting checklist / 迁移预检（换宿主前必读）

1. Model is vision-capable: GPT-4o / Claude 3.5+ / Gemini 1.5+ / Qwen-VL / GLM-4V, or docs state image-input support.
2. The chat UI accepts pasted/uploaded images.
3. The pricing page lists an image-token tier (pure-text pricing means no native vision).
4. The host's file tool declares image support.

If still unsure, run a one-message minimum check: paste a screenshot containing text and ask "What does this image say?" An accurate answer means it passes; an error like "cannot process images" means it does not — stop there.

If the host fails the check: **do NOT silently lower the readability gate or fidelity guarantees.** Either switch to a vision-capable model, or treat the task as a different (OCR-based, lower-fidelity) workflow.

## Hard rules / 硬约束

- **Readability gate is absolute** — any unreadable page → error + stop. No exceptions.
- **Order is sacred** — paste order is reading order; assemble exactly in that order.
- **Fidelity** — extract the original text; never invent, guess, or smooth over content.
- **One file** — always a single Markdown file per article, never one per screenshot.
- **Tool-agnostic** — binds to no specific tool name or OCR library.
- **Ask before saving to an unspecified path** — never silently pick a default directory.

## License / 许可证

[MIT](./LICENSE)
