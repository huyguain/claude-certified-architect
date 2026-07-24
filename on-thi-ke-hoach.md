# Kế hoạch & Nhật ký ôn thi — Claude Certified Architect (7 ngày)

> File này lưu lại toàn bộ lộ trình + nội dung đã ôn, để tiếp tục học ở bất kỳ máy nào (nhà, công ty...).
> Nguồn gốc: `guide_vi.md` (3405 dòng) trong repo này.

---

## Tiến độ

- [x] **Ngày 1** — Nền tảng Claude API & Tool Use (Ch.1–2)
- [x] **Ngày 2** — Kiến trúc & Điều phối Agent (Ch.3, Ch.8, Lĩnh vực 1 — 27%)
- [ ] **Ngày 3** — Tool Design, MCP & cấu hình Claude Code (Ch.4, 5, 13, Lĩnh vực 2 — 18% & 3 — 20%)
- [ ] **Ngày 4** — Prompt Engineering & Structured Output (Ch.6–7, Lĩnh vực 4 — 20%)
- [ ] **Ngày 5** — Quản lý Context & Độ tin cậy (Ch.9–12, Lĩnh vực 5 — 15%)
- [ ] **Ngày 6** — Ôn tổng hợp + 12 câu hỏi mẫu có giải thích
- [ ] **Ngày 7** — Thi thử toàn phần (76 câu luyện tập)

---

## Bối cảnh thi (nhắc nhanh)

| Tham số | Giá trị |
|---|---|
| 5 lĩnh vực | 1. Kiến trúc/điều phối agent (**27%**) · 2. Tool/MCP (**18%**) · 3. Claude Code (**20%**) · 4. Prompt engineering/structured output (**20%**) · 5. Context/độ tin cậy (**15%**) |
| Định dạng | Trắc nghiệm 1/4 đáp án, 4 trong 8 kịch bản (ngẫu nhiên) |
| Thang điểm | 100–1000, đạt **720** |
| Phạt đoán | Không — trả lời hết mọi câu |

**8 kịch bản thi**: (1) Customer Support Agent, (2) Sinh mã với Claude Code, (3) Multi-Agent Research, (4) Developer Productivity Tools, (5) Claude Code cho CI/CD, (6) Trích xuất Dữ liệu Có cấu trúc, (7) Mẫu kiến trúc AI hội thoại, (8) Công cụ Agentic AI *(tài liệu gốc ghi nhận thiếu nội dung)*.

⚠️ **Lưu ý quan trọng**: bộ 76 câu luyện tập trong `guide_vi.md` chỉ phủ 5/8 kịch bản (Multi-agent Research, CI/CD, Sinh mã, Customer Support, Mẫu hội thoại). Ba kịch bản còn lại — *Developer Productivity Tools*, *Trích xuất Dữ liệu Có cấu trúc*, *Công cụ Agentic AI* — không có câu hỏi riêng, cần tự ánh xạ sang lý thuyết tương ứng (Ch.13 + Lĩnh vực 5.4 cho kịch bản 4; Ch.2/6/7 + Lĩnh vực 4 cho kịch bản 6).

**Mẹo làm bài**: (1) không bị trừ điểm khi đoán — trả lời mọi câu; (2) câu hỏi kiểm tra *trade-off* kiến trúc trong tình huống thực tế, không phải học thuộc định nghĩa; (3) ưu tiên thời gian ôn theo trọng số — Lĩnh vực 1 (27%) và 3-4 (20% mỗi lĩnh vực) nên chiếm phần lớn thời gian.

---

## NGÀY 1 — Nền tảng Claude API & Tool Use

### Chương 1 — Claude API

- Request gồm 6 trường chính: `model`, `max_tokens`, `system`, `messages`, `tools`, `tool_choice`.
- ⚠️ Model **không lưu trạng thái** giữa các request — phải gửi lại toàn bộ `messages` mỗi lần.
- 3 vai trò trong `messages`: `user`, `assistant`; kết quả tool xuất hiện dưới dạng content block `tool_result` (không phải role "tool" riêng).

**Bảng `stop_reason`:**

| Giá trị | Ý nghĩa | Hành động |
|---|---|---|
| `end_turn` | Model đã xong | Hiển thị cho người dùng |
| `tool_use` | Model muốn gọi tool | Thực thi tool, trả kết quả |
| `max_tokens` | Hết giới hạn token | Có thể bị cắt cụt (lỗi cấu hình) |
| `stop_sequence` | Gặp stop sequence | Tùy logic ứng dụng |

