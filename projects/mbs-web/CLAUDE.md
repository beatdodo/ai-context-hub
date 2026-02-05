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

## Commit 規則

**格式**：
```
(<app>) <type>(<scope>): <description>
```

**app 標示**：
| 標示 | 說明 |
|------|------|
| `(ops)` | 僅影響 mbs-ops |
| `(admin)` | 僅影響 mbs-admin |
| `(warehouse)` | 僅影響 mbs-warehouse |
| `(libs)` | 僅影響共用函式庫 |
| `(ops, admin)` | 影響多個 app |
| 無標示 | 全域設定（eslint, tsconfig 等） |

**type**：
| type | 說明 |
|------|------|
| `feat` | 新功能 |
| `fix` | 修復 bug |
| `refactor` | 重構（不影響功能） |
| `style` | 樣式調整（不影響邏輯） |
| `chore` | 雜務（建置、設定等） |
| `docs` | 文件 |
| `perf` | 效能優化 |

**範例**：
```
(ops) feat(預約行事曆): 支援平板滾動
(warehouse) fix(庫存盤點): 修正數量計算錯誤
(libs) refactor(components): 重構 UserCard 元件
(admin, ops) chore(版本號): 更新至 v26.02.02
chore(eslint): 調整 depConstraints 規則
```

## 架構說明

### 應用程式 (apps/)
- **mbs-admin** - 技術後台
- **mbs-ops** - 商戶/門戶後台
- **mbs-warehouse** - 倉儲後台

Apps 應保持輕量：僅負責路由配置 (Routing)、全域設定 (Config) 與頁面組裝，不應包含複雜的 UI 元件或商業邏輯。

### 共用函式庫 (libs/)
- **@mbs/components** - 共用 UI 元件 (Dumb Components)
- **@mbs/services** - API 呼叫與商業邏輯
- **@mbs/state** - 多 app 共用的狀態管理邏輯（狀態本身各 app 獨立）
- **@mbs/models** - TypeScript 介面與型別定義
- **@mbs/utils** - 工具函式
- **@mbs/interceptors** - HTTP 攔截器
- **@mbs/directives** - 共用指令

**Library 導出原則**：每個 Lib 應透過 `index.ts` 導出 Standalone Component 或 Service，供 Apps 引用。

