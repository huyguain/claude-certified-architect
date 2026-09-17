# Tổng hợp kiến thức — Lĩnh vực 1: Kiến trúc & Điều phối Agent (27%)

> Bao phủ **toàn bộ** blueprint LV1 (1.1 → 1.7).
> Nguồn: `../guide_vi.md` Ch.1.3 (177–189), Ch.3 (327–466), Ch.5.10 (789–822), Ch.8 (1097–1149), Ch.9.3 (1210–1233), blueprint 1536–1627
> Kế hoạch vá lỗ hổng từ câu sai: [`09-ke-hoach-lanh-vuc-1.md`](./09-ke-hoach-lanh-vuc-1.md)
> Liên quan: [`12-tong-hop-tool-va-mcp.md`](./12-tong-hop-tool-va-mcp.md) (giới hạn tool subagent) · [`15-tong-hop-context-va-do-tin-cay.md`](./15-tong-hop-context-va-do-tin-cay.md) (lan truyền lỗi, escalation)

**27% đề thi — lĩnh vực NẶNG NHẤT.** Với ~76 câu → khoảng **20–21 câu**. Đầu tư ôn nhiều nhất ở đây.

---

## Bản đồ 7 subdomain

| # | Chủ đề | Câu hỏi cốt lõi đề hay hỏi | Chương tham chiếu |
|---|---|---|---|
| 1.1 | Agentic loop | *Vòng lặp dừng khi nào? Dấu hiệu nào đáng tin?* | Ch.1.3, Ch.3.1 |
| 1.2 | Coordinator–subagent | *Ai nói chuyện với ai? Chia việc thế nào?* | Ch.3.3 |
| 1.3 | Spawn & truyền context | *Subagent biết gì? Gọi song song ra sao?* | Ch.3.2, Ch.3.4 |
| 1.4 | Workflow & bàn giao | *Làm sao ép đúng thứ tự? Bàn giao gì cho người?* | Ch.3.5, Ch.9.3 |
| 1.5 | Hook | *Hook hay prompt?* | Ch.3.5 |
| 1.6 | Phân rã tác vụ | *Pipeline cố định hay thích ứng động?* | Ch.8 |
| 1.7 | Session state | *Resume, fork, hay bắt đầu mới?* | Ch.5.10 |

**Ba trục xuyên suốt:**
- **Tín hiệu cấu trúc thắng tín hiệu văn bản** — 1.1: `stop_reason` là sự thật, chữ nghĩa của assistant thì không.
- **Đảm bảo thắng ảnh hưởng** — 1.4, 1.5: hook/điều kiện tiên quyết là tất định; prompt là xác suất. *(Cùng trục với `tool_choice` ở LV2 và `allowed-tools` ở LV3 — đây là trục lặp lại nhiều nhất toàn kỳ thi.)*
- **Context phải được truyền tường minh** — 1.2, 1.3: subagent **không** kế thừa gì cả.

---

# 1.1 — Agentic loop

## Vòng đời

```
1. Gửi request tới Claude kèm tools
2. Nhận response
3. Kiểm tra stop_reason:
     "tool_use"  → thực thi tool → NỐI kết quả vào lịch sử → quay lại bước 1
     "end_turn"  → tác vụ hoàn tất → trả kết quả cho người dùng
4. Lặp đến khi xong
```

## Bảng `stop_reason` — thuộc lòng

| Giá trị | Nghĩa | Hành động |
|---|---|---|
| **`"tool_use"`** | Model muốn gọi tool | **Thực thi tool, trả kết quả → vòng lặp tiếp** |
| **`"end_turn"`** | Model đã hoàn tất | **Dừng vòng lặp**, hiển thị kết quả |
| `"max_tokens"` | Chạm giới hạn token | Phản hồi **bị cắt cụt** — có thể cần tăng giới hạn |
| `"stop_sequence"` | Gặp stop sequence | Xử lý theo logic ứng dụng |

Với hệ agentic, chỉ **hai giá trị đầu** điều khiển vòng lặp.

## Ba anti-pattern ⚠️

| Anti-pattern | Vì sao sai |
|---|---|
| **Phân tích văn bản assistant** tìm chữ "Task completed" | Tín hiệu ngôn ngữ **không đáng tin** — model có thể nói xong mà chưa xong, hoặc xong mà không nói |
| **`max_iterations=5`** làm cơ chế dừng **chính** | Cắt ngang tác vụ chưa xong; chỉ nên là **lưới an toàn**, không phải điều kiện dừng |
| **Kiểm tra assistant có sinh text hay không** | Model có thể vừa sinh text vừa gọi tool trong cùng một lượt |

