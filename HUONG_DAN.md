# AI Báo Cáo PDF – v3.0

Google Sheet tự đọc các báo cáo PDF trong một thư mục Drive bằng AI, ghi số liệu vào các tab kết quả và dựng tab **Tóm tắt báo cáo** cho báo cáo mới nhất. Bản 3.0 hợp nhất:

- **Engine v6 "AI Tự Động"** (đã chạy thật trên CN8-2): mỗi nhóm thông tin = 1 prompt hẹp + 1 schema; vòng xoay nhà cung cấp có model dự phòng; kiểm chéo số liệu; cache theo file + nhóm; đọc kỳ báo cáo từ tên file; đọc PDF bằng chữ (Drive OCR).
- **Bảng điều khiển v2.0**: sidebar có nút cho mọi thao tác; nhóm "Chỉ số tự khai báo" cho báo cáo bất kỳ; lịch tự động chọn thứ/giờ.

## 1. Cài đặt (5 phút, Sheet mới)

1. Tạo Google Sheet mới → **Extensions → Apps Script**.
2. Dán `Code.gs` vào file `Code.gs`.
3. **+ → HTML**, đặt tên đúng `Sidebar`, dán `Sidebar.html`.
4. **Project Settings** (⚙️) → tick *Show "appsscript.json" manifest file* → mở `appsscript.json`, dán nội dung file kèm theo (có đủ scope Drive/Docs để bóc chữ PDF).
5. Lưu, quay về Sheet, tải lại trang. Menu **🚀 AI Báo Cáo → 🎛️ Mở bảng điều khiển**. Lần đầu Google hỏi cấp quyền → Allow.

> Muốn có nút trên sheet: **Insert → Drawing** vẽ nút → ⋮ → **Assign script** → `moBangDieuKhien`.

## 2. Bảng điều khiển

| Tab | Chức năng |
|---|---|
| ▶️ **Quét** | Checklist sẵn sàng · **BẮT ĐẦU QUÉT PDF MỚI** · mở các tab kết quả · dựng lại tab từ cache (không gọi API) |
| 🔑 **API** | Kéo ▲▼ thứ tự ưu tiên · bật/tắt từng nhà cung cấp · **model chính + model dự phòng** · nhập/xóa key · **danh sách model dùng được với key** (bấm tên để điền) · **Kiểm tra kết nối** (ping cả 2 model, báo rõ HẠN MỨC / KEY SAI / SAI TÊN MODEL / QUÁ TẢI) |
| 📁 **Thư mục** | Dán link → **Xác nhận** (hiện tên, số PDF, số file chờ đọc kèm kỳ báo cáo) · số ngày lùi · thư mục con · **Thử bóc chữ** file mới nhất · xóa cache |
| 🧩 **Dữ liệu** | Tick bật/tắt 7 nhóm thông tin · sửa danh sách chỉ số · cách đọc PDF · email thông báo |
| ⏰ **Lịch** | Bật quét hằng tuần, chọn thứ + giờ (mặc định **Thứ Sáu 05:00** giờ VN) |
| 📝 **Nhật ký** | 50 lượt gọi API gần nhất với kết luận từng lượt |

## 3. Bảy nhóm thông tin

| Nhóm | Dành cho | Mặc định |
|---|---|---|
| Tiến độ tổng (% KH / % thực tế / trễ-sớm) | CN8-2 VICONS | Bật – bắt buộc cho tab Tổng hợp tuần |
| Công tác A.1/A.2 + KH tuần sau | CN8-2 VICONS | Bật |
| Nhân lực (mục F) | CN8-2 VICONS | Tắt |
| 5 mốc MS Project | CN8-2 VICONS | Tắt |
| Công tác chậm + hồ sơ Rev 3+ | CN8-2 VICONS | Tắt |
| **Chỉ số tự khai báo** | Báo cáo bất kỳ | Bật |
| Hạng mục & vấn đề | Báo cáo bất kỳ | Tắt |

Mỗi nhóm bật = 1 lần gọi API cho mỗi file. Bật thêm nhóm sau, script chỉ gọi cho nhóm còn thiếu.

