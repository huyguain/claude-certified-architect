# Kế hoạch ôn lại trước khi thi thật

> Rút ra từ kết quả thi thử 76 câu (bài luyện tập trong [`01-nhat-ky-on-tap.md`](./01-nhat-ky-on-tap.md), mục Ngày 7).

📝 **Xem lại toàn bộ bài làm kèm giải thích từng câu**: https://claude.ai/code/artifact/5c357286-5fa6-46c9-b2c0-7dec3c636bdd (76 câu, tô màu đúng/sai, lọc theo "chỉ câu sai" — lưu vĩnh viễn, không phụ thuộc trình duyệt/thiết bị).
> Làm lại bài thi thử mới (trắng, chưa tô sẵn): https://claude.ai/code/artifact/921cfadd-9e3c-4c91-aec7-42fef0ffc5eb
> Bản tiếng Anh (gốc) của 76 câu — làm trực tiếp, hiện giải thích sau mỗi câu: https://claude.ai/code/artifact/6b78c074-650f-4237-8e19-b3a9971173b3 (nguồn: phần "Practice Test" trong `../guide_en.md`; file cục bộ `../practical_test_en_76.html`)
> Bản tiếng Anh — xem lại bài làm đã chấm (đáp án đã chọn vs. đáp án đúng, breakdown theo kịch bản, lọc "chỉ câu sai"): https://claude.ai/code/artifact/e428cc2d-bfbb-4dc0-8e68-6ba3ff32e230 (file cục bộ `../practical_test_en_76_review.html`)

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

---

# Review Plan Before the Real Exam (English)

> Derived from the 76-question mock exam results (practice run in [`01-nhat-ky-on-tap.md`](./01-nhat-ky-on-tap.md), Day 7 section).

📝 **Review the full attempt with per-question explanations**: https://claude.ai/code/artifact/5c357286-5fa6-46c9-b2c0-7dec3c636bdd (76 questions, correct/incorrect highlighting, "wrong answers only" filter — persists permanently, independent of browser/device).
> Retake a fresh mock exam (blank, unmarked): https://claude.ai/code/artifact/921cfadd-9e3c-4c91-aec7-42fef0ffc5eb
> English (original) version of the 76 questions — take it directly, explanation shown after each question: https://claude.ai/code/artifact/6b78c074-650f-4237-8e19-b3a9971173b3 (source: the "Practice Test" section in `../guide_en.md`; local file `../practical_test_en_76.html`)
> English version — review the graded attempt (selected answer vs. correct answer, breakdown by scenario, "wrong answers only" filter): https://claude.ai/code/artifact/e428cc2d-bfbb-4dc0-8e68-6ba3ff32e230 (local file `../practical_test_en_76_review.html`)

## Score

**Total: 52/76 (68.4%)** — below the 72% reference threshold (≈720/1000 on the real exam).

| Scenario | Score | Assessment |
|---|---|---|
| **Claude Code for CI/CD** | 7/15 (47%) | 🔴 Weakest — top priority |
| Multi-Agent Research System | 10/15 (67%) | 🟡 Needs review |
| Customer Support Agent | 10/15 (67%) | 🟡 Needs review |
| Code Generation with Claude Code | 11/15 (73%) | 🟢 Fairly solid |
| Conversational AI Architecture Patterns | 14/16 (88%) | 🟢 Good |

Good news: CI/CD (the weakest) and most other errors fall under Domain 1 (27%) and Domain 3 (20%) — exactly the two highest-weighted domains, so fixing these mistakes has the largest impact on the real exam score.

## 24 wrong answers (question # · scenario · chosen → correct)

| Q# | Scenario | Chosen | Correct |
|---|---|---|---|
| 1 | Multi-Agent Research | C | D |
| 7 | Multi-Agent Research | C | B |
| 10 | Multi-Agent Research | D | A |
| 11 | Multi-Agent Research | D | B |
| 13 | Multi-Agent Research | D | C |
| 16 | CI/CD | C | B |
| 17 | CI/CD | D | A |
| 18 | CI/CD | D | B |
| 20 | CI/CD | C | D |
| 23 | CI/CD | C | A |
| 25 | CI/CD | A | D |
| 28 | CI/CD | B | A |
| 29 | CI/CD | C | A |
| 31 | Code Generation | A | B |
| 33 | Code Generation | C | D |
| 36 | Code Generation | A | C |
| 38 | Code Generation | C | D |
| 47 | Customer Support | A | C |
| 48 | Customer Support | A | C |
| 50 | Customer Support | B | C |
| 51 | Customer Support | D | C |
| 52 | Customer Support | D | A |
| 68 | Conversational Patterns | B | C |
| 72 | Conversational Patterns | B | A |

## 5 recurring error patterns (more important than any single question)

