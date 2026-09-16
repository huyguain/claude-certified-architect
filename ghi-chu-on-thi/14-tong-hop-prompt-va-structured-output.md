# Tổng hợp kiến thức — Lĩnh vực 4: Prompt Engineering & Structured Output (20%)

> Bao phủ **toàn bộ** blueprint LV4 (4.1 → 4.6).
> Nguồn: `../guide_vi.md` Ch.2.4–2.5 (273–326), Ch.6 (823–1036), Ch.7 (1037–1096), Ch.8.3 (1130–1149), blueprint 1786–1867
> Liên quan: [`12-tong-hop-tool-va-mcp.md`](./12-tong-hop-tool-va-mcp.md) (`tool_choice`) · [`13-tong-hop-claude-code-workflow.md`](./13-tong-hop-claude-code-workflow.md) (tinh chỉnh lặp, CI review)

**20% đề thi.** Với ~76 câu → khoảng **15 câu** — ngang Lĩnh vực 3.

---

## Bản đồ 6 subdomain

| # | Chủ đề | Câu hỏi cốt lõi đề hay hỏi | Chương tham chiếu |
|---|---|---|---|
| 4.1 | Tiêu chí tường minh | *Quá nhiều false positive — sửa prompt thế nào?* | Ch.6.2 (908–946) |
| 4.2 | Few-shot prompting | *Output không nhất quán — kỹ thuật nào?* | Ch.6.1 (827–907) |
| 4.3 | Structured output | *Làm sao đảm bảo JSON đúng schema?* | Ch.2.4–2.5 (273–326) |
| 4.4 | Validation & retry | *Retry có cứu được không?* | Ch.6.5–6.6 (988–1036) |
| 4.5 | Batch processing | *Batch hay synchronous? Gửi lúc nào?* | Ch.7 (1037–1096) |
| 4.6 | Review đa instance/đa lượt | *Ai review? Chia lượt thế nào?* | Ch.8.3 (1130–1149) |

**Ba trục xuyên suốt cả lĩnh vực:**
- **Cụ thể thắng mơ hồ** — chi phối 4.1 và 4.2 (tiêu chí phân loại, ví dụ input/output).
- **Cú pháp vs ngữ nghĩa** — chi phối 4.3 và 4.4 (`tool_use` diệt lỗi cú pháp; validation/retry lo lỗi ngữ nghĩa).
- **Độc lập thắng tự đánh giá** — chi phối 4.6 (và nối sang 3.6 CI).

---

# 4.1 — Tiêu chí tường minh để tăng độ chính xác

## Nguyên lý nền

> **Tiêu chí phân loại cụ thể** hiệu quả hơn hẳn **chỉ dẫn chung chung về mức độ thận trọng.**

Câu dặn *"hãy thận trọng"*, *"chỉ báo cáo phát hiện có độ tin cậy cao"* **không** làm giảm false positive — vì model không có thêm thông tin nào để phân loại; nó chỉ đoán lại cùng một thứ với giọng dè dặt hơn.

## Đối chiếu mơ hồ vs tường minh

**Tệ:**
```
Check code comments for accuracy.
Be conservative—report only high-confidence findings.
```

**Tốt:**
```
Flag a comment as problematic ONLY if:
1. The comment describes behavior that CONTRADICTS the actual code behavior
2. The comment references a non-existent function or variable
3. A TODO/FIXME refers to a bug already fixed in code

Do NOT flag:
- Comments merely stylistically outdated
- Comments with minor wording inaccuracies
- Missing comments (separate category)
```

Điểm khác biệt: bản tốt liệt kê **cái gì BÁO CÁO** và **cái gì BỎ QUA** thành các **hạng mục rời rạc**, không dựa vào ngưỡng tin cậy.

## Tiêu chí mức độ nghiêm trọng phải kèm ví dụ code

```
CRITICAL: Runtime failure for users
  Example: NullPointerException while processing a payment
HIGH: Security vulnerability
  Example: SQL injection, XSS, missing authorization checks
MEDIUM: Logic bug without immediate impact
  Example: Wrong sorting, off-by-one error
LOW: Code quality
  Example: Duplication, suboptimal algorithm for small data
```

Không có ví dụ cho từng cấp → phân loại không nhất quán giữa các lần chạy.

## Tác động của false positive lên niềm tin ⚠️

> Một hạng mục có tỷ lệ false-positive cao **làm xói mòn niềm tin vào cả những hạng mục chính xác.**

Hệ quả thực tiễn — chiến thuật hai bước:
1. **Tạm thời vô hiệu hóa** hạng mục nhiễu (ví dụ tắt "comment accuracy") để khôi phục niềm tin ngay.
2. **Cải thiện prompt** cho hạng mục đó ở nền, bật lại khi đã đạt chất lượng.

Đáp án sai thường là "giữ nguyên tất cả nhưng dặn model thận trọng hơn" — không giải quyết được gì.

