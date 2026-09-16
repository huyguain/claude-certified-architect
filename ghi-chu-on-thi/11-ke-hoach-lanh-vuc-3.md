# Kế hoạch ôn Lĩnh vực 3 — từ 10 câu sai (quiz Domain 3)

> Nguồn: quiz riêng về **Domain 3: Claude Code Configuration & Workflows** (Q1, 2, 4, 7, 10, 12, 15, 16, 17, 19).
> **Không phải** bài thi thử 76 câu. Tham chiếu: `../guide_vi.md`
> Cùng bộ: [`09-ke-hoach-lanh-vuc-1.md`](./09-ke-hoach-lanh-vuc-1.md) · [`10-ke-hoach-lanh-vuc-2.md`](./10-ke-hoach-lanh-vuc-2.md)

---

## Chẩn đoán lại theo subdomain

Đếm số câu sai theo subdomain cho ra thứ tự ưu tiên **khác** với bản tóm tắt gốc:

| Subdomain | Câu sai | Số lần |
|---|---|---|
| **3.6 CI/CD** | Q7, Q12, Q16 | **3** 🔴 ← yếu nhất |
| **3.4 Plan mode** | Q2, Q4 | **2** 🔴 |
| **3.5 Tinh chỉnh lặp** | Q1, **Q17** | **2** 🔴 |
| 3.1 CLAUDE.md | Q19 | 1 🟡 |
| 3.2 Skill/command | Q15 | 1 🟡 |
| 3.3 Quy tắc theo đường dẫn | Q10 | 1 🟡 |

⚠️ Hai điều chỉnh so với bản tóm tắt gốc:
- **3.6 sai 3 lần**, không phải 2 — Q7 (giới hạn 10MB stdin) cũng thuộc 3.6. Đây là subdomain yếu nhất.
- **3.5 sai 2 lần**, không phải 1 — Q17 (nhóm phản hồi theo phụ thuộc) cũng thuộc 3.5, không phải lỗi đơn lẻ.

**Quan sát trực tiếp:**
- Q12, Q16: chọn **thay đổi định dạng đầu ra** cho vấn đề về **thực thi** và **context**
- Q2, Q4: chọn plan mode dựa trên **tầm quan trọng nghiệp vụ** và **loại thao tác**
- Q1: chọn **thay từ đồng nghĩa** để làm rõ chữ "concise"
- Q7: quy lỗi cho **định dạng đầu vào** thay vì **giới hạn dung lượng**
- Q10, Q15, Q19: nhầm chi tiết cấu hình cụ thể

**Suy ra (chưa có bằng chứng trực tiếp):**
- Có xu hướng **với tới cờ/tùy chọn CLI** trước khi hỏi *"vấn đề này nằm ở tầng nào"* — biểu hiện ở Q12, Q16, và cả Q7. Đây là cùng một phản xạ, xuất hiện 3 lần.
- Q1 + Q17 gợi ý chưa nắm **"ví dụ cụ thể > mô tả bằng lời"** như một nguyên tắc chung — đây là cùng nguyên tắc đã sai ở Q20/Q31 của bài 76 câu.

---

## Thứ tự học

```
Chủ đề A (tầng: thực thi / context / đầu ra)  ← trục nền, chi phối 3 câu
        ↓
Chủ đề B (plan mode: tiêu chí phạm vi)        ← 2 câu, quy tắc rõ ràng
        ↓
Chủ đề C (tinh chỉnh lặp)                     ← 2 câu, nối với LV4
        ↓
Chủ đề D (chi tiết cấu hình)                  ← 3 câu rời, học thuộc
```

---

# Chủ đề A — Tầng thực thi vs context vs đầu ra 🔴

**Đọc:** `guide_vi.md` **dòng 761–788** (Ch.5.9), **1768–1786** (LV 3.6)

## Lý thuyết cốt lõi

Mọi lần chạy Claude Code trong CI có **ba tầng tách biệt**. Sửa nhầm tầng thì vấn đề không biến mất:

| Tầng | Kiểm soát cái gì | Cơ chế |
|---|---|---|
| **Thực thi (execution)** | **Cái gì được nạp/khám phá** khi khởi động | `--bare` (chặn auto-discovery), `-p` |
| **Context** | **Model NHÌN THẤY gì** trong prompt | Nhét kết quả lần trước, nhét file test hiện có, CLAUDE.md |
| **Đầu ra (output)** | **Kết quả được TRẢ VỀ ra sao** | `--output-format json`, `--json-schema`, streaming, lọc sự kiện |

> **Quy tắc:** đổi định dạng đầu ra **không bao giờ** đổi được model làm gì hay biết gì.

## Đối chiếu ba câu đã sai

**Q12** — job đêm phải chạy **giống hệt nhau** trên mọi self-hosted runner, không phụ thuộc cấu hình cục bộ của máy đó.

- Vấn đề: mỗi runner có **hook, MCP server, CLAUDE.md** riêng → tự động được nạp → hành vi khác nhau
- **`--bare`**: ngăn **auto-discovery** — hook, MCP server, CLAUDE.md **không được nạp chút nào**
- ❌ Lọc sự kiện đầu ra: cấu hình cục bộ **vẫn được nạp và vẫn ảnh hưởng thực thi**, chỉ là không hiện trong log

> **"Chạy giống hệt nhau bất kể môi trường" / "reproducible" → `--bare`.**

**Q16** — bot review chạy lại sau mỗi commit, **đăng lại comment trùng** về lỗi đã báo.

- Vấn đề: lần chạy mới **không biết** lần trước đã báo gì → đây là vấn đề **context**
- Đúng: **đưa kết quả review trước đó vào context** + chỉ thị *"chỉ báo vấn đề mới/chưa sửa"*
- ❌ Đổi sang streaming/định dạng khác: chỉ đổi **cách giao kết quả**, không đổi việc model **có so sánh với comment cũ hay không**

> **Khử trùng lặp là bài toán context engineering, không phải bài toán định dạng.**

**Q7** — `cat build-error.txt | claude -p ...` thất bại với log lớn.

- Nguyên nhân: **giới hạn 10MB cho stdin qua pipe** → thoát kèm lỗi
- **Không phải** vấn đề định dạng đầu vào: **text thuần là mặc định** của `-p`
- `stream-json` chỉ cần khi **ghép cặp input/output có cấu trúc**, không cần cho text thường

## Các cờ CI cần thuộc

| Cờ | Tác dụng |
|---|---|
| `-p` / `--print` | Chế độ **non-interactive** — bắt buộc trong CI để không bị treo chờ nhập |
| `--bare` | **Chặn auto-discovery**: hook, MCP server, CLAUDE.md không nạp |
| `--output-format json` + `--json-schema` | Structured output để parse (comment inline trên PR) |

⚠️ **`--bare` và giới hạn 10MB KHÔNG có trong `guide_vi.md`** — nhớ trực tiếp.

**Kiến thức 3.6 liên quan cần nhớ cùng:**
- **Cô lập session**: session đã sinh ra code thì **review chính nó kém hơn** một instance độc lập
- Nhét **file test hiện có** vào context khi sinh test mới → tránh trùng lặp, giữ phong cách
- CLAUDE.md cung cấp context dự án cho lần chạy do CI kích hoạt

## Tự kiểm tra

1. Job CI cho kết quả khác nhau giữa các runner. `--output-format json` có sửa được không? Cơ chế nào mới sửa được, và nó chặn **những gì** cụ thể?
2. Phân loại vào 3 tầng: (a) bot đăng lại comment cũ, (b) log 12MB pipe vào thất bại, (c) cần JSON để parse thành comment PR.

---

# Chủ đề B — Plan mode vs thực thi trực tiếp 🔴

**Đọc:** `guide_vi.md` **dòng 716–745** (Ch.5.6), **1740–1753** (LV 3.4)

## Ba tiêu chí ĐÚNG

Dùng **plan mode** khi có **ít nhất một**:

1. **Phạm vi nhiều file / nhiều module** (migration chạm 45+ file, thêm microservice)
2. **Có nhiều phương án hợp lệ** — cần cân nhắc, so sánh
3. **Còn quyết định thiết kế bỏ ngỏ** — phải khám phá mới biết làm thế nào