> **Tín hiệu hoàn thành đáng tin cậy DUY NHẤT là `stop_reason == "end_turn"`.**

## Model-driven vs cây quyết định cứng

| | Model-driven (agentic loop) | Cây quyết định hard-code |
|---|---|---|
| Ai chọn tool tiếp theo | **Claude**, dựa trên context + kết quả tool trước | Lập trình viên, cố định sẵn |
| Thích ứng với phát hiện mới | ✅ Có | ❌ Không |
| Dùng khi | Tác vụ mở, đường đi chưa biết trước | Quy trình cố định, cần tái lập tuyệt đối |

Điểm mấu chốt: **kết quả tool được NỐI vào lịch sử hội thoại**, nhờ đó model suy luận được về hành động kế tiếp. Không nối = model mù ở vòng sau.

---

# 1.2 — Hub-and-spoke: coordinator & subagent

## Topology

```
         Coordinator
        /     |      \
  Subagent1  Subagent2  Subagent3
   (search)  (analysis) (synthesis)
```

**Mọi giao tiếp đi qua coordinator** — subagent **không** nói chuyện trực tiếp với nhau. Lý do: **observability**, **xử lý lỗi nhất quán**, **kiểm soát luồng thông tin**.

## Sáu trách nhiệm của coordinator

1. **Phân rã** tác vụ thành tác vụ con
2. **Quyết định cần subagent nào** — lựa chọn **động**, không phải luôn chạy hết pipeline
3. **Ủy thác** công việc
4. **Tổng hợp + validate** kết quả
5. **Xử lý lỗi và retry**
6. **Truyền đạt** kết quả tới người dùng

## Nguyên tắc cốt lõi: subagent có context TÁCH BIỆT ⚠️

- Subagent **KHÔNG** tự động kế thừa lịch sử hội thoại của coordinator
- Mọi context cần thiết phải được **truyền tường minh trong prompt**
- Subagent **không chia sẻ bộ nhớ** giữa các lần gọi

> Đây là kiến thức bị hỏi nhiều nhất của 1.2–1.3. Mọi đáp án giả định subagent "tự biết" những gì coordinator đã thấy đều sai.

## Hai rủi ro thiết kế

**Phân rã quá hẹp** — coordinator chia đề tài nghiên cứu rộng thành các truy vấn quá hẹp → **bao phủ thiếu**. Đối ứng: **vòng lặp tinh chỉnh lặp lại**:

```
coordinator đánh giá bản tổng hợp → phát hiện LỖ HỔNG
  → ủy thác lại cho search/analysis với TRUY VẤN NHẮM ĐÚNG lỗ hổng
  → gọi lại synthesis
  → lặp đến khi độ bao phủ đủ
```

**Trùng lặp công việc** — nhiều subagent tìm cùng một thứ. Đối ứng: **chia phạm vi** — giao mỗi agent một **chủ đề con riêng** hoặc một **loại nguồn riêng**.

---

# 1.3 — Spawn subagent & truyền context

## Tool `Task`

Subagent được sinh ra qua tool **`Task`**.

> ⚠️ **`allowedTools` của coordinator PHẢI bao gồm `"Task"`** — thiếu nó thì coordinator không spawn được subagent nào.

```python
coordinator_agent = AgentDefinition(
    allowed_tools=["Task", "get_customer"]
)
```

## `AgentDefinition`

```python
agent = AgentDefinition(
    name="customer_support",
    description="Handles customer requests for returns and order issues",
    system_prompt="You are a customer support agent...",
    allowed_tools=["get_customer", "lookup_order", "process_refund", "escalate_to_human"],
)
```

| Tham số | Vai trò |
|---|---|
| `name` / `description` | Định danh + mô tả agent |
| `system_prompt` | Chỉ dẫn cho agent |
| `allowed_tools` | Danh sách tool được phép — **nguyên tắc least privilege** (nối sang 2.3) |

## Truyền context tường minh là BẮT BUỘC

```
# Tệ: subagent không có context
Task: "Analyze the document"

# Tốt: đầy đủ context trong prompt
Task: "Analyze the following document.
Document: [toàn văn tài liệu]
Prior search results: [kết quả tìm kiếm web]
Output format requirements: [schema]"
```

