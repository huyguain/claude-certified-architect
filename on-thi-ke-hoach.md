# Kế hoạch & Nhật ký ôn thi — Claude Certified Architect (7 ngày)

> File này lưu lại toàn bộ lộ trình + nội dung đã ôn, để tiếp tục học ở bất kỳ máy nào (nhà, công ty...).
> Nguồn gốc: `guide_vi.md` (3405 dòng) trong repo này.

---

## Tiến độ

- [x] **Ngày 1** — Nền tảng Claude API & Tool Use (Ch.1–2)
- [x] **Ngày 2** — Kiến trúc & Điều phối Agent (Ch.3, Ch.8, Lĩnh vực 1 — 27%)
- [x] **Ngày 3** — Tool Design, MCP & cấu hình Claude Code (Ch.4, 5, 13, Lĩnh vực 2 — 18% & 3 — 20%)
- [x] **Ngày 4** — Prompt Engineering & Structured Output (Ch.6–7, Lĩnh vực 4 — 20%)
- [x] **Ngày 5** — Quản lý Context & Độ tin cậy (Ch.9–12, Lĩnh vực 5 — 15%)
- [x] **Ngày 6** — Ôn tổng hợp + 12 câu hỏi mẫu có giải thích
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

**Cùng ví dụ trên, kể theo kiểu "câu chuyện" (dễ nhớ hơn):**

Một trung tâm chăm sóc khách hàng có "quản lý ca" (Coordinator), một "nhân viên xử lý hoàn tiền" (Subagent), và một **máy kiểm soát tự động** ở cửa (Hook) mà không ai tắt được.

1. Khách nhắn: "Hoàn tiền đơn #123, $800."
2. Quản lý **không tự xử lý** — giao việc cho nhân viên hoàn tiền, nhưng giao **rất cụ thể**: "Đơn #123, khách đã xác minh mã 456, số tiền $800" (không nói cộc lốc "đi hoàn tiền đi", vì nhân viên mới không hề biết cuộc trò chuyện trước đó — nó chỉ biết đúng những gì được ghi trong tờ giấy giao việc).
3. Nhân viên chuẩn bị bấm "Hoàn tiền $800".
4. Máy kiểm soát tự động ở cửa kêu "bíp", chặn lại — vì có luật cài sẵn: "hoàn tiền trên $500 phải dừng". Máy này chặn **100% các trường hợp**, không có ngoại lệ, kể cả khi bảng nội quy trên tường (giống system prompt) có ghi "cố gắng giải quyết nhanh cho khách". Bảng nội quy có thể bị đọc lướt/quên; **cái máy thì không bao giờ quên**.
5. Nhân viên không tự quyết được nữa → báo lại cho quản lý: "Vượt hạn mức, cần chuyển cấp trên."
6. Quản lý viết một **tờ trình rõ ràng** (đơn nào, khách nào, số tiền, lý do bị chặn) — không chỉ nói miệng — để người xử lý tiếp theo không phải hỏi lại từ đầu.
7. Quản lý báo khách: "Yêu cầu đã được ghi nhận, chuyển cấp trên phê duyệt."

| Trong câu chuyện | Thuật ngữ kỹ thuật |
|---|---|
| Quản lý ca | Coordinator |
| Giao việc kèm đầy đủ chi tiết | Task với context tường minh |
| Nhân viên hoàn tiền | Subagent (`refund_specialist`) |
| Máy kiểm soát tự động ở cửa | `PreToolUse` hook |
| Máy chặn 100%, không ngoại lệ | Deterministic (đảm bảo tuyệt đối) |
| Bảng nội quy trên tường | Chỉ dẫn trong system prompt (chỉ xác suất) |
| Nhân viên báo lại cho quản lý | Escalation (subagent → coordinator) |
| Tờ trình rõ ràng | Structured handoff |
| "Đã ghi nhận, chuyển cấp trên" | `stop_reason: "end_turn"` |

> 🎯 **Bài học cốt lõi cho đề thi**: câu hỏi kiểu "làm sao đảm bảo *chắc chắn 100%*" một quy tắc tiền bạc/an toàn → đáp án luôn là **hook**, không bao giờ là "viết thêm vào prompt" (vì prompt chỉ mang tính xác suất).

