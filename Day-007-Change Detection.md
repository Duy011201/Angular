# Change Detection

Change detection là một cơ chế theo dõi sự thay đổi, cho phép nội dung ở phần giao diện luôn được đồng bộ với sự thay đổi trong model tương ứng. Sự thay đổi này có thể được phát sinh bởi những trường hợp sau:

- DOM events (click, hover, …)
- AJAX requests
- Timers (setTimers(), setInterval())

## View như một khái niệm cốt lõi

Như chúng ta đã biết, một ứng dụng Angular được mô tả là một cây components. Tuy nhiên, về bản chất thì Angular sử dụng một khái niệm trừu tượng bậc thấp được gọi là view. Có một mối quan hệ trực tiếp giữa view và component – một view liên kết với một component và ngược lại. Mỗi view có một thuộc tính component, tham chiếu tới một đối tượng component. Mọi tác vụ như kiểm tra thuộc tính và cập nhật DOM được thực hiện trên views. Do đó, nói chính xác hơn thì Angular là một cây view, trong khi component có thể được mô tả là một khái niệm bậc cao của view. Mỗi view lại chứa một liên kết tới những view con thông qua thuộc tính nodes nên những tác vụ có thể được thực hiện trên những views con này.

## Trạng thái view (View state)

Mỗi view có một trạng thái nắm giữ vai trò rất quan trọng để Angular xác định xem có thực hiện change detection trên view và child view hay không. Có nhiều loại trạng thái nhưng trong phạm vi bài viết này sẽ chỉ đề cập đến những trạng thái sau:

1. FirstCheck
2. ChecksEnabled
3. Errored
4. Destroyed

Change detection sẽ được bỏ qua nếu `ChecksEnabled` có giá trị false hoặc view đang ở trạng thái `Errored` hoặc `Destroyed`. Mặc định, tất cả các views đều được khởi tạo với trạng thái `ChecksEnabled`, ngoại trừ trường hợp `ChangeDetectionStrategy.OnPush` được sử dụng. Trạng thái có thể tồn tại cùng nhau, ví dụ : một view có trạng thái đồng thời là `FirstCheck` và `ChecksEnabled`.

Angular có một số lượng nhất định các khái niệm bậc cao để xử lý views. Một trong số đó là ViewRef. Class này được dùng để mô tả một view bên dưới và có một hàm tương ứng là `detectChanges`. Khi một sự kiện bất đồng bộ xảy ra, Angular thực hiện theo dõi thay đổi trên ViewRef từ trên xuống dưới.

`viewRef` có thể được truyền vào hàm khởi tạo component sử dụng `ChangeDetectorRef`.

```ts
export class AppComponent {
    constructor(cd: ChangeDetectorRef) { ... }
}
```

Định nghĩa của hàm:

```ts
export declare abstract class ChangeDetectorRef {
    abstract checkNoChanges(): void;
    abstract detach(): void;
    abstract detectChanges(): void;
    abstract markForCheck(): void;
    abstract reattach(): void;
}
export abstract class ViewRef extends ChangeDetectorRef {
   ...
}
```

## Những tác vụ trong Change detection

Hàm logic có nhiệm vụ chính trong việc theo dõi thay đổi của một view là `checkAndUpdateView`. Chức năng của nó chủ yếu được thực hiện trên các child component view. Hàm này được gọi đệ quy trên mỗi component bắt đầu từ host component. Điều đó có nghĩa là component con trở thành component cha ở lần gọi đệ quy tiếp theo.

Khi hàm này được gọi ở một view cụ thể, nó sẽ thực hiện các tác vụ theo thứ tự sau:

1. Gán `ViewState.firstCheck` về `true` nếu view được kiểm tra lần đầu và false nếu nó đã từng được kiểm tra trước đó.

2. Kiểm tra và cập nhật những thuộc tính input trên child `component/directive`.

3. Cập nhật trạng thái thay đổi của `child view`.