**Tách nội dung khỏi metadata** bằng định dạng có cấu trúc (source URL, tên tài liệu, số trang) khi truyền context giữa các agent — để **bảo toàn quy kết nguồn** (nối sang 5.6).

## Spawn song song

> Coordinator gọi **nhiều `Task` trong MỘT response** → các subagent chạy **đồng thời**.

```
# Một response duy nhất của coordinator chứa:
Task 1: "Search for articles about X"
Task 2: "Analyze document Y"
Task 3: "Search for articles about Z"
# Cả ba chạy song song
```

⚠️ Chia ra **nhiều lượt** thì chúng chạy **tuần tự** — mất hết lợi ích song song. Đây là chi tiết hay bị hỏi.

## Prompt coordinator: mục tiêu, không phải quy trình

Viết prompt coordinator theo hướng **mục tiêu nghiên cứu + tiêu chí chất lượng**, **không phải** hướng dẫn từng bước. Lý do: để subagent **thích ứng** được với những gì chúng phát hiện — ép quy trình cứng thì mất luôn ưu thế model-driven của 1.1.

---

# 1.4 — Workflow nhiều bước & bàn giao

## Cưỡng chế theo lập trình vs hướng dẫn bằng prompt ⚠️

| | Cưỡng chế lập trình (hook, điều kiện tiên quyết) | Hướng dẫn bằng prompt |
|---|---|---|
| Mức đảm bảo | **Tất định (100%)** | **Xác suất (>90%, không 100%)** |
| Dùng khi | Quy tắc nghiệp vụ then chốt, tài chính, tuân thủ, xác minh danh tính | Tùy chọn chung, khuyến nghị, định dạng |

> **Khi thất bại gây hậu quả tài chính / pháp lý / an toàn → dùng HOOK, không dùng prompt.**
> Prompt có **tỷ lệ thất bại khác 0** — với xác minh danh tính trước thao tác tài chính, con số đó là không chấp nhận được.

## Điều kiện tiên quyết theo lập trình

Chặn tool phía sau cho tới khi bước trước hoàn tất:

```
Chặn process_refund cho đến khi get_customer trả về customer ID ĐÃ XÁC MINH
```

Đây là phiên bản LV1 của cùng một ý với `tool_choice` forced ở LV2: **thứ tự bắt buộc phải được cưỡng chế bằng cơ chế, không bằng lời dặn.**

## Phân rã yêu cầu nhiều khía cạnh

Khách gửi một tin nhắn chứa 3 vấn đề → **tách thành các mục riêng biệt** → **điều tra song song dùng context chung** → **tổng hợp thành một giải pháp thống nhất**. Không xử lý tuần tự từng cái như ba hội thoại rời.

## Bàn giao có cấu trúc khi escalation

```json
{"customer_id": "CUST-12345", "customer_name": "...",
 "issue_summary": "Refund request for a damaged item",
 "order_id": "ORD-67890",
 "root_cause": "Item arrived damaged; photos attached",
 "actions_taken": ["Verified customer via get_customer",
                   "Confirmed order via lookup_order",
                   "Offered standard replacement — customer insists on refund"],
 "refund_amount": "$89.99",
 "recommended_action": "Approve a full refund",
 "escalation_reason": "Customer requested to speak with a manager"}
```

> **Người vận hành KHÔNG có quyền truy cập bản ghi hội thoại** — họ chỉ thấy bản tóm tắt này. Nó phải **đầy đủ và tự chứa**.

---

# 1.5 — Hook trong Agent SDK

## Hai loại hook cần thuộc

**`PostToolUse`** — chặn **kết quả tool** *trước khi* model đọc:

```python
@hook("PostToolUse")
def normalize_dates(tool_result):
    # Unix timestamp → ISO 8601
    # "Mar 5, 2025"   → "2025-03-05"
    return normalized_result
```
Công dụng chính: **chuẩn hóa dữ liệu không đồng nhất** từ các MCP tool khác nhau (Unix timestamp, ISO 8601, mã trạng thái dạng số). Công dụng thứ hai: **cắt gọn output dài dòng** (nối sang 5.1).

**`PreToolUse`** — chặn **lời gọi tool đi ra** *trước khi* thực thi:

```python
@hook("PreToolUse")
def enforce_refund_limit(tool_call):
    if tool_call.name == "process_refund" and tool_call.args.amount > 500:
        return redirect_to_escalation(tool_call)
```
Công dụng: **cưỡng chế tuân thủ** — chặn hành động vi phạm chính sách và **chuyển hướng sang workflow thay thế** (escalation cho người).

## Hook vs prompt

