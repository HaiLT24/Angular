# 3: ANGULAR DATA BINDING

## INTERPOLATION

- Một component trong Angular chỉ là 1 TypeScript (TS) class thông thường, nó đi kèm với một decorator để gắn thêm meta-data như template mà nó định nghĩa là gì. Như vậy TS class và template này hoàn toàn ko biết đến nhau, nó sẽ được Angular xử lý để gắn chúng lại.
- Interpolation (Nội suy) sẽ giải quyết vấn đề nay `{{ expression }}`. Có thể hiểu nếu object có trả về dữ liệu thì sẽ được display(hiển thị) nó ngay ra vị trí dấu `{{}}`

- Ví dự:

```typescript
import { Component } from '@angular/core';
@Component({
  selector: 'app-hello',
  template: `
    <h2>Hello there!</h2>
    <h3>Your name: {{ user?.name }}</h3>
    <p>Your age: {{ user?.age }}</p>
    <p>Your phone: {{ user?.phone }}</p>
  `,
})
export class HelloComponent {
  user = {
    name: 'Tran Van A',
    age: 28,
    phone: '0236578945'
  };
}
```

## PROPERTY BINDING

- DOM là gì? 
    + Khi browser load trang web, nó sẽ parse (phân tích cú pháp) phần HTML và xây dựng lên một cây Document Object Model (DOM) từ đó để biểu diễn tương ứng những gì HTML đang được dựng, cho phép chúng ta tương tác với phần HTML như đọc, sửa HTML bằng JavaScript
    
    + Ví dụ : khi bạn có phần HTML: Sau khi parse xong sẽ có một object (node) thuộc type HTMLInputElement được tạo ra. Ở đây type=”text” hay value=”something” là các HTML attribute. Mỗi tag HTML có thể có nhiều attribute khác nữa. Object được tạo tương ứng sẽ có dạng

```typescript
obj = {
  type: 'text',
  value: 'something',
  attributes: [], // thuộc type NamedNodeMap, dạng như một array
  // … các thuộc tính, method khác
};
```
- Attribute chỉ là những gì được vào phần opening tag của một tag, còn lại là property của object(node). Các attributes sẽ được lưu trữ vào property attributes của node tương ứng. Có những attribute sẽ được map sang thành property tương ứng, ví dụ như type và value ở trên, nhưng có những attribute không được map sang. 
- Khi muốn sử dụng linh động hơn trong Angular (1 template có thể thay đổi data để hiển thị). Lúc này cần update các properties của DOM để làm cho nó đáp ứng. Nhưng nếu phải dùng document.querySelector/jQuery để manipulate DOM thì quá vô vị. Sử dụng property binding của Angular sẽ giải quyết được vấn đề : 

```html
<input type="text" [value]="user.name" />
```

**Lưu ý** : ngoài property binding cho các phần tử HTML, chúng ta có thể áp dụng property binding cho các component

## EVENT BINDING
- JavaScript sử dụng rất nhiều đến khái niệm event. Khi người dùng click vào button => hiển thị alert cho người dùng nhín thấy:
    + Js sử dung addEventListener
    + Angular sử dụng Event binding. Để gắn event listener vào một phần tử nào đó trên template, có thể sử dụng cú pháp ngoặc tròn như sau:

```typescript
@Component({
  selector: 'app-hello',
  template: `
    <h2>Hello there!</h2>
    <button (click)="showInfo()">Click me!</button>
  `,
})
export class HelloComponent {
  showInfo() {
    alert('Inside Angular Component method');
  }
}
```

=> Lúc này sẽ trỏ đến methoa được khai báo trong Component(file TS). Khá giống như sử dụng inline event listener

```html
<button onclick="showInfo()">Click me!</button>
```

## TWO-WAY BINDING

- Two-way binding là sự kết hợp giữa property binding (dữ liệu từ class ra template) và event binding (dữ liệu từ template vào class). Có chứa cú pháp ngắn gọn :

```html
<input type="text" [(ngModel)]="user.name" />
```

- Đây là dạng đầy đủ

```html
<input type="text" [ngModel]="user.name" (ngModelChange)="user.name = $event" />
```

## SUMMARY

- Data Binding là bước đột phá trong Angular, tính ứng dụng rất cao. Tiết kiệm cực lớn thời gian coding so với Js.