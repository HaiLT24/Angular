# COMPONENT INTERACTION

## PASS DATA FROM PARENT TO CHILD WITH INPUT BINDING

- Khi thiết kế hệ thống, chúng ta luôn muốn thiết kế được những component có tính tái sử dụng cao (giảm thời gian coding, dễ bảo trì, tường minh code) như các tính năng (upload file, view-image, view-audio, loading, ...). Vậy câu hỏi đặt ra thì cần làm thế nào?

Tạo component :

```
ng g c progress-bar --skip-test
```

### @INPUT DECORATOR

Để thêm 1 property cho phép thiết lập progress hiện tại của progress-bar, chúng ta có thể khai báo một property cho class và thêm 1 decorator như sau :

```typescript
export class ProgressBarComponent implements OnInit {
  @Input() progress = 0;
  constructor() {}
  ngOnInit() {}
}
```

Như vậy đã có thể báo cho Angular biết rằng, cần nhận vào 1 property tên là progress và giá trị default bằng 0. Ngoài ra cũng có thể thêm 1 số property khác (không giới hạn)

```typescript
export class ProgressBarComponent implements OnInit {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  @Input() progress = 0;
  constructor() {}
  ngOnInit() {}
}
```
**Lưu ý:**
- @Input được gọi là một property decorator, nó sẽ gắn thêm meta data cho property ngay phía sau nó.
- Nếu không khai báo decorator @Input thì sẽ không thể nhận giá trị truyền vào từ component khác, vì Angular sẽ không biết cách để binding, do đó property khai báo chỉ là 1 property bình thường của class.

**Cách sử dụng**
```html
<app-progress-bar
  [progress]="15"
  [backgroundColor]="'#9e9e9e'"
  [progressColor]="'#2e8b57'"
>
</app-progress-bar>
```

#### Component progress-bar

**CSS**
```css
    .progress-bar-container,
    .progress {
      height: 20px;
    }
    .progress-bar-container {
      width: 100%;
    }
```
**HTML**
```html
    <div
      class="progress-bar-container"
      [style.backgroundColor]="backgroundColor"
    >
      <div
        class="progress"
        [style]="{
          backgroundColor: progressColor,
          width: progress + '%'
        }"
      ></div>
    </div>
```
**TS**
```typescript
import { Component, OnInit, Input } from '@angular/core';

@Component({
  selector: 'app-progress-bar',
  templateUrl: './progress-bar.component.html',
  styles: ['./progress-bar.component.css'],
})
export class ProgressBarComponent implements OnInit {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  @Input() progress = 0;
  constructor() {}
  ngOnInit() {}
}
```
### THAY ĐỔI GIÁ TRỊ CỦA INPUT

Trường hợp muốn validate có giá trị input đầu vào (null | undefined thành các giá trị default). Vì ngOnit sẽ hứng các dữ liệu Input được binding ở lần đầu tiên => không phải giải pháp toàn diện. Giải pháp cho vấn đề này life-cycle ngOnChanges.

`ngOnChanges` sẽ chạy lại mỗi khi có một input nào bị thay đổi, nó sẽ được tự động gọi bởi Angular, do đó có thể validate propery progress như sau:

```typescript
export class ProgressBarComponent implements OnInit, OnChanges {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  @Input() progress = 0;

  constructor() {}

  ngOnChanges(changes: SimpleChanges) {
    if ('progress' in changes) {
      if (typeof changes['progress'].currentValue !== 'number') {
        const progress = Number(changes['progress'].currentValue);
        if (Number.isNaN(progress)) {
          this.progress = 0;
        } else {
          this.progress = progress;
        }
      }
    }
  }

  ngOnInit() {}
}
```
Trong trường hợp không dùng `ngOnChanges`, có thể dùng `getter/setter` để làm điều này
```typescript
export class ProgressBarComponent implements OnInit {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  private $progress = 0;
  @Input()
  get progress(): number {
    return this.$progress;
  }
  set progress(value: number) {
    if (typeof value !== 'number') {
      const progress = Number(value);
      if (Number.isNaN(progress)) {
        this.$progress = 0;
      } else {
        this.$progress = progress;
      }
    } else {
      this.$progress = value;
    }
  }

  constructor() {}

  ngOnInit() {}
}
```

## PARENT LISTEN FOR CHILD EVENT
Thông thường, trong một trang HTML khi có một sự kiện nào đó phát sinh ở một thẻ HTML (ví dụ sự kiện click của thẻ button, submit của form, etc) thì có thể listen ở đâu đó trong code JavaScript.

Vậy với những Component mà tự định nghĩa thì có cách nào bắn ra các event mong muốn hay không (component event). Câu trả lời cho vấn đề này chính là EventEmitter và @Output decorator.

### KHỞI TẠO COMPONENTS

Khởi tạo 2 Components là: AuthorListComponent, AuthorDetailComponent
```bash
ng g c author-list
ng g c author-detail
```
`Author List component` sẽ chứa một danh sách các authors, và nó sẽ truyền vào từng author cho Author Detail hiển thị. Author Detail sẽ cho phép truyền vào input là thông tin của một author:

```typescript
export interface Author {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
  gender: string;
  ipAddress: string;
}
```

```typescript
import { Component, OnInit } from '@angular/core';
import { authors } from '../authors';
@Component({
  selector: 'app-author-list',
  template: `<app-author-detail
    *ngFor="let author of authors"
    [author]="author"
  ></app-author-detail>`,
  styles: [``],
})
export class AuthorListComponent implements OnInit {
  authors = authors;
  constructor() {}
  ngOnInit() {}
}
```

`Author Detail Component`

```typescript
import { Component, OnInit, Input } from '@angular/core';
import { Author } from '../authors';
@Component({
  selector: 'app-author-detail',
  template: `
    <div *ngIf="author">
      <strong>{{ author.firstName }} {{ author.lastName }}</strong>
      <button (click)="handleDelete()">x</button>
    </div>
  `,
  styles: [``],
})
export class AuthorDetailComponent implements OnInit {
  @Input() author: Author;
  constructor() {}
  ngOnInit() {}
  handleDelete() {}
}
```

handleDelete() sẽ không hoạt động. Vì thực thế thông tin này thuộc về AuthorDetailComponent, nên không được phép edit/modify/remove, mà cần phải gửi 1 event cho parent component là AuthorListComponent để báo rằng muốn xóa phần tử. Lúc này cần dùng đến `EvenEmitter`(trình phát sự kiện) và `@Output` decorator

```typescript
export class AuthorDetailComponent implements OnInit {
  @Input() author: Author;
  @Output() deleteAuthor = new EventEmitter<Author>();
  constructor() {}
  ngOnInit() {}
  handleDelete() {
    this.deleteAuthor.emit(this.author);
  }
}
```
và giờ ở AuthorListComponent sẽ hứng sự kiện được phát ra từ ChildComponent

```typescript
@Component({
  selector: 'app-author-list',
  template: `<app-author-detail
    *ngFor="let author of authors"
    [author]="author"
    (deleteAuthor)="handleDelete($event)"
  >
  </app-author-detail>`,
  styles: [``],
})
export class AuthorListComponent implements OnInit {
  authors = authors;
  constructor() {}
  ngOnInit() {}
  handleDelete(author: Author) {
    this.authors = this.authors.filter((item) => item.id !== author.id);
  }
}
```

Có nhận thấy đoạn code trên có vấn đề gì không??
Đúng rồi, thiếu trackByFunction. Có thể xem lại lesson 4 nhé để biết tại sao nên sử dụng trackByFunction.