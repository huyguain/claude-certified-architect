# Lĩnh vực 3 — Bảng ghi nhớ 1 trang & Tự kiểm tra toàn lĩnh vực

> Cấu hình Claude Code & Workflow — **20% đề thi** (~15 câu / 76).
> Bao phủ **toàn bộ** 3.1 → 3.6. Kế hoạch vá 10 câu sai: [`11-ke-hoach-lanh-vuc-3.md`](./11-ke-hoach-lanh-vuc-3.md)
> Nguồn: `../guide_vi.md` Ch.5 (568–822), blueprint 1699–1786

---

# Bảng ghi nhớ một trang

```
════ 3.1 PHÂN CẤP CLAUDE.md ════════════════════════════════════════════════
  ~/.claude/CLAUDE.md        NGƯỜI DÙNG  · KHÔNG qua VCS · sở thích cá nhân
  .claude/CLAUDE.md | CLAUDE.md (gốc)  DỰ ÁN · QUA VCS · chuẩn code/test/kiến trúc
  <subdir>/CLAUDE.md         THƯ MỤC     · chỉ khi làm việc trong thư mục đó
  ⚠ Lỗi kinh điển: thành viên mới không nhận chỉ dẫn → đặt nhầm ở CẤP NGƯỜI DÙNG
  @path  → @./standards/testing.md · KHÔNG dấu cách sau @ · tương đối tính từ
           file chứa import · lồng tối đa 5 cấp · mỗi package chỉ import chuẩn liên quan
  .claude/rules/  → tách CLAUDE.md nguyên khối thành testing.md, api-conventions.md,
                    deployment.md, react-patterns.md
  /memory  → xem file memory nào đang nạp, chẩn đoán hành vi khác nhau giữa session

════ 3.2 COMMAND & SKILL ════════════════════════════════════════════════════
  .claude/commands/   DỰ ÁN, qua VCS, cả nhóm dùng
  ~/.claude/commands/ CÁ NHÂN, không chia sẻ
  (bản Claude Code hiện tại: commands ĐÃ HỢP NHẤT vào .claude/skills/<name>/SKILL.md;
   đề thi vẫn tham chiếu .claude/commands/ — cả hai đều tạo /name)
  SKILL.md frontmatter:
    context: fork    → chạy trong SUBAGENT tách biệt; output dài dòng KHÔNG nhiễu
                       session chính (phân tích codebase, brainstorm phương án)
    allowed-tools    → giới hạn tool khi skill chạy (bảo mật, chặn thao tác phá hủy)
    argument-hint    → nhắc tham số khi gọi mà không truyền đối số
  Skill cá nhân → ~/.claude/skills/ với TÊN KHÁC để không ảnh hưởng đồng đội
  SKILL vs CLAUDE.md → skill: gọi THEO NHU CẦU cho tác vụ cụ thể
                       CLAUDE.md: chuẩn chung LUÔN được nạp

════ 3.3 QUY TẮC THEO ĐƯỜNG DẪN ════════════════════════════════════════════
  .claude/rules/x.md  + YAML frontmatter:  paths: ["terraform/**/*"]
  Quy tắc CHỈ nạp khi sửa file KHỚP mẫu → tiết kiệm context & token
  paths ["**/*.test.tsx"] → áp dụng theo LOẠI FILE bất kể vị trí
  CHỌN paths-rule  khi quy ước trải XUYÊN NHIỀU thư mục (test, migration)
  CHỌN CLAUDE.md thư mục  khi quy ước GẮN với đúng một thư mục

════ 3.4 PLAN MODE vs THỰC THI TRỰC TIẾP ═══════════════════════════════════
  PLAN MODE: chỉ Read/Grep/Glob · KHÔNG sửa file · sinh kế hoạch chờ duyệt
    → nhiều file (hàng chục / 45+) HOẶC nhiều phương án hợp lý
      HOẶC còn quyết định kiến trúc bỏ ngỏ HOẶC codebase chưa quen
  TRỰC TIẾP: 1 file + stack trace rõ · thêm 1 validation · phạm vi đã hiểu rõ
  ✗ KHÔNG dùng làm tiêu chí: tầm quan trọng nghiệp vụ · loại thao tác ·
    đụng schema · mức rủi ro
  Câu hỏi chốt: "còn gì phải KHÁM PHÁ hay QUYẾT ĐỊNH không?"
  KẾT HỢP: plan để điều tra/thiết kế → user duyệt → trực tiếp để triển khai
  Subagent EXPLORE → tách output khám phá dài dòng, chỉ trả TÓM TẮT,
                     chống cạn context trong tác vụ nhiều giai đoạn

════ 3.5 TINH CHỈNH LẶP ════════════════════════════════════════════════════
  Mô tả bằng văn xuôi bị hiểu khác nhau → 2–3 CẶP VÍ DỤ input/output cụ thể
    (không phải từ đồng nghĩa, không phải quy tắc cứng)
  TEST-DRIVEN: viết bộ test trước (hành vi kỳ vọng + edge case + hiệu năng)
               → chia sẻ THẤT BẠI để lặp
  EDGE CASE sai (null trong migration) → đưa TEST CASE cụ thể input + expected
  MẪU PHỎNG VẤN: để Claude ĐẶT CÂU HỎI trước khi triển khai → lộ ra cân nhắc
                 chưa nghĩ tới (invalidation cache, chế độ lỗi) — miền lạ
  GỘP hay TÁCH phản hồi: vấn đề GẮN KẾT (fix này ảnh hưởng fix kia) → CÙNG
    một thông điệp chi tiết | vấn đề ĐỘC LẬP → sửa TUẦN TỰ
    → nhóm theo PHỤ THUỘC, không theo triệu chứng

════ 3.6 CI/CD ═════════════════════════════════════════════════════════════
  -p / --print      chế độ KHÔNG TƯƠNG TÁC: xử lý → in stdout → thoát
                    cách ĐÚNG DUY NHẤT để chạy trong pipeline; tránh treo chờ input
  --output-format json  +  --json-schema   → kết quả máy đọc được
                    → tự động đăng comment INLINE trên PR
  CLAUDE.md = kênh cấp CONTEXT DỰ ÁN cho Claude do CI gọi
                    (chuẩn test, fixture sẵn có, tiêu chí review)
                    → giảm test vô giá trị, tăng chất lượng sinh test
  Đưa FILE TEST HIỆN CÓ vào context → tránh sinh test trùng kịch bản
  Chạy lại sau commit mới → nhét KẾT QUẢ REVIEW TRƯỚC vào context,
                    dặn chỉ báo cáo vấn đề MỚI / CHƯA sửa → tránh comment trùng
  ⚠ CÔ LẬP CONTEXT SESSION: session đã SINH code thì REVIEW kém hơn
                    (giữ lại context lập luận của chính nó) → dùng INSTANCE ĐỘC LẬP

════ BA TẦNG PHÂN LOẠI MỌI CÂU CI (dùng để loại đáp án nhanh) ══════════════
  thực thi : cái gì được NẠP      → -p, --bare (chặn hook/MCP/CLAUDE.md)
  context  : model NHÌN THẤY gì   → kết quả review trước, file test có sẵn, CLAUDE.md
  đầu ra   : trả về RA SAO        → --output-format json, --json-schema

════ TỪ QUIZ — KHÔNG CÓ TRONG guide_vi.md, phải nhớ thẳng ══════════════════
  CLAUDE.local.md      ghi chú tự do RIÊNG TƯ cho một dự án (URL sandbox, fixture ưa dùng)
                       ≠ settings.local.json (schema cố định, chỉ cho setting)
  Quyền lệnh ghép      mẫu quyền phải khớp TỪNG subcommand;
                       git status && rm -rf build → một vế lệch = VẪN HỎI
  paths + symlink      khớp ĐƯỜNG DẪN LOGIC (đường người dùng gõ), không canonical
```