| Thuộc tính | Hook | Prompt |
|---|---|---|
| Đảm bảo | **Deterministic 100%** | **Xác suất >90%** |
| Ví dụ | Chặn hoàn tiền > $500 | "Cố gắng giải quyết trước khi escalation" |

Nhận diện trong đề: thấy **"guaranteed", "must never", "compliance", ngưỡng tiền cụ thể** → **hook**. Thấy **"prefer", "generally", "try to"** → prompt là đủ.

---

# 1.6 — Chiến lược phân rã tác vụ

## Pipeline cố định (prompt chaining)

```
Document → Metadata extraction → Data extraction → Validation → Enrichment → Final output
```

**Dùng khi:** cấu trúc tác vụ **dự đoán được** · **mọi bước biết trước** · cần **ổn định và tái lập**.

## Phân rã thích ứng động

```
1. "Add tests for a legacy codebase"
2. → Trước hết: lập bản đồ cấu trúc (Glob, Grep)
3. → Phát hiện: 3 module không có test, 2 module phủ một phần
4. → Ưu tiên: bắt đầu từ module payments (rủi ro cao)
5. → Trong lúc làm: phát hiện phụ thuộc vào API bên ngoài
6. → Thích ứng: thêm mock cho API đó trước khi viết test
```

**Dùng khi:** tác vụ **điều tra mở** · **không biết trước toàn bộ phạm vi** · **mỗi bước phụ thuộc kết quả bước trước**.

> Câu hỏi chốt để chọn: **"Mình có biết trước đầy đủ các bước không?"**
> Biết → pipeline cố định. Không biết, phải dò ra → thích ứng động.

## Review nhiều lượt (multi-pass)

Với PR 10+ file:
```
Pass 1 (per-file)    : auth.ts / database.ts / routes.ts → vấn đề CỤC BỘ
Pass 2 (integration) : quan hệ giữa các file → kiểu không nhất quán, phụ thuộc vòng
```

Một lượt duy nhất qua 14 file gây: **pha loãng chú ý** · **nhận xét không nhất quán** · **bỏ sót lỗi hiển nhiên** do quá tải nhận thức.

*(Trùng với 4.6 — cùng một kiến thức, hai lĩnh vực đều hỏi.)*

---

# 1.7 — Session state, resume & fork

## `--resume <session-name>`

```bash
claude --resume investigation-auth-bug
```
Tiếp tục một hội thoại đã đặt tên, kèm context đã lưu. Hữu ích cho điều tra dài trải nhiều session.

⚠️ **Rủi ro: nếu file đã thay đổi kể từ session trước, kết quả tool có thể LỖI THỜI.**

## `fork_session`

```
        Codebase investigation
                 |
            fork_session
            /           \
   Approach A:          Approach B:
     Redux               Context API
```

Cả hai nhánh **kế thừa context tới điểm phân nhánh**, sau đó **phân kỳ độc lập**. Dùng để **so sánh các phương án** hoặc thử nghiệm chiến lược song song từ **một baseline phân tích chung**.

## Resume hay bắt đầu mới ⚠️

| Tình huống | Chọn |
|---|---|
| Context trước **vẫn còn hiện hành** | **Resume** |
| **Kết quả tool đã lỗi thời** (file đã đổi) | **Session mới + bản tóm tắt có cấu trúc** |
| Đã trôi qua **nhiều thời gian**, context suy giảm | **Session mới** |

> Tốt hơn nên khởi động lại với *"Đây là bản tóm tắt ngắn về những gì chúng ta đã tìm thấy: ..."* thay vì resume mang theo dữ liệu tool cũ.

Nếu vẫn resume sau khi code đổi: **báo cho agent biết chính xác file nào đã thay đổi** để nó **phân tích lại có trọng điểm**, thay vì bắt khám phá lại toàn bộ.

---

# Bảng ghi nhớ một trang — Lĩnh vực 1

