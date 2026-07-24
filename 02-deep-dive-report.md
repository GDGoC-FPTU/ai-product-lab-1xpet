# Lab 02 — Deep-Dive Report

| Thông tin nhóm | Nội dung |
|---|---|
| **Tên nhóm** | `[ĐIỀN TÊN NHÓM]` |
| **Thành viên và MSSV** | `[ĐIỀN HỌ TÊN — MSSV CỦA TỪNG THÀNH VIÊN]` |
| **Bài toán được chọn** | Điều phối sự cố pin yếu thực địa cho tài xế Xanh SM |

> **Tính trung thực của bằng chứng:** Số liệu về thời gian, khối lượng và chi phí trong báo cáo là giả định có ghi nhãn để ra quyết định scoping. Trước production, nhóm phải đo baseline tối thiểu hai tuần từ log thật.

## 1. Quyết định lựa chọn

Nhóm chọn bài toán **“Hỗ trợ điều phối sự cố pin yếu thực địa”** vì nó có tác nhân rõ, luồng xử lý ngắn, tần suất đủ để tạo giá trị và có thể chặn rủi ro bằng rule cùng bước duyệt của con người. Nhóm không chọn agent tự trị: hệ thống không cần tự lập kế hoạch nhiều bước, và việc tự gửi chỉ dẫn sai có hậu quả vận hành đáng kể.

## 2. Current-State Workflow

Artifact chi tiết: [`04-workflow-diagram.png`](04-workflow-diagram.png).

| Bước | Actor / hệ thống | Input → Output | Thời gian giả định | Handoff / vấn đề |
|---:|---|---|---:|---|
| 1 | Tài xế / điện thoại | Mô tả sự cố → cuộc gọi | 2 phút | 🔄 Tài xế → điều phối viên; dữ liệu có thể thiếu. |
| 2 | Điều phối viên / ticket | Cuộc gọi → biển số, pin, vị trí, loại xe | 2 phút | Nhập lại thủ công. |
| 3 | Điều phối viên / bản đồ đội xe | Biển số → tọa độ xác nhận | 2 phút | 🔄 Ticket → bản đồ; chuyển màn hình. |
| 4 | Điều phối viên / dashboard trạm sạc | Tọa độ + loại xe → danh sách trạm phù hợp | 4 phút | 🔴 **Bottleneck:** so khớp khoảng cách, cổng và khả dụng. |
| 5 | Điều phối viên / công cụ nhắn tin | Dữ liệu thô → hướng dẫn hoặc yêu cầu cứu hộ | 4 phút | 🔴 **Bottleneck:** tổng hợp và soạn tay; 🔄 dashboard → app. |
| 6 | Điều phối viên → tài xế/đội cứu hộ | Phương án → tin nhắn/cuộc gọi | 1 phút | 🔄 Handoff cuối; chưa có kiểm tra bắt buộc thống nhất. |

**Tổng lead time giả định: 15 phút/lượt.** Bước 4–5 chiếm 8 phút và là phạm vi tối ưu chính.

## 3. Problem Statement 6-field

| Field | Nội dung |
|---|---|
| **1. Actor / Operator** | Điều phối viên Trung tâm Điều vận Xanh SM; người nhận phương án là tài xế, đội cứu hộ pin di động hoặc cả hai. |
| **2. Current Workflow** | Điều phối viên nhận cuộc gọi, ghi lại biển số/pin/vị trí, xác minh GPS, tra dashboard trạm, so khớp loại cổng và khoảng cách, soạn hướng dẫn hoặc gọi cứu hộ, rồi gửi tài xế. Quy trình đi qua ít nhất ba giao diện và mất khoảng 15 phút/lượt. |
| **3. Bottleneck** | Tra và so khớp trạm (4 phút) cùng việc tổng hợp, soạn hướng dẫn (4 phút). Sai dữ liệu pin/vị trí hoặc nhầm tương thích có thể dẫn đến phương án không an toàn. |
| **4. Business Impact** | Giả định pilot có **80 case/ngày**: current-state tiêu thụ khoảng **20 giờ điều phối/ngày** (80 × 15 phút). Nếu giảm còn 3 phút, giải phóng tối đa **16 giờ/ngày**, đồng thời rút thời gian xe ngừng khai thác. Đây là cơ hội lý thuyết, chưa quy đổi thành doanh thu trước khi có log thật. |
| **5. Success Metric** | Trong pilot 4 tuần: (a) P90 lead time từ lúc đủ 4 trường dữ liệu đến lúc điều phối viên duyệt **≤3 phút** so với baseline 15 phút; (b) **≥98%** phương án đúng trạm/cổng trên golden set; (c) **100%** output mang `[DRAFT_ONLY]` và có audit người duyệt; (d) **0** đề xuất trạm >5 km khi pin <5%; (e) tỷ lệ fallback **≤10%** sau tuần 2. |
| **6. Operational Boundary** | AI được đọc snapshot đã xác thực, tóm tắt và soạn bản nháp. Rule engine quyết định danh sách trạm hợp lệ. AI **không được** tự gửi, tự đặt chỗ, điều xe cứu hộ, sửa dữ liệu nguồn hoặc bỏ qua giới hạn an toàn. Pin <5% và không có trạm hợp lệ trong 5 km → chỉ tạo đề xuất `dispatch_mobile_charger`. Mọi hành động ra ngoài hệ thống cần điều phối viên duyệt. |