---

# Tự kiểm tra toàn lĩnh vực (21 câu)

> Che phần **Đáp án** lại, tự trả lời trước rồi mới đối chiếu.

## 3.1 — Phân cấp CLAUDE.md

**Câu 1.** Thành viên mới clone repo nhưng Claude không tuân theo chuẩn code của nhóm, trong khi máy của tech lead thì có. Nguyên nhân nhiều khả năng nhất, và sửa thế nào? Lệnh nào dùng để chẩn đoán?

<details><summary><b>Đáp án</b></summary>

Nguyên nhân: chỉ dẫn nằm ở **`~/.claude/CLAUDE.md`** (cấp người dùng) — chỉ áp dụng cho máy của tech lead, **không đi qua VCS** nên người clone repo không có.

Sửa: chuyển sang **cấp dự án** — `.claude/CLAUDE.md` hoặc `CLAUDE.md` ở thư mục gốc, commit vào git.

Chẩn đoán: lệnh **`/memory`** — xem file memory nào đang thực sự được nạp. Đây là cách phát hiện hành vi khác nhau giữa các máy/session.
</details>

**Câu 2.** Monorepo có 8 package, mỗi package cần chuẩn khác nhau. Nêu **hai** cơ chế mô-đun hóa và điểm khác nhau cơ bản giữa chúng.

<details><summary><b>Đáp án</b></summary>

| Cơ chế | Cách hoạt động | Ai quyết định nạp |
|---|---|---|
| **`@path` import** | CLAUDE.md của mỗi package import đúng file chuẩn liên quan: `@../../standards/api.md` | **Người viết CLAUDE.md chủ động chọn** — dựa trên hiểu biết về miền của package |
| **`.claude/rules/` + `paths`** | File quy tắc tự kích hoạt khi Claude sửa file khớp glob | **Tự động theo file đang sửa** |

Khác biệt cốt lõi: `@path` = **tĩnh, do người cấu hình chọn trước**; `paths` rule = **động, theo ngữ cảnh file đang chỉnh sửa**.
</details>

**Câu 3.** Viết đúng cú pháp import file chuẩn test nằm ở `standards/testing.md` cạnh CLAUDE.md. Độ sâu lồng nhau tối đa là bao nhiêu?

<details><summary><b>Đáp án</b></summary>

```markdown
Test requirements are in @./standards/testing.md
```
- `@` **ngay trước** đường dẫn, **không có dấu cách**
- Hỗ trợ đường dẫn tương đối lẫn tuyệt đối; tương đối được phân giải **tính từ file chứa import**
- **Độ sâu lồng nhau tối đa: 5**
</details>

---

## 3.2 — Command & Skill

**Câu 4.** Một skill phân tích kiến trúc sinh ra hàng nghìn dòng output khiến session chính hết chỗ. Frontmatter nào giải quyết, và nó làm gì?

<details><summary><b>Đáp án</b></summary>

**`context: fork`** — chạy skill trong một **subagent tách biệt**. Session chính chỉ nhận kết quả, không nhận toàn bộ output trung gian.

Hai nhóm ứng viên điển hình: skill **dài dòng** (phân tích codebase, quét toàn repo) và skill **thăm dò** (brainstorm nhiều phương án) — thứ mà context khám phá không nên đọng lại trong hội thoại chính.
</details>

**Câu 5.** Bạn muốn một skill chỉ được ghi file, tuyệt đối không xóa/chạy lệnh phá hủy. Dùng gì? Cơ chế này mạnh hơn hay yếu hơn việc dặn trong nội dung skill?

<details><summary><b>Đáp án</b></summary>

**`allowed-tools`** trong frontmatter SKILL.md, ví dụ giới hạn ở các tool ghi file, không cấp Bash.

**Mạnh hơn** hẳn việc dặn trong nội dung skill. Đây đúng trục "đảm bảo vs ảnh hưởng" của LV2: văn bản chỉ dẫn **làm lệch xác suất**, còn `allowed-tools` là **ràng buộc ở tầng cấu hình** — tool không tồn tại thì model không gọi được. Cùng họ với `tool_choice` (2.3) và `load_document` thay `fetch_url`.
</details>

**Câu 6.** Khi nào đặt một quy ước vào **skill**, khi nào vào **CLAUDE.md**? Tiêu chí phân biệt là gì?

<details><summary><b>Đáp án</b></summary>

| | Skill | CLAUDE.md |
|---|---|---|
| Nạp khi nào | **Theo nhu cầu**, khi người dùng gọi `/name` | **Luôn luôn**, mọi session |
| Nội dung phù hợp | Workflow cho một tác vụ cụ thể (review, sinh test, phân tích) | Chuẩn và quy ước chung |

Tiêu chí phân biệt: *quy ước này cần áp dụng **mọi lúc**, hay chỉ khi làm đúng việc đó?* Nhét workflow tác vụ vào CLAUDE.md = lãng phí context mọi session; nhét chuẩn chung vào skill = quên gọi thì mất chuẩn.
</details>

**Câu 7.** Bạn muốn sửa skill `/review` của nhóm cho hợp gu mình mà không ảnh hưởng đồng đội. Làm thế nào?

<details><summary><b>Đáp án</b></summary>

Tạo biến thể trong **`~/.claude/skills/`** với **tên khác** (ví dụ `/review-huy`). Không sửa file trong `.claude/skills/` của repo — vì nó đi qua VCS và sẽ ảnh hưởng cả nhóm.
</details>

**Câu 8.** Command nằm ở `.claude/commands/` và `~/.claude/commands/` khác nhau ở điểm nào?

<details><summary><b>Đáp án</b></summary>

- **`.claude/commands/`** — của **dự án**, lưu trong VCS, mọi người clone repo đều có → đảm bảo workflow nhất quán toàn nhóm.
- **`~/.claude/commands/`** — của **cá nhân**, không chia sẻ.

> Bản Claude Code hiện tại đã hợp nhất command vào `.claude/skills/<name>/SKILL.md`; cả hai định dạng đều tạo ra `/name`. **Đề thi vẫn tham chiếu `.claude/commands/`** và định dạng đó vẫn được hỗ trợ — đừng loại đáp án chỉ vì nó nhắc `commands/`.
</details>

---

## 3.3 — Quy tắc theo đường dẫn

**Câu 9.** Quy ước viết test áp dụng cho file test **nằm rải rác khắp codebase**. Chọn `.claude/rules/` với `paths` hay CLAUDE.md cấp thư mục? Vì sao?

<details><summary><b>Đáp án</b></summary>

Chọn **`.claude/rules/` với `paths`**:

```yaml
---
paths: ["**/*.test.tsx", "**/*.test.ts"]
---
```

Lý do: glob áp dụng theo **loại file bất kể vị trí**. CLAUDE.md cấp thư mục chỉ phủ **đúng một thư mục** — với file test nằm rải rác, bạn sẽ phải **nhân bản cùng một quy ước vào hàng chục thư mục**, vừa trùng lặp vừa dễ lệch phiên bản.

Quy tắc chọn: quy ước trải **xuyên nhiều thư mục** (test, migration) → `paths` rule · quy ước **gắn với đúng một thư mục** → CLAUDE.md thư mục.
</details>

**Câu 10.** Viết frontmatter cho quy tắc chỉ nạp khi sửa file Terraform. Lợi ích cụ thể của việc nạp có điều kiện là gì?

<details><summary><b>Đáp án</b></summary>

```yaml
---
paths: ["terraform/**/*"]
---

Mọi resource phải có tag owner và environment.
Không hardcode region — dùng biến.
```

Lợi ích: quy tắc **chỉ được nạp khi Claude đang sửa file khớp mẫu** → **tiết kiệm context và token**, không nhồi quy ước Terraform vào session đang làm frontend.
</details>

---

## 3.4 — Plan mode vs trực tiếp

**Câu 11.** Phân loại: (a) sửa NullPointerException trong một file, có stack trace rõ; (b) migration thư viện chạm 45 file; (c) thêm một validation ngày tháng; (d) chọn giữa hai cách tích hợp có yêu cầu hạ tầng khác nhau. Cái nào plan mode, cái nào trực tiếp?

<details><summary><b>Đáp án</b></summary>

| Tình huống | Chế độ | Lý do |
|---|---|---|
| (a) NullPointerException 1 file, stack trace rõ | **Trực tiếp** | Phạm vi rõ, không còn gì phải khám phá |
| (b) Migration thư viện chạm 45 file | **Plan** | Thay đổi quy mô lớn, nhiều file |
| (c) Thêm một validation ngày tháng | **Trực tiếp** | Thay đổi đơn lẻ, đã hiểu rõ |
| (d) Chọn giữa hai cách tích hợp, yêu cầu hạ tầng khác nhau | **Plan** | **Nhiều phương án hợp lý**, quyết định còn bỏ ngỏ |
</details>

**Câu 12.** Nêu **bốn** tiêu chí **KHÔNG** dùng để quyết định plan mode. Câu hỏi chốt đúng là gì?

<details><summary><b>Đáp án</b></summary>

Không dùng để quyết định plan mode:
1. **Tầm quan trọng nghiệp vụ** ("đây là module thanh toán nên phải plan")
2. **Loại thao tác** ("mọi refactor đều phải plan")
3. **Có đụng schema / database hay không**
4. **Mức rủi ro**

Cả bốn đều là trực giác hợp lý nhưng **không phải tiêu chí trong tài liệu**. Một sửa lỗi một dòng trong module thanh toán vẫn là thực thi trực tiếp.

Câu hỏi chốt đúng: **"còn gì phải KHÁM PHÁ hay QUYẾT ĐỊNH không?"** Nếu còn → plan. Nếu đã biết chính xác phải sửa gì ở đâu → trực tiếp.
</details>

**Câu 13.** Tác vụ nhiều giai đoạn, giai đoạn khám phá sinh output rất dài. Cơ chế nào bảo vệ context window, và nó trả về cái gì?

<details><summary><b>Đáp án</b></summary>

**Subagent Explore** — cô lập output khám phá dài dòng ra khỏi context chính, **chỉ trả về một bản tóm tắt**. Mục đích: ngăn cạn kiệt context window trong các tác vụ nhiều giai đoạn.
</details>

---

## 3.5 — Tinh chỉnh lặp

**Câu 14.** Mô tả "làm sạch dữ liệu" bị hiểu khác nhau mỗi lần chạy. Kỹ thuật hiệu quả nhất là gì — và **không phải** là gì?

<details><summary><b>Đáp án</b></summary>

Hiệu quả nhất: **2–3 cặp ví dụ input/output cụ thể**.

```
Input:  "  John SMITH  "   → Output: "John Smith"
Input:  "j.smith@X.COM"    → Output: "j.smith@x.com"
Input:  null                → Output: null  (giữ nguyên, không đổi thành "")
```

**Không phải**: viết lại bằng từ đồng nghĩa, thêm tính từ ("làm sạch **kỹ** dữ liệu"), hay áp một quy tắc cứng nhắc bao trùm. Vấn đề là **thuật ngữ chưa được neo**, nên phải neo bằng **dữ liệu cụ thể**, không phải bằng nhiều chữ hơn.
</details>

**Câu 15.** Bạn tìm ra 5 vấn đề trong code Claude vừa viết. Khi nào gộp cả 5 vào một thông điệp, khi nào sửa tuần tự? Nguyên tắc nhóm là gì?

<details><summary><b>Đáp án</b></summary>

- Các fix **gắn kết / ảnh hưởng lẫn nhau** (sửa cái này làm hỏng cái kia, hoặc cùng chạm một đoạn logic) → **gộp vào MỘT thông điệp chi tiết**, để model thấy toàn cảnh ràng buộc.
- Các vấn đề **độc lập** → **sửa tuần tự**, từng cái một, dễ kiểm chứng.

Nguyên tắc: nhóm theo **phụ thuộc**, **không** theo triệu chứng. Năm lỗi cùng kiểu "null pointer" nhưng ở năm module không liên quan vẫn là năm vấn đề **độc lập**.
</details>

**Câu 16.** Bạn phải triển khai caching trong một miền chưa quen. Mẫu nào giúp lộ ra các cân nhắc chưa nghĩ tới trước khi viết code?

<details><summary><b>Đáp án</b></summary>

**Mẫu phỏng vấn (interview pattern)** — yêu cầu Claude **đặt câu hỏi trước khi triển khai**. Nó làm lộ ra những thứ bạn chưa nghĩ tới: chiến lược vô hiệu hóa cache, TTL, chế độ lỗi khi cache miss, tính nhất quán khi ghi.

Bổ sung cho miền đã quen nhưng cần chất lượng: **lặp theo hướng test** — viết bộ test trước (hành vi kỳ vọng + edge case + yêu cầu hiệu năng), rồi **chia sẻ các thất bại** để dẫn dắt cải thiện từng vòng.
</details>

---

## 3.6 — CI/CD

**Câu 17.** Job CI chạy Claude Code bị treo vô hạn. Cờ nào thiếu và nó làm gì?

<details><summary><b>Đáp án</b></summary>

Thiếu **`-p`** (hay `--print`). Nó bật **chế độ không tương tác**: xử lý prompt → in ra stdout → thoát. Không chờ người dùng nhập liệu. Đây là **cách đúng duy nhất** để chạy Claude Code trong pipeline.
</details>

**Câu 18.** Cần kết quả review đăng thành comment inline trên PR một cách tự động. Hai cờ nào kết hợp?

<details><summary><b>Đáp án</b></summary>

**`--output-format json`** kết hợp **`--json-schema`**:

```bash
claude -p "Review this PR" --output-format json --json-schema '{"type":"object",...}'
```

`--output-format json` cho đầu ra JSON; `--json-schema` **validate** đầu ra theo schema → kết quả **máy phân tích cú pháp được** để tự động đăng comment inline.
</details>

**Câu 19.** Sau mỗi commit mới, bot review đăng lại y nguyên các comment cũ. Sửa ở **tầng nào** trong ba tầng (thực thi / context / đầu ra), và làm gì cụ thể?

<details><summary><b>Đáp án</b></summary>

Tầng **context** — không phải tầng thực thi (cờ CLI), cũng không phải tầng đầu ra (định dạng).

Cụ thể: **đưa kết quả review của lần trước vào context** khi chạy lại sau commit mới, và **chỉ dẫn Claude chỉ báo cáo vấn đề mới hoặc chưa được giải quyết**.

Đây là điểm dùng "ba tầng" để loại đáp án nhanh: mọi đáp án đề xuất thêm cờ CLI đều sai tầng.
</details>

**Câu 20.** Vì sao không nên để chính session đã sinh code đi review code đó?

<details><summary><b>Đáp án</b></summary>

Session đó **giữ lại context lập luận của chính nó** — nó đã "thuyết phục" mình rằng các quyết định là đúng, nên **ít có khả năng phản biện** chính những quyết định đó. Dùng **instance độc lập** để review.

Đây là **cô lập context session**, một dạng khác của nguyên tắc "người kiểm tra phải độc lập với người làm".
</details>

**Câu 21.** Test do CI sinh ra đa phần vô giá trị và trùng với test có sẵn. Nêu **hai** biện pháp khác nhau.

<details><summary><b>Đáp án</b></summary>

Hai biện pháp **khác tầng nhau**:

| Biện pháp | Giải quyết |
|---|---|
| Tài liệu hóa **tiêu chuẩn test, tiêu chí "test có giá trị", fixture sẵn có** trong **CLAUDE.md** | Chất lượng — giảm test vô nghĩa, dùng đúng fixture, đúng phong cách |
| Đưa **file test hiện có** vào context | Trùng lặp — model biết kịch bản nào đã được phủ |

Cả hai đều ở **tầng context**, nhưng một cái cấp **chuẩn**, một cái cấp **hiện trạng**. Đề hay hỏi "nêu hai biện pháp" và đáp án sai thường gộp cả hai thành một.
</details>
