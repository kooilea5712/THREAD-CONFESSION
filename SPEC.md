# ĐẶC TẢ: Tool "Thread → Video" (cào post/comment hot, ghép lên video nền, chèn meme)

> Tài liệu này dành cho AI lập trình (STUDIO AI) đọc và tự xây dựng tool. Hãy làm đúng theo từng mục, theo thứ tự các mốc (Milestone) ở cuối.

---

## 1. Mục tiêu

Tạo tool chạy trên máy cá nhân (Windows 11) giúp tự động sản xuất video dọc kiểu "thread confession / drama Threads" đang viral trên TikTok:

1. **Lấy nội dung**: lấy bài post + các comment "hot" từ Threads (và có thể mở rộng Reddit, Facebook group, VOZ…).
2. **Dựng thẻ (card)**: vẽ bài post và từng comment thành ảnh thẻ đẹp, dễ đọc.
3. **Ghép video**: đặt các thẻ lên **video nền do người dùng cung cấp** (gameplay, cảnh đẹp, nấu ăn…), thẻ hiện lần lượt theo nhịp đọc.
4. **Chèn meme**: tự chọn meme (ảnh/GIF/video ngắn + âm thanh) hợp cảm xúc từng comment, cho bật ra đúng lúc "punchline".
5. **Xuất video** 9:16 sẵn sàng đăng TikTok/Reels/Shorts, kèm gợi ý caption + hashtag.

### Phong cách tham khảo (từ các video mẫu)
- Video dọc 1080×1920, nền là video chạy liên tục (thường làm tối/mờ nhẹ).
- Thẻ post ở giữa/trên màn hình: avatar tròn, tên, thời gian, nội dung, số like/reply.
- Comment hiện **từng cái một**, hiệu ứng pop/slide, có thể chồng lên nhau hoặc thay thế cái trước.
- Meme bật lên sau comment "gắt" (mặt ngạc nhiên, cười, "bruh", hiệu ứng zoom + tiếng "vine boom"…).
- Có thể có giọng đọc AI (TTS) tiếng Việt + phụ đề, nhạc nền nhỏ.
- 3 giây đầu phải có **hook** (tiêu đề giật, bài post chính).

---

## 2. Phạm vi & nguyên tắc bắt buộc

- **Ưu tiên nguồn hợp lệ**: dùng Threads API chính thức nếu có token; nếu không thì cho phép (a) dán URL bài viết để tool đọc trang công khai, (b) dán text thủ công. Cào phải có giới hạn tốc độ (≥ 3–5 giây/request, ngẫu nhiên), cache kết quả, không vượt đăng nhập/CAPTCHA.
- **Riêng tư**: mặc định **làm mờ/ẩn avatar và đổi username** thành tên giả (vd: `user_8xx2`). Có bật/tắt trong cài đặt.
- **Lọc nội dung**: bỏ comment chứa số điện thoại, địa chỉ, thông tin cá nhân, từ ngữ thù ghét; có danh sách từ cấm tùy chỉnh.
- **Thẻ dùng giao diện trung tính** (giống phong cách mạng xã hội chung), không chèn logo thương hiệu thật.
- Meme và video nền là **file do người dùng cung cấp** trong thư mục local; tool không tự tải meme có bản quyền.

---

## 3. Công nghệ đề xuất

| Phần | Công nghệ |
|---|---|
| Ngôn ngữ | Python 3.11+ |
| Giao diện | Web UI local: FastAPI + HTML/JS đơn giản (hoặc Gradio/Streamlit nếu nhanh hơn) |
| Lấy dữ liệu | Threads Graph API (`httpx`) ; fallback Playwright (headless Chromium) đọc trang công khai |
| Vẽ thẻ | Template HTML/CSS → chụp PNG bằng Playwright (đẹp, hỗ trợ emoji & tiếng Việt). Fallback: Pillow |
| Ghép video | FFmpeg (gọi qua `subprocess`, dùng `filter_complex`) ; MoviePy chỉ dùng nếu cần |
| Giọng đọc (tùy chọn) | `edge-tts` (giọng `vi-VN-HoaiMyNeural`, `vi-VN-NamMinhNeural`) |
| Chọn meme / tóm tắt / viết hook | LLM (Claude API, model cấu hình được) ; fallback luật theo từ khóa |
| Lưu trữ | SQLite (bài đã cào, đã dùng, lịch sử render) |
| Cấu hình | `config.yaml` + `.env` (API key) |

Yêu cầu FFmpeg có sẵn trong PATH; tool phải kiểm tra và báo lỗi rõ ràng nếu thiếu.

---

## 4. Cấu trúc thư mục

