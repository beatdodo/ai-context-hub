---
name: review-angular
description: 審查 Angular 程式碼是否符合 MBS 專案開發原則。使用時機：「審查程式碼」、「review code」、「檢查 Angular」。
---

# Angular 程式碼審查

根據 MBS 專案的開發原則審查 Angular 程式碼。

## 使用方式

```
/review-angular <file-or-pattern>
```

- `<file-or-pattern>`：要審查的檔案路徑或 glob pattern（例如 `src/**/*.ts`）

若未提供，會詢問要審查哪些檔案。

## 審查項目

### 1. 響應式狀態管理

- [ ] 畫面變數是否使用 `signal()`, `computed()`
- [ ] 是否避免在 libs 以外手動訂閱 RxJS
- [ ] 派生狀態是否使用 `computed()`

```typescript
// ❌ 錯誤
isLoading = false
items: Item[] = []

// ✅ 正確
isLoading = signal(false)
items = signal<Item[]>([])
```

### 2. Angular 20 新語法

- [ ] 使用 `@if {}`, `@for {}` 區塊語法
- [ ] 使用 `input()`, `output()` 函數語法
- [ ] 使用 `viewChild()`, `viewChildren()` 查詢

```typescript
// ❌ 錯誤
@Input() value: string
@Output() change = new EventEmitter()
@ViewChild('ref') ref: ElementRef

// ✅ 正確
value = input<string>()
change = output<void>()
ref = viewChild<ElementRef>('ref')
```

```html
<!-- ❌ 錯誤 -->
<div *ngIf="condition">...</div>
<div *ngFor="let item of items">...</div>

<!-- ✅ 正確 -->
@if (condition) { <div>...</div> }
@for (item of items; track item.id) { <div>...</div> }
```

### 3. 元件成員順序

**有 inject：**
1. Injections & Services
2. Enums & Constants
3. State (signals)
4. Computed
5. Lifecycle hooks & Methods

**無 inject（Dumb Component）：**
1. Inputs
2. Two-way binding (model)
3. Outputs
4. Constants & Enums
5. 內部狀態
6. Computed & Methods

### 4. 禁止事項

- [ ] 禁止使用 `document.querySelector()` / `document.querySelectorAll()`
- [ ] 禁止使用相對路徑引用 libs（應用 `@mbs/xxx`）
- [ ] 禁止使用 NgModules（應用 Standalone Components）
- [ ] 禁止使用分號（ASI 風格）

### 5. 樣式規範

- [ ] 優先使用 Tailwind CSS utility classes
- [ ] UI 元件優先使用 PrimeNG
- [ ] 客製化樣式定義在 libs/components 內部

### 6. 路徑別名

```typescript
// ❌ 錯誤
import { UserService } from '../../libs/services/user.service'

// ✅ 正確
import { UserService } from '@mbs/services'
```

## 輸出格式

審查結果以 `file:line` 格式輸出：

```
src/app/user/user.component.ts:15 - 使用舊語法 @Input()，應改用 input()
src/app/user/user.component.ts:23 - 使用 *ngIf，應改用 @if {}
src/app/user/user.component.html:8 - 缺少 @for 的 track 表達式
```

## 嚴重程度

- 🔴 **Error**：必須修正（禁止事項、錯誤語法）
- 🟡 **Warning**：建議修正（不符合最佳實踐）
- 🔵 **Info**：可選改進（程式碼風格建議）