---

# 4.2 — Few-shot prompting

## Nguyên lý nền

> **Few-shot = 2–4 ví dụ input/output trong prompt.** Đây là kỹ thuật **hiệu quả nhất** khi chỉ dẫn chi tiết bằng văn xuôi vẫn cho ra kết quả không nhất quán.

Vì sao mạnh hơn mô tả bằng chữ:
- Chỉ dẫn mơ hồ ("hãy chính xác hơn") hiểu được theo nhiều cách.
- Một ví dụ cho thấy **không nhập nhằng** cả **định dạng** lẫn **logic ra quyết định**.
- Model **khái quát hóa mẫu sang trường hợp mới** — không chỉ lặp lại đúng các ví dụ đã cho.

> Điểm hay bị hỏi: few-shot giúp model **tổng quát hóa phán đoán**, chứ **không phải** chỉ khớp các ca đã liệt kê sẵn.

## Năm loại ví dụ few-shot (thuộc để nhận diện đề)

| Loại | Dùng khi | Ví dụ trong guide |
|---|---|---|
| **Kịch bản mơ hồ** | Chọn tool / hành động khi yêu cầu nhập nhằng | *"My order is broken"* → `get_customer` → `lookup_order`; *"Get me a manager"* → escalate ngay |
| **Định dạng output** | Output không nhất quán, thiếu trường | `{"location", "issue", "severity", "suggested_fix"}` |
| **Chấp nhận được vs có vấn đề** | Giảm false positive nhưng vẫn tổng quát hóa | `x.active` (bỏ qua) vs `x.active == true` (gắn cờ) |
| **Cấu trúc tài liệu khác nhau** | Trích xuất từ nguồn không đồng nhất | inline citation `(Smith, 2023)` vs bibliography `[1]` |
| **Đo lường không chính thức** | Giảm hallucination khi trích xuất | *"two handfuls of rice"* → `{"amount": "~100g", "precision": "approximate"}` |

**Ví dụ cho kịch bản mơ hồ phải kèm LÝ DO** — không chỉ nêu hành động mà nói rõ **vì sao chọn hành động này thay vì phương án hợp lý khác**:

```
Request: "Get me a manager"
Action: Immediately call escalate_to_human.
Rationale: The customer explicitly requests a human. Do not attempt to solve autonomously.
```

## Quy tắc chuẩn hóa định dạng đi kèm schema

Schema nghiêm ngặt **không** đảm bảo giá trị nhất quán → thêm quy tắc chuẩn hóa vào **prompt**:

```
Normalization:
- Dates: always ISO 8601 (YYYY-MM-DD); "yesterday" → compute an absolute date
- Currency: numeric amount + currency code; "five bucks" → {"amount": 5, "currency": "USD"}
- Percentages: decimal fraction; "half" → 0.5
```

Đây là chỗ 4.2 giao với 4.3: **schema lo cấu trúc, prompt lo giá trị.**

---

# 4.3 — Structured output bằng `tool_use` + JSON Schema

## Nguyên lý nền

> **`tool_use` kèm JSON schema là cách ĐÁNG TIN CẬY NHẤT** để có structured output.

Nó đảm bảo:
- ✅ JSON **hợp lệ về cú pháp** (không thiếu ngoặc, không dấu phẩy dư)
- ✅ **Đúng cấu trúc** yêu cầu (trường bắt buộc đều có mặt)
- ❌ **KHÔNG** đảm bảo **đúng về ngữ nghĩa** — giá trị vẫn có thể sai

## Cú pháp vs ngữ nghĩa ⚠️

| Loại lỗi | Ví dụ | Cách diệt |
|---|---|---|
| **Cú pháp** | JSON không hợp lệ, sai kiểu trường | **`tool_use` + JSON schema — loại bỏ hẳn** |
| **Ngữ nghĩa** | Tổng không khớp, giá trị nằm sai trường, hallucination | **Validation + retry kèm phản hồi + tự sửa lỗi** |

> Đây là bẫy hay gặp nhất của 4.3: đề mô tả "line items không cộng ra tổng" rồi hỏi cách khắc phục — **siết schema chặt hơn KHÔNG giải quyết được**, vì đó là lỗi ngữ nghĩa.

## `tool_choice` trong ngữ cảnh trích xuất

| Giá trị | Hành vi | Dùng khi |
|---|---|---|
| `{"type": "auto"}` | Model **có thể trả về văn bản** thay vì gọi tool | Mặc định — **không** đảm bảo structured output |
| `{"type": "any"}` | **Phải gọi một tool**, tự chọn tool nào | **Nhiều schema trích xuất, chưa biết loại tài liệu** — vẫn chắc chắn có structured output |
| `{"type": "tool", "name": "extract_metadata"}` | **Phải gọi đúng tool đó** | **Ép một bước trích xuất chạy trước** các bước làm giàu |

