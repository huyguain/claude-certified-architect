# Tổng hợp kiến thức — Lĩnh vực 2: Thiết kế Tool & Tích hợp MCP (18%)

> Bao phủ **toàn bộ** blueprint LV2 (2.1 → 2.5), không chỉ phần đã sai.
> Kế hoạch vá lỗ hổng từ 6 câu sai: [`10-ke-hoach-lanh-vuc-2.md`](./10-ke-hoach-lanh-vuc-2.md)
> Nguồn: `../guide_vi.md` Ch.2 (218–326), Ch.4 (467–567), Ch.10 (1251–1312), Ch.13 (1498–1531), blueprint 1628–1698

**18% đề thi.** Với ~76 câu → khoảng **13–14 câu**.

---

## Bản đồ 5 subdomain

| # | Chủ đề | Câu hỏi cốt lõi đề hay hỏi | Chương tham chiếu |
|---|---|---|---|
| 2.1 | Mô tả & ranh giới tool | *Tại sao model chọn sai tool? Sửa bằng cách nào?* | Ch.2.2 (226–258) |
| 2.2 | Phản hồi lỗi có cấu trúc | *Cái này là lỗi hay là kết quả rỗng? Có retry không?* | Ch.4.4 (525–554), Ch.10 (1251–1294) |
| 2.3 | Phân bổ tool & `tool_choice` | *Agent nào được cầm tool nào? Làm sao ép thứ tự?* | Ch.2.3 (259–272) |
| 2.4 | Tích hợp MCP server | *Cấu hình ở đâu? Dùng có sẵn hay tự xây?* | Ch.4.3–4.5 (486–567) |
| 2.5 | Tool tích hợp sẵn | *Grep hay Glob? Edit hay Read+Write?* | Ch.13 (1498–1531) |

---

# 2.1 — Thiết kế giao diện tool: mô tả rõ ràng và ranh giới

## Nguyên lý nền

> **Mô tả tool là CƠ CHẾ CHÍNH mà LLM dùng để chọn tool.**
> Không phải tên tool. Không phải thứ tự khai báo. Không phải schema. Là **mô tả**.

Hệ quả trực tiếp: mô tả tối giản (`"Truy xuất thông tin khách hàng"`) → chọn tool **không đáng tin cậy** khi có nhiều tool na ná nhau.

## Một mô tả tool đầy đủ phải có 4 thành phần

| Thành phần | Nội dung | Thiếu thì sao |
|---|---|---|
| **Làm gì / trả về gì** | Hành vi + hình dạng output | Model không biết output có dùng được không |
| **Định dạng đầu vào + ví dụ** | `email (format: user@domain.com)` hoặc `numeric customer_id` | Model gọi sai tham số → validation error |
| **Edge case & ràng buộc** | Giới hạn, trường hợp biên | Model gọi trong tình huống tool không xử lý được |
| **Khi nào dùng cái này vs cái tương tự** | So sánh trực tiếp với tool anh em | **Misrouting** — nguyên nhân số 1 |

Ví dụ chuẩn trong guide (dòng 230):
```
"Finds a customer by email or ID. Returns the customer profile, including name,
 email, order history, and account status. Use this tool BEFORE lookup_order to
 verify the customer's identity. Accepts an email (format: user@domain.com) or
 a numeric customer_id."
```
→ Có đủ: hành vi · output · **thứ tự dùng** · định dạng đầu vào + ví dụ.

## Ba bệnh và ba cách chữa

| Bệnh | Triệu chứng trong đề | Cách chữa |
|---|---|---|
| **Mô tả chồng lấn** | `analyze_content` vs `analyze_document` mô tả gần giống hệt → model nhầm | **Đổi tên + viết lại mô tả theo miền cụ thể**: `analyze_content` → `extract_web_results` với mô tả chỉ nói về web |
| **Tool quá đa năng** | Một `analyze_document` làm mọi thứ | **Tách thành tool chuyên biệt** có hợp đồng I/O riêng: `extract_data_points` · `summarize_content` · `verify_claim_against_source` |
| **System prompt bẻ lái** | Thêm *"If in doubt, use the search tool"* → model gọi `search_web` cả khi tool khác hợp hơn | **Rà soát system prompt** tìm chỉ dẫn nhạy keyword; bỏ hoặc thu hẹp điều kiện |

