# notebooklm-easy-use

[📃 Tampermonkey Script](https://greasyfork.org/en/scripts/561819-notebooklm-easy-use)

[🌐 Chrome Web Store](https://chrome.google.com/webstore/detail/ihlfmnkaohiafbaoaiebjnjnnkelbali) | [🗄️ Download ZIP](https://github.com/greenzorro/notebooklm-easy-use/raw/main/notebooklm-easy-use.zip)

*Extension automatically created with [browser-script-to-extension](https://github.com/greenzorro/browser-script-to-extension)*

**Batch Update Google Drive Sources in NotebookLM**

Combining NotebookLM with Google Docs, Sheets, and Slides is a game-changer, as it allows the tool to function as a natural language interface for databases in long-term projects. 

The catch is that NotebookLM captures a static snapshot of your files when they are first added. Syncing updates requires a manual process, and since there is no "Update All" feature, you’re forced to click into each source's settings individually—a massive time sink.

This script automates that workflow. It scans your source list for all Google Workspace files and triggers an update for each one, handling the entire process in a single click.

**NotebookLM 快捷更新Google Drive资料内容**

NotebookLM 和 Google Docs/Sheets/Slides 配合使用有奇效，因为这些文档的更新，可以反映到 NotebookLM 中。这使得 NotebookLM 可以被当做自然语言的数据库检索工具，用于需要持续更新数据的长期项目。

但是 NotebookLM 实际上会在添加资料时把当时的内容固化下来。资料的更新，在 NotebookLM 这边不手动操作无法同步过来。NotebookLM 的界面上没有全部更新的功能，更新按钮在资料的详情里，一个个点进去非常耗费精力。

于是有了这个脚本，从资料列表中找出所有 Docs/Sheets/Slides 资料，逐个进入详情更新。全程一键搞定。

![](https://github.com/user-attachments/assets/8b37ea37-37a5-46a9-b53b-98fd0b00fe27)

Video below:

https://github.com/user-attachments/assets/782674f3-912f-42a4-bf28-0d53dccf66ce

---

Created by [Victor42](https://victor42.work/) & [Agent Vik](https://github.com/agent-vik)

# For Agent

Install this browser enhancement, verify it on NotebookLM, run one batch Drive-source update if the notebook has Google Workspace sources, then hand routine syncing to the human.

This tool runs in the browser. You need a browser tool plus a logged-in Google session for real use.

## Prerequisites

- Chromium-based browser + Tampermonkey or unpacked extension from this repo
- `https://notebooklm.google.com/notebook/*` with at least one Docs/Sheets/Slides source when testing the main flow

## Steps

1. Install:
   - Tampermonkey: root `notebooklm-easy-use.js` (or GreasyFork), **or**
   - Load `notebooklm-easy-use.zip` / unpacked `extension/` via `chrome://extensions`
2. **One handoff:** human approves install + Google login if needed.
3. Open a notebook URL matching `@match`. Confirm the script UI / batch-update control is present.
4. Trigger the one-click update for Google Workspace sources; wait until sources show as refreshed (or equivalent success signal in the UI).
5. Stop. Recurring “update all sources” clicks are the human’s habit loop.

## Hand off to the human

- Choosing which notebooks matter
- Google login and NotebookLM access permissions

## Red lines

- No credential harvesting
- Stay on NotebookLM notebook URLs (`@match` above)
- Selector drift → `notes.md`; no drive-by full rewrite of the userscript unless asked
- Do not republish the extension unless asked