4. Thực hiện việc theo dõi thay đổi trên những embedded view.
   Gọi lifecycle hook `OnChanges` trên child component nếu bindings thay đổi.

5. Gọi `OnInit` và `ngDoCheck` rên child component (OnInit chỉ được gọi trong lần kiểm tra đầu tiên)

6. Cập nhật danh sách truy vấn `ContentChildren` trên view component con.

7. Gọi lifecycle hooks `AfterContentInit` và `AfterContentChecked` trên child component(`AfterContentInit` chỉ được gọi trong lần kiểm tra đầu tiên)

8. Cập nhật DOM interpolations cho view hiện tại nếu những thuộc tính trên view component hiện tại thay đổi.

9. Thực hiện theo dõi thay đổi trên child view.
   Cập nhật danh sách truy vấn `ViewChildren` trên view component hiện tại.

10. Gọi lifecycle hooks `AfterViewInit` và `AfterViewChecked` trên child component (`AfterViewInit` chỉ được gọi trong lần kiểm tra đầu tiên)

11. Vô hiệu việc kiểm tra trên view hiện tại.

Có một vài điểm cần chú ý dựa trên danh sách các tác vụ được liệt kê phía trên:

Đầu tiên, lifecycle hook `OnChanges` được gọi trên chid component trước khi child view được kiểm tra và nó sẽ được gọi ngay cả khi việc theo dõi trên child view bị bỏ qua.

Thứ hai là việc DOM của view được cập nhật là một phần của cơ chế theo dõi thay đổi khi view đang được kiểm tra. Điều này có nghĩa là nếu một component không được kiểm tra thì DOM sẽ không cập cập nhật, ngay cả khi những thuộc tính của component được sử dụng ở template thay đổi. Templates được render trước lần kiểm tra đầu tiên. Việc cập nhật DOM được đề cập ở đây thật ra là cập nhật interpolation. Do đó nếu có một template như sau: `<span>some {{name}}</span>` thì thẻ DOM `span` sẽ được render trước lần kiểm tra đầu tiên. Trong quá trình kiểm tra, phần `{{name}}` mới được render.

Một phát hiện thú vị khác đó là trạng thái của một child component view có thể được thay đổi trong Change detection. Tất cả component view đều được khởi tạo với trạng thái `ChecksEnabled` mặc định. Tuy nhiên, đối với các component sử dụng strategy `OnPush` thì Change detection sẽ bị vô hiệu sau lần kiểm tra đầu tiên (bước số 9 trong danh sách)

Điều đó có nghĩa rằng trong quá trình theo dõi thay đổi, việc kiểm tra component view và các component con của nó sẽ bị bỏ qua. Tài liệu gốc về `OnPush` có nói rằng một component chỉ được kiểm tra nếu những binding của nó bị thay đổi. Vì vậy, việc kiểm tra phải được kích hoạt bằng thiết lập `ChecksEnabled`. Đoạn code dưới đây thực hiện điều đó (bước số 2):

```ts
if (compView.def.flags & ViewFlags.OnPush) {
  compView.state |= ViewState.ChecksEnabled;
}
```

Trạng thái chỉ được cập nhật nếu bindings của view cha thay đổi và component con được khởi tạo với `ChangeDetectionStrategy.OnPush`.

Cuối cùng, cơ chế theo dõi thay đổi đối với view hiện tại có nhiệm vụ theo dõi thay đổi ở cả những view con (bước số 8). Đây là lúc mà trạng thái của component view con được kiểm tra và nếu là `ChecksEnabled` thì Change detection được thực hiện. Đoạn code tương ứng:

```ts
viewState = view.state;
...
case ViewAction.CheckAndUpdate:
  if ((viewState & ViewState.ChecksEnabled) &&
    (viewState & (ViewState.Errored | ViewState.Destroyed)) === 0) {
    checkAndUpdateView(view);
  }
}
```

Từ đó có thể thấy trạng thái của view kiểm soát cơ chế theo dõi thay đổi có được thực hiện trên view và child views hay không. Câu hỏi được đặt ra là liệu chúng ta có thể kiểm soát được trạng thái? Câu trả lời là có và đó là phần thứ 2 của bài viết sẽ đề cập tới.