(Chi tiết đầy đủ ở [`12-tong-hop-tool-va-mcp.md`](./12-tong-hop-tool-va-mcp.md) mục 2.3 — cùng một bảng, hai lĩnh vực đều hỏi.)

## Bốn quy tắc thiết kế schema

```json
{
  "type": "object",
  "properties": {
    "category":        {"type": "string", "enum": ["bug","feature","docs","unclear","other"]},
    "category_detail": {"type": ["string","null"], "description": "Details if category = 'other' or 'unclear'"},
    "severity":        {"type": "string", "enum": ["critical","high","medium","low"]},
    "confidence":      {"type": "number", "minimum": 0, "maximum": 1},
    "optional_field":  {"type": ["string","null"], "description": "Null if not found in the source"}
  },
  "required": ["category", "severity"]
}
```

1. **Bắt buộc vs tùy chọn** — chỉ đánh dấu `required` khi thông tin **luôn có sẵn**. ⚠️ **Trường bắt buộc thúc đẩy model BỊA ra giá trị khi thiếu dữ liệu.**
2. **Trường nullable** — `"type": ["string","null"]` cho thông tin có thể vắng mặt → model trả `null` thay vì hallucinate.
3. **Enum có `"other"` + chuỗi chi tiết** — tránh mất dữ liệu nằm ngoài danh mục định trước (danh mục **mở rộng được**).
4. **Enum có `"unclear"`** — cho ca model không tự tin chọn được. **Một câu trả lời "unclear" trung thực tốt hơn một danh mục sai.**

---

# 4.4 — Validation, retry và vòng lặp phản hồi

## Quy trình ba bước

```
Bước 1: Trích xuất dữ liệu từ tài liệu
Bước 2: Validate (Pydantic / JSON Schema / quy tắc nghiệp vụ)
Bước 3: Nếu lỗi — retry KÈM NGỮ CẢNH:
        · tài liệu gốc
        · bản trích xuất SAI của lần trước
        · lỗi validation CỤ THỂ:
          "Field 'total' = 150, but sum(line_items) = 145. Re-check values."
```

Ba thành phần ở bước 3 là điểm hay bị hỏi — thiếu bản trích xuất sai hoặc thiếu lỗi cụ thể thì retry chỉ là gọi lại y nguyên.

## Khi nào retry cứu được, khi nào không ⚠️

| Retry **hiệu quả** | Retry **KHÔNG giúp** |
|---|---|
| Lỗi **định dạng** (ngày sai format) | **Thông tin không có trong tài liệu nguồn** |
| Lỗi **cấu trúc** (trường đặt sai vị trí) | **Ngữ cảnh cần thiết nằm ở tài liệu khác không được cung cấp** |
| **Không nhất quán số học** (model kiểm tra lại được) | |

> Nguyên tắc: retry chỉ sửa được thứ **model có đủ dữ liệu để tự sửa**. Thiếu dữ liệu nguồn thì retry bao nhiêu lần cũng vô ích — phải **cung cấp thêm tài liệu**, không phải thử lại.

## Mẫu tự sửa lỗi (self-correction)

```json
{
  "stated_total": "$150.00",
  "calculated_total": "$145.00",
  "conflict_detected": true,
  "line_items": [{"name": "Widget A", "price": 75.00},
                 {"name": "Widget B", "price": 70.00}]
}
```

Ý tưởng: bắt model trích xuất **cả giá trị được nêu lẫn giá trị tự tính**, rồi thêm cờ `conflict_detected` → phát hiện mâu thuẫn **ngay trong output**, không cần vòng validate riêng.

Biến thể cùng họ: `conflict_detected` cho **dữ liệu nguồn mâu thuẫn nội tại**.

## `detected_pattern` — vòng lặp phản hồi dài hạn

Thêm trường **`detected_pattern`** vào mỗi finding để ghi lại **cấu trúc code nào đã kích hoạt phát hiện đó**.

Khi lập trình viên **bác bỏ** (dismiss) một finding, bạn có dữ liệu để **phân tích hệ thống mẫu nào hay sinh false positive** → sửa đúng chỗ, thay vì đoán mò. Đây là cầu nối sang 4.1: dữ liệu dismiss cho biết **hạng mục nào cần tắt tạm và cần viết lại tiêu chí**.

## Pydantic (mức cần biết cho đề)

- **Validation cấu trúc** — kiểu, bắt buộc, enum, kiểm trong code **sau khi** nhận JSON
- **Validation ngữ nghĩa** — validator tùy chỉnh: tổng các mục = tổng cuối, `start_date < end_date`
- **Vòng validate–retry** — thất bại → dựng thông báo lỗi → prompt lại kèm ngữ cảnh
- **Sinh JSON Schema** — model Pydantic xuất ra schema cho `tool_use` → **một nguồn chân lý duy nhất**

---

# 4.5 — Batch processing

