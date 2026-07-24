# 02-deep-dive-report

**THÔNG TIN NHÓM**
*   **Tên nhóm:** 1xPET
*   **Thành viên: ...**

---

# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Quyết định lựa chọn của nhóm
Nhóm thống nhất chọn bài toán: **Thẩm định Hợp đồng Pháp lý (Vinhomes)** (Card #1).

**Lý do lựa chọn và loại bỏ các thẻ khác:**
*   **Lý do chọn Card #1:** Bài toán này tác động trực tiếp đến rủi ro pháp lý và tài chính cốt lõi của Vingroup. Các bài toán CSKH thông thường có thể chờ, nhưng một điều khoản hợp đồng sai luật có thể dẫn đến tranh chấp lớn. Hơn nữa, với kiến trúc RAG hiện tại kết hợp các cơ chế suy luận logic nghiêm ngặt, bài toán này hoàn toàn khả thi về mặt kỹ thuật.
*   **Loại bỏ Card #2 (VinFast - Lỗi ngoại quan):** Triển khai hệ thống thị giác máy tính tại xưởng VinFast không chỉ là bài toán thuật toán mà còn yêu cầu đầu tư lớn về thiết bị ngoại vi (hệ thống camera công nghiệp độ phân giải cao, máy chủ Edge AI đặt tại xưởng). So với giải pháp LLM xử lý hợp đồng của Vinhomes chỉ cần tích hợp phần mềm, dự án VinFast đòi hỏi vốn đầu tư ban đầu lớn và thời gian setup vật lý kéo dài, không phù hợp để làm nguyên mẫu kiểm chứng nhanh.
*   **Loại bỏ Card #3 (Vinpearl - Agent):** Việc xây dựng một Agent hoàn chỉnh có khả năng gọi API nội bộ đòi hỏi môi trường thử nghiệm phức tạp và tiềm ẩn rủi ro lộ lọt dữ liệu khách hàng nếu thiết lập ranh giới không cẩn thận trong thời gian ngắn.

## 3.2. Problem Statement (6-field) — Vin Smart Future Standard

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Chuyên viên Pháp chế / Thư ký dự án Vinhomes. |
| **2. Current Workflow** | Nhận bản nháp hợp đồng (Word/PDF) ──> Đọc thủ công ──> Tra cứu các quy định pháp luật trên hệ thống thư viện luật ──> Đối chiếu tính hợp lệ của từng điều khoản ──> Ghi chú các điểm vi phạm ──> Gửi lại file đã review. Mất 45 - 60 phút/hợp đồng. |
| **3. Bottleneck** | Bước tra cứu, đối chiếu chéo (cross-reference) và áp dụng luật (chiếm ~40 phút). Rủi ro rất cao ở việc áp dụng sai văn bản pháp luật đã hết hiệu lực hoặc hiểu sai tiền đề pháp lý. |
| **4. Business Impact** | Chậm tiến độ ký kết hợp đồng dự án (SLA giảm). Rủi ro pháp lý/đền bù lớn nếu hợp đồng được phát hành có chứa các điều khoản trái với quy định hiện hành của Nhà nước. |
| **5. Success Metric** | 1. Tốc độ: Giảm thời gian rà soát ban đầu từ 60 phút xuống dưới 10 phút/hợp đồng.<br>2. Độ chính xác: Đạt tỷ lệ trích dẫn luật định (hallucination rate) dưới 1%, khớp với các tiêu chuẩn benchmark kiểm thử công khai. |
| **6. Operational Boundary** | Quy tắc an toàn: (1) Cách ly tuyệt đối luồng truy xuất "Căn cứ pháp lý gốc" khỏi luồng "Suy luận logic" để hệ thống đạt chuẩn kiểm thử pháp lý (Syllogism benchmarks). (2) Cấm hệ thống tự ý đánh giá tính hợp lệ nếu không bóc tách được nguyên văn điều luật tương ứng. (3) 100% kết quả đầu ra chỉ được xem là bản nháp và bắt buộc phải có chuyên gia pháp chế rà soát, phê duyệt (HITL). |

## 3.3. Future-State Flow & AI Fit

*   **Đánh giá AI Fit:** Giải pháp thuộc nhóm LLM Feature + RAG kết hợp xử lý suy luận đa bước (Multi-hop Reasoning). Không cần tác nhân tự trị (Agent) vì quy trình mang tính phân tích tĩnh.
*   **Future-State Flow:**

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Upload file  │     │ 🔵 RAG Pull &│     │ 🔵 AI Logic  │     │ 🟢 Legal     │
│ Hợp đồng nháp│ ──→ │ Trích xuất   │ ──→ │ Đối chiếu    │ ──→ │ Expert     │
│ lên hệ thống │     │ Luật gốc     │     │ (Reasoning)  │     │ Review & Duyệt│
│ (Pháp chế)   │     │ (Hệ thống AI)│     │ (Hệ thống AI)│     │ (Pháp chế)   │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ↩️ Fallback:
                                                               Nếu hệ thống RAG không
                                                               tìm thấy văn bản luật
                                                               khớp, AI dừng suy luận
                                                               và trả cảnh báo để xử
                                                               lý thủ công.
```

---

# 🏁 EVALUATE

### AI Readiness Checklist:
1.  [x] **Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test?** (Có, sử dụng các kho dữ liệu luật tiếng Việt tiêu chuẩn và hợp đồng mẫu của Vinhomes).
2.  [x] **Rủi ro khi AI sai có nằm trong tầm kiểm soát?** (Có, cơ chế tách biệt luồng truy xuất "Căn cứ pháp lý gốc" khỏi luồng "Suy luận logic" ngăn chặn sinh luật giả, và luôn có bước Human-in-the-loop cuối cùng).
3.  [x] **Stakeholders sẵn sàng thay đổi quy trình làm việc cũ?** (Có, chuyên viên pháp chế được giảm tải công việc lặp lại).

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future:
[x] **GO (Bắt đầu xây dựng Prototype):** Bắt đầu phát triển với scope hẹp trên một loại hợp đồng mua bán cụ thể.
[ ] **NOT YET (Cần tích lũy thêm dữ liệu/xác lập baseline):** Trì hoãn để chuẩn bị thêm.
[ ] **NO-GO (Không khả thi / Rule-based tốt hơn):** Hủy bỏ dự án AI này.