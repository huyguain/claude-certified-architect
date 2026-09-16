# Tổng hợp kiến thức — Lĩnh vực 5: Quản lý Context & Độ tin cậy (15%)

> Bao phủ **toàn bộ** blueprint LV5 (5.1 → 5.6).
> Nguồn: `../guide_vi.md` Ch.1.5 (200–216), Ch.9 (1150–1250), Ch.10 (1251–1312), Ch.11 (1313–1431), Ch.12 (1432–1497), blueprint 1870–1960
> Liên quan: [`12-tong-hop-tool-va-mcp.md`](./12-tong-hop-tool-va-mcp.md) (lỗi có cấu trúc — trùng nhiều với 5.3) · [`14-tong-hop-prompt-va-structured-output.md`](./14-tong-hop-prompt-va-structured-output.md) (độ tin cậy, review)

**15% đề thi** — lĩnh vực **nhẹ nhất**. Với ~76 câu → khoảng **11 câu**.

---

## Bản đồ 6 subdomain

| # | Chủ đề | Câu hỏi cốt lõi đề hay hỏi | Chương tham chiếu |
|---|---|---|---|
| 5.1 | Context hội thoại | *Thông tin quan trọng bị mất — cứu thế nào?* | Ch.1.5, Ch.11.1–11.3 |
| 5.2 | Escalation & mơ hồ | *Khi nào chuyển cho người, khi nào tự xử?* | Ch.9.1–9.3 (1152–1233) |
| 5.3 | Lan truyền lỗi | *Subagent báo lỗi lên coordinator ra sao?* | Ch.10 (1251–1312) |
| 5.4 | Điều tra codebase lớn | *Session dài bị suy giảm — chống bằng gì?* | Ch.11.4–11.6 (1375–1431) |
| 5.5 | Giám sát người & hiệu chỉnh | *Bao giờ được tự động hóa?* | Ch.9.4 (1234–1250) |
| 5.6 | Provenance & bất định | *Hai nguồn mâu thuẫn — làm gì?* | Ch.12 (1432–1497) |

**Ba trục xuyên suốt:**
- **Bảo toàn thay vì nén** — 5.1, 5.4, 5.6: thứ quan trọng phải nằm **ngoài** vùng bị tóm tắt.
- **Quy tắc tường minh thay vì suy đoán** — 5.2: escalation theo **tác nhân kích hoạt rõ ràng**, không theo cảm xúc hay self-confidence.
- **Phân đoạn thay vì tổng hợp** — 5.5: con số tổng che giấu điểm yếu cục bộ.

---

# 5.1 — Quản lý context hội thoại

## Ba cơ chế làm mất thông tin

| Cơ chế | Mất cái gì |
|---|---|
| **Tóm tắt tiệm tiến** | **Giá trị số, phần trăm, ngày tháng, kỳ vọng khách hàng nêu ra** → biến thành "khoảng", "tầm", "một vài" |
| **Lost-in-the-middle** | Model xử lý tin cậy phần **đầu** và **cuối** đầu vào dài; **bỏ sót phần giữa** |
| **Tích lũy kết quả tool** | Tool trả **40+ trường** nhưng chỉ **5 trường** có nghĩa → phần lớn context bị lãng phí |

Thêm một điểm nền: **phải gửi toàn bộ lịch sử hội thoại** trong các request API tiếp theo — API không tự nhớ; thiếu lịch sử là mất mạch hội thoại.

## Bốn kỹ thuật đối ứng

**1. Khối "case facts" bền vững** — trích sự kiện giao dịch ra **ngoài** lịch sử bị tóm tắt, chèn vào **mọi prompt**:

```
=== CASE FACTS (updated whenever a new fact appears) ===
Customer ID: CUST-12345    Order ID: ORD-67890
Order Date: 2025-01-15     Order Amount: $89.99
Issue: Damaged item on delivery
Customer Request: Full refund
Status: Pending manager approval
===
```

> Ý tưởng cốt lõi: **đừng trông cậy vào lịch sử hội thoại để nhớ con số.** Con số phải sống ở một lớp riêng.
> Với session nhiều vấn đề (multi-issue): tách dữ liệu có cấu trúc của từng issue (order ID, số tiền, trạng thái) sang **lớp context riêng**.

**2. Cắt gọn kết quả tool** — dùng **PostToolUse hook**, chỉ giữ trường liên quan:

```python
@hook("PostToolUse", tool="lookup_order")
def trim_order_fields(result):
    return {"order_id": ..., "status": ..., "total": ...,
            "items": ..., "return_eligible": ...}
```
Tiết kiệm context **và** giảm nhiễu. Cắt **trước khi** nó tích lũy, không phải dọn sau.

**3. Đầu vào nhận biết vị trí** — chống lost-in-the-middle:

```
[KEY FINDINGS — ĐẦU]      Found 3 critical vulnerabilities...
[DETAILED RESULTS — GIỮA] === File auth.ts === ... === File database.ts === ...
[ACTION ITEMS — CUỐI]     Priority: fix auth.ts before merge.
```
Hai việc: **đặt phát hiện then chốt lên đầu** + **tiêu đề mục tường minh** cho phần chi tiết.

