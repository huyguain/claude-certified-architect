# Từ vựng học theo câu gốc (không học từ đơn lẻ)

> Nguồn: 76 câu hỏi luyện tập trong [`03-chan-doan-thi-thu.md`](./03-chan-doan-thi-thu.md) (bản tiếng Anh: artifact `e428cc2d-...` — bài làm đã chấm).

**Cách dùng file này:**
1. Che phần nghĩa (cột phải / dòng gạch đầu dòng), chỉ đọc câu tiếng Anh trong khối `>` — cố đoán nghĩa từ in đậm dựa vào ngữ cảnh trước.
2. Lật ra xem nghĩa để kiểm tra.
3. Từ nào đoán sai 2 lần trở lên → note lại riêng để ôn ưu tiên (Anki/spaced repetition).
4. Học theo cụm câu, không tách từ ra học rời — nhớ theo collocation (cụm từ đi cùng nhau) sẽ bền hơn nhiều.

---

## 1. Multi-agent Research System (Q1–Q15)

**Q1**
> "Both sources look **credible**, and the **discrepancy** could **materially** affect the research conclusions."
- credible: đáng tin cậy
- discrepancy: sự khác biệt/mâu thuẫn (số liệu)
- materially (affect): ảnh hưởng đáng kể

> "Apply credibility **heuristics** to pick the most likely correct number... add a **footnote** mentioning the discrepancy."
- heuristics: nguyên tắc/thủ thuật suy luận gần đúng
- footnote: chú thích cuối trang

> "Stop analysis and immediately **escalate** to the coordinator, asking it to decide which source is more **authoritative**."
- escalate: chuyển/báo cáo vấn đề lên cấp cao hơn
- authoritative: có thẩm quyền, đáng tin cậy nhất

> "Complete analysis with both numbers, explicitly **annotate** the conflict with source **attribution**, and let the coordinator decide how to **reconcile** the data before passing to **synthesis**."
- annotate: chú thích, gắn nhãn
- attribution: sự quy kết nguồn gốc
- reconcile: dung hòa, giải quyết mâu thuẫn dữ liệu
- synthesis: sự tổng hợp

**Q2**
> "The coordinator **forwards** both result sets to the synthesis agent for **centralized** integration, preserving control."
- forwards: chuyển tiếp
- centralized: tập trung

> "Each agent sends its results directly to the report-writing agent, **bypassing** the coordinator."
- bypassing: bỏ qua, đi vòng qua

> "The coordinator **concatenates** the raw outputs from both agents."
- concatenates: nối/ghép (chuỗi kết quả)

**Q3**
> "some have **corrupted** sections that trigger parsing exceptions, others are **password-protected**, and sometimes the parsing library **hangs** on large files."
- corrupted: bị hỏng (dữ liệu)
- password-protected: được bảo vệ bằng mật khẩu
- hangs: bị treo, đứng máy

> "Currently, any exception immediately **terminates** the subagent... This causes **excessive** coordinator involvement in **routine** error handling."
- terminates: chấm dứt
- excessive: quá mức
- routine: thường lệ, thông thường

> "Implement local recovery in the subagent for **transient** failures and escalate... errors it cannot resolve, including attempted steps and partial results."
- transient (failure): (lỗi) tạm thời, thoáng qua

> "still escalating truly **unrecoverable** issues with full context."
- unrecoverable: không thể khôi phục

**Q4**
> "the coordinator **decomposed** the topic into three subtasks."
- decomposed: phân tách, chia nhỏ

> "The coordinator's task **decomposition** is too **narrow**, assigning subagents work that does not cover all relevant areas."
- decomposition: sự phân tách
- narrow: hẹp, thiếu bao quát

> "Since subagents executed their assignments correctly, the narrow decomposition is the obvious **root cause**."
- root cause: nguyên nhân gốc rễ

**Q5**
> "news archives and social feeds **time out**."
- time out: hết thời gian chờ

> "Structure the synthesis output with **coverage annotations** that indicate which conclusions are well-supported and where gaps exist."
- coverage annotations: chú thích về độ bao phủ dữ liệu

> "Coverage annotations implement **graceful degradation** with **transparency**, preserving value from completed work while **propagating uncertainty**."
- graceful degradation: suy giảm từ từ (không sập đột ngột)
- transparency: sự minh bạch
- propagating (uncertainty): lan truyền (sự không chắc chắn)

**Q6**
> "Silently skip the corrupted document... to avoid **interrupting** the workflow."
- interrupting: làm gián đoạn

> "Automatically retry parsing the document three times with **exponential backoff**."
- exponential backoff: giãn cách thử lại theo cấp số nhân

> "lets the coordinator make an **informed decision**—skip the file, try an alternative parsing method, or notify the user."
- informed decision: quyết định dựa trên đầy đủ thông tin