- **System prompt**: tách khỏi `messages`, ưu tiên cao hơn user message. Bẫy thi: câu như "luôn xác minh khách hàng" có thể khiến model lạm dụng tool xác minh dù không cần thiết → nên viết có điều kiện.
- **Context window** — 3 vấn đề:
  1. *Lost-in-the-middle*: thông tin giữa input dài dễ bị bỏ sót → đặt thông tin quan trọng ở đầu/cuối.
  2. *Tích lũy tool result*: tool trả 40 trường nhưng chỉ cần 5 → lãng phí context.
  3. *Tóm tắt lũy tiến*: số liệu/ngày tháng dễ mất khi nén lịch sử ("127 đơn" → "khoảng 100 đơn").

### Chương 2 — Tools và `tool_use`

- **Mô tả tool là cơ chế lựa chọn chính** — model chọn tool dựa vào `description`. Mô tả tốt cần: tool làm gì/trả gì, định dạng input, edge case, và *khi nào dùng tool này thay vì tool tương tự*.
- ⚠️ Agent có thể ưu tiên built-in tool (Read, Grep) hơn MCP tool tương đương → cần viết mô tả MCP tool nhấn mạnh lợi thế/dữ liệu độc nhất.

**Bảng `tool_choice`:**

| Giá trị | Hành vi | Khi dùng |
|---|---|---|
| `{"type":"auto"}` | Model tự quyết | Mặc định |
| `{"type":"any"}` | Bắt buộc gọi *một* tool nào đó | Cần đảm bảo có structured output |
| `{"type":"tool","name":"x"}` | Bắt buộc gọi *đúng* tool đó | Cần ép thứ tự/bước đầu tiên cụ thể |

- **JSON schema**: đảm bảo đúng cú pháp/cấu trúc, **KHÔNG** đảm bảo đúng ngữ nghĩa (giá trị vẫn có thể sai).
- Quy tắc thiết kế schema: chỉ `required` nếu thông tin *luôn* có sẵn; dùng nullable (`["string","null"]`) cho thông tin có thể vắng mặt; enum nên có `"other"` (+ chi tiết) và `"unclear"`.
- **Cú pháp vs ngữ nghĩa**: lỗi cú pháp bị loại bỏ hoàn toàn bằng `tool_use` + schema; lỗi ngữ nghĩa (hallucination, sai trường) chỉ giảm được bằng validation + retry-with-feedback.

### Tự kiểm tra Ngày 1 (đã làm — xem lại nếu cần ôn lại)

1. `tool_choice: "any"` + 3 tool trích xuất → model bắt buộc gọi 1 tool nhưng tự chọn tool phù hợp nhất.
2. `stop_reason: "tool_use"` → thực thi tool, trả `tool_result` vào `messages`, gọi lại API.
3. Trường chỉ có ở 30% đơn hàng → nullable, không `required` (tránh model bịa giá trị).
4. Agent dùng Grep thay vì MCP tool tương đương → mô tả MCP tool chưa đủ nổi bật.
5. System prompt "luôn xác minh khách hàng" → rủi ro lạm dụng tool xác minh không cần thiết.

---

## NGÀY 2 — Kiến trúc & Điều phối Agent (Lĩnh vực 1 — 27%, nặng nhất)

### 3.1 Agentic Loop

```
1. Gửi request tới Claude kèm tools
2. Nhận response
3. Kiểm tra stop_reason:
   - "tool_use" → thực thi tool, nối kết quả vào history, quay lại bước 1
   - "end_turn" → hoàn tất, trả kết quả cho người dùng
4. Lặp lại đến khi xong
```

Model-driven: Claude tự quyết định tool kế tiếp dựa trên context, khác với cây quyết định hard-code.

⚠️ **3 anti-pattern**: (1) parse text tìm "Task completed"; (2) dùng `max_iterations` tùy ý làm điều kiện dừng chính; (3) coi "có sinh text" là dấu hiệu xong.
✅ Tín hiệu hoàn thành duy nhất đáng tin cậy: `stop_reason == "end_turn"`.

### 3.2–3.4 AgentDefinition, Hub-and-Spoke, Tool `Task`

- `AgentDefinition` gồm `name`, `description`, `system_prompt`, `allowed_tools` (nguyên tắc least privilege).
- Topology hub-and-spoke: Coordinator ở giữa, spawn nhiều subagent chuyên biệt.
- Trách nhiệm coordinator: phân rã tác vụ → chọn subagent động → ủy thác → tổng hợp/validate → xử lý lỗi/retry → giao tiếp người dùng.
- 🔑 **Nguyên tắc quan trọng nhất**: subagent có **context tách biệt hoàn toàn** — không tự kế thừa lịch sử coordinator, không chia sẻ bộ nhớ giữa các lần gọi. Mọi thứ cần phải truyền tường minh vào prompt.
- Ví dụ TỆ: `Task: "Analyze the document"` (không có tài liệu đính kèm) → subagent bịa nội dung.
- Ví dụ TỐT: đính kèm toàn bộ tài liệu + kết quả trước đó + yêu cầu định dạng output.
- **Sinh song song**: nhiều `Task` trong cùng một response của coordinator → chạy đồng thời.
- `allowedTools` của coordinator **phải** chứa `"Task"`.

### 3.5 Hooks

| | Hook | Chỉ dẫn trong prompt |
|---|---|---|
| Đảm bảo | Deterministic (100%) | Xác suất (>90%, không phải 100%) |
| Dùng khi | Quy tắc tài chính/pháp lý/tuân thủ | Tùy chọn chung, khuyến nghị |

- `PostToolUse`: biến đổi/chuẩn hóa **kết quả** tool trước khi model thấy (VD: chuẩn hóa timestamp).
- `PreToolUse`: chặn **hành động** trước khi xảy ra (VD: chặn refund > $500, redirect escalation).
- **Quy tắc**: sai sót gây thiệt hại tiền bạc/pháp lý/an toàn → dùng hook; chỉ là khuyến nghị/định dạng → prompt là đủ.

### Chương 8 — Chiến lược phân rã tác vụ

| Chiến lược | Khi dùng | Ví dụ |
|---|---|---|
| Pipeline cố định (prompt chaining) | Cấu trúc dự đoán được, cần ổn định | Metadata → Data extraction → Validation → Enrichment |
| Phân rã thích ứng động | Tác vụ điều tra mở, không biết trước phạm vi | "Thêm test cho codebase cũ" → map cấu trúc → phát hiện dần → điều chỉnh |
| Multi-pass review | PR có 10+ file | Lượt 1: từng file riêng; Lượt 2: phân tích quan hệ chéo file |

⚠️ Lý do multi-pass tốt hơn 1 lượt: attention dilution, nhận xét không nhất quán, bỏ sót lỗi do quá tải.

### Session, Resume, Fork (Lĩnh vực 1, mục 1.7)

- `--resume <session-name>`: tiếp tục session đã đặt tên — cần báo agent nếu file đã thay đổi.
- `fork_session`: tạo nhánh điều tra độc lập từ cùng context gốc — so sánh song song nhiều phương án.
- Chọn resume (context còn hiện hành) vs session mới với tóm tắt có cấu trúc (kết quả cũ đã lỗi thời).

### Tự kiểm tra Ngày 2 (đã làm — xem lại nếu cần ôn lại)

1. Nghiên cứu 3 chủ đề độc lập nhanh nhất → gọi `Task` 3 lần trong **cùng một response** để chạy song song.
2. Dùng text "Done" để xác định hoàn tất → anti-pattern; chỉ tin `stop_reason == "end_turn"`.
3. Chính sách "luôn xác minh trước khi xóa tài khoản" → cần precondition theo lập trình (hook), không chỉ prompt.
4. Coordinator giao việc không kèm tài liệu → subagent có thể bịa nội dung (context tách biệt).
5. PR 18 file → multi-pass (từng file riêng + 1 lượt tích hợp).
6. `fork_session` dùng khi so sánh song song nhiều phương án từ cùng context, không ảnh hưởng session gốc.

### Chương 3 — giải thích lại dễ hiểu (ẩn dụ)

