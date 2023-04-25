# Directives

**Directives** là một đối tượng giúp chúng ta dễ dàng thay đổi một đối tượng khác và cách áp dụng rất đơn giản và linh hoạt. Directives có thể hiểu như là các đoạn mã typescript (hoặc javascript) kèm theo cả HTML.

## Kind Directives
---

`Components directives`: Không có nghi ngờ gì khi gọi component là directive cũng được, vì rõ ràng là component cho phép định nghĩa selector và gọi ra như một thẻ html tag (<component-name></component-name>).

`Structural directives`: Là directive cấu trúc, dùng để vẽ html, hiển thị data lên giao diện html, và thay đổi cấu trúc DOM bằng việc thêm bớt các phần tử trong DOM. Các structural directive thường có dấu '*' ở trước của directive. Ví dụ *ngFor, *ngIf.

`Attribute directives`: Thay đổi giao diện, tương tác của các đối tượng hoặc thay đổi directive khác hoặc thêm các thuộc tính động cho element html. ví dụ *ngStyle.

### Components directives
---

Components là một khối code trong app Angular. Nó là sự kết hợp của bộ template html (bộ khung html) và nhúng kèm code TypeScript (hoặc Javascript). Các components là độc lập với nhau và độc lập với hệ thống. Nó có thể được cài vào hoặc tháo ra khỏi hệ thống dễ dàng. Một component có thể hiểu như một control trên màn hình hiển thị, gồm giao diện html và code logic xử lý sự kiện đi kèm control đó. Một component cũng có thể to lớn như là cả 1 màn hình chứa nhiều control hoặc một nhóm nhiều màn hình. Tức là là một component cũng có thể chứa và gọi được nhiều component khác nối vào. Như vậy Angular rất linh hoạt trong việc chia nhỏ code ra các component.

- selector
- template
- templateUrl
- styles
- styleUrls

### Structural directives
Chỉ thị góc cung cấp một cách tuyệt vời để đóng gói các hành vi có thể tái sử dụng chỉ thị có thể áp dụng các thuộc tính, lớp CSS và trình xử lý sự kiện cho một phần tử.

```ts
hostDirectives?: (Type<unknown> | {
  directive: Type<unknown>;
  inputs?: string[];
  outputs?: string[];
})[];
```

#### Ng-if… else

```ts
<div *ngIf="time; else noTime">
  Time: {{time}}
</div>
<ng-template #noTime> No time. </ng-template>
```

#### Ng-Template

```html
<div *ngIf="isTrue; then tmplWhenTrue else tmplWhenFalse"></div>
<ng-template #tmplWhenTrue >I show-up when isTrue is true. </ng-template>
<ng-template #tmplWhenFalse > I show-up when isTrue is false </ng-template>
```

#### Ng-Container

Tương tự như `Ng-Template` dùng để gom html. Nhưng điểm mạnh của Ng-Container là thẻ directive này không render ra tag html như là `<ng-template>` mà tag sẽ được ẩn đi, giúp cho layout css không bị vỡ nếu bạn gom html (Không sợ bị nhảy từ div cha sang div con, cấu trúc html không hề thay đổi khi gom vào tag `<ng-container></ng-container>`)

```html
Welcome <div *ngIf="title">to <i>the</i> {{title}} world.</div>
Welcome <ng-container *ngIf="title">to <i>the</i> {{title}} world.</ng-container>
```

#### NgSwitch and NgSwitchCase

```html
<div [ngSwitch]="isMetric">
  <div *ngSwitchCase="true">Degree Celsius</div>
  <div *ngSwitchCase="false">Fahrenheit</div>
  <div *ngSwitchDefault>Default </div>
</div>
```
*ngSwitchDefault Trong trường hợp muốn dùng switch case default (nếu toàn bộ case k thỏa màn thì vào default).

### Attribute directive
---

```ts
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() highlightColor: string | undefined;

  ngOnChanges(changes: SimpleChanges) {
    console.log(changes);
  }

  constructor(private el: ElementRef) { }

  @HostListener('mouseenter') onMouseEnter() {
    this.highlight(this.highlightColor || 'red');
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.highlight('');
  }

  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}
```

```html
<p appHighlight highlightColor="yellow">Highlighted in yellow</p>
```