### Chương 8 & Session/Resume/Fork — giải thích dễ hiểu (ẩn dụ)

**8.1 Pipeline cố định** = giống một **công thức nấu ăn quen thuộc**: sơ chế → ướp → xào → nêm nếm → trình bày, lần nào cũng đúng thứ tự đó. Dùng khi công việc **lặp lại theo cùng khuôn mẫu**, cần ổn định/dễ tái lập hơn là linh hoạt.

**8.2 Phân rã thích ứng động** = giống một **thám tử điều tra vụ án**: không thể lên kế hoạch "bước 1, 2, 3" trước khi biết vụ án là gì — phải tìm manh mối đầu tiên, dựa vào đó quyết định điều tra tiếp ở đâu, phát hiện thêm thì lại điều chỉnh hướng đi. Dùng khi **không biết trước phạm vi**, mỗi bước phụ thuộc kết quả bước trước.

**8.3 Multi-pass review** = giống **chấm thi tránh gian lận**: nếu đọc liên tục 14 bài trong 1 lượt, bài đầu được chấm kỹ còn bài cuối bị đọc lướt (không nhất quán), và muốn phát hiện 2 bài copy nhau thì phải nhớ chi tiết cả 14 bài cùng lúc (gần như bất khả thi). Cách đúng: chấm **từng bài riêng trước** (lượt 1), rồi làm **một lượt riêng chỉ để so sánh chéo** (lượt 2) — áp dụng cho review PR 10+ file: lượt 1 từng file riêng, lượt 2 phân tích quan hệ chéo file (lỗi kiểu dữ liệu không khớp, phụ thuộc vòng tròn).

**Session/Resume/Fork** = giống **file lưu game (savegame)**:
- `--resume` = load lại đúng file save đang chơi dở, tiếp tục từ chỗ dừng. ⚠️ Nhưng nếu ai đó "âm thầm sửa code" trong lúc bạn vắng mặt, phải **báo cho agent biết** — nếu không nó vẫn tưởng mọi thứ y hệt lúc dừng, dẫn đến quyết định sai vì dùng thông tin lỗi thời.
- `fork_session` = **copy file save thành 2 bản** để thử 2 hướng đi khác nhau mà không sợ hỏng bản chính — dùng khi phân vân giữa 2 phương án kiến trúc, muốn so sánh song song từ cùng một điểm xuất phát.
- Chọn resume khi context cũ **vẫn còn đúng**; tạo session mới kèm tóm tắt có cấu trúc khi context cũ **đã lỗi thời**.

| Khái niệm | Ẩn dụ dễ nhớ |
|---|---|
| Pipeline cố định | Công thức nấu ăn — luôn theo đúng thứ tự |
| Phân rã thích ứng động | Thám tử điều tra — quyết định bước tiếp theo dựa vào manh mối vừa tìm |
| Multi-pass review | Chấm thi 2 vòng — từng bài riêng, rồi so sánh chéo |
| `--resume` | Load file save cũ — nhớ báo nếu "thế giới" đã thay đổi khi vắng mặt |
| `fork_session` | Copy file save thành 2 nhánh — thử song song không sợ hỏng bản chính |

---

## NGÀY 3 — Tool Design, MCP & Cấu hình Claude Code (Lĩnh vực 2 — 18% & 3 — 20%)

### Chương 4 — MCP

- 3 loại tài nguyên: **Tools** (hành động), **Resources** (dữ liệu để đọc, không cần hành động), **Prompts** (mẫu định sẵn).
- Kết nối MCP server → tool được khám phá tự động, tool từ **mọi server đã kết nối dùng được đồng thời**.
- `.mcp.json` (dự án, qua VCS, secret bằng biến môi trường `${GITHUB_TOKEN}`) vs `~/.claude.json` (người dùng, không chia sẻ, thử nghiệm cá nhân).
- Ưu tiên MCP server cộng đồng có sẵn (Jira/GitHub/Slack); chỉ tự xây cho workflow đặc thù riêng.
- ⚠️ **Cờ `isError`**: lỗi có cấu trúc (`errorCategory`, `isRetryable`, `message`) giúp agent quyết định đúng; lỗi chung chung (`"Operation failed"`) không cho biết nên retry/đổi truy vấn/escalate.
- **MCP Resources**: agent không cần tool "thăm dò" để hiểu dữ liệu gì tồn tại — resource cho sẵn "tấm bản đồ".

