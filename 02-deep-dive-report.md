# Lab 02 — Deep-Dive Report

| Thông tin nhóm | Nội dung |
|----------------|----------|
| **Tên nhóm** | `[ĐIỀN TÊN NHÓM]` |
| **Thành viên và MSSV** | `[ĐIỀN HỌ TÊN — MSSV CỦA TỪNG THÀNH VIÊN]` |
| **Bài toán được chọn** | Điều phối xử lý sự cố pin yếu cho tài xế Xanh SM |

> **Lưu ý về dữ liệu:** Tất cả số liệu liên quan đến thời gian xử lý, khối lượng công việc và chi phí trong tài liệu này đều là giả định phục vụ việc xác định phạm vi (scoping). Trước khi triển khai thực tế, cần thu thập tối thiểu hai tuần dữ liệu vận hành để hiệu chỉnh các giả định.

---

# 1. Lý do lựa chọn bài toán

Nhóm quyết định tập trung vào bài toán **"Điều phối hỗ trợ tài xế gặp sự cố pin yếu"** vì đây là quy trình có phạm vi rõ ràng, số lượng tác nhân tham gia ít và các bước xử lý tương đối cố định. Tần suất phát sinh đủ lớn để việc tối ưu đem lại giá trị cho vận hành, đồng thời các rủi ro có thể được kiểm soát bằng tập luật (rule engine) kết hợp với bước phê duyệt của điều phối viên.

Nhóm không lựa chọn mô hình agent tự trị vì quy trình này không yêu cầu lập kế hoạch nhiều bước. Nếu hệ thống tự động gửi chỉ dẫn hoặc phát lệnh sai, hậu quả có thể ảnh hưởng trực tiếp đến hoạt động điều phối.

---

# 2. Current-State Workflow

**Workflow chi tiết:** `04-workflow-diagram.png`

| Bước | Actor / Hệ thống | Input → Output | Thời gian (giả định) | Điểm bàn giao / vấn đề |
|------|------------------|----------------|----------------------|------------------------|
| 1 | Tài xế / Điện thoại | Báo sự cố → Cuộc gọi | 2 phút | 🔄 Thông tin ban đầu có thể thiếu hoặc chưa đầy đủ. |
| 2 | Điều phối viên / Ticket | Cuộc gọi → Biển số, mức pin, vị trí, loại xe | 2 phút | Phải nhập dữ liệu thủ công vào hệ thống. |
| 3 | Điều phối viên / Bản đồ đội xe | Biển số → Xác nhận vị trí GPS | 2 phút | 🔄 Chuyển đổi giữa ticket và bản đồ. |
| 4 | Điều phối viên / Dashboard trạm sạc | Vị trí + loại xe → Danh sách trạm khả dụng | 4 phút | 🔴 **Điểm nghẽn:** tìm trạm phù hợp theo khoảng cách, cổng sạc và trạng thái hoạt động. |
| 5 | Điều phối viên / Công cụ nhắn tin | Thông tin tổng hợp → Soạn hướng dẫn hoặc yêu cầu cứu hộ | 4 phút | 🔴 **Điểm nghẽn:** tổng hợp dữ liệu và soạn nội dung thủ công. |
| 6 | Điều phối viên → Tài xế hoặc đội cứu hộ | Phương án → Tin nhắn hoặc cuộc gọi | 1 phút | 🔄 Chưa có cơ chế bắt buộc kiểm tra trước khi gửi. |

**Lead time hiện tại (giả định): khoảng 15 phút/case.** Trong đó hai bước 4 và 5 chiếm hơn một nửa tổng thời gian nên được xem là trọng tâm để tối ưu.

---

# 3. Problem Statement