Một số lifecycle hooks được gọi trước khi việc cập nhật ở DOM diễn ra (bước số 3, 4, 5) và một số thì sau (bước số 9). Nếu có một cây component như sau `A -> B -> C` thì thứ tự các lời gọi hook và cập nhật binding như sau:

```ts
A: AfterContentInit
A: AfterContentChecked
A: Update bindings
    B: AfterContentInit
    B: AfterContentChecked
    B: Update bindings
        C: AfterContentInit
        C: AfterContentChecked
        C: Update bindings
        C: AfterViewInit
        C: AfterViewChecked
    B: AfterViewInit
    B: AfterViewChecked
A: AfterViewInit
A: AfterViewChecked
```

## Tìm hiểu những hàm liên quan

Giả sử có một cây component như sau:

![markdown](./img/change-detection-img-01.webp)

Như đã đề cập ở phần trên của bài viết, mỗi component được liên kết với một component view. Mỗi view được khởi tạo với `ViewState.ChecksEnabled` đồng nghĩa với việc khi Angular thực hiện Change detection thì mọi component trong cây component sẽ được kiểm tra.

Nếu muốn vô hiệu Change detection với `AComponent` và các thành phần con của nó thì đơn giản chỉ cần đặt `ViewState.ChecksEnabled` về `false`. Việc thay đổi trạng thái là thao tác ở mức thấp, vì vậy Angular cung cấp một số lượng nhất định các hàm public xử lý trên view. Mỗi component có thể có một liên kết với view tương ứng thông qua `ChangeDetectorRef` token. Public interface cho điều này của Angular như sau:

```ts
class ChangeDetectorRef {
  markForCheck(): void;
  detach(): void;
  reattach(): void;

  detectChanges(): void;
  checkNoChanges(): void;
}
```

### detach

Hàm đầu tiên này cho phép xử lý trạng thái detach, vô hiệu việc kiểm tra trên view hiện tại:

```ts
detach(): void { this._view.state &= ~ViewState.ChecksEnabled; }
```

Sử dụng như sau:

```ts
export class AComponent {
  constructor(public cd: ChangeDetectorRef) {
    this.cd.detach();
  }
}
```

Hệ quả là phần bên trái (màu cam) của cây component sẽ không được kiểm tra:

![markdown](./img/change-detection-img-02.webp)

Có 2 điều cần lưu ý ở đây: đầu tiên là tất cả components con đều không được kiểm tra ngay cả khi chúng ta đã thay đổi trạng thái của `AComponent`. Thứ hai là bởi Change detection không được thực hiện ở nhánh component bên trái nên DOM ở phần template sẽ không được cập nhật. Đoạn code mô tả ví dụ như sau:

```ts
@Component({
  selector: "a-comp",
  template: `<span>See if I change: {{ changed }}</span>`,
})
export class AComponent {
  constructor(public cd: ChangeDetectorRef) {
    this.changed = "false";

    setTimeout(() => {
      this.cd.detach();
      this.changed = "true";
    }, 2000);
  }
}
```

Lần đầu component sẽ được kiểm tra, thẻ `span` được render với text là `See if I change: false`. Sau 2 giây, thuộc tính `changed` được thay đổi thành `true` nhưng nội dung text bên trong thẻ `span` vẫn không thay đổi. Mọi thứ sẽ hoạt động trở lại nếu bỏ đi dòng `this.cd.detach()`.

### reattach

Như đã đề cập ở phần đầu của bài viết, lifecycle hook `OnPush` sẽ được gọi trên `Acomponent` nếu input binding `aProp` thay đổi trên `AppComponent`. Điều này có nghĩa khi chúng ta nhận thấy sự thay đổi của thuộc tính input, chúng ta có thể kích hoạt Change detection đối với component hiện tại và loại bỏ Change detection ở lần tiếp theo. Đoạn code mô tả như sau:

```ts
export class AComponent {
  @Input() inputAProp;

  constructor(public cd: ChangeDetectorRef) {
    this.cd.detach();
  }

  ngOnChanges(values) {
    this.cd.reattach();
    setTimeout(() => {
      this.cd.detach();
    });
  }
}
```

`reattach` chỉ đơn giản là thay đổi trạng thái `ViewState.ChecksEnabled`

```ts
reattach(): void { this._view.state |= ViewState.ChecksEnabled; }
```

Điều này tương đương với việc `ChangeDetectionStrategy` được thiết lập về `OnPush`: vô hiệu kiểm tra ở lần theo dõi đầu tiên và kích hoạt lại khi có sự thay đổi thuộc tính đã được bind của component và vô hiệu sau khi chạy.

**Chú ý là hook OnPush chỉ được gọi trên component cao nhất của cây component trên nhánh đã bị vô hiệu, không phải trên mọi component của nhánh này.**

### markForCheck

Hàm `reattach` chỉ cho phép kiểm tra trên component hiện tại. Nếu Change detection không được cho phép trên component cha thì hàm không có tác dụng. Điều này có nghĩa rằng hàm `reattach` chỉ hữu dụng trên component cao nhất của nhánh đã bị vô hiệu. Hàm `markForCheck` cho phép kiểm tra trên tất cả components cha lên tới root component.

```ts
let currView: ViewData | null = view;
while (currView) {
  if (currView.def.flags & ViewFlags.OnPush) {
    currView.state |= ViewState.ChecksEnabled;
  }
  currView = currView.viewContainerParent || currView.parent;
}
```

Hàm trên thực hiện vòng lập từ dưới lên và cho phép Change detection trên mỗi component đến tới root component.

Khi nào điều này trở lên hữu dụng? Với `ngOnChanges`, lifecycle hook `ngDoCheck` được gọi ngay cả khi component sử dụng `OnPush`. Nhắc lại lần nữa rằng nó chỉ được gọi trên component cao nhất của nhánh bị vô hiệu, không phải trên mọi component trong nhánh đó. Tuy nhiên chúng ta có thể sử dụng hook này để thực hiện những logic tùy biến và đánh dấu component là hợp lệ đối với Change detection. Vì Angular chỉ kiểm tra những tham chiếu đối tượng, nên chúng ta có thể triển khai việc kiểm tra cứng một vài thuộc tính đối tượng như sau:

```ts
Component({
   ...,
   changeDetection: ChangeDetectionStrategy.OnPush
})
MyComponent {
   @Input() items;
   prevLength;
   constructor(cd: ChangeDetectorRef) {}

   ngOnInit() {
      this.prevLength = this.items.length;
   }

   ngDoCheck() {
      if (this.items.length !== this.prevLength) {
         this.cd.markForCheck();
         this.prevLenght = this.items.length;
      }
   }
}
```

### detectChanges

Hàm này thực hiện theo dõi thay đổi một lần trên component hiện tại và tất cả thành phần con của nó, bất kể trạng thái của component là gì. Việc kiểm tra component và view hiện tại có bị vô hiệu hay không cũng sẽ bị bỏ qua. Đây là một ví dụ:

```ts
export class AComponent {
  @Input() inputAProp;

  constructor(public cd: ChangeDetectorRef) {
    this.cd.detach();
  }

  ngOnChanges(values) {
    this.cd.detectChanges();
  }
}
```

DOM được cập nhật khi có sự thay đổi thuộc tính ngay cả khi tham chiếu change dectector đã bị loại bỏ (detached).

### checkNoChanges

Hàm này đảm bảo rằng không có bất cứ thay đổi nào được thực hiện trong mỗi lần thực hiện Change detection. Về cơ bản thì nó sẽ thực hiện các bước 1, 7, 8 ở danh sách các tác vụ đề cập ở đầu bài viết và trả về một exception nếu nó tìm được một binding thay đổi hoặc xác định thấy DOM có thể được cập nhật.
