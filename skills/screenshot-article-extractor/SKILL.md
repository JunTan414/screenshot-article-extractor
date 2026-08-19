---
name: screenshot-article-extractor
description: "Extract an article's full text from a series of screenshots (clipboard images pasted in order) into a single structured Markdown file. Use when the user pastes multiple sequential screenshots of an article / document / web page (e.g. Clipboard_Screenshot.png, -1, -2, ...) and asks to 提取文章文字、截图转文字、识别截图内容、整理成 md/文章, or to save the extracted text into a target folder. MUST first verify every image is clearly readable; if any image is blurry, cut off, too small, or otherwise unreadable, report an error and STOP - never guess content, never produce a partial file."
agent_created: true
---

# Screenshot Article Extractor

## Overview

Turn an ordered series of article screenshots into one faithful Markdown document:
read every image in order, verify readability, extract the full text, preserve the
original structure (headings, bold, blockquotes, lists, tables), and save as a single
`.md` file in the target directory.

## Workflow

### 1. Inventory the images, by paste order

- The user pastes screenshots in reading order. File names are NOT a reliable
  pattern: they may look like `Clipboard_Screenshot.png`, `Clipboard_Screenshot-1.png`,
  or be completely random. Never sort by file name and never assume a naming rule.
- The paste order is the ONLY source of truth. Number the images 1..N in the
  exact order they appear in the user's message. Never reorder, dedupe, or skip pages.
- Use the file name only as a label to identify a specific image in reports.

### 2. Read every image (read-only pass)

- Use your multimodal vision to view each image, in order. In WorkBuddy this
  is the built-in Read tool; in other agent environments, map this to that
  host's own image-reading capability. This skill never depends on a specific
  tool name or on any OCR library.
- Extract the text content of every page into working notes, page by page.
- Do NOT write any output file yet — assembly happens only after the
  readability check passes for ALL pages.

### 3. Readability gate — check every page (mandatory, blocks everything)

For EACH image, check:

1. **Clarity**: text is sharp and legible; no blur, motion blur, heavy noise,
   or compression artifacts that make strokes ambiguous.
2. **Completeness of characters**: no characters clipped at image edges, no
   lines truncated mid-sentence at the boundary (except where it is clearly a
   natural page break).
3. **No obstruction**: no watermark, overlay, highlight, cursor, or UI element
   covering text.
4. **Confidence**: every character can be read with high confidence. Any
   character that can only be guessed counts as a failure.

If **any** page fails:

- **STOP immediately.** Do not continue reading the rest, do not assemble,
  do not create any `.md` file, do not save anything.
- Report the error in this format:
  - Which image(s) failed, by paste order (e.g. "粘贴顺序第 3 张，文件名
    `Clipboard_Screenshot-2.png`")
  - Why it failed (模糊 / 文字被截断 / 有水印遮挡 / 字号太小 / 无法辨认的字符)
  - The exact positions that are unreadable, if identifiable.
- Ask the user to re-provide the failed image(s) with better quality, then
  restart the whole flow from Step 1.
- Never fill in guessed text, never "best-effort" extract, never deliver a
  partial article.

### 4. Assemble the article (only after ALL pages pass)

- Concatenate the page contents strictly in the given order.
- Reconstruct structure from visual cues:
  - Headings → `#` / `##` / `###` by font size, weight, and numbering.
  - Bold / italic text → `**text**` / `*text*`.
  - Blockquotes → `> `.
  - Bullet / numbered lists → `- ` / `1. `.
  - Tables → GitHub-flavored Markdown tables; keep the exact cell content.
  - Standalone lines / centered text (e.g. title, author) → plain paragraphs.
- Preserve the original wording exactly. Do not summarize, paraphrase, or
  "correct" the author's text. Keep punctuation and paragraph breaks.
- Strip nothing except obvious UI junk (screenshot toolbars, timestamps) that
  is not part of the article body.

### 5. Name and save the file

- Filename: derive from the article title (usually at the top of the first
  image), e.g. `索隐《牛来》——一部借牛喻汉、梦回朱明的"反清悼明"之作.md`.
  Remove characters illegal on Windows (`\ / : * ? " < > |`) from the title.
- Target directory:
  - If the user gave a directory, use it exactly (create it if missing).
  - If NOT given, you MUST ask the user for the target directory first and
    wait for their reply before saving anything. Ask in a clear, direct
    question (e.g. "请指定保存目录") — do not silently pick a default, do not
    guess, do not fall back to any convention, and do not proceed without an
    explicit answer.
- Save as a single `.md` file with UTF-8 encoding.

### 6. Present

- Call present_files with the saved `.md` file so the user can view it.
- Reply with the full path and a one-line summary of what the article contains
  (number of chapters / notable structure, e.g. "共六个章节，含一处表格").

## Porting checklist (before moving to another agent)

Never validate by running the full flow. Verify in this order, cheapest first:

1. **Model is vision-capable**: the host's default/selected model is multimodal
   (e.g. GPT-4o, Claude Sonnet/Opus 3.5+, Gemini 1.5/2.x, Qwen-VL, GLM-4V) or
   its docs state image-input support.
2. **Image input exists**: the chat UI accepts pasted/uploaded images.
3. **Billing has image tokens**: the model's pricing page lists an image-token
   tier; pure-text pricing means no native vision.
4. **File tool reads images**: the host's read-file tool declares image support
   (WorkBuddy's built-in Read does).

Minimum verification (one message, not the whole flow): paste a single
screenshot containing text and ask "What does this image say?" An accurate
answer means the host passes; an error such as "cannot process images" means it
does not — stop there.

If the host fails: do NOT silently lower the readability gate or fidelity
guarantees. Either switch the host to a vision-capable model, or treat the task
as a different (OCR-based, lower-fidelity) workflow.

## Hard rules

- **Readability gate is absolute**: any unreadable page → error + stop. No exceptions.
- **Order is sacred**: images are pasted in reading order; assemble exactly in that order.
- **Fidelity**: extract the original text; never invent, guess, or smooth over content.
- **One file**: always a single Markdown file per article, never one file per screenshot.
- **Tool-agnostic**: the workflow binds to no specific tool or library. "Read"
  means the host agent's multimodal image-reading ability (built-in Read in
  WorkBuddy); other hosts map it to their own equivalent.
- **Ask before saving to an unspecified path**: if the user did not specify a
  target directory, you MUST ask them for it and wait for the answer. Never
  silently pick a default directory, never guess, never assume a convention.
  No file is written until the user answers.
