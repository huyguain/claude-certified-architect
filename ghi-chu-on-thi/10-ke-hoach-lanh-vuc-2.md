# Kế hoạch ôn Lĩnh vực 2 — từ 6 câu sai (quiz Domain 2)

> Nguồn: quiz riêng về **Domain 2: Tool Design & MCP Integration** (Q2, Q4, Q12, Q13, Q15, Q20).
> **Không phải** bài thi thử 76 câu — đánh số khác, đừng lẫn.
> Tham chiếu: `../guide_vi.md` · [`09-ke-hoach-lanh-vuc-1.md`](./09-ke-hoach-lanh-vuc-1.md)

---

## Chẩn đoán: 6 câu sai = 4 lỗ hổng, nhưng thực chất là 2 trục

| # | Lỗ hổng | Câu | Ưu tiên | Subdomain |
|---|---|---|---|---|
| 1 | **Ảnh hưởng vs đảm bảo** trong chọn tool | Q4, Q12 | 🔴 Lặp 2 lần | 2.1, 2.3 |
| 2 | **Server có sẵn vs tự xây** (MCP) | Q13, Q20 | 🔴 Lặp 2 lần | 2.4 |
| 3 | **Thiết kế lỗi có cấu trúc** | Q15 | 🟠 Nền tảng | 2.2 |
| 4 | **Grep vs Glob** | Q2 | 🟡 Đơn lẻ | 2.5 |

**Hai trục xuyên suốt (nhận diện từ pattern):**

- **Trục A — "tác động ≠ bảo đảm"**: chi phối lỗ hổng 1 (Q4, Q12). Cùng trục với hook-vs-prompt ở Lĩnh vực 1.
- **Trục B — "mặc định là dùng cái có sẵn"**: chi phối lỗ hổng 2 (Q13, Q20). Tự xây là ngoại lệ, phải biện minh.

**Quan sát trực tiếp:**
- Q4: quy việc chọn tool cho **quy tắc đặt tên**, không nhận ra **chỉ dẫn trong system prompt** đã lấn át
- Q12: tin rằng **mô tả tool + `auto`** đủ để đảm bảo thứ tự
- Q13: chọn **dịch vụ trung gian** thay vì `headersHelper`
- Q20: chọn **tự xây server** cho Jira (tích hợp tiêu chuẩn); cho rằng **chỉ server tự viết mới đáng tin** với credential production
- Q15: coi customer ID **không tồn tại** là kết quả rỗng hợp lệ
- Q2: chọn **Glob** cho việc tìm tham chiếu biến **bên trong** file

**Suy ra (chưa có bằng chứng trực tiếp):**
- Có xu hướng mặc định **"tự làm thì kiểm soát được hơn"** → dẫn tới cả Q20 (tự xây server) lẫn Q13 (tự dựng trung gian). Đây là cùng một thiên lệch, xuất hiện hai lần.
- Có thể chưa phân biệt được **"không có kết quả nào khớp"** với **"thứ được hỏi không tồn tại"** — Q15 là biểu hiện, và trục này cũng chi phối cách thiết kế lỗi nói chung.

---

## Thứ tự học

```
Chủ đề 1 (tác động vs đảm bảo)   ← trục nền, lặp 2 lần, nối với hook ở LV1
        ↓
Chủ đề 3 (lỗi có cấu trúc)       ← nền tảng, chi phối nhiều kịch bản
        ↓
Chủ đề 2 (MCP: có sẵn vs tự xây) ← lặp 2 lần nhưng là quy tắc rời rạc, dễ thuộc
        ↓
Chủ đề 4 (Grep vs Glob)          ← 5 phút
```

---

# Chủ đề 1 — Ảnh hưởng vs Đảm bảo trong lựa chọn tool 🔴

**Đọc:** `guide_vi.md` **dòng 259–272** (Ch.2.3 `tool_choice`), **1657–1670** (LV 2.3)

## Lý thuyết cốt lõi

Có **hai tầng** tác động lên việc model gọi tool nào:

| Tầng | Cơ chế | Mức đảm bảo |
|---|---|---|
| **Ảnh hưởng (xác suất)** | Mô tả tool, tên tool, chỉ dẫn trong system prompt | **Chỉ làm lệch xác suất** — không bao giờ 100% |
| **Đảm bảo (tất định)** | `tool_choice` | **Bắt buộc** |