## Điểm dễ bị hỏi mẹo ⚠️

**Chỉ dẫn trong system prompt có thể LẤN ÁT mô tả tool.** Dù mô tả tool viết hoàn hảo, một câu dặn dò thường trực kiểu *"khi nghi ngờ hãy dùng X"* vẫn biến X thành mặc định trên mọi lượt.

→ Khi đề hỏi *"tại sao model cứ gọi tool X?"*, xét **hai nguồn**: (a) mô tả tool, (b) **system prompt**. Đáp án sai hay gán cho "quy tắc đặt tên" hoặc "thứ tự khai báo tool" — **không phải cơ chế thật**.

## Built-in tool lấn át MCP tool

Agent có xu hướng **ưu tiên built-in** (Read, Grep) hơn MCP tool cùng chức năng.
**Khắc phục:** làm mạnh mô tả MCP tool — nhấn **ưu thế cụ thể, dữ liệu độc nhất, context mà built-in không thể cung cấp**. (Đây là điểm giao giữa 2.1 và 2.4.)

---

# 2.2 — Phản hồi lỗi có cấu trúc cho MCP tool

## Cờ `isError`

MCP tool báo thất bại bằng `isError: true` trong phản hồi. Đây là tín hiệu để agent biết lệnh gọi đã hỏng.

## Phân biệt sống còn: rỗng-hợp-lệ vs không-tồn-tại ⚠️

| Tình huống | Bản chất | Phản hồi |
|---|---|---|
| Khách hàng **tồn tại**, có **0 đơn hàng** | **THÀNH CÔNG** — tập rỗng hợp lệ | Danh sách rỗng, **không** `isError` |
| Customer ID **không tồn tại** | **LỖI** — tài nguyên không tìm thấy | `isError: true` + `errorCategory: not_found_error` + message mô tả |
| Tìm kiếm **thất bại** (timeout) nhưng trả về mảng rỗng | **Ém lỗi âm thầm** — anti-pattern | Phải là `isError` + transient |

> **"Không có gì khớp" ≠ "cái được hỏi không tồn tại" ≠ "truy vấn thất bại".**
> Vế 1 là câu trả lời. Vế 2, 3 là lỗi — và là **hai loại lỗi khác nhau**.

⚠️ `not_found_error` **không có trong `guide_vi.md`** — phải nhớ trực tiếp.

## Bốn nhóm lỗi (Ch.10.1, dòng 1253)

| Nhóm | Ví dụ | Retry? | Hành động của agent |
|---|---|---|---|
| **Transient** | Timeout, 503, lỗi mạng | ✅ Có | Retry với **exponential backoff** |
| **Validation** | Sai định dạng, thiếu trường bắt buộc | ❌ Không | **Sửa input** rồi gọi lại |
| **Business** | Vi phạm chính sách, vượt ngưỡng (hoàn tiền > $500) | ❌ Không | **Giải thích cho người dùng**, đề xuất phương án |
| **Permission** | Bị từ chối truy cập | ❌ Không | **Escalation** |

Mẹo nhớ: chỉ **transient** mới retry. Ba nhóm còn lại mỗi nhóm có một hành động khác nhau — đề hay hỏi *"agent nên làm gì tiếp theo"*, và đáp án đúng bám theo cột cuối.

**Business error cần thêm:** `retryable: false` **+ giải thích thân thiện hướng người dùng** — để agent truyền đạt đúng thay vì im lặng thử lại.

## Lỗi có cấu trúc vs lỗi chung chung

**Tốt:**
```json
{"isError": true, "content": {
  "errorCategory": "transient",
  "isRetryable": true,
  "message": "The service is temporarily unavailable. Timeout while calling the orders API.",
  "attempted_query": "order_id=12345",
  "partial_results": null}}
```