## Thông số Message Batches API — thuộc lòng

| Thuộc tính | Giá trị |
|---|---|
| Tiết kiệm chi phí | **50%** so với gọi đồng bộ |
| Cửa sổ xử lý | **Lên đến 24 giờ** — **KHÔNG có cam kết SLA về latency** |
| Tool calling nhiều lượt | **KHÔNG hỗ trợ** — một yêu cầu = một phản hồi |
| Tương quan request/response | Trường **`custom_id`** |

> "Không có SLA" nghĩa là **không được hứa thời điểm xong** — chỉ biết trần 24 giờ. Đáp án nào nói "batch đảm bảo xong trong X giờ" đều sai.

## Batch hay Synchronous

| Tác vụ | API | Vì sao |
|---|---|---|
| Kiểm tra PR **trước khi merge** | **Synchronous** | Lập trình viên **đang chờ** — 24 giờ không chấp nhận được |
| Báo cáo nợ kỹ thuật **qua đêm** | **Batch** | Cần vào sáng hôm sau; tiết kiệm 50% |
| Kiểm toán bảo mật **hằng tuần** | **Batch** | Không khẩn cấp |
| Code review **tương tác** | **Synchronous** | Cần phản hồi tức thì |
| Xử lý **10.000 tài liệu** | **Batch** | Khối lượng lớn, tiết kiệm đáng kể |

> Tiêu chí duy nhất: **có ai đang chờ để đi tiếp không?** Chặn (blocking) → synchronous. Không chặn, chịu được độ trễ → batch.

## Tính nhịp gửi batch theo SLA

Cần kết quả trong **30 giờ**, batch có thể mất tới **24 giờ**:
- Cửa sổ gửi = 30 − 24 = **6 giờ**
- Batch phải được gửi **không muộn hơn 24 giờ trước hạn chót**
- Để gửi thường xuyên: chia thành các **cửa sổ 4 giờ** (nằm an toàn trong 6 giờ)

Công thức: **cửa sổ gửi = SLA − 24h**, rồi chọn nhịp gửi **nhỏ hơn** cửa sổ đó.

## Xử lý thất bại trong batch

```
1. Gửi batch 100 tài liệu
2. 95 thành công, 5 thất bại (vượt giới hạn context)
3. Xác định tài liệu lỗi bằng custom_id
4. ĐIỀU CHỈNH chiến lược — ví dụ chia tài liệu dài thành đoạn nhỏ
5. Gửi lại CHỈ 5 tài liệu lỗi
```

Hai điểm hay bị hỏi: **chỉ gửi lại phần thất bại** (nhờ `custom_id`) và **phải điều chỉnh** chứ không gửi lại y nguyên.

**Trước khi chạy khối lượng lớn:** tinh chỉnh prompt trên **một mẫu nhỏ** → tối đa hóa tỷ lệ thành công ngay lần đầu, giảm chi phí gửi lại nhiều vòng.

---

# 4.6 — Review đa instance & đa lượt

## Hạn chế của tự review ⚠️

> Model **giữ lại context suy luận từ lúc sinh code**, nên **ít có khả năng thách thức chính quyết định của mình** trong cùng session.

Cách khắc phục đúng: **một instance Claude thứ hai, độc lập, không có context sinh ra**.

Cách khắc phục **sai** (hay xuất hiện làm đáp án nhiễu): dặn model "hãy tự review kỹ hơn", bật extended thinking, thêm một lượt tự kiểm tra trong cùng session. Tất cả đều **không** gỡ được thiên lệch, vì context suy luận vẫn còn nguyên.

> Đây là cùng nguyên tắc với **cô lập context session** ở 3.6 (CI review). Hai lĩnh vực, một khái niệm.

## Review đa lượt cho PR lớn

Với PR 10+ file:

```
Pass 1 (per-file): auth.ts     → liệt kê vấn đề CỤC BỘ
Pass 1 (per-file): database.ts → liệt kê vấn đề CỤC BỘ
Pass 1 (per-file): routes.ts   → liệt kê vấn đề CỤC BỘ
...
Pass 2 (integration): phân tích QUAN HỆ giữa các file
  → kiểu dữ liệu không nhất quán, phụ thuộc vòng, luồng dữ liệu xuyên file
```

**Vì sao một lượt duy nhất qua 14 file là tệ:**
- **Pha loãng chú ý** — phân tích sâu vài file, hời hợt phần còn lại
- **Nhận xét không nhất quán** — một mẫu bị gắn cờ ở file này, được chấp nhận ở file khác
- **Bỏ sót lỗi** — lỗi hiển nhiên bị bỏ qua vì quá tải nhận thức

> Nhớ cấu trúc **hai loại lượt**: lượt **cục bộ theo từng file** + lượt **tích hợp xuyên file**. Đáp án chỉ chia nhỏ theo file mà bỏ lượt tích hợp sẽ **mất hết lỗi xuyên file**.

