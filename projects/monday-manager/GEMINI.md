# Angular 19 開發指南

## 開發原則

### 1. 響應式狀態管理
- **畫面上的變數必須使用 signal, computed**
- 所有組件狀態都要使用 Angular 的 signal 系統
- 計算屬性使用 computed 來處理派生狀態

### 2. 控制流程語法
- **所有 `ngIf`, `ngFor` 皆由新語法 `@if {}`, `@for {}` 取代**
- 使用 Angular 19+ 的新控制流程語法
- 提供更好的類型安全和效能

### 3. 組件通訊
- **所有 `@Input`, `@Output` 皆由新語法 `input()`, `output()` 取代**
- 使用函數式的輸入輸出定義方式
- 提供更好的類型推斷

### 4. 模板引用
- **所有 `@ViewChild()` 皆由新語法 `viewchild()` 取代**
- 使用新的 viewchild 函數來獲取模板引用

### 5. 組件架構
- **不使用 NgModule，全部使用 standalone component**
- 組件獨立，不依賴 NgModule 系統
- 更好的樹搖和按需載入

### 6. 程式碼風格
- **提供的 TypeScript 不需要任何分號 `;`**
- 保持程式碼簡潔
- 依賴 TypeScript 的自動分號插入

### 7. 設計原則
- **程式碼保持單一職責**
- 每個函數、組件只負責一件事
- 提高可讀性和可維護性

### 8. 樣式處理
- **搭配 `"tailwindcss": "^3.4.17"`**
- 可以使用 Tailwind CSS 就不要自己寫 style
- 提高樣式的一致性和維護性

### 9. 溝通語言
- **使用正體中文回答**
- 所有說明和註解都使用繁體中文

## 範例程式碼結構

```typescript
// 組件範例
import { Component, signal, computed, input, output, viewchild } from '@angular/core'

@Component({
  selector: 'app-example',
  standalone: true,
  template: `
    <div class="p-4 bg-white rounded-lg shadow-md">
      @if (isVisible()) {
        <h2 class="text-xl font-bold mb-4">{{ title() }}</h2>
        @for (item of items(); track item.id) {
          <div class="mb-2 p-2 bg-gray-100 rounded">
            {{ item.name }}
          </div>
        }
      }
    </div>
  `
})
export class ExampleComponent {
  // 使用 input() 取代 @Input()
  title = input<string>('預設標題')
  items = input<any[]>([])
  
  // 使用 output() 取代 @Output()
  itemClick = output<any>()
  
  // 使用 signal 管理狀態
  isVisible = signal(true)
  
  // 使用 computed 處理派生狀態
  itemCount = computed(() => this.items().length)
  
  // 使用 viewchild() 取代 @ViewChild()
  containerRef = viewchild<ElementRef>('container')
}
```

## 注意事項

- 遵循這些指南可以充分利用 Angular 19 的新特性
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
