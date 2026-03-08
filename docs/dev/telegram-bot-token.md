# Cách Lấy Token Bot Telegram

SkyClaw cần một **Telegram Bot Token** để kết nối với Telegram. Token này là chuỗi dài dạng `123456789:AABBccDDeeFFggHH...` mà Telegram cấp cho bot của bạn.

---

## Hướng Dẫn Từng Bước

### Bước 1 — Mở @BotFather

1. Mở ứng dụng Telegram (điện thoại hoặc máy tính đều được).
2. Trong thanh tìm kiếm, gõ **@BotFather** và chọn tài khoản có dấu tích xanh ✔️.
3. Nhấn **Start** (hoặc gõ `/start`) để bắt đầu.

> **Lưu ý**: Chỉ có duy nhất một @BotFather chính thức của Telegram. Hãy chắc chắn chọn tài khoản có dấu tích xanh.

---

### Bước 2 — Tạo Bot Mới

Gửi lệnh sau cho @BotFather:

```
/newbot
```

BotFather sẽ hỏi bạn lần lượt:

1. **Tên hiển thị** của bot (ví dụ: `My SkyClaw Agent`)  
   Đây là tên người dùng thấy trong Telegram, có thể dùng khoảng trắng.

2. **Username** của bot (ví dụ: `my_skyclaw_bot`)  
   - Phải kết thúc bằng `bot` (ví dụ: `skyclaw_bot`, `SkyclawBot`).  
   - Chỉ dùng chữ cái, số và dấu gạch dưới `_`.  
   - Phải là duy nhất trên toàn Telegram.

---

### Bước 3 — Nhận Token

Sau khi tạo thành công, BotFather sẽ trả về một tin nhắn có dạng:

```
Done! Congratulations on your new bot. You will find it at t.me/my_skyclaw_bot.
You can now add a description, about section and profile picture for your bot,
see /help for a list of commands. By the way, when you've finished creating your
cool bot, ping our Bot Support if you want a better username for it.
Just make sure the bot is fully operational before contacting us.

Use this token to access the HTTP API:
123456789:AABBccDDeeFFggHHiiJJkkLLmmNNoo

Keep your token secure and store it safely, it can be used by anyone to
control your bot.
```

Chuỗi `123456789:AABBccDDeeFFggHHiiJJkkLLmmNNoo` chính là **token bot** của bạn.

---

### Bước 4 — Cấu Hình SkyClaw

Dùng token vừa nhận được để chạy SkyClaw:

**Cách 1 — Biến môi trường (khuyến nghị):**

```bash
export TELEGRAM_BOT_TOKEN="123456789:AABBccDDeeFFggHHiiJJkkLLmmNNoo"
./skyclaw start
```

**Cách 2 — File cấu hình (`~/.skyclaw/config.toml`):**

```toml
[channel.telegram]
enabled = true
token = "123456789:AABBccDDeeFFggHHiiJJkkLLmmNNoo"
```

**Cách 3 — docker-compose (`docker-compose.yml`):**

```yaml
environment:
  - TELEGRAM_BOT_TOKEN=123456789:AABBccDDeeFFggHHiiJJkkLLmmNNoo
```

---

## Quản Lý Token Hiện Có

### Xem lại token của bot đã tạo

```
/mybots
```

BotFather sẽ liệt kê tất cả bot của bạn. Chọn bot cần xem → nhấn **API Token**.

### Tạo token mới (nếu token cũ bị lộ)

```
/mybots → chọn bot → API Token → Revoke current token
```

> ⚠️ **Quan trọng**: Sau khi revoke, token cũ sẽ lập tức hết hiệu lực. Cập nhật ngay token mới vào cấu hình SkyClaw rồi khởi động lại dịch vụ.

---

## Bảo Mật Token

- **Không bao giờ** commit token vào source code hoặc đăng lên mạng.
- Dùng biến môi trường hoặc vault thay vì ghi thẳng vào file config.
- Nếu token bị lộ, hãy revoke ngay lập tức qua @BotFather.
- SkyClaw tự mã hóa token bằng ChaCha20-Poly1305 khi lưu vào vault.

---

## Cài Đặt Tùy Chọn Cho Bot

Sau khi tạo bot, bạn có thể cấu hình thêm qua @BotFather:

| Lệnh | Mô Tả |
|------|-------|
| `/setdescription` | Mô tả ngắn hiển thị trên trang bot |
| `/setabouttext` | Văn bản "About" trong profile bot |
| `/setuserpic` | Ảnh đại diện cho bot |
| `/setprivacy` | Chế độ privacy (nên để `Disable` nếu dùng trong nhóm) |
| `/setcommands` | Danh sách lệnh gợi ý hiển thị khi gõ `/` |

---

## Liên Kết Hữu Ích

- [Tài liệu chính thức Telegram Bot API](https://core.telegram.org/bots)
- [Hướng dẫn Getting Started](getting-started.md)
- [README Tiếng Việt](../../README.vi.md)
