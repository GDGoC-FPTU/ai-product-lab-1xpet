# 📄 02-deep-dive-report.md — Báo cáo nhóm Deep-Dive & Evaluation

## 👥 Thông tin Nhóm & Thành viên
* **Tên nhóm:** Vin Smart Future - AI Core 1
* **Lớp / Khóa:** AI Product Lab 2026
* **Danh sách thành viên:**


---

# 🎯 1. Quyết định lựa chọn bài toán

Nhóm thống nhất chọn **Quick Card #1: Xanh SM (GSM) — Xử lý sự cố hết pin & hỗ trợ điều xe khẩn cấp thực địa** để làm bài toán trọng tâm cho bài báo cáo Deep-Dive.

### Lý do lựa chọn & Loại bỏ các bài toán khác:
* **Lý do chọn Card #1:** Bài toán tác động trực tiếp đến hiệu suất vận hành thời gian thực (real-time) của đội xe taxi điện Xanh SM. Thời gian chờ xử lý thủ công (15 phút/lượt) gây lãng phí nguồn lực điều phối và trực tiếp ảnh hưởng đến doanh thu cũng như sự an toàn của tài xế.
* **Loại bỏ Card #2 (Vinhomes CSKH):** Rủi ro sai lệch thông tin liên quan đến pháp lý/chi phí căn hộ tương đối cao, dữ liệu chưa được chuẩn hóa.
* **Loại bỏ Card #3 (Vinmec Hồ sơ):** Đòi hỏi chứng chỉ bảo mật y tế nghiêm ngặt (HIPAA/GDPR) và quy trình thẩm định lâm sàng phức tạp.

---

# 🏗️ 2. Problem Statement (6-field) — Vin Smart Future Standard

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Điều phối viên (Dispatcher) tại Trung tâm Điều vận Xanh SM. |
| **2. Current Workflow** | Khi tài xế báo sự cố hết pin, điều phối viên nhận cuộc gọi, tra định vị GPS xe trên hệ thống nội bộ, mở Dashboard trạm sạc VinFast tra cứu thủ công trụ trống, tự viết tin nhắn SMS hướng dẫn gửi qua App tài xế, và gọi cứu hộ nếu pin dưới 5%. Quy trình 5 bước thủ công mất ~15 phút/lượt. |
| **3. Bottleneck** | **Bước 3 & Bước 4 (mất 10 phút/lượt):** Tra cứu thủ công trạm sạc trống tương thích cổng sạc xe (VF5/VFe34/VF8) và soạn thảo tin nhắn Tiếng Việt chỉ đường chuẩn xác. |
| **4. Business Impact** | ~80 sự cố pin/ngày tại Hà Nội. Làm lãng phí 20 giờ làm việc/ngày của bộ phận điều hành, làm giảm 15% hiệu suất đón khách của đội xe Xanh SM. |
| **5. Success Metric** | 1. Giảm tổng thời gian xử lý sự cố từ 15 phút xuống dưới 3 phút (Efficiency).<br>2. Tỉ lệ chỉ dẫn đúng trạm sạc còn trụ trống đạt trên 98% (Quality). |
| **6. Operational Boundary** | **AI ĐƯỢC PHÉP:** Lấy dữ liệu vị trí GPS, tra cứu trạm sạc trống và tự động soạn thảo tin nhắn dự thảo (draft).<br>**CẤM AI:** Tuyệt đối KHÔNG được gửi tin nhắn trực tiếp cho tài xế mà chưa qua phê duyệt (Bắt buộc Human-in-the-loop); KHÔNG gợi ý trạm sạc > 5km khi pin < 5% (Bắt buộc kích hoạt xe sạc di động). |

---

# 🔄 3. Future-State Flow & AI Fit Matrix

### Phân tích AI-Fit Matrix:
* **Mức độ ứng dụng:** **LLM Feature** (có tích hợp Guardrails kiểm soát ranh giới).
* **Lý do:** Quy trình có luồng nghiệp vụ cố định, không cần Agent tự trị hoàn toàn để tránh rủi ro điều xe sai vị trí gây cạn pin giữa đường.

### Quy trình tương lai (Future-State Flow):

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Nhận cuộc    │ ──> │ 🔵 Auto-pull │ ──> │ 🔵 LLM Draft │ ──> │ 🟢 Dispatcher│
│ gọi sự cố    │     │ vị trí &     │     │ SMS & kiểm   │     │ Review & Send│
│              │     │ trạm sạc     │     │ tra ranh giới│     │ (HITL)       │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ↩️ Fallback:
                                                               Nếu AI draft lỗi,
                                                               Dispatcher tự viết
                                                               tay lại như cũ.
```

---

# 📊 4. Đánh giá độ sẵn sàng & Quyết định đầu tư (Evaluation)

### Checklist Đánh Giá:
- [x] Bài toán có tần suất lặp lại cao (80 cuốc/ngày).
- [x] Có metric đo lường bằng con số rõ ràng (15 min ──> 3 min).
- [x] Ranh giới an toàn được kiểm soát triệt để bằng Prompt Guardrails & HITL.
- [x] Kiến trúc LLM Feature đơn giản, chi phí API thấp (~0.002$/cuốc).

### 🏁 QUYẾT ĐỊNH CUỐI CÙNG: **GO (TIẾN HÀNH TRIỂN KHAI)**
**Luận điểm kỹ thuật & Vận hành:** Dự án mang lại ROI cao, giảm 80% thời gian xử lý sự cố, rủi ro được cô lập nhờ thẻ `[DRAFT_ONLY]` bắt buộc và quy định xử lý pin nguy cấp (< 5%).