**Anti-pattern:** `{"isError": true, "content": "Operation failed"}`

Vì sao tệ dù `isError` đã đúng: agent **không có thông tin nào** để quyết định — retry? đổi truy vấn? escalation? Lỗi đồng nhất triệt tiêu khả năng phục hồi.

## Bốn anti-pattern xử lý lỗi (Ch.10.2, dòng 1262)

| Anti-pattern | Vấn đề | Đúng là |
|---|---|---|
| Trạng thái chung chung "search unavailable" | Coordinator không quyết định được | Trả về **loại lỗi + query + kết quả một phần + phương án thay thế** |
| Ém lỗi âm thầm (rỗng = thành công) | Coordinator tưởng không có kết quả khớp | **Phân biệt "không có kết quả" với "tìm kiếm thất bại"** |
| Hủy cả workflow khi một bước hỏng | Mất toàn bộ kết quả một phần | Tiếp tục với **partial results**, chú thích lỗ hổng |
| Retry vô hạn trong subagent | Latency, lãng phí | **Phục hồi cục bộ 1–2 lần**, rồi lan truyền lên coordinator |

## Phục hồi cục bộ vs lan truyền

> **Subagent tự xử lý lỗi transient tại chỗ (1–2 retry).**
> **Chỉ lan truyền lên coordinator những gì không tự giải quyết được** — kèm **kết quả một phần** và **đã thử những gì**.

Mẫu lỗi subagent có cấu trúc (Ch.10.3, dòng 1271):
```json
{"status": "partial_failure", "failure_type": "timeout",
 "attempted_query": "AI impact on music industry 2024",
 "partial_results": [...],
 "alternative_approaches": ["Try a narrower query: ...", "Use an alternative data source"],
 "coverage_impact": "Not covered: AI impact on music production"}
```
Bốn trường này cho coordinator đủ dữ kiện để chọn: retry query đã sửa · dùng partial · giao subagent khác · bỏ qua và chú thích lỗ hổng.

---

# 2.3 — Phân bổ tool giữa các agent & cấu hình `tool_choice`

## Hai tầng tác động lên việc chọn tool ⚠️

| Tầng | Cơ chế | Mức đảm bảo |
|---|---|---|
| **Ảnh hưởng (xác suất)** | Mô tả tool, tên tool, chỉ dẫn system prompt | **Chỉ làm lệch xác suất** — không bao giờ 100% |
| **Đảm bảo (tất định)** | `tool_choice` | **Bắt buộc** |

## Bảng `tool_choice` — thuộc lòng

| Giá trị | Model bắt buộc làm gì | Dùng khi |
|---|---|---|
| `{"type": "auto"}` | Tự quyết: gọi tool **hoặc** trả lời văn bản | Mặc định cho hầu hết trường hợp |
| `{"type": "any"}` | **Phải gọi một tool nào đó** (tự chọn tool nào) | Cần chắc chắn có structured output, **không muốn** nhận câu trả lời văn bản |
| `{"type": "tool", "name": "X"}` | **Phải gọi đúng tool X** | Cần **ép bước đầu tiên / ép thứ tự** |

**Kịch bản guide nêu riêng:**
- `"any"` + nhiều tool trích xuất → model **tự chọn tool tốt nhất**, nhưng bạn vẫn chắc chắn có structured output.
- Forced → ép `extract_metadata` chạy trước mọi tool làm giàu, **rồi xử lý các bước sau ở lượt tiếp theo**.

> **Từ khóa trong đề: "always / must / required first / guarantee the order" → `tool_choice` ép buộc.**
> **Không bao giờ** là "viết rõ hơn trong mô tả tool".

## Số lượng tool và độ tin cậy

> **Quá nhiều tool làm GIẢM độ tin cậy chọn tool** — 18 tool tệ hơn 4–5 tool.
> Lý do: tăng độ phức tạp của quyết định.

Đây là con số cụ thể đề hay dùng — nhớ cặp **18 vs 4–5**.

## Scoped tool access