**Q7**
> "requests... are **routed** to the web-search agent 45% of the time instead of the document analysis agent."
- routed: được định tuyến

> "eliminating **semantic overlap** between the two tool names and descriptions. This makes each tool's purpose **unambiguous**."
- semantic overlap: sự chồng lấn về nghĩa
- unambiguous: rõ ràng, không mơ hồ

**Q8**
> "The coordinator can observe all interactions, handle errors **uniformly**, and decide what information each subagent should receive."
- uniformly: một cách đồng nhất

> "The coordinator pattern provides **centralized visibility**... and **fine-grained control** over what information each subagent receives."
- centralized visibility: khả năng quan sát tập trung
- fine-grained control: khả năng kiểm soát chi tiết/tinh vi

> "the primary advantages of a **star-shaped communication topology**."
- star-shaped topology: cấu trúc liên kết hình sao

**Q9**
> "Return **structured** error context... including the failure type, the query executed, any partial results, and potential **alternative approaches**."
- structured: có cấu trúc
- alternative approaches: cách tiếp cận thay thế

> "only returning a **generic** "search unavailable" status after **exhausting** retries."
- generic: chung chung
- exhausting (retries): dùng hết, cạn kiệt (các lần thử lại)

> "**Propagate** the timeout exception directly to the top-level handler, terminating the entire research workflow."
- propagate: lan truyền, truyền đi

**Q10**
> "this agent now frequently downloads search engine results pages to perform **ad hoc** web search—behavior... causing **inconsistent** results."
- ad hoc: tùy biến, không theo kế hoạch sẵn
- inconsistent: không nhất quán

> "Replace fetch_url with a load_document tool that **validates** that URLs point to document formats."
- validates: xác thực, kiểm tra hợp lệ

> "This follows the **principle of least privilege**, making **undesired** search behavior impossible rather than merely **discouraged**."
- principle of least privilege: nguyên tắc đặc quyền tối thiểu
- undesired: không mong muốn
- discouraged: bị khuyến cáo không nên làm (nhưng vẫn có thể)

**Q11**
> "leading to **substantial** duplication in their outputs. Token usage nearly doubles without a **proportional** increase in research breadth or depth."
- substantial: đáng kể
- proportional: tương xứng, tỉ lệ thuận
- breadth / depth: chiều rộng / chiều sâu

> "The coordinator explicitly **partitions** the research space before delegating, assigning each agent **distinct** subtopics."
- partitions: phân chia
- distinct: riêng biệt, khác biệt

> "addresses the root cause—**unclear** task boundaries—before any work begins."
- unclear: không rõ ràng

**Q12**
> "academic databases return 15 relevant papers, industry reports return "0 results," and patent databases return "Connection timeout.""
> "**Distinguish** access failures (timeout)... from valid empty results ("0 results") that represent successful queries."
- distinguish: phân biệt

> "A timeout... and "0 results"... are **semantically** different outcomes requiring different responses."
- semantically: về mặt ngữ nghĩa

**Q13**
> "the synthesis agent **reliably cites** information from the first 15K tokens... but often misses **critical findings** in the middle 50K tokens."
- reliably cites: trích dẫn một cách đáng tin cậy
- critical findings: các phát hiện quan trọng

> "Place a key-findings summary at the start... and organize detailed results with **explicit** section headings for easier **navigation**."
- explicit: rõ ràng, tường minh
- navigation: sự điều hướng (đọc/duyệt nội dung)

> "**leverages** primacy effects... directly **mitigating** the "lost in the middle" **phenomenon**."
- leverages: tận dụng
- mitigating: giảm nhẹ
- phenomenon: hiện tượng

**Q14**
> "the document analysis agent (70K tokens including **chains of thought**)."
- chains of thought: chuỗi suy luận (của mô hình)

> "Modify upstream agents to return structured data... instead of **verbose** content and reasoning."
- verbose: dài dòng

> "It avoids passing **bulky** page content and reasoning traces that **inflate** tokens."
- bulky: cồng kềnh
- inflate: làm phồng lên (không cần thiết)

**Q15**
> "Your assessment shows 85% of these verifications are simple fact checks... and 15% require deeper research."
> "Give the synthesis agent a **limited-scope** verify_fact tool for simple checks, while routing complex verifications through the coordinator."
- limited-scope: phạm vi giới hạn

> "eliminating most loops... This applies **least privilege** while significantly reducing **latency**."
- latency: độ trễ

---

## 2. Claude Code for Continuous Integration (Q16–Q30)

**Q16**
> "Claude outputs **narrative paragraphs** that must be manually copied into PR comments."
- narrative paragraphs: đoạn văn dạng tường thuật/kể chuyện

> "Use the CLI flags --output-format json... to **enforce** structured findings."
- enforce: ép buộc, áp đặt

> "guaranteeing **well-formed** JSON... that can be reliably **parsed**."
- well-formed: đúng định dạng
- parsed: được phân tích cú pháp