**4. Bắt subagent trả metadata có cấu trúc** — ngày tháng, vị trí nguồn, ngữ cảnh phương pháp luận, để bước tổng hợp phía sau diễn giải đúng. Khi agent hạ nguồn có **ngân sách context hẹp**, sửa agent **thượng nguồn** để trả **dữ liệu có cấu trúc** (sự kiện chính, trích dẫn, điểm liên quan) thay vì nội dung dài dòng và chuỗi suy luận.

---

# 5.2 — Escalation và giải quyết mơ hồ

## Năm tác nhân kích hoạt escalation ĐÁNG TIN

| Tình huống | Hành động |
|---|---|
| Khách **yêu cầu rõ ràng** gặp người ("get me a manager") | **Escalation NGAY** — không điều tra trước |
| **Chính sách không bao quát** yêu cầu | Escalation (so giá đối thủ khi chính sách chỉ nói về site của mình) |
| Agent **không thể tiến triển** | Escalation sau một số lần thử hợp lý |
| Thao tác tài chính **vượt ngưỡng** | Escalation — **tốt nhất cưỡng chế bằng hook**, không phải bằng prompt |
| **Nhiều kết quả khớp** khi tra khách hàng | **Hỏi thêm định danh** — không phỏng đoán |

## Ba tác nhân KHÔNG đáng tin ⚠️

| Phương pháp | Vì sao thất bại |
|---|---|
| **Phân tích cảm xúc (sentiment)** | Tâm trạng khách **không tương quan** với độ phức tạp vụ việc |
| **Self-confidence do model tự chấm (1–10)** | Model có thể **sai một cách tự tin**; hiệu chỉnh kém |
| **Bộ phân loại tự động** | Overengineering; đòi dữ liệu huấn luyện bạn không có |

> Đây là mục hay ra đề nhất của 5.2. Ba thứ trên nghe đều "thông minh" nên rất dễ làm đáp án nhiễu.

## Ba mẫu escalation — phân biệt kỹ

**Ngay lập tức** — khách nói thẳng muốn gặp người:
```
Customer: "I want to speak to a manager"
Agent: [gọi escalate_to_human NGAY]
KHÔNG: "I can help with your issue, let me..."
```

**Sau khi cố gắng giải quyết** — vấn đề nằm trong tầm agent:
```
Customer: "My refrigerator broke two days after purchase"
Agent: [kiểm tra đơn, đề xuất đổi bảo hành]
Nếu khách không hài lòng → escalation
```

**Tinh tế: ghi nhận → đề xuất → escalation khi khách NHẮC LẠI**:
```
Customer: "This is outrageous, I'm very unhappy!"
Agent: [ghi nhận] "I understand your frustration."
       [đề xuất cụ thể] "I can offer a replacement or a refund."
Customer: "No, I want to talk to someone!"
Agent: [khách khẳng định lại → escalation ngay]
```

> **Ranh giới cốt lõi:** *bày tỏ bực bội* **≠** *yêu cầu gặp người thật*.
> Bực bội → ghi nhận + đề xuất giải pháp. Yêu cầu gặp người → escalation ngay.
> Escalation ngay lần đầu khách phàn nàn là **sai** — nhưng chần chừ khi khách đã nói rõ muốn gặp người cũng **sai**.

## Bàn giao có cấu trúc (structured handoff)

Khi escalation phải chuyển một bản tóm tắt **tự chứa**:

```json
{"customer_id", "customer_name", "issue_summary", "order_id", "root_cause",
 "actions_taken": ["Verified customer...", "Confirmed order...", "Offered replacement — customer insists on refund"],
 "refund_amount", "recommended_action", "escalation_reason"}
```

> **Người vận hành KHÔNG thấy bản ghi hội thoại** — họ chỉ có bản tóm tắt này. Nên nó phải **đầy đủ và tự chứa**.

## Tiêu chí escalation nằm ở đâu

Viết **tiêu chí tường minh + ví dụ few-shot** vào **system prompt** (nối sang 4.1/4.2). Riêng **ngưỡng tài chính** thì cưỡng chế bằng **hook**, vì prompt chỉ là ảnh hưởng, hook là đảm bảo.

---

# 5.3 — Lan truyền lỗi trong hệ thống multi-agent

> Phần này **trùng nhiều** với 2.2 ở [`12-tong-hop-tool-va-mcp.md`](./12-tong-hop-tool-va-mcp.md). Dưới đây là góc nhìn **multi-agent**: subagent ↔ coordinator.

## Bốn trường bắt buộc trong lỗi có cấu trúc

```json
{"status": "partial_failure",
 "failure_type": "timeout",
 "attempted_query": "AI impact on music industry 2024",
 "partial_results": [{"title": "...", "url": "...", "relevance": 0.8}],
 "alternative_approaches": ["Try a narrower query: ...", "Use an alternative data source"],
 "coverage_impact": "Not covered: AI impact on music production"}
```

Bốn thứ này cho coordinator quyết định được: **retry query đã sửa · dùng partial · giao subagent khác · bỏ qua và chú thích lỗ hổng**. Thiếu chúng thì coordinator mù.

