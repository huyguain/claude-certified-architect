# Từ nối logic & khuôn mẫu câu (lớp nền, quan trọng hơn từ chuyên ngành ở trình độ A1)

> Bổ sung cho [`06-tu-vung-theo-cau-goc.md`](./06-tu-vung-theo-cau-goc.md). File đó có 200 từ **chuyên ngành** (dễ tra, dễ đoán qua ngữ cảnh). File này có **từ nối + động từ/tính từ học thuật lặp lại ở gần như MỌI câu** — nhóm này quyết định bạn có hiểu đúng quan hệ giữa các vế câu hay không (đối lập, nguyên nhân, điều kiện...), quan trọng hơn từ chuyên ngành rất nhiều với người mới học.
>
> **Không được dùng công cụ dịch khi thi thật** → phải nhận diện được các từ/khuôn này bằng mắt, không cần dịch từng chữ.

---

## Phần 1 — Từ nối logic (connectors)

Đây là nhóm từ nhỏ (~25 từ) nhưng lặp lại hàng trăm lần trong 76 câu. Học thuộc lòng nhóm này trước tiên — không cần hiểu hết câu, chỉ cần nhận ra "2 vế câu đang đối lập nhau" hay "đây là nguyên nhân" là đã loại được nhiều đáp án sai.

### A. Đối lập / nhượng bộ (dù... nhưng...)

| Từ | Nghĩa | Ghi chú |
|---|---|---|
| **while** | trong khi / còn (đối lập) | rất hay dùng để nối 2 vế trái ngược |
| **but** | nhưng | |
| **even though / even when** | mặc dù, dù cho | vế sau luôn là điều "đáng lẽ không nên xảy ra" |
| **however** | tuy nhiên | thường đứng đầu câu, sau dấu phẩy |
| **rather than** | thay vì (chọn A, không chọn B) | A **rather than** B = ưu tiên A |
| **instead of** | thay vì | giống "rather than" nhưng đứng linh hoạt hơn |
| **despite / regardless of** | bất kể, mặc kệ | theo sau luôn là danh từ/cụm danh từ |

> "a government report states 40% growth, **while** an industry analysis states 12%."
→ "while" ở đây báo hiệu: 2 nguồn đang **mâu thuẫn nhau**.

> "the agent often calls get_customer when users ask about order status, **even though** lookup_order would be more appropriate."
→ "even though" báo hiệu: hành vi hiện tại (gọi get_customer) là **sai**, vì lẽ ra nên dùng cái khác.

> "Test files are distributed across the codebase... and you want all tests to follow the same conventions **regardless of** location."
→ "regardless of location" = bất kể vị trí file ở đâu.

> "This is the only approach that enforces the constraint at the code level **rather than** relying on LLM compliance with instructions."
→ đáp án đúng dựa vào code, **không phải** dựa vào việc dặn dò LLM.

### B. Nguyên nhân — mục đích — điều kiện

| Từ | Nghĩa |
|---|---|
| **so that / to** (+ động từ) | để mà, nhằm mục đích |
| **because / since** | bởi vì |
| **if / unless** | nếu / trừ khi |
| **before / after** | trước khi / sau khi |
| **whether... or...** | liệu... hay là... |
| **such as** | chẳng hạn như, ví dụ như |

> "you must decide **whether** to retry, skip, or fail the whole task."
→ "whether...or" = phải chọn 1 trong nhiều lựa chọn.

> "Distinguish access failures (timeout) that require a retry decision from valid empty results ("0 results") that represent successful queries."
→ (không có "whether" nhưng là ví dụ điển hình của **so sánh 2 loại khác nhau**, xem thêm phần D).

> "Focused per-file passes address the root cause... A separate integration-oriented pass then covers cross-file concerns **such as** dependency and data-flow interactions."
→ "such as" = liệt kê ví dụ cụ thể cho ý vừa nêu.

### C. Trình tự thời gian (rất hay dùng để mô tả quy trình agent)

| Từ | Nghĩa |
|---|---|
| **before** | trước khi |
| **after** | sau khi |
| **then** | rồi thì, sau đó |
| **once** | một khi, ngay khi |
| **until** | cho đến khi |