| Nguyên tắc | Nội dung |
|---|---|
| **Giới hạn theo vai trò** | Mỗi subagent chỉ cầm tool liên quan đến vai trò của nó |
| **Tool ngoài chuyên môn = dùng sai** | Agent tổng hợp có tool web search → sẽ đi search thay vì tổng hợp |
| **Cross-role có giới hạn** | Cho phép **một tập hạn chế** tool liên vai trò cho nhu cầu tần suất cao (ví dụ `verify_fact` cho agent tổng hợp) |
| **Ca phức tạp → qua coordinator** | Cross-role tool chỉ giải quyết ca đơn giản, tần suất cao; ca phức tạp định tuyến qua coordinator |

## Thay tool tổng quát bằng tool bị ràng buộc

`fetch_url` (tải bất kỳ URL nào) → **`load_document`** (chỉ nhận URL tài liệu, **có validate**).

Nguyên tắc: **thu hẹp bề mặt tool** thay vì dặn agent "đừng dùng sai". Ràng buộc trong code mạnh hơn chỉ dẫn trong prompt — cùng trục "đảm bảo vs ảnh hưởng".

---

# 2.4 — Tích hợp MCP server vào Claude Code & workflow agent

## MCP là gì

Giao thức chuẩn để Claude kết nối với hệ thống bên ngoài. **MCP server** phơi bày **tool** (hành động) và **resource** (dữ liệu context).

## Phạm vi cấu hình

| File | Phạm vi | Version control | Dùng cho |
|---|---|---|---|
| **`.mcp.json`** (gốc dự án) | **Cả nhóm** — mọi người đóng góp dự án đều có | ✅ Có | Server dùng chung của team |
| **`~/.claude.json`** (thư mục home) | **Cá nhân** | ❌ Không chia sẻ | Thử nghiệm, server riêng, kiểm thử cá nhân |

## Biến môi trường cho secret

```json
{"mcpServers": {
  "github": {
    "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {"GITHUB_TOKEN": "${GITHUB_TOKEN}"}}}}
```

> **Token KHÔNG BAO GIỜ được commit.** `.mcp.json` nằm trong git, nên secret phải đi qua `${ENV_VAR}` expansion.

## Chọn server: có sẵn vs tự xây ⚠️

> **Tích hợp tiêu chuẩn (Jira, GitHub, Slack, Notion…) → ưu tiên MCP server cộng đồng/nhà cung cấp có sẵn.**
> **Chỉ tự xây cho workflow đặc thù, riêng của nhóm** (pipeline nội bộ, hệ thống không có bản công khai tương đương).

**Bẫy kinh điển:** *"chỉ server tự viết mới đáng tin với credential production"* — **SAI**.
→ **Niềm tin đến từ việc RÀ SOÁT (review) server có sẵn**, không phải từ việc tự tay viết. Tự viết cũng có thể có lỗ hổng.

**Nhận diện trong đề:** "standard read/write operations", tên sản phẩm phổ biến → **có sẵn**. "in-house", "no public equivalent", "team-specific" → **tự xây**. Nếu đề mô tả **hai nhu cầu khác nhau**, đáp án đúng thường **xử lý khác nhau** cho từng cái.

## Xác thực MCP ⚠️

| Cơ chế xác thực | Cấu hình |
|---|---|
| **OAuth** | Dùng cơ chế **OAuth discovery** sẵn có |
| **Không phải OAuth** — Kerberos, SSO nội bộ, token phải **mint mới mỗi lần kết nối** | **`headersHelper`** — sinh header động cho từng kết nối |

> OAuth discovery **không phủ** Kerberos / SSO nội bộ.
> Gặp "token phải mint mới mỗi lần kết nối, không hỗ trợ OAuth" → **`headersHelper`**, **không** dựng dịch vụ trung gian (thừa hạ tầng cho việc đã có sẵn giải pháp).

⚠️ `headersHelper` **không có trong `guide_vi.md`** — nhớ trực tiếp.

## Khám phá tool

> Tool từ **tất cả** MCP server đã cấu hình được **khám phá tại thời điểm kết nối** và **khả dụng đồng thời** cho agent.