## Lượt xác minh có độ tin cậy

Chạy lượt xác minh trong đó model **tự báo độ tin cậy kèm mỗi finding** → dùng để **định tuyến review một cách được hiệu chỉnh** (finding tin cậy thấp đưa cho người xem, tin cậy cao tự động hóa).

⚠️ Lưu ý tương phản với LV5: model **tự đánh giá độ tin cậy là đại lượng thay thế KHÔNG đáng tin** cho **độ phức tạp của vụ việc** khi quyết định escalation. Cùng một cơ chế, nhưng dùng để **định tuyến review** thì chấp nhận được, dùng để **thay thế tiêu chí escalation** thì sai.

---

# Bảng ghi nhớ một trang — Lĩnh vực 4

```
════ 4.1 TIÊU CHÍ TƯỜNG MINH ═══════════════════════════════════════════════
  "hãy thận trọng" / "chỉ báo cáo high-confidence"  → KHÔNG giảm false positive
  ĐÚNG: liệt kê hạng mục BÁO CÁO vs BỎ QUA, rời rạc, cụ thể
  Severity phải kèm VÍ DỤ CODE cho từng cấp (CRITICAL/HIGH/MEDIUM/LOW)
  ⚠ Một hạng mục nhiễu làm XÓI MÒN NIỀM TIN vào cả hạng mục chính xác
    → TẮT TẠM hạng mục đó + sửa prompt ở nền, rồi bật lại

════ 4.2 FEW-SHOT ══════════════════════════════════════════════════════════
  2–4 ví dụ input/output = kỹ thuật HIỆU QUẢ NHẤT khi văn xuôi cho kết quả lệch
  Model KHÁI QUÁT HÓA sang ca mới, không chỉ khớp ca đã liệt kê
  5 loại: ca mơ hồ (KÈM LÝ DO) · định dạng output · chấp nhận-được vs có-vấn-đề
          · cấu trúc tài liệu khác nhau · đo lường không chính thức (chống hallucination)
  Chuẩn hóa định dạng để trong PROMPT: ISO 8601 · amount+currency · 0.5 thay "half"
    → schema lo CẤU TRÚC, prompt lo GIÁ TRỊ

════ 4.3 STRUCTURED OUTPUT ═════════════════════════════════════════════════
  tool_use + JSON schema = ĐÁNG TIN CẬY NHẤT
    ✅ diệt lỗi CÚ PHÁP + ép đúng cấu trúc   ❌ KHÔNG diệt lỗi NGỮ NGHĨA
  auto → có thể trả VĂN BẢN | any → phải gọi MỘT tool | forced → đúng tool X
  any  = nhiều schema, CHƯA BIẾT loại tài liệu
  4 quy tắc schema:
    required chỉ khi LUÔN có → required ép model BỊA
    nullable ["string","null"] → trả null thay vì hallucinate
    enum + "other" + chuỗi chi tiết → danh mục MỞ RỘNG được
    enum "unclear" → thà thú nhận còn hơn phân loại sai

════ 4.4 VALIDATION & RETRY ════════════════════════════════════════════════
  Retry phải kèm 3 thứ: tài liệu GỐC + bản trích xuất SAI + LỖI CỤ THỂ
  Retry CỨU được: sai format · sai vị trí trường · lệch số học
  Retry VÔ ÍCH : thông tin KHÔNG CÓ trong nguồn · ngữ cảnh ở tài liệu khác
  Tự sửa lỗi: stated_total + calculated_total + conflict_detected
  detected_pattern → phân tích mẫu nào hay bị dismiss → sửa tiêu chí (nối 4.1)
  Pydantic: cấu trúc + ngữ nghĩa + vòng validate-retry + sinh schema cho tool_use

════ 4.5 BATCH ═════════════════════════════════════════════════════════════
  50% rẻ hơn · tới 24h · KHÔNG CAM KẾT SLA · KHÔNG tool-calling nhiều lượt
  custom_id → nối request↔response, gửi lại CHỈ phần lỗi
  Tiêu chí chọn: CÓ AI ĐANG CHỜ KHÔNG? chặn → synchronous | không chặn → batch
    pre-merge/review tương tác → SYNC | qua đêm, hằng tuần, 10k tài liệu → BATCH
  Cửa sổ gửi = SLA − 24h   (SLA 30h → cửa sổ 6h → gửi theo nhịp 4h)
  Thất bại: nhận diện bằng custom_id → ĐIỀU CHỈNH (chunk) → gửi lại chỉ phần lỗi
  Tinh chỉnh prompt trên MẪU NHỎ trước khi chạy khối lượng lớn

════ 4.6 REVIEW ĐA INSTANCE / ĐA LƯỢT ══════════════════════════════════════
  Tự review kém vì GIỮ CONTEXT SUY LUẬN → dùng INSTANCE THỨ HAI ĐỘC LẬP
    ✗ nhiễu: "dặn model tự review kỹ hơn", bật extended thinking, thêm lượt tự kiểm
  PR 10+ file: Pass 1 CỤC BỘ từng file + Pass 2 TÍCH HỢP xuyên file
    một lượt duy nhất → pha loãng chú ý · nhận xét mâu thuẫn · bỏ sót lỗi
  Lượt xác minh có self-reported confidence → ĐỊNH TUYẾN review
    ⚠ nhưng self-confidence KHÔNG thay được tiêu chí escalation (LV5)

════ BA TRỤC NỐI CẢ LĨNH VỰC ═══════════════════════════════════════════════
  CỤ THỂ thắng MƠ HỒ          → 4.1 tiêu chí · 4.2 ví dụ
  CÚ PHÁP vs NGỮ NGHĨA        → 4.3 schema diệt cú pháp · 4.4 validation diệt ngữ nghĩa
  ĐỘC LẬP thắng TỰ ĐÁNH GIÁ   → 4.6 (và 3.6 CI review)
```

