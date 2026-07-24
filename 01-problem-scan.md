# 01 — Problem Scan (Cá nhân)
### AI Product Scoping — Vin Smart Future

---

## 🔍 Phase 1 — SCAN

### 📝 List bài toán của tôi:

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|---|---|---|
| 1 | Xanh SM | Stakeholder Pain | Tài xế nhận gợi ý trạm sạc không tính đến mức pin hiện tại và khoảng cách an toàn, dẫn đến tài xế bị kẹt pin giữa đường hoặc phải tự tìm trạm sạc ngoài hệ thống. |
| 2 | VinFast | Lặp lại | Nhân viên bảo hành đối soát thủ công log lỗi pin từ xe với phiếu yêu cầu bảo hành để xác định lỗi có nằm trong diện bảo hành hay không. |
| 3 | Vinhomes | Tốn thời gian | Nhân viên CSKH tòa nhà đọc và soạn phản hồi thủ công cho từng đánh giá 1-2 sao của cư dân trên app quản lý căn hộ. |
| 4 | Vinmec | AI-upgrade | Tổng đài viên tiếp nhận cuộc gọi đặt lịch khám phải hỏi thủ công triệu chứng để xếp đúng chuyên khoa, gây chờ lâu và xếp sai khoa ở giờ cao điểm. |
| 5 | Vinpearl | AI-upgrade | Khách quốc tế đặt vé/dịch vụ tại VinWonders qua chatbot CSKH hiện tại chỉ trả lời rập khuôn bằng tiếng Việt, không xử lý được câu hỏi đa ngôn ngữ hoặc yêu cầu phức tạp (đổi vé, hoàn tiền). |

---

## 🃏 Phase 2 — QUICK-ASSESS

### QUICK PROBLEM CARD #1

- **Bài toán (1 câu):** Hệ thống điều vận Xanh SM gợi ý trạm sạc cho tài xế mà không xét đến ngưỡng pin nguy hiểm, khiến tài xế có nguy cơ hết pin giữa đường thay vì được điều xe sạc di động kịp thời.
- **Công ty thành viên:** [x] Xanh SM
- **Ai đang đau (Actor)?** Tài xế Xanh SM đang chạy chuyến với pin thấp; gián tiếp là dispatcher phải xử lý cuộc gọi khẩn cấp khi tài xế kẹt pin.
- **Workflow thủ công hiện tại (4 bước):**
  1. Tài xế thấy pin thấp, mở app/gọi tổng đài báo tình trạng.
  2. Dispatcher tra cứu thủ công trạm sạc gần nhất trên bản đồ.
  3. Dispatcher đọc thông tin khoảng cách, ước lượng bằng kinh nghiệm xem tài xế có đến kịp không.
  4. Dispatcher nhắn trạm sạc cho tài xế hoặc gọi điều xe cứu hộ nếu nhận ra quá xa.
- **Bước nào tốn thời gian/lỗi nhất?** Bước 3 (ước lượng thủ công) — ⏱ ước tính 3-5 phút/lượt, và dễ sai khi dispatcher không kiểm tra chính xác ngưỡng pin/khoảng cách trong lúc gấp.
- **AI có thể nhảy vào hỗ trợ ở bước nào?** Bước 2-3: AI đọc mức pin + tọa độ, tự động áp ngưỡng an toàn (pin < 5% → không đề xuất trạm > 5km, thay vào đó soạn sẵn đề xuất điều xe sạc di động) để dispatcher chỉ cần duyệt và gửi.
- **Đo thành công bằng gì (Metric có số)?** Giảm thời gian xử lý một ca báo pin thấp từ ~4 phút xuống dưới 1 phút; giảm số ca tài xế hết pin giữa đường về gần 0.
- **Quick Architecture:** [x] LLM (có ranh giới cứng dạng rule để chặn đề xuất sai khi pin nguy hiểm)

---

### QUICK PROBLEM CARD #2

- **Bài toán (1 câu):** Nhân viên CSKH Vinhomes mất nhiều thời gian đọc và soạn phản hồi thủ công cho từng đánh giá tiêu cực của cư dân trên app quản lý tòa nhà.
- **Công ty thành viên:** [x] Vinhomes
- **Ai đang đau (Actor)?** Nhân viên CSKH quản lý tòa nhà.
- **Workflow thủ công hiện tại (4 bước):**
  1. Đánh giá 1-2 sao xuất hiện trên hệ thống quản lý.
  2. Nhân viên đọc nội dung, xác định vấn đề (kỹ thuật, dịch vụ, an ninh...).
  3. Nhân viên tra cứu chính sách/quy trình xử lý tương ứng.
  4. Nhân viên soạn phản hồi, gửi và cập nhật trạng thái ticket.
- **Bước nào tốn thời gian/lỗi nhất?** Bước 3-4 — ⏱ ước tính 8-10 phút/đánh giá, dễ phản hồi không nhất quán giữa các nhân viên.
- **AI có thể nhảy vào hỗ trợ ở bước nào?** Bước 2-4: AI phân loại vấn đề, gợi ý phản hồi nháp bám theo chính sách, nhân viên chỉ cần kiểm tra và gửi.
- **Đo thành công bằng gì (Metric có số)?** Giảm thời gian soạn phản hồi từ 10 phút xuống dưới 2 phút; tăng tỷ lệ phản hồi trong 24h.
- **Quick Architecture:** [x] LLM

---

### QUICK PROBLEM CARD #3

- **Bài toán (1 câu):** Tổng đài viên Vinmec phải hỏi thủ công triệu chứng qua điện thoại để xếp đúng chuyên khoa, gây chờ lâu và xếp sai khoa vào giờ cao điểm.
- **Công ty thành viên:** [x] Vinmec
- **Ai đang đau (Actor)?** Tổng đài viên đặt lịch khám; gián tiếp là bệnh nhân chờ lâu hoặc bị xếp sai khoa.
- **Workflow thủ công hiện tại (4 bước):**
  1. Bệnh nhân gọi điện, mô tả triệu chứng.
  2. Tổng đài viên hỏi thêm để xác định chuyên khoa phù hợp.
  3. Tổng đài viên tra lịch trống của bác sĩ theo khoa đã chọn.
  4. Tổng đài viên xác nhận lịch hẹn với bệnh nhân.
- **Bước nào tốn thời gian/lỗi nhất?** Bước 2 — ⏱ ước tính 4-6 phút/cuộc gọi vào giờ cao điểm, và là bước dễ xếp sai khoa nhất do phụ thuộc kinh nghiệm cá nhân của tổng đài viên.
- **AI có thể nhảy vào hỗ trợ ở bước nào?** Bước 2: AI hỏi triệu chứng theo kịch bản chuẩn hóa và gợi ý chuyên khoa phù hợp nhất, tổng đài viên xác nhận trước khi đặt lịch.
- **Đo thành công bằng gì (Metric có số)?** Giảm thời gian trung bình mỗi cuộc gọi từ 6 phút xuống dưới 3 phút; giảm tỷ lệ xếp sai khoa cần chuyển lại lịch.
- **Quick Architecture:** [x] LLM (có fallback chuyển tổng đài viên nếu triệu chứng không rõ ràng hoặc khẩn cấp)