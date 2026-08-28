# Tổng hợp ôn thi — Claude Certified Architect

> Trang tra cứu tổng hợp toàn bộ 7 ngày ôn tập, kèm chẩn đoán từ bài thi thử 76 câu. Xem [`01-nhat-ky-on-tap.md`](./01-nhat-ky-on-tap.md) và [`03-chan-doan-thi-thu.md`](./03-chan-doan-thi-thu.md) để có đầy đủ chi tiết/ví dụ/ẩn dụ; file này là bản cô đọng để tra cứu nhanh.

---

## Bối cảnh thi

| | |
|---|---|
| 5 lĩnh vực | Kiến trúc/điều phối agent (**27%**) · Tool/MCP (**18%**) · Claude Code (**20%**) · Prompt/Structured output (**20%**) · Context/Độ tin cậy (**15%**) |
| Định dạng | Trắc nghiệm 1/4 đáp án · 4 trong 8 kịch bản (ngẫu nhiên) · không phạt đoán |
| Thang điểm | 100–1000, đạt **720** |
| Kết quả thi thử | **52/76 (68.4%)** — xem chi tiết bên dưới |

⚠️ Mẹo cốt lõi: câu hỏi kiểm tra **trade-off kiến trúc trong tình huống thực tế**, không phải học thuộc định nghĩa. Đáp án đúng thường là phương án **ít cực đoan nhất**, sửa đúng phần lỗi, giữ nguyên phần đang ổn.

---

## Lĩnh vực 1 — Kiến trúc & Điều phối Agent (27%, nặng nhất)

**Agentic Loop**: gửi request → nhận response → kiểm tra `stop_reason`: `"tool_use"` → thực thi tool, nối kết quả, lặp lại; `"end_turn"` → xong. Model-driven, không phải cây quyết định cứng.
- 3 anti-pattern: parse text tìm "Task completed" · dùng `max_iterations` tùy ý làm điều kiện dừng chính · coi "có sinh text" là dấu hiệu xong. Tín hiệu duy nhất đáng tin: `stop_reason == "end_turn"`.

**Hub-and-Spoke**: `AgentDefinition` gồm name, description, system_prompt, allowed_tools (least privilege). Coordinator: phân rã → chọn subagent động → ủy thác → tổng hợp/validate → xử lý lỗi → giao tiếp người dùng.
- 🔑 Nguyên tắc quan trọng nhất cả lĩnh vực: subagent có **context tách biệt hoàn toàn** — không kế thừa lịch sử coordinator. Mọi thứ phải truyền tường minh vào prompt `Task`. Nhiều `Task` trong cùng 1 response → chạy song song. `allowedTools` coordinator phải có `"Task"`.

**Hooks vs Chỉ dẫn trong Prompt:**

| | Hook | Chỉ dẫn trong prompt |
|---|---|---|
| Đảm bảo | Deterministic (100%) | Xác suất (>90%, không phải 100%) |
| Dùng khi | Quy tắc tài chính/pháp lý/an toàn | Khuyến nghị chung, định dạng |

`PostToolUse`: chuẩn hóa kết quả tool trước khi model thấy. `PreToolUse`: chặn hành động trước khi xảy ra.

**Chiến lược phân rã tác vụ:**

| Chiến lược | Khi dùng |
|---|---|
| Pipeline cố định | Cấu trúc dự đoán được, cần ổn định |
| Phân rã thích ứng động | Điều tra mở, không biết trước phạm vi |
| Multi-pass review | PR 10+ file — lượt 1 từng file riêng, lượt 2 phân tích quan hệ chéo file |

**Session/Resume/Fork**: `--resume` tiếp tục session đã đặt tên (báo agent nếu file đã đổi). `fork_session` tạo nhánh độc lập từ context chung, so sánh song song. Context còn hiện hành → resume; kết quả cũ lỗi thời → session mới kèm tóm tắt có cấu trúc.

---

## Lĩnh vực 2 (18%) & 3 (20%) — Tool Design, MCP & Claude Code

**MCP**: 3 loại tài nguyên — Tools (hành động), Resources (dữ liệu đọc), Prompts (mẫu định sẵn). `.mcp.json` (dự án, qua VCS, secret bằng biến môi trường `$GITHUB_TOKEN`) vs `~/.claude.json` (cá nhân).
- ⚠️ Cờ `isError`: lỗi có cấu trúc (`errorCategory`, `isRetryable`, message) cho agent đủ thông tin quyết định; lỗi chung chung thì không.

**Mô tả Tool** là cơ chế lựa chọn chính. Mô tả tốt: tool làm gì/trả gì, định dạng input, edge case, khi nào dùng thay vì tool tương tự. Cách sửa rẻ nhất khi định tuyến sai: đổi tên + viết lại mô tả — không phải few-shot hay routing layer.

**Tool dựng sẵn**: Glob (tìm file) · Grep (tìm nội dung) · Read/Write (đọc/ghi toàn bộ) · Edit (sửa 1 đoạn khớp duy nhất) · Bash (lệnh shell). Điều tra tăng dần: Grep entry point → Read → Grep nơi dùng → Read tiếp. Edit thất bại → fallback Read + sửa + Write.