Hệ quả nối sang 2.3: cắm nhiều server → agent đột ngột có rất nhiều tool → **giảm độ tin cậy chọn tool**. Hai kiến thức này hay được ghép trong một câu hỏi.

## MCP Resources

**Resource = dữ liệu agent có thể yêu cầu để lấy context, KHÔNG phải hành động.**

Ví dụ: danh mục nội dung (danh sách toàn bộ task dự án, điều hướng phân cấp) · **schema cơ sở dữ liệu** · tài liệu (API reference, hướng dẫn nội bộ) · **tóm tắt issue**.

> **Lợi thế:** agent **không cần các lời gọi tool thăm dò** để biết dữ liệu nào đang tồn tại. Resource đưa ngay một **"tấm bản đồ"**.

Nhận diện trong đề: *"giảm số lời gọi tool khám phá / exploratory"*, *"agent không biết có sẵn dữ liệu gì"* → **MCP resources**.

## Mô tả MCP tool đủ mạnh

Agent ưu tiên built-in (Grep, Read) hơn MCP tool cùng chức năng → **viết mô tả MCP tool chi tiết về khả năng và output**, nhấn thứ built-in không làm được. (Giao với 2.1.)

---

# 2.5 — Tool tích hợp sẵn của Claude Code

## Bảng chọn tool

| Tác vụ | Tool | Ví dụ |
|---|---|---|
| Tìm file theo **TÊN**/mẫu | **Glob** | `**/*.test.tsx`, `src/components/**/*.ts` |
| Tìm kiếm **NỘI DUNG BÊN TRONG** file | **Grep** | tên hàm, thông báo lỗi, import, **tên biến** |
| Đọc toàn bộ một file | **Read** | nạp file để phân tích |
| Ghi file mới | **Write** | tạo file từ đầu |
| Sửa chính xác file có sẵn | **Edit** | thay đoạn cụ thể qua **khớp văn bản duy nhất** |
| Chạy lệnh shell | **Bash** | git, npm, test, build |

> **Glob = TÊN file. Grep = NỘI DUNG trong file.**
> Tìm tham chiếu biến / caller của hàm / thông điệp lỗi → luôn là **Grep**.

## Kiểm kê trước khi sửa

> **Grep liệt kê ĐẦY ĐỦ mọi chỗ xuất hiện TRƯỚC, rồi mới sửa.**
> Vừa tìm vừa sửa từng file → đổi tên sót, không có danh sách để rà soát.

Áp dụng điển hình: đổi tên biến/hằng/hàm trên toàn codebase.

## Điều tra tăng dần (Ch.13.2)

Đừng đọc tất cả file cùng lúc:
```
1. Grep  → tìm entry point (định nghĩa hàm, export)
2. Read  → đọc file tìm được
3. Grep  → tìm nơi dùng (import, lời gọi)
4. Read  → đọc file tiêu thụ
5. Lặp lại cho đến khi có bức tranh đầy đủ
```

**Truy vết qua module bao bọc (wrapper):** trước hết **xác định mọi tên đã export**, sau đó **Grep từng tên** trên toàn codebase. Chỉ grep tên gốc sẽ sót các chỗ gọi qua wrapper.

## Dự phòng khi Edit thất bại (Ch.13.3)

Edit hỏng do **khớp văn bản không duy nhất** →
1. **Read** — nạp toàn bộ nội dung
2. Sửa nội dung bằng chương trình
3. **Write** — ghi đè phiên bản đã cập nhật

---

# Bảng ghi nhớ một trang — Lĩnh vực 2

