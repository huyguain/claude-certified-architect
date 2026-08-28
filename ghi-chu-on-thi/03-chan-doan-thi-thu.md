# Kế hoạch ôn lại trước khi thi thật

> Rút ra từ kết quả thi thử 76 câu (bài luyện tập trong [`01-nhat-ky-on-tap.md`](./01-nhat-ky-on-tap.md), mục Ngày 7).

📝 **Xem lại toàn bộ bài làm kèm giải thích từng câu**: https://claude.ai/code/artifact/5c357286-5fa6-46c9-b2c0-7dec3c636bdd (76 câu, tô màu đúng/sai, lọc theo "chỉ câu sai" — lưu vĩnh viễn, không phụ thuộc trình duyệt/thiết bị).
> Làm lại bài thi thử mới (trắng, chưa tô sẵn): https://claude.ai/code/artifact/921cfadd-9e3c-4c91-aec7-42fef0ffc5eb

## Điểm số

**Tổng: 52/76 (68.4%)** — dưới ngưỡng tham chiếu 72% (≈720/1000 của đề thật).

| Kịch bản | Điểm | Đánh giá |
|---|---|---|
| **Claude Code cho CI/CD** | 7/15 (47%) | 🔴 Yếu nhất — ưu tiên số 1 |
| Hệ thống Nghiên cứu Đa tác nhân | 10/15 (67%) | 🟡 Cần ôn lại |
| Customer Support Agent | 10/15 (67%) | 🟡 Cần ôn lại |
| Sinh mã với Claude Code | 11/15 (73%) | 🟢 Khá ổn |
| Mẫu kiến trúc AI hội thoại | 14/16 (88%) | 🟢 Tốt |

Tin tốt: CI/CD (yếu nhất) và phần lớn lỗi khác rơi vào Lĩnh vực 1 (27%) và Lĩnh vực 3 (20%) — đúng 2 lĩnh vực nặng điểm nhất, nên sửa đúng các lỗi này tác động lớn nhất lên điểm thi thật.

## 24 câu sai (số câu · kịch bản · đã chọn → đáp án đúng)

| Câu | Kịch bản | Chọn | Đúng |
|---|---|---|---|
| 1 | Nghiên cứu Đa tác nhân | C | D |
| 7 | Nghiên cứu Đa tác nhân | C | B |
| 10 | Nghiên cứu Đa tác nhân | D | A |
| 11 | Nghiên cứu Đa tác nhân | D | B |
| 13 | Nghiên cứu Đa tác nhân | D | C |
| 16 | CI/CD | C | B |
| 17 | CI/CD | D | A |
| 18 | CI/CD | D | B |
| 20 | CI/CD | C | D |
| 23 | CI/CD | C | A |
| 25 | CI/CD | A | D |
| 28 | CI/CD | B | A |
| 29 | CI/CD | C | A |
| 31 | Sinh mã | A | B |
| 33 | Sinh mã | C | D |
| 36 | Sinh mã | A | C |
| 38 | Sinh mã | C | D |
| 47 | Customer Support | A | C |
| 48 | Customer Support | A | C |
| 50 | Customer Support | B | C |
| 51 | Customer Support | D | C |
| 52 | Customer Support | D | A |
| 68 | Mẫu hội thoại | B | C |
| 72 | Mẫu hội thoại | B | A |

## 5 nhóm lỗi lặp lại (quan trọng hơn từng câu riêng lẻ)

### Nhóm 1 — Vẫn chọn "sửa bằng prompt" thay vì "cơ chế cứng" (Câu 10, 16, 17, 51)

Pattern quan trọng nhất, đã nhấn mạnh từ Ngày 2 nhưng vẫn sai ở đề thật.

**Quy tắc tự vấn 2 bước:**
1. Hậu quả nếu model "quên"/hiểu sai là gì? → Tiền bạc/pháp lý/an toàn/không hoàn tác được → PHẢI dùng cơ chế cứng (hook, precondition code, cờ CLI, tool validation, instance tách biệt). Chỉ là phong cách/định dạng, sửa lại được → prompt/few-shot là đủ.
2. Cơ chế cứng cụ thể là gì?
   - Business rule về thứ tự/ngưỡng → `PreToolUse` hook / precondition lập trình
   - Cần structured output đáng tin cậy → `tool_use` + schema, hoặc `--output-format json`/`--json-schema`
   - Tool bị lạm dụng ngoài phạm vi → thay bằng tool chuyên biệt hẹp hơn + validation
   - Model tự review chính nó → instance độc lập khác, không phải prompt "tự phê bình"

⚠️ Lưu ý chiều ngược: đừng áp dụng "hook" cho mọi thứ — nếu hậu quả chỉ là bất tiện nhỏ (giọng văn, ngôn ngữ ưu tiên...), prompt/few-shot vẫn là lựa chọn đúng, dùng hook ở đó là over-engineering.

