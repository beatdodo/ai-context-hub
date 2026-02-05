---
name: new-feature
description: 建立符合專案規範的功能模組目錄結構。使用時機：「建立功能」、「新增 feature」、「create feature module」。
---

# 建立功能模組

根據專案規範建立功能模組目錄結構，放在 `src/app/pages/<feature>/`。

## 使用方式

```
/new-feature <feature-name>
```

- `<feature-name>`：功能名稱（kebab-case，例如 `gift-card`）

## 流程

1. 詢問結構複雜度：
   - **簡單結構**：單一頁面、少量邏輯，檔案平鋪在 feature/ 底下
   - **完整結構**：多頁面、複雜邏輯，使用子資料夾分類
2. 根據選擇建立對應的目錄結構與基礎檔案

---

## 簡單結構（單一頁面、少量邏輯）

適用於簡單功能，檔案直接放在 feature/ 底下一層：

```
feature/
├── feature.component.ts
├── feature.component.html
├── feature.component.scss
├── feature.service.ts       # API 呼叫（可選）
└── feature.model.ts         # 介面定義（可選）
```

---

## 完整結構（多頁面、複雜邏輯）

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

---

## 檔案範本

### feature.service.ts（API 呼叫）

```typescript
import { Injectable, inject } from '@angular/core'
import { HttpClient } from '@angular/common/http'
import { Observable, map } from 'rxjs'
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

## 命名規則

| 類型 | 命名 | 範例 |
|------|------|------|
| API Service | `xxx.service.ts` | `gift-card.service.ts` |
| State Service | `xxx-state.service.ts` | `gift-card-state.service.ts` |
| 純工具函數 | `xxx.util.ts` | `gift-card-status.util.ts` |
| 工具 Service | `xxx-util.service.ts` | `gift-card-util.service.ts` |
| 介面定義 | `xxx.model.ts` | `gift-card.model.ts` |