```
2.1 MÔ TẢ TOOL
    Mô tả = cơ chế chọn tool CHÍNH (không phải tên, không phải thứ tự)
    4 thành phần: làm gì/trả về · định dạng+ví dụ · edge case · khi nào dùng vs tool tương tự
    Chồng lấn  → ĐỔI TÊN + mô tả theo miền (analyze_content → extract_web_results)
    Đa năng    → TÁCH (analyze_document → extract_data_points / summarize_content /
                        verify_claim_against_source)
    ⚠ System prompt LẤN ÁT mô tả tool — "If in doubt use X" = anti-pattern

2.2 LỖI CÓ CẤU TRÚC
    tồn tại + 0 kết quả  = THÀNH CÔNG (danh sách rỗng, KHÔNG isError)
    không tồn tại        = isError + not_found_error + message
    truy vấn hỏng        = isError + transient  (rỗng ≠ thất bại!)
    4 nhóm: transient(RETRY+backoff) / validation(sửa input) /
            business(giải thích cho user, retryable:false) / permission(escalation)
    "Operation failed" = anti-pattern
    Subagent: retry cục bộ 1–2 lần → mới lan truyền, kèm partial_results + attempted_query

2.3 PHÂN BỔ TOOL & TOOL_CHOICE
    mô tả/prompt = ẢNH HƯỞNG (xác suất) | tool_choice = ĐẢM BẢO (tất định)
    auto           → tự quyết (mặc định)
    any            → phải gọi MỘT tool nào đó (cần structured output)
    {tool, name:X} → phải gọi ĐÚNG X (ép thứ tự / bước đầu)
    "always/must/required first" → forced, KHÔNG phải sửa mô tả
    18 tool tệ hơn 4–5 · tool ngoài chuyên môn → dùng sai
    cross-role hạn chế (verify_fact) · ca phức tạp → qua coordinator
    fetch_url → load_document (ràng buộc + validate)

2.4 MCP
    .mcp.json = NHÓM, trong git, secret qua ${ENV_VAR} | ~/.claude.json = CÁ NHÂN
    tiêu chuẩn (Jira/GitHub/Slack) → CỘNG ĐỒNG · tự xây chỉ cho workflow riêng
    niềm tin đến từ RÀ SOÁT, không từ việc tự viết
    OAuth → discovery | Kerberos/SSO nội bộ, mint mỗi kết nối → headersHelper
    tool của MỌI server được khám phá khi kết nối, khả dụng đồng thời
    RESOURCES = danh mục/schema/tóm tắt → giảm lời gọi tool THĂM DÒ
    mô tả MCP tool phải mạnh, nếu không agent chọn built-in (Grep/Read)

2.5 BUILT-IN
    Glob = TÊN file | Grep = NỘI DUNG trong file
    Kiểm kê bằng Grep TRƯỚC, sửa SAU
    Grep entry point → Read → Grep usage → Read (tăng dần, đừng đọc hết)
    Wrapper: liệt kê mọi tên export → Grep từng tên
    Edit lỗi do khớp không duy nhất → Read + Write
```

---

# Tự kiểm tra toàn lĩnh vực (10 câu)

> Che phần **Đáp án** lại, tự trả lời trước rồi mới đối chiếu.

## 2.1 — Mô tả & ranh giới tool

**Câu 1.** Model liên tục gọi `search_web` dù tool khác phù hợp hơn. Nêu **hai** nguồn có thể gây ra, và nguồn nào **không** phải cơ chế thật?

<details><summary><b>Đáp án</b></summary>

Hai nguồn **thật**:
- **Mô tả tool** — `search_web` mô tả quá rộng / chồng lấn với tool khác, hoặc tool đúng có mô tả tối giản nên thua.
- **System prompt** — một chỉ dẫn thường trực nhạy keyword kiểu *"If in doubt, use the search tool"* bẻ lái mọi lượt, **lấn át** cả mô tả tool viết tốt.

**Không** phải cơ chế thật: **quy tắc đặt tên tool** (và thứ tự khai báo tool). Model chọn theo **mô tả**, không theo tên hay vị trí.

> Đây chính là bẫy của Q4. Khi đề hỏi *"tại sao model cứ gọi tool X?"*, luôn xét **hai nguồn**: mô tả tool và system prompt.
</details>

**Câu 2.** `analyze_content` và `analyze_document` bị nhầm lẫn. Hai kỹ thuật sửa khác nhau là gì?

<details><summary><b>Đáp án</b></summary>

