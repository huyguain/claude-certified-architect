# Kế hoạch ôn Lĩnh vực 1 — từ 4 câu sai (quiz Domain 1)

> Nguồn: quiz riêng về **Domain 1: Agentic Architecture & Orchestration** (Q2, Q5, Q13, Q16).
> **Không phải** bài thi thử 76 câu — đánh số câu khác nhau, đừng lẫn.
> Tham chiếu: `../guide_vi.md` · [`05-chien-thuat-lam-bai.md`](./05-chien-thuat-lam-bai.md)

---

## Chẩn đoán: 4 câu sai thực chất là 3 lỗ hổng

| # | Lỗ hổng | Câu | Mức ưu tiên | Subdomain |
|---|---|---|---|---|
| 1 | **Tiêu chí chọn chiến lược phân rã** | Q2, Q13 | 🔴 **Lặp lại 2 lần** | 1.6 |
| 2 | **Hook ngăn chặn thật hay chỉ ghi log** | Q5 | 🟠 **Nền tảng** | 1.5 |
| 3 | **Resume session bằng tên** | Q16 | 🟡 Đơn lẻ | 1.7 |

**Quan sát được** (bằng chứng trực tiếp từ bài làm):
- Q2 + Q13: chọn **dynamic/adaptive** cho các quy trình **cố định, không rẽ nhánh**
- Q5: tưởng quyết định `deny` của `PreToolUse` chỉ hiện trong transcript, tool **vẫn chạy**
- Q16: chọn **tự ghi lại session ID máy sinh** thay vì đặt tên

**Suy ra** (chưa có bằng chứng trực tiếp nhưng nhiều khả năng):
- Đang dùng **mức rủi ro / độ nhạy cảm dữ liệu / kích thước đầu vào** làm tiêu chí chọn kiến trúc — sai trục hoàn toàn
- Có thể chưa nắm được **hook chạy trước khi tool thực thi**, nên chưa thấy nó khác prompt ở đâu

---

## Thứ tự học (nền tảng trước)

```
Chủ đề 2 (hook: chắc chắn vs xác suất)   ← nền, chi phối nhiều kịch bản
        ↓
Chủ đề 1 (phân rã: cố định vs thích ứng) ← lỗi lặp lại, ưu tiên cao nhất
        ↓
Chủ đề 3 (session: tên vs ID)            ← độc lập, học 10 phút
```

**Vì sao hook học trước dù chỉ sai 1 câu:** nó là trục **"đảm bảo chắc chắn vs chỉ tác động xác suất"**, xuất hiện lại ở tool design, escalation, compliance, CI/CD. Chủ đề 1 sai nhiều hơn nhưng phạm vi hẹp hơn.

---

# Chủ đề 2 — Deterministic enforcement vs prompt influence

**Đọc:** `guide_vi.md` **dòng 432–470** (Ch.3.5 Hooks)

## Lý thuyết cốt lõi

**Hook = code chạy lúc runtime, chặn tại một điểm trong vòng đời agent.** Không phải văn bản, không phải gợi ý — là **code thật sự thực thi**.

| Hook | Chặn ở đâu | Làm được gì |
|---|---|---|
| **PreToolUse** | **TRƯỚC khi tool chạy** | Trả `deny` → **tool KHÔNG bao giờ thực thi**; hoặc chuyển hướng sang luồng khác (escalation) |
| **PostToolUse** | **SAU khi tool chạy, TRƯỚC khi model đọc kết quả** | Biến đổi/chuẩn hóa kết quả (Unix timestamp → ISO 8601, gọn bớt trường thừa) |

**Điểm Q5 hiểu sai:** `deny` từ `PreToolUse` **không phải là một ghi chú trong transcript**. Nó **hủy lời gọi tool**. Tool không chạy, hệ thống bên dưới không bị đụng tới. Việc transcript có hiển thị gì hay không là **chuyện khác hoàn toàn** với việc tool có thực thi hay không.

## Bảng đối lập cốt lõi (thuộc lòng)

| | **Hook** | **Chỉ dẫn trong prompt** |
|---|---|---|
| Đảm bảo | **Deterministic — 100%** | **Xác suất — >90%, không bao giờ 100%** |
| Bản chất | Code chạy lúc runtime | Văn bản tác động lên xác suất sinh |
| Sai sót | Không thể sai nếu điều kiện khớp | **Tỉ lệ thất bại khác 0**, luôn luôn |
| Dùng khi | Tài chính, pháp lý, an toàn, tuân thủ | Sở thích chung, khuyến nghị, định dạng |

> **Quy tắc vàng:** hậu quả về **tài chính / pháp lý / an toàn** → **hook**. Không bao giờ prompt.

## Mở rộng — prerequisite gate (Subdomain 1.4)

Cùng nguyên lý: **chặn tool phía sau cho tới khi bước trước hoàn tất**.
Ví dụ: chặn `process_refund` cho tới khi `get_customer` trả về customer ID đã xác minh.
Đây là **điều kiện tiên quyết bằng code**, không phải dặn dò trong prompt.

## Tự kiểm tra

1. Một `PreToolUse` hook trả `deny` cho lệnh xóa database ngoài giờ bảo trì. Lệnh xóa có chạy không? Nếu thay hook bằng một câu trong system prompt, **điều gì thay đổi về mặt đảm bảo**?
2. Cần chuẩn hóa định dạng ngày tháng trả về từ 3 MCP server khác nhau trước khi agent đọc. Dùng `PreToolUse` hay `PostToolUse`? Vì sao không dùng prompt?

---

# Chủ đề 1 — Fixed chain vs Adaptive decomposition 🔴

**Đọc:** `guide_vi.md` **dòng 947–970** (Ch.6.3) và **1099–1150** (Ch.8.1–8.3)

## Tiêu chí DUY NHẤT

> **Bước tiếp theo có phụ thuộc vào kết quả phát hiện ở bước trước không?**
>
> - **Không → Fixed chain (prompt chaining)**
> - **Có → Adaptive decomposition**

Chỉ một câu hỏi này. Không có câu hỏi thứ hai.

## Bốn tiêu chí SAI — chính là những cái đã dùng ở Q2 và Q13

| Tiêu chí sai | Vì sao sai |
|---|---|
| ❌ "Dữ liệu đầu vào thay đổi mỗi lần" | File Terraform mỗi quý khác nhau, nhưng **11 quy tắc kiểm tra vẫn y nguyên** → vẫn là fixed chain (**Q2**) |
| ❌ "Việc này rủi ro cao" | Onboarding có dính dữ liệu tài chính, nhưng **3 bước vẫn luôn theo đúng thứ tự đó** → fixed chain (**Q13**) |
| ❌ "Dữ liệu nhạy cảm" | Độ nhạy cảm quyết định **hook và quyền hạn**, không quyết định cách phân rã |
| ❌ "File to / nhiều file" | Kích thước quyết định **chia lượt (multi-pass)**, không quyết định fixed hay adaptive |

## Đối chiếu bằng chính hai câu đã sai

| | Mô tả trong đề | Có rẽ nhánh theo phát hiện? | Đúng |
|---|---|---|---|
| **Q2** | 11 quy tắc compliance, **giống hệt nhau mỗi quý**, **độc lập với nhau** | **Không** — biết trước cả 11 bước | **Fixed chain** |
| **Q13a** | Onboarding: tạo tài khoản → gán quyền → gửi email chào mừng, **luôn cùng thứ tự** | **Không** | **Fixed chain** |
| **Q13b** | Điều tra gian lận | **Có** — bước sau tùy thuộc phát hiện ở bước trước | **Adaptive** |

**Chữ khóa trong đề cần khoanh ngay:**

| Nghiêng về **Fixed chain** | Nghiêng về **Adaptive** |
|---|---|
| *always, same order, identical, each quarter, every PR, checklist, template, standard process, independent* | *open-ended, investigate, depends on what is found, legacy, comprehensive audit, modernize, scope unknown* |

**Q2 có cả "identical" lẫn "independent"** — hai tín hiệu fixed chain nằm ngay trong đề.
**Q13 có "always ... in the same order"** — tín hiệu rõ nhất có thể.

## Khái niệm lân cận đừng lẫn

| | Là gì | Chia theo |
|---|---|---|
| **Prompt chaining** | Nhiều lượt gọi tuần tự, cùng một agent | **Bước công việc** (biết trước) |
| **Adaptive decomposition** | Tác vụ con sinh dần theo phát hiện | **Manh mối tìm được** |
| **Multi-pass review** | Mỗi file một lượt + 1 lượt tích hợp | **Chống pha loãng chú ý** (Ch.8.3) |
| **Multi-agent** | Nhiều agent + coordinator | **Vai trò** |

⚠️ **Multi-pass ≠ adaptive.** Chia 14 file thành 14 lượt vẫn là **fixed chain** — biết trước hết các lượt.

## Tự kiểm tra

1. Một agent rà soát bảo mật: luôn chạy đúng 5 kiểm tra giống nhau trên mọi repo, nhưng mỗi repo có ngôn ngữ và kích thước rất khác nhau. Fixed hay adaptive? **Biện minh chỉ bằng tiêu chí rẽ nhánh.**
2. Vì sao "quy trình này xử lý dữ liệu y tế nên phải dùng adaptive decomposition" là lập luận sai? Độ nhạy cảm dữ liệu thực ra quyết định điều gì?

---

# Chủ đề 3 — Session state, resumption, forking

**Đọc:** `guide_vi.md` **dòng 789–812** (Ch.5.10)

## Ba cơ chế

| Cơ chế | Dùng để | Ghi chú |
|---|---|---|
| **`--resume <session-name>`** | Tiếp tục session đã **đặt tên** | Tên do người đặt, dễ nhớ: `investigation-auth-bug` |
| **`fork_session`** | Tách **nhánh độc lập** từ một baseline chung | Cả hai nhánh kế thừa context tới điểm rẽ, sau đó phân kỳ |
| **Session mới + tóm tắt có cấu trúc** | Khi kết quả tool cũ **đã lỗi thời** | Đáng tin hơn resume với dữ liệu cũ |

**Điểm Q16 hiểu sai:** handle dễ nhớ đến từ việc **đặt tên (lúc khởi tạo) hoặc đổi tên (rename)** — **không** phải từ việc tự chép session ID máy sinh ra bảng tính. Ghi tay ID là thao tác thủ công, dễ tra nhầm, và là **anti-pattern** trong đề.

## Resume hay bắt đầu mới?

| Tình huống | Chọn |
|---|---|
| Context cũ **vẫn còn đúng** | **Resume** |
| File đã thay đổi kể từ session trước | Resume **nhưng phải báo agent biết file nào đổi** để phân tích lại có trọng điểm |
| Kết quả tool cũ **đã lỗi thời hẳn** | **Session mới + tóm tắt có cấu trúc** |
| Muốn so sánh 2 phương án từ cùng một phân tích | **`fork_session`** |

## Tự kiểm tra

1. Nhóm chạy nhiều session điều tra ad-hoc mỗi tuần và muốn tìm lại dễ dàng. Cơ chế nào? Vì sao bảng tính ghi ID là sai hướng?
2. Resume một session sau khi đã sửa code. Rủi ro cụ thể là gì, và khi nào thì **bắt đầu mới** tốt hơn **resume**?

---

# Lộ trình gợi ý

| Buổi | Nội dung | Thời lượng |
|---|---|---|
| **1** | Ch.3.5 Hooks (dòng 432–470). Thuộc bảng deterministic/xác suất. Trả lời 2 câu tự kiểm tra chủ đề 2 | 30' |
| **2** | Ch.6.3 + Ch.8.1–8.3. Viết ra **tiêu chí rẽ nhánh** bằng chữ của mình. Phân loại lại Q2, Q13 | 45' |
| **3** | Ch.5.10 Session. Trả lời 2 câu tự kiểm tra chủ đề 3 | 15' |
| **4** | **Rà lại toàn bộ Lĩnh vực 1**: đọc `guide_vi.md` dòng **1538–1626** (blueprint Việt hóa), tự trả lời từng gạch đầu dòng mà không nhìn tài liệu | 40' |

**Phần Lĩnh vực 1 chưa được quiz kiểm tra — đọc thêm ở buổi 4:**
- **1.1** Anti-pattern vòng lặp agent: đọc chữ trong phản hồi để đoán "xong chưa", đặt **giới hạn số vòng lặp** làm cơ chế dừng chính, kiểm tra "có text không" → **đều sai**. Chỉ `stop_reason` mới đúng (dòng 351)
- **1.3** `allowedTools` của coordinator **bắt buộc có `"Task"`**; subagent **không tự kế thừa** context; spawn song song = **nhiều lời gọi `Task` trong CÙNG một phản hồi**
- **1.2** Hai rủi ro phân rã: **quá hẹp** (thiếu phủ) và **chồng lấn** (trùng lặp, phí token)

---

# Bảng ghi nhớ một trang

```
PHÂN RÃ    → chỉ hỏi: bước sau có phụ thuộc phát hiện bước trước không?
             Không → fixed chain | Có → adaptive
             KHÔNG dùng: rủi ro, độ nhạy cảm, kích thước, biến động đầu vào

HOOK       → PreToolUse: chặn TRƯỚC khi tool chạy, deny = KHÔNG thực thi
             PostToolUse: biến đổi kết quả SAU khi chạy, TRƯỚC khi model đọc
             Hook = 100% | Prompt = >90%, không bao giờ 100%
             Tài chính/pháp lý/an toàn → hook

SESSION    → --resume <tên>: đặt tên hoặc rename, KHÔNG chép ID thủ công
             fork_session: 2 nhánh từ 1 baseline
             Kết quả cũ lỗi thời → session mới + tóm tắt, đừng resume
```
