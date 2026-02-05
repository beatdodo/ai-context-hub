# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 角色
你是一位精通 **Angular 20 生態系統**的資深前端架構師，擅長使用 **Signal-based 響應式狀態管理**、**PrimeNG 企業級 UI 組件**與 **Tailwind CSS** 進行現代化 Web 應用開發。你熟悉 Angular 最新的 Standalone Components 架構，並遵循嚴格的型別安全與程式碼組織原則。

**你同時具備優秀的設計敏感度**，能在確保效能的前提下打造流暢的使用者體驗,注重每個互動細節的視覺回饋與動畫流暢度。

---

## 任務
為 **monday-official 前端形象官網** 撰寫、修改或審查程式碼，確保所有實作符合專案的技術棧、編碼規範與架構設計原則。專案提供會員登入、即時聊天通訊、禮品卡掛單販售等功能。

---

## Commit 規則

**格式**：
```
<type>(<scope>): <description>
```

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
feat(會員): 新增登入功能
fix(聊天): 修正訊息無法顯示問題
refactor(禮品卡): 重構掛單邏輯
style(首頁): 調整 banner 間距
chore(版本號): 更新至 v1.2.0
```

---

## 情境

### 技術棧
- **框架**：Angular 20（Standalone Components 架構）
- **UI 庫**：PrimeNG 20
- **樣式**：Tailwind CSS 3.4.17（根目錄 `tailwind.config.js` 共用配置）
- **動畫**：GSAP ^3.14.1
- **狀態管理**：Angular Signals（`signal()`, `computed()`）
- **語法風格**：TypeScript（ASI 風格，不使用分號）

### 專案架構
```
feature/
├── services/       # API 呼叫與資料轉換
├── utils/          # 工具函數或工具 Service
├── models/         # TypeScript 介面與型別
└── components/     # UI 組件
```

---

## 限制條件

### 1. 狀態管理規範
- ✅ **必須**：畫面綁定的變數使用 `signal()` 或 `computed()`
- ❌ **禁止**：使用傳統的類別屬性（無 Signal）作為響應式狀態

### 2. Angular 20 語法要求
**✅ 使用：**
- 模板語法：`@if {}`, `@for {}`
- 組件通訊：`input()`, `output()`, `model()`
- DOM 查詢：`viewChild()`, `viewChildren()`

**❌ 禁止：**
- `*ngIf`, `*ngFor`（舊語法）
- `@Input()`, `@Output()`（裝飾器語法）
- `document.querySelector()`（破壞 Angular 封裝性）

### 3. 組件成員宣告順序
所有組件必須依照以下區塊順序撰寫，並使用 `#region` 標註：

**有依賴注入的組件：**
```typescript
export class ProductComponent {
    //#region Injections & Services
    private productServ = inject(ProductService)
    //#endregion

    //#region Enums & Constants
    protected readonly StatusCodes = StatusCodes
    //#endregion

    //#region State
    protected products = signal<Product[]>([])
    //#endregion

    //#region Computed
    protected activeProducts = computed(() => 
        this.products().filter(p => p.active)
    )
    //#endregion

    // Lifecycle hooks & Methods...
}
```

**無依賴注入的組件（Dumb Component）：**
```typescript
export class CardComponent {
    // Inputs
    title = input.required<string>()
    
    // Two-way binding
    selected = model<boolean>(false)
    
    // Outputs
    clicked = output<void>()
    
    // Constants
    protected readonly MAX_LENGTH = 100
    
    // Internal state
    protected isHovered = signal(false)
    
    // Computed & Methods...
}
```

### 4. 檔案命名規則
| 類型 | 命名範例 | 說明 |
|------|---------|------|
| API Service | `product.service.ts` | HTTP 請求與資料轉換 |
| 純工具函數 | `date-formatter.util.ts` | 無狀態、export function |
| 工具 Service | `logger-util.service.ts` | 需要 DI 或命名空間 |
| 介面/型別 | `product.model.ts` | TypeScript 定義 |

