<p align="center">
  <img src="assets/banner.png" alt="SkyClaw" width="100%">
</p>

# SkyClaw

Runtime AI agent thuần cloud, viết bằng Rust. Tích hợp Telegram. Một binary duy nhất, không cần file cấu hình.

## SkyClaw là gì?

SkyClaw là một **AI agent tự trị chạy trên server của bạn** và giao tiếp qua các ứng dụng nhắn tin như Telegram, Discord, Slack. Agent có thể chạy lệnh shell, duyệt web, đọc/ghi file và tải URL — tất cả thông qua hội thoại tự nhiên.

Không cần web dashboard. Không cần chỉnh file cấu hình. Chỉ cần deploy, dán API key vào Telegram là xong.

---

## Cài Đặt Nhanh (3 bước)

### Bước 1 — Lấy Telegram Bot Token

Bạn cần tạo một Telegram bot và lấy token thông qua **@BotFather**:

1. Mở Telegram, tìm kiếm **@BotFather** (chọn tài khoản có dấu tích xanh ✔️).
2. Gửi lệnh `/newbot` và làm theo hướng dẫn:
   - Đặt **tên hiển thị** cho bot (ví dụ: `My SkyClaw Agent`)
   - Đặt **username** kết thúc bằng `bot` (ví dụ: `my_skyclaw_bot`)
3. BotFather trả về token dạng: `123456789:AABBccDDeeFFggHH...`

> 📖 Xem hướng dẫn chi tiết: [Cách Lấy Telegram Bot Token](docs/dev/telegram-bot-token.md)

### Bước 2 — Build và Chạy

```bash
git clone https://github.com/nagisanzenin/skyclaw.git
cd skyclaw
cargo build --release
export TELEGRAM_BOT_TOKEN="123456789:AABBccDDeeFFggHH..."
./target/release/skyclaw start
```

### Bước 3 — Kết Nối AI Provider

Mở bot trong Telegram và dán API key của nhà cung cấp AI:

```
sk-ant-...          ← Anthropic Claude
sk-...              ← OpenAI GPT
AIzaSy...           ← Google Gemini
```

SkyClaw tự nhận diện nhà cung cấp và bắt đầu hoạt động ngay.

---

## Nhà Cung Cấp AI Được Hỗ Trợ

Dán bất kỳ API key nào dưới đây vào Telegram — SkyClaw tự nhận diện:

| Dạng Key | Nhà Cung Cấp | Model Mặc Định |
|----------|--------------|----------------|
| `sk-ant-*` | Anthropic | claude-sonnet-4-6 |
| `sk-*` | OpenAI | gpt-5.2 |
| `AIzaSy*` | Google Gemini | gemini-3-flash-preview |

---

## Tính Năng

| Công Cụ | Mô Tả |
|---------|-------|
| **Shell** | Chạy bất kỳ lệnh nào trên server |
| **Browser** | Chrome headless — điều hướng, click, gõ phím, chụp ảnh màn hình, trích xuất văn bản |
| **File** | Đọc, ghi, liệt kê file trên server |
| **Web Fetch** | HTTP GET với trích xuất nội dung có kiểm soát token |
| **Messaging** | Gửi cập nhật thời gian thực trong các tác vụ nhiều bước |

### Tự Cấu Hình

SkyClaw có thể tự thay đổi cài đặt qua ngôn ngữ tự nhiên:

- "Đổi model sang claude-opus-4-6"
- "Chuyển sang GPT-5.2"

Cấu hình lưu tại `~/.skyclaw/credentials.toml` — SkyClaw tự đọc và chỉnh sửa file này.

---

## Kiến Trúc

SkyClaw là **Cargo workspace gồm 13 crate**:

```
skyclaw (binary)
├── skyclaw-core         Trait, kiểu dữ liệu, config, lỗi (không có logic nghiệp vụ)
├── skyclaw-gateway      HTTP/WebSocket server (axum), routing, quản lý session
├── skyclaw-agent        Vòng lặp runtime của agent (context → LLM → tools → reply)
├── skyclaw-providers    Tích hợp AI: Anthropic, OpenAI-compatible, Google Gemini
├── skyclaw-channels     Kênh nhắn tin: Telegram, Discord, Slack, WhatsApp, CLI
├── skyclaw-memory       Lưu trữ hội thoại: SQLite, PostgreSQL, Markdown
├── skyclaw-vault        Quản lý bí mật với mã hóa ChaCha20-Poly1305
├── skyclaw-tools        Công cụ tích hợp: shell, browser, file ops, web fetch
├── skyclaw-skills       Registry và thực thi skill
├── skyclaw-automation   Heartbeat và lịch cron
├── skyclaw-observable   Tracing, OpenTelemetry, metrics
├── skyclaw-filestore    Lưu trữ file: local và S3
└── skyclaw-test-utils   Tiện ích test dùng chung
```

### Luồng Xử Lý Tin Nhắn

```
Channel.start() → nhận tin nhắn qua mpsc::channel
  → Gateway router
    → Vòng lặp agent runtime
      → Provider.complete() hoặc Provider.stream()
      ← CompletionResponse (có thể chứa tool_use)
      → Tool.execute() nếu có tool_use
      ← ToolOutput được đưa lại cho provider
    ← Phản hồi cuối cùng
  → Channel.send_message(OutboundMessage)
```

### Nguyên Tắc Thiết Kế

1. **Trait trong core, implementation trong crate riêng** — Tất cả trait dùng chung (`Channel`, `Provider`, `Memory`, `Tool`…) được định nghĩa trong `skyclaw-core`; implementation nằm trong crate tương ứng.
2. **Không phụ thuộc chéo giữa các implementation** — Crate lá (providers, channels, tools, memory) không được phụ thuộc lẫn nhau. Kiểu dùng chung nằm trong `skyclaw-core`.
3. **Feature flag cho dependency tùy chọn** — Kênh theo platform (Telegram, Discord, Slack, WhatsApp) và công cụ (browser) ẩn sau Cargo feature flag.
4. **Pattern Factory** — Mỗi crate expose một hàm `create_*()` để khởi tạo bằng chuỗi tên.

---

## Bảo Mật

- **Auto-whitelist**: Người đầu tiên nhắn tin bot sẽ được thêm vào whitelist. Những người khác bị từ chối.
- **Không mở truy cập công khai**: Whitelist rỗng = từ chối tất cả. Không ai dùng được bot cho đến khi có người đầu tiên kích hoạt.
- **Chỉ dùng ID số**: Whitelist khớp theo Telegram user ID (số), không theo username (username có thể thay đổi).
- **Mã hóa bí mật**: Tất cả API key và credential được mã hóa bằng ChaCha20-Poly1305 trước khi lưu.
- **Sandbox bắt buộc**: Công cụ luôn chạy trong sandbox — không thể tắt.

---

## Tham Khảo CLI

```
skyclaw start              Khởi động gateway daemon
skyclaw chat               Chat tương tác qua CLI
skyclaw status             Xem trạng thái đang chạy
skyclaw config validate    Kiểm tra cấu hình
skyclaw config show        In cấu hình đã được resolve
skyclaw version            Hiển thị thông tin phiên bản
```

---

## Phát Triển

### Yêu Cầu

- Rust 1.82+
- Chrome/Chromium (cho browser tool)
- Telegram bot token

### Lệnh Build và Test

```bash
cargo check --workspace          # Kiểm tra compile nhanh
cargo build --workspace          # Debug build
cargo test --workspace           # Chạy toàn bộ test
cargo clippy --workspace -- -D warnings   # Lint
cargo fmt --all                  # Format code
cargo build --release            # Release build
```

### Feature Flags

| Feature | Nội dung |
|---------|---------|
| `telegram` | Kênh Telegram (teloxide) |
| `discord` | Kênh Discord (serenity + poise) |
| `slack` | Kênh Slack |
| `whatsapp` | Kênh WhatsApp |
| `browser` | Tự động hóa trình duyệt (chromiumoxide) |
| `postgres` | Memory backend PostgreSQL |

```bash
# Build tối giản (chỉ CLI, không browser, không PostgreSQL)
cargo build --no-default-features

# Chỉ Telegram
cargo build --no-default-features --features telegram
```

### Cấu Hình cho Dev

```bash
mkdir -p ~/.skyclaw
cp config/default.toml ~/.skyclaw/config.toml
```

Cấu hình tối thiểu:

```toml
[skyclaw]
mode = "local"

[gateway]
host = "127.0.0.1"
port = 8080

[provider]
name = "anthropic"
api_key = "${ANTHROPIC_API_KEY}"
model = "claude-sonnet-4-6"

[memory]
backend = "sqlite"
```

---

## Cấu Trúc Tài Liệu

```
docs/
  dev/
    getting-started.md      Hướng dẫn setup môi trường dev
    telegram-bot-token.md   Cách lấy Telegram Bot Token từ @BotFather
    architecture.md         Kiến trúc chi tiết, đồ thị phụ thuộc crate
    adding-channel.md       Hướng dẫn thêm kênh nhắn tin mới
    adding-provider.md      Hướng dẫn thêm nhà cung cấp AI mới
  api/
    traits.md            Tài liệu tham khảo các trait cốt lõi
    types.md             Các kiểu dữ liệu dùng chung
    config.md            Tài liệu cấu hình
  ops/
    deployment.md        Hướng dẫn deploy production
    configuration.md     Tài liệu cấu hình đầy đủ
    monitoring.md        Observability và metrics
  runbooks/              Runbook xử lý sự cố vận hành
  architecture/adr/      Architecture Decision Records
```

---

## Docker

```bash
# Build image
docker build -t skyclaw:latest .

# Chạy với docker-compose
docker-compose up -d
```

---

## Giấy Phép

MIT