**Claude Code:**
- 3 cấp CLAUDE.md: người dùng (không qua VCS) / dự án (qua VCS, cả nhóm) / thư mục. ⚠️ Bẫy kinh điển: chỉ dẫn dự án đặt nhầm ở mức người dùng → thành viên mới không nhận được.
- `@path`: import file khác, độ sâu lồng tối đa **5**.
- `.claude/rules/` + `paths`: nạp **tự động theo loại file** khớp glob, bất kể đang làm tác vụ gì.
- **Skill**: nạp **theo nhu cầu** khi gọi slash command, gắn với một quy trình cụ thể (sinh endpoint mới, review PR, migration). `context: fork` = subagent tách biệt. `allowed-tools` = giới hạn tool.

**Planning mode** (thay đổi lớn, nhiều phương án, codebase lạ) vs **thực thi trực tiếp** (sửa lỗi rõ ràng). Subagent **Explore** tách output khám phá dài khỏi context chính.

**CI/CD**: `-p`/`--print` bắt buộc cho non-interactive. `--output-format json` + `--json-schema` cho structured output đáng tin cậy. ⚠️ Cô lập context session: session vừa sinh code thì kém khách quan khi tự review chính nó. Review lại sau commit mới: đưa kết quả review trước đó vào context, chỉ báo cáo vấn đề mới/chưa sửa.

---

## Lĩnh vực 4 — Prompt Engineering & Structured Output (20%)

**Few-shot**: 2–4 ví dụ input/output hiệu quả hơn mô tả bằng lời. 5 loại: kịch bản nhập nhằng, định dạng output, code chấp nhận được/có vấn đề, trích xuất đa định dạng, đo lường phi chuẩn.

**Tiêu chí rõ ràng vs mơ hồ**: liệt kê điều kiện NÊN/KHÔNG NÊN gắn cờ cụ thể, thay vì tính từ mơ hồ. False positive cao ở 1 hạng mục → xói mòn niềm tin cả các hạng mục đúng khác.

**Validation & Retry-with-feedback:**

| Retry CÓ tác dụng | Retry KHÔNG tác dụng |
|---|---|
| Sai định dạng/cấu trúc | Thông tin không có trong tài liệu nguồn |
| Không nhất quán số học (tự kiểm tra được) | Cần ngữ cảnh từ tài liệu khác không được cung cấp |

Retry đúng cách: kèm tài liệu gốc + bản sai + lỗi cụ thể. Tự sửa lỗi: trích cả `stated_total` và `calculated_total`, gắn cờ `conflict_detected`.

**Message Batches API**: tiết kiệm 50%, cửa sổ tới 24h (không SLA latency), **không hỗ trợ tool nhiều lượt** (không chỉ chậm mà không tương thích kỹ thuật với tool-calling nhiều vòng), `custom_id` để tương quan. Batch cho báo cáo qua đêm/audit; Synchronous cho kiểm tra chặn trước merge.

**Review đa instance & đa lượt**: model tự review code mình viết → khó tự thách thức quyết định của chính mình → cần instance độc lập. Review đa lượt: từng file riêng + 1 lượt tích hợp.

---

## Lĩnh vực 5 — Quản lý Context & Độ tin cậy (15%)

**Escalation**: 5 tác nhân đáng tin cậy — yêu cầu rõ ràng gặp người · chính sách không bao quát (im lặng) · agent không tiến triển · thao tác tài chính vượt ngưỡng (qua hook) · nhiều khách khớp (hỏi thêm định danh).
- ⚠️ KHÔNG đáng tin cậy: phân tích cảm xúc, model tự chấm điểm tự tin, bộ phân loại tự động riêng.
- 4 mẫu: ngay lập tức / sau khi cố giải quyết / **tinh tế** (ghi nhận → đề xuất → chỉ escalate nếu khách NHẮC LẠI) / vì lỗ hổng chính sách.
- **Structured handoff**: bản tóm tắt phải tự chứa hoàn toàn. **Hiệu chỉnh độ tin cậy**: điểm tin cậy cấp trường; lấy mẫu ngẫu nhiên phân tầng vì độ chính xác tổng có thể che giấu lỗi cục bộ.

**Xử lý lỗi đa agent:**

| Nhóm lỗi | Retry? | Hành động |
|---|---|---|
| Tạm thời | Có | Retry + exponential backoff |
| Validation | Không | Sửa input rồi retry |
| Nghiệp vụ | Không | Giải thích, đề xuất thay thế |
| Quyền | Không | Escalate |

4 anti-pattern: trạng thái chung chung / ém lỗi âm thầm / hủy toàn bộ workflow vì 1 lỗi / retry vô hạn trong subagent.

**Quản lý Context Production**: case facts block (chống tóm tắt lũy tiến làm mất số liệu) · cắt gọn tool result qua `PostToolUse` · đầu vào nhận biết vị trí (chống lost-in-the-middle) · file scratchpad · ủy quyền subagent Explore.

**Bảo toàn Provenance**: luôn giữ `claim + source_url + source_name + publication_date + confidence`. Dữ liệu xung đột → giữ cả hai giá trị, đánh dấu `conflict_detected`, để coordinator đối soát. Thiếu ngày tháng → khác biệt thời gian dễ hiểu nhầm mâu thuẫn.

---

## Kết quả thi thử (76 câu)

**Tổng: 52/76 (68.4%)** — dưới ngưỡng tham chiếu 72% (≈720/1000 đề thật).

| Kịch bản | Điểm |
|---|---|
| Claude Code cho CI/CD | 🔴 7/15 (47%) — yếu nhất |
| Hệ thống Nghiên cứu Đa tác nhân | 🟡 10/15 (67%) |
| Customer Support Agent | 🟡 10/15 (67%) |
| Sinh mã với Claude Code | 🟢 11/15 (73%) |
| Mẫu kiến trúc AI hội thoại | 🟢 14/16 (88%) |

Tin tốt: CI/CD (yếu nhất) và phần lớn lỗi khác rơi vào Lĩnh vực 1 (27%) và Lĩnh vực 3 (20%) — 2 lĩnh vực nặng điểm nhất.

---

## 5 nhóm lỗi lặp lại — quan trọng hơn từng câu riêng lẻ

**1. Vẫn chọn "sửa bằng prompt" thay vì "cơ chế cứng"** (Câu 10, 16, 17, 51)
Quy tắc 2 bước: (1) Hậu quả nếu model quên/hiểu sai — tiền bạc/pháp lý/an toàn → PHẢI cơ chế cứng (hook, precondition, cờ CLI, tool validation, instance tách biệt); chỉ phong cách/định dạng → prompt/few-shot đủ. (2) Cơ chế cứng cụ thể: business rule → `PreToolUse` hook · structured output → `tool_use`+schema/`--json-schema` · tool lạm dụng → tool chuyên biệt hẹp hơn · tự review → instance độc lập. ⚠️ Chiều ngược: đừng áp dụng hook cho mọi thứ — hậu quả nhỏ thì prompt/few-shot vẫn đúng.

**2. Nhầm khi nào dùng Few-shot** (Câu 7, 20, 47, 52) — sai cả 2 chiều
Quy tắc 3 bước: (1) Tool sai vì tên/mô tả chồng lấn? → sửa interface trước, KHÔNG few-shot. (2) Mô tả đã ổn, chỉ lúng túng 1 mẫu hành vi lặp lại minh họa được? → few-shot đúng, đừng xây hạ tầng thừa. (3) Khoảng trống thay đổi ngẫu nhiên theo case? → cần self-critique động, không phải ví dụ tĩnh.

**3. Nhầm `.claude/rules/` (theo path) với Skill (theo nhu cầu)** (Câu 33, 38 — sai giống hệt 2 lần)
Tự vấn: "Áp dụng vì chạm MỘT LOẠI FILE, hay vì làm MỘT TÁC VỤ cụ thể?" Rules/paths → tự động theo loại file. Skill → chỉ khi gọi theo nhu cầu, gắn 1 quy trình cụ thể.

**4. Escalation: "bằng chứng mâu thuẫn lời khách" vs "chính sách im lặng"** (Câu 50)
Chính sách CÓ đề cập + có bằng chứng rõ ràng → agent tự tin trình bày, KHÔNG escalate. Chính sách THỰC SỰ im lặng → escalate vì agent không có thẩm quyền tự đặt luật mới.

**5. 2 kỹ thuật ngoài phạm vi 13 chương lý thuyết** (Câu 68, 72 — chỉ cần nhớ trực tiếp)
Hội thoại dài hàng tháng, tra lại kết luận cụ thể → **semantic search/embedding**, không phải tóm tắt lũy tiến. Loại bỏ lời chào lặp → **prefill phần đầu tin nhắn assistant**, không phải dặn prompt/hạ temperature.

---

## Việc cần làm trước khi thi thật

1. Đọc lại Chương 5.9 (ngăn bình luận trùng lặp CI) và Chương 7 (Batch API không tương thích tool-calling) — ưu tiên cao nhất, CI/CD đang yếu nhất (47%).
2. Làm lại tự kiểm tra Ngày 2 & 3, tự hỏi mỗi câu: "đáp án nào là code/cờ CLI/tool riêng thay vì chỉ dặn prompt?"
3. Đọc lại mục 3.2–3.3 Phần II, tự hỏi "theo loại file hay theo tác vụ?" trước khi chọn rules/paths hay Skill.
4. Ôn lại đúng đoạn phân biệt "chính sách im lặng" vs "bằng chứng mâu thuẫn" trong Chương 9.1.
5. Ghi nhớ trực tiếp 2 fact ngoài phạm vi (Câu 68, 72).

---

*Nguồn: `../guide_vi.md` · `01-nhat-ky-on-tap.md` · `03-chan-doan-thi-thu.md` — bản artifact trực quan: https://claude.ai/code/artifact/936b4b5e-5444-4745-aaa3-75247365db31*

*Kết quả bài thi thử kèm giải thích từng câu: https://claude.ai/code/artifact/5c357286-5fa6-46c9-b2c0-7dec3c636bdd*