```
════ 1.1 AGENTIC LOOP ══════════════════════════════════════════════════════
  "tool_use" → thực thi tool → NỐI kết quả vào lịch sử → lặp
  "end_turn" → DỪNG  (tín hiệu hoàn thành ĐÁNG TIN DUY NHẤT)
  "max_tokens" = bị cắt cụt | "stop_sequence" = theo logic ứng dụng
  ✗ 3 anti-pattern: đọc chữ "Task completed" · max_iterations làm cơ chế dừng CHÍNH
                    · kiểm tra có sinh text hay không
  Model-driven (Claude chọn tool kế) ≠ cây quyết định hard-code

════ 1.2 COORDINATOR–SUBAGENT ══════════════════════════════════════════════
  Hub-and-spoke: MỌI giao tiếp qua coordinator → observability, lỗi nhất quán
  Coordinator: phân rã · CHỌN ĐỘNG subagent · ủy thác · tổng hợp+validate
               · xử lý lỗi/retry · truyền đạt
  ⚠ Subagent context TÁCH BIỆT — KHÔNG kế thừa lịch sử, KHÔNG chia sẻ bộ nhớ
  Phân rã QUÁ HẸP → thiếu bao phủ → VÒNG LẶP TINH CHỈNH:
      đánh giá synthesis → thấy lỗ hổng → ủy thác lại truy vấn NHẮM ĐÚNG → lặp
  Chống trùng lặp: chia CHỦ ĐỀ CON hoặc LOẠI NGUỒN riêng cho mỗi agent

════ 1.3 SPAWN & CONTEXT ═══════════════════════════════════════════════════
  Tool "Task" spawn subagent — allowedTools của coordinator PHẢI có "Task"
  AgentDefinition: name · description · system_prompt · allowed_tools
  Context phải TRUYỀN TƯỜNG MINH trong prompt (tài liệu + kết quả trước + schema)
  Metadata tách khỏi nội dung (URL, tên tài liệu, số trang) → giữ quy kết nguồn
  SONG SONG = nhiều Task trong MỘT response  (nhiều lượt = TUẦN TỰ, mất lợi ích)
  Prompt coordinator: MỤC TIÊU + TIÊU CHÍ CHẤT LƯỢNG, không phải từng bước

════ 1.4 WORKFLOW & BÀN GIAO ═══════════════════════════════════════════════
  Cưỡng chế lập trình = TẤT ĐỊNH | prompt = XÁC SUẤT (>90%, khác 0 thất bại)
  → tài chính / pháp lý / an toàn / xác minh danh tính = HOOK, không prompt
  Điều kiện tiên quyết: chặn process_refund đến khi get_customer trả ID ĐÃ XÁC MINH
  Yêu cầu nhiều khía cạnh → TÁCH MỤC → điều tra SONG SONG dùng context chung
                            → tổng hợp một giải pháp thống nhất
  Handoff TỰ CHỨA: customer_id · root_cause · actions_taken · refund_amount
                   · recommended_action · escalation_reason
                   (người vận hành KHÔNG thấy bản ghi hội thoại)

════ 1.5 HOOK ══════════════════════════════════════════════════════════════
  PostToolUse → chặn KẾT QUẢ trước khi model đọc
                chuẩn hóa Unix ts / ISO 8601 / mã trạng thái số · cắt gọn output
  PreToolUse  → chặn LỜI GỌI ĐI RA, chặn vi phạm chính sách + CHUYỂN HƯỚNG
                (refund > $500 → escalation)
  "guaranteed / must never / compliance / ngưỡng tiền" → HOOK
  "prefer / generally / try to"                        → prompt đủ

════ 1.6 PHÂN RÃ TÁC VỤ ════════════════════════════════════════════════════
  PIPELINE CỐ ĐỊNH (prompt chaining): cấu trúc dự đoán được, mọi bước biết trước,
                                       cần ổn định + tái lập
  THÍCH ỨNG ĐỘNG: điều tra mở, chưa biết phạm vi, mỗi bước phụ thuộc bước trước
  Câu hỏi chốt: "có biết trước đầy đủ các bước không?"
  MULTI-PASS: Pass 1 CỤC BỘ từng file + Pass 2 TÍCH HỢP xuyên file
              (một lượt 14 file → pha loãng chú ý, nhận xét mâu thuẫn, bỏ sót)

════ 1.7 SESSION ═══════════════════════════════════════════════════════════
  --resume <tên>  tiếp tục session đã đặt tên; RỦI RO kết quả tool LỖI THỜI
  fork_session    hai nhánh kế thừa context tới điểm rẽ, rồi PHÂN KỲ độc lập
                  → so sánh phương án từ MỘT baseline chung
  context còn hiện hành → RESUME
  kết quả lỗi thời / đã lâu → SESSION MỚI + BẢN TÓM TẮT CÓ CẤU TRÚC
  Nếu vẫn resume sau khi code đổi → BÁO file nào đã thay đổi để phân tích lại
                                     CÓ TRỌNG ĐIỂM

════ BA TRỤC ═══════════════════════════════════════════════════════════════
  TÍN HIỆU CẤU TRÚC thắng VĂN BẢN   → 1.1 stop_reason
  ĐẢM BẢO thắng ẢNH HƯỞNG           → 1.4, 1.5 (+ tool_choice LV2, allowed-tools LV3)
  CONTEXT PHẢI TRUYỀN TƯỜNG MINH    → 1.2, 1.3
```

