# 01-problem-scan
## Phase 1 — SCAN

Dưới đây là 5 bài toán thực tế được quét qua các hoạt động vận hành của các công ty thành viên Vingroup thông qua 4 thấu kính Lenses: Lặp lại, Tốn thời gian, AI-upgrade, và Stakeholder Pain.

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|:---|:---|:---|:---|
| 1 | **Vinhomes** | Tốn thời gian | Thẩm định và đối chiếu tự động các điều khoản trong hợp đồng mua bán bất động sản với kho dữ liệu pháp luật hiện hành. |
| 2 | **VinFast** | AI-upgrade | Nhận diện lỗi ngoại quan trên thân vỏ xe điện tại xưởng kiểm định trong điều kiện ánh sáng yếu. |
| 3 | **Vinpearl** | Lặp lại | Trợ lý tự động phân luồng khiếu nại, tra cứu và cập nhật trạng thái đặt phòng trên hệ thống nội bộ. |
| 4 | **Xanh SM** | Stakeholder Pain | Tối ưu hóa hệ thống điều vận thông minh (Smart Dispatching) để giảm thiểu thời gian tài xế chạy xe không khách. |
| 5 | **Vinmec** | Tốn thời gian | Bóc tách và cấu trúc hóa dữ liệu từ hồ sơ bệnh án giấy hoặc ghi chú viết tay của bác sĩ vào hệ thống quản lý bệnh án điện tử. |

---

## Phase 2 — QUICK-ASSESS (3 Quick Problem Cards)

Dưới đây là 3 thẻ bài toán tiềm năng nhất được lựa chọn để phân tích sâu, tập trung vào tính khả thi của việc tích hợp các mô hình AI/ML.

### 🃏 QUICK PROBLEM CARD #1: Thẩm định Hợp đồng Pháp lý

```text
┌─────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1: Thẩm định Hợp đồng Pháp lý               │
│                                                                 │
│ Bài toán: Tự động hóa quá trình trích xuất và đối chiếu         │
│ điều khoản hợp đồng bất động sản với các quy định pháp luật     │
│ hiện hành để phát hiện rủi ro pháp lý.                          │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [x] Vinhomes      │
│                     [ ] Vinmec   [ ] Khác                       │
│                                                                 │
│ Ai đang đau (Actor)? Nhân viên Pháp chế / Thư ký dự án Vinhomes.│
│                                                                 │
│ Workflow thủ công hiện tại (4 bước):                            │
│   1. Nhận hợp đồng draft (Word/PDF) ──> 2. Đọc thủ công và      │
│   tra cứu luật ──> 3. Đối chiếu tính hợp lệ của điều khoản ──>  │
│   4. Ghi chú lỗi và trả file review.                            │
│                                                                 │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 & 3: Tra cứu và đối     │
│ chiếu luật (~45 - 60 phút/hợp đồng). Dễ xảy ra sai sót nếu      │
│ áp dụng sai văn bản luật đang có hiệu lực.                      │
│ AI có thể nhảy vào hỗ trợ ở bước nào? AI bóc tách điều khoản    │
│ hợp đồng và truy xuất dữ liệu luật.                             │
│                                                                 │
│ Đo thành công bằng gì (Metric có số)?                           │
│ Giảm thời gian rà soát từ 45 phút xuống dưới 5 phút/hợp đồng.   │
│ Tỷ lệ trích dẫn sai luật (Hallucination rate) dưới 1%.          │
│                                                                 │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent     │
│ (Cụ thể: RAG Pipeline với mô hình tối ưu cho tiếng Việt).       │
└─────────────────────────────────────────────────────────────────┘
```

---

### 🃏 QUICK PROBLEM CARD #2: Kiểm định ngoại quan xe điện trong điều kiện thiếu sáng