**Q17**
> "**non-obvious** issues—performance optimizations that break edge cases... are only caught when another team member reviews the PR."
- non-obvious: không rõ ràng ngay, khó nhận ra

> "Run a second **independent** instance of Claude Code to review the changes without access to the generator's reasoning."
- independent: độc lập

> "directly addresses the root cause by avoiding **confirmation bias**. This "**fresh eyes**" perspective **mirrors** human **peer review**."
- confirmation bias: thiên kiến xác nhận
- fresh eyes: góc nhìn mới, chưa bị ảnh hưởng
- mirrors: phản ánh, giống như
- peer review: bình duyệt đồng cấp

**Q18**
> "Claude analyzes the changed file, then may request related files... via tool calls."
> "The asynchronous model cannot execute tools **mid-request** and return results for Claude to continue analysis."
- mid-request: giữa chừng một request

> "A "**fire-and-forget**" asynchronous Batch API model has no **mechanism** to **intercept** a tool call during a request."
- fire-and-forget: gửi đi rồi không chờ phản hồi
- mechanism: cơ chế
- intercept: chặn lại, can thiệp giữa chừng

**Q19**
> "fast style checks on every PR that **block** merging until completion."
- block: chặn, ngăn cản

> "**comprehensive** weekly security **audits** of the entire codebase."
- comprehensive: toàn diện
- audits: cuộc kiểm toán/rà soát

> "scheduled tasks with **flexible deadlines** that can **tolerate** up to a 24-hour batch window."
- flexible deadlines: thời hạn linh hoạt
- tolerate: chịu được, dung sai

**Q20**
> "developers report the feedback is not **actionable**. Findings include phrases... without **specifying** what exactly to change."
- actionable: thiết thực, có thể hành động ngay
- specifying: chỉ rõ, xác định cụ thể

> "the model still produces **inconsistent** output—sometimes detailed, sometimes **vague**."
- inconsistent: không nhất quán
- vague: mơ hồ

> "Add 3–4 **few-shot examples** showing the exact required format."
- few-shot examples: ví dụ minh họa mẫu (số lượng ít) đưa vào prompt

**Q21**
> "a pre-merge-commit **hook** that blocks PR merge until completion, and a "deep analysis" that runs overnight, **polls** for batch completion."
- hook: móc nối (đoạn code chạy tự động khi có sự kiện)
- polls: thăm dò định kỳ

> "Deep analysis is an **ideal candidate** for batch processing because it already runs overnight, tolerates delay."
- ideal candidate: ứng viên lý tưởng

**Q22**
> "Findings often **flag** acceptable patterns (TODO markers, simple descriptions) while missing comments describing behavior the code no longer implements."
- flag (v.): gắn cờ, đánh dấu (cảnh báo)

> "Include git blame data so Claude can identify comments that **predate** recent code changes."
- predate: có trước, xảy ra trước

> "Specify **explicit criteria**: flag comments only when the behavior they claim **contradicts** the code's actual behavior."
- explicit criteria: tiêu chí rõ ràng
- contradicts: mâu thuẫn với

**Q23 / Q29**
> "similar issues like null pointer risks are rated "critical" in some PRs but only "medium" in others."
> "Developer surveys show growing **distrust**—many start **dismissing** findings without reading."
- distrust: sự mất lòng tin
- dismissing: gạt bỏ, bác bỏ

> "High-false-positive categories **erode trust** in accurate categories."
- erode trust: làm xói mòn niềm tin

> "Temporarily disable high-false-positive categories... and keep only **high-precision** categories while improving prompts."
- high-precision: có độ chính xác cao

**Q24**
> "Claude suggests 10 test cases, but developer feedback shows that 6 **duplicate** scenarios already covered by the existing test suite."
- duplicate (adj.): trùng lặp

> "Implement post-processing that filters suggestions whose descriptions match existing test names via **keyword overlap**."
- keyword overlap: sự trùng từ khóa

**Q25**
> "developers report that 5 duplicate previous comments on code that was already fixed in the new commits."
> "Run review only when the PR is created and in the final pre-merge state, skipping **intermediate** commits."
- intermediate: trung gian, ở giữa

> "Include previous review findings in context and instruct Claude to report only new or still-**unresolved** issues."
- unresolved: chưa được giải quyết

**Q26**
> "the job hangs **indefinitely**. Logs show Claude Code is waiting for **interactive input**."
- indefinitely: vô thời hạn
- interactive input: dữ liệu nhập tương tác (chờ người dùng gõ)

> "The -p (or --print) flag is the documented way to run Claude Code **non-interactively**."
- non-interactively: không cần tương tác (chạy tự động)

**Q27**
> "detailed feedback on some files but **shallow** comments on others, missed obvious bugs, and **contradictory** feedback."
- shallow: hời hợt, nông
- contradictory: mâu thuẫn nhau

