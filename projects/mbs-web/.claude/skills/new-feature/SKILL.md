---
name: new-feature
description: 建立符合專案規範的功能模組目錄結構。使用時機：「建立功能」、「新增 feature」、「create feature module」。
---

# 建立功能模組

根據 MBS 專案規範建立功能模組目錄結構。分為兩種情境：
- **Apps 功能模組**：各 app 獨立的業務功能（如預約管理、客戶管理）
- **Libs 共用模組**：跨 app 共用的 services、components、utils

## 使用方式

```
/new-feature <feature-name>
```

- `<feature-name>`：功能名稱（kebab-case，例如 `appointment`）

## 流程

1. 詢問要建立哪種類型：
   - **Apps 功能模組**：放在 `apps/<app-name>/src/app/<feature>/`
   - **Libs 共用模組**：放在 `libs/<category>/`
2. 若為 Apps 功能模組，詢問結構複雜度：
   - **簡單結構**：單一頁面、少量邏輯，檔案平鋪在 feature/ 底下
   - **完整結構**：多頁面、複雜邏輯，使用子資料夾分類
3. 根據選擇建立對應的目錄結構與基礎檔案

---

## Apps 功能模組結構

適用於各 app 獨立的業務功能，放在 `apps/<app-name>/src/app/<feature>/`

### 簡單結構（單一頁面、少量邏輯）

適用於簡單功能，檔案直接放在 feature/ 底下一層：

```
feature/
├── feature.component.ts
├── feature.component.html
├── feature.component.scss
├── feature.service.ts       # API 呼叫（可選）
└── feature.model.ts         # 介面定義（可選）
```

### 完整結構（多頁面、複雜邏輯）

適用於較複雜的功能模組：

```
feature/
├── services/
│   ├── feature.service.ts          # API 呼叫（HTTP 請求、資料轉換）
│   └── feature-state.service.ts    # 狀態管理（集中管理該功能的 signals）
├── utils/
│   ├── feature-status.util.ts      # 純工具函數（無狀態、無 DI）
│   └── feature-util.service.ts     # 工具 Service（需要 DI）
├── models/
│   └── feature.model.ts            # 介面與型別定義
├── components/
│   └── feature-xxx/                # 功能專用元件
│       ├── feature-xxx.component.ts
│       ├── feature-xxx.component.html
│       └── feature-xxx.component.scss
├── pages/
│   ├── feature-list/               # 列表頁
│   └── feature-detail/             # 詳情頁
└── feature.routes.ts               # 路由設定
```

### feature.service.ts（API 呼叫）

```typescript
import { Injectable, inject } from '@angular/core'
import { HttpClient } from '@angular/common/http'
import { Observable, map } from 'rxjs'
import { APIResponse, APIPaginationResponse } from '@mbs/models'
import { Feature, FeatureRequest } from '../models/feature.model'

@Injectable({ providedIn: 'root' })
export class FeatureService {
    private http = inject(HttpClient)

    /** 取得列表（含分頁，照實回傳） */
    getList(page: number): Observable<APIPaginationResponse<Feature[]>> {
        return this.http.get<APIPaginationResponse<Feature[]>>(`/api/feature?page=${page}`)
    }

    /** 根據 ID 取得單筆資料（取出 data 回傳） */
    getById(id: number): Observable<Feature> {
        return this.http.get<APIResponse<Feature>>(`/api/feature/${id}`).pipe(
            map(res => res.data)
        )
    }

    /** 建立新資料 */
    create(request: FeatureRequest): Observable<Feature> {
        return this.http.post<APIResponse<Feature>>('/api/feature', request).pipe(
            map(res => res.data)
        )
    }

    /** 更新資料 */
    update(id: number, request: FeatureRequest): Observable<Feature> {
        return this.http.put<APIResponse<Feature>>(`/api/feature/${id}`, request).pipe(
            map(res => res.data)
        )
    }

    /** 刪除資料 */
    delete(id: number): Observable<void> {
        return this.http.delete<APIResponse<void>>(`/api/feature/${id}`).pipe(
            map(res => res.data)
        )
    }
}
```

### feature-state.service.ts（狀態管理）

State Service 只操作內部狀態，不呼叫 API。

