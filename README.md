# notebooklm-easy-use

*Chrome extension builds of this script are produced with [browser-script-to-extension](https://github.com/greenzorro/browser-script-to-extension).*

**Batch Update Google Drive Sources in NotebookLM**

Combining NotebookLM with Google Docs, Sheets, and Slides is powerful for long-lived projects—but NotebookLM snapshots sources when added, and there is no built-in “Update All”. This script finds Workspace sources and updates them in one click.

**NotebookLM 快捷更新 Google Drive 资料**

Docs/Sheets/Slides 很适合当长期资料库，但 NotebookLM 添加时会固化快照，界面又没有「全部更新」。本脚本从列表找出这些资料并逐个点更新，一键搞定。

![](https://github.com/user-attachments/assets/8b37ea37-37a5-46a9-b53b-98fd0b00fe27)

https://github.com/user-attachments/assets/782674f3-912f-42a4-bf28-0d53dccf66ce

---

Created by [Victor42](https://victor42.work/) & [Agent Vik](https://github.com/agent-vik)

---

# For Agent

Install this browser enhancement, verify it on NotebookLM, run one batch Drive-source update if the notebook has Google Workspace sources, then hand routine syncing to the human.

This tool runs in the browser. You need a browser tool plus a logged-in Google session for real use.

## Prerequisites

- Chromium-based browser + Tampermonkey or unpacked extension from this repo
- `https://notebooklm.google.com/notebook/*` with at least one Docs/Sheets/Slides source when testing the main flow

## Steps

1. Install:
   - Tampermonkey: root `notebooklm-easy-use.js` or https://greasyfork.org/en/scripts/561819-notebooklm-easy-use, **or**
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