## Phân biệt lỗi truy cập vs kết quả rỗng hợp lệ ⚠️

| Tình huống | Bản chất | Coordinator phải làm |
|---|---|---|
| Truy vấn chạy xong, **không có khớp nào** | **THÀNH CÔNG** | Ghi nhận "không có kết quả", đi tiếp |
| **Timeout / không truy cập được** | **LỖI** | Ra quyết định **retry** |

Trả về mảng rỗng cho ca thứ hai = **ém lỗi âm thầm**, khiến coordinator tưởng đã tìm và không có gì.

## Bốn anti-pattern

| Anti-pattern | Vấn đề |
|---|---|
| Trạng thái chung chung "search unavailable" | **Che giấu context có giá trị** khỏi coordinator |
| Ém lỗi âm thầm (rỗng = thành công) | Coordinator hiểu sai bản chất |
| **Hủy cả workflow** khi một bước hỏng | **Mất toàn bộ kết quả một phần** |
| Retry vô hạn trong subagent | Latency + lãng phí |

Đúng: **phục hồi cục bộ 1–2 lần trong subagent** → chỉ lan truyền cái không tự giải quyết được, **kèm partial results và đã thử gì**.

## Chú thích độ bao phủ trong bản tổng hợp cuối

```markdown
### Visual Art (BAO PHỦ ĐẦY ĐỦ)
[kết quả]

### Music (BAO PHỦ MỘT PHẦN — search agent timeout)
[kết quả một phần]
⚠️ Note: coverage limited due to a timeout in the search agent.

### Literature (BAO PHỦ ĐẦY ĐỦ)
[kết quả]
```

> Nguyên tắc: **lỗ hổng phải hiện ra trong sản phẩm cuối**, không được im lặng biến mất. Người đọc phải biết phần nào được hỗ trợ tốt, phần nào thiếu nguồn.

---

# 5.4 — Điều tra codebase lớn

## Triệu chứng suy giảm context ⚠️

> Session kéo dài → model bắt đầu cho **câu trả lời không ổn định** và **dẫn chiếu "các mẫu điển hình"** thay vì **các lớp cụ thể đã phát hiện trước đó**.

Đây là dấu hiệu nhận diện trong đề: khi thấy mô tả *"model bắt đầu nói chung chung về pattern thay vì tên class cụ thể"* → đó là **suy giảm context**, không phải model kém.

## Bốn cơ chế đối ứng

**1. File scratchpad** — ghi phát hiện then chốt ra file, dẫn chiếu lại sau:
```markdown
# investigation-scratchpad.md
## Key findings
- PaymentProcessor in src/payments/processor.ts inherits from BaseProcessor
- refund() is called from 3 places: OrderController, AdminPanel, CronJob
- External PaymentGateway API rate limit: 100 req/min
- Migration #47 added refund_reason (NOT NULL) — 2024-12-01
```
Khi context suy giảm **hoặc sang session mới**, agent tham khảo scratchpad thay vì chạy lại khám phá.

**2. Ủy thác subagent** — cô lập output dài dòng:
```
Main agent: "Investigate dependencies of the payments module"
  → Subagent (Explore): đọc 15 file, truy vết import
  → Trả về: "Payments depends on AuthService, OrderModel, external PaymentGateway API"
Main agent: giữ MỘT DÒNG trong context thay vì 15 file
```
Agent chính giữ vai trò **điều phối cấp cao**; subagent nhận câu hỏi cụ thể ("tìm mọi file test", "truy vết phụ thuộc luồng hoàn tiền").

**Ngân sách context bị ràng buộc cho subagent:** gửi context **tối thiểu** · yêu cầu trả **kết quả có cấu trúc**, không phải bãi dữ liệu thô · dùng **`allowedTools`** giới hạn tool (ít tool = ít phân tâm + rẻ context).

**Coordinator là một lớp context riêng biệt** — tổng hợp output subagent, giữ trạng thái toàn cục, phân bổ context. Ngăn **"rò rỉ context"**: một agent chiếm cửa sổ bằng thông tin không liên quan tới agent khác.

**3. Tóm tắt giữa các giai đoạn** — tóm tắt phát hiện của giai đoạn trước **rồi mới** spawn subagent giai đoạn sau, **tiêm bản tóm tắt vào context khởi đầu** của chúng.

**4. `/compact`** — nén lịch sử khi context đầy vì output khám phá dài dòng.
⚠️ Rủi ro: **giá trị số chính xác, ngày tháng, chi tiết cụ thể có thể mất** khi tóm tắt. Đây là lý do scratchpad và case-facts tồn tại.

## Phục hồi sau sự cố (crash recovery)

Mỗi agent **xuất trạng thái** ra vị trí đã biết:
```json
// agent-state/web-search-agent.json
{"status": "completed", "queries_executed": [...], "results_count": 12,
 "key_findings": [...], "coverage": ["music composition"], "gaps": ["music licensing"]}
```
Coordinator **nạp manifest** khi tiếp tục và **tiêm vào prompt của agent**:
```json
// agent-state/manifest.json
{"web-search": "completed", "doc-analysis": "in_progress", "synthesis": "not_started"}
```

---

# 5.5 — Giám sát của con người & hiệu chỉnh độ tin cậy

## Bẫy con số tổng hợp ⚠️

> **Độ chính xác tổng thể 97% có thể che giấu 40% lỗi trên MỘT loại tài liệu cụ thể.**

Vì vậy: **phân tích độ chính xác theo LOẠI TÀI LIỆU và theo TRƯỜNG**, không chỉ nhìn con số tổng. Phải xác nhận hiệu năng **ổn định trên mọi phân đoạn** trước khi giảm bớt review của con người.

## Quy trình ba bước

1. **Điểm tin cậy cấp TRƯỜNG** — model xuất confidence cho **từng trường** trích xuất (không phải một điểm cho cả tài liệu).
2. **Hiệu chỉnh (calibration)** — dùng **tập validation đã gán nhãn** để tinh chỉnh ngưỡng. Confidence thô chưa hiệu chỉnh thì vô nghĩa.
3. **Định tuyến (routing)**:
   - Tin cậy **cao** + độ chính xác **ổn định trên mọi phân đoạn** → xử lý **tự động**
   - Tin cậy **thấp** **hoặc** nguồn **mơ hồ/mâu thuẫn** → **con người review**

Mục tiêu của routing: **ưu tiên năng lực reviewer có hạn** vào chỗ đáng ngờ nhất.

## Lấy mẫu ngẫu nhiên phân tầng (stratified random sampling)

Ngay cả với trích xuất **độ tin cậy cao**, vẫn phải **kiểm toán định kỳ một mẫu**. Hai mục đích:
- **Đo tỷ lệ lỗi thực tế** đang trôi qua mà không ai xem
- **Phát hiện mẫu lỗi MỚI** chưa từng thấy

"Phân tầng" = lấy mẫu **theo từng phân đoạn** (loại tài liệu, trường), không lấy ngẫu nhiên trên toàn khối — vì lấy ngẫu nhiên đều sẽ bỏ sót đúng những loại tài liệu hiếm mà model làm tệ.

---

# 5.6 — Provenance & xử lý bất định trong tổng hợp đa nguồn

## Mất quy kết nguồn khi tóm tắt

```
Tệ:  "The AI music market is estimated at $3.2B."   ← không nguồn, không năm

Tốt: {"claim": "The AI music market is estimated at $3.2B.",
      "source_url": "https://example.com/report",
      "source_name": "Global AI Music Report 2024",
      "publication_date": "2024-06-15",
      "confidence": 0.9}
```

Yêu cầu subagent xuất **ánh xạ claim → source** có cấu trúc (URL, tên tài liệu, **trích đoạn liên quan**), và agent tổng hợp phải **bảo toàn + hợp nhất** chúng qua các bước — không được nén mất.

## Xử lý dữ liệu xung đột ⚠️

```json
{"claim": "Share of AI-generated music on streaming platforms",
 "values": [
   {"value": "12%", "source": "Spotify Annual Report 2024", "date": "2024-03",
    "methodology": "Automated classification"},
   {"value": "8%",  "source": "Music Industry Association Survey", "date": "2024-07",
    "methodology": "Survey of 500 labels"}],
 "conflict_detected": true,
 "possible_explanation": "Difference in methodology and time period"}
```

> **Đừng tùy tiện chọn một giá trị.** Giữ **cả hai**, kèm **quy kết nguồn + ngày + phương pháp luận**, và để **coordinator quyết định** cách đối soát trước khi chuyển sang tổng hợp.

Agent phân tích tài liệu phải **hoàn thành công việc** với giá trị mâu thuẫn **được giữ lại và chú thích rõ** — không dừng lại, không tự chọn.

## Ngày tháng chống hiểu nhầm mâu thuẫn

```
Tệ:  "Source A says 10%, source B says 15%. Contradiction."
Tốt: "Source A (2023) says 10%, source B (2024) says 15%.
      Likely +5% growth over a year."
```
→ Bắt subagent đưa **ngày xuất bản / ngày thu thập dữ liệu** vào output có cấu trúc.

## Cấu trúc báo cáo: ổn định vs đang tranh chấp

Tách mục rõ ràng giữa **phát hiện đã vững** và **phát hiện còn tranh cãi**, giữ nguyên **cách diễn đạt gốc của nguồn** và **ngữ cảnh phương pháp luận** — đừng làm phẳng thành một giọng khẳng định duy nhất.

## Render theo loại nội dung

| Loại nội dung | Trình bày |
|---|---|
| Dữ liệu tài chính | **Bảng** |
| Tin tức, phân tích | **Văn xuôi** |
| Phát hiện kỹ thuật | **Danh sách có cấu trúc** |
| Chuỗi thời gian | **Sắp theo trình tự thời gian** |

> Đừng ép mọi thứ vào **một định dạng đồng nhất** — mỗi loại dữ liệu mất thông tin theo một kiểu khác nhau khi bị ép sai khuôn.

---

# Bảng ghi nhớ một trang — Lĩnh vực 5

```
════ 5.1 CONTEXT HỘI THOẠI ═════════════════════════════════════════════════
  3 cơ chế mất tin: tóm tắt tiệm tiến (mất SỐ/NGÀY/%) · lost-in-the-middle
                    · tool tích lũy (40+ trường, cần 5)
  CASE FACTS block  → sự kiện giao dịch sống NGOÀI lịch sử tóm tắt, chèn MỌI prompt
  Cắt tool output   → PostToolUse hook, giữ trường liên quan, cắt TRƯỚC khi tích lũy
  Vị trí            → KEY FINDINGS lên ĐẦU · chi tiết có TIÊU ĐỀ MỤC · action ở CUỐI
  Subagent phải trả METADATA (ngày, nguồn, phương pháp) trong output có cấu trúc
  Phải gửi TOÀN BỘ lịch sử trong request kế tiếp (API không tự nhớ)

════ 5.2 ESCALATION ════════════════════════════════════════════════════════
  ĐÁNG TIN: khách YÊU CẦU RÕ người · chính sách KHÔNG BAO QUÁT · không tiến triển
            · vượt ngưỡng tài chính (cưỡng chế bằng HOOK) · nhiều khớp → HỎI ĐỊNH DANH
  ✗ KHÔNG ĐÁNG TIN: sentiment · self-confidence 1–10 · bộ phân loại tự động
  3 mẫu: yêu cầu rõ → NGAY | vấn đề trong tầm → thử giải quyết trước
         | bực bội → GHI NHẬN + ĐỀ XUẤT, escalation khi khách NHẮC LẠI
  ⚠ bày tỏ bực bội ≠ yêu cầu gặp người
  Handoff phải TỰ CHỨA — người vận hành KHÔNG thấy bản ghi hội thoại

════ 5.3 LAN TRUYỀN LỖI ════════════════════════════════════════════════════
  4 trường: failure_type · attempted_query · partial_results · alternative_approaches
            (+ coverage_impact)
  rỗng hợp lệ = THÀNH CÔNG | timeout/không truy cập được = LỖI cần quyết định retry
  ✗ "search unavailable" chung chung · ém lỗi · hủy cả workflow · retry vô hạn
  Subagent phục hồi CỤC BỘ 1–2 lần → mới lan truyền, kèm partial
  Bản tổng hợp cuối phải CHÚ THÍCH ĐỘ BAO PHỦ (đầy đủ / một phần + lý do)

════ 5.4 CODEBASE LỚN ══════════════════════════════════════════════════════
  Triệu chứng suy giảm: trả lời KHÔNG ỔN ĐỊNH, nói "mẫu điển hình" thay vì LỚP CỤ THỂ
  SCRATCHPAD file → phát hiện then chốt sống qua ranh giới context / session mới
  SUBAGENT       → đọc 15 file, trả 1 dòng; main agent giữ ĐIỀU PHỐI CẤP CAO
                   context tối thiểu · output có cấu trúc · allowedTools hạn chế
                   coordinator = LỚP CONTEXT RIÊNG, chống "rò rỉ context"
  TÓM TẮT giai đoạn trước → TIÊM vào context khởi đầu của subagent giai đoạn sau
  /compact       → nén khi đầy; RỦI RO mất số/ngày chính xác
  CRASH RECOVERY → mỗi agent xuất state ra vị trí biết trước;
                   coordinator nạp MANIFEST khi resume và tiêm vào prompt

════ 5.5 GIÁM SÁT & HIỆU CHỈNH ═════════════════════════════════════════════
  ⚠ 97% tổng thể có thể che giấu 40% lỗi trên MỘT loại tài liệu
  → phân tích theo LOẠI TÀI LIỆU và theo TRƯỜNG trước khi giảm review
  confidence cấp TRƯỜNG → hiệu chỉnh bằng TẬP VALIDATION ĐÃ GÁN NHÃN → định tuyến
    cao + ổn định mọi phân đoạn → tự động | thấp hoặc nguồn mơ hồ → NGƯỜI
  LẤY MẪU NGẪU NHIÊN PHÂN TẦNG ngay cả với đợt tin cậy cao
    → đo tỷ lệ lỗi thực + phát hiện MẪU LỖI MỚI

════ 5.6 PROVENANCE ════════════════════════════════════════════════════════
  claim → source mapping có cấu trúc (URL, tên tài liệu, TRÍCH ĐOẠN) phải được
          BẢO TOÀN + HỢP NHẤT qua tổng hợp, không nén mất
  XUNG ĐỘT: GIỮ CẢ HAI giá trị + nguồn + ngày + PHƯƠNG PHÁP LUẬN
            + conflict_detected → COORDINATOR quyết định, KHÔNG tự chọn
  NGÀY THÁNG bắt buộc → tránh đọc nhầm khác biệt THỜI GIAN thành MÂU THUẪN
  Báo cáo tách mục: đã vững vs ĐANG TRANH CHẤP, giữ diễn đạt gốc của nguồn
  RENDER theo loại: tài chính→BẢNG · tin tức→VĂN XUÔI · kỹ thuật→DANH SÁCH
                    · chuỗi thời gian→THEO TRÌNH TỰ

════ BA TRỤC ═══════════════════════════════════════════════════════════════
  BẢO TOÀN thay vì NÉN          → 5.1 case facts · 5.4 scratchpad · 5.6 provenance
  QUY TẮC TƯỜNG MINH thay vì SUY ĐOÁN → 5.2 (không sentiment, không self-confidence)
  PHÂN ĐOẠN thay vì TỔNG HỢP    → 5.5 (97% che giấu 40%)
```

---

# Tự kiểm tra toàn lĩnh vực (14 câu)

> Che phần **Đáp án** lại, tự trả lời trước rồi mới đối chiếu.

## 5.1 — Context hội thoại

**Câu 1.** Sau nhiều lượt, agent hỗ trợ quên mất số tiền đơn hàng và ngày đặt, chỉ còn nói "khoảng chín chục đô". Nguyên nhân là gì và cơ chế đối ứng đúng?

<details><summary><b>Đáp án</b></summary>

Nguyên nhân: **tóm tắt tiệm tiến** — khi nén lịch sử, **giá trị số, phần trăm, ngày tháng** bị cô đọng thành mô tả mơ hồ ("khoảng", "tầm", "một vài").

Đối ứng: **khối "case facts" bền vững** — trích các sự kiện giao dịch ra **ngoài** lịch sử bị tóm tắt và chèn vào **mọi prompt**:

```
=== CASE FACTS ===
Customer ID: CUST-12345    Order ID: ORD-67890
Order Date: 2025-01-15     Order Amount: $89.99
Issue: Damaged item        Customer Request: Full refund
===
```

Nguyên tắc: **đừng trông cậy vào lịch sử hội thoại để nhớ con số** — con số phải sống ở một lớp riêng.
</details>

**Câu 2.** Kết quả tổng hợp từ 8 subagent rất dài; các phát hiện ở giữa hay bị bỏ qua. Hiệu ứng gì, và hai việc cần làm?

<details><summary><b>Đáp án</b></summary>

**Lost-in-the-middle** — model xử lý tin cậy phần **đầu** và **cuối** của đầu vào dài, nhưng **bỏ sót phần giữa**.

Hai việc:
1. **Đặt bản tóm tắt phát hiện then chốt lên ĐẦU**
2. **Tổ chức phần chi tiết với tiêu đề mục tường minh** (và đặt action items ở cuối)

```
[KEY FINDINGS — đầu]      3 critical vulnerabilities...
[DETAILED RESULTS — giữa] === auth.ts ===  === database.ts ===
[ACTION ITEMS — cuối]     Fix auth.ts before merge.
```
</details>

**Câu 3.** `lookup_order` trả về 40+ trường nhưng luồng xử lý trả hàng chỉ cần 5. Sửa ở đâu, bằng cơ chế nào, và vì sao phải sửa sớm?

<details><summary><b>Đáp án</b></summary>

Sửa ở **PostToolUse hook** — cắt gọn output tool xuống **chỉ các trường liên quan** (`order_id`, `status`, `total`, `items`, `return_eligible`).

Phải sửa **trước khi kết quả tích lũy vào context**, không phải dọn dẹp sau: mỗi lời gọi tool đều cộng thêm output vào context, nên 35 trường thừa × nhiều lượt sẽ ăn hết cửa sổ. Lợi ích kép: **tiết kiệm context** và **giảm nhiễu**.
</details>

---

## 5.2 — Escalation

**Câu 4.** Khách viết: *"This is outrageous, I'm very unhappy with the quality!"*. Escalation ngay hay không? Nếu khách viết *"I want to speak to a manager"* thì sao?

<details><summary><b>Đáp án</b></summary>

- **"This is outrageous..."** → **KHÔNG escalation ngay**. Đây là **bày tỏ bực bội**, không phải yêu cầu gặp người. Mẫu đúng: **ghi nhận cảm xúc** → **đề xuất giải pháp cụ thể** ("I can offer a replacement or a refund") → **chỉ escalation nếu khách nhắc lại** mong muốn gặp người thật.
- **"I want to speak to a manager"** → **escalation NGAY**, không điều tra trước, không "let me try to help first".

Ranh giới: *bày tỏ bực bội* **≠** *yêu cầu rõ ràng gặp người*. Sai cả hai chiều đều bị trừ điểm: escalation vội khi khách mới phàn nàn, hoặc chần chừ khi khách đã nói rõ.
</details>

**Câu 5.** Nhóm đề xuất escalation dựa trên (a) phân tích cảm xúc, (b) model tự chấm độ tự tin 1–10, (c) một bộ phân loại tự động. Đánh giá từng cái. Vậy tác nhân nào mới đáng tin?

<details><summary><b>Đáp án</b></summary>

Cả ba đều **không đáng tin**:

| | Vì sao thất bại |
|---|---|
| (a) Sentiment | Tâm trạng khách **không tương quan** với độ phức tạp vụ việc |
| (b) Self-confidence | Model có thể **sai một cách tự tin**; hiệu chỉnh kém |
| (c) Bộ phân loại | Overengineering; đòi dữ liệu huấn luyện không có sẵn |

Tác nhân **đáng tin**: khách **yêu cầu rõ ràng** gặp người · **chính sách không bao quát** yêu cầu · agent **không thể tiến triển** · thao tác **vượt ngưỡng tài chính** (nên cưỡng chế bằng **hook**) · **nhiều kết quả khớp** → **hỏi thêm định danh**.
</details>

**Câu 6.** Khách đòi so giá với đối thủ; chính sách chỉ nói về điều chỉnh giá trên site của mình. Tra khách hàng thì ra 3 kết quả trùng tên. Xử lý từng việc?

<details><summary><b>Đáp án</b></summary>

- **So giá đối thủ** → **escalation**. Chính sách **im lặng** với yêu cầu cụ thể này (nó chỉ bao quát điều chỉnh giá trên site của mình). Lỗ hổng/ngoại lệ chính sách là tác nhân escalation hợp lệ — agent **không được tự suy diễn** ra chính sách mới.
- **3 kết quả trùng tên** → **hỏi thêm định danh** (email, số điện thoại, mã đơn). **Không** chọn theo heuristic ("chắc là người mới mua gần nhất"), vì đoán sai nghĩa là thao tác lên tài khoản của người khác.
</details>

**Câu 7.** Khi escalation, cần bàn giao những gì cho người xử lý? Vì sao không thể chỉ nói "khách muốn gặp quản lý"?

<details><summary><b>Đáp án</b></summary>

Bàn giao **có cấu trúc, tự chứa**:

```json
{"customer_id", "customer_name", "issue_summary", "order_id", "root_cause",
 "actions_taken": ["Verified customer...", "Confirmed order...",
                   "Offered replacement — customer insists on refund"],
 "refund_amount", "recommended_action", "escalation_reason"}
```

Vì **người vận hành KHÔNG có quyền truy cập bản ghi hội thoại** — họ chỉ thấy bản tóm tắt này. Thiếu `actions_taken` thì họ lặp lại đúng những gì agent đã làm; thiếu `root_cause`/`recommended_action` thì họ phải điều tra lại từ đầu.
</details>

---

## 5.3 — Lan truyền lỗi

**Câu 8.** Một subagent tìm kiếm bị timeout sau khi đã lấy được 3/10 kết quả. Nó nên trả về gì? Và bản báo cáo tổng hợp cuối cùng phải thể hiện điều gì?

<details><summary><b>Đáp án</b></summary>

Trước tiên **phục hồi cục bộ** (1–2 retry). Nếu vẫn hỏng, trả về **context lỗi có cấu trúc**:

```json
{"status": "partial_failure", "failure_type": "timeout",
 "attempted_query": "...", "partial_results": [3 kết quả đã có],
 "alternative_approaches": ["Try a narrower query", "Use alternative source"],
 "coverage_impact": "Not covered: ..."}
```

Tuyệt đối **không** trả mảng rỗng như thể thành công, và **không** hủy cả workflow (mất luôn 3 kết quả đã có).

Báo cáo cuối phải **chú thích độ bao phủ**:
```markdown
### Music (BAO PHỦ MỘT PHẦN — search agent timeout)
⚠️ Note: coverage limited due to a timeout in the search agent.
```
Lỗ hổng phải **hiện ra trong sản phẩm cuối**, không được im lặng biến mất.
</details>

**Câu 9.** Phân biệt: truy vấn chạy xong và không có kết quả nào khớp — vs — tool không truy cập được nguồn. Coordinator phải làm gì khác nhau?

<details><summary><b>Đáp án</b></summary>

| Tình huống | Bản chất | Coordinator làm gì |
|---|---|---|
| Chạy xong, **không có khớp** | **THÀNH CÔNG** — kết quả rỗng hợp lệ | Ghi nhận "không có kết quả", đi tiếp |
| **Không truy cập được** (timeout, 503) | **LỖI truy cập** | Ra **quyết định retry** / đổi nguồn / chú thích lỗ hổng |

Gộp hai thứ này là anti-pattern **"ém lỗi âm thầm"**: coordinator tưởng đã tìm và thực sự không có gì, trong khi thực tế là chưa tìm được. Đây cũng chính là phân biệt ở 2.2 (rỗng-hợp-lệ vs not_found/transient).
</details>

---

## 5.4 — Codebase lớn

**Câu 10.** Sau vài giờ điều tra, model bắt đầu trả lời không nhất quán và nói về "các pattern điển hình" thay vì tên lớp cụ thể đã tìm thấy lúc đầu. Đây là hiện tượng gì? Nêu **ba** cơ chế đối ứng.

<details><summary><b>Đáp án</b></summary>

**Suy giảm context** trong session kéo dài — dấu hiệu nhận diện chính là **dẫn chiếu "mẫu điển hình" thay vì lớp cụ thể đã phát hiện trước đó**.

Ba cơ chế:
1. **File scratchpad** — ghi phát hiện then chốt ra file, dẫn chiếu lại thay vì chạy lại khám phá (sống được cả qua session mới).
2. **Ủy thác subagent** — subagent đọc 15 file, trả về **một dòng**; agent chính giữ **điều phối cấp cao**, context không bị ngập.
3. **Tóm tắt giữa các giai đoạn** — tóm tắt phát hiện giai đoạn trước rồi **tiêm vào context khởi đầu** của subagent giai đoạn sau. (Cộng thêm **`/compact`** khi context đầy — nhưng lưu ý nó **có thể làm mất số liệu và ngày tháng chính xác**, nên scratchpad vẫn cần.)
</details>

