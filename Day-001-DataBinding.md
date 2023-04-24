# Day 1: Data Bindings

So **What is data biding** là một kỹ thuật, trong đó dữ liệu được đồng bộ hóa giữa component và template. Trong Angular chúng ta đã được Angular làm nhiệm vụ đồng bộ những dữ liệu bị ràng buộc lại với nhau, chúng ta chỉ cần update data/state cần thiết.

```ts
//Syntax
Use [] to bind from source to view
Use () to bind from view to source
Use [()] to bind in a two-way sequence of view to source to view
```

## INTERPOLATION
---
Một component trong Angular chỉ là một TypeScript (TS) class thông thường, và nó đi kèm với một decorator để gắn thêm meta-data như template mà nó định nghĩa là gì. Như vậy, TS class và template này hoàn toàn không biết đến nhau, mà nó sẽ được Angular xử lý để gắn chúng lại. Bất kỳ dữ liêu nào cũng có thể được hiển thị (mà chúng ta gọi là `interpolation`) `{{ expression }}`.

```ts
import { Component } from '@angular/core';
@Component({
  selector: 'app-hello',
  template: `
    <h2>Hello there!</h2>
    <h3>Your name: {{ user.name }}</h3>
    <p>Your name: {{ user.age }}</p>
  `,
})
export class HelloComponent {
  user = {
    name: 'Tiep Phan',
    age: 30,
  };
}
```

## PROPERTY BINDING
---
Một số tag khi sử dụng bạn cần phải truyền vào data cho một property nào đó. Đối với **DOM** chúng ta có 2 khái niệm khác nhau là property và attribute. Ở đây type=”text” hay value=”something” là các HTML attribute. Mỗi tag HTML có thể có nhiều attribute khác nữa. Object được tạo tương ứng sẽ có dạng

```ts
obj = {
  type: 'text',
  value: 'something',
  attributes: [], // thuộc type NamedNodeMap, dạng như một array
  // … các thuộc tính, method khác
};
```

Như bạn có thể thấy, attribute được dùng để chỉ những gì các bạn đặt vào phần opening tag của một tag, còn lại là property của object (node). Các attributes sẽ được lưu trữ vào property attributes của node tương ứng. Có những attribute sẽ được map sang thành property tương ứng. Chúng ta chỉ cần sử dụng một cú pháp để báo cho Angular biết chúng ta cần binding một property qua template.

```ts
//Interpolation Property Attribute Class Style

{{expression}} 
[target]="expression"

<input type="text" [value]="user.name" />
<input type="text" value={{user.name}} />
```

## EVENTS BINDING
---

**Event biding** cho phép liên kết các sự kiện keystrokes, clicks, hover, touch, etc, v.v. với một method trong class component. Để gắn event listener vào một phần tử nào đó ở trên template, chúng ta có thể sử dụng cú pháp ngoặc tròn tròn.

```ts
(target)="statement"

<button type="button" onclick="showInfo()">Click me!</button>
```

## TWO-WAY BINDING
---

Trong thực tế two-way binding chính là kết hợp của binding dữ liệu từ class ra template thông qua property binding, và từ template vào class thông qua event binding. Nó chứa cú pháp ngắn gọn dạng vuông vuông tròn tròn.

```ts
<input type="text" [(ngModel)]="user.name" />
```

Để sử dụng ngModel bạn cần imports FormsModule, nhưng trong bài này chúng ta chỉ cần hiểu, nó là cách viết tắt của dạng tương ứng là:

```ts
<input type="text" [ngModel]="user.name" (ngModelChange)="user.name = $event" />
```