> "Stop analysis and immediately escalate to the coordinator, asking it to decide which source is more authoritative **before** continuing."
→ "before continuing" = làm việc này TRƯỚC KHI tiếp tục việc khác — dấu hiệu của quy trình có thứ tự bắt buộc.

> "A programmatic precondition that blocks lookup_order... **until** get_customer returns a verified customer identifier."
→ "until" = chặn lại, chờ đến khi điều kiện xảy ra mới cho đi tiếp — đây là khuôn quan trọng cho câu hỏi về "precondition/enforcement".

---

## Phần 2 — Động từ & tính từ học thuật lặp lại (không phải thuật ngữ AI, nhưng xuất hiện ở MỌI câu)

Bảng dưới xếp theo **tần suất xuất hiện thực tế** trong 76 câu (đếm được qua grep) — học đúng thứ tự này để tối ưu thời gian.

| Từ | Số lần gặp | Nghĩa | Câu ví dụ |
|---|---|---|---|
| **require** | 35 | yêu cầu, đòi hỏi | "Subagents use isolated memory, and direct communication would **require** complex serialization." |
| **address** (động từ) | 28 | giải quyết, xử lý (một vấn đề) — KHÔNG phải nghĩa "địa chỉ" | "What is the most effective way to **address** this?" |
| **directly** | 27 | trực tiếp | "two credible sources contain **directly** contradictory statistics." |
| **reduce** | 20 | giảm bớt | "Local recovery **reduces** coordinator workload." |
| **specific** | 17 | cụ thể, riêng biệt | "the synthesis agent often needs to verify **specific** claims." |
| **maintain** | 16 | duy trì | "maintaining visibility into the failure" |
| **allow** | 14 | cho phép | "Return an error with context to the coordinator, **allowing** it to decide how to proceed." |
| **enable** | 13 | cho phép, giúp có thể làm được | "**enabling** the coordinator to reliably distinguish document analysis from web search." |
| **provide** | 11 | cung cấp | "The coordinator pattern **provides** centralized visibility." |
| **relevant** | 11 | liên quan, có liên hệ | "the web-search agent finds **relevant** articles." |
| **preserve** | 10 | bảo toàn, giữ nguyên | "this approach **preserves** separation of responsibilities." |
| **avoid** | 10 | tránh | "to **avoid** interrupting the workflow." |
| **ensure** | 9 | đảm bảo | "to **ensure** both sources get equal top positioning." |
| **consistent** | 9 | nhất quán | "ensuring **consistent** depth and reliable local issue detection." |
| **explicitly** | 9 | một cách rõ ràng, tường minh | "**explicitly** annotate the conflict with source attribution." |
| **effectively** | 9 | một cách hiệu quả | luôn xuất hiện trong câu hỏi: "What is the most **effective** way...?" |
| **reliable / reliably** | 7 | đáng tin cậy / một cách đáng tin cậy | "the synthesis agent **reliably** cites information..." |
| **eliminate** | 6 | loại bỏ hoàn toàn | "What is the most effective way to **eliminate** redundant feedback?" |
| **distinguish** | 6 | phân biệt | "**Distinguish** access failures... from valid empty results." |
| **indicate** | 5 | chỉ ra, cho thấy | "coverage annotations that **indicate** which conclusions are well-supported." |
| **immediately** | 5 | ngay lập tức | "any exception **immediately** terminates the subagent." |
| **determine** | 4 | xác định | "so Claude can **determine** what scenarios are already covered." |
| **appropriate** | 4 | phù hợp, thích hợp | "**Which next step is most appropriate?**" (khuôn câu hỏi rất hay gặp) |

**Mẹo nhớ nhanh:** hầu hết các từ này đi theo cặp **vấn đề → hành động khắc phục**:
- *reduce / eliminate / avoid / prevent* = LÀM GIẢM/MẤT một điều xấu (lỗi, trùng lặp, quá tải...)
- *ensure / maintain / preserve* = GIỮ một điều tốt (tính nhất quán, ngữ cảnh, độ tin cậy...)
- *enable / allow / provide* = TẠO ĐIỀU KIỆN cho một hành động xảy ra được

---

## Phần 3 — Khuôn mẫu câu lặp lại (không cần hiểu ngữ pháp, chỉ cần nhận diện khuôn)

