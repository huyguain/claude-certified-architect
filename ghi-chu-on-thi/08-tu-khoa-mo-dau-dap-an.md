# Bảng từ khóa mở đầu đáp án — chiến lược đoán không cần đọc hiểu hết câu

> Xây dựng bằng cách phân tích **có hệ thống cả 76 câu** (không đoán): parse toàn bộ 304 đáp án (76 câu × 4 lựa chọn), gắn nhãn nhóm cơ chế cho đáp án đúng của từng câu, rồi đếm tần suất. Số liệu dưới đây là **dữ liệu thật của bộ đề**, không phải cảm tính.
>
> Dùng cùng [`05-chien-thuat-lam-bai.md`](./05-chien-thuat-lam-bai.md) (chiến thuật loại đáp án) và [`07-tu-noi-va-khuon-cau.md`](./07-tu-noi-va-khuon-cau.md) (khuôn câu). File này thêm lớp thứ 3: **nhận diện qua vài từ mở đầu của đáp án**, gần như không cần đọc hiểu ngữ pháp.

---

## Phát hiện quan trọng nhất: "Hạ tầng mới" gần như luôn là bẫy

Quét toàn bộ 304 đáp án tìm các từ: `classifier`, `routing layer`, `separate model`, `dedicated model/agent`, `vector database`, `fine-tune`, `trained on historical...`, `preprocessing layer/classifier`.

**Kết quả: xuất hiện ở 10 câu — và SAI cả 10/10 lần (100% là bẫy).**

| Câu | Đáp án chứa từ khóa "hạ tầng mới" | Đúng/Sai |
|---|---|---|
| Q3 | "Create a **dedicated** error-handling agent..." | Sai |
| Q7 | "Add a pre-routing **classifier**..." | Sai |
| Q14 | "Store findings in a **vector database**..." | Sai |
| Q46 | "Implement a preprocessing **classifier**..." | Sai |
| Q47 | "...uses a **separate model** call to decompose..." | Sai |
| Q49 | "Deploy a **separate classifier model** trained on historical tickets..." | Sai |
| Q51 | "Implement a routing **classifier**..." | Sai |
| Q56 | "...should be **fine-tuned** on examples..." | Sai |
| Q57 | "Implement a **routing layer**..." | Sai |
| Q64 | "Claude requires a **vector database** connection..." | Sai |

**→ Chiến lược:** Chỉ cần nhận mặt các từ tiếng Anh `classifier`, `routing layer`, `separate/dedicated model`, `vector database`, `fine-tune` xuất hiện trong 1 đáp án → **loại ngay gần như không cần đọc gì thêm**, không cần hiểu cả câu. Đây là cách "không cần đọc hiểu hết tiếng Anh" hiệu quả nhất trong toàn bộ đề.

**Ngoại lệ duy nhất (1/76 câu):** Q68 — hội thoại dài **3 tháng**, câu hỏi cần tra lại 1 kết luận cũ → đáp án đúng là "**Semantic embeddings** with retrieval of relevant exchanges" (không chứa các từ khóa bị cấm ở trên, nhưng vẫn là kỹ thuật "hạ tầng"). Dấu hiệu để nhận ra ngoại lệ: tình huống nói tới **hàng tháng/rất nhiều lần hội thoại**, không phải 1 tác vụ đơn lẻ.

---

## Bảng 7 nhóm cơ chế + tần suất đúng thực tế (trên 76 câu)

| Nhóm | Ký hiệu | Từ khóa mở đầu tiếng Anh | Số lần ĐÚNG /76 | Khi nào nhóm này thường đúng |
|---|---|---|---|---|
| **Chọn đúng vị trí/chế độ** | P | in the project/user-level, planning mode, .claude/rules, Skill, synchronous/batch | **14 (18%)** | Câu hỏi "where should...", "which mode", sync vs batch, CLAUDE.md vs Skill vs rules |
| **Khác (phát biểu nguyên nhân/factual)** | O | (không phải "fix", mà là câu khẳng định sự thật) | **14 (18%)** | Câu hỏi "What is the root cause / most likely cause?", "What determines...?" |
| **Cơ chế cứng (code enforcement)** | C | precondition, validate, schema, hook, token, block until, PostToolUse, self-critique stage | **13 (17%)** | Hậu quả nghiêm trọng (tiền/pháp lý/bảo mật) hoặc lỗi hệ thống lặp lại |
| **Mềm (prompt/few-shot)** | M | add instructions, few-shot examples, system prompt, state assumptions explicitly | **13 (17%)** | Vấn đề là *định dạng/phong cách*, hệ thống đã tốt chỉ cần ví dụ minh họa |
| **Tái cấu trúc dữ liệu/luồng** | D | structure the output, extract structured data, decompose, distinguish, hybrid approach | **10 (13%)** | Vấn đề là *cách tổ chức thông tin* (thứ tự, gộp/tách), không phải thiếu cơ chế |
| **Escalate / hỏi thêm** | E | escalate, ask the user, request an additional identifier, surface the contradiction | **6 (8%)** | Chính sách thật sự thiếu, hoặc 2 yêu cầu người dùng mâu thuẫn nhau |
| **Sửa mô tả/giao diện tool** | T | rename the tool, expand the tool's description, check the tool descriptions | **5 (7%)** | Lỗi routing do mô tả tool mơ hồ/chồng lấn |
| **Hạ tầng mới** | H | classifier, routing layer, vector database, fine-tune | **1 (1%)** | Gần như luôn SAI — chỉ đúng khi quy mô cực lớn/kéo dài nhiều tháng |

**Quan sát quan trọng để tránh lệch bù (overcorrection):** Nhóm **M (mềm)** vẫn đúng tới 17% — đừng nghĩ "cứ có few-shot/instructions là sai". Nó SAI khi hậu quả nghiêm trọng hoặc lỗi có hệ thống, nhưng ĐÚNG khi vấn đề chỉ là thiếu ví dụ minh họa cho định dạng/lý luận. Muốn phân biệt 2 trường hợp này vẫn cần đọc lướt tình huống (bảng "tự vấn" ở file 05), từ khóa mở đầu chỉ giúp **loại nhanh nhóm H**, không thay thế hoàn toàn việc đọc.

---

## Quy trình đoán nhanh khi không đọc hiểu hết câu

1. Đọc 4 đáp án, **chỉ cần 3–5 từ đầu tiên** mỗi đáp án (đủ để xác định nhóm cơ chế theo bảng trên).
2. **Loại ngay** đáp án nào rơi vào nhóm **H** (classifier/routing layer/vector database/fine-tune) — trừ khi tình huống có chữ "months"/số lần hội thoại rất lớn.
3. Với các đáp án còn lại, nếu 1 đáp án thuộc nhóm **T** (sửa mô tả tool) và tình huống có nhắc đến việc **định tuyến/chọn nhầm tool** (routing, misrouting, wrong tool) → ưu tiên T.
4. Nếu tình huống có số liệu %/lặp lại nhiều lần + hậu quả cụ thể (tiền, tài khoản sai, dữ liệu bị xóa) → ưu tiên nhóm **C** (cứng).
5. Nếu không có tín hiệu hậu quả nghiêm trọng, hệ thống đã hoạt động tốt phần lớn, chỉ cần cải thiện 1 mẫu cụ thể → nhóm **M** có thể đúng.
6. Nếu câu hỏi không phải "cách sửa" mà là "vị trí đặt cấu hình" hoặc "chế độ nào" → chuyển hẳn sang nhóm **P**, không áp bảng cứng/mềm.
7. Nếu câu hỏi là "root cause" / "what determines" → đây là nhóm **O**, các đáp án là *câu khẳng định*, không phải *hành động sửa* — so khớp trực tiếp với dữ kiện đã nêu trong tình huống, không áp bảng cơ chế.

---

## Bảng tra cứu 76 câu (nhóm đáp án đúng + từ khóa nhận diện)

### Multi-agent Research System

| Câu | Nhóm | Đáp án đúng bắt đầu bằng... |
|---|---|---|
| Q1 | E | "Complete analysis with both numbers, explicitly annotate..." |
| Q2 | P | "The coordinator passes both sets of results..." |
| Q3 | E | "Implement local recovery in the subagent for..." |
| Q4 | O | "The coordinator's task decomposition is too narrow..." |
| Q5 | D | "Structure the synthesis output with coverage annotations..." |
| Q6 | E | "Return an error with context to the..." |
| Q7 | T | "Rename the web-search tool to extract_web_results..." |
| Q8 | O | "The coordinator can observe all interactions..." |
| Q9 | D | "Return structured error context to the coordinator..." |
| Q10 | T | "Replace fetch_url with a load_document tool..." |
| Q11 | P | "The coordinator explicitly partitions the research space..." |
| Q12 | D | "Distinguish access failures (timeout) that require..." |
| Q13 | D | "Place a key-findings summary at the start..." |
| Q14 | D | "Modify upstream agents to return structured data..." |
| Q15 | T | "Give the synthesis agent a limited-scope verify_fact..." |

### Claude Code for Continuous Integration

| Câu | Nhóm | Đáp án đúng bắt đầu bằng... |
|---|---|---|
| Q16 | C | "Use the CLI flags --output-format json..." |
| Q17 | O | "Run a second independent instance of Claude..." |
| Q18 | O | "The asynchronous model cannot execute tools mid-request..." |
| Q19 | P | "Use synchronous calls for PR style checks..." |
| Q20 | M | "Add 3–4 few-shot examples showing the exact..." |
| Q21 | P | "Only the deep analysis." |
| Q22 | M | "Specify explicit criteria: flag comments only when..." |
| Q23 | O | "Temporarily disable high-false-positive categories..." |
| Q24 | O | "Include the existing test file in context..." |
| Q25 | M | "Include previous review findings in context and..." |
| Q26 | O | "Add the -p flag: claude -p ..." |
| Q27 | D | "Split into focused passes: review each file..." |
| Q28 | M | "Require Claude to include its rationale and..." |
| Q29 | O | "Temporarily disable high-false-positive categories..." |
| Q30 | P | "Use batch processing only for technical debt..." |