> "a pattern is **flagged** in one file but **identical** code is **approved** in another file in the same PR."
- flagged: bị gắn cờ cảnh báo
- identical: giống hệt
- approved: được chấp thuận

> "Split into focused passes: review each file individually... then run a separate **integration-oriented** pass to examine **cross-file** data flows."
- integration-oriented: hướng tới việc tích hợp
- cross-file: liên quan nhiều file khác nhau

> "Focused per-file passes address the root cause—**attention dilution**."
- attention dilution: sự pha loãng độ tập trung

**Q28**
> "The **bottleneck** is investigation time: developers must click into each finding to read Claude's **rationale**."
- bottleneck: nút thắt cổ chai
- rationale: lý do, cơ sở lập luận

> "**stakeholders** rejected any approach that filters findings before developers see them."
- stakeholders: các bên liên quan

> "reduces investigation time by letting developers quickly **triage** without opening each finding."
- triage: phân loại mức độ ưu tiên xử lý

**Q30**
> "a **technical debt** report generated overnight for review the next morning."
- technical debt: nợ kỹ thuật

> "Message Batches API processing can take up to 24 hours with no **latency SLA**, which is acceptable... but **unacceptable** for blocking pre-merge checks."
- latency SLA: cam kết chất lượng dịch vụ về độ trễ
- unacceptable: không thể chấp nhận được

---

## 3. Code Generation with Claude Code (Q31–Q45)

**Q31**
> "the output structure still doesn't match expectations—some fields are **nested** differently and timestamps are formatted incorrectly."
- nested: lồng nhau

> "You described requirements in **prose**, but Claude **interprets** them differently each time."
- prose: văn xuôi (mô tả bằng lời thường, không có cấu trúc)
- interprets: diễn giải

> "Provide 2–3 **concrete** input-output examples showing the expected **transformation** for **representative** API responses."
- concrete: cụ thể
- transformation: sự biến đổi
- representative: mang tính đại diện

**Q32**
> "Slack's API offers **fundamentally** different integration approaches—**incoming webhooks** (simple, one-way), bot tokens (support **delivery confirmation** and **programmatic control**), or Slack Apps (two-way events, requires **workspace approval**)."
- fundamentally: về căn bản
- incoming webhooks: webhook nhận sự kiện đến
- delivery confirmation: xác nhận đã gửi thành công
- programmatic control: điều khiển bằng chương trình
- workspace approval: sự phê duyệt của không gian làm việc

> "Switch to planning mode to explore integration options and **architectural implications**."
- architectural implications: hệ quả/tác động về mặt kiến trúc

> "Planning mode lets you evaluate **trade-offs** among webhooks, bot tokens, and Slack Apps and **align** on an approach."
- trade-offs: sự đánh đổi
- align: thống nhất, đồng thuận

**Q33**
> "Your CLAUDE.md file has grown to 400+ lines containing coding standards, testing conventions, a detailed PR review **checklist**, **deployment** instructions, and database **migration** procedures."
- checklist: danh sách kiểm tra
- deployment: việc triển khai
- migration: sự di trú/chuyển đổi (dữ liệu, schema)

> "Split CLAUDE.md into files... with **path-bound** glob patterns so each rule loads only for the relevant file types."
- path-bound: gắn/ràng buộc theo đường dẫn

> "create Skills for **workflow-specific** guidance... with **trigger keywords**."
- workflow-specific: đặc thù cho từng quy trình làm việc
- trigger keywords: từ khóa kích hoạt

**Q34**
> "**restructuring** your team's **monolithic** application into **microservices**."
- restructuring: tái cấu trúc
- monolithic: nguyên khối
- microservices: vi dịch vụ

> "it allows safe exploration and informed decisions about boundaries before **committing** to potentially expensive changes."
- committing (to a change): cam kết/thực hiện dứt khoát

**Q35**
> "performs deep code analysis—**dependency scanning**, test coverage counts, and code quality **metrics**."
- dependency scanning: quét sự phụ thuộc
- metrics: các chỉ số đo lường

> "Claude becomes less **responsive** in the session and loses the context of the original task."
- responsive: đáp ứng nhanh, phản hồi tốt

> "context: fork runs the analysis in an isolated subagent context so the large output does not **pollute** the main session's context window."
- pollute: làm ô nhiễm, làm nhiễu

**Q36**
> "A developer wants to **customize** it for their **personal workflow**... without affecting teammates."
- customize: tùy chỉnh
- personal workflow: quy trình làm việc cá nhân

> "Personal skills take **precedence** over project skills with the same name."
- precedence: quyền ưu tiên

**Q37**
> "three developers report Claude follows the guidance... but a fourth developer who just **joined** says Claude does not follow it."
- joined: vừa gia nhập, tham gia