## 4. AI Fit: Rule, LLM hay Agent?

| Phương án | Dùng cho phần nào | Ưu điểm | Hạn chế | Quyết định |
|---|---|---|---|---|
| **No AI / thao tác tay** | Toàn quy trình | Dễ hiểu, không cần tích hợp | Chậm, không nhất quán, nhiều handoff | Giữ làm fallback. |
| **Rule / state machine** | Kiểm tra trường bắt buộc, pin <5%, bán kính 5 km, cổng tương thích, trạng thái trạm | Xác định, test được, chi phí thấp | Không phù hợp để hiểu ghi chú tự do và soạn hướng dẫn tự nhiên | **Bắt buộc dùng** cho safety gate. |
| **LLM Feature** | Chuẩn hóa mô tả sự cố, tóm tắt và tạo bản nháp tiếng Việt từ dữ liệu đã lọc | Xử lý ngôn ngữ linh hoạt, dễ giữ scope hẹp | Có thể bịa dữ kiện hoặc không tuân thủ format | **Chọn**, kèm schema validation và HITL. |
| **Agentic Loop** | Tự tra, chọn, nhắn, điều cứu hộ | Tự động hóa cao | Quyền quá rộng, khó kiểm soát lỗi và audit; không cần thiết cho luồng cố định | **Không chọn**. |

**Kết luận AI Fit:** Kiến trúc lai **deterministic rules + một LLM feature + Human-in-the-loop**. AI không thay thế logic khoảng cách, tương thích hay quyết định phát lệnh.

## 5. Future-State Flow

```text
[1] Nhận sự cố
      |
      v
[2] Auto-pull snapshot: xe, pin, GPS, loại cổng, trạm
      |
      v
[3] RULE GATE: đủ dữ liệu? dữ liệu còn mới? pin <5%? trạm ≤5 km?
      | lỗi/thiếu ------------------> [FALLBACK] màn hình tra cứu thủ công
      |
      +-- pin <5%, không có trạm hợp lệ --> tạo đề xuất dispatch_mobile_charger
      |
      v
[4] 🔵 LLM tạo [DRAFT_ONLY] theo JSON schema, chỉ dùng dữ liệu đã lọc
      |
      v
[5] Validator kiểm schema, dữ kiện, tag, khoảng cách và cổng
      | không đạt ------------------> [FALLBACK] template rule-based / soạn tay
      |
      v
[6] 🟢 Điều phối viên xem bằng chứng, sửa và DUYỆT hoặc TỪ CHỐI
      |
      v
[7] Hệ thống gửi sau khi duyệt, ghi audit log và nhận xác nhận
```

### Human-in-the-loop và quyền hạn

- Điều phối viên nhìn thấy pin, timestamp, tọa độ, loại cổng, khoảng cách và nguồn dữ liệu cạnh bản nháp.
- Nút gửi bị khóa cho đến khi có danh tính người duyệt; chỉnh sửa sau duyệt làm mất trạng thái duyệt.
- `dispatch_mobile_charger` chỉ là **đề xuất**. Điều phối viên xác nhận nguồn lực và phát lệnh qua hệ thống cứu hộ hiện hành.
- LLM không có credential ghi vào app tài xế, dashboard trạm hoặc hệ thống cứu hộ.

### Fallback