### Chương 13 — Tool dựng sẵn

| Tác vụ | Tool |
|---|---|
| Tìm file theo tên/mẫu | Glob |
| Tìm nội dung trong file | Grep |
| Đọc toàn bộ file | Read |
| Ghi file mới | Write |
| Sửa chính xác 1 đoạn | Edit |
| Chạy lệnh shell | Bash |

- Điều tra tăng dần: Grep tìm entry point → Read → Grep tìm nơi dùng → Read tiếp → lặp lại, không đọc hết mọi file cùng lúc.
- Khi Edit thất bại (khớp không duy nhất) → fallback Read (nạp toàn bộ) → sửa bằng chương trình → Write (ghi lại).

### Chương 5 — Claude Code: Cấu hình & Workflow

- **3 cấp CLAUDE.md**: người dùng (`~/.claude/CLAUDE.md`, không qua VCS) / dự án (`.claude/CLAUDE.md` hoặc root, qua VCS, cho cả nhóm) / thư mục (quy ước riêng phần đó). ⚠️ Bẫy kinh điển: chỉ dẫn dự án bị đặt nhầm ở mức người dùng → thành viên mới không nhận được.
- `@path`: import file khác để mô-đun hóa, không cách sau `@`, độ sâu lồng tối đa **5**.
- `.claude/rules/` + YAML frontmatter `paths`: chỉ nạp quy tắc khi sửa file khớp glob → tiết kiệm context; dùng khi quy ước trải rộng nhiều thư mục (test, migration), khác CLAUDE.md cấp thư mục (gắn với 1 thư mục cụ thể).
- Slash command/Skill: `.claude/commands/` (cũ) đã hợp nhất `.claude/skills/` (hiện tại, `SKILL.md` + frontmatter). `context: fork` = chạy trong subagent tách biệt; `allowed-tools` = giới hạn tool; `argument-hint` = gợi ý tham số.
- **Planning mode** (thay đổi lớn, nhiều phương án, codebase lạ, migration 45+ file) vs **thực thi trực tiếp** (sửa lỗi 1 file rõ ràng). Kết hợp: planning điều tra/thiết kế → duyệt → thực thi. Subagent **Explore** tách output khám phá dài khỏi context chính.
- `/compact`: nén context, rủi ro mất số liệu chính xác. `/memory`: mở CLAUDE.md để ghi nhớ qua session.
- CI/CD: `-p`/`--print` bắt buộc cho non-interactive; `--output-format json` + `--json-schema` cho structured output. ⚠️ **Cô lập context session**: session vừa sinh code thì kém khách quan khi tự review chính nó → dùng instance độc lập.

### Trắc nghiệm Ngày 3 (đã làm)

1. Lỗi MCP chung chung "Operation failed" → agent không đủ thông tin quyết định retry/escalate.
2. Secret trong `.mcp.json` dùng chung → biến môi trường `${GITHUB_TOKEN}`, không commit token thật.
3. Thành viên mới không theo chuẩn code dự án → chỉ dẫn bị đặt nhầm ở mức người dùng thay vì mức dự án.
4. Quy ước chỉ áp dụng cho `src/api/**` → `.claude/rules/` với `paths` để tiết kiệm context.
5. Pipeline CI treo chờ input → dùng `-p` + `--output-format json --json-schema`.
6. Tự review code mình vừa viết → kém khách quan, nên dùng instance độc lập.

### Giải thích dễ hiểu — ẩn dụ Ngày 3