| Thành phần | Nội dung |
|------------|----------|
| **1. Actor / Operator** | Quy trình được vận hành bởi điều phối viên tại Trung tâm Điều vận Xanh SM. Kết quả xử lý sẽ được gửi đến tài xế, đội cứu hộ pin lưu động hoặc cả hai tùy từng tình huống. |
| **2. Current Workflow** | Sau khi nhận cuộc gọi, điều phối viên ghi nhận thông tin xe, mức pin và vị trí; tiếp theo xác minh GPS, tra cứu các trạm sạc, kiểm tra khả năng tương thích của cổng sạc, sau đó soạn hướng dẫn hoặc điều xe cứu hộ trước khi gửi cho tài xế. Quy trình phải thao tác trên nhiều hệ thống khác nhau và trung bình mất khoảng 15 phút cho mỗi trường hợp. |
| **3. Bottleneck** | Phần lớn thời gian bị tiêu tốn ở khâu tra cứu trạm phù hợp và biên soạn hướng dẫn. Ngoài ra, sai lệch về vị trí, mức pin hoặc loại cổng sạc có thể dẫn tới đề xuất không chính xác hoặc thiếu an toàn. |
| **4. Business Impact** | Với giả định pilot xử lý khoảng **80 trường hợp mỗi ngày**, tổng thời gian điều phối hiện nay tương đương gần **20 giờ làm việc/ngày**. Nếu rút ngắn còn khoảng 3 phút/case, có thể tiết kiệm khoảng **16 giờ điều phối/ngày**, đồng thời giảm thời gian phương tiện phải dừng khai thác. Đây mới là ước tính lý thuyết và cần được xác nhận bằng dữ liệu thực tế. |
| **5. Success Metric** | Mục tiêu của pilot trong 4 tuần gồm: (1) P90 lead time ≤3 phút kể từ khi đủ dữ liệu đầu vào; (2) ít nhất 98% đề xuất đúng trạm và đúng loại cổng trên golden set; (3) mọi phản hồi đều mang nhãn `[DRAFT_ONLY]` và có người phê duyệt; (4) không đề xuất trạm cách quá 5 km khi pin dưới 5%; (5) tỷ lệ fallback dưới 10% từ tuần thứ hai. |
| **6. Operational Boundary** | AI chỉ được sử dụng để đọc snapshot đã xác thực, tóm tắt dữ liệu và tạo bản nháp. Việc xác định trạm hợp lệ hoàn toàn do rule engine đảm nhiệm. Hệ thống AI không được phép gửi tin nhắn, đặt chỗ, điều xe cứu hộ hoặc thay đổi dữ liệu nguồn. Nếu pin dưới 5% và không tồn tại trạm phù hợp trong bán kính 5 km, hệ thống chỉ được đề xuất `dispatch_mobile_charger`. Mọi hành động cuối cùng đều phải được điều phối viên xác nhận. |

---

# 4. AI Fit

| Phương án | Vai trò | Ưu điểm | Hạn chế | Quyết định |
|-----------|---------|----------|----------|------------|
| **Thủ công** | Toàn bộ quy trình | Không cần tích hợp hệ thống | Chậm, dễ sai sót và phụ thuộc người vận hành | Giữ làm phương án dự phòng |
| **Rule Engine** | Kiểm tra dữ liệu bắt buộc, khoảng cách, mức pin và loại cổng | Kết quả ổn định, dễ kiểm thử, chi phí thấp | Không xử lý tốt dữ liệu ngôn ngữ tự nhiên | **Bắt buộc sử dụng** |
| **LLM Feature** | Chuẩn hóa mô tả sự cố, tạo bản tóm tắt và bản nháp | Linh hoạt trong xử lý ngôn ngữ | Có nguy cơ sinh thông tin không đúng hoặc sai định dạng | **Lựa chọn**, kết hợp validator và Human-in-the-loop |
| **Agentic System** | Tự tìm trạm, liên hệ tài xế và điều xe cứu hộ | Mức tự động hóa cao | Khó kiểm soát, khó audit và không phù hợp với quy trình cố định | **Không lựa chọn** |