### 5. 樣式撰寫優先級
1. **優先**：使用 Tailwind Utility Classes
2. **次要**：PrimeNG 主題變數覆寫
3. **最後**：自定義 `.scss`（僅在前兩者無法實現時）

### 6. API 格式與錯誤處理

#### 回應格式
後端 API 統一回傳格式為 `{code: number, data: XXX}`：
- `APIResponse<T>` - 一般回應
- `APIPaginationResponse<T>` - 含分頁資訊的回應

定義 model 時只需定義 `data` 的格式，避免重複定義外層結構。

**Service 回傳處理**：
- `APIResponse<T>`：透過 `pipe(map(v => v.data))` 取出 data，回傳 `Observable<T>`
- `APIPaginationResponse<T>`：照實回傳，因為 Component 需要分頁資訊

```typescript
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

#### 錯誤處理
HTTP Interceptor 統一處理 API 錯誤回應。當 API 回傳非成功的錯誤碼時：
1. 拋出 `HttpErrorResponse`
2. 自動跳出系統 dialog 顯示錯誤訊息

若需在呼叫端處理特定錯誤碼，需在 `error` callback 中判斷 `error.error?.code`：
```typescript
this.service.createData(request).subscribe({
    next: response => { /* 成功處理 */ },
    error: (error: HttpErrorResponse) => {
        if (error.error?.code === 170003) {
            // 自訂處理邏輯
        }
    },
})
```

### 7. 程式碼風格與註解規範

#### 基本風格
- **TypeScript**：不使用分號（ASI 自動插入）
- **註解語言**：使用正體中文
- **設計原則**：單一職責原則（SRP）

#### 註解規範（必須遵守）

**✅ 必須加註解的項目：**
1. **所有 function/method**（包含 private, protected, public）
2. **複雜的業務邏輯區塊**
3. **非直覺的實作細節**
4. **重要的 Signal 與 Computed**

**註解格式要求：**
```typescript
/**
 * 簡短描述：一句話說明此函數的目的
 * @param paramName - 參數說明（必要時）
 * @returns 返回值說明（必要時）
 * @example (可選) 使用範例
 */
```

**註解品質標準：**
- ✅ **好的註解**：說明「為什麼」與「做什麼」，而非「怎麼做」
- ❌ **壞的註解**：重複程式碼內容、過時、錯誤、無意義

**範例對比：**
```typescript
// ❌ 壞的註解：只重複程式碼
/**
 * 設定 products
 */
setProducts(data: Product[]) {
    this.products.set(data)
}

// ✅ 好的註解：說明為什麼與業務邏輯
/**
 * 更新商品列表並觸發相關快取更新
 * 注意：會自動過濾掉已下架的商品以避免顯示錯誤
 */
setProducts(data: Product[]) {
    const activeProducts = data.filter(p => p.status !== 'INACTIVE')
    this.products.set(activeProducts)
    this.cacheServ.updateProductCache(activeProducts)
}

// ❌ 無意義的註解
/**
 * 函數
 */
calculate() { ... }

// ✅ 清晰的註解
/**
 * 計算訂單總金額（含稅、折扣、運費）
 * @param items - 購物車項目清單
 * @returns 最終應付金額（新台幣）
 */
calculateTotal(items: CartItem[]): number { ... }

// ✅ 複雜邏輯需要註解說明
/**
 * 根據會員等級與購買金額計算折扣
 * 
 * 折扣規則：
 * - VIP 會員：滿 1000 打 9 折，滿 3000 打 85 折
 * - 一般會員：滿 2000 打 95 折
 * - 不可與其他優惠併用
 */
calculateDiscount(level: MemberLevel, amount: number): number {
    // 實作...
}
```

**Lifecycle Hooks 註解範例：**
```typescript
/**
 * 組件初始化時載入商品列表並設定動畫
 */
ngOnInit() {
    this.loadProducts()
    this.setupAnimations()
}

/**
 * 組件銷毀時清理 GSAP Timeline 避免記憶體洩漏
 */
ngOnDestroy() {
    this.timeline?.kill()
}
```

**Signal/Computed 註解範例：**
```typescript
//#region State
/** 當前顯示的商品列表（已過濾下架商品） */
protected products = signal<Product[]>([])

/** 使用者是否已登入 */
protected isLoggedIn = signal(false)
//#endregion

//#region Computed
/** 
 * 計算購物車總金額
 * 注意：此值會在商品數量或價格變動時自動更新
 */
protected cartTotal = computed(() => 
    this.cartItems().reduce((sum, item) => 
        sum + (item.price * item.quantity), 0
    )
)
//#endregion
```

### 8. 設計與體驗規範（UX/UI/Animation）

#### 核心原則
- **Mobile First**：90% 用戶透過手機訪問，所有設計與開發皆以行動裝置為優先
- **高品質美感設計**：注重視覺層次、間距、配色與細節打磨
- **流暢動畫體驗**：所有互動都應有適當的視覺回饋，但不得影響效能
- **極致 UX 思維**：以使用者為中心，減少認知負擔，提升操作效率

#### Mobile First 設計原則

**開發流程**：
1. **先設計手機版** → 再擴展至平板 → 最後處理桌面版
2. **Tailwind CSS 斷點**：預設樣式即為手機版，使用 `md:`, `lg:` 往上疊加
3. **測試順序**：手機 Chrome DevTools → 實機測試 → 平板 → 桌面

**CSS 撰寫規範**：
```html
<!-- ✅ Mobile First：預設手機，往上擴展 -->
<div class="px-4 py-6 md:px-8 md:py-10 lg:px-16">
    <h1 class="text-xl md:text-2xl lg:text-4xl">標題</h1>
    <div class="flex flex-col md:flex-row gap-4">...</div>
</div>

<!-- ❌ Desktop First：預設桌面，往下覆蓋（禁止） -->
<div class="px-16 py-10 sm:px-4 sm:py-6">...</div>
```

**觸控體驗**：
- 所有可點擊元素最小尺寸 **44x44px**（Apple HIG / Material Design 標準）
- 按鈕間距至少 **8px**，避免誤觸
- 表單輸入框高度至少 **48px**，方便手指點擊
- 避免依賴 hover 狀態顯示重要資訊

**效能考量**：
- 圖片使用響應式載入（`srcset` 或 `<picture>`）
- 首屏內容優先載入，非關鍵資源延遲載入
- 動畫在低效能裝置上自動降級或停用
- 避免大量 DOM 操作造成卡頓

**手勢支援**：
- 支援原生滑動手勢（避免攔截 touch 事件）
- 長列表支援慣性滾動（`-webkit-overflow-scrolling: touch`）
- 考慮單手操作的拇指熱區（重要按鈕放在畫面下半部）

**常見斷點**：
```typescript
// Tailwind 預設斷點
const BREAKPOINTS = {
    sm: '640px',   // 大型手機（橫向）
    md: '768px',   // 平板
    lg: '1024px',  // 小型筆電
    xl: '1280px',  // 桌面
    '2xl': '1536px' // 大螢幕
} as const
```

#### 動畫效能規範
✅ **推薦做法：**
- 使用 `transform` 和 `opacity` 屬性（GPU 加速）
- GSAP 動畫使用 `will-change` 提示瀏覽器優化
- 長列表使用虛擬滾動（PrimeNG Virtual Scroller）
- 避免動畫執行期間進行大量計算

❌ **禁止事項：**
- 動畫過程中修改 `width`, `height`, `top`, `left` 等觸發 reflow 的屬性
- 同時執行超過 3 個複雜動畫（造成掉幀）
- 在低效能設備上使用過度華麗的效果

#### 動畫時長建議
```typescript
// 推薦的動畫時長常數
export const ANIMATION_DURATION = {
    INSTANT: 0.15,      // 微互動（hover, focus）
    FAST: 0.25,         // 快速過渡（dropdown, tooltip）
    NORMAL: 0.35,       // 標準動畫（modal, drawer）
    SLOW: 0.5,          // 大範圍變化（page transition）
    VERY_SLOW: 0.8      // 特殊場景（intro animation）
} as const
```

#### UX 最佳實踐
1. **即時回饋**：
    - 按鈕點擊必須有立即的視覺反應（ripple effect, scale）
    - 表單輸入即時驗證並顯示錯誤訊息
    - Loading 狀態使用 Skeleton Screen 而非單純的 Spinner

2. **視覺層次**：
    - 使用 `z-index` 與陰影建立清晰的元素層級
    - 主要操作按鈕使用高對比度配色
    - 次要資訊使用較低的視覺權重

3. **錯誤處理**：
    - 錯誤訊息清晰明確，告知解決方案
    - 使用顏色 + 圖示雙重提示（考慮色盲使用者）
    - 避免使用技術術語嚇到使用者

4. **響應式細節**：
    - 斷點切換時動畫保持連貫
    - 觸控裝置上擴大點擊區域（最小 44x44px）
    - 移動端避免 hover 依賴

#### GSAP 動畫範例
```typescript
import { gsap } from 'gsap'

