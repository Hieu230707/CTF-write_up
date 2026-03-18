# Mark The Lyrics - Writeup
**Category:** Web

## Mô tả
Đây là một bài web challenge khá thú vị khi flag không được giấu trong JavaScript, comment hay request bất thường, mà lại được chèn trực tiếp vào lời bài hát trong HTML thông qua các thẻ `<mark>`.

## Phân tích ban đầu
Khi mở trang challenge, giao diện trông như một trang web âm nhạc bình thường:

- Có video YouTube được nhúng sẵn
- Có tiêu đề bài hát
- Có phần lời bài hát được chia thành nhiều đoạn

Tuy nhiên, hint của đề bài cho biết:

> the lyrics seem a little bit odd

Từ đó có thể đoán rằng phần lời bài hát chính là nơi chứa dữ liệu ẩn.

## Quá trình giải

### 1. Mở source code của trang
Sau khi xem source HTML, có thể thấy xen kẽ trong nội dung lời bài hát là nhiều thẻ `<mark>`.

Ví dụ:

```html
<div class="tag">{<mark>V</mark>erse <mark>1</mark>: Sơn Tùng M-<mark>T</mark>P}</div>
```

hoặc:

```html
<div class="tag"><mark>{</mark>Pre-Chorus: Sơn Tùng M-TP}</div>
```

Ngoài ra còn xuất hiện ở nhiều vị trí khác trong lyrics.

### 2. Thu thập toàn bộ nội dung trong thẻ `<mark>`
Lấy lần lượt các giá trị nằm giữa các cặp thẻ:

```html
<mark>...</mark>
```

Từ source code, các phần được đánh dấu là:

- `V`
- `1`
- `T`
- `{`
- `MCK`
- `-pap-`
- `cool`
- `-ooh-`
- `yeah`
- `}`

 Ghép các giá trị theo thứ tự xuất hiện. Nối toàn bộ các đoạn trên lại theo đúng thứ tự trong HTML, ta thu được flag:

## Flag

```txt
V1T{MCK-pap-cool-ooh-yeah}
```

## Kết luận
Đây là một bài web challenge nhẹ nhàng nhưng khá hay ở ý tưởng giấu flag trong nội dung lyrics bằng các thẻ HTML. Điểm đáng chú ý là các thẻ `<mark>` gần như không gây khác biệt rõ ràng trên giao diện, nên nếu chỉ nhìn bằng mắt thường rất dễ bỏ qua.

Bài học rút ra:

- Khi làm web challenge, luôn kiểm tra source code
- Những tag HTML bất thường có thể chứa dữ liệu quan trọng
- Hint của đề bài thường bám rất sát cách giải

