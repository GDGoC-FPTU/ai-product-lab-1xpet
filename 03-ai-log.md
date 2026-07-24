# 03-ai-log -- Nhật ký sử dụng AI (AI Usage Log)

## 1. AI đã giúp gì?
Trong quá trình làm bài, tôi đã sử dụng AI đóng vai trò như một trợ lý tư duy và lập trình viên hỗ trợ trong các tác vụ sau:
*   **Brainstorming kiến trúc:** Thảo luận để chuyển hướng thiết kế từ Multi-Agent (có nguy cơ rủi ro cao trong ngành pháp chế) sang kiến trúc multi-step RAG tĩnh, có kiểm soát ranh giới an toàn chặt chẽ (Human-in-the-loop).
*   **Viết System Prompt:** Hỗ trợ phác thảo các dòng code Python cho file `prompt_prototype.py`, thiết lập cấu trúc cơ bản để gọi API.

## 2. AI đã sai gì?
Điểm yếu lớn nhất của AI khi xử lý văn bản pháp lý là xu hướng "ảo giác" (hallucination) và sự thiếu chính xác trong việc trích xuất văn bản quy phạm pháp luật. Cụ thể:
*   Khi được yêu cầu xây dựng luồng suy luận để rà soát hợp đồng, AI ban đầu có xu hướng tự ý tóm tắt, diễn giải hoặc tự chế lại các điều khoản luật thay vì giữ nguyên văn bản gốc.
*   Mô hình thường gặp lỗi trộn lẫn giữa phần căn cứ pháp lý và phần lập luận đánh giá, khiến cho ranh giới giữa việc trích xuất luật và việc đưa ra nhận định bị mờ nhạt. Điều này gây rủi ro lớn trong ngành luật, vì bất kỳ sự thay đổi nhỏ nào về câu chữ trong văn bản luật cũng dẫn đến sai lệch pháp lý nghiêm trọng.

## 3. Sửa đổi và kiểm soát ra sao?
Để ép AI hoạt động chuẩn xác và kiểm soát triệt để rủi ro ảo giác pháp lý, tôi đã chủ động can thiệp và điều chỉnh trực tiếp logic trong quá trình thiết kế Prompt:
*   Chuẩn hóa văn bản trích xuất: Yêu cầu mô hình phải giữ nguyên văn bản quy phạm pháp luật gốc, tuyệt đối cấm tự ý tóm tắt, diễn giải hoặc biến đổi câu chữ của điều luật. 
*   Chia tách rõ ràng hai luồng độc lập: Thiết kế cấu trúc lệnh tách biệt hoàn toàn giữa luồng trích xuất "Căn cứ pháp lý" (chỉ thực hiện tìm kiếm và trích xuất đúng văn bản luật, không đưa ra nhận định) và luồng "Suy luận" (chỉ được phép đối chiếu hợp đồng dựa trên văn bản luật đã được trích xuất).
*   Thiết lập cơ chế Fallback an toàn: Bổ sung quy tắc cứng vào System Prompt: Trường hợp hệ thống không tìm thấy căn cứ pháp lý phù hợp trong cơ sở dữ liệu (No Citation Found), mô hình phải lập tức dừng toàn bộ quá trình suy luận, trả về cảnh báo và chuyển giao ngay (Handoff) cho chuyên viên pháp chế xử lý thủ công.