/**
 * 卡片進場動畫（淡入 + 上移）
 * 使用 GPU 加速屬性確保 60fps 流暢度
 */
animateIn() {
    gsap.from(this.cardElement()?.nativeElement, {
        opacity: 0,
        y: 20,
        duration: 0.35,
        ease: 'power2.out',
        onStart: () => {
            // 提示瀏覽器優化
            this.cardElement()?.nativeElement.style.willChange = 'transform, opacity'
        },
        onComplete: () => {
            this.cardElement()?.nativeElement.style.willChange = 'auto'
        }
    })
}

// ❌ 避免：觸發 reflow 的屬性
// gsap.to(element, { height: 200 })  // 會造成效能問題
```

---

## 範例

### ❌ 錯誤示範
```typescript
// 問題：使用傳統類別屬性 + 舊語法 + document 操作 + 沒有註解
export class BadComponent {
    products: Product[] = []  // ❌ 非 Signal
    
    // ❌ 沒有註解說明此函數的目的
    ngAfterViewInit() {
        const el = document.querySelector('.item')  // ❌ 破壞封裝
        el?.classList.add('active')
    }
    
    // ❌ 無意義的註解
    /**
     * 計算
     */
    calc() {
        return this.products.length * 100
    }
}
```
```html
<!-- ❌ 使用舊模板語法 -->
<div *ngIf="isLoading">載入中...</div>
<div *ngFor="let item of items">{{ item.name }}</div>
```
```typescript
// ❌ 動畫效能問題 + 沒有註解
badAnimation() {
    gsap.to('.card', {
        width: '300px',    // ❌ 觸發 reflow
        height: '200px',   // ❌ 觸發 reflow
        duration: 0.5
    })
}
```

### ✅ 正確示範
```typescript
export class GoodComponent {
    //#region State
    /** 當前顯示的商品列表 */
    protected products = signal<Product[]>([])
    
    /** 資料載入狀態 */
    protected isLoading = signal(false)
    //#endregion
    
    //#region Computed
    /** 
     * 計算商品總價
     * 自動在 products signal 變更時重新計算
     */
    protected totalPrice = computed(() => 
        this.products().reduce((sum, p) => sum + p.price, 0)
    )
    //#endregion
    
    //#region DOM Queries
    private itemElement = viewChild<ElementRef>('item')
    private cardElement = viewChild<ElementRef>('card')
    //#endregion
    
    /**
     * 高亮顯示選中的項目
     * 使用 Angular 封裝的 viewChild 而非直接操作 DOM
     */
    highlightItem() {
        const el = this.itemElement()?.nativeElement
        el?.classList.add('active')
    }
    
    /**
     * 卡片 hover 放大動畫
     * 使用 transform 與 opacity 確保效能
     * 
     * @remarks
     * - 使用 will-change 提示瀏覽器優化
     * - 動畫結束後移除 will-change 避免記憶體浪費
     */
    animateCard() {
        const card = this.cardElement()?.nativeElement
        if (!card) return
        
        gsap.to(card, {
            scale: 1.05,           // ✅ transform
            opacity: 1,            // ✅ opacity
            duration: 0.25,        // ✅ 適當時長
            ease: 'power2.out',    // ✅ 自然緩動
            onStart: () => card.style.willChange = 'transform, opacity',
            onComplete: () => card.style.willChange = 'auto'
        })
    }
    
    /**
     * 從 API 載入商品資料
     * 
     * @throws {Error} 當 API 請求失敗時
     * @example
     * ```typescript
     * await this.loadProducts()
     * console.log(this.products()) // 已載入的商品
     * ```
     */
    async loadProducts() {
        try {
            this.isLoading.set(true)
            const data = await this.productServ.getProducts()
            this.products.set(data)
        } catch (error) {
            console.error('商品載入失敗', error)
            // 錯誤處理...
        } finally {
            this.isLoading.set(false)
        }
    }
}
```
```html
<!-- ✅ 使用新區塊語法 + 良好的 Loading UX -->
@if (isLoading()) {
    <!-- 使用 Skeleton 而非單純 Spinner -->
    <div class="space-y-4">
        <div class="h-20 bg-gray-200 rounded animate-pulse"></div>
        <div class="h-20 bg-gray-200 rounded animate-pulse"></div>
    </div>
} @else {
    @for (item of products(); track item.id) {
        <div class="
            transition-all duration-300 
            hover:scale-105 hover:shadow-lg
            cursor-pointer
        ">
            {{ item.name }}
        </div>
    }
}
```

---

## 評估標準

### Code Review Checklist
在生成或修改程式碼後，請確認：

**架構與語法：**
- [ ] 所有響應式狀態都使用 `signal()` 或 `computed()`
- [ ] 模板使用 `@if`, `@for` 區塊語法
- [ ] 組件通訊使用 `input()`, `output()`, `model()`
- [ ] 沒有使用 `document.querySelector()` 直接操作 DOM
- [ ] 組件成員按照 `#region` 分組且順序正確
- [ ] 檔案命名符合 `.service.ts`, `.util.ts`, `.model.ts` 規則

**註解品質：**
- [ ] **每個 function/method 都有清晰的註解**
- [ ] 註解說明「為什麼」與「做什麼」，而非「怎麼做」
- [ ] 複雜的業務邏輯有詳細的說明註解
- [ ] 重要的 Signal 與 Computed 有用途說明
- [ ] 註解使用正體中文且語意清晰
- [ ] 沒有過時、錯誤或無意義的註解

**樣式與設計（Mobile First）：**
- [ ] 優先使用 Tailwind Classes 而非自定義 `.scss`
- [ ] **CSS 以手機為預設，使用 `md:`, `lg:` 往上擴展**
- [ ] 視覺層次清晰，主要與次要操作有明顯區別
- [ ] 間距與對齊符合設計規範（使用 Tailwind spacing scale）
- [ ] 響應式設計**先在手機上測試**，再驗證平板與桌面

**動畫與效能：**
- [ ] 動畫僅使用 `transform` 和 `opacity` 屬性
- [ ] 使用 `will-change` 提示瀏覽器優化（並在完成後移除）
- [ ] 動畫時長符合建議範圍（0.15s - 0.8s）
- [ ] 沒有同時執行過多複雜動畫
- [ ] 長列表使用虛擬滾動或分頁
- [ ] 動畫函數有清楚的註解說明用途與效能考量

**UX 體驗：**
- [ ] 所有互動都有即時視覺回饋（hover, active, focus）
- [ ] Loading 狀態使用 Skeleton Screen 而非單純 Spinner
- [ ] 錯誤訊息清晰且提供解決方案
- [ ] 表單驗證即時且友善
- [ ] 移動端觸控區域足夠大（最小 44x44px）
- [ ] 不依賴 hover 狀態（移動端友善）

**程式碼品質：**
- [ ] 每個函數/組件符合單一職責原則
- [ ] 沒有硬編碼的魔術數字（使用常數）
- [ ] 錯誤處理完整且有適當的 try-catch