### Pattern 1 — Still picking "fix with a prompt" instead of "a hard mechanism" (Q10, 16, 17, 51)

The most important pattern, already emphasized since Day 2, yet still missed on the mock exam.

**2-step self-check rule:**
1. What happens if the model "forgets" or misunderstands? → Money/legal/safety/irreversible consequences → MUST use a hard mechanism (hook, precondition code, CLI flag, tool validation, a separate instance). Just style/formatting, easily correctable → a prompt/few-shot is enough.
2. What is the concrete hard mechanism?
   - A business rule about order/threshold → a `PreToolUse` hook / programmatic precondition
   - Need reliable structured output → `tool_use` + schema, or `--output-format json`/`--json-schema`
   - A tool being misused outside its scope → replace it with a narrower, purpose-built tool + validation
   - The model reviewing itself → a genuinely separate instance, not a "self-critique" prompt

⚠️ Watch the reverse direction too: don't reach for a "hook" for everything — if the consequence is only a minor inconvenience (tone, preferred language, etc.), a prompt/few-shot is still the right choice; using a hook there is over-engineering.

### Pattern 2 — Confusing when to use few-shot (Q7, 20, 47, 52)

Wrong in both directions (under-using and over-using it).

**3-step self-check rule:**
1. Does the agent pick the wrong tool/entity because names/descriptions overlap? → YES → fix the tool's description/name first, NOT few-shot (Q7).
2. If the description is already fine: is the agent already good on simple cases, only stumbling on ONE recurring behavior pattern that can be illustrated with a few examples? → YES → few-shot is the right answer, don't build extra infrastructure — a routing layer/preprocessing/two-pass approach is all overkill (Q20, 47).
3. If the gap/error varies randomly case by case and can't be enumerated in advance → a DYNAMIC mechanism is needed (a self-critique stage), not fixed few-shot examples (Q52).

### Pattern 3 — Confusing `.claude/rules/` (path/glob-based) with Skills (invoked on demand) (Q33, 38)

Made the exact same mistake twice.

**Self-check question:** "Does this convention apply because I'm touching ONE TYPE OF FILE, or because I'm doing ONE SPECIFIC TASK?"
- `.claude/rules/` + `paths`: auto-loads any time a matching-glob file is opened/edited, regardless of what task is being done (e.g., every `*.test.tsx`).
- Skill: only loads when invoked on demand (a slash command), tied to a specific workflow (generating a new endpoint, reviewing a PR, a migration) — opening a file in that same directory for unrelated work does not load it.

**Example:** editing `orders.test.ts` → the test rule auto-loads. Typing `/new-endpoint` → the skill loads a sample endpoint template. Debugging an old bug in `orders.ts` → neither loads.

### Pattern 4 — Escalation: "evidence contradicts the customer's claim" vs. "policy is silent" (Q50)

**Rule:** If policy DOES address the situation (even if unfavorable to the customer) AND there is clear evidence → the agent should confidently present the evidence/policy, NOT escalate (e.g., customer claims non-delivery but tracking + signature confirmation exist). If policy is TRULY silent, with no rule covering the situation → escalate, because the agent has no authority to invent a new rule (e.g., matching a competitor's price, when policy only covers discounts on the company's own site).

### Pattern 5 — 2 techniques outside the 13-chapter theory scope (Q68, 72)

Just memorize directly, no reasoning required:
- A months-long conversation that needs to retrieve one specific old conclusion → **semantic search/embeddings** to fetch the exact relevant passage (progressive summarization loses detail because it compresses into generic concepts).
- Wanting to eliminate repetitive greetings ("Certainly!"...) → **prefill the start of the assistant's message**, not prompting for it or lowering temperature (neither reliably controls a specific fixed phrase).

## Review plan (5 priorities, in order)

1. **Priority 1 — CI/CD (47%, weakest scenario)**: re-read Chapter 5.9 (especially the "preventing duplicate comments" section — Q25 was missed exactly here despite the documentation being very explicit) and Chapter 7 (Batch API — Q18 shows understanding that batch is "slow" but not yet that it is **technically incompatible** with multi-turn tool-calling, not merely a speed issue).

2. **Priority 2 — the "prompt vs. deterministic" pattern**: redo the Day 2 and Day 3 self-check sections, and for each question additionally ask: "which answer is code/a CLI flag/a dedicated tool rather than just a prompt instruction?"

3. **Priority 3 — `.claude/rules/` vs. Skill**: re-read sections 3.2 and 3.3 in Part II, and ask "does this apply based on file type or based on a specific task?" before choosing.

4. **Priority 4 — the escalation edge case**: re-review the exact section distinguishing "policy is silent" from "evidence contradicts the customer's claim" in Chapter 9.1.

5. Memorize the 2 out-of-scope facts directly (Q68, 72) without needing to understand the underlying mechanism deeply.