```text
┌─────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2: Kiểm định ngoại quan xe điện trong điều  │
│ kiện thiếu sáng                                                 │
│                                                                 │
│ Bài toán: Hệ thống thị giác máy tính nhận diện và đánh          │
│ dấu lỗi ngoại quan trên xe điện tại trạm kiểm tra trong xưởng   │
│ sản xuất.                                                       │
│ Công ty thành viên: [x] VinFast  [ ] Xanh SM  [ ] Vinhomes      │
│                     [ ] Vinmec   [ ] Khác                       │
│                                                                 │
│ Ai đang đau (Actor)? Nhân viên kiểm soát chất lượng (QA/QC      │
│ Inspector).                                                     │
│                                                                 │
│ Workflow thủ công hiện tại (4 bước):                            │
│   1. Xe vào trạm kiểm tra ──> 2. QC dùng đèn pin soi rà soát    │
│   toàn bộ xe ──> 3. Ghi chép lỗi vào biên bản ──> 4. Chuyển xe  │
│   sang khu vực khắc phục.                                       │
│                                                                 │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2: Soi rà soát thủ công   │
│ trong môi trường nhà xưởng (~5 phút/xe). Mắt người dễ bỏ        │
│ sót các lỗi xước dăm khó thấy.                                  │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Camera chụp ảnh xe; mô    │
│ hình học sâu phân tích hình ảnh để tự động khoanh vùng          │
│ (bounding box) các vết xước/lỗi ngay trên màn hình để nhân viên │
│ xác nhận.                                                       │
│                                                                 │
│ Đo thành công bằng gì (Metric có số)?                           │
│ Nâng độ chính xác nhận diện lỗi từ 85% lên 98% trong môi trường │
│ ánh sáng yếu; Giảm thời gian kẹt tại trạm QC xuống dưới 45      │
│ giây/xe.                                                        │
│                                                                 │
│ Quick Architecture: [ ] No AI  [x] Rule  [ ] LLM  [ ] Agent     │
└─────────────────────────────────────────────────────────────────┘
```

---

### 🃏 QUICK PROBLEM CARD #3: Trợ lý Agent xử lý Booking & Khiếu nại

```text
┌─────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                           │
│                                                                 │
│ Bài toán: Triển khai Agent tự động tiếp nhận yêu cầu,           │
│ tra cứu API hệ thống nội bộ để phản hồi và cập nhật trạng thái  │
│ đặt dịch vụ của khách hàng.                                     │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes      │
│                     [ ] Vinmec   [x] Khác: Vinpearl             │
│                                                                 │
│ Ai đang đau (Actor)? Nhân viên Chăm sóc khách hàng (CS Agent).  │
│                                                                 │
│ Workflow thủ công hiện tại (4 bước):                            │
│   1. Nhận yêu cầu/khiếu nại ──> 2. Chuyển tab sang hệ thống để  │
│   tra cứu mã booking ──> 3. Soạn phản hồi theo template ──>     │
│   4. Cập nhật trạng thái vé.                                    │
│                                                                 │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 & 3: Thao tác chuyển    │
│ đổi qua lại giữa các phần mềm và tra cứu dữ liệu thủ công       │
│ (~7 phút/ticket).                                               │
│ AI có thể nhảy vào hỗ trợ ở bước nào? AI Agent đọc hiểu luồng   │
│ tin nhắn, tự động trích xuất thông tin, gọi API tích hợp (qua   │
│ các nền tảng nội bộ) để truy vấn, sau đó thực thi cập nhật hoặc │
│ soạn sẵn câu trả lời.                                           │
│                                                                 │
│ Đo thành công bằng gì (Metric có số)?                           │
│ Giảm thời gian phản hồi lần đầu từ 2 giờ xuống dưới 2 phút; Tỷ  │
│ lệ giải quyết ngay trong lần đầu đạt trên 70%.                  │
│                                                                 │
│ Quick Architecture: [ ] No AI  [ ] Rule  [ ] LLM  [x] Agent     │
└─────────────────────────────────────────────────────────────────┘
```