---
name: new-component
description: 建立符合專案規範的 Angular 元件。使用時機：「建立元件」、「新增 component」、「create component」。
---

# 建立 Angular 元件

根據 MBS 專案規範建立新的 Angular Standalone Component。

## 使用方式

```
/new-component <component-name> [--dumb]
```

- `<component-name>`：元件名稱（kebab-case，例如 `user-card`）
- `--dumb`：建立無 inject 的 Dumb Component（純 UI 元件）

## 流程

1. 詢問元件要放在哪個位置（apps/ 或 libs/components/）
2. 確認元件類型：
   - **Smart Component**（預設）：有 inject services
   - **Dumb Component**（`--dumb`）：純 UI，只有 input/output
3. 建立元件檔案，遵循以下結構

## Smart Component 結構（有 inject）

```typescript
import { Component, inject, signal, computed } from '@angular/core'

@Component({
    selector: 'mbs-xxx',
    imports: [],
    templateUrl: './xxx.component.html',
    styleUrl: './xxx.component.scss',
})
export class XxxComponent {
    //#region Injections & Services
    private serv = inject(XxxService)
    //#endregion

    //#region Enums & Constants
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

## Dumb Component 結構（無 inject）

```typescript
import { Component, input, output, model, signal, computed } from '@angular/core'

@Component({
    selector: 'mbs-xxx',
    imports: [],
    templateUrl: './xxx.component.html',
    styleUrl: './xxx.component.scss',
})
export class XxxComponent {
    // Inputs
    value = input.required<string>()
    label = input<string>('')

    // Two-way binding
    selected = model<number | null>(null)

    // Outputs
    change = output<void>()

    // Constants & Enums
    protected readonly EnumXxx = EnumXxx

    // 內部狀態
    protected xxxSignal = signal<...>(...)

    // Computed & Methods
}
```