---

# Tự kiểm tra toàn lĩnh vực (14 câu)

> Che phần **Đáp án** lại, tự trả lời trước rồi mới đối chiếu.

## 4.1 — Tiêu chí tường minh

**Câu 1.** Bot review sinh quá nhiều false positive ở hạng mục "comment accuracy". Vì sao thêm câu *"be conservative, only report high-confidence findings"* không giúp ích? Làm gì mới đúng?

<details><summary><b>Đáp án</b></summary>

Không giúp ích vì chỉ dẫn đó **không cung cấp thêm thông tin phân loại nào** — model vẫn đoán lại cùng một thứ, chỉ với giọng dè dặt hơn. "Độ tin cậy" là thang tự đánh giá, không phải tiêu chí.

Đúng: viết **tiêu chí phân loại cụ thể**, liệt kê rõ **cái gì BÁO CÁO** và **cái gì BỎ QUA**:

```
Flag ONLY if: comment CONTRADICTS actual behavior / references non-existent
              function / TODO refers to an already-fixed bug
Do NOT flag: stylistically outdated · minor wording · missing comments
```
</details>

**Câu 2.** Một hạng mục có tỷ lệ false-positive rất cao, các hạng mục khác thì tốt. Chiến thuật hai bước là gì, và vì sao không để nguyên?

<details><summary><b>Đáp án</b></summary>

1. **Tạm thời vô hiệu hóa** hạng mục nhiễu → khôi phục niềm tin của lập trình viên ngay lập tức.
2. **Cải thiện prompt** cho hạng mục đó ở nền, bật lại khi đã đạt chất lượng.

Không để nguyên vì **một hạng mục nhiễu làm xói mòn niềm tin vào cả những hạng mục chính xác** — lập trình viên bắt đầu bỏ qua toàn bộ output, kể cả phần đúng.
</details>

---

## 4.2 — Few-shot prompting

**Câu 3.** Chỉ dẫn chi tiết bằng văn xuôi vẫn cho ra output định dạng lung tung. Kỹ thuật nào hiệu quả nhất, bao nhiêu ví dụ, và vì sao nó mạnh hơn mô tả?

<details><summary><b>Đáp án</b></summary>

**Few-shot prompting** — **2–4 ví dụ input/output**.

Mạnh hơn vì: chỉ dẫn mơ hồ hiểu được theo nhiều cách, còn một ví dụ cho thấy **không nhập nhằng** cả **định dạng** lẫn **logic ra quyết định**. Quan trọng: model **khái quát hóa mẫu sang trường hợp mới**, không chỉ lặp lại đúng các ví dụ đã cho.
</details>

**Câu 4.** Ví dụ few-shot cho một **kịch bản mơ hồ** khác gì ví dụ cho **định dạng output**? Thành phần bắt buộc của loại thứ nhất là gì?

<details><summary><b>Đáp án</b></summary>

- **Định dạng output** — chỉ cần cho thấy hình dạng kết quả: `{"location", "issue", "severity", "suggested_fix"}`.
- **Kịch bản mơ hồ** — phải kèm **LÝ DO (rationale)**: nói rõ **vì sao chọn hành động này thay vì phương án hợp lý khác**.

```
Request: "Get me a manager"
Action: Immediately call escalate_to_human.
Rationale: The customer explicitly requests a human. Do not attempt to solve autonomously.
```

Thiếu rationale thì model học được hành động nhưng không học được **ranh giới quyết định**.
</details>

**Câu 5.** Trích xuất bị bỏ trống (null) các trường bắt buộc vì tài liệu nguồn có định dạng rất khác nhau. Nêu **hai** biện pháp khác tầng nhau.

<details><summary><b>Đáp án</b></summary>

