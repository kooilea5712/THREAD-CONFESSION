# PROMPT dán vào STUDIO AI

(Đính kèm file `SPEC.md` cùng prompt này.)

---

Bạn là kỹ sư phần mềm cấp cao chuyên Python, FFmpeg và tự động hóa video. Hãy xây dựng cho tôi một tool chạy local trên Windows 11 tên là **"Thread → Video"**, đúng theo đặc tả trong file `SPEC.md` đính kèm.

**Tool làm gì:** lấy bài post và các comment "hot" từ Threads (qua API chính thức, dán URL, hoặc nhập tay), vẽ chúng thành các thẻ đẹp giống giao diện mạng xã hội, ghép lần lượt lên **video nền do tôi cung cấp**, tự chèn **meme + hiệu ứng âm thanh** đúng lúc comment "gắt", có thể thêm giọng đọc AI tiếng Việt + phụ đề + nhạc nền, rồi xuất video dọc 1080×1920 sẵn sàng đăng TikTok, kèm caption và hashtag. Phong cách giống các video TikTok "thread confession / drama" đang viral: 3 giây đầu có hook, comment hiện từng cái với hiệu ứng pop, meme bật ra kèm tiếng "vine boom".

**Yêu cầu khi làm:**
1. Đọc kỹ toàn bộ `SPEC.md` trước khi viết code. Chỗ nào không rõ thì chọn phương án hợp lý nhất, ghi lại trong `README.md` mục "Giả định".
2. Làm **theo đúng thứ tự các mốc M1 → M7** ở mục 11. Mỗi mốc phải chạy được, có test, và render 1 video mẫu vào `output/samples/` trước khi sang mốc tiếp theo.
3. Dùng Python 3.11+, FFmpeg, Playwright (vẽ thẻ HTML→PNG và đọc trang công khai), FastAPI cho web UI, SQLite, `edge-tts`. LLM là tùy chọn và phải có fallback chạy không cần API key.
4. Tôn trọng các nguyên tắc ở mục 2: giới hạn tốc độ cào, không vượt đăng nhập/CAPTCHA, mặc định ẩn danh avatar + username, lọc thông tin cá nhân, không dùng logo thương hiệu thật.
5. Hỗ trợ hoàn hảo tiếng Việt có dấu và emoji màu trên thẻ.
6. Viết code sạch, chia module như cấu trúc thư mục trong spec, có `requirements.txt`, `.env.example`, `config.yaml`, `setup.bat` (cài thư viện + Playwright Chromium + kiểm tra FFmpeg).
7. `README.md` viết bằng **tiếng Việt**, hướng dẫn từng bước cho người không chuyên: cài đặt, bỏ video nền/meme vào thư mục nào, gán tag meme, chạy web UI, chạy CLI và batch.
8. Mọi lỗi phải hiện thông báo dễ hiểu trên UI, không crash.

Bắt đầu bằng việc tóm tắt lại kế hoạch của bạn trong 10 dòng, rồi thực hiện mốc M1.