- **Agentic loop** = trò chuyện lặp lại với "nhân viên" không nhớ gì trừ khi bạn gửi lại toàn bộ lịch sử. Claude chỉ **chỉ đường** ("tôi muốn gọi tool X"), code của bạn mới là người **thực sự đi** (chạy tool).
- **AgentDefinition** = bản mô tả công việc: tên, nhiệm vụ, quy tắc, và **công cụ được phép dùng** (least privilege).
- **Hub-and-spoke** = trưởng nhóm giao việc cho các chuyên viên; mỗi chuyên viên **chỉ biết những gì được nói cho nghe** — không tự "nghe lỏm" các cuộc họp riêng trước đó. Quên chép thông tin → chuyên viên tự bịa.
- **Tool `Task`** = hành động giao việc. Giao việc kèm đầy đủ ngữ cảnh (tài liệu, kết quả trước, format mong muốn) — không giao việc mập mờ. Việc độc lập → giao song song (nhiều `Task` trong 1 lượt).
- **Hooks** = trạm kiểm soát an ninh tự động (chặn cứng, luôn đúng) khác với biển báo nhắc nhở (chỉ dẫn prompt, đa số tuân theo nhưng không phải 100%). Sai sót gây hậu quả tiền bạc/pháp lý/an toàn → hook; còn lại → prompt là đủ.

**Ví dụ trace toàn bộ chương 3** (hoàn tiền $800, vượt ngưỡng $500):
```
1. Coordinator (allowed_tools có "Task") nhận yêu cầu hoàn tiền đơn #123, $800.
2. Coordinator gọi Task → spawn "refund_specialist", TRUYỀN KÈM: đơn #123, ID khách đã xác minh=456, số tiền $800.
3. Subagent gọi process_refund(amount=800).
4. PreToolUse hook: 800 > 500 → CHẶN, redirect sang escalate_to_human (chặn cứng, không phụ thuộc prompt).
5. Subagent trả kết quả escalation về Coordinator.
6. Coordinator tạo bản tóm tắt bàn giao có cấu trúc cho con người.
7. stop_reason = "end_turn" → hoàn tất, báo khách hàng đã được chuyển lên cấp trên.
```

---

## NGÀY 3 (chưa học) — Tool Design, MCP & cấu hình Claude Code

- [ ] Chương 4 — MCP server, cấu hình, cờ `isError`, MCP Resources (`guide_vi.md` L446–546)
- [ ] Chương 13 — Lựa chọn tool dựng sẵn, điều tra tăng dần, Read+Write thay Edit (L1477–1510)
- [ ] Chương 5 — Phân cấp CLAUDE.md, `@path`, `.claude/rules/`, slash command & skill, planning mode, `/compact`, `/memory`, CI/CD, `fork_session` (L547–801)
- [ ] Phần II — Lĩnh vực 2 (L1607–1677) và Lĩnh vực 3 (L1678–1763)

## NGÀY 4 (chưa học) — Prompt Engineering & Structured Output

- [ ] Chương 6 — Few-shot, tiêu chí rõ ràng, prompt chaining, mẫu "phỏng vấn", validation/retry, tự sửa lỗi (L802–1015)
- [ ] Chương 7 — Batch API: khi nào dùng, `custom_id`, xử lý thất bại, SLA (L1016–1075)
- [ ] Phần II — Lĩnh vực 4 (L1764–1848)

## NGÀY 5 (chưa học) — Quản lý Context & Độ tin cậy

- [ ] Chương 9 — Escalation, mẫu escalation, structured handoff, hiệu chỉnh độ tin cậy (L1129–1229)
- [ ] Chương 10 — Nhóm lỗi, anti-pattern, lỗi subagent có cấu trúc, chú thích độ bao phủ (L1230–1291)
- [ ] Chương 11 — Trích xuất sự kiện, cắt gọn tool result, scratchpad, ủy quyền subagent (L1292–1410)
- [ ] Chương 12 — Bảo toàn provenance, xử lý dữ liệu xung đột (L1411–1476)
- [ ] Phần II — Lĩnh vực 5 (L1849–1937)

## NGÀY 6 (chưa học) — Ôn tổng hợp

- [ ] 12 câu hỏi mẫu có giải thích (L1938–2119)
- [ ] Đọc "Các chủ đề ngoài phạm vi" cuối tài liệu
- [ ] Rà lại checklist "Kiến thức/Kỹ năng trọng tâm" của 5 lĩnh vực

## NGÀY 7 (chưa học) — Thi thử toàn phần

- [ ] 76 câu bài luyện tập (L2120–3405), tính giờ như thi thật
- [ ] Rà soát câu sai, quay lại đúng chương liên quan
- [ ] Ôn nhanh 8 kịch bản, đặc biệt 3 kịch bản không có câu hỏi riêng

---

*Cập nhật lần cuối: sau khi hoàn thành Ngày 2 + đào sâu Chương 3.*
