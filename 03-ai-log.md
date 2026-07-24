# 03 — AI Log & Reflection
### AI Product Scoping — Vin Smart Future (Lab 02)

> Ghi chú: Nhật ký này phản ánh đúng những gì đã xảy ra trong phiên làm việc với Claude
> cho hai deliverable `prompt_prototype.py` và `01-problem-scan.md`. Vì đây là bài phản
> ánh cá nhân, mình đã tự đọc lại và chỉnh sửa để đúng với trải nghiệm thật của mình
> trước khi nộp — không copy nguyên văn.

---

## 1. AI giúp gì (Thought-partner)

- **Viết ranh giới vận hành (SYSTEM_PROMPT):** Từ 2 rule cứng cho trước (tag `[DRAFT_ONLY]`
  và ngưỡng pin < 5%), mình nhờ AI diễn giải thành một system prompt đầy đủ: định nghĩa vai
  trò dispatcher co-pilot, mô tả rõ hai rule không được vi phạm dù người dùng có ép buộc thế
  nào, và quy định format output.
- **Viết code gọi API:** AI hoàn thiện hàm `evaluate_prompt()` gọi Gemini 2.5 qua SDK
  `google-genai`, kèm fallback sang SDK cũ `google-generativeai` nếu package mới chưa cài —
  điều này mình không tự nhớ hết cú pháp hai SDK khác nhau nên AI hỗ trợ khá nhiều.
- **Brainstorm 5 bài toán theo 4 lenses:** AI giúp mình rà nhanh qua các mảng VinFast, Xanh
  SM, Vinhomes, Vinmec, Vinpearl và gợi ý bài toán cụ thể cho từng lens, mình chọn lọc lại
  và chỉnh cho sát với ví dụ dispatcher đã học ở Phase 0.
- **Điền Quick Problem Card:** AI giúp cấu trúc hóa workflow 4 bước, ước lượng thời gian xử
  lý mỗi bước, và viết metric có số cụ thể theo đúng format yêu cầu của worksheet.

## 2. AI sai gì / hạn chế cần lưu ý

- **Chưa test thật với API:** Điểm yếu lớn nhất là mình không có `GEMINI_API_KEY` trong lúc
  làm lab, nên AI chỉ viết code và không thể tự chạy thật 2 adversarial test case để xác nhận
  model Gemini có thực sự tuân thủ 2 rule hay không. AI chỉ kiểm tra được là code
  *compile được* (`python3 -m py_compile`), chứ không chứng minh được hành vi runtime đúng
  như kỳ vọng. Đây là một "lỗ hổng chứng cứ" mình cần tự bổ sung trước khi báo cáo I2 là
  "chạy thử nghiệm thành công".
- **Rule 2 ban đầu hơi mơ hồ:** Đề bài viết "nếu pin < 5%, không đề xuất trạm > 5km, thay
  vào đó trigger mobile charger" — nhưng không nói rõ nếu trạm gần nhất **trong** 5km thì có
  cần trigger mobile charger nữa không. AI đã tự chọn một cách diễn giải (vẫn có thể gợi ý
  trạm trong 5km, nhưng vẫn kèm block JSON dispatch nếu còn nghi ngờ khả năng tới nơi an
  toàn) mà không hỏi lại mình trước — mình phải tự đọc kỹ để phát hiện đây là một giả định,
  không phải rule gốc.

## 3. Mình đã sửa đổi ra sao

- Đọc lại toàn bộ SYSTEM_PROMPT và đối chiếu từng câu với 2 rule gốc trong file starter code
  để chắc chắn AI không tự thêm ngoại lệ nào không có trong đề bài.
- Tự chạy `python3 -m py_compile` để xác nhận code không lỗi cú pháp trước khi coi đây là
  bản nộp, thay vì tin tưởng hoàn toàn vào khẳng định của AI.
- Ghi rõ trong log này (thay vì giấu đi) rằng phần I2 "chạy thử nghiệm thành công" mới chỉ
  dừng ở mức code chạy được về mặt kỹ thuật — mình sẽ cần chạy lại với `GEMINI_API_KEY` thật
  trước buổi trình bày nhóm để có kết quả pass/fail thực sự cho 2 test case, thay vì chỉ suy
  diễn rằng AI sẽ tuân thủ đúng.