> "The guidance lives in the original developers' **user-level** ~/.claude/CLAUDE.md files, not in the **project-level** file."
- user-level / project-level: cấp người dùng / cấp dự án

**Q38**
> "including 2–3 full endpoint implementation examples as context **significantly** improves consistency."
- significantly: một cách đáng kể

> "this context is useful only when creating new endpoints—not when **debugging**, reviewing code."
- debugging: gỡ lỗi

> "Create a skill... invoked on demand via a **slash command**."
- slash command: lệnh bắt đầu bằng dấu gạch chéo (/)

**Q39**
> "developers often run the skill without arguments, causing **poorly named** files."
- poorly named: được đặt tên kém/không tốt

> "the skill sometimes uses database schema details from **unrelated prior** conversations."
- unrelated: không liên quan
- prior: trước đó

> "a developer **accidentally** ran **destructive** test cleanup when the skill had **broad** tool access."
- accidentally: một cách vô tình
- destructive: mang tính phá hủy
- broad (access): rộng, không giới hạn

> "context: fork prevents context **leakage** from prior conversations, and allowed-tools **constrains** the skill to safe file-writing operations."
- leakage: sự rò rỉ
- constrains: giới hạn, ràng buộc

**Q40**
> "React components use **functional style** with hooks... database models follow the **repository pattern**."
- functional style: phong cách lập trình hàm
- repository pattern: mẫu thiết kế repository

> "Create rule files... specifying glob patterns to **conditionally** apply conventions based on file paths."
- conditionally: một cách có điều kiện

**Q41**
> "It should be available to every developer when they **clone** or update the repository."
- clone: sao chép (kho mã nguồn) về máy

**Q42**
> "Developers find it hard to **locate** and update the right sections."
- locate: xác định vị trí, tìm ra

> "allowing teams to organize large instruction sets into focused, **maintainable modules**."
- maintainable: dễ bảo trì
- modules: các mô-đun

**Q43**
> "your team uses to **brainstorm** and **evaluate** implementation approaches before choosing one."
- brainstorm: động não, đưa ra nhiều ý tưởng
- evaluate: đánh giá

> "**subsequent** Claude responses are influenced... sometimes **referencing** rejected approaches or **retaining** exploration context that **interferes with** actual implementation."
- subsequent: tiếp theo sau đó
- referencing: nhắc lại, tham chiếu đến
- retaining: giữ lại
- interferes with: gây cản trở, can thiệp vào

**Q44**
> "Each of six developers has their own personal GitHub **access token**. You want consistent tooling... without **committing credentials** to version control."
- access token: mã truy cập
- committing credentials: đưa thông tin xác thực vào (kho lưu trữ mã nguồn)

> "it provides a single version-controlled **source of truth** for MCP configuration while letting each developer **supply** credentials via environment variables."
- source of truth: nguồn dữ liệu chuẩn/gốc duy nhất
- supply: cung cấp

**Q45**
> "The work has three phases: (1) discover all **call sites** and patterns, (2) **collaboratively** design the error-handling approach."
- call sites: các vị trí gọi hàm/API trong code
- collaboratively: một cách hợp tác, cùng nhau

> "Claude generates large output... quickly **filling** the context window before discovery finishes."
- filling: lấp đầy

> "Use an Explore subagent for Phase 1 to **isolate** verbose discovery output and return a **concise** summary."
- isolate: cô lập, tách riêng
- concise: ngắn gọn, súc tích

---

## 4. Customer Support Agent (Q46–Q60)

**Q46**
> "the agent often calls get_customer when users ask about order status, even though lookup_order would be **more appropriate**."
- more appropriate: phù hợp hơn

> "Tool descriptions are the **primary** input the model uses to decide which tool to call."
- primary: chính, chủ yếu

> "the first **diagnostic** step is to verify that tool descriptions clearly separate each tool's purpose and **usage boundaries**."
- diagnostic: mang tính chẩn đoán
- usage boundaries: ranh giới sử dụng

**Q47**
> "The agent usually solves only one issue or **mixes** parameters across requests."
- mixes: trộn lẫn, gộp nhầm

> "Implement a **preprocessing** layer that uses a separate model call to **decompose** multi-issue messages into separate requests, handle each **independently**, and **merge** results."
- preprocessing: tiền xử lý
- decompose: phân tách
- independently: một cách độc lập
- merge: hợp nhất, gộp lại

> "Add few-shot examples... **demonstrating** correct reasoning and tool **sequencing**."
- demonstrating: minh họa, chứng minh
- sequencing: sắp xếp trình tự

**Q48**
> "the agent averages 12+ tool calls with only 54% success—often investigating issues **sequentially** and **fetching redundant** customer data for each."
- sequentially: theo trình tự tuần tự
- fetching redundant data: lấy dữ liệu bị trùng lặp/thừa

