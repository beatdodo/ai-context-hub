# Nx Monorepo + Angular 20 開發指南 (Project: mbs-web)

## 開發原則

### 1. 響應式狀態管理
- **畫面上的變數必須使用 signal, computed**
- 所有組件狀態都要使用 Angular 的 signal 系統
- 計算屬性使用 computed 來處理派生狀態
- **禁止**在 `libs` 以外的地方手動訂閱 RxJS (盡量使用 Signal 或 AsyncPipe)

### 2. Nx 架構分層原則 (Monorepo)
- **Apps (`apps/`) 應保持輕量**
    - `apps/mbs-admin`, `apps/mbs-ops` 僅負責路由配置 (Routing)、全域設定 (Config) 與頁面組裝。
    - 不要在 `apps` 內直接撰寫複雜的 UI 組件或商業邏輯。
- **Libs (`libs/`) 負責核心邏輯**
    - **`libs/components`**: 共用的 UI 元件 (Dumb Components)。
    - **`libs/services`**: API 呼叫與商業邏輯狀態管理。
    - **`libs/models`**: TypeScript 介面與型別定義。
    - **`libs/utils`**: 共用工具函式。
    - **`libs/interceptors`**: HTTP 攔截器。
- **依賴方向**：Apps -> Libs。Apps 之間不可互相引用。

### 3. 控制流程語法
- **所有 `ngIf`, `ngFor` 皆由新語法 `@if {}`, `@for {}` 取代**
- 必須使用 Angular 20+ 的新控制流程語法 (Block Syntax)

### 4. 組件通訊
- **所有 `@Input`, `@Output` 皆由新語法 `input()`, `output()` 取代**
- 使用函數式的輸入輸出定義方式，以獲得更佳的型別推斷

### 5. 模板引用
- **所有 `@ViewChild()` 皆由新語法 `viewchild()` 取代**
- 透過 Signal-based query 獲取元素參照

### 6. 組件架構
- **全域使用 Standalone Component**
- 不使用 NgModule
- **Library 導出原則**：每個 Lib 應透過 `index.ts` 導出 Standalone Component 或 Service，供 Apps 引用。

### 7. 程式碼風格
- **提供的 TypeScript 不需要任何分號 `;`**
- 保持程式碼簡潔，依賴 TypeScript 自動分號插入 (ASI)
- **路徑引用**：引用 `libs` 內容時，必須使用 `tsconfig.base.json` 定義的路徑別名 (如 `@mbs/components`)，嚴禁使用相對路徑 `../../libs/...`。

### 8. 設計原則
- **單一職責 (SRP)**
- 每個函數、組件只負責一件事
- 邏輯抽取至 `libs/services`，UI 抽取至 `libs/components`

### 9. 樣式處理
- **搭配 Tailwind CSS v3.4.17**
- `tailwind.config.js` 位於根目錄，所有 Apps 共用設定
- 可以使用 Tailwind Utility Class 就不要自己寫 `.scss`
- 若需客製化樣式，盡量定義在 `libs/components` 內部

### 10. 溝通語言
- **使用正體中文回答**
- 所有說明和註解都使用繁體中文

## 範例程式碼結構

### 情境：在 Admin App 中使用共用組件與服務

```typescript
// 1. 定義共用介面 (libs/models/src/lib/user.interface.ts)
export interface User {
  id: number
  name: string
  role: 'admin' | 'ops'
}

// 2. 定義共用服務 (libs/services/src/lib/user.service.ts)
import { Injectable, signal } from '@angular/core'
import { User } from '@mbs/models' // 使用 Alias 引用

@Injectable({ providedIn: 'root' })
export class UserService {
  currentUser = signal<User | null>(null)
  
  updateUser(user: User) {
    this.currentUser.set(user)
  }
}

// 3. 定義共用 UI 組件 (libs/components/src/lib/user-card.component.ts)
import { Component, input, output } from '@angular/core'
import { User } from '@mbs/models'

@Component({
  selector: 'mbs-user-card',
  standalone: true,
  template: `
    <div class="p-4 border rounded shadow-sm">
      <h3 class="font-bold">{{ user().name }}</h3>
      <button (click)="onEdit.emit(user())" class="btn-primary">編輯</button>
    </div>
  `
})
export class UserCardComponent {
  // Angular 20 Signal Inputs
  user = input.required<User>()
  onEdit = output<User>()
}

// 4. App 頁面組裝 (apps/mbs-admin/src/app/app.component.ts)
import { Component, inject } from '@angular/core'
import { UserCardComponent } from '@mbs/components' // 從 Lib 匯入
import { UserService } from '@mbs/services'         // 從 Lib 匯入

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [UserCardComponent], // 直接導入 Standalone Component
  template: `
    <div class="container mx-auto p-4">
      @if (userService.currentUser(); as user) {
        <mbs-user-card 
          [user]="user" 
          (onEdit)="handleEdit($event)"
        />
      } @else {
        <p>載入中...</p>
      }
    </div>
  `
})
export class AppComponent {
  userService = inject(UserService)

  handleEdit(user: any) {
    console.log('Editing', user)
  }
}

## 注意事項

- 遵循這些指南可以充分利用 Angular 20 的新特性
- 確保程式碼的現代化和效能最佳化
- 保持程式碼的簡潔性和可讀性


你必須在回答前先進行「事實檢查思考」(fact-check thinking)。
除非使用者明確提供、或資料中確實存在，否則不得假設、推測或自行創造內容。

具體規則如下：

1. **嚴格依據來源**
    - 僅使用使用者提供的內容、你內部明確記載的知識、或經明確查證的資料。
    - 若資訊不足，請直接說明「沒有足夠資料」或「我無法確定」，不要臆測。

2. **顯示思考依據**
    - 若你引用資料或推論，請說明你依據的段落或理由。
    - 若是個人分析或估計，必須明確標註「這是推論」或「這是假設情境」。

3. **避免裝作知道**
    - 不可為了讓答案完整而「補完」不存在的內容。
    - 若遇到模糊或不完整的問題，請先回問確認或提出選項，而非自行決定。

4. **保持語意一致**
    - 不可改寫或擴大使用者原意。
    - 若你需要重述，應明確標示為「重述版本」，並保持語義對等。

5. **回答格式**
    - 若有明確資料：回答並附上依據。
    - 若無明確資料：回答「無法確定」並說明原因。
    - 不要在回答中使用「應該是」「可能是」「我猜」等模糊語氣，除非使用者要求。

6. **思考深度**
    - 在產出前，先檢查答案是否：
      a. 有清楚依據  
      b. 未超出題目範圍  
      c. 沒有出現任何未被明確提及的人名、數字、事件或假設

最終原則：**寧可空白，不可捏造。**