```typescript
import { Injectable, signal, computed } from '@angular/core'
import { Feature } from '../models/feature.model'

@Injectable({ providedIn: 'root' })
export class FeatureStateService {
    //#region State
    private _list = signal<Feature[]>([])
    private _selected = signal<Feature | null>(null)
    private _isLoading = signal(false)
    //#endregion

    //#region Selectors (readonly)
    readonly list = this._list.asReadonly()
    readonly selected = this._selected.asReadonly()
    readonly isLoading = this._isLoading.asReadonly()
    //#endregion

    //#region Computed
    readonly count = computed(() => this._list().length)
    readonly isEmpty = computed(() => this._list().length === 0)
    //#endregion

    //#region Actions（只操作狀態，不呼叫 API）
    /** 設定列表資料 */
    setList(data: Feature[]): void {
        this._list.set(data)
    }

    /** 設定載入狀態 */
    setLoading(value: boolean): void {
        this._isLoading.set(value)
    }

    /** 選取項目 */
    select(item: Feature | null): void {
        this._selected.set(item)
    }

    /** 重置所有狀態 */
    reset(): void {
        this._list.set([])
        this._selected.set(null)
        this._isLoading.set(false)
    }
    //#endregion
}
```

**Component 使用範例**：
```typescript
export class FeatureListComponent {
    private api = inject(FeatureService)
    private state = inject(FeatureStateService)

    // 暴露 state 給 template
    protected list = this.state.list
    protected isLoading = this.state.isLoading

    /** 載入列表資料 */
    loadList(): void {
        this.state.setLoading(true)
        this.api.getList().subscribe({
            next: response => {
                this.state.setList(response.data)
                this.state.setLoading(false)
            },
            error: () => this.state.setLoading(false),
        })
    }
}
```

### feature.model.ts

```typescript
/** Feature 資料結構 */
export interface Feature {
    id: number
    name: string
    // ...
}

/** 建立/更新 Feature 的請求參數 */
export interface FeatureRequest {
    name: string
    // ...
}

// API 回應直接使用 APIResponse<Feature>、APIPaginationResponse<Feature[]>
// 不需要額外定義 type alias
```

### feature.routes.ts

```typescript
import { Routes } from '@angular/router'

export const featureRoutes: Routes = [
    {
        path: '',
        loadComponent: () =>
            import('./pages/feature-list/feature-list.component').then(
                m => m.FeatureListComponent
            ),
    },
    {
        path: ':id',
        loadComponent: () =>
            import('./pages/feature-detail/feature-detail.component').then(
                m => m.FeatureDetailComponent
            ),
    },
]
```

---

## Libs 共用模組結構

適用於跨 app 共用的程式碼，放在 `libs/` 底下。

```
libs/
├── components/          # 共用 UI 元件（Dumb Components）
│   └── src/lib/
│       ├── user-card/
│       │   ├── user-card.component.ts
│       │   ├── user-card.component.html
│       │   └── user-card.component.scss
│       └── index.ts     # 導出所有元件
│
├── services/            # API 呼叫與商業邏輯（不放狀態）
│   └── src/lib/
│       ├── user.service.ts
│       ├── auth.service.ts
│       └── index.ts
│
├── state/               # 多 app 共用的狀態管理邏輯（狀態本身各 app 獨立）
│   └── src/lib/
│       ├── user-state.service.ts
│       ├── auth-state.service.ts
│       └── index.ts
│
├── models/              # TypeScript 介面與型別
│   └── src/lib/
│       ├── user.model.ts
│       ├── auth.model.ts
│       └── index.ts
│
├── utils/               # 工具函式
│   └── src/lib/
│       ├── date.util.ts
│       ├── format.util.ts
│       └── index.ts
│
├── directives/          # 共用指令
│   └── src/lib/
│       └── index.ts
│
└── interceptors/        # HTTP 攔截器
    └── src/lib/
        └── index.ts
```

### Libs 導出規則

每個 lib 透過 `index.ts` 導出，供 apps 使用路徑別名引用：

```typescript
// libs/services/src/lib/index.ts
export * from './user.service'
export * from './auth.service'

// libs/state/src/lib/index.ts
export * from './user-state.service'
export * from './auth-state.service'

// 在 apps 中引用
import { UserService } from '@mbs/services'
import { UserStateService } from '@mbs/state'
```

---

## 命名規則

| 類型 | 命名 | 範例 |
|------|------|------|
| API Service | `xxx.service.ts` | `appointment.service.ts` |
| State Service | `xxx-state.service.ts` | `appointment-state.service.ts` |
| 純工具函數 | `xxx.util.ts` | `appointment-status.util.ts` |
| 工具 Service | `xxx-util.service.ts` | `appointment-util.service.ts` |
| 介面定義 | `xxx.model.ts` | `appointment.model.ts` |