> "Decompose the request into separate issues, then investigate each **in parallel** using **shared** customer context before **synthesizing** a final resolution."
- in parallel: song song
- shared: được chia sẻ chung
- synthesizing: tổng hợp lại

**Q49**
> "it **escalates** simple cases (standard replacements for damaged goods with photo proof) while trying to handle complex situations requiring **policy exceptions autonomously**."
- damaged goods: hàng hóa bị hư hỏng
- photo proof: bằng chứng ảnh chụp
- policy exceptions: các trường hợp ngoại lệ của chính sách
- autonomously: một cách tự chủ, tự động

> "directly address the root cause—unclear **decision boundaries** between simple and complex cases."
- decision boundaries: ranh giới ra quyết định

**Q50**
> "A customer **claims** they didn't receive an order, but **tracking** shows it was **delivered** and **signed for** at their address three days ago."
- claims: tuyên bố, khẳng định (thường hàm ý chưa chắc đúng)
- tracking: dữ liệu theo dõi vận chuyển
- delivered: đã được giao
- signed for: đã có người ký nhận

> "A customer requests **competitor price matching**. Your policies allow **price adjustments** for **price drops** on your own site."
- competitor price matching: khớp giá theo đối thủ cạnh tranh
- price adjustments: điều chỉnh giá
- price drops: giảm giá

> "This is a **genuine** policy gap... The agent must not **invent** policy and should escalate for human **judgment**."
- genuine: thực sự, chính đáng
- invent: bịa ra, tự tạo ra
- judgment: sự phán xét, phán đoán

**Q51**
> "your agent skips get_customer and calls lookup_order directly using only the **customer-provided** name, sometimes leading to **misidentified** accounts."
- customer-provided: do khách hàng cung cấp
- misidentified: bị nhận diện sai

> "Add a programmatic **precondition** that blocks lookup_order... until get_customer returns a **verified** customer **identifier**."
- precondition: điều kiện tiên quyết
- verified: đã được xác minh
- identifier: mã định danh

> "A programmatic precondition provides a **deterministic guarantee** that required sequencing is followed."
- deterministic: có tính xác định (không ngẫu nhiên)
- guarantee: sự đảm bảo

**Q52**
> "the agent provides accurate solutions but **inconsistently** explains rationale: sometimes **omitting** relevant policy details."
- inconsistently: một cách không nhất quán
- omitting: bỏ sót, lược bỏ

> "Add a **self-critique** stage where the agent evaluates a **draft** response for **completeness**—...and **anticipates follow-up** questions."
- self-critique: tự phê bình, tự đánh giá
- draft: bản nháp
- completeness: tính đầy đủ
- anticipates follow-up: lường trước câu hỏi tiếp theo

> "forcing the agent to **assess** its own draft against **concrete criteria**... without human **oversight**."
- assess: đánh giá
- concrete criteria: tiêu chí cụ thể
- oversight: sự giám sát

**Q53**
> "Claude often requests get_customer and lookup_order in separate **sequential turns** even when both are needed initially."
- sequential turns: các lượt tuần tự (nối tiếp nhau)

> "Create **composite** tools like get_customer_with_orders that **bundle** common lookup combinations into single calls."
- composite: tổng hợp, kết hợp
- bundle: gộp lại, đóng gói chung

> "leverages its **native ability** to request multiple tools at once."
- native ability: khả năng vốn có

**Q54**
> "these details were mentioned 20+ turns ago and **condensed** into vague summaries."
- condensed: được cô đọng lại

> "Extract **transactional facts** (amounts, dates, order numbers) into a persistent "case facts" **block**."
- transactional facts: các dữ kiện liên quan đến giao dịch
- block: khối (dữ liệu)

> "Summarization **inherently** loses **precise** details."
- inherently: về bản chất, vốn dĩ
- precise: chính xác, chi tiết

**Q55**
> "this selects the wrong account 15% of the time for **ambiguous matches**."
- ambiguous matches: các kết quả khớp mơ hồ, không rõ ràng

> "Instruct Claude to request an **additional** identifier... before taking any customer-specific action."
- additional: bổ sung, thêm vào

> "the user has **definitive** knowledge of their identity."
- definitive: chắc chắn, dứt khoát

**Q56**
> "The system prompt contains **keyword-sensitive** instructions that **steer** behavior based on terms like "account," creating **unintended** tool-selection patterns."
- keyword-sensitive: nhạy với từ khóa cụ thể
- steer: định hướng, chi phối
- unintended: không chủ ý

> "The **systematic** keyword-driven pattern... strongly indicates explicit routing logic."
- systematic: có tính hệ thống

**Q57**
> "Both tools have **minimal** descriptions... and accept **similar-looking** identifier formats."
- minimal: tối thiểu, sơ sài
- similar-looking: trông giống nhau

> "a **low-effort, high-impact** first step that improves the primary mechanism the LLM uses for tool selection."
- low-effort, high-impact: ít công sức, tác động lớn