- **MCP** = chuẩn ổ cắm USB-C chung cho mọi hệ thống ngoài — cắm vào là dùng được ngay, không cần "hàn dây" riêng.
- **`.mcp.json` vs `~/.claude.json`** = hộp đồ nghề để ở văn phòng (chung cả team, không chứa mật khẩu thật) vs hộp đồ nghề để ở nhà (riêng tư, thử nghiệm).
- **`isError` có cấu trúc vs chung chung** = kết quả khám bệnh chi tiết ("cảm cúm nhẹ, nghỉ ngơi 3 ngày") vs chỉ nói "bạn bị bệnh" (vô dụng để quyết định hành động).
- **MCP Resource** = tấm bản đồ thành phố, thay vì phải hỏi đường từng người (tool thăm dò).
- **6 tool dựng sẵn** = bộ đồ nghề thám tử: danh bạ (Glob), kính lúp (Grep), sổ đọc (Read), giấy viết (Write), bút xóa (Edit), bộ đàm (Bash).
- **3 cấp CLAUDE.md** = sổ tay cá nhân / nội quy công ty (chung cả nhóm) / sổ tay riêng từng phòng ban.
- **`@path`** = trích dẫn tài liệu gốc thay vì chép lại — một nguồn sự thật duy nhất.
- **`.claude/rules/` + `paths`** = tờ hướng dẫn chỉ phát đúng lúc cần theo loại việc, không phát tràn lan cho mọi người.
- **Skill + `context: fork`** = thẻ công thức nấu ăn có sẵn + bếp phụ riêng để không làm bẩn bếp chính.
- **Planning mode vs thực thi trực tiếp** = kiến trúc sư vẽ bản vẽ chờ duyệt vs thợ sửa ống nước làm ngay việc đã rõ.
- **`/compact`** = tóm tắt biên bản họp dài — dễ mất số liệu chi tiết. **`/memory`** = sổ tay để bàn, còn nguyên qua các ca làm việc.
- **`-p` trong CI** = thanh tra viên viết báo cáo rồi rời đi, không đứng chờ trả lời.
- **Cô lập session khi review** = nguyên tắc "không tự chấm bài của chính mình".

---

## NGÀY 4 — Prompt Engineering & Structured Output (Lĩnh vực 4 — 20%)

### Chương 6 — Prompt Engineering

- **Few-shot**: 2-4 ví dụ input/output hiệu quả hơn mô tả bằng lời (mơ hồ, hiểu nhiều cách). 5 loại: kịch bản nhập nhằng, định dạng output, phân biệt code chấp nhận được/có vấn đề, trích xuất đa định dạng, đo lường phi chuẩn ("một nhúm muối" → quy đổi cụ thể).
- **Tiêu chí rõ ràng vs mơ hồ**: liệt kê điều kiện NÊN/KHÔNG NÊN gắn cờ cụ thể, thay vì tính từ mơ hồ ("thận trọng hơn"). ⚠️ False positive cao ở 1 hạng mục → xói mòn niềm tin cả các hạng mục đúng khác.
- **Prompt chaining**: từng bước tập trung (từng file riêng → 1 lượt tích hợp) tránh attention dilution; dùng cho tác vụ dự đoán được, khác phân rã động (điều tra mở).
- **Mẫu "phỏng vấn"**: Claude hỏi lại trước khi triển khai (lĩnh vực lạ, hệ quả không hiển nhiên, nhiều cách tiếp cận khả thi).
- **Validation & Retry-with-feedback**: retry kèm tài liệu gốc + bản sai + lỗi cụ thể. ⚠️ Retry CÓ tác dụng: sai định dạng/cấu trúc/số học tự kiểm tra được. Retry KHÔNG tác dụng: thông tin không có trong nguồn, hoặc cần ngữ cảnh từ tài liệu khác không được cung cấp.
- Pydantic: validate cấu trúc (kiểu, required, enum) + validate ngữ nghĩa (business rule tùy chỉnh) + có thể tự sinh JSON Schema cho `tool_use`.
- **Tự sửa lỗi**: trích xuất cả `stated_total` và `calculated_total`, gắn cờ `conflict_detected` nếu lệch nhau.

### Chương 7 — Message Batches API

| Thuộc tính | Giá trị |
|---|---|
| Tiết kiệm | 50% so với đồng bộ |
| Cửa sổ xử lý | Tới 24 giờ, không cam kết SLA latency |
| Tool nhiều lượt | Không hỗ trợ (1 request = 1 response) |
| Tương quan | `custom_id` |