- **Tầng prompt** — thêm **ví dụ few-shot cho từng cấu trúc tài liệu**: inline citation `(Smith, 2023)` vs bibliography `[1]`; phần methodology vs chi tiết nhúng trong thân bài. Few-shot đặc biệt mạnh cho các dạng không chuẩn (đo lường không chính thức: *"two handfuls"* → `{"amount": "~100g", "precision": "approximate"}`).
- **Tầng schema** — xem lại có nên để trường đó **`required`** không. Trường bắt buộc mà nguồn không có sẽ **ép model bịa**; chuyển sang **nullable** nếu thông tin có thể vắng mặt thật.

Hai biện pháp giải quyết hai nguyên nhân khác nhau: *model không biết đọc dạng này* vs *model bị ép phải điền*.
</details>

---

## 4.3 — Structured output

**Câu 6.** Kết quả trích xuất hóa đơn có `total` không bằng tổng `line_items`. Siết schema chặt hơn có giải quyết được không? Vì sao?

<details><summary><b>Đáp án</b></summary>

**Không.** Đây là **lỗi NGỮ NGHĨA**, còn `tool_use` + JSON schema chỉ diệt **lỗi CÚ PHÁP** và ép đúng **cấu trúc** — nó không kiểm tra giá trị có đúng hay không.

| Loại lỗi | Cách diệt |
|---|---|
| Cú pháp (JSON hỏng, sai kiểu) | `tool_use` + schema — loại bỏ hẳn |
| Ngữ nghĩa (tổng lệch, sai trường, hallucinate) | Validation + retry kèm phản hồi + tự sửa lỗi |

Giải pháp đúng: validation sau trích xuất + retry kèm lỗi cụ thể, hoặc mẫu `stated_total` / `calculated_total` / `conflict_detected`.
</details>

**Câu 7.** Có nhiều schema trích xuất, không biết trước tài liệu thuộc loại nào, nhưng bắt buộc phải nhận structured output. `tool_choice` nào?

<details><summary><b>Đáp án</b></summary>

**`tool_choice: {"type": "any"}`** — model **phải gọi một tool** (nên chắc chắn có structured output) nhưng **tự chọn** schema phù hợp với loại tài liệu.

Không dùng `auto` (model có thể trả về văn bản thuần). Không dùng forced (bạn chưa biết nên ép schema nào).
</details>

**Câu 8.** Model liên tục bịa giá trị cho các trường mà tài liệu nguồn không hề có. Sửa schema thế nào? Nêu thêm hai kỹ thuật enum liên quan.

<details><summary><b>Đáp án</b></summary>

Nguyên nhân: các trường đó đang là **`required`** — **trường bắt buộc thúc đẩy model bịa ra giá trị khi thiếu dữ liệu**.

Sửa: chuyển sang **nullable** — `"type": ["string", "null"]`, mô tả rõ *"Null if the information was not found in the source"*. Chỉ giữ `required` cho thông tin **luôn có sẵn**.

Hai kỹ thuật enum:
- **`"other"` + chuỗi chi tiết** (`category_detail`) → danh mục **mở rộng được**, không mất dữ liệu nằm ngoài danh sách định trước.
- **`"unclear"`** → cho ca model không tự tin phân loại. **Một câu trả lời "unclear" trung thực tốt hơn một danh mục sai.**
</details>

---

## 4.4 — Validation & retry

**Câu 9.** Retry lần nào cũng thất bại: tài liệu cần trích ngày hiệu lực hợp đồng, nhưng ngày đó chỉ nằm trong phụ lục **không được cung cấp**. Nêu ranh giới giữa retry hiệu quả và vô ích.

<details><summary><b>Đáp án</b></summary>

Đây là ca **retry vô ích** — thông tin **không tồn tại trong nguồn đã cho**. Thử lại bao nhiêu lần cũng không sinh ra dữ liệu; chỉ làm tăng nguy cơ model **bịa** để lấp chỗ trống.

| Retry **hiệu quả** | Retry **vô ích** |
|---|---|
| Sai định dạng (ngày sai format) | Thông tin **không có** trong tài liệu nguồn |
| Sai cấu trúc (trường đặt sai vị trí) | Ngữ cảnh nằm ở **tài liệu khác không được cung cấp** |
| Lệch số học (model tự kiểm lại được) | |

Nguyên tắc: retry chỉ sửa được thứ **model có đủ dữ liệu để tự sửa**. Ở đây phải **cung cấp thêm tài liệu**, không phải thử lại.
</details>

**Câu 10.** Prompt retry cần chứa những gì? Và nêu mẫu schema cho phép phát hiện mâu thuẫn ngay trong output.

<details><summary><b>Đáp án</b></summary>

Retry phải kèm **ba thứ**:
1. **Tài liệu gốc**
2. **Bản trích xuất SAI của lần trước**
3. **Lỗi validation CỤ THỂ** — `"Field 'total' = 150, but sum(line_items) = 145. Re-check values."`

Thiếu (2) hoặc (3) thì retry chỉ là gọi lại y nguyên.