Đề thi này dùng đi dùng lại rất ít khuôn câu. Thuộc khuôn = đọc nhanh hơn nhiều dù vốn từ ít.

### Khuôn câu hỏi chính (luôn ở cuối phần tình huống)

| Khuôn tiếng Anh | Ý nghĩa | Phải trả lời loại gì |
|---|---|---|
| "**What is the most effective way to** [động từ] this?" | Cách hiệu quả nhất để làm gì đó | Giải pháp |
| "**Which approach is most effective?**" | Cách tiếp cận nào hiệu quả nhất | Giải pháp |
| "**What is the most likely root cause?**" | Nguyên nhân gốc rễ có khả năng nhất là gì | Nguyên nhân, KHÔNG phải giải pháp |
| "**What is the next step?**" | Bước tiếp theo là gì | Hành động kế tiếp trong quy trình |
| "**How should you [động từ]...?**" | Bạn nên làm X như thế nào | Cách thực hiện |
| "**Which approach best restores/ensures/enables...?**" | Cách nào giúp khôi phục/đảm bảo/cho phép... tốt nhất | Giải pháp nhắm đúng mục tiêu nêu trong động từ |
| "**What [something] addresses the root cause?**" | Thay đổi nào giải quyết đúng nguyên nhân gốc | Giải pháp gốc rễ, không phải giải pháp vá tạm |

### Khuôn mô tả tình huống "so sánh trước/sau" hoặc "trường hợp A vs trường hợp B"

> "Your agent handles single-issue requests with 94% accuracy... **But when** customers include multiple issues in one message... accuracy drops to 58%."

Khuôn: **"X works fine in case 1. But in case 2 (phức tạp hơn), X fails."** — rất hay gặp, báo hiệu câu hỏi đang hỏi giải pháp cho phần **case 2**, không phải sửa lại case 1 (case 1 đã ổn, đừng động vào).

### Khuôn giải thích đáp án đúng ("Why X: ...")

Phần giải thích luôn có cấu trúc: **[Đáp án đúng] + vì nó [động từ hiệu quả] + [vấn đề], trong khi [đáp án khác] + [nhược điểm]**.

> "Why D: The two-tool token-binding approach makes it architecturally impossible to execute without a prior preview... **rather than** relying on LLM compliance with instructions (C), timing heuristics (A), or orchestration infrastructure (B)."

→ Khuôn: đáp án đúng dùng cơ chế **cứng** (architecturally impossible), các đáp án sai dựa vào **LLM tự giác tuân thủ** (relying on compliance) — đây cũng là pattern chiến thuật đã có ở `05-chien-thuat-lam-bai.md`, nhưng nhìn từ góc độ ngôn ngữ: cụm "**rather than relying on**" gần như luôn là dấu hiệu chỉ ra đáp án SAI ở phía sau nó.

### Khuôn liệt kê % / số liệu (luôn là dữ kiện then chốt để loại đáp án)

> "8% are network timeouts that succeed when retried, **and** 4% are query syntax errors that never succeed **regardless of** retries."

Khuôn: **"X% là loại lỗi A (retry được), Y% là loại lỗi B (retry không được)"** → đáp án đúng luôn là đáp án **phân biệt 2 loại** (distinguish), đáp án sai luôn là đáp án **gộp chung xử lý cùng 1 kiểu** (uniformly / identically).

---

## Cách ôn nhóm này

1. Học **Phần 1 (từ nối)** trước — chỉ ~25 từ, nhưng ảnh hưởng đến việc hiểu đúng/sai mọi câu.
2. Học **Phần 2** theo đúng thứ tự tần suất trong bảng (từ `require` xuống `appropriate`).
3. Đọc lại **Phần 3** ngay trước khi thi — không cần học thuộc, chỉ cần "quen mặt" khuôn câu để đọc nhanh hơn, không bị khớp lần đầu thấy câu dài.
4. Kết hợp với [`05-chien-thuat-lam-bai.md`](./05-chien-thuat-lam-bai.md): khuôn ngôn ngữ ở đây + chiến thuật loại đáp án ở đó là 2 nửa của cùng một kỹ năng.
