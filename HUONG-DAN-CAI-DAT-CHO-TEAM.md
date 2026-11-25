# Cài CIT Voice Studio cho team (máy muse-6 / máy khác)

Hướng dẫn ngắn để tự cài CIT Voice Studio 1.0.7 trên máy của mình, phục vụ
làm narration (lời thoại) cho video demo của team muse-teamwork.
Repo gốc: https://github.com/Cuongyd196/cit-voice-studio (Apache 2.0).
Fork này của team: https://github.com/qsang2025/cit-voice-studio

## CIT Voice Studio là gì

App chuyển văn bản thành giọng nói tiếng Việt, miễn phí, mã nguồn mở,
chạy offline trên máy, không cần tài khoản. 25 giọng Việt (Bắc/Trung/Nam),
nhân bản giọng từ mẫu 3–8 giây, có API tại http://127.0.0.1:8001 để gọi
từ script.

Team đang thử giọng Trúc Ly (nữ Bắc, tự nhiên) với mô hình
VieNeu-TTS v3 Turbo d3 để so với giọng MAI_03 hiện tại — xem kết quả
so sánh trước khi chốt dùng cho video thật.

## Yêu cầu máy

- Ubuntu/Debian 64-bit (hoặc Windows/macOS — xem README gốc).
- RAM 8 GB trở lên, ổ trống 5 GB, CPU 4 nhân trở lên.
- Không cần card NVIDIA (bản CPU đủ dùng; bản GPU chỉ khi có NVIDIA).

## Cài đặt (Linux)

1. Tải file `CIT-Voice-Studio-v1.0.7-amd64.deb` tại trang Releases của
   fork này: https://github.com/qsang2025/cit-voice-studio/releases/tag/1.0.7
   (file ~457 MB; sha256 `5964d4d1ed5b54d013ffebc67919ec77bfa570776d78714e5d80cb742124809d`)
2. Cài:
   ```bash
   sudo apt install ./CIT-Voice-Studio-v1.0.7-amd64.deb
   ```
3. Gỡ (khi cần): `sudo apt remove cit-voice-studio`

## Chạy server

```bash
# Chạy nền (sống độc lập với terminal):
setsid nohup /opt/cit-voice-studio/CIT-Voice-Studio > ~/cit-voice-server.log 2>&1 < /dev/null &

# Kiểm tra sống:
curl -s http://127.0.0.1:8001/health
# Trả về {"status":"ok",...} là chạy tốt.

# Dừng:
pkill -f 'CIT-Voice-Studio'
```

Giao diện web + tài liệu API: http://127.0.0.1:8001
(đầy đủ tại http://127.0.0.1:8001/docs). Lần đầu mở giao diện cần đồng
ý điều khoản; gọi API thì không cần.

## Tải mô hình VieNeu-TTS v3 Turbo d3 (bản team đang thử)

Trong giao diện: **Cài đặt → Mô hình**, chọn
`VieNeu-TTS-v3-Turbo d3 (by Blaze AI)` bản CPU (~520 MB), bấm tải.
Hoặc gọi API:

```bash
curl -s -X POST http://127.0.0.1:8001/api/models/download \
  -H 'Content-Type: application/json' \
  -d '{"model_id": "VieNeu-TTS-v3-Turbo d3 (fp32)", "auto_apply": true}'
```

Chờ tới khi trạng thái `ready` 100% (tải theo từng file, có lúc tưởng
đứng yên ở 32% rồi nhảy lên 78% — không phải lỗi). Kiểm tra mô hình
đang chạy: `curl -s http://127.0.0.1:8001/health` (trường `engine`).

Mô hình mặc định kèm bộ cài là `VieNeu-TTS-v3-Turbo (int8)` (nhẹ hơn
nhiều). Đổi lại bằng `POST /api/models/load` hoặc trong giao diện.

## Tạo giọng thử qua API

```bash
curl -s -X POST http://127.0.0.1:8001/api/tts/generate \
  -H 'Content-Type: application/json' \
  -d '{"text": "Xin chào, đây là giọng nói tiếng Việt.", "voice_id": "Trúc Ly", "format": "wav"}' \
  -o thu.wav

# Kiểm chứng file là audio thật:
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1 thu.wav
```

Danh sách 25 giọng: `curl -s http://127.0.0.1:8001/api/voices`
Giọng nữ: Trúc Ly, Mai Anh, Thùy Dung, Ngọc Huyền, Ngọc Trân, Ngọc Linh,
Đoan Trang, Thục Đoan, Mỹ Duyên, Quỳnh Anh, Kim Thanh.
Thẻ cảm xúc `[cười]`, `[thở dài]` và thẻ nghỉ `[nghỉ 2s]` dùng được qua API.

## Lưu ý đã kiểm chứng (26/11/2026, máy ảo muse-main)

- Tốc độ tạo giọng bản fp32 trên CPU yếu khoảng 0,55–0,65x thời gian thực
  (7,2 giây audio mất 12,8 giây tạo). Câu ngắn không vấn đề; bài dài nên
  chia chương hoặc dùng tab Tạo hàng loạt trong giao diện.
- Server d3 fp32 chiếm ~1,2–1,9 GB RAM. Máy ít RAM thì dùng mô hình INT8
  mặc định thay vì d3.
- Truy cập từ máy khác mặc định TẮT — chỉ bật trong mạng tin cậy
  (Cài đặt → Cho máy khác truy cập API), vì khóa đi qua HTTP không mã hoá.
- Đừng commit file MP4/WAV lớn vào git của repo muse-teamwork; nộp file
  theo gate của skill video-demo-production.