Mẫu tự sửa lỗi:
```json
{"stated_total": "$150.00", "calculated_total": "$145.00",
 "conflict_detected": true, "line_items": [...]}
```
Bắt model trích **cả giá trị được nêu lẫn giá trị tự tính** + cờ `conflict_detected` → phát hiện mâu thuẫn ngay trong output.
</details>

**Câu 11.** Lập trình viên hay bác bỏ một số loại finding nhất định nhưng bạn không biết loại nào. Trường nào cần thêm, và nó phục vụ việc gì?

<details><summary><b>Đáp án</b></summary>

Thêm trường **`detected_pattern`** vào mỗi finding — ghi lại **cấu trúc code nào đã kích hoạt phát hiện đó**.

Khi có dữ liệu dismiss, bạn **phân tích hệ thống** được mẫu nào hay sinh false positive, thay vì đoán mò. Kết quả phân tích quay lại nuôi 4.1: biết **hạng mục nào cần tắt tạm** và **tiêu chí nào cần viết lại**.
</details>

---

## 4.5 — Batch processing

**Câu 12.** Phân loại: (a) kiểm tra PR trước khi merge; (b) báo cáo nợ kỹ thuật chạy qua đêm; (c) sinh test hằng đêm; (d) code review tương tác. Cái nào batch, cái nào synchronous? Tiêu chí duy nhất là gì?

<details><summary><b>Đáp án</b></summary>

| | API | Vì sao |
|---|---|---|
| (a) pre-merge check | **Synchronous** | Lập trình viên **đang chờ** để merge — 24 giờ không chấp nhận được |
| (b) báo cáo qua đêm | **Batch** | Cần vào sáng hôm sau; tiết kiệm 50% |
| (c) sinh test hằng đêm | **Batch** | Không chặn ai |
| (d) review tương tác | **Synchronous** | Cần phản hồi tức thì |

Tiêu chí duy nhất: **có ai đang chờ để đi tiếp không?** Chặn → synchronous. Chịu được độ trễ → batch.

Nhớ thêm: batch **không có cam kết SLA** (chỉ biết trần 24 giờ) và **không hỗ trợ tool calling nhiều lượt** trong một yêu cầu.
</details>

**Câu 13.** SLA nội bộ là 30 giờ. Tính cửa sổ gửi batch và nhịp gửi hợp lý. Batch 100 tài liệu có 5 cái lỗi vì vượt context — xử lý thế nào?

<details><summary><b>Đáp án</b></summary>

**Cửa sổ gửi = SLA − 24h = 30 − 24 = 6 giờ.** Batch phải được gửi **không muộn hơn 24 giờ trước hạn chót**. Để gửi thường xuyên, chia thành **cửa sổ 4 giờ** — nằm an toàn trong 6 giờ.

Xử lý thất bại:
1. Xác định 5 tài liệu lỗi bằng **`custom_id`**
2. **Điều chỉnh chiến lược** — chia tài liệu dài thành đoạn nhỏ (chunk)
3. **Gửi lại chỉ 5 tài liệu đó**, không gửi lại 95 cái đã thành công

Phòng ngừa: **tinh chỉnh prompt trên một mẫu nhỏ** trước khi chạy khối lượng lớn, để tối đa tỷ lệ thành công ngay lần đầu.
</details>

---

## 4.6 — Review đa instance & đa lượt

**Câu 14.** Claude vừa sinh một module. Bạn muốn bắt lỗi tinh tế. Vì sao dặn chính nó "tự review kỹ hơn" không hiệu quả? Và với PR 14 file thì kiến trúc review đúng là gì?

<details><summary><b>Đáp án</b></summary>

**Tự review kém** vì model **giữ lại context suy luận từ lúc sinh code** — nó đã tự thuyết phục mình rằng các quyết định là đúng, nên **ít có khả năng thách thức chính quyết định đó**. Dặn "review kỹ hơn", bật extended thinking, hay thêm một lượt tự kiểm trong cùng session đều **không gỡ được thiên lệch**.

Đúng: dùng **một instance Claude thứ hai, độc lập, không có context sinh ra**. (Cùng nguyên tắc "cô lập context session" ở 3.6.)

Với PR 14 file — **review đa lượt**:
```
Pass 1 (per-file)    : từng file → vấn đề CỤC BỘ
Pass 2 (integration) : quan hệ giữa các file → kiểu không nhất quán,
                       phụ thuộc vòng, luồng dữ liệu xuyên file
```

Một lượt duy nhất qua 14 file gây: **pha loãng chú ý** · **nhận xét không nhất quán** (một mẫu bị gắn cờ ở file này, chấp nhận ở file khác) · **bỏ sót lỗi hiển nhiên** do quá tải nhận thức.

Bỏ Pass 2 thì mất hết lỗi xuyên file — đây là nửa hay bị quên.
</details>