### Bảng `tool_choice` — thuộc lòng

| Giá trị | Model bắt buộc làm gì | Dùng khi |
|---|---|---|
| `{"type": "auto"}` | Tự quyết: gọi tool **hoặc** trả lời bằng văn bản | Mặc định |
| `{"type": "any"}` | **Phải gọi một tool nào đó** (tự chọn tool nào) | Cần chắc chắn có structured output, không muốn nhận văn bản |
| `{"type": "tool", "name": "X"}` | **Phải gọi đúng tool X** | Cần **ép bước đầu tiên / ép thứ tự** |

## Đối chiếu hai câu đã sai

**Q12** — agent có `extract_metadata` + vài tool làm giàu (`add_tags`, `link_related`…). Yêu cầu: `extract_metadata` **luôn phải chạy trước**.

| Cách | Kết quả |
|---|---|
| Mô tả tool nhấn mạnh "hãy gọi tool này trước" + `auto` ❌ | **Chỉ là gợi ý** — model vẫn có thể bỏ qua |
| `tool_choice: {"type":"tool","name":"extract_metadata"}` ✅ | **Bảo đảm** lượt đó gọi đúng tool đó |

> **Đề có chữ "always / must / required first / guarantee the order" → `tool_choice` ép buộc. Không bao giờ là mô tả tool.**

**Q4** — thêm câu *"If in doubt, use the search tool"* vào system prompt → model gọi `search_web` cả khi tool khác phù hợp hơn.

Bài học: **chỉ dẫn trong system prompt có thể LẤN ÁT đánh giá của model** về tool nào phù hợp, kể cả khi mô tả tool nói ngược lại. Nguyên nhân **không phải** quy tắc đặt tên — mà là **một câu dặn dò thường trực đang bẻ lái mọi lượt**.

→ Hệ quả thực tiễn: chỉ dẫn kiểu *"khi nghi ngờ hãy dùng X"* là **anti-pattern** — nó biến X thành lựa chọn mặc định.

## Kiến thức liên quan cần nhớ cùng

- **Quá nhiều tool làm giảm độ tin cậy chọn tool** (18 tool tệ hơn 4–5) — dòng 1660
- Agent có tool **ngoài chuyên môn** thì hay dùng sai
- **Built-in tool lấn át MCP tool**: agent thường ưu tiên Read/Grep hơn MCP tool cùng chức năng. Khắc phục: **làm mạnh mô tả MCP tool** — nhấn dữ liệu độc nhất mà built-in không có (dòng 257)

## Tự kiểm tra

1. Quy trình bắt buộc `validate_input` chạy trước mọi tool khác. Viết vào mô tả tool và vào system prompt đã đủ chưa? Cơ chế nào mới **đảm bảo**?
2. Phân biệt `tool_choice: "any"` với `{"type":"tool","name":"X"}` — cái nào dùng khi cần structured output nhưng **không quan tâm** tool nào?

---

# Chủ đề 3 — Thiết kế phản hồi lỗi có cấu trúc

**Đọc:** `guide_vi.md` **dòng 525–554** (Ch.4.4 `isError`), **1643–1656** (LV 2.2)

## Phân biệt cốt lõi (chính là Q15)

| Tình huống | Đây là gì | Phản hồi |
|---|---|---|
| Khách hàng **tồn tại**, có **0 đơn hàng** | **THÀNH CÔNG** — tập rỗng hợp lệ | Danh sách rỗng, **không** `isError` |
| Customer ID **không tồn tại** | **LỖI** — tài nguyên không tìm thấy | `isError: true` + `errorCategory: not_found_error` + thông điệp mô tả |

> **"Không có gì khớp" ≠ "cái được hỏi không tồn tại".**
> Vế đầu là câu trả lời. Vế sau là lỗi.

⚠️ `not_found_error` và `headersHelper` **KHÔNG có trong `guide_vi.md`** — phải nhớ trực tiếp.

## Bốn nhóm lỗi (Ch.10.1 + LV 2.2)

| Nhóm | Ví dụ | `isRetryable` |
|---|---|---|
| **Transient** | Timeout, service tạm ngưng | ✅ true |
| **Validation** | Đầu vào sai định dạng | ❌ false |
| **Business** | Vi phạm chính sách (hoàn tiền > $500) | ❌ false — kèm giải thích cho người dùng |
| **Permission / not_found** | Không có quyền, tài nguyên không tồn tại | ❌ false |

**Lỗi có cấu trúc (tốt):**
```json
{"isError": true, "content": {
  "errorCategory": "transient", "isRetryable": true,
  "message": "Service tạm thời không khả dụng...",
  "attempted_query": "order_id=12345", "partial_results": null}}
```

**Anti-pattern:** `{"isError": true, "content": "Operation failed"}` — agent không có thông tin nào để quyết định retry / đổi truy vấn / escalation.

## Tự kiểm tra

1. Tool `get_user_permissions` gọi cho user hợp lệ nhưng chưa được cấp quyền nào. `isError` hay không? Nếu user ID sai thì sao?
2. Vì sao `"Operation failed"` là anti-pattern dù đã đặt `isError: true` đúng?

---

# Chủ đề 2 — MCP: server có sẵn vs tự xây 🔴

**Đọc:** `guide_vi.md` **dòng 486–524** (Ch.4.3), **1671–1683** (LV 2.4)

## Quy tắc mặc định

> **Tích hợp tiêu chuẩn (Jira, GitHub, Slack, Notion…) → dùng MCP server cộng đồng/nhà cung cấp có sẵn.**
> **Chỉ tự xây cho workflow đặc thù riêng của nhóm** (ví dụ: pipeline nội bộ không ai khác có).

**Q20 sai ở hai chỗ:**
1. Chọn tự xây cho **Jira** — đây là tích hợp tiêu chuẩn, đã có server sẵn
2. Cho rằng **chỉ server tự viết mới đáng tin** với credential production

→ **Niềm tin đến từ việc RÀ SOÁT (review) server có sẵn**, không phải từ việc tự tay viết. Tự viết cũng có thể có lỗ hổng; rà soát mới là cơ chế tạo niềm tin.

**Đúng cho Q20:** Jira → server cộng đồng (sau khi rà soát) · pipeline nội bộ → tự xây.

## Xác thực MCP (Q13)

| Cơ chế xác thực | Cấu hình |
|---|---|
| **OAuth** | Dùng cơ chế discovery OAuth sẵn có |
| **Không phải OAuth** — Kerberos, SSO nội bộ, token phải mint mới mỗi lần kết nối | **`headersHelper`** — sinh header động cho từng kết nối |

**Q13 sai:** chọn dựng **dịch vụ trung gian** để đổi token. Đó là **thêm hạ tầng** cho một việc `headersHelper` làm sẵn.

> OAuth discovery **không phủ** Kerberos / SSO nội bộ. Gặp "token phải mint mới mỗi lần kết nối" → **`headersHelper`**.

## Phạm vi cấu hình (hay hỏi kèm)

| File | Phạm vi | Dùng cho |
|---|---|---|
| `.mcp.json` (gốc dự án) | **Cả nhóm**, vào version control | Server dùng chung; secret qua `${GITHUB_TOKEN}` |
| `~/.claude.json` | **Cá nhân**, không chia sẻ | Thử nghiệm, server riêng |

**Nhớ:** token **không bao giờ commit** — luôn thay thế bằng biến môi trường.

## Tự kiểm tra

1. Nhóm cần tích hợp Slack + một hệ thống phê duyệt nội bộ tự viết. Cái nào dùng server cộng đồng, cái nào tự xây? Lý lẽ "chỉ tự viết mới an toàn cho credential" sai ở đâu?
2. MCP server nội bộ cần token Kerberos mint mới mỗi lần kết nối, không hỗ trợ OAuth. Cấu hình bằng gì, và vì sao dịch vụ trung gian là lựa chọn thừa?

---

# Chủ đề 4 — Grep vs Glob

**Đọc:** `guide_vi.md` **dòng 1500–1522** (Ch.13.1–13.2)

| Tool | Tìm cái gì | Ví dụ |
|---|---|---|
| **Glob** | **TÊN file** theo mẫu | `**/*.test.tsx` |
| **Grep** | **NỘI DUNG bên trong file** | tên hàm, thông điệp lỗi, import, **tên biến** |
| **Read** | Đọc nguyên file | |
| **Edit** | Sửa chính xác qua khớp văn bản **duy nhất** | |
| **Write** | Tạo file mới / ghi đè | |
| **Bash** | Lệnh shell | git, npm, test |

**Q2:** đổi tên biến `API_TIMEOUT_MS` → `REQUEST_TIMEOUT_MS` ở mọi nơi. Tham chiếu biến nằm **bên trong** file → **Grep**, không phải Glob.

**Nguyên tắc thứ hai của Q2 — kiểm kê trước khi sửa:**
> **Grep liệt kê ĐẦY ĐỦ mọi chỗ xuất hiện TRƯỚC, rồi mới sửa.**
> Sửa từng file khi chưa có danh sách đầy đủ → đổi tên sót, không rà soát được.

**Chiến lược điều tra tăng dần (13.2):** Grep điểm vào → Read file tìm được → Grep nơi dùng → Read file tiêu thụ → lặp.

**Dự phòng (13.3):** Edit thất bại do khớp không duy nhất → **Read + sửa + Write**.

## Tự kiểm tra

1. Tìm mọi file test trong dự án → tool nào? Tìm mọi nơi gọi hàm `processPayment` → tool nào?
2. Vì sao phải Grep kiểm kê trước khi Edit thay vì vừa tìm vừa sửa?

---

# Lộ trình

| Buổi | Nội dung | Thời lượng |
|---|---|---|
| **1** | Ch.2.3 `tool_choice` (dòng 259–272) + LV 2.3 (1657). Thuộc bảng 3 giá trị. Đọc lại dòng 257 (built-in lấn át MCP tool) | 35' |
| **2** | Ch.4.4 `isError` (525–554) + LV 2.2 (1643). Thuộc bảng 4 nhóm lỗi + phân biệt rỗng-hợp-lệ vs not_found | 30' |
| **3** | Ch.4.3 MCP config (486–524) + LV 2.4 (1671). Thuộc quy tắc cộng đồng-vs-tự xây và bảng OAuth/headersHelper | 30' |
| **4** | Ch.13.1–13.3 (1500–1535). Thuộc bảng 6 tool | 15' |
| **5** | Đọc toàn bộ LV 2 blueprint Việt hóa: **dòng 1630–1700**, tự trả lời từng gạch đầu dòng | 30' |

**Mục LV 2 chưa bị quiz kiểm tra — đọc ở buổi 5:**
- **2.1** Tool đa năng bị lạm dụng → thay bằng tool hẹp có validate (`fetch_url` → `load_document`)
- **2.3** Giới hạn bộ tool mỗi subagent theo vai trò; 18 tool kém tin cậy hơn 4–5
- **2.4** MCP **resources** (danh mục nội dung, schema DB) để giảm lời gọi tool khám phá
- **2.4** Tool của **mọi** MCP server đã kết nối đều được khám phá và khả dụng đồng thời

---

# Bảng ghi nhớ một trang

```
CHỌN TOOL   → mô tả/prompt = ẢNH HƯỞNG (xác suất)
              tool_choice   = ĐẢM BẢO (tất định)
              "always/must/required first" → {"type":"tool","name":"X"}
              "phải gọi tool nào đó"       → {"type":"any"}
              ⚠ "If in doubt use X" trong prompt = anti-pattern, biến X thành mặc định

LỖI MCP     → tồn tại + 0 kết quả = THÀNH CÔNG (danh sách rỗng)
              không tồn tại        = isError + not_found_error + message
              4 nhóm: transient(retry) / validation / business / permission
              "Operation failed" = anti-pattern

MCP SERVER  → tiêu chuẩn (Jira/GitHub/Slack) → CỘNG ĐỒNG, tự xây chỉ cho workflow riêng
              niềm tin đến từ RÀ SOÁT, không từ việc tự viết
              OAuth → discovery | Kerberos/SSO nội bộ → headersHelper
              .mcp.json = nhóm, ${ENV_VAR} | ~/.claude.json = cá nhân

TOOL SẴN    → Glob = TÊN file | Grep = NỘI DUNG trong file
              Kiểm kê bằng Grep TRƯỚC, sửa SAU
              Edit lỗi do khớp không duy nhất → Read + Write
```