1. API lỗi, snapshot quá 60 giây hoặc thiếu trường → hiển thị cảnh báo và chuyển về current-state.
2. LLM timeout/JSON sai/vi phạm validator → dùng template rule-based với các trường đã xác thực; nếu vẫn thiếu, điều phối viên soạn tay.
3. Không có trạm phù hợp hoặc vị trí không chắc chắn → không suy đoán; gọi xác minh và/hoặc đề xuất cứu hộ.
4. Mọi fallback, sửa tay và quyết định duyệt/từ chối được log để đánh giá.

## 6. Dữ liệu, kiểm thử và bảo mật

- **Minimum input:** mã case, mã xe/loại cổng, phần trăm pin + timestamp, GPS + timestamp, danh sách trạm gồm khoảng cách/trạng thái/cổng.
- **Golden set:** tối thiểu 300 case đã khử định danh, bao phủ pin 0–4%, đúng 5%, GPS thiếu, trạm hết chỗ, cổng không tương thích, dữ liệu cũ và prompt injection trong ghi chú.
- **Kiểm thử:** unit test rule, contract test API, schema/factuality test LLM, adversarial test và shadow mode trước pilot.
- **Privacy:** chỉ gửi trường cần thiết; không đưa tên/số điện thoại tài xế vào prompt; mã hóa khi truyền/lưu; phân quyền và retention log theo chính sách nội bộ.
- **Monitoring:** lead time P50/P90, acceptance/edit/fallback rate, lỗi validator, safety violation, độ mới dữ liệu và drift theo tuần.

## 7. Phase 5 — EVALUATE

### AI Readiness Checklist

| Câu hỏi | Trạng thái | Bằng chứng / điều kiện |
|---|---|---|
| Có dữ liệu mẫu/log sạch để test? | ⚠️ **Một phần** | Có schema và nguồn dự kiến, nhưng repo không có log thật. Cần trích xuất, khử định danh và gán nhãn golden set trước pilot. |
| Rủi ro khi AI sai có kiểm soát được? | ✅ **Có, trong scope hẹp** | Safety gate bằng rule, output draft-only, schema validator, quyền read-only, HITL và fallback thủ công. |
| Stakeholder sẵn sàng đổi workflow? | ⚠️ **Chưa xác nhận** | Cần điều phối trưởng chấp thuận UI duyệt và phân công owner theo dõi metric. |

### Ước lượng chi phí pilot (4 tuần)

Các số dưới đây là **working assumptions**, không phải báo giá nhà cung cấp:

| Hạng mục | Giả định | Ước lượng |
|---|---|---:|
| Kỹ sư tích hợp | 2 người × 15 ngày công × 2,5 triệu VND | 75 triệu VND |
| Product/Ops/QA/Security | 20 ngày công tổng × 2 triệu VND | 40 triệu VND |
| Hạ tầng, logging, monitoring, model usage | Hạn mức pilot | 10 triệu VND |
| Dự phòng 20% | Trên 125 triệu VND | 25 triệu VND |
| **Tổng pilot** |  | **≈150 triệu VND** |

Giả định 80 case/ngày, 30 ngày pilot, mỗi case 4.000 token vào + ra → **9,6 triệu token**. Chi phí model phải thay bằng đơn giá nội bộ tại thời điểm phê duyệt; trong pilot, chi phí nhân sự/tích hợp nhiều khả năng lớn hơn chi phí suy luận. Không tuyên bố ROI tài chính cho đến khi đo được tần suất case, thời gian tiết kiệm thực và giá trị giờ xe.

### Quyết định cuối cùng: **NOT YET — chuẩn bị dữ liệu, sau đó pilot có điều kiện**

Không chọn GO production vì hai điều kiện readiness quan trọng chưa có bằng chứng: log/golden set sạch và cam kết workflow từ stakeholder. Cũng không NO-GO: phần ngôn ngữ có lợi thế hợp lý cho LLM, trong khi rủi ro đã được giới hạn bằng rule và HITL.

**Điều kiện chuyển sang GO cho pilot:**

1. Thu thập tối thiểu hai tuần baseline và ≥300 case đã khử định danh/gán nhãn.
2. Điều phối trưởng và Security ký duyệt workflow, retention và quyền truy cập.
3. Rule test đạt 100% cho invariant pin <5%/5 km và tương thích cổng.
4. Chạy shadow mode một tuần, không phát sinh hành động ngoài hệ thống.
5. Hội đồng pilot duyệt nếu đạt metric quality và không có safety violation nghiêm trọng.

Sau pilot 4 tuần, chỉ xem xét mở rộng nếu các success metric ở mục 3 đạt và không có sự cố an toàn; nếu không, giữ rule/template hoặc quay về current-state.