**Q58**
> "Check the **stop_reason** field in Claude's response—continue if it is tool_use and stop if it is end_turn."
> "stop_reason is Claude's explicit structured **signal** for loop control."
- signal: tín hiệu

**Q59**
> "the agent **misinterprets** outputs from your MCP tools: Unix timestamps... ISO 8601 dates... and **numeric status codes**."
- misinterprets: hiểu sai, diễn giải sai
- numeric status codes: mã trạng thái dạng số

> "Use a PostToolUse hook to intercept tool outputs and apply formatting **transformations**."
- transformations: các phép biến đổi

> "A PostToolUse hook provides a centralized, deterministic point to intercept and **normalize** all tool outputs—including **third-party** MCP server data."
- normalize: chuẩn hóa
- third-party: bên thứ ba

**Q60**
> "especially for ambiguous queries like "I need help with my recent purchase.""
> "Add 4–6 examples **targeted at** ambiguous scenarios, each with rationale for why one tool was chosen over **plausible** alternatives."
- targeted at: nhắm/hướng vào
- plausible: hợp lý, có vẻ đúng

> "teaches the model the **comparative** decision process needed for edge cases."
- comparative: mang tính so sánh

---

## 5. Conversational AI Architecture Patterns (Q61–Q76)

**Q61**
> "Production monitoring shows the agent **bypasses** the preview step by calling with dry_run=false directly."
- bypasses: bỏ qua, lách qua

> "Add server-side validation that **permits** dry_run=false only when a dry_run=true call with **identical** parameters occurred within the past 60 seconds."
- permits: cho phép
- identical: giống hệt

> "configure the orchestration layer to prompt the user for **approval** before forwarding any calls to annotated tools."
- approval: sự phê duyệt

> "execute_remove_member requires that token, **binding** execution to the preview."
- binding: ràng buộc

> "The two-tool token-binding approach makes it **architecturally** impossible to execute without a **prior** preview."
- architecturally: về mặt kiến trúc
- prior: trước đó

**Q62**
> "8% are network timeouts that succeed when retried, and 4% are **query syntax errors** that never succeed regardless of retries."
- query syntax errors: lỗi cú pháp truy vấn

> "return syntax errors immediately with **parameter validation** details."
- parameter validation: xác thực tham số

> "the tool has **definitive** knowledge of the error type and can implement **deterministic** retry logic without **relying on** the agent to interpret a flag."
- definitive: dứt khoát, chắc chắn
- relying on: dựa vào, phụ thuộc vào

**Q63**
> "a user stated "I have a very low **risk tolerance**" and later "I want to **maximize** my returns.""
- risk tolerance: mức độ chấp nhận rủi ro
- maximize: tối đa hóa

> "**Surface** the contradiction and ask the user to clarify which matters more."
- surface (v.): đưa ra ánh sáng, làm nổi bật

> "maximizing returns and low risk tolerance are fundamentally **incompatible** goals that require a human decision."
- incompatible: không tương thích, xung khắc

**Q64**
> "Two messages after a user said "I love jazz," Claude asks "What **genres** do you enjoy?""
- genres: các thể loại

> "Your application isn't including prior messages in the messages **array**."
- array: mảng (dữ liệu)

> "Claude has no server-side memory—every API call is **stateless**."
- stateless: không lưu trạng thái

**Q65**
> "History includes **allergies**, recipe scaling, clarified cooking terms, and general discussion."
- allergies: các dị ứng

> "Extract critical structured data (allergies, **quantities**, preferences), summarize general discussion, and keep recent **exchanges** verbatim."
- quantities: số lượng, định lượng
- exchanges: các lượt trao đổi (hội thoại)
- verbatim: nguyên văn

> "Critical facts... are extracted into a **compact** structured block (preventing the **precision loss** that occurs during summarization)."
- compact: gọn, cô đọng
- precision loss: sự mất độ chính xác

**Q66**
> "Users report that during **extended** conversations the assistant **loses track of** earlier topics and preferences."
- extended: kéo dài
- loses track of: mất dấu, không theo dõi được nữa

> "maintaining a **compressed representation** of earlier preferences (preventing total loss when pairs are **dropped**)."
- compressed representation: biểu diễn được nén lại
- dropped: bị loại bỏ, bị xóa

**Q67**
> "Users report that latency increases and costs rise when conversations **exceed** 50 turns."
- exceed: vượt quá

> "The model does not **maintain** any internal state between calls."
- maintain: duy trì

**Q68**
> "the assistant gives **generic** answers instead of referencing previous discussions."
- generic: chung chung

> "**Semantic embeddings** with **retrieval** of relevant exchanges."
- semantic embeddings: biểu diễn vector mang ngữ nghĩa
- retrieval: sự truy xuất, tìm lại

