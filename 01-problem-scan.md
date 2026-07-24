# Lab 02 — Problem Scan & Quick Assess

## Phase 1 — SCAN

| # | Công ty thành viên | Lens | Bài toán vận hành quan sát được |
|---:|---|---|---|
| 1 | Xanh SM (GSM) | Tốn thời gian | Điều phối viên phải tra vị trí xe, mức pin và tình trạng trụ sạc trên nhiều màn hình rồi soạn hướng dẫn khi tài xế báo pin yếu; ước tính 15 phút/lượt. |
| 2 | Vinhomes | AI-upgrade | Nhân viên CSKH đọc nội dung tự do, phân loại và chuyển tuyến phản ánh của cư dân; phản hồi mẫu hiện tại khó bao quát ngữ cảnh và sắc thái. |
| 3 | VinFast | Lặp lại | Nhân viên tài chính đối chiếu phiên sạc, hóa đơn và giao dịch thanh toán từ các nguồn khác nhau; ngoại lệ phải kiểm tra thủ công. |
| 4 | Vinmec | Tốn thời gian | Bác sĩ tổng hợp dữ kiện từ bệnh án để soạn bản nháp tóm tắt xuất viện; cần kiểm tra y khoa trước khi phát hành. |
| 5 | Vinpearl / VinWonders | Stakeholder Pain | Nhân viên quầy mất thời gian trả lời lặp lại về điều kiện vé, giờ hoạt động và đổi lịch; khách phải chờ trong giờ cao điểm. |

Danh sách gồm đúng **5 bài toán** và sử dụng đủ bốn lens. Ba bài toán được chọn để quick-assess là **#1, #2 và #4**, dựa trên tần suất giả định, thời gian xử lý và khả năng đặt ranh giới an toàn.

## Phase 2 — QUICK-ASSESS

### Quick Problem Card #1 — Điều phối sự cố pin yếu thực địa

| Thành phần | Nội dung |
|---|---|
| **Bài toán / công ty** | Rút ngắn việc xử lý yêu cầu hỗ trợ pin yếu của tài xế — **Xanh SM (GSM)**. |
| **Actor đang đau** | Điều phối viên bị quá tải; tài xế phải chờ và xe ngừng khai thác. |
| **Workflow hiện tại** | (1) Tài xế gọi tổng đài → (2) điều phối viên ghi nhận biển số, pin và vị trí → (3) tra bản đồ xe → (4) tra trạm sạc/cổng sạc còn khả dụng → (5) soạn hướng dẫn hoặc gọi đội cứu hộ → (6) gửi cho tài xế. |
| **Bottleneck** | Bước 3–5, khoảng **10/15 phút mỗi lượt**; phải chuyển qua nhiều hệ thống và dễ chọn nhầm trạm/cổng sạc. |
| **AI hỗ trợ** | Rule lọc điều kiện an toàn và tương thích; LLM soạn **bản nháp** hướng dẫn từ dữ liệu đã được xác thực; điều phối viên duyệt rồi mới gửi. |
| **Metric pilot** | P90 thời gian từ lúc đủ dữ liệu đến lúc có phương án được duyệt: **15 → ≤3 phút**; **≥98%** phương án đúng trạm/cổng theo bộ test; **100%** tin nhắn có duyệt; **0** đề xuất trạm xa hơn 5 km khi pin <5%. |
| **Quick Architecture** | **LLM Feature + deterministic rules + HITL**, không dùng agent tự trị. |

### Quick Problem Card #2 — Phân loại phản ánh cư dân

| Thành phần | Nội dung |
|---|---|
| **Bài toán / công ty** | Phân loại, ưu tiên và soạn bản nháp phản hồi đầu tiên cho phản ánh cư dân — **Vinhomes**. |
| **Actor đang đau** | Nhân viên CSKH và cư dân chờ phản hồi. |
| **Workflow hiện tại** | (1) Nhận ticket → (2) đọc nội dung/ảnh đính kèm → (3) gán loại và mức ưu tiên → (4) tra chính sách → (5) soạn phản hồi → (6) chuyển ban quản lý hoặc bộ phận kỹ thuật. |
| **Bottleneck** | Bước 2–5, ước tính **8 phút/ticket**; nội dung tự do làm phân loại không nhất quán. |
| **AI hỗ trợ** | LLM trích xuất chủ đề, tóm tắt và soạn bản nháp dựa trên kho chính sách được kiểm soát; rule chuyển ngay các từ khóa an toàn/cháy nổ. |
| **Metric pilot** | **≥90% macro-F1** trên tập gán nhãn; thời gian xử lý trung vị **8 → ≤2 phút**; **100%** ticket an toàn được rule chuyển tuyến; không tự gửi phản hồi. |
| **Quick Architecture** | **LLM Feature + Rule router + HITL**. |

### Quick Problem Card #3 — Bản nháp tóm tắt xuất viện

| Thành phần | Nội dung |
|---|---|
| **Bài toán / công ty** | Tạo bản nháp tóm tắt xuất viện từ dữ liệu bệnh án — **Vinmec**. |
| **Actor đang đau** | Bác sĩ điều trị; bệnh nhân chờ hoàn tất thủ tục. |
| **Workflow hiện tại** | (1) Mở bệnh án → (2) đọc diễn biến, xét nghiệm và thuốc → (3) chọn dữ kiện chính → (4) viết tóm tắt → (5) đối chiếu → (6) ký số/phát hành. |
| **Bottleneck** | Bước 2–4, ước tính **20 phút/hồ sơ**; rủi ro bỏ sót dữ kiện. |
| **AI hỗ trợ** | LLM chỉ tạo bản nháp có trích dẫn tới trường dữ liệu nguồn; bác sĩ đối chiếu và ký. Không đề xuất chẩn đoán hoặc đơn thuốc mới. |
| **Metric pilot** | Thời gian soạn **20 → ≤8 phút**; **≥99% factual consistency** trên trường bắt buộc; **100%** hồ sơ được bác sĩ duyệt; **0** bản nháp tự phát hành. |
| **Quick Architecture** | **LLM Feature + validation rules + bắt buộc HITL**. |