### Nhóm 2 — Nhầm khi nào dùng Few-shot (Câu 7, 20, 47, 52)

Sai theo cả 2 chiều (dùng thiếu lẫn dùng thừa).

**Quy tắc tự vấn 3 bước:**
1. Agent chọn sai tool/entity vì tên/mô tả chồng lấn? → CÓ → sửa mô tả/tên tool trước, KHÔNG few-shot (Câu 7).
2. Nếu mô tả đã ổn: agent đã tốt với ca đơn giản, chỉ lúng túng ở MỘT mẫu hành vi lặp lại, minh họa được bằng vài ví dụ? → CÓ → few-shot đúng là đáp án, đừng xây thêm hạ tầng — routing layer/preprocessing/two-pass đều thừa (Câu 20, 47).
3. Nếu khoảng trống/lỗi thay đổi ngẫu nhiên theo từng case, không liệt kê trước được → cần cơ chế ĐỘNG (giai đoạn tự phê bình/self-critique), không phải few-shot cố định (Câu 52).

### Nhóm 3 — Nhầm `.claude/rules/` (theo path/glob) với Skill (gọi theo nhu cầu) (Câu 33, 38)

Sai giống hệt nhau 2 lần.

**Câu hỏi tự vấn:** "Quy ước này áp dụng vì tôi đang chạm vào MỘT LOẠI FILE, hay vì tôi đang làm MỘT TÁC VỤ cụ thể?"
- `.claude/rules/` + `paths`: tự động nạp mỗi khi mở/sửa file khớp glob, bất kể đang làm gì (VD: mọi `*.test.tsx`).
- Skill: chỉ nạp khi được gọi theo nhu cầu (slash command), gắn với một quy trình cụ thể (sinh endpoint mới, review PR, migration) — mở file trong đúng thư mục đó để làm việc khác thì không cần.

**Ví dụ:** sửa `orders.test.ts` → rule test tự nạp. Gõ `/new-endpoint` → skill nạp ví dụ mẫu endpoint. Debug bug cũ trong `orders.ts` → không cái nào nạp cả.

### Nhóm 4 — Escalation: "bằng chứng mâu thuẫn lời khách" vs "chính sách im lặng" (Câu 50)

**Quy tắc:** Chính sách CÓ đề cập (dù không có lợi cho khách) + có bằng chứng rõ ràng → agent tự tin trình bày bằng chứng/quy định, KHÔNG escalate (VD: khách nói chưa nhận hàng nhưng tracking + chữ ký đã có). Chính sách THỰC SỰ im lặng, không quy định gì → escalate vì agent không có thẩm quyền tự đặt luật mới (VD: so giá đối thủ, chính sách chỉ nói về giảm giá trên chính site mình).

### Nhóm 5 — 2 kỹ thuật ngoài phạm vi 13 chương lý thuyết (Câu 68, 72)

Chỉ cần nhớ trực tiếp, không cần suy luận:
- Hội thoại dài hàng tháng, cần tra lại một kết luận cụ thể cũ → **semantic search/embedding** để truy xuất đúng đoạn liên quan (tóm tắt lũy tiến sẽ làm mất chi tiết vì nén thành khái niệm chung chung).
- Muốn loại bỏ lời chào lặp lại ("Certainly!"...) → **prefill sẵn phần đầu tin nhắn assistant**, không phải dặn prompt hay hạ temperature (không kiểm soát được cụm từ cố định một cách đáng tin cậy).

## Kế hoạch ôn lại (5 ưu tiên, theo thứ tự)

1. **Ưu tiên 1 — CI/CD (47%, kịch bản yếu nhất)**: đọc lại Chương 5.9 (đặc biệt đoạn "ngăn bình luận trùng lặp" — Câu 25 sai đúng vào phần này dù tài liệu ghi rất rõ) và Chương 7 (Batch API — Câu 18 cho thấy hiểu batch "chậm" nhưng chưa hiểu nó **không tương thích kỹ thuật** với tool-calling nhiều vòng, không chỉ là vấn đề tốc độ).

2. **Ưu tiên 2 — pattern "prompt vs deterministic"**: làm lại phần tự kiểm tra Ngày 2 và Ngày 3, tự hỏi thêm ở mỗi câu: "đáp án nào là code/cờ CLI/tool riêng thay vì chỉ dặn prompt?"

3. **Ưu tiên 3 — `.claude/rules/` vs Skill**: đọc lại mục 3.2 và 3.3 ở Phần II, tự đặt câu hỏi "cái này áp dụng theo loại file hay theo tác vụ cụ thể?" trước khi chọn.

4. **Ưu tiên 4 — Escalation edge case**: ôn lại đúng đoạn phân biệt "chính sách im lặng" vs "bằng chứng mâu thuẫn lời khách" trong Chương 9.1.

5. Ghi nhớ trực tiếp 2 fact ngoài phạm vi (Câu 68, 72) mà không cần hiểu sâu cơ chế.