Hai kỹ thuật **khác nhau**, tùy nguyên nhân:

| Nguyên nhân | Kỹ thuật |
|---|---|
| Hai tool **chức năng khác nhau** nhưng mô tả gần giống hệt | **Đổi tên + viết lại mô tả theo miền cụ thể**: `analyze_content` → `extract_web_results`, mô tả chỉ nói về kết quả web |
| Một tool **quá đa năng**, ôm nhiều việc | **Tách thành tool chuyên biệt** có hợp đồng I/O riêng: `analyze_document` → `extract_data_points` · `summarize_content` · `verify_claim_against_source` |

Mục tiêu chung: loại bỏ **chồng lấn chức năng**, để mỗi mô tả trả lời được "khi nào dùng cái này thay vì cái kia".
</details>

---

## 2.2 — Phản hồi lỗi có cấu trúc

**Câu 3.** Tool `get_user_permissions` gọi cho user hợp lệ nhưng chưa được cấp quyền nào → `isError` hay không? Nếu user ID sai thì sao?

<details><summary><b>Đáp án</b></summary>

| Tình huống | Phản hồi |
|---|---|
| User **hợp lệ**, chưa có quyền nào | **KHÔNG** `isError` — thành công, trả về **danh sách rỗng**. Truy vấn chạy đúng; câu trả lời là "không có quyền nào" |
| **User ID sai / không tồn tại** | `isError: true` + `errorCategory: not_found_error` + message mô tả |

Lỗi của Q15 là gộp hai dòng này làm một. **"Không có gì khớp" là câu trả lời; "cái được hỏi không tồn tại" là lỗi.**
</details>

**Câu 4.** Một subagent gặp timeout khi tìm kiếm. Nó nên làm gì trước, và khi lan truyền lên coordinator thì kèm những trường nào?

<details><summary><b>Đáp án</b></summary>

**Trước tiên: phục hồi cục bộ.** Timeout là lỗi **transient** → tự retry **1–2 lần** (exponential backoff) ngay trong subagent. Không retry vô hạn (latency + lãng phí), cũng không lan truyền ngay.

Nếu vẫn hỏng → lan truyền lên coordinator kèm:

```json
{"status": "partial_failure",
 "failure_type": "timeout",
 "attempted_query": "...",          ← đã thử gì
 "partial_results": [...],           ← những gì lấy được
 "alternative_approaches": [...],    ← gợi ý hướng khác
 "coverage_impact": "..."}           ← phần nào không bao phủ được
```

Bốn trường cuối cho coordinator đủ dữ kiện để chọn: retry query đã sửa · dùng partial · giao subagent khác · bỏ qua và chú thích lỗ hổng.
</details>

**Câu 5.** Business error (hoàn tiền vượt $500) khác permission error ở **hành động của agent** như thế nào?

<details><summary><b>Đáp án</b></summary>

Cả hai đều `retryable: false`, nhưng **hành động khác hẳn**:

| | Business (hoàn tiền > $500) | Permission (bị từ chối truy cập) |
|---|---|---|
| Hành động | **Giải thích cho người dùng** + đề xuất phương án thay thế | **Escalation** lên con người / quyền cao hơn |
| Ai xử lý tiếp | Agent tự truyền đạt, hội thoại tiếp tục | Ra khỏi vòng tự chủ của agent |
| Cần thêm | Giải thích **thân thiện, hướng người dùng** | Thông tin để người có quyền quyết định |

Business = "hệ thống chạy đúng, chính sách nói không" → giải thích. Permission = "agent không đủ thẩm quyền" → chuyển lên.
</details>

---

## 2.3 — Phân bổ tool & `tool_choice`

**Câu 6.** Cần đảm bảo `validate_input` chạy trước mọi tool khác. Viết vào mô tả tool và system prompt đã đủ chưa? Cơ chế nào mới **đảm bảo**?

<details><summary><b>Đáp án</b></summary>

**Chưa đủ.** Mô tả tool và system prompt chỉ **làm lệch xác suất** — model vẫn có thể bỏ qua.