```
thread-video-tool/
├─ app/
│  ├─ main.py               # FastAPI + route UI
│  ├─ config.py             # đọc config.yaml / .env
│  ├─ sources/
│  │  ├─ base.py            # interface Source: search(), fetch_post(url)
│  │  ├─ threads_api.py     # Threads API chính thức
│  │  ├─ threads_web.py     # Playwright đọc trang công khai (fallback)
│  │  └─ manual.py          # nhập tay / dán text / import JSON
│  ├─ ranking.py            # tính điểm "hot", chọn comment
│  ├─ moderation.py         # lọc PII, từ cấm, ẩn danh hóa
│  ├─ llm.py                # gọi LLM: phân loại cảm xúc, chọn meme, viết hook/caption
│  ├─ cards/
│  │  ├─ render.py          # HTML → PNG
│  │  └─ templates/         # post.html, comment.html, title.html, style.css (light/dark)
│  ├─ memes.py              # đọc thư viện meme + chọn meme
│  ├─ tts.py                # tạo giọng đọc + đo thời lượng
│  ├─ timeline.py           # xếp lịch xuất hiện của thẻ/meme/âm thanh
│  ├─ compose.py            # build lệnh FFmpeg & render
│  └─ db.py                 # SQLite
├─ assets/
│  ├─ backgrounds/          # video nền người dùng bỏ vào
│  ├─ memes/                # ảnh/gif/mp4 meme + memes.json
│  ├─ sfx/                  # vine_boom.mp3, pop.mp3, whoosh.mp3...
│  ├─ music/                # nhạc nền
│  └─ fonts/                # Be Vietnam Pro, Inter, Noto Color Emoji
├─ output/                  # video xuất ra + caption.txt
├─ config.yaml
├─ .env.example
├─ requirements.txt
└─ README.md                # hướng dẫn cài & dùng bằng tiếng Việt
```

---

## 5. Mô hình dữ liệu

```python
class Post:
    id: str; source: str; url: str
    author: str; avatar_url: str | None
    text: str; images: list[str]
    likes: int; replies: int; reposts: int
    created_at: datetime
    comments: list["Comment"]

class Comment:
    id: str; author: str; avatar_url: str | None
    text: str; likes: int; created_at: datetime
    parent_id: str | None          # để giữ reply lồng nhau 1 cấp
    emotion: str | None            # LLM gán: funny, shock, savage, sad, wholesome, cringe, angry, relatable
    punchline: bool = False        # comment "gắt" → chèn meme

class Meme:
    file: str; type: "image"|"gif"|"video"
    tags: list[str]                # cảm xúc + từ khóa
    sfx: str | None                # âm thanh đi kèm
    duration: float = 1.5
    position: "center"|"top"|"bottom"|"corner"
    chroma_key: str | None         # vd "#00FF00" cho meme nền xanh
```

`assets/memes/memes.json` ví dụ:
```json
[
  {"file": "shock_cat.png", "type": "image", "tags": ["shock", "bất ngờ", "hả"], "sfx": "vine_boom.mp3", "duration": 1.2, "position": "center"},
  {"file": "laugh_guy.mp4", "type": "video", "tags": ["funny", "cười", "haha"], "position": "bottom", "chroma_key": "#00FF00"}
]
```
Nếu không có `memes.json`, tool tự quét thư mục và lấy tag từ **tên thư mục con** (vd `memes/shock/*.png` → tag `shock`).

---

## 6. Chức năng chi tiết

### 6.1. Lấy nội dung (Sources)
- **Tìm theo từ khóa/hashtag**: vd "người yêu cũ", "confession", "drama công sở"; lọc theo khoảng thời gian (24h / 7 ngày).
- **Theo URL**: dán 1 hoặc nhiều link bài viết.
- **Nhập tay**: ô text cho post + danh sách comment (mỗi dòng 1 comment, cú pháp `tên | nội dung | like`).
- Lấy tối đa N comment (mặc định 50) rồi xếp hạng.
- Lưu mọi bài vào SQLite, đánh dấu "đã dùng" để không làm trùng video.

### 6.2. Xếp hạng "hot"
- Điểm post: `score = (likes + 2*replies + 3*reposts) / (giờ_kể_từ_đăng + 2) ^ 1.5`.
- Chọn comment: lấy theo like, ưu tiên comment ngắn (≤ 220 ký tự), đa dạng cảm xúc, loại trùng ý.
- Mặc định 5–8 comment / video, sao cho tổng thời lượng 30–75 giây (cấu hình được).
- Người dùng có thể tick/bỏ tick, kéo thả đổi thứ tự, sửa chữ trước khi render.

