# Day 2: Two Way Binding

Angular có một directive để áp dụng two-way binding đó là ngModel. Chúng ta có thể bao ngModel trong cặp dấu **[()]**, nó sẽ thực hiện đồng bộ dữ liệu từ Component vs **DOM - Template** và ngược lại.

## Using ngModel
---

```ts
//app.component.html
<input type="text" [(ngModel)]="message">
<button (click)="updateMessages()">Click me</button>
<p>{{message}}</p>

//app.component.ts
export class AppComponent {
  messages: string[] = [];
  message: string = '';
  updateMessages() {
    this.messages.push(this.message);
    this.message = '';
  }
}


//or

//app.component.html
<input type="text" [value]="message" (input)="message = $event.target.value">
<button (click)="updateMessages()">Click me</button>
<p>{{message}}</p>

//app.component.html
<input type="text" 
    [ngModel]="message" 
    (ngModelChange)="message = $event">
<button (click)="updateMessages()">Click me</button>
<p>{{message}}</p>
```

## Customizing
---
Để tạo được Custom Two-way Data Binding, chúng ta kết hợp **@Input** (property binding) và **@Output** (event binding).

```ts
//app.component.html
<input type="number" [(ngModel)]="myNumber">
<app-child [(value)]="myNumber"></app-child>

//app.component.ts
myNumber: number = 1;

//child.component.ts
export class ChildComponent {
  @Input() value: number | undefined;
  @Output() valueChange = new EventEmitter();

  reset() {
    this.valueChange.emit(0);
    this.value = 0;
  }
}
```