**Kết luận:** Giải pháp phù hợp là kết hợp **rule engine + một tính năng LLM + Human-in-the-loop**. Rule engine chịu trách nhiệm toàn bộ logic nghiệp vụ và điều kiện an toàn; LLM chỉ hỗ trợ xử lý ngôn ngữ và tạo bản nháp. Quyền ra quyết định cuối cùng vẫn thuộc về điều phối viên.

---

# 5. Future-State Workflow

```text
[1] Tiếp nhận thông tin sự cố
      |
      v
[2] Tự động lấy snapshot:
    xe, pin, GPS, loại cổng, danh sách trạm
      |
      v
[3] RULE GATE
    - Đủ dữ liệu?
    - Dữ liệu còn mới?
    - Pin <5%?
    - Có trạm phù hợp trong 5 km?
      |
      |------ Không đạt ------------------> [FALLBACK] Quy trình thủ công
      |
      |------ Pin <5% và không có trạm --> Đề xuất dispatch_mobile_charger
      |
      v
[4] LLM tạo bản nháp [DRAFT_ONLY]
    theo JSON schema
      |
      v
[5] Validator kiểm tra:
    - Schema
    - Dữ kiện
    - Khoảng cách
    - Loại cổng
      |
      |------ Không đạt ------------------> Template rule-based / Soạn tay
      |
      v
[6] Điều phối viên xem xét,
    chỉnh sửa và PHÊ DUYỆT hoặc TỪ CHỐI
      |
      v
[7] Hệ thống gửi sau khi được duyệt,
    lưu audit log và ghi nhận phản hồi
```

## Human-in-the-loop

- Điều phối viên luôn nhìn thấy dữ liệu gốc (mức pin, GPS, thời gian cập nhật, loại cổng và khoảng cách) bên cạnh bản nháp AI tạo ra.
- Chỉ khi có người phê duyệt thì chức năng gửi mới được mở.
- Nếu nội dung bị chỉnh sửa sau khi duyệt, trạng thái phê duyệt sẽ bị hủy và phải xác nhận lại.
- `dispatch_mobile_charger` chỉ là một gợi ý; việc điều xe cứu hộ vẫn thực hiện qua hệ thống hiện hành.
- LLM chỉ có quyền đọc dữ liệu, không được ghi trực tiếp vào bất kỳ hệ thống vận hành nào.

## Fallback

1. Snapshot quá cũ, API lỗi hoặc thiếu dữ liệu → quay về quy trình hiện tại.
2. LLM timeout, sinh JSON không hợp lệ hoặc không vượt qua validator → dùng template rule-based; nếu cần sẽ soạn thủ công.
3. Không tìm thấy trạm phù hợp hoặc vị trí chưa đủ tin cậy → yêu cầu xác minh thêm thay vì tự suy diễn.
4. Tất cả trường hợp fallback và thao tác chỉnh sửa đều được ghi log để phục vụ đánh giá sau này.

---

# 6. Dữ liệu, kiểm thử và bảo mật

- **Dữ liệu đầu vào tối thiểu:** mã sự cố, mã xe, loại cổng sạc, mức pin và thời điểm ghi nhận, vị trí GPS cùng timestamp, danh sách trạm gồm khoảng cách, trạng thái và loại cổng.
- **Golden set:** tối thiểu 300 trường hợp đã ẩn danh, bao phủ các tình huống như pin dưới 5%, đúng ngưỡng 5%, GPS thiếu, trạm hết chỗ, cổng không tương thích, dữ liệu cũ và prompt injection.
- **Kiểm thử:** bao gồm unit test cho rule engine, contract test API, kiểm thử schema và factuality của LLM, adversarial test và chạy shadow mode trước pilot.
- **Bảo mật:** chỉ cung cấp cho LLM những trường dữ liệu cần thiết; không đưa thông tin cá nhân của tài xế vào prompt; mã hóa dữ liệu khi truyền và lưu; quản lý phân quyền và thời gian lưu log theo quy định.
- **Monitoring:** theo dõi P50/P90 lead time, tỷ lệ chấp nhận bản nháp, tỷ lệ chỉnh sửa, fallback rate, validator error, safety violation và độ mới của dữ liệu.