- Batch cho: báo cáo qua đêm, kiểm toán hàng tuần, khối lượng lớn không cần phản hồi ngay. Synchronous cho: kiểm tra chặn trước merge, review tương tác — nơi có người đang chờ.
- `custom_id`: liên kết kết quả↔tài liệu gốc, khi lỗi chỉ gửi lại đúng phần lỗi.
- SLA: cần trong 30h, Batch mất tới 24h → cửa sổ gửi an toàn = 30 − 24 = 6h.

### Review đa instance & đa lượt (mục 4.6)

- Model tự review code mình viết → khó tự thách thức quyết định của chính mình; instance độc lập tìm lỗi tinh vi tốt hơn.
- Review đa lượt: từng file riêng + 1 lượt tích hợp; dùng độ tự tin tự đánh giá để định tuyến review cần người kiểm tra thêm.

### Trắc nghiệm Ngày 4 (đã làm)

1. Lỗi số học "total=150 nhưng sum=145" → retry kèm tài liệu gốc + bản sai + lỗi cụ thể.
2. Thông tin hoàn toàn không có trong tài liệu nguồn → retry không giúp ích.
3. 10.000 tài liệu, không cần phản hồi tức thì → Batch API (tiết kiệm 50%).
4. Batch 200 tài liệu, 12 lỗi → dùng `custom_id` định vị, chỉ gửi lại 12 tài liệu lỗi.
5. Tự review code mình vừa viết → kém khách quan, nên dùng instance độc lập.
6. "Hãy thận trọng" gây nhiều false positive → thay bằng tiêu chí cụ thể NÊN/KHÔNG NÊN gắn cờ.

### Giải thích dễ hiểu — ẩn dụ Ngày 4

- **Few-shot** = dạy gấp áo bằng cách gấp mẫu trước mặt, không chỉ nói "gọn gàng vào".
- **Tiêu chí rõ ràng** = bảo vệ cổng có checklist tuổi rõ ràng, thay vì "nhìn mặt đoán tuổi".
- **Prompt chaining** = chấm thi 2 vòng: từng bài riêng, rồi so sánh chéo.
- **Mẫu phỏng vấn** = thợ may đo ni trước khi cắt vải, tránh cắt sai phải bỏ cả tấm.
- **Retry-with-feedback** = giáo viên chỉ đúng chỗ sai cụ thể, thay vì chỉ nói "sai, làm lại".
- **Khi retry vô ích** = cố đoán số điện thoại chưa ai từng cho bạn biết — dù đoán bao nhiêu lần cũng vô ích.
- **Tự sửa lỗi** = thu ngân cộng tiền 2 cách (từng món + máy tính tiền) rồi đối chiếu.
- **Synchronous vs Batch** = chuyển phát nhanh (đắt, ngay) vs bưu điện thường (rẻ 50%, chậm, không hẹn giờ chính xác).
- **`custom_id`** = mã vận đơn trên từng kiện hàng — kiện nào thất lạc thì chỉ gửi lại đúng kiện đó.
- **Lập kế hoạch SLA** = tính ngược thời gian gửi thư để kịp deadline.
- **Review đa instance** = không tự chấm bài của chính mình; đa lượt = đánh dấu bài nào chưa chắc để gửi giáo viên khác chấm phúc tra.

---

## NGÀY 5 — Quản lý Context & Độ tin cậy (Lĩnh vực 5 — 15%)

### Chương 9 — Escalation & Human-in-the-Loop

**5 tác nhân escalation đáng tin cậy**: (1) khách yêu cầu rõ ràng gặp người thật → escalate ngay; (2) chính sách không bao quát (im lặng, không phải cấm) → escalate; (3) agent không tiến triển sau vài lần thử → escalate; (4) thao tác tài chính vượt ngưỡng → escalate, tốt nhất qua **hook**; (5) nhiều khách hàng khớp → hỏi thêm định danh, không đoán.

⚠️ **3 thứ KHÔNG đáng tin cậy**: phân tích cảm xúc (tâm trạng ≠ độ phức tạp), model tự chấm điểm tự tin (có thể sai rất tự tin, hiệu chỉnh kém), bộ phân loại tự động riêng (overengineering).

**4 mẫu escalation**: ngay lập tức (yêu cầu rõ ràng) / sau khi cố giải quyết (trong phạm vi agent) / **tinh tế** (ghi nhận cảm xúc → đề xuất giải pháp → chỉ escalate nếu khách NHẮC LẠI yêu cầu gặp người — không escalate ngay từ lời than phiền đầu tiên) / vì lỗ hổng chính sách.

**Structured handoff**: bản tóm tắt phải **tự chứa hoàn toàn** — người vận hành không có quyền xem lại toàn bộ hội thoại, chỉ thấy đúng tờ tóm tắt (customer_id, order_id, root_cause, actions_taken, recommended_action, escalation_reason).

**Hiệu chỉnh độ tin cậy**: điểm tin cậy **cấp trường** (không phải cấp toàn văn bản); tin cậy cao/ổn định → tự động, thấp/nguồn mơ hồ → người review. ⚠️ **Lấy mẫu ngẫu nhiên phân tầng**: độ chính xác tổng 97% có thể che giấu 40% lỗi ở một loại tài liệu cụ thể — phải kiểm tra riêng theo loại tài liệu/trường.

### Chương 10 — Xử lý lỗi đa agent

**4 nhóm lỗi**: Tạm thời (timeout, retry+backoff) / Validation (sai input, sửa rồi retry) / Nghiệp vụ (vi phạm chính sách, không retry, giải thích) / Quyền (từ chối truy cập, escalate).

**4 anti-pattern**: trạng thái chung chung ("search unavailable") / ém lỗi âm thầm (rỗng = thành công) / hủy toàn bộ workflow vì 1 lỗi / retry vô hạn trong subagent.

✅ Lỗi có cấu trúc: `status`, `failure_type`, `partial_results`, `alternative_approaches`, `coverage_impact` → coordinator đủ thông tin quyết định. Chú thích độ bao phủ trong báo cáo cuối ("BAO PHỦ ĐẦY ĐỦ" vs "BAO PHỦ MỘT PHẦN — lý do").

### Chương 11 — Quản lý Context Production

| Kỹ thuật | Giải quyết |
|---|---|
| Case facts block riêng | Tóm tắt lũy tiến làm mất số liệu |
| Cắt gọn tool result (`PostToolUse`) | Tích lũy tool result thừa |
| Đầu vào nhận biết vị trí | Lost-in-the-middle |
| File scratchpad | Bảo toàn phát hiện qua ranh giới context/session |
| Ủy quyền subagent | Bảo vệ context agent chính |

⚠️ **Lớp context riêng biệt**: mỗi subagent có ngân sách context giới hạn; coordinator ngăn "rò rỉ context" giữa các agent. Lưu trạng thái có cấu trúc (`agent-state/*.json` + `manifest.json`) → phục hồi sau sự cố.

### Chương 12 — Bảo toàn Provenance

- Mất quy kết nguồn khi tóm tắt → luôn giữ `claim` + `source_url` + `source_name` + `publication_date` + `confidence`.
- Dữ liệu xung đột → giữ cả hai giá trị kèm quy kết, đánh dấu `conflict_detected`, để coordinator đối soát — không tự ý chọn 1 giá trị.
- Thiếu ngày tháng → khác biệt thời gian dễ bị hiểu nhầm mâu thuẫn.
- Trình bày theo loại nội dung: tài chính→bảng, tin tức→văn xuôi, kỹ thuật→danh sách, chuỗi thời gian→theo trình tự.

### Trắc nghiệm Ngày 5 (đã làm)

1. Khách than phiền lần đầu (chưa đòi gặp người) → ghi nhận + đề xuất giải pháp; chỉ escalate nếu khách nhắc lại.
2. Độ chính xác tổng 97% → vẫn cần lấy mẫu ngẫu nhiên phân tầng theo loại tài liệu/trường.
3. Kết quả tìm kiếm rỗng → phân biệt rõ "không có kết quả" (hợp lệ) vs "tìm kiếm thất bại" (lỗi).
4. Xung đột dữ liệu không có ngày tháng → có thể là khác biệt thời gian bị hiểu nhầm mâu thuẫn.
5. Lỗi "search unavailable" chung chung → coordinator không đủ thông tin quyết định retry/dùng kết quả một phần.
6. Điều tra codebase dài gây câu trả lời không ổn định → dùng scratchpad + ủy quyền subagent Explore.

### Giải thích dễ hiểu — ẩn dụ Ngày 5

