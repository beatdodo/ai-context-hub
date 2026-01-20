# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

MBS (Medical Beauty System) 前端專案，採用 Nx Monorepo 架構，包含三個 Angular 20 應用程式共用通用函式庫。

**技術棧**：Angular 20 + PrimeNG 20 + Tailwind CSS 3.4.17 + Nx 22

## 常用指令

```bash
# 開發伺服器
npm run start:admin      # mbs-admin 於 port 4202
npm run start:ops        # mbs-ops 於 port 4203
npm run start:warehouse  # mbs-warehouse 於 port 4204

# 正式環境建置
npm run build:mbs-admin
npm run build:mbs-ops
npm run build:mbs-warehouse

# 程式碼檢查與格式化
npm run fix              # ESLint + Prettier 自動修復
npm run lint             # 僅執行 ESLint 檢查

# Nx 指令
npx nx serve <app-name>
npx nx build <app-name>
npx nx lint <project-name>
npx nx graph             # 視覺化專案依賴關係
```

## 架構說明

### 應用程式 (apps/)
- **mbs-admin** - 技術後台
- **mbs-ops** - 商戶/門戶後台
- **mbs-warehouse** - 倉儲後台

Apps 應保持輕量：僅負責路由配置 (Routing)、全域設定 (Config) 與頁面組裝，不應包含複雜的 UI 元件或商業邏輯。

### 共用函式庫 (libs/)
- **@mbs/components** - 共用 UI 元件 (Dumb Components)
- **@mbs/services** - API 呼叫與商業邏輯狀態管理
- **@mbs/models** - TypeScript 介面與型別定義
- **@mbs/utils** - 工具函式
- **@mbs/interceptors** - HTTP 攔截器
- **@mbs/directives** - 共用指令

**Library 導出原則**：每個 Lib 應透過 `index.ts` 導出 Standalone Component 或 Service，供 Apps 引用。

### 依賴規則 (由 ESLint 強制執行)
- Apps → Libs（Apps 之間不可互相依賴）
- UI → service, util, types
- Service → service, util, types
- Util → types
- Types → 無依賴（最底層）

### 路徑別名 (tsconfig.base.json)
引用 libs 時必須使用路徑別名：
```typescript
import { UserService } from '@mbs/services'
import { User } from '@mbs/models'
import { UserCardComponent } from '@mbs/components'
```
禁止使用相對路徑如 `../../libs/...`

## 開發原則

### 響應式狀態管理
- **畫面上的變數必須使用 `signal()`, `computed()`**
- 所有組件狀態都要使用 Angular 的 signal 系統
- 計算屬性使用 `computed()` 來處理派生狀態
- **禁止**在 `libs` 以外的地方手動訂閱 RxJS（盡量使用 Signal 或 AsyncPipe）

### Angular 20 新語法
- 使用 `@if {}`, `@for {}` 區塊語法（取代 `*ngIf`, `*ngFor`）
- 使用 `input()`, `output()` 函數語法（取代 `@Input()`, `@Output()`）
- 使用 `viewChild()`, `viewChildren()` Signal-based 查詢（取代 `@ViewChild()`, `@ViewChildren()`）
- 全面使用 Standalone Components（不使用 NgModules）
- **禁止**使用 `document.querySelector()` 或 `document.querySelectorAll()`，改用 `viewChild()` 或 `viewChildren()` 保持 Angular 的封裝性與變更偵測機制

### 元件成員宣告順序
元件內的成員應依照以下順序宣告，並使用 `#region` 標註分組：

**有 inject 的元件：**
```typescript
export class XxxComponent {
    //#region Injections & Services
    private serv = inject(XxxService)
    //#endregion

    //#region Enums & Constants
    protected readonly PrivilegeCodes = PrivilegeCodes
    protected readonly EnumXxx = EnumXxx
    //#endregion

    //#region State
    protected xxxSignal = signal<...>(...)
    //#endregion

    //#region Computed
    protected xxx = computed(() => ...)
    //#endregion

    // Lifecycle hooks & Methods
}
```

**無 inject 的元件（Dumb Component）：**
```typescript
export class XxxComponent {
    // Inputs
    value = input.required<string>()

    // Two-way binding
    selected = model<number | null>(null)

    // Outputs
    change = output<void>()

    // Constants & Enums
    protected readonly PrivilegeCodes = PrivilegeCodes
    protected readonly EnumXxx = EnumXxx

    // 內部狀態
    protected xxxSignal = signal<...>(...)

    // Computed & Methods
}
```

### 功能模組目錄結構
各功能模組內部的目錄結構：
```
feature/
├── services/
│   └── feature.service.ts          # API 呼叫（處理 HTTP 請求、資料轉換）
├── utils/
│   ├── feature-status.util.ts      # 純工具函數（無狀態、無 DI）
│   └── feature-util.service.ts     # 工具 Service（需要 DI 或有命名空間需求）
├── models/
│   └── feature.model.ts            # 介面與型別定義
└── components/
```

**命名規則**：
- API 相關 Service：`xxx.service.ts`
- 純工具函數：`xxx.util.ts`（export function）
- 工具 Service：`xxx-util.service.ts`（@Injectable）
- 統一使用 `utils/` 資料夾（不使用 `helpers/`）

### 設計原則
- **單一職責 (SRP)**：每個函數、組件只負責一件事
- 邏輯抽取至 `libs/services`，UI 抽取至 `libs/components`

### 樣式處理
- 搭配 **Tailwind CSS v3.4.17**，`tailwind.config.js` 位於根目錄，所有 Apps 共用設定
- 可以使用 Tailwind Utility Class 就不要自己寫 `.scss`
- 若需客製化樣式，盡量定義在 `libs/components` 內部
- UI 元件優先使用 **PrimeNG 20**

### TypeScript
- 不使用分號（ASI 風格）

### 語言
- 使用正體中文撰寫註解與文件
