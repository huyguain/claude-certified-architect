# Chẩn đoán bài thi thật — CCAR-F, 17/09/2026

> **Điểm: 696 / cần 720 → Trượt.** Thiếu **24 điểm ≈ 3,3% thang điểm**.
> Score report liệt kê **29 test objective** với % đúng từng mục.
> Phân biệt với [`03-chan-doan-thi-thu.md`](./03-chan-doan-thi-thu.md) (bài thi thử 76 câu) — đây là **đề thật**.

---

## Kết luận một dòng

Khoảng cách rất nhỏ. Với 29 objective, **24 điểm nhiều khả năng chỉ tương đương 2–3 câu**. Vá đúng **2 trong 5 mục 0%** gần như chắc chắn đủ qua.

---

## Toàn bộ objective theo điểm

### 0% — năm mục

| # | Objective | Lĩnh vực |
|---|---|---|
| A | Design **orchestration-layer safeguards** ensuring every agent session ends with a completed resolution or human escalation, **regardless of how the agentic loop terminates** | LV1 |
| B | Design **structured handoff packages** that preserve accumulated context, findings, **and authorization state** when transferring control between agent steps or to a human operator | LV1 |
| C | **Distinguish** instructions that must be enforced through **settings permissions or hooks** from those appropriately placed in **CLAUDE.md**, **and restructure** configurations accordingly | LV3 |
| D | Implement **PostToolUse hooks** that automatically enforce **code quality constraints** — formatting, linting, test execution — after every file edit, **independent of model instruction-following** | LV3 |
| E | Configure **`tool_choice`** to guarantee tool invocation when structured output is required, **and sequence multi-tool workflows so prerequisite data is obtained before dependent tools are called** | LV2 |

### 25–67% — năm mục

| % | Objective | Lĩnh vực |
|---|---|---|
| 25% | Apply **extraction accuracy patterns** — schema optional fields + format normalization + few-shot — to reduce hallucination across varied document formats | LV4 |
| 33% | Design **human review routing** based on confidence scores, document characteristics, field-level ambiguity **rather than random sampling** | LV5 |
| 50% | Explain why **conversation history must be explicitly included** in each API request; cơ chế giữ state trong **stateless API** | LV5 |
| 50% | Improve **tool selection reliability** by expanding tool descriptions with use-case examples, input format specs, explicit disambiguation | LV2 |
| 50% | **Integrate MCP servers** — server scope + auth via **env var expansion** + **verifying tool discovery** | LV2 |
| 67% | Design **extraction schemas** with optional fields, nullable values, enum definitions | LV4 |

### 75–83% — ba mục

| % | Objective |
|---|---|
| 75% | Decompose complex tasks into **dynamically generated subtasks** |
| 75% | Structure **iterative refinement** — concrete I/O examples, targeted feedback, batched issues |
| 83% | Apply **session resumption** — targeted re-analysis of changed files + context injection |

### 100% — mười lăm mục

stop_reason & agentic loop · PreToolUse/PostToolUse cho **business rules** · slash command đúng thư mục · plan mode vs direct · chọn cơ chế cấu hình Claude Code · Grep/Glob/Read · MCP server scope · context management (subagent isolation, scratchpad) · `@` references vs CLAUDE.md · escalation decision criteria · batch vs sync API · chọn phương pháp structured output · `tool_use` + JSON schema + `tool_choice` cho structured output · feedback loop mechanisms · MCP tool error handling

---

## Phát hiện quan trọng nhất: năm cặp đối lập

Đây là thứ score report cho thấy rõ mà bài thi thử không cho thấy. **Mỗi mục 0% đều có một mục "anh em" đạt 100%.**

| Đạt 100% | Nhưng 0% | Chênh ở đâu |
|---|---|---|
| Explain how the agentic loop uses **stop_reason** | **A** — safeguards để mọi phiên kết thúc bằng resolution **hoặc** escalation, **bất kể loop kết thúc thế nào** | Biết **cơ chế** vòng lặp, không biết **lớp bảo đảm trạng thái cuối** |
| Apply **escalation decision criteria** (khi nào escalation) | **B** — **gói bàn giao** bảo toàn context, findings, **authorization state** | Biết **KHI NÀO** escalation, không biết **BÀN GIAO CÁI GÌ** |
| **Select** the correct config mechanism (CLAUDE.md / rules / Skills / hooks / settings) | **C** — **phân biệt + TÁI CẤU TRÚC** cấu hình đang sai | Chọn đúng khi được hỏi trực tiếp, nhưng không sửa được cấu hình sai có sẵn |
| Apply PreToolUse/PostToolUse cho **business rules & policy** | **D** — PostToolUse cho **code quality** (format, lint, test) sau mỗi lần sửa file | Biết hook trong **Agent SDK**, không biết hook trong **Claude Code settings** |
| `tool_use` + JSON schema + `tool_choice` cho **structured output** | **E** — `tool_choice` để **SEQUENCE** workflow, lấy dữ liệu tiên quyết trước | Nắm **một nửa** khái niệm `tool_choice` |

