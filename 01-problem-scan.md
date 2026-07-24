# 📄 01-problem-scan.md — Báo cáo cá nhân Phase 1 & Phase 2

## 🏛️ Thông tin cá nhân
* **Họ và tên:** Bùi Duy Hải
* **Mã số sinh viên:** 2A202601878

---

# 🔍 Phase 1 — SCAN (Bảng quét cơ hội)

Dưới đây là danh sách 6 bài toán/bottleneck vận hành thực tế quét qua hoạt động của các công ty thành viên Vingroup thông qua 4 Lenses (Lặp lại, Tốn thời gian, AI-upgrade, Stakeholder Pain):

| # | Subsidiary | Lens | Mô tả ngắn bài toán & Bottleneck |
|---|------------|------|-----------------------------------|
| 1 | **Xanh SM (GSM)** | Tốn thời gian | Điều phối viên xử lý thủ công các báo cáo sự cố hết pin/cạn kiệt pin giữa đường của tài xế (mất 15 phút/lượt). |
| 2 | **VinFast** | Lặp lại | So khớp hóa đơn sạc điện và đối chiếu dữ liệu thanh toán trạm sạc đối tác hằng tuần. |
| 3 | **Vinhomes** | AI-upgrade | Phân loại và phản hồi tự động các ý kiến/khiếu nại của cư dân trên ứng dụng Vinhomes Resident (CSKH phản hồi rập khuôn, trễ SLA). |
| 4 | **Vinmec** | Pain từ người khác | Bác sĩ mất nhiều thời gian tổng hợp hồ sơ bệnh án và viết tóm tắt xuất viện (mất 20-30 phút/bệnh nhân). |
| 5 | **Xanh SM (GSM)** | Lặp lại | Phân tích lý do khách hàng hủy chuyến từ ghi âm cuộc gọi và ghi chú tài xế để phát hiện lỗi hệ thống. |
| 6 | **Vinpearl** | AI-upgrade | Hỗ trợ tư vấn và đặt lịch tự động các gói dịch vụ vui chơi giải trí VinWonders cho khách hàng quốc tế. |

---

# 🃏 Phase 2 — QUICK-ASSESS (3 Quick Problem Cards)

### 📌 QUICK PROBLEM CARD #1 (Bài toán được chọn cho Deep-Dive)

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán: Tài xế Xanh SM báo cáo sự cố pin cực kỳ khẩn cấp │
│ giữa đường cần chỉ dẫn trạm sạc gần nhất hoặc xe cứu hộ.    │
│ Công ty thành viên: [x] Xanh SM (GSM)                       │
│                                                             │
│ Ai đang đau (Actor)? Tài xế xe điện & Điều phối viên (Dispatch)│
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Tài xế gọi tổng đài điều vận báo sự cố pin             │
│   ──> 2. Tra cứu thủ công định vị GPS xe trên bản đồ         │
│   ──> 3. Tra cứu trạm sạc VinFast còn trụ trống phù hợp 🔴   │
│   ──> 4. Soạn thảo tin nhắn hướng dẫn đường đi gửi qua App 🔴│
│   ──> 5. Liên hệ xe sạc lưu động nếu pin cạn kiệt (< 5%)    │
│                                                             │
│ Bước tốn thời gian nhất? Bước 3 & 4 (⏱ 10 phút/lượt)        │
│ AI hỗ trợ ở bước nào? Bước 3 & 4 (Auto pull data + Draft SMS)│
│                                                             │
│ Metric đo thành công: Giảm thời gian xử lý sự cố từ 15 phút  │
│ ──> dưới 3 phút (Efficiency) & đạt 98% chính xác địa điểm. │
│                                                             │
│ Quick Architecture: [x] LLM Feature (Draft SMS + Guardrails)│
└─────────────────────────────────────────────────────────────┘
```

---

### 📌 QUICK PROBLEM CARD #2

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Phân loại và soạn phản hồi tự động cho phản ánh   │
│ khiếu nại của cư dân trên App Vinhomes Resident.           │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên Ban quản lý tòa nhà (CSKH)  │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Tiếp nhận phản ánh từ App cư dân                       │
│   ──> 2. Phân loại thủ công phòng ban (Kỹ thuật/Vệ sinh) 🔴  │
│   ──> 3. Kiểm tra quy trình & soạn văn bản trả lời 🔴        │
│   ──> 4. Gửi thông báo đến cư dân                           │
│                                                             │
│ Bước tốn thời gian nhất? Bước 2 & 3 (⏱ 30 phút/khiếu nại)   │
│ AI hỗ trợ ở bước nào? Phân loại tự động & Gợi ý phản hồi draft│
│                                                             │
│ Metric đo thành công: Rút ngắn thời gian phản hồi từ 12 giờ │
│ ──> dưới 30 phút.                                           │
│                                                             │
│ Quick Architecture: [x] LLM Feature (Routing + Draft Gen)   │
└─────────────────────────────────────────────────────────────┘
```

---

### 📌 QUICK PROBLEM CARD #3

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán: Tóm tắt hồ sơ bệnh án và tạo dự thảo giấy xuất    │
│ viện cho bệnh nhân tại Vinmec.                              │
│ Công ty thành viên: [x] Vinmec                              │
│                                                             │
│ Ai đang đau (Actor)? Bác sĩ điều trị & Y sĩ hành chính      │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Đọc lại lịch sử khám, xét nghiệm và đơn thuốc         │
│   ──> 2. Tổng hợp các chỉ số quan trọng vào mẫu xuất viện 🔴 │
│   ──> 3. Soạn thảo khuyến nghị chăm sóc sau xuất viện 🔴     │
│   ──> 4. Bác sĩ rà soát và ký duyệt                         │
│                                                             │
│ Bước tốn thời gian nhất? Bước 2 & 3 (⏱ 25 phút/bệnh nhân)   │
│ AI hỗ trợ ở bước nào? Trích xuất thông tin & Tóm tắt draft  │
│                                                             │
│ Metric đo thành công: Giảm thời gian làm hồ sơ từ 25 phút   │
│ ──> dưới 5 phút.                                            │
│                                                             │
│ Quick Architecture: [x] Agentic Loop (Medical Extraction)   │
└─────────────────────────────────────────────────────────────┘
```