---

# Tự kiểm tra toàn lĩnh vực (16 câu)

> Che phần **Đáp án** lại, tự trả lời trước rồi mới đối chiếu.

## 1.1 — Agentic loop

**Câu 1.** Vòng lặp agent nên tiếp tục khi nào và dừng khi nào? Nêu **ba** anti-pattern về điều kiện dừng.

<details><summary><b>Đáp án</b></summary>

- **Tiếp tục** khi `stop_reason == "tool_use"` → thực thi tool, **nối kết quả vào lịch sử**, gửi lại.
- **Dừng** khi `stop_reason == "end_turn"`.

Ba anti-pattern:
1. **Phân tích văn bản assistant** tìm chữ như "Task completed" — tín hiệu ngôn ngữ không đáng tin.
2. **Dùng `max_iterations=5` làm cơ chế dừng chính** — cắt ngang tác vụ chưa xong (chỉ nên là lưới an toàn).
3. **Kiểm tra assistant có sinh text hay không** — model có thể vừa sinh text vừa gọi tool trong cùng một lượt.

> Tín hiệu hoàn thành **đáng tin cậy duy nhất** là `stop_reason == "end_turn"`.
</details>

**Câu 2.** Vì sao phải nối kết quả tool vào lịch sử hội thoại giữa các vòng lặp? Và `stop_reason = "max_tokens"` nghĩa là gì?

<details><summary><b>Đáp án</b></summary>

Nối kết quả tool để model **suy luận được về hành động tiếp theo** dựa trên thông tin mới. Không nối thì ở vòng sau model mù — nó không biết tool vừa trả về gì.

`"max_tokens"` = **chạm giới hạn token**, phản hồi **bị cắt cụt**. Đây **không** phải tín hiệu hoàn thành; có thể cần tăng `max_tokens`.
</details>

**Câu 3.** Phân biệt cách tiếp cận model-driven với cây quyết định hard-code. Mỗi cái dùng khi nào?

<details><summary><b>Đáp án</b></summary>

| | Model-driven | Cây quyết định cứng |
|---|---|---|
| Ai chọn tool kế tiếp | **Claude**, dựa trên context + kết quả tool trước | Lập trình viên, cố định sẵn |
| Thích ứng phát hiện mới | ✅ | ❌ |
| Dùng khi | Tác vụ mở, đường đi chưa biết trước | Quy trình cố định, cần tái lập tuyệt đối |

Agentic loop là model-driven. Nếu đề nhấn "chuỗi hành động cố định, biết trước" thì đó không còn là bài toán agentic loop nữa (nối sang 1.6: pipeline cố định).
</details>

---

## 1.2 — Coordinator & subagent

**Câu 4.** Vì sao mọi giao tiếp giữa subagent phải đi qua coordinator? Nêu **ba** lý do.

<details><summary><b>Đáp án</b></summary>

1. **Observability** — nhìn được toàn bộ luồng thông tin ở một chỗ.
2. **Xử lý lỗi nhất quán** — một nơi quyết định retry / dùng partial / bỏ qua.
3. **Kiểm soát luồng thông tin** — coordinator quyết định ai được biết gì, tránh rò rỉ context.

Đây là bản chất của **hub-and-spoke**: subagent **không** nói chuyện trực tiếp với nhau.
</details>

**Câu 5.** Bản tổng hợp cuối bị thiếu bao phủ vì coordinator chia đề tài thành các truy vấn quá hẹp. Cơ chế nào khắc phục? Và chống trùng lặp giữa các subagent bằng cách nào?

<details><summary><b>Đáp án</b></summary>

Khắc phục thiếu bao phủ: **vòng lặp tinh chỉnh lặp lại**:
```
coordinator đánh giá bản tổng hợp → phát hiện LỖ HỔNG
  → ủy thác lại cho search/analysis với TRUY VẤN NHẮM ĐÚNG lỗ hổng
  → gọi lại synthesis → lặp đến khi độ bao phủ đủ
```

Chống trùng lặp: **chia phạm vi ngay từ đầu** — giao mỗi subagent một **chủ đề con riêng** hoặc một **loại nguồn riêng**.
</details>