**Quy luật chung:** mọi objective 0% đều là **mệnh đề kép** — vế đầu là khái niệm nền (anh đã đúng 100% ở mục riêng của nó), vế sau là phần mở rộng: *bảo đảm đầu-cuối* · *authorization state* · *tái cấu trúc* · *độc lập với model* · *thứ tự phụ thuộc*.

> Anh **không thiếu kiến thức nền**. Anh mất điểm ở **vế thứ hai** — phần áp dụng khái niệm vào trường hợp biên và vào việc sửa hệ thống đang sai.

Cặp **E** đặc biệt đáng chú ý: đây đúng là lỗ hổng Q12 đã nhận diện từ đầu đợt ôn (mô tả tool + `auto` vs forced `tool_choice`). Ghi chú đã phủ, nhưng đề thật ghép nó với **prerequisite sequencing** của LV1 1.4 — và ở dạng ghép này thì trượt hoàn toàn.

---

## Ba chỗ ghi chú hiện tại CHƯA phủ

Rà lại `12`–`16`, đây là những thứ **không có** trong bộ ghi chú:

1. **Lớp bảo đảm trạng thái cuối phiên** (mục A). Ghi chú `16` phủ kỹ cơ chế `stop_reason`, nhưng **không** có phần: phải làm gì khi loop kết thúc bằng `end_turn` mà vấn đề chưa được giải quyết, hoặc `max_tokens` cắt ngang, hoặc hết vòng lặp. Nguyên tắc cần bổ sung: **mọi phiên phải kết thúc ở một trong hai trạng thái — đã giải quyết, hoặc đã chuyển cho người.** Không có trạng thái thứ ba "im lặng dừng".

2. **`authorization state` trong gói bàn giao** (mục B). Ghi chú `16` mục 1.4 và `15` mục 5.2 đều có structured handoff, nhưng cả hai chỉ liệt kê `customer_id / root_cause / actions_taken / recommended_action`. **Thiếu hẳn `authorization state`** — những gì đã được phê duyệt, còn chờ phê duyệt, ai phê duyệt, phạm vi quyền đã cấp. Đây nhiều khả năng chính là chỗ mất điểm.

3. **Hook trong `settings.json` của Claude Code** (mục D). Toàn bộ ghi chú coi hook là cơ chế **Agent SDK** để chặn business rule. Không có phần hook **tự động hóa quy trình dev**: chạy formatter/linter/test sau mỗi `Edit`/`Write`, **độc lập với việc model có nghe lời hay không**. Đây là chủ đề `settings.json` → `hooks` → `PostToolUse` matcher, không phải Agent SDK.

---

## Thứ tự vá cho lần thi lại

| Ưu tiên | Nội dung | Vì sao |
|---|---|---|
| **1** | **D + C** — hook trong `settings.json` (format/lint/test sau edit) và phân biệt settings-permissions/hooks vs CLAUDE.md, kèm **tái cấu trúc** | Hai mục 0% cùng một chủ đề, hoàn toàn chưa có trong ghi chú. Vá được là gần như đủ điểm qua |
| **2** | **A + B** — trạng thái cuối phiên và `authorization state` trong handoff | Hai mục 0% cùng một chủ đề LV1, chỉ cần bổ sung vế thứ hai vào phần đã nắm |
| **3** | **E** — `tool_choice` cho **sequencing**, ghép với prerequisite gate của 1.4 | Lỗ hổng lặp lại lần thứ ba (Q12 → quiz → đề thật). Phải học ở dạng **ghép**, không phải dạng rời |
| **4** | 25–50%: extraction accuracy (few-shot + normalization), human review routing, tool description disambiguation, MCP tool discovery verification | Điểm thấp nhưng không phải 0 — nền đã có, cần luyện dạng áp dụng |
| **5** | 50% — vì sao phải gửi lại toàn bộ lịch sử trong **stateless API** | Một mục lẻ, học nhanh |

**Cách ôn khác lần trước:** lần này đừng học lại định nghĩa — anh đã 100% ở 15/29 mục toàn dạng "select/explain/apply". Hãy luyện dạng câu **"hệ thống này đang sai, tái cấu trúc thế nào"** và **"đảm bảo X xảy ra bất kể Y"**.

---

## Cần bổ sung vào ghi chú

- [ ] `16-tong-hop-kien-truc-agent.md` — thêm mục **1.8 Bảo đảm trạng thái cuối phiên**; bổ sung `authorization state` vào mẫu handoff ở 1.4
- [ ] `13-tong-hop-claude-code-workflow.md` — thêm mục về **hook trong `settings.json`** (PostToolUse cho format/lint/test) và **ma trận quyết định** settings permissions / hooks / CLAUDE.md / rules / skills kèm bài tập tái cấu trúc
- [ ] `12-tong-hop-tool-va-mcp.md` — thêm mục **`tool_choice` cho chuỗi phụ thuộc**, nối tường minh sang prerequisite gate của LV1 1.4
- [ ] `15-tong-hop-context-va-do-tin-cay.md` — làm rõ 5.5: routing theo **confidence + đặc điểm tài liệu + mơ hồ cấp trường**, đối lập với **random sampling** (33% là mục thấp thứ hai)
