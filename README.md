# 🧠 AI Context Hub

> **My Personal AI Configuration Registry**
>
> 這是我的 AI 設定檔中央倉庫，統一管理不同模型（Claude, Gemini, OpenAI）的 Context 與 System Prompts。

## 🎯 目的 (Why this repo?)

隨著經手的專案變多，我發現自己在重複撰寫相同的 AI 指令（如 Coding Style、專案架構說明）。建立這個 Repo 是為了：

1.  **Single Source of Truth**：將分散在各處的 Prompt 集中管理，確保我的 AI 助手在不同環境下有一致的「工作習慣」。
2.  **DRY (Don't Repeat Yourself)**：將常用的技能（如 `Refactoring Guide`, `Commit Message 規範`）模組化，新專案直接引用或複製。
3.  **Prompt Versioning**：像管理程式碼一樣管理 Prompt，記錄哪些指令有效、哪些需要迭代。

## 📂 目錄結構 (Directory Map)

```text
.
├── configs/                  # [核心設定] 各大模型的專屬設定檔 (Profiles)
│   ├── _global/              # 通用原則 (我的 Coding Style, 偏好語氣)
│   ├── claude/               # Claude 專用 (含 CLAUDE.md 範本)
│   ├── openai/               # ChatGPT/OpenAI API Custom Instructions
│   └── gemini/               # Gemini System Instructions
│
├── skills/                   # [技能庫] 模組化的能力 (Modular Capabilities)
│   ├── programming/          # (e.g., Python Expert, React Patterns)
│   ├── writing/              # (e.g., Tech Blog, Documentation)
│   └── architecture/         # (e.g., System Design Templates)
│
├── templates/                # [快速啟動] 複製即用的專案包
│   ├── project-init/         # 新專案開局用的 AI 設定懶人包
│   └── role-play/            # 特殊角色設定 (e.g., 資深架構師, Code Reviewer)
│
└── docs/                     # 筆記與心得 (Prompt Engineering Notes)