> "Progressive summarization compresses discussions into **abstractions** that lose the specific conclusions users are asking about."
- abstractions: sự trừu tượng hóa

**Q69**
> "Claude follows system prompt guidelines for the first 10–15 turns, but later responses **deviate**."
- deviate: đi chệch hướng, lệch khỏi

> "Insert user-role messages **reinforcing** guidelines at conversation **breakpoints**."
- reinforcing: củng cố
- breakpoints: điểm ngắt/điểm dừng

> "**Periodic injection** of behavioral reminders directly **combats instruction drift** by **re-establishing** constraints at regular intervals as conversation history **accumulates**."
- periodic injection: sự chèn định kỳ
- combats: chống lại
- instruction drift: sự trôi dạt khỏi chỉ dẫn ban đầu
- re-establishing: tái thiết lập
- accumulates: tích lũy dần

**Q70**
> "After 12 turns, the assistant starts ignoring **proficiency levels**."
- proficiency levels: các mức độ thành thạo

> "Replace verbose rules with few-shot examples demonstrating **proficiency-level adaptation**."
- adaptation: sự thích ứng, điều chỉnh cho phù hợp

> "**abstract** rules require the model to reason about them on every turn... gives the model clear **behavioral patterns** to match."
- abstract: trừu tượng
- behavioral patterns: các mẫu hành vi

**Q71**
> "Your assistant must maintain an enthusiastic tone, explain its reasoning, and ask **clarifying** questions."
- clarifying (questions): (câu hỏi) làm rõ

> "The system prompt is specifically designed for **persistent behavioral constraints** and guidelines that apply **throughout** the entire conversation."
- persistent behavioral constraints: các ràng buộc hành vi mang tính lâu dài
- throughout: xuyên suốt

**Q72**
> "Users report **repetitive** response **openings** like "Certainly!" and "I'd be happy to help!""
- repetitive: lặp đi lặp lại
- openings: phần mở đầu

> "**Append** a **partial** assistant message with a direct response opening."
- append: nối thêm vào (cuối)
- partial: một phần, chưa hoàn chỉnh

> "**Prefilling** the assistant's response with the beginning of a direct answer prevents greeting patterns at the **generation level**."
- prefilling: điền sẵn trước một phần nội dung
- generation level: cấp độ sinh văn bản (của mô hình)

**Q73**
> "A webhook **notifies** your system that a user's package has **shipped** while the user is **actively** chatting."
- notifies: thông báo cho
- shipped: đã được vận chuyển
- actively: một cách chủ động, đang diễn ra

> "**Append** the status update as a **prefix** to the next user message."
- prefix: tiền tố, phần thêm vào đầu

> "Prefixing the status update... **injects real-time** context at a natural conversation boundary without **disrupting** the flow."
- injects: chèn vào, tiêm vào
- real-time: thời gian thực
- disrupting: làm gián đoạn

**Q74**
> "Users frequently send requests like "Book a **venue** for the party." The assistant asks 4+ clarifying questions, causing 35% **abandonment**."
- venue: địa điểm (tổ chức sự kiện)
- abandonment: sự từ bỏ giữa chừng

> "State **assumptions** explicitly and proceed while **inviting corrections**."
- assumptions: các giả định
- inviting corrections: mời/khuyến khích người dùng sửa lại (nếu sai)

**Q75**
> "**Accumulated** assistant responses **dilute** system prompt influence."
- accumulated: được tích lũy
- dilute: pha loãng

> "The model **increasingly** pattern-matches to its own prior outputs rather than the system prompt, **compounding** drift."
- increasingly: ngày càng nhiều
- compounding: làm chồng chất, tích lũy thêm

**Q76**
> "Users ask vague requests like "Can you help with the report?" ...causing 40% abandonment."
> "Make **reasonable** assumptions, state them explicitly, and **offer to adjust**."
- reasonable: hợp lý
- offer to adjust: đề nghị sẵn sàng điều chỉnh (nếu cần)

> "Proceeding with reasonable stated assumptions eliminates the **back-and-forth** entirely while keeping the user informed and **in control**."
- back-and-forth: sự qua lại nhiều lượt (hỏi đáp)
- in control: nắm quyền kiểm soát

---

## Tổng kết ôn tập

- Tổng cộng ~230 mục từ/cụm từ, đi kèm câu gốc trong 76 câu luyện tập.
- Ưu tiên ôn theo cụm câu trong 2 chủ đề yếu nhất theo [`03-chan-doan-thi-thu.md`](./03-chan-doan-thi-thu.md): **CI/CD (47%)** và **Multi-agent Research (67%)** — vừa ôn từ vựng vừa củng cố lại pattern làm bài.
- Gợi ý lịch ôn: xem lại toàn bộ 1 lượt hôm nay → active recall (che nghĩa, chỉ đọc câu) sau 1 ngày → sau 3 ngày → sau 1 tuần.