### Code Generation with Claude Code

| Câu | Nhóm | Đáp án đúng bắt đầu bằng... |
|---|---|---|
| Q31 | M | "Provide 2–3 concrete input-output examples..." |
| Q32 | P | "Switch to planning mode to explore integration..." |
| Q33 | P | "Keep universal standards in CLAUDE.md and create..." |
| Q34 | P | "Switch to planning mode to explore the..." |
| Q35 | C | "Add context: fork in the skill frontmatter..." |
| Q36 | P | "Create a personal version at ~/.claude/skills/commit/SKILL.md..." |
| Q37 | P | "The guidance lives in the original developers'..." |
| Q38 | P | "Create a skill that references the endpoint..." |
| Q39 | C | "Add argument-hint in frontmatter... context: fork... allowed-tools..." |
| Q40 | P | "Create rule files under .claude/rules/ with YAML..." |
| Q41 | P | "In the project repository under .claude/commands/." |
| Q42 | P | "Create separate Markdown files in .claude/rules/..." |
| Q43 | C | "Add context: fork in the skill frontmatter." |
| Q44 | C | "Add the server to the project .mcp.json..." |
| Q45 | C | "Use an Explore subagent for Phase 1..." |

### Customer Support Agent

| Câu | Nhóm | Đáp án đúng bắt đầu bằng... |
|---|---|---|
| Q46 | T | "Check the tool descriptions to ensure they..." |
| Q47 | M | "Add few-shot examples to the prompt demonstrating..." |
| Q48 | D | "Decompose the request into separate issues..." |
| Q49 | M | "Add explicit escalation criteria to the system..." |
| Q50 | E | "A customer requests competitor price matching..." |
| Q51 | C | "Add a programmatic precondition that blocks lookup_order..." |
| Q52 | C | "Add a self-critique stage where the agent..." |
| Q53 | M | "Instruct Claude in the prompt to bundle..." |
| Q54 | D | "Extract transactional facts (amounts, dates, order numbers)..." |
| Q55 | E | "Instruct Claude to request an additional identifier..." |
| Q56 | O | "The system prompt contains keyword-sensitive instructions..." |
| Q57 | T | "Expand each tool's description to include input..." |
| Q58 | O | "Check the stop_reason field in Claude's response..." |
| Q59 | C | "Use a PostToolUse hook to intercept tool..." |
| Q60 | M | "Add 4–6 examples targeted at ambiguous scenarios..." |

### Conversational AI Architecture Patterns

| Câu | Nhóm | Đáp án đúng bắt đầu bằng... |
|---|---|---|
| Q61 | C | "Replace with two tools: preview_remove_member returns impact..." |
| Q62 | C | "Implement automatic retry with backoff for network..." |
| Q63 | E | "Surface the contradiction and ask the user..." |
| Q64 | O | "Your application isn't including prior messages in..." |
| Q65 | D | "Extract critical structured data (allergies, quantities...)..." |
| Q66 | D | "Hybrid approach: summarize older messages while keeping..." |
| Q67 | O | "The entire conversation history is included with..." |
| Q68 | H | "Semantic embeddings with retrieval of relevant exchanges." |
| Q69 | M | "Insert user-role messages reinforcing guidelines at conversation..." |
| Q70 | M | "Replace verbose rules with few-shot examples demonstrating..." |
| Q71 | O | "In the system prompt." |
| Q72 | C | "Append a partial assistant message with a..." |
| Q73 | C | "Append the status update as a prefix..." |
| Q74 | M | "State assumptions explicitly and proceed while inviting..." |
| Q75 | O | "Accumulated assistant responses dilute system prompt influence." |
| Q76 | M | "Make reasonable assumptions, state them explicitly..." |

---

## Lưu ý quan trọng: đây là công cụ hỗ trợ, không phải máy đoán đáp án

Bảng tần suất trên phản ánh **đúng bộ 76 câu này**. Đề thi thật sẽ có câu khác — tỷ lệ % các nhóm nhiều khả năng tương tự (vì cùng 8 chủ đề, cùng logic kiến trúc), nhưng **không đảm bảo giống hệt**. Dùng bảng này để:
- Loại nhanh, tự tin nhóm **H** (gần như miễn phí, rủi ro thấp).
- Thu hẹp còn 2 đáp án bằng nhóm cơ chế, rồi mới đọc kỹ tình huống để chọn giữa 2 đáp án đó theo `05-chien-thuat-lam-bai.md`.
- Không dùng để bỏ qua hoàn toàn việc đọc — chỉ dùng để đọc **nhanh hơn và tự tin hơn**.