**Câu 6.** Coordinator có nên luôn chạy hết pipeline search → analysis → synthesis cho mọi truy vấn không?

<details><summary><b>Đáp án</b></summary>

**Không.** Một trong sáu trách nhiệm của coordinator là **lựa chọn subagent một cách ĐỘNG** — phân tích yêu cầu của truy vấn rồi quyết định cần gọi những subagent nào, dựa trên **độ phức tạp của truy vấn**.

Luôn chạy full pipeline là lãng phí với truy vấn đơn giản, và là dấu hiệu của thiết kế cứng nhắc.
</details>

---

## 1.3 — Spawn & truyền context

**Câu 7.** Coordinator không spawn được subagent nào. Nguyên nhân cấu hình nhiều khả năng nhất là gì?

<details><summary><b>Đáp án</b></summary>

**`allowedTools` của coordinator thiếu `"Task"`.** Tool `Task` là cơ chế spawn subagent; không có nó trong danh sách tool được phép thì coordinator không gọi được.

```python
coordinator_agent = AgentDefinition(allowed_tools=["Task", "get_customer"])
```
</details>

**Câu 8.** Subagent tổng hợp (synthesis) cho ra kết quả rời rạc, không dùng được kết quả tìm kiếm mà coordinator đã thu thập. Vì sao? Prompt đúng phải chứa gì?

<details><summary><b>Đáp án</b></summary>

Vì **subagent có context TÁCH BIỆT** — nó **không tự động kế thừa** lịch sử hội thoại của coordinator, và **không chia sẻ bộ nhớ** giữa các lần gọi.

Prompt đúng phải **đưa toàn bộ phát hiện của các agent trước vào trực tiếp**:
```
Task: "Analyze the following document.
Document: [toàn văn]
Prior search results: [kết quả tìm kiếm web]
Output format requirements: [schema]"
```

Thêm: dùng **định dạng có cấu trúc tách nội dung khỏi metadata** (source URL, tên tài liệu, số trang) để **bảo toàn quy kết nguồn** qua các bước.
</details>

**Câu 9.** Muốn ba subagent chạy song song thì coordinator phải làm gì? Làm sai thì hậu quả ra sao?

<details><summary><b>Đáp án</b></summary>

Phát **nhiều lời gọi `Task` trong MỘT response duy nhất** của coordinator:

```
# Một response chứa:
Task 1: "Search for articles about X"
Task 2: "Analyze document Y"
Task 3: "Search for articles about Z"
→ cả ba chạy đồng thời
```

Làm sai (mỗi `Task` một lượt riêng) → các subagent chạy **tuần tự**, mất hoàn toàn lợi ích song song và kéo dài thời gian gấp nhiều lần.
</details>

**Câu 10.** Prompt của coordinator nên viết theo hướng nào — quy trình từng bước hay mục tiêu? Vì sao?

<details><summary><b>Đáp án</b></summary>

Theo hướng **mục tiêu nghiên cứu + tiêu chí chất lượng**, **không phải** hướng dẫn từng bước.

Lý do: để subagent **thích ứng** được với những gì chúng thực sự phát hiện. Ép quy trình cứng vào prompt là quay về **cây quyết định hard-code** — mất luôn ưu thế model-driven vốn là lý do dùng kiến trúc agentic (nối ngược về 1.1).
</details>

---

## 1.4 & 1.5 — Cưỡng chế và hook

**Câu 11.** Quy tắc: phải xác minh danh tính khách hàng trước mọi thao tác tài chính. Viết vào system prompt đã đủ chưa? Cơ chế đúng là gì?

<details><summary><b>Đáp án</b></summary>

**Chưa đủ.** Chỉ dẫn prompt cho **mức tuân thủ xác suất (>90% nhưng khác 100%)** — với thao tác tài chính, **tỷ lệ thất bại khác 0 là không chấp nhận được**.

Cơ chế đúng: **điều kiện tiên quyết theo lập trình** — chặn `process_refund` cho tới khi `get_customer` **trả về một customer ID đã được xác minh**. Cưỡng chế bằng hook/gate, không bằng lời dặn.

> Nguyên tắc: thất bại gây hậu quả **tài chính / pháp lý / an toàn** → **hook**, không prompt.
</details>

**Câu 12.** Phân biệt `PostToolUse` và `PreToolUse`: mỗi cái chặn cái gì, dùng cho việc gì?

<details><summary><b>Đáp án</b></summary>

| Hook | Chặn cái gì | Công dụng chính |
|---|---|---|
| **`PostToolUse`** | **Kết quả tool**, *trước khi model đọc* | **Chuẩn hóa dữ liệu không đồng nhất** từ các MCP tool khác nhau (Unix timestamp / ISO 8601 / mã trạng thái dạng số); cũng dùng để **cắt gọn output dài dòng** |
| **`PreToolUse`** | **Lời gọi tool đi ra**, *trước khi thực thi* | **Cưỡng chế tuân thủ** — chặn hành động vi phạm chính sách (refund > $500) và **chuyển hướng** sang workflow thay thế (escalation) |

Mẹo nhớ: **Post = dữ liệu vào**, **Pre = hành động ra**.
</details>

**Câu 13.** Khi nào chọn hook, khi nào prompt là đủ? Nêu từ khóa nhận diện trong đề.

<details><summary><b>Đáp án</b></summary>

| | Hook | Prompt |
|---|---|---|
| Đảm bảo | **Tất định 100%** | **Xác suất >90%** |
| Dùng cho | Quy tắc nghiệp vụ then chốt, tài chính, tuân thủ | Tùy chọn chung, khuyến nghị, định dạng |
| Ví dụ | Chặn hoàn tiền > $500 | "Cố gắng giải quyết trước khi escalation" |

Từ khóa: **"guaranteed", "must never", "compliance", ngưỡng tiền cụ thể** → **hook**. **"prefer", "generally", "try to"** → prompt đủ.
</details>

---

## 1.6 — Phân rã tác vụ

**Câu 14.** Phân loại: (a) quy trình review tài liệu luôn theo cùng template; (b) "thêm test toàn diện cho một codebase legacy". Mỗi cái dùng chiến lược phân rã nào? Câu hỏi chốt là gì?

<details><summary><b>Đáp án</b></summary>

- **(a) → Pipeline cố định (prompt chaining)**: cấu trúc **dự đoán được**, mọi bước **biết trước**, cần **ổn định và tái lập**.
  `Document → Metadata → Data extraction → Validation → Enrichment → Output`
- **(b) → Phân rã thích ứng động**: tác vụ **mở**, **chưa biết phạm vi**, mỗi bước phụ thuộc kết quả bước trước.
  `Lập bản đồ cấu trúc (Glob/Grep) → phát hiện 3 module không test → ưu tiên payments (rủi ro cao) → phát hiện phụ thuộc API ngoài → thích ứng: thêm mock trước khi viết test`

**Câu hỏi chốt: "Mình có biết trước đầy đủ các bước không?"** Biết → cố định. Phải dò ra → thích ứng.
</details>

---

## 1.7 — Session state

**Câu 15.** Bạn muốn so sánh hai chiến lược refactor xuất phát từ cùng một phân tích codebase. Cơ chế nào? Nó khác `--resume` ở chỗ nào?

<details><summary><b>Đáp án</b></summary>

**`fork_session`** — tạo hai nhánh **kế thừa context tới điểm phân nhánh**, sau đó **phân kỳ độc lập**:

```
        Codebase investigation
            /           \
    Approach A:        Approach B:
      Redux             Context API
```

Khác `--resume`: `--resume <tên>` **tiếp tục MỘT dòng hội thoại** đã đặt tên; `fork_session` **tạo NHIỀU nhánh song song** từ một baseline chung để so sánh phương án.
</details>

**Câu 16.** Bạn quay lại một session điều tra sau hai ngày, trong đó nhiều file đã bị sửa. Resume hay bắt đầu mới? Nếu vẫn resume thì phải làm gì thêm?

<details><summary><b>Đáp án</b></summary>

Nghiêng về **bắt đầu session mới kèm bản tóm tắt có cấu trúc** — vì **kết quả tool đã lỗi thời** (file đã đổi) và **context đã suy giảm sau thời gian dài**. Khởi động lại với *"Đây là bản tóm tắt ngắn về những gì chúng ta đã tìm thấy: ..."* **đáng tin cậy hơn** resume mang theo dữ liệu tool cũ.

| Tình huống | Chọn |
|---|---|
| Context trước vẫn hiện hành | Resume |
| Kết quả tool lỗi thời / đã lâu | **Session mới + tóm tắt có cấu trúc** |

Nếu vẫn resume: **báo cho agent biết chính xác những file nào đã thay đổi**, để nó **phân tích lại có trọng điểm** thay vì phải khám phá lại toàn bộ.
</details>