### 6.3. LLM (tùy chọn, có fallback)
Một lần gọi cho mỗi video, trả về JSON:
```json
{
  "hook_title": "Bạn gái đòi chia tay vì tôi ăn bún đậu không chấm mắm tôm 😭",
  "comments": [{"id": "c1", "emotion": "savage", "punchline": true, "meme_tag": "shock"}],
  "caption": "...", "hashtags": ["#thread", "#confession", "#drama"]
}
```
- Không có API key → dùng luật từ khóa (`haha`, `=))`, `vãi`, `ủa`, `trời ơi`…) để gán cảm xúc, comment nhiều like nhất = punchline.
- Validate JSON; nếu lỗi thì thử lại 1 lần rồi fallback.

### 6.4. Vẽ thẻ
- Kích thước thẻ rộng 920px (trên khung 1080), bo góc 28px, đổ bóng, padding 32px.
- Thành phần: avatar tròn (hoặc avatar mặc định nếu ẩn danh), tên + dấu tích (tùy chọn), thời gian ("2 giờ"), nội dung (font 40–46px, tự xuống dòng, hỗ trợ emoji màu), hàng icon tim/bình luận/chia sẻ + số.
- Chủ đề **sáng** và **tối**; chọn được màu nhấn.
- Text quá dài → tự giảm cỡ chữ tới mức tối thiểu 34px, sau đó chia thành 2 thẻ.
- Comment reply hiển thị lùi lề + đường nối như giao diện thread.
- Xuất PNG nền trong suốt, độ phân giải 2x rồi scale để nét.

### 6.5. Video nền
- Người dùng chọn 1 file hoặc "ngẫu nhiên" trong `assets/backgrounds/`.
- Tự crop/scale về 1080×1920 (cover, căn giữa), bắt đầu tại điểm ngẫu nhiên, **loop** nếu ngắn hơn video.
- Tùy chọn: làm tối (brightness −0.15), làm mờ (boxblur), tắt tiếng gốc hoặc giữ âm lượng thấp.

### 6.6. Dòng thời gian (timeline)
1. **0–3s Hook**: thẻ tiêu đề + bài post chính, hiệu ứng zoom-in nhẹ, SFX "whoosh".
2. **Mỗi comment**: thời lượng = `max(2.5s, số_ký_tự / 15)` (hoặc = độ dài giọng TTS + 0.4s). Hiệu ứng vào: pop (scale 0.85→1.0 trong 0.2s) hoặc slide từ dưới lên. Kèm SFX "pop".
3. **Meme**: chèn sau comment có `punchline=true` (hoặc theo xác suất cấu hình, tối đa 1 meme / 2 comment), xuất hiện 1–1.5s, kèm SFX riêng, có thể rung màn hình nhẹ (shake) và zoom.
4. **Kết**: thẻ "Bạn nghĩ sao? Comment nhé 👇" 2s.
- Chế độ hiển thị comment: `replace` (thay thế) hoặc `stack` (xếp chồng tối đa 3 thẻ, cái cũ đẩy lên).

### 6.7. Âm thanh
- TTS (bật/tắt): đọc tiêu đề + post + comment; chuẩn hóa text (bỏ emoji, đọc "=))" là "cười", viết tắt "ny" → "người yêu", "ko" → "không"… có từ điển tùy chỉnh).
- Nhạc nền: âm lượng 10–15%, tự **ducking** khi có giọng đọc (`sidechaincompress`).
- Chuẩn hóa loudness đầu ra −14 LUFS (`loudnorm`).

### 6.8. Phụ đề (tùy chọn)
- Khi bật TTS: tạo phụ đề từng cụm 3–5 từ (từ word-boundary của edge-tts), chữ trắng viền đen, đặt dưới thẻ, burn vào video (ASS).

### 6.9. Render (FFmpeg)
- Xây **một** lệnh `filter_complex`: nền → overlay từng PNG với `enable='between(t,a,b)'` và biểu thức scale/fade cho hiệu ứng → overlay meme (GIF/video dùng `-ignore_loop 0` / `colorkey` cho nền xanh) → trộn audio (`amix`/`adelay`).
- Đầu ra: MP4 H.264 (`libx264`, `-crf 20`, `-preset medium`, `yuv420p`), AAC 192k, 30fps, 1080×1920.
- Render chạy nền, có thanh tiến trình (đọc `-progress pipe:1`), cho phép hủy.
- Kèm file `output/<tên>.txt`: caption + hashtag + link nguồn.
- **Xem trước nhanh**: render bản 540×960, `-preset ultrafast` trong < 30s.

### 6.10. Hàng loạt (batch)
- Chọn từ khóa + số lượng video (vd 10) → tool tự cào, chọn bài hot chưa dùng, render lần lượt với nền/meme ngẫu nhiên khác nhau.