Cơ chế **đảm bảo**: `tool_choice: {"type": "tool", "name": "validate_input"}` trên lượt đó, rồi xử lý các bước còn lại ở **lượt tiếp theo** (chuyển về `auto` / `any`).

> Từ khóa nhận diện: **always / must / required first / guarantee the order** → luôn là forced `tool_choice`, **không bao giờ** là "viết rõ hơn trong mô tả".
</details>

**Câu 7.** Khi nào dùng `"any"` thay vì `{"type":"tool","name":"X"}`?

<details><summary><b>Đáp án</b></summary>

- **`"any"`** — cần chắc chắn có **structured output** (không nhận câu trả lời văn bản) nhưng **không quan tâm tool nào**. Ví dụ: nhiều tool trích xuất, để model tự chọn cái phù hợp nhất.
- **Forced** — cần **đúng tool X**, thường để **ép bước đầu tiên / ép thứ tự**.

Một câu: `any` = *phải gọi tool gì đó*; forced = *phải gọi đúng tool này*.
</details>

---

## 2.4 — Tích hợp MCP server

**Câu 8.** Nhóm cần tích hợp Slack + một hệ thống phê duyệt nội bộ tự viết. Cái nào dùng server cộng đồng, cái nào tự xây? Lý lẽ "chỉ tự viết mới an toàn cho credential" sai ở đâu?

<details><summary><b>Đáp án</b></summary>

- **Slack** = tích hợp tiêu chuẩn → **MCP server cộng đồng / nhà cung cấp có sẵn** (rà soát trước khi dùng).
- **Hệ thống phê duyệt nội bộ** = workflow đặc thù của nhóm, không có bản công khai tương đương → **tự xây**.

Lý lẽ *"chỉ tự viết mới an toàn cho credential"* sai ở chỗ: **niềm tin đến từ việc RÀ SOÁT mã nguồn, không từ việc ai là tác giả.** Server tự viết cũng có thể có lỗ hổng — thậm chí nhiều hơn, vì không qua nhiều mắt kiểm tra. Rà soát mới là cơ chế tạo niềm tin; tự viết chỉ là **cảm giác** kiểm soát.

> Đề mô tả **hai nhu cầu khác nhau** → đáp án đúng thường **xử lý khác nhau** cho từng cái. Mọi đáp án chọn cùng một cách cho cả hai đều sai ngay từ cấu trúc.
</details>

**Câu 9.** Agent tốn nhiều lời gọi tool chỉ để dò xem dự án có những dữ liệu gì. Cơ chế MCP nào giải quyết?

<details><summary><b>Đáp án</b></summary>

**MCP Resources.** Resource là dữ liệu agent yêu cầu để lấy **context** (không phải hành động): danh mục nội dung, **schema cơ sở dữ liệu**, tóm tắt issue, tài liệu.

Lợi thế: agent có ngay một **"tấm bản đồ"** về dữ liệu nào đang tồn tại, **không cần các lời gọi tool thăm dò**.
</details>

---

## 2.5 — Tool tích hợp sẵn

**Câu 10.** Tìm mọi file test → tool nào? Tìm mọi nơi gọi `processPayment`, kể cả qua wrapper → quy trình nào?

<details><summary><b>Đáp án</b></summary>

- **Mọi file test** → **Glob**, ví dụ `**/*.test.tsx` (tìm theo **tên** file).
- **Mọi nơi gọi `processPayment`, kể cả qua wrapper** → quy trình nhiều bước:
  1. **Grep** `processPayment` → tìm định nghĩa và các lời gọi trực tiếp
  2. **Read** file định nghĩa + các module bao bọc → **xác định mọi tên đã export** (alias, re-export, tên wrapper)
  3. **Grep từng tên export** đó trên toàn codebase
  4. Lặp cho đến khi không phát sinh tên mới

Chỉ grep tên gốc sẽ **sót** các chỗ gọi qua wrapper. Và phải **kiểm kê đầy đủ trước, sửa sau** — vừa tìm vừa sửa sẽ đổi tên sót.
</details>
