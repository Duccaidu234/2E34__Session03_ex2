# BÁO CÁO PHÂN TÍCH & SO SÁNH TRADE-OFF GIẢI PHÁP PROMPTING

**Chủ đề:** Ứng dụng AI trong Học tập & Nghiên cứu — Tối ưu hóa quy trình *Feynman Prompt gia sư* và *Rubric chấm bài tự động*.

---

## BƯỚC 1: ĐỀ XUẤT 2 PROMPT MẪU

Để giải quyết bài toán tự động hóa gia sư theo phương pháp Feynman và chấm điểm theo Rubric, dưới đây là 2 phương án thiết kế Prompt đại diện cho hai trường phái tiếp cận khác nhau: **Zero-shot ngắn gọn** và **Structured Prompting có ràng buộc biên**.

### Giải pháp 1: Prompt đơn giản, ngắn gọn (Zero-shot / Minimalist Prompt)

Phương án này tập trung vào sự súc tích, chỉ định vai trò và nhiệm vụ cốt lõi mà không thiết lập các quy tắc kiểm soát lỗi biên hay định dạng đầu ra khắt khe.

```text
Bạn là một gia sư sử dụng phương pháp Feynman kiêm giám khảo chấm bài. 
Hãy đọc chủ đề và phần giải thích của học viên dưới đây:
- Chủ đề: {{topic_concept}}
- Phần giải thích của học viên: {{learner_explanation}}
- Tài liệu tham khảo (nếu có): {{reference_material}}

Yêu cầu:
1. Giải thích lại những chỗ học viên hiểu sai hoặc diễn đạt khó hiểu theo cách đơn giản, dễ hiểu nhất (tinh thần Feynman).
2. Chấm điểm bài làm theo thang điểm 10 dựa trên các tiêu chí: Hiểu bản chất, Diễn đạt đơn giản, Ví dụ minh họa, và Tính chính xác. Đưa ra nhận xét ngắn gọn và gợi ý cách cải thiện.
```

### Giải pháp 2: Prompt chi tiết, có nhiều ràng buộc & kiểm soát lỗi (Structured & Guardrailed Prompt)

Phương án này phân tách rõ ràng vai trò, biến đầu vào, cơ chế bẫy dữ liệu (Edge Cases), Rubric mặc định khi thiếu dữ liệu và ép khuôn cấu trúc đầu ra (Output Schema).

```text
# VAI TRÒ (ROLE)
Bạn là Gia sư phương pháp Feynman kiêm Chuyên gia Khảo thí. Nhiệm vụ của bạn là kiểm tra dữ liệu đầu vào, phản hồi lỗ hổng tư duy bằng ngôn ngữ đời thường và chấm bài minh bạch theo Rubric.

# DỮ LIỆU ĐẦU VÀO (INPUT)
- Chủ đề (`topic_concept`): {{topic_concept}}
- Bài làm người học (`learner_explanation`): {{learner_explanation}}
- Tài liệu chuẩn (`reference_material`): {{reference_material}}
- Tiêu chí Rubric (`rubric_criteria`): {{rubric_criteria}}

# QUY TẮC CHẶN LỖI BIÊN & BẪY DỮ LIỆU (GUARDRAILS)
Kiểm tra tuần tự trước khi chấm:
1. Dữ liệu trống / rác: Nếu `topic_concept` trống -> Dừng và báo `[LỖI: THIẾU CHỦ ĐỀ]`. Nếu `learner_explanation` trống, vô nghĩa hoặc dưới 5 từ -> Gắn cờ `[CẢNH BÁO: DỮ LIỆU TRỐNG]`, chấm 0 điểm, tuyệt đối không tự bịa nội dung để chấm hộ.
2. Lạc đề: Nếu bài làm không liên quan chủ đề -> Gắn cờ `[CẢNH BÁO: LẠC ĐỀ]`, trừ tối đa điểm chính xác và yêu cầu làm lại.
3. Sai lệch kiến thức: Nếu mâu thuẫn với `reference_material` hoặc kiến thức chuẩn -> Gắn cờ `[CẢNH BÁO: SAI LỆCH KIẾN THỨC]`, chỉ rõ câu sai.
4. Thiếu biến tùy chọn: Nếu `rubric_criteria` trống -> Dùng Rubric mặc định 4 tiêu chí (Hiểu bản chất: 3đ, Đơn giản kiểu Feynman: 3đ, Ví dụ thực tế: 2đ, Logic/Chính xác: 2đ). Nếu `reference_material` trống -> Đối chiếu kiến thức chuẩn và ghi chú rõ.

# CẤU TRÚC ĐẦU RA BẮT BUỘC (OUTPUT FORMAT)
Trình bày đúng 3 phần:
1. **Trạng thái & Cảnh báo dữ liệu:** Liệt kê cờ cảnh báo và ghi chú mặc định.
2. **Phản hồi Gia sư Feynman:** Tóm tắt cách hiểu hiện tại -> Chỉ ra lỗ hổng bằng ví dụ đời thường -> Đặt 1 câu hỏi gợi mở.
3. **Bảng điểm Rubric & Hành động tiếp theo:** Bảng Markdown (`Tiêu chí | Điểm tối đa | Điểm đạt | Lý do`), Tổng điểm, và 2 bước sửa bài cụ thể.
```