Dùng **thực thi trực tiếp** khi: **một file / phạm vi đã biết hết, chỉ có một cách triển khai rõ ràng** (bug có stack trace rõ, thêm một validation).

## Bốn tiêu chí SAI — chính là những cái đã dùng ở Q2, Q4

| Tiêu chí sai | Vì sao sai |
|---|---|
| ❌ "Service này quan trọng với nghiệp vụ" (checkout) | **Tầm quan trọng không tạo ra sự mơ hồ.** Thay đổi nhỏ, rõ ràng trong service quan trọng vẫn là thực thi trực tiếp (**Q2**) |
| ❌ "Có đụng tới schema" | Đụng schema mà phạm vi đã biết hết, một cách làm → vẫn trực tiếp (**Q2**) |
| ❌ "Đây là thao tác rename" | **Loại thao tác không quyết định.** Rename 1 helper + vài chỗ gọi trong **một module** = phạm vi đã biết hết → trực tiếp (**Q4**) |
| ❌ "Rủi ro cao / dữ liệu nhạy cảm" | Rủi ro quyết định **hook và quyền hạn**, không quyết định plan mode |

## Câu hỏi chốt

> **"Còn gì phải KHÁM PHÁ hoặc phải QUYẾT ĐỊNH không?"**
> - Còn → plan mode
> - Phạm vi nhìn thấy hết, chỉ một đường đi → trực tiếp

⚠️ Đây **cùng một trục** với fixed-chain vs adaptive ở Lĩnh vực 1: cả hai đều hỏi *"bước tiếp theo đã biết trước chưa?"* — không hỏi về rủi ro hay tầm quan trọng.

**Bổ sung:** **subagent Explore** cô lập đầu ra khám phá dài dòng, chống cạn context trong tác vụ nhiều giai đoạn. Có thể **kết hợp**: plan để khám phá → execute để triển khai.

## Tự kiểm tra

1. Đổi tên một hằng số nội bộ và 5 chỗ gọi nó, tất cả trong một file, trong service thanh toán. Plan mode hay trực tiếp? **Biện minh chỉ bằng phạm vi và sự mơ hồ.**
2. Vì sao "service này xử lý tiền nên phải plan mode" là lập luận sai? Tầm quan trọng nghiệp vụ thực ra quyết định điều gì?

---

# Chủ đề C — Tinh chỉnh lặp (3.5) 🔴

**Đọc:** `guide_vi.md` **dòng 1754–1767** (LV 3.5)

## C1 — Neo thuật ngữ mơ hồ bằng ví dụ (Q1)

Vấn đề: tóm tắt lúc **một câu**, lúc **ba đoạn**. Prompt nói *"concise"*.

| Cách | Kết quả |
|---|---|
| Thay bằng từ đồng nghĩa (*brief, succinct, short*) ❌ | **Vẫn mơ hồ** — đổi từ không đổi được mức chuẩn |
| Ép cứng số câu chính xác ❌ | **Đánh đổi chất lượng lấy nhất quán** — cắt cụt nội dung |
| **2–3 cặp ví dụ đầu vào → đầu ra** ✅ | **Neo được cả độ dài lẫn văn phong** bằng minh chứng |

> **Ví dụ cụ thể là cách hiệu quả nhất để truyền đạt kỳ vọng.** Mục tiêu là **neo phong cách**, không phải **cắt cụt cứng nhắc**.

🔗 Cùng nguyên tắc đã gặp ở bài 76 câu: **Q20** (định dạng feedback không nhất quán → few-shot) và **Q31** (mô tả văn xuôi mơ hồ → few-shot).
Nhưng đối chiếu **Q22**: nếu model tìm **sai thứ cần tìm** (báo thừa + bỏ sót) thì phải **sửa tiêu chí**, không phải thêm ví dụ.

| Triệu chứng | Giải pháp |
|---|---|
| Đầu ra **lúc thế này lúc thế kia** (độ dài, văn phong, định dạng) | **Ví dụ cụ thể** |
| Model **tìm nhầm thứ** (false positive + bỏ sót) | **Tiêu chí tường minh** |
| **Bắt buộc không được sai** | **Hook / cơ chế cứng** |

## C2 — Nhóm phản hồi theo phụ thuộc, không theo triệu chứng (Q17)

Diff có 2 vấn đề: một lỗi logic và một lỗi thẩm mỹ, **nhưng cả hai cùng dính tới một timer dùng chung**.

| Quan hệ giữa các vấn đề | Cách đưa phản hồi |
|---|---|
| **Phụ thuộc lẫn nhau** (chung một cơ chế/biến/timer) | **Mô tả CÙNG một thông điệp** — nêu rõ sự gắn kết |
| **Độc lập** | **Tách riêng**, đưa tuần tự |

**Lỗi ở Q17:** nhóm theo **triệu chứng** (logic ở nhóm này, thẩm mỹ ở nhóm kia) và **bỏ mất** việc chúng dùng chung timer. Sửa từng cái rời rạc → sửa xong cái này hỏng cái kia.

> **Trình tự phản hồi phải theo cấu trúc PHỤ THUỘC, không theo loại vấn đề.**

## Tự kiểm tra

1. Đầu ra dao động giữa "trang trọng" và "thân mật". Thay bằng từ đồng nghĩa chính xác hơn, hay đưa ví dụ? Vì sao ép cứng định dạng lại phản tác dụng?
2. Ba vấn đề trong một diff: hai cái cùng sửa một hàm cache, một cái là lỗi chính tả trong comment. Nhóm thế nào?

---

# Chủ đề D — Chi tiết cấu hình (3.1, 3.2, 3.3)

## D1 — `CLAUDE.local.md` vs `settings.local.json` (Q19)

| File | Chứa gì | Dạng |
|---|---|---|
| **`CLAUDE.local.md`** | **Context cá nhân tự do** cho một dự án: URL database sandbox, fixture ưa dùng, ghi chú riêng | **Memory file** — văn bản tự do |
| **`settings.local.json`** | **Cài đặt có cấu trúc** | **Schema cố định** — không nhét ghi chú tự do được |

> **Ghi chú tự do → memory file. Cài đặt có cấu trúc → settings.**
> `settings.local.json` có schema cố định; nội dung tùy ý không thuộc về đó.

**Phân cấp CLAUDE.md đầy đủ:** người dùng (`~/.claude/CLAUDE.md`) · dự án (`CLAUDE.md` gốc hoặc `.claude/CLAUDE.md`) · thư mục con · **`CLAUDE.local.md`** (riêng tư, theo dự án).

## D2 — Quyền cho lệnh ghép (Q15)

Skill có `allowed-tools` cấp quyền Bash theo **mẫu (pattern)**. Contractor chạy skill vẫn bị hỏi xác nhận.

> **Mẫu quyền khớp với TỪNG SUBCOMMAND RIÊNG LẺ.**
> Lệnh ghép (`a && b`, `a | b`) mà **một bước không khớp mẫu** → **vẫn phải xin phê duyệt**, dù bước kia đã được cấp sẵn.

**Q15 sai ở chỗ:** quy nguyên nhân cho **fork context** hoặc **trạng thái tin cậy của repo**. Nguyên nhân thật: **một subcommand nằm ngoài mẫu Bash đã khai báo**.

**Frontmatter SKILL.md cần thuộc:**

| Trường | Tác dụng |
|---|---|
| `allowed-tools` | Giới hạn tool skill được dùng (bảo mật) |
| `context: fork` | Chạy trong context subagent tách biệt — không làm ô nhiễm session chính |
| `argument-hint` | Nhắc tham số bắt buộc |

## D3 — Quy tắc theo đường dẫn và symlink (Q10)

Repo thật ở `/Users/eng/code/service`, symlink tới một đường dẫn khác. Quy tắc `.claude/rules/` có `paths:` khớp theo đường nào?

> **Khớp theo ĐƯỜNG DẪN LOGIC** — đường nhìn từ thư mục làm việc, **đi qua symlink**.
> **Không** phải đường dẫn canonical của hệ thống file.

⚠️ **Đừng lẫn** với phân giải canonical trong các kiểm tra bảo mật — đó là chuyện khác. Việc **nạp quy tắc** dùng **đường dẫn logic**.

**Quy tắc theo đường dẫn — kiến thức nền:**
- File `.claude/rules/` có frontmatter YAML `paths` với mẫu glob
- **Chỉ tải khi chỉnh sửa file khớp** → tiết kiệm context/token
- Ưu tiên hơn CLAUDE.md cấp thư mục khi quy ước trải **xuyên nhiều thư mục** (ví dụ: test)

## Tự kiểm tra

1. Muốn lưu URL sandbox cá nhân + fixture ưa dùng cho một dự án, không chia sẻ qua git. File nào, vì sao không phải `settings.local.json`?
2. Skill có `allowed-tools` cho `Bash(git:*)`. Lệnh `git status && rm -rf build` có chạy thẳng không? Vì sao?

---

# Lộ trình

| Buổi | Nội dung | Thời lượng |
|---|---|---|
| **1** | Ch.5.9 + LV 3.6 (1768–1786). Vẽ bảng 3 tầng execution/context/output. Phân loại lại Q7, Q12, Q16 | 40' |
| **2** | Ch.5.6 + LV 3.4 (1740–1753). Viết 3 tiêu chí plan mode bằng chữ của mình. Phân loại lại Q2, Q4 | 30' |
| **3** | LV 3.5 (1754–1767). Bảng "triệu chứng → giải pháp". Đối chiếu Q1/Q17 với Q20/Q22/Q31 bài 76 câu | 30' |
| **4** | Ch.5.1–5.5 + LV 3.1–3.3 (1701–1739). Thuộc phân cấp CLAUDE.md và frontmatter SKILL.md | 35' |
| **5** | Rà toàn bộ LV 3: `guide_vi.md` **1701–1786**, tự trả lời từng gạch đầu dòng | 30' |

**Mục LV 3 chưa bị quiz kiểm tra — đọc ở buổi 5:**
- **3.1** Cú pháp `@path` (`@./standards/testing.md`) để mô-đun hóa CLAUDE.md; chia CLAUDE.md lớn thành nhiều file `.claude/rules/`
- **3.1** Chẩn đoán lỗi phân cấp: thành viên mới bỏ lỡ hướng dẫn vì nó nằm ở **cấp người dùng** thay vì **cấp dự án**
- **3.2** Command **dự án** (`.claude/commands/`, chia sẻ qua VCS) vs **người dùng** (`~/.claude/commands/`)
- **3.4** Subagent **Explore** chống cạn context; kết hợp plan → execute
- **3.5** **Lặp theo hướng test** (viết test trước rồi lặp theo thất bại) và **mẫu phỏng vấn**

---

# Bảng ghi nhớ một trang

```
BA TẦNG CI  → thực thi: cái gì được NẠP      → --bare (chặn hook/MCP/CLAUDE.md), -p
              context : model NHÌN THẤY gì   → nhét kết quả lần trước, file test có sẵn
              đầu ra  : trả về RA SAO        → --output-format json, --json-schema
              "reproducible bất kể runner" → --bare
              "đừng đăng lại comment cũ"   → nhét kết quả trước vào context
              pipe stdin: giới hạn 10MB; text thuần là mặc định của -p

PLAN MODE   → nhiều file HOẶC nhiều phương án HOẶC còn quyết định thiết kế bỏ ngỏ
              KHÔNG dùng: tầm quan trọng nghiệp vụ, loại thao tác, đụng schema, rủi ro
              hỏi: "còn gì phải KHÁM PHÁ hay QUYẾT ĐỊNH không?"

TINH CHỈNH  → thuật ngữ mơ hồ → 2-3 CẶP VÍ DỤ (không phải từ đồng nghĩa, không ép cứng)
              phản hồi: gắn kết → CÙNG thông điệp | độc lập → TÁCH riêng
              nhóm theo PHỤ THUỘC, không theo triệu chứng

CẤU HÌNH    → ghi chú tự do riêng tư → CLAUDE.local.md (settings.local.json có schema cố định)
              lệnh ghép: mẫu quyền khớp TỪNG subcommand → một bước lệch = vẫn hỏi
              quy tắc paths: khớp ĐƯỜNG DẪN LOGIC qua symlink, không phải canonical
```