- **Escalation** = lễ tân bệnh viện có luật rõ ràng khi nào gọi bác sĩ; escalation tinh tế = ghi nhận → đề xuất → chỉ chuyển nếu bệnh nhân khăng khăng.
- **Structured handoff** = tờ bệnh án đầy đủ bàn giao bác sĩ mới, vì bác sĩ mới chưa từng gặp bệnh nhân.
- **Hiệu chỉnh độ tin cậy** = máy dò kim loại sân bay, hiệu chỉnh độ nhạy bằng vật mẫu đã biết.
- **Lấy mẫu phân tầng** = kiểm tra riêng từng dây chuyền sản xuất, không chỉ nhìn tỷ lệ đạt tổng.
- **4 nhóm lỗi** = 4 lý do giao hàng thất bại (kẹt xe / sai địa chỉ / hàng cấm / thiếu giấy phép).
- **Case facts block** = bảng thông tin đầu giường bệnh nhân, không đổi qua các ca trực.
- **Cắt gọn tool result** = thư ký lọc báo cáo 40 trang còn 5 số liệu cần thiết.
- **Scratchpad** = sổ tay điều tra của thám tử.
- **Ủy quyền subagent** = cử thực tập sinh đọc kho tài liệu, chỉ báo lại 1 câu tóm tắt.
- **Provenance** = nguyên tắc trích dẫn báo chí, luôn ghi rõ nguồn/ngày/độ tin cậy.
- **Dữ liệu xung đột** = giữ cả 2 lời khai nhân chứng, để thẩm phán (coordinator) đối chiếu.

---

## NGÀY 6 — Ôn tổng hợp: 12 câu hỏi mẫu chính thức

*(12 câu hỏi đầy đủ + đáp án nằm trong lịch sử hội thoại ngày ôn — tự làm lại nếu cần, dưới đây là các pattern rút ra.)*

**5 pattern lặp lại giúp đoán đúng hướng khi gặp câu lạ:**

1. "Quy tắc nghiệp vụ quan trọng/thứ tự bắt buộc" → luôn chọn **code/hook**, không bao giờ chọn "cải thiện prompt".
2. "Chọn sai tool giữa các tool giống nhau" → luôn chọn **sửa mô tả tool** trước, không nhảy lên giải pháp phức tạp (routing layer, bộ phân loại riêng).
3. "Model tự chấm điểm tự tin" hoặc "phân tích cảm xúc" → luôn là đáp án **SAI** khi liên quan escalation/calibration.
4. Khi coordinator/subagent lỗi → hỏi "vấn đề ở khâu **phân công** hay khâu **thực thi**?" trước khi đổ lỗi subagent.
5. Đáp án đúng thường là phương án **ít cực đoan nhất** — giữ nguyên phần đang ổn, chỉ sửa đúng phần lỗi (không "cấp toàn quyền", không "chuyển hết sang 1 API", không "hủy hết").

**Các chủ đề KHÔNG xuất hiện trong đề thi** (để không ôn lệch): fine-tuning/huấn luyện model tùy chỉnh · xác thực API/thanh toán/tài khoản · chi tiết ngôn ngữ lập trình/framework · triển khai/hosting MCP server (hạ tầng, mạng, container) · kiến trúc nội bộ Claude/training/model weight · Constitutional AI/RLHF · embedding/vector database · Computer use · Vision · Streaming API/SSE · rate limiting/chi phí API chi tiết · OAuth/xoay API key · cấu hình riêng theo cloud · benchmark hiệu năng model · chi tiết prompt caching · thuật toán tokenization.

---

## NGÀY 7 (chưa học) — Thi thử toàn phần

- [ ] 76 câu bài luyện tập (L2120–3405), tính giờ như thi thật
- [ ] Rà soát câu sai, quay lại đúng chương liên quan
- [ ] Ôn nhanh 8 kịch bản, đặc biệt 3 kịch bản không có câu hỏi riêng

---

*Cập nhật lần cuối: sau khi hoàn thành Ngày 3, 4, 5, 6 (kèm bản giải thích dễ hiểu bằng ẩn dụ) và 12 câu hỏi mẫu chính thức. Còn lại: Ngày 7 — thi thử toàn phần 76 câu.*
