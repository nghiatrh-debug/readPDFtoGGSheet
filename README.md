# readPDFtoGGSheet – AI Báo Cáo PDF v3.0

Google Apps Script gắn Google Sheet: tự đọc báo cáo PDF trong thư mục Drive bằng AI (vòng xoay Claude / OpenRouter / Cloudflare / Groq / Gemini), bóc số liệu theo từng nhóm, cache theo file, dựng tab kết quả và tab Tóm tắt. Có bảng điều khiển (sidebar) để nhập API key, chọn thứ tự ưu tiên, model chính/dự phòng, thư mục quét, nhóm dữ liệu, chỉ số tự khai báo cho báo cáo bất kỳ và lịch quét hằng tuần.

| File | Dùng để |
|---|---|
| `Code.gs` | Toàn bộ logic (dán vào Apps Script) |
| `Sidebar.html` | Bảng điều khiển (tạo file HTML tên `Sidebar`) |
| `appsscript.json` | Manifest: múi giờ + OAuth scopes |
| `HUONG_DAN.md` | Hướng dẫn cài đặt, vận hành, cách hệ thống hoạt động |
| `reference/v6_cn82_weekly_report.gs` | Bản v6 gốc (CN8-2), giữ để đối chiếu |

Cài đặt chi tiết: xem [HUONG_DAN.md](HUONG_DAN.md).