**Câu 11.** Workflow nhiều agent chạy 2 tiếng thì crash. Thiết kế thế nào để resume được mà không chạy lại từ đầu?

<details><summary><b>Đáp án</b></summary>

**Lưu trữ trạng thái có cấu trúc:**

- Mỗi agent **xuất trạng thái ra một vị trí đã biết**:
```json
// agent-state/web-search-agent.json
{"status": "completed", "queries_executed": [...], "results_count": 12,
 "key_findings": [...], "coverage": [...], "gaps": [...]}
```
- Coordinator **nạp manifest** khi resume và **tiêm vào prompt của agent**:
```json
// agent-state/manifest.json
{"web-search": "completed", "doc-analysis": "in_progress", "synthesis": "not_started"}
```

Nhờ manifest, coordinator biết chính xác phần nào đã xong, phần nào phải chạy lại.
</details>

---

## 5.5 & 5.6 — Hiệu chỉnh và provenance

**Câu 12.** Hệ thống trích xuất đạt **97% chính xác tổng thể**; nhóm đề xuất bỏ review của con người. Rủi ro gì, và cần làm gì trước khi tự động hóa?

<details><summary><b>Đáp án</b></summary>

Rủi ro: **chỉ số tổng hợp che giấu hiệu năng kém trên phân đoạn cụ thể** — 97% tổng thể có thể đang che **40% lỗi trên một loại tài liệu** nào đó.

Trước khi tự động hóa:
1. **Phân tích độ chính xác theo LOẠI TÀI LIỆU và theo TRƯỜNG**, xác nhận ổn định trên **mọi** phân đoạn.
2. Cho model xuất **confidence cấp trường**, **hiệu chỉnh ngưỡng bằng tập validation đã gán nhãn**.
3. **Định tuyến**: tin cậy cao + ổn định mọi phân đoạn → tự động; tin cậy thấp **hoặc nguồn mơ hồ/mâu thuẫn** → người review.
4. Kể cả sau khi tự động hóa, duy trì **lấy mẫu ngẫu nhiên phân tầng** trên các đợt tin cậy cao để **đo tỷ lệ lỗi thực** và **phát hiện mẫu lỗi mới**.
</details>

**Câu 13.** Hai nguồn uy tín đưa hai con số khác nhau cho cùng một chỉ số (12% và 8%). Agent phân tích tài liệu nên làm gì? Và nếu hai nguồn khác năm xuất bản thì sao?

<details><summary><b>Đáp án</b></summary>

**Không tự chọn một giá trị.** Giữ **cả hai**, kèm **quy kết nguồn + ngày + phương pháp luận**, gắn cờ xung đột, rồi **để coordinator quyết định** cách đối soát:

```json
{"claim": "Share of AI-generated music",
 "values": [{"value":"12%","source":"Spotify Annual Report 2024","date":"2024-03",
             "methodology":"Automated classification"},
            {"value":"8%","source":"Music Industry Association Survey","date":"2024-07",
             "methodology":"Survey of 500 labels"}],
 "conflict_detected": true,
 "possible_explanation": "Difference in methodology and time period"}
```

Agent vẫn **hoàn thành công việc** — chỉ là mang theo cả hai giá trị đã được chú thích, không dừng lại.

Nếu **khác năm**: đây có thể **không phải mâu thuẫn** mà là **thay đổi theo thời gian**. Vì vậy phải bắt subagent luôn đưa **ngày xuất bản / ngày thu thập** vào output:
> Tệ: *"A nói 10%, B nói 15%. Mâu thuẫn."*
> Tốt: *"A (2023) 10%, B (2024) 15% — nhiều khả năng tăng 5% trong một năm."*
</details>

**Câu 14.** Bản tổng hợp cuối gồm số liệu tài chính, tin ngành và phát hiện kỹ thuật. Vì sao không nên ép tất cả về một định dạng? Và làm sao giữ được quy kết nguồn qua các bước tóm tắt?

<details><summary><b>Đáp án</b></summary>

**Render theo loại nội dung:**

| Loại | Trình bày |
|---|---|
| Dữ liệu tài chính | **Bảng** |
| Tin tức, phân tích | **Văn xuôi** |
| Phát hiện kỹ thuật | **Danh sách có cấu trúc** |
| Chuỗi thời gian | **Theo trình tự thời gian** |

Ép về một khuôn duy nhất làm mỗi loại mất thông tin theo một kiểu khác nhau (bảng biến tin tức thành vụn rời; văn xuôi làm số liệu khó đối chiếu).

Giữ quy kết nguồn: bắt subagent xuất **ánh xạ claim → source có cấu trúc** (`source_url`, `source_name`, `publication_date`, **trích đoạn liên quan**, confidence), và agent tổng hợp phải **bảo toàn + hợp nhất** chúng qua từng bước — không được nén mất. Báo cáo cũng nên **tách mục** giữa phát hiện **đã vững** và phát hiện **đang tranh chấp**, giữ nguyên cách diễn đạt gốc và ngữ cảnh phương pháp luận.
</details>