### 依賴規則 (由 ESLint 強制執行)
- Apps → Libs（Apps 之間不可互相依賴）
- UI → state, service, util, types
- State → util, types（不可依賴 service，只操作內部狀態）
- Service → state, service, util, types
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
- `[(ngModel)]` 已支援 Signal 雙向綁定，直接使用 `[(ngModel)]="mySignal"` 即可，不需要分開寫 `[ngModel]` 和 `(ngModelChange)`

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
│   ├── feature.service.ts          # API 呼叫（處理 HTTP 請求、資料轉換）
│   └── feature-state.service.ts    # 狀態管理（集中管理該功能的 signals）
├── utils/
│   ├── feature-status.util.ts      # 純工具函數（無狀態、無 DI）
│   └── feature-util.service.ts     # 工具 Service（需要 DI 或有命名空間需求）
├── models/
│   └── feature.model.ts            # 介面與型別定義
└── components/
```

**命名規則**：
- API 相關 Service：`xxx.service.ts`
- 狀態管理 Service：`xxx-state.service.ts`
- 純工具函數：`xxx.util.ts`（export function）
- 工具 Service：`xxx-util.service.ts`（@Injectable）
- 統一使用 `utils/` 資料夾（不使用 `helpers/`）

**Service 放置原則**：
- 功能專用的 service（含 `xxx-state.service.ts`）放在該功能模組的 `services/` 內
- 跨 app 共用的 API service 放在 `libs/services`
- 多 app 共用的狀態管理邏輯放在 `libs/state`（不要混在 `libs/services`）

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
- 每個 function 都必須寫註解，說明其用途

### 命名規則

**檔案命名**：
| 類型 | 格式 | 範例 |
|------|------|------|
| Component | `xxx.component.ts` | `user-card.component.ts` |
| API Service | `xxx.service.ts` | `user.service.ts` |
| State Service | `xxx-state.service.ts` | `user-state.service.ts` |
| 工具 Service | `xxx-util.service.ts` | `date-util.service.ts` |
| 純工具函數 | `xxx.util.ts` | `format.util.ts` |
| 介面定義 | `xxx.model.ts` | `user.model.ts` |
| 路由設定 | `xxx.routes.ts` | `user.routes.ts` |

**程式碼命名**：
| 類型 | 格式 | 範例 |
|------|------|------|
| Enum | `EnumXxx` | `EnumStatus`, `EnumRole` |
| Interface | `Xxx`（不加 `I` 前綴） | `User`, `ApiResponse` |
| Type Alias | 語意化名稱，視情況加 `Type` 後綴 | `Status`, `UserIdType` |
| Constant | `UPPER_SNAKE_CASE` | `API_BASE_URL`, `MAX_RETRY` |
| Signal | 語意化名稱（不需加 `Signal` 後綴） | `isLoading`, `userList` |

### 語言
- 使用正體中文撰寫註解與文件

## API 格式與錯誤處理

### 回應格式
後端 API 統一回傳格式為 `{code: number, data: XXX}`，已定義在 `@mbs/models`：
- `APIResponse<T>` - 一般回應
- `APIPaginationResponse<T>` - 含分頁資訊的回應

定義 model 時只需定義 `data` 的格式，避免重複定義外層結構。使用時直接用 `APIResponse<T>` 泛型，不需額外定義 type alias。

**Service 回傳處理**：
- `APIResponse<T>`：透過 `pipe(map(v => v.data))` 取出 data，回傳 `Observable<T>`，讓 Component 不用再多取一層
- `APIPaginationResponse<T>`：照實回傳，因為 Component 需要分頁資訊

```typescript
import { APIResponse, APIPaginationResponse } from '@mbs/models'
import { map } from 'rxjs'

// 一般回應：取出 data 後回傳
getFeature(id: number): Observable<Feature> {
    return this.http.get<APIResponse<Feature>>(`/api/feature/${id}`).pipe(
        map(res => res.data)
    )
}

// 分頁回應：照實回傳，保留分頁資訊
getFeatureList(page: number): Observable<APIPaginationResponse<Feature[]>> {
    return this.http.get<APIPaginationResponse<Feature[]>>(`/api/feature?page=${page}`)
}
```

### 錯誤處理
HTTP Interceptor (`@core/interceptors/http.interceptor.ts`) 會統一處理 API 錯誤回應。

### 預設行為
當 API 回傳非 200 的錯誤碼時，Interceptor 會：
1. 拋出 `HttpErrorResponse`
2. 自動跳出系統 dialog 顯示錯誤訊息（透過 `AppDialogService`）

### 特殊錯誤碼處理
以下錯誤碼有特殊處理邏輯，**不會**跳出系統 dialog：

| 錯誤碼 | 說明 | 處理方式 |
|--------|------|----------|
| `200` | 成功 | 正常回傳 |
| `100012` | 特殊狀態 | 不拋錯誤，正常回傳 |
| `110004` | Token 失效 | 跳 dialog + 登出 |
| `100001` | 連線參數錯誤 | 跳 dialog + 登出 |
| `170003` | 客戶已存在 | 不跳 dialog，由呼叫端自行處理 |

### 呼叫端自行處理錯誤
若需要在呼叫端處理特定錯誤碼（如 `170003`），需：
1. 在 Interceptor 的 `handleError` 中新增該錯誤碼的特例（不跳 dialog）
2. 在 Service 呼叫處的 `error` callback 中判斷 `error.error?.code` 並處理

```typescript
// 範例：處理 170003 客戶已存在
this.service.createClient(request).subscribe({
    next: response => { /* 成功處理 */ },
    error: (error: HttpErrorResponse) => {
        if (error.error?.code === 170003) {
            // 自訂處理邏輯
        }
    },
})
```