---

# 7. Phase 5 — Evaluation

## AI Readiness Checklist

| Tiêu chí | Trạng thái | Giải thích |
|----------|------------|------------|
| Có dữ liệu thật để đánh giá? | ⚠️ Một phần | Đã có thiết kế schema nhưng chưa có log thực tế. Cần xây dựng golden set trước pilot. |
| Có cơ chế giảm thiểu rủi ro khi AI sai? | ✅ Có | Rule engine, validator, Human-in-the-loop, quyền chỉ đọc và fallback giúp giới hạn rủi ro. |
| Stakeholder sẵn sàng thay đổi quy trình? | ⚠️ Chưa xác nhận | Cần thống nhất với điều phối trưởng về quy trình mới và các chỉ số đánh giá. |

---

## Ước lượng chi phí pilot (4 tuần)

Các con số dưới đây chỉ nhằm phục vụ việc lập kế hoạch.

| Hạng mục | Giả định | Chi phí |
|----------|----------|----------|
| Kỹ sư phát triển | 2 người × 15 ngày × 2,5 triệu VND | 75 triệu VND |
| Product, Operations, QA, Security | 20 ngày công × 2 triệu VND | 40 triệu VND |
| Hạ tầng, logging, monitoring và model usage | Ngân sách pilot | 10 triệu VND |
| Dự phòng (20%) | Trên tổng chi phí | 25 triệu VND |
| **Tổng cộng** | | **≈150 triệu VND** |

Giả sử hệ thống xử lý khoảng 80 trường hợp mỗi ngày trong 30 ngày, mỗi trường hợp sử dụng khoảng 4.000 token thì tổng lượng token vào khoảng **9,6 triệu token**. Chi phí sử dụng mô hình sẽ phụ thuộc vào đơn giá thực tế tại thời điểm triển khai và nhiều khả năng vẫn nhỏ hơn chi phí tích hợp hệ thống cũng như nhân sự. Vì chưa có dữ liệu vận hành thực, nhóm chưa đưa ra kết luận về ROI.

---

## Kết luận: **NOT YET**

Nhóm chưa đề xuất triển khai production ngay vì còn thiếu hai điều kiện quan trọng:

- Chưa có dữ liệu thực để đánh giá chất lượng.
- Chưa có xác nhận từ stakeholder về quy trình vận hành mới.

Tuy nhiên, đây cũng chưa phải là **NO-GO** vì bài toán có phạm vi phù hợp với LLM và phần lớn rủi ro đều có thể được giới hạn bằng rule engine cùng Human-in-the-loop.

### Điều kiện để chuyển sang pilot

1. Thu thập tối thiểu hai tuần log thực tế và xây dựng golden set với ít nhất 300 trường hợp.
2. Hoàn thành phê duyệt từ bộ phận vận hành và bảo mật.
3. Rule engine vượt qua toàn bộ bài kiểm thử liên quan đến mức pin, khoảng cách và loại cổng.
4. Chạy shadow mode trong một tuần mà không phát sinh hành động ngoài hệ thống.
5. Chỉ triển khai pilot khi các tiêu chí chất lượng và an toàn đều đạt yêu cầu.

Sau giai đoạn pilot kéo dài bốn tuần, hệ thống chỉ nên được mở rộng nếu các success metric đã đề ra đều được đáp ứng và không xuất hiện sự cố nghiêm trọng liên quan đến an toàn. Nếu không đạt, nhóm sẽ tiếp tục sử dụng quy trình dựa trên rule hoặc quay trở lại quy trình hiện tại.