---

## BƯỚC 2: LẬP BẢNG SO SÁNH TRADE-OFF

Bảng dưới đây đánh giá sự đánh đổi (Trade-off) giữa hai giải pháp dựa trên các tiêu chí vận hành thực tế trong môi trường học tập và nghiên cứu:

| Tiêu chí đánh giá | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) | Phân tích sự đánh đổi (Trade-off) |
| :--- | :--- | :--- | :--- |
| **Độ dễ hiểu (Dễ viết & bảo trì)** | **Cao** | **Trung bình** | Giải pháp 1 đọc vào hiểu ngay, ai cũng có thể viết trong 1 phút. Giải pháp 2 dài hơn, đòi hỏi tư duy cấu trúc và logic điều kiện rẽ nhánh (`If-Else`). |
| **Tốn token (Chi phí & Tốc độ)** | **Ít** (~120 tokens input) | **Nhiều hơn** (~380 tokens input) | Giải pháp 2 tiêu tốn lượng token đầu vào gấp ~3 lần Giải pháp 1, làm tăng nhẹ chi phí API và độ trễ phản hồi (latency) ở mỗi lượt gọi. |
| **Chặn lỗi biên (Edge Cases)** | **Trung bình (Kém ổn định)** | **Cực tốt** | Khi học viên nộp bài trống, lạc đề hoặc cố tình viết lan man, Giải pháp 1 dễ bị "ảo giác" (tự khen hoặc tự bịa ý để chấm điểm). Giải pháp 2 bắt chính xác và gắn cờ cảnh báo. |
| **Tính nhất quán của Output** | **Thấp – Trung bình** | **Rất cao** | Giải pháp 1 trả về văn bản tự do, mỗi lần chạy một kiểu trình bày khác nhau, khó tích hợp vào hệ thống lưu trữ. Giải pháp 2 ép khuôn bảng biểu chuẩn xác. |
| **Tính minh bạch khi chấm Rubric** | **Trung bình** | **Cao** | Giải pháp 1 thường cho điểm cảm tính, thiếu trích dẫn minh chứng. Giải pháp 2 bắt buộc đối chiếu từng tiêu chí, giải thích rõ lý do trừ/cộng điểm. |

---

## BƯỚC 3: LỰA CHỌN GIẢI PHÁP TỐI ƯU VÀ CHẠY THỬ

### 3.1. Lựa chọn phương án & Lý do

* **Phương án được chọn:** **Giải pháp 2 (Prompt chi tiết, có nhiều ràng buộc & kiểm soát lỗi)**.
* **Lý do lựa chọn (Tư duy phản biện):**
  1. **Đặc thù của bài toán Giáo dục & Khảo thí:** Trong học tập và nghiên cứu, việc chấm sai điểm hoặc gia sư giảng sai bản chất gây hậu quả nghiêm trọng hơn nhiều so với việc tốn thêm vài trăm token. Sự chính xác và công bằng của Rubric là ưu tiên số 1.
  2. **Khả năng chống chịu dữ liệu thực tế:** Người học thực tế thường xuyên nộp thiếu ý, lạc đề, dùng từ sáo rỗng hoặc bỏ trống ô trả lời. Giải pháp 1 tuy tiết kiệm token nhưng thất bại hoàn toàn ở khâu kiểm soát ngoại lệ (Edge Cases).
  3. **Tối ưu chi phí dài hạn:** Dù Giải pháp 2 tốn thêm ~260 input tokens mỗi lượt (chi phí gần như không đáng kể với các mô hình hiện nay), nhưng nó giúp **giảm chi phí vận hành con người** vì giáo viên không phải ngồi rà soát lại các bài bị AI chấm nhầm hoặc định dạng lộn xộn.

### 3.2. Kết quả chạy thử với Prompt được chọn (Giải pháp 2)

Để kiểm chứng hiệu quả thực tế của **Giải pháp 2**, hệ thống được chạy thử với một tình huống đầu vào có chứa **lỗi sai lệch kiến thức** kết hợp với **thiếu biến tùy chọn** (`rubric_criteria` và `reference_material` để trống):

**Dữ liệu đầu vào (Test Input):**
* `topic_concept`: `Overfitting (Quá khớp) trong Machine Learning`
* `learner_explanation`: `Overfitting là khi mô hình AI học quá kém nên đoán sai hết cả dữ liệu lúc học lẫn dữ liệu mới. Cách khắc phục là cho mô hình học thêm thật nhiều vòng (epochs) nữa.`
* `reference_material`: *(để trống)*
* `rubric_criteria`: *(để trống)*

**Kết quả phản hồi thực tế từ AI:**

> ### 1. Trạng thái & Cảnh báo dữ liệu
> * **Trạng thái:** Có cảnh báo chuyên môn.
> * **Các cờ đã kích hoạt:** `[CẢNH BÁO: SAI LỆCH KIẾN THỨC]`
> * **Ghi chú hệ thống:** Không có `reference_material` và `rubric_criteria` đính kèm. Hệ thống tự động áp dụng **Rubric mặc định 4 tiêu chí (thang 10)** và đối chiếu theo chuẩn kiến thức nền tảng về Học máy (Machine Learning).
>
> ### 2. Phản hồi Gia sư Feynman
> * **Cách bạn đang hiểu:** Bạn cho rằng Overfitting là việc mô hình học kém trên mọi dữ liệu và cần tăng số vòng học (epochs) để sửa.
> * **Chỉ ra lỗ hổng tư duy:** Bạn đang bị nhầm lẫn ngược giữa *Overfitting* và *Underfitting*! 
>   * Hãy tưởng tượng một học sinh **học vẹt**: bạn ấy học thuộc lòng từng chữ trong đề cương ôn tập nên làm bài kiểm tra thử được 10 điểm tuyệt đối (dữ liệu lúc học). Nhưng khi đi thi thật gặp câu hỏi đổi số liệu một chút, bạn ấy lại làm sai hết (dữ liệu mới). Đó chính là **Overfitting** — mô hình học "quá kỹ" cả chi tiết nhiễu của tập huấn luyện nên mất khả năng tổng quát hóa.
>   * Nếu bạn tiếp tục bắt học sinh đó "học vẹt thêm nhiều vòng nữa" (tăng epochs), tình trạng học vẹt sẽ càng nặng hơn chứ không hề giảm đi!
> * **Câu hỏi gợi mở:** Nếu một học sinh đang bị "học vẹt" đề cương, theo bạn chúng ta nên làm gì để bạn ấy hiểu bản chất thay vì bắt học thuộc lại đề cương đó thêm 10 lần?
>
> ### 3. Bảng điểm Rubric & Hành động tiếp theo
>
> | Tiêu chí | Điểm tối đa | Điểm đạt | Lý do & Minh chứng từ bài làm |
> | :--- | :---: | :---: | :--- |
> | **Hiểu đúng bản chất cốt lõi** | 3.0 | 0.5 | Hiểu sai bản chất: nhầm việc đoán sai trên dữ liệu huấn luyện (Underfitting) thành Overfitting. |
> | **Diễn đạt đơn giản kiểu Feynman** | 3.0 | 2.0 | Câu văn ngắn gọn, dễ đọc, không lạm dụng thuật ngữ khó, nhưng nội dung truyền tải chưa đúng. |
> | **Ví dụ minh họa thực tế** | 2.0 | 0.0 | Bài làm chưa đưa ra ví dụ minh họa trực quan nào. |
> | **Tính logic & Chính xác** | 2.0 | 0.0 | Đề xuất giải pháp "tăng thêm epochs" mâu thuẫn hoàn toàn với cách xử lý Overfitting (làm lỗi nặng hơn). |
>
> **Tổng điểm: 2.5 / 10**
>
> **2 bước sửa bài cụ thể:**
> 1. **Phân tách lại hiệu suất trên 2 tập dữ liệu:** Viết lại định nghĩa trong 2 câu, nêu rõ sự đối lập giữa kết quả trên *tập huấn luyện (rất tốt)* và *tập kiểm thử mới (kém)*.
> 2. **Bổ sung ví dụ "học vẹt" & cách sửa đúng:** Thay giải pháp "tăng epochs" bằng một hướng xử lý thực tế (ví dụ: dừng học sớm - Early Stopping, đơn giản hóa mô hình, hoặc bổ sung dữ liệu đa dạng hơn).