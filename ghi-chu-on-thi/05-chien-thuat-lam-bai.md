# Chiến thuật đọc câu hỏi & chọn đáp án trong phòng thi

> Đúc kết từ định dạng đề (`../guide_en.md` §Exam Format) và 24 câu sai của bài thi thử (xem [`03-chan-doan-thi-thu.md`](./03-chan-doan-thi-thu.md)). Mục tiêu: biến các "quy tắc tự vấn" đã học thành quy trình cố định, làm y hệt nhau cho mọi câu.

## Định dạng cần nhớ

| Yếu tố | Giá trị | Hệ quả khi làm bài |
|---|---|---|
| Loại câu | Trắc nghiệm, **1 đúng / 4 lựa chọn** | Luôn có 2 đáp án loại được ngay, 2 đáp án "gần đúng" để phân vân |
| Chấm điểm | Thang 100–1000, đậu **720** (~72%) | Cần đúng ~3/4. Sai rải đều còn hơn sai dồn 1 kịch bản |
| Phạt đoán mò | **Không** | Không bao giờ bỏ trống. Phân vân → chọn 1 cái, đánh dấu, đi tiếp |
| Kịch bản | 4/8, chọn ngẫu nhiên, ~15 câu/kịch bản | Không đoán được sẽ gặp kịch bản nào — ôn đều 8 |
| Lĩnh vực nặng điểm | LV1 (27%) + LV3 (20%) + LV4 (20%) | Câu về agent orchestration, Claude Code config, prompt/structured output đáng đầu tư thời gian nhất |

## Quy trình 6 bước cho MỖI câu

**Bước 1 — Đọc dòng câu hỏi in đậm TRƯỚC, chưa đọc tình huống.**
Xác định đang bị hỏi *loại* gì — trả lời sai loại là lỗi mất điểm oan phổ biến nhất:

| Câu hỏi hỏi gì | Phải trả lời |
|---|---|
| "What is the **root cause**?" | Nguyên nhân, KHÔNG phải cách sửa |
| "Which **fix / improvement** is most effective?" | Giải pháp nhắm đúng nguyên nhân gốc |
| "What is the **next step**?" | Hành động kế tiếp trong quy trình, không nhảy cóc |
| "How should you **evaluate** this proposal?" | Đánh giá có điều kiện ("dùng cho X, không dùng cho Y"), hiếm khi "đồng ý/từ chối" tuyệt đối |
| "...**most** effective / **best**" | Có thể nhiều đáp án "đúng" — chọn cái đúng nguyên nhân gốc với **ít cơ chế thừa nhất** |

**Bước 2 — Đọc tình huống, gạch chân 4 thứ:**
1. **Hậu quả nếu làm sai** — tiền / pháp lý / an toàn / không hoàn tác được? → nghiêng về cơ chế cứng. Chỉ là giọng văn / định dạng? → prompt là đủ.
2. **Triệu chứng cố định hay ngẫu nhiên** — lỗi lặp theo MỘT mẫu → few-shot / rule. Lỗi đổi ngẫu nhiên theo từng case → cần cơ chế động (self-critique, validation-retry).
3. **Cái gì đã hoạt động tốt rồi** — nếu hệ thống đã ổn với ca đơn giản, đừng đề xuất đập đi xây lại.
4. **Con số / ràng buộc** — deadline, %, token, số vòng lặp. Thường là chìa khóa loại đáp án (VD: Batch API 24h vs check chặn merge).

**Bước 3 — Gắn vào 1 lĩnh vực + 1 chương lý thuyết.** Nếu không gắn được, nhiều khả năng là 1 trong vài fact "ngoài 13 chương" (semantic search cho hội thoại dài; prefill để bỏ lời chào lặp) — nhớ trực tiếp, đừng suy luận.

**Bước 4 — Loại 2 đáp án sai rõ.** Các dấu hiệu đáp án sai:
- Chuyển gánh nặng sang người khác thay vì sửa hệ thống ("bắt dev chia nhỏ PR", "yêu cầu khách gửi thêm bằng chứng").
- "Dùng model mạnh hơn / context window lớn hơn" để né vấn đề chất lượng chú ý — gần như luôn sai.
- Đúng về kỹ thuật nhưng không trả lời đúng câu được hỏi (Bước 1).
- Tuyệt đối hóa ("luôn escalate", "không bao giờ dùng batch") khi tình huống có sắc thái.
- Thêm hạ tầng (routing layer, preprocessing, two-pass) cho vấn đề mà vài ví dụ mẫu giải quyết được.

**Bước 5 — Với 2 đáp án còn lại, chạy đúng "quy tắc tự vấn" theo chủ đề** (bảng dưới).

**Bước 6 — Kiểm tra thiên lệch của chính mình trước khi bấm chọn.**
Trong bài thi thử, các câu sai dồn vào đáp án **C/D dài, nhiều cơ chế** khi đáp án đúng là **A/B đơn giản** (rõ nhất ở kịch bản CI/CD). Trước khi chọn C/D: *"Mình đang chọn cái này vì nó đúng nguyên nhân gốc, hay vì nó trông 'kỹ lưỡng/an toàn' hơn?"* Nếu là vế sau → xem lại A/B.

## Bảng "quy tắc tự vấn" theo chủ đề

| Nếu câu hỏi xoay quanh... | Tự hỏi |
|---|---|
| Model "quên" / hiểu sai một quy tắc | Hậu quả nghiêm trọng (tiền/pháp lý/an toàn/không undo)? → cơ chế cứng: `PreToolUse` hook, precondition code, cờ CLI, `tool_use`+schema, instance review độc lập. Chỉ là style? → prompt/few-shot. |
| Có nên dùng **few-shot** không | (1) Agent chọn sai tool vì tên/mô tả chồng lấn → sửa mô tả tool, KHÔNG few-shot. (2) Đã tốt với ca đơn giản, chỉ lúng túng 1 mẫu lặp minh họa được → few-shot, đừng thêm hạ tầng. (3) Lỗi đổi ngẫu nhiên từng case → cơ chế động, không phải few-shot cố định. |
| `.claude/rules/` vs **Skill** | "Áp dụng vì tôi chạm vào MỘT LOẠI FILE (glob) hay vì tôi làm MỘT TÁC VỤ cụ thể?" File → `rules/` + `paths`. Tác vụ/quy trình → Skill gọi theo nhu cầu. |
| Agent có nên **escalate** không | Chính sách CÓ quy định (dù bất lợi cho khách) + bằng chứng rõ → agent tự trình bày, KHÔNG escalate. Chính sách THỰC SỰ im lặng → escalate (agent không có thẩm quyền tự đặt luật). |
| **Batch API** cho quy trình CI | Batch = tối đa 24h, KHÔNG có SLA độ trễ, và không hợp với tool-calling nhiều vòng. Chỉ dùng cho việc chạy nền qua đêm (báo cáo tech-debt). Việc chặn merge / có người đang chờ → synchronous. |
| **Review nhiều file** cho kết quả không đều | Chia pass tập trung: mỗi file 1 lượt xét lỗi cục bộ + 1 pass tích hợp cho luồng dữ liệu chéo. Không phải "dùng context lớn hơn", không phải "bắt dev chia PR". |
| **Structured output** không đáng tin | `tool_use` + JSON schema (hoặc `--output-format json` / `--json-schema`), không phải dặn prompt "hãy trả JSON". |
| Lỗi tool trong hệ đa tác nhân | Trả lỗi kèm ngữ cảnh (`errorCategory`, `isRetryable`, mô tả) về coordinator để nó quyết định — không ném exception làm sập cả workflow, không tự retry mù. |
| Hội thoại rất dài, cần tra 1 kết luận cũ | **Semantic search / embedding**. Tóm tắt lũy tiến làm mất chi tiết. |
| Bỏ lời chào lặp ("Certainly!") | **Prefill** phần đầu tin nhắn assistant. Không phải hạ temperature, không phải prompt. |
| Yêu cầu người dùng mơ hồ | Nêu giả định hợp lý một cách rõ ràng rồi làm tiếp + mời chỉnh — không hỏi dồn nhiều câu, không diễn giải ngầm. |

## Quản lý thời gian & lượt rà

1. **Lượt 1** — làm tuần tự. Câu nào áp quy trình 6 bước ra đáp án trong ~1 phút thì chốt luôn. Câu phân vân giữa 2 đáp án > ~90 giây → chọn cái nhỉnh hơn theo Bước 6, **đánh dấu**, đi tiếp. Không để 1 câu ăn mất thời gian của 3 câu khác.
2. **Lượt 2** — chỉ quay lại các câu đã đánh dấu. Đọc lại dòng câu hỏi in đậm (Bước 1) — thường lần đầu trả lời lệch loại.
3. **Lượt 3** — quét nhanh đảm bảo **không câu nào bỏ trống** (không phạt đoán mò).
4. Chỉ đổi đáp án ở lượt rà khi tìm ra **lý do cụ thể** (đọc sót ràng buộc, nhầm loại câu hỏi). Đừng đổi chỉ vì "cảm thấy".

## Sai lầm tâm lý cần chặn

- **Thiên lệch "đáp án kỹ lưỡng nhất"** — đề thưởng giải pháp *đúng nguyên nhân gốc, tối giản*, không thưởng giải pháp nhiều tầng nhất.
- **Đọc tình huống trước, quên mất bị hỏi gì** — luôn Bước 1 trước.
- **Neo vào đáp án đầu tiên nghe hợp lý** — bắt buộc đọc hết 4 đáp án rồi mới loại.
- **Đem giả định ngoài đề** — chỉ dùng thông tin trong tình huống; đừng tự thêm "chắc là hệ thống còn có X".

## Ví dụ áp dụng đầy đủ — Câu 10 (đã sai: chọn D, đúng là A)

> **Tình huống:** Agent phân tích tài liệu được cấp tool đa năng `fetch_url` để tải tài liệu theo URL. Log cho thấy agent lại dùng nó để tải trang kết quả tìm kiếm — việc lẽ ra thuộc về agent web-search — gây kết quả không nhất quán. **Which fix is most effective?**
> A) Thay `fetch_url` bằng `load_document` — xác thực URL phải trỏ đến định dạng tài liệu. **[ĐÚNG]**
> B) Bỏ hẳn `fetch_url` khỏi agent này, định tuyến mọi việc tải URL qua coordinator → web-search.
> C) Lọc chặn `fetch_url` với các domain công cụ tìm kiếm đã biết.
> D) Thêm hướng dẫn vào prompt: `fetch_url` chỉ để tải tài liệu, không phải để search. *(đã chọn — sai)*

1. **Loại câu hỏi:** "Which **fix** is most effective?" → cần nhắm đúng nguyên nhân gốc.
2. **Gạch chân:** hành vi sai đã xảy ra thật, lặp lại có hệ thống (không ngẫu nhiên, không phải giọng văn) → cơ chế cứng. Từ khóa then chốt: tool **"đa năng"** = nguyên nhân gốc. Cái cần giữ: agent vẫn phải tải được tài liệu theo URL.
3. **Gắn lĩnh vực:** LV2 (Tool design) — nguyên tắc *least privilege*, không phải LV4 (prompt engineering).
4. **Loại rõ:** D = "sửa bằng prompt" cho lỗi *tool bị lạm dụng ngoài phạm vi* — đúng bẫy Nhóm 1 (xem `03-chan-doan-thi-thu.md`), tool vẫn đa năng nên không gì ngăn tái diễn. C = vá triệu chứng (danh sách domain né được bằng domain mới/redirect), tool vẫn đa năng.
5. **A vs B (đều là cơ chế cứng):** B xóa hẳn khả năng tải tài liệu của agent — quá tay, phá luôn nhu cầu hợp lệ. A thu hẹp đúng phạm vi (validate ở tầng interface) mà vẫn giữ chức năng cần — least privilege, ít thiệt hại phụ nhất.
6. **Chặn thiên lệch:** B trông "chặt chẽ, an toàn hơn" vì cắt hẳn quyền — đúng bẫy "chọn đáp án nghe kỹ lưỡng hơn" thay vì đáp án tối giản đúng nguyên nhân gốc.

→ **A**.