### Chỉ số tự khai báo
Sheet **"Chỉ số cần lọc"** có sẵn 13 chỉ số mẫu; sửa/xóa/thêm tùy ý. Cột *Mô tả* càng rõ (ghi cả tên tiếng Anh, viết tắt) AI tìm càng đúng. Mỗi chỉ số nhận một trong ba trạng thái: **✅ Tìm thấy** (kèm trích dẫn, trang) · **⚠️ Cần kiểm tra** · **❌ Không tìm thấy** (giá trị ghi "Không có"). Tab **Bảng so sánh chỉ số**: 1 dòng = 1 báo cáo, 1 cột = 1 chỉ số.

## 4. Các tab

`Tóm tắt báo cáo` (báo cáo mới nhất: nhận xét nhanh, tiến độ tổng, công tác đạt/chưa đạt, chỉ số, hạng mục) · `Tổng hợp tuần` · `Công tác chính` · `Mốc tiến độ MSP` · `Chậm trễ & hồ sơ` · `Kết quả chỉ số` · `Bảng so sánh chỉ số` · `Hạng mục - Vấn đề` · `Nhật ký API` · `_Cache` (ẩn, JSON thô – không sửa tay).

## 5. Cách hệ thống hoạt động

- **Vòng xoay**: nhóm nào cũng thử nhà cung cấp số 1. Hỏng vì 429 (hết hạn mức), 401/403 (key), 404 (model), 5xx (quá tải) hoặc quá 70 giây → thử model dự phòng → vẫn hỏng thì nhảy sang số 2… Nhà cung cấp hỏng bị loại khỏi phần còn lại của lượt chạy. Model không nhận `json_schema` được tự hạ xuống `json_object` và ghi nhớ.
- **Đọc PDF**: bóc chữ bằng Drive OCR (nhẹ, rẻ). Không đủ chữ (PDF scan) → gửi nguyên file cho **Claude** hoặc **Gemini** (các bên khác chỉ nhận chữ). Có thể ép "chỉ chữ" hoặc "chỉ PDF" ở tab Dữ liệu.
- **Cache & chống trùng**: khóa = File ID + thời điểm sửa. File bị thay bản mới → đọc lại. Các tab kết quả luôn được dựng lại từ cache, không ghi trùng. Mỗi File ID chỉ hiện phiên bản mới nhất.
- **Giới hạn Apps Script**: mỗi lượt đọc tối đa 3 file, dừng nhận file mới sau 150 giây; còn file thì tự chạy tiếp sau 1 phút; model quá tải thì tự thử lại sau 5 phút (tối đa 4 lần).
- **Kỳ báo cáo** đọc từ tên file: `WR-33`, `Week 33`, `Tuần 33`, `MR-10`, `2026-11-05`, `05-11-2026`, `05/11/2026`.
- **Kiểm chéo**: % ngoài 0–100 bị từ chối; thực tế > kế hoạch mà "Trễ" dương → từ chối; cộng 12 chức danh lệch Tổng > 2 người → từ chối. Bị từ chối thì thử model/nhà cung cấp kế tiếp.

## 6. Bảo mật & lưu ý

- API key lưu trong **User Properties** của tài khoản đang đăng nhập: không nằm trong code, không hiện lại trong giao diện, không ghi vào nhật ký (nhật ký chỉ ghi 300 ký tự đầu của lỗi API).
- **Lịch tự động chạy bằng tài khoản của người bật lịch**, nên người đó phải là người đã nhập key.
- Báo cáo được gửi tới các nhà cung cấp đang bật. Tài liệu nhạy cảm (công nợ, tranh chấp) → chỉ bật nhà cung cấp có cam kết không dùng dữ liệu để huấn luyện.
- **Tên model mặc định** chưa được kiểm chứng với key của anh: Claude `claude-sonnet-5-5` / `claude-haiku-5-5`; OpenRouter `openrouter/auto` / `openai/gpt-oss-120b`; Cloudflare `@cf/meta/llama-3.3-70b-instruct-fp8-fast`; Groq `openai/gpt-oss-120b` / `llama-3.3-70b-versatile`; Gemini `gemini-2.5-flash`. Dùng nút **"Model dùng được với key này"** và **"Kiểm tra kết nối"** ở tab API để chốt tên đúng trước lần quét đầu.
- Tham chiếu: `reference/v6_cn82_weekly_report.gs` là bản v6 gốc, giữ để đối chiếu.