---

## 7. Giao diện Web UI (localhost:8000)

1. **Nguồn**: tab Từ khóa / URL / Nhập tay → nút "Lấy bài".
2. **Danh sách bài**: bảng bài viết với điểm hot, like, số comment, trạng thái đã dùng.
3. **Biên tập**: chọn post → danh sách comment có checkbox, kéo thả thứ tự, sửa text, đổi cảm xúc, chọn meme thủ công (dropdown có ảnh thumbnail).
4. **Thiết lập video**: video nền, theme thẻ, bật/tắt TTS + giọng, nhạc, chế độ replace/stack, ẩn danh, độ dài mục tiêu.
5. **Xem trước** (bản nhẹ) & **Render** (bản đầy đủ) + tiến trình + nút tải/ mở thư mục.
6. **Thư viện**: quản lý video nền, meme (gán tag), SFX, nhạc.
7. **Cài đặt**: API key, model LLM, giới hạn tốc độ cào, từ cấm, từ điển TTS.

Có cả **CLI** cho chạy tự động:
```bash
python -m app.cli make --url "https://www.threads.net/@x/post/abc" --bg random --tts on
python -m app.cli batch --keyword "người yêu cũ" --count 5
```

---

## 8. `config.yaml` mẫu

```yaml
video: {width: 1080, height: 1920, fps: 30, target_seconds: [30, 75]}
background: {folder: assets/backgrounds, darken: 0.15, blur: 0, keep_audio: false}
cards: {theme: dark, width: 920, font: "Be Vietnam Pro", min_font_px: 34, anonymize: true}
comments: {max: 7, max_chars: 220, mode: replace, reading_cps: 15}
memes: {folder: assets/memes, max_ratio: 0.5, default_duration: 1.3}
tts: {enabled: true, voice: vi-VN-HoaiMyNeural, rate: "+10%"}
music: {folder: assets/music, volume: 0.12, ducking: true}
llm: {enabled: true, provider: anthropic, model: claude-sonnet-5-5}
scrape: {min_delay_s: 3, max_delay_s: 6, cache_hours: 12}
moderation: {block_words: [], strip_pii: true}
```

---

## 9. Xử lý lỗi bắt buộc
- Thiếu FFmpeg / Playwright browser → hướng dẫn cài ngay trên UI.
- Cào thất bại (bị chặn, đổi giao diện) → báo rõ, gợi ý dùng URL/nhập tay; không crash.
- Thư mục nền/meme trống → báo lỗi thân thiện.
- Emoji/ký tự lạ không render → fallback font.
- Ghi log vào `logs/app.log`.

---

## 10. Kiểm thử & tiêu chí nghiệm thu
- Unit test: ranking, moderation (ẩn số điện thoại), timeline (tổng thời lượng đúng), parser nhập tay, chọn meme theo tag.
- Test tích hợp: từ 1 post nhập tay + 1 video nền mẫu 10s + 2 meme → xuất MP4 hợp lệ (kiểm bằng `ffprobe`: 1080×1920, có audio, thời lượng 20–60s).
- Nghiệm thu khi:
  - [ ] Dán 1 URL hoặc text → có video hoàn chỉnh trong ≤ 3 phút trên máy thường.
  - [ ] Chữ tiếng Việt có dấu + emoji hiển thị đúng, nét.
  - [ ] Meme xuất hiện đúng sau comment punchline, có âm thanh.
  - [ ] Video nền loop mượt, không méo hình.
  - [ ] Ẩn danh hoạt động khi bật.
  - [ ] Batch 3 video chạy không cần thao tác.

---

## 11. Các mốc thực hiện (làm theo thứ tự, mỗi mốc phải chạy được)

1. **M1 – Lõi offline**: nhập tay → vẽ thẻ PNG → ghép lên video nền bằng FFmpeg (chưa meme, chưa TTS). CLI chạy được.
2. **M2 – Meme & SFX**: thư viện meme, chọn theo luật từ khóa, hiệu ứng pop/zoom/shake.
3. **M3 – TTS, nhạc, phụ đề**.
4. **M4 – Nguồn dữ liệu**: Threads API + fallback đọc URL công khai, ranking, SQLite, moderation.
5. **M5 – LLM**: hook, cảm xúc, chọn meme, caption.
6. **M6 – Web UI** đầy đủ + xem trước + batch.
7. **M7 – README tiếng Việt**, script cài đặt `setup.bat`, test.

Sau mỗi mốc: chạy test, render 1 video mẫu vào `output/samples/`, và báo cáo ngắn những gì đã xong.
