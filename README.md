# Đưa AI Agent lên Cloud: Kiến trúc, Workflow phức tạp & Kiểm định đầu ra

Nov 14, 2026 · @Dang Tuan Anh NAVER VIETNAM

## Tóm tắt

Đưa agent lên cloud thành công phụ thuộc vào ba thứ: **sandbox cô lập theo session, state bền vững, và bộ eval nhiều lớp** — không phải vào việc chọn framework.

- Các nền tảng lớn (AWS AgentCore, Microsoft Foundry, Google Gemini Enterprise Agent Platform, Claude Managed Agents) đã hội tụ về cùng mô hình: mỗi session một microVM/sandbox, session + memory managed, tracing OpenTelemetry.
- Chọn nền tảng theo cloud đang dùng và yêu cầu dữ liệu; tự host SDK trên K8s khi cần toàn quyền kiểm soát.
- Workflow phức tạp nên ưu tiên pipeline có cấu trúc, chỉ dùng orchestrator–workers khi số việc con không biết trước; bọc bằng durable execution để sống sót qua lỗi.
- Kiểm tra đầu ra: chấm outcome (trạng thái thật) thay vì lời agent tự báo; kết hợp grader code, LLM judge đã hiệu chuẩn và người; đo pass^k cho agent tự động.
- Bắt đầu với 20–50 task eval từ lỗi thật; regression eval chạy trong CI mỗi lần đổi prompt/model.
- Chi phí bị chi phối bởi token, nên đặt giới hạn số vòng và ngân sách token ngay từ ngày đầu.

## 1. Bức tranh tổng quan

Đưa agent lên cloud nghĩa là biến một vòng lặp "LLM suy luận → gọi tool → quan sát → lặp lại" thành một dịch vụ chạy lâu, cô lập, có trạng thái và đo lường được. Khác biệt cốt lõi so với chạy local nằm ở 4 điểm:

| Khía cạnh | Chạy local / notebook | Chạy trên cloud (production) |
| --- | --- | --- |
| Thời gian chạy | Vài phút, người dùng ngồi chờ | Vài phút đến nhiều giờ, chạy nền, phải sống sót qua lỗi mạng, restart |
| Môi trường thực thi | Máy dev, quyền rộng | Sandbox/microVM tách biệt theo phiên, quyền tối thiểu |
| Trạng thái | Trong RAM | Lưu bền: session, checkpoint, memory dài hạn |
| Kiểm soát chất lượng | Mắt người xem output | Trace, eval tự động, guardrail, cảnh báo |

Có ba mô hình triển khai chính:

1. **Managed agent runtime** — nền tảng lo runtime, sandbox, memory, identity, observability (AWS Bedrock AgentCore, Google Vertex AI Agent Engine, Azure AI Foundry Agent Service, Claude Managed Agents). Nhanh lên production, đánh đổi bằng lock-in.
2. **Framework + hạ tầng serverless/container** — tự viết agent bằng SDK (Claude Agent SDK, LangGraph, OpenAI Agents SDK, Google ADK) rồi deploy lên Cloud Run, Lambda, ECS/K8s. Linh hoạt, tốn công vận hành.
3. **Durable workflow engine + LLM step** — dùng Temporal, Step Functions, Inngest… làm xương sống, mỗi bước LLM/tool là một activity có retry. Phù hợp quy trình dài, cần đảm bảo không mất việc giữa chừng.

## 2. Kiến trúc tham chiếu

Kiến trúc production có ba tầng: tầng điều phối (gateway + workflow engine), tầng thực thi (sandbox riêng mỗi session) và tầng dịch vụ dùng chung (model, state, egress, quan sát).

&#91;embedded content: kiến trúc tham chiếu · 3 tầng\]

Agent không bao giờ giữ credential hay gọi thẳng ra ngoài: mọi request đi qua egress proxy, mọi trạng thái được ghi ra store bền vững để sandbox có thể bị hủy và dựng lại bất cứ lúc nào.

## 3. So sánh nền tảng

Cả bốn hyperscaler/nhà cung cấp model lớn đều đã hội tụ về cùng một thiết kế: **mỗi session một sandbox cô lập + session/memory được quản lý + tracing OpenTelemetry**. Khác biệt nằm ở hệ sinh thái định danh, giới hạn thời gian chạy và mức độ cho phép mang framework riêng.

| Nền tảng | Cô lập & thời gian chạy | State / memory | Framework | Điểm mạnh | Lưu ý |
| --- | --- | --- | --- | --- | --- |
| [AWS Bedrock AgentCore Runtime](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-sessions.html) | microVM riêng mỗi session; tối đa 8 giờ (microVM) hoặc 14 ngày (Instances); idle timeout mặc định 15 phút | Session storage sống qua stop/resume; AgentCore Memory cho dữ liệu dài hạn | Framework-agnostic; giao thức HTTP, MCP, A2A, AG-UI | IAM scope theo từng session, hệ sinh thái AWS | Bạn tự quản lý mapping user ↔ session; cần retry khi gặp 409 |
| [Microsoft Foundry Agent Service – hosted agents](https://devblogs.microsoft.com/foundry/introducing-the-new-hosted-agents-in-foundry-agent-service-secure-scalable-compute-built-for-agents) | Sandbox cô lập bằng hypervisor mỗi session; scale-to-zero | File, disk, session identity giữ qua scale-to-zero; long-term memory (preview) | BYO: LangGraph, Agent Framework, Claude Agent SDK, OpenAI Agents SDK… qua Dockerfile | Entra Agent ID cho từng agent, OBO OAuth, continuous eval + red teaming | Pull image qua private endpoint chưa hỗ trợ (theo bài gốc) |
| [Gemini Enterprise Agent Platform (trước là Vertex AI Agent Engine)](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/overview) | Agent Runtime managed; Code Execution sandbox, Computer Use | Sessions + Memory Bank | ADK, A2A, LangChain, LangGraph, AG2, LlamaIndex, custom | Evaluation Service tích hợp sẵn | Tên sản phẩm đã đổi, tài liệu cũ còn dùng tên Vertex |
| [Claude Managed Agents](https://the-decoder.com/anthropic-launches-managed-infrastructure-for-autonomous-ai-agents/) | Anthropic host vòng lặp agent; sandbox cloud hoặc self-hosted; session chạy nhiều giờ, kết quả giữ khi rớt kết nối | Do nền tảng quản lý; memory/multi-agent ở research preview | Chỉ model Claude | Không cần tự viết harness; tool bash, file, web search, MCP sẵn | Public beta từ 04/2026; giá token + 0,08 USD/session-giờ |
| [Tự host Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/hosting) (Docker/K8s/Modal) | Mỗi session = 1 subprocess; bạn chọn gVisor, Firecracker… | `SessionStore` mirror transcript ra DB/object store | Claude (qua API, Bedrock, Google Cloud) | Toàn quyền về mạng, dữ liệu, chi phí | Không có timeout tổng, phải đặt `maxTurns`; tự lo isolation multi-tenant |

**Cách chọn nhanh:** đã gắn chặt với một cloud → dùng runtime của cloud đó vì định danh và mạng riêng đã có sẵn. Cần multi-model hoặc dữ liệu phải nằm trong hạ tầng nội bộ → tự host SDK trên K8s. Muốn ra sản phẩm nhanh nhất với Claude → Managed Agents.

## 4. Thiết kế workflow phức tạp

Nguyên tắc số 1: **dùng workflow có cấu trúc khi các bước đã biết trước, chỉ để agent tự quyết ở những chỗ thật sự cần linh hoạt** ([Anthropic – Building effective agents](https://www.anthropic.com/research/building-effective-agents)). Càng nhiều quyết định giao cho LLM, càng khó kiểm thử và càng tốn token.

### Các pattern nên biết

| Pattern | Khi nào dùng | Ví dụ |
| --- | --- | --- |
| Prompt chaining (pipeline) | Bước cố định, có cổng kiểm tra giữa các bước | Trích xuất → chuẩn hóa → sinh báo cáo |
| Routing | Đầu vào nhiều loại, mỗi loại xử lý khác | Phân loại ticket rồi chuyển sub-agent phù hợp |
| Parallelization (fan-out/fan-in) | Việc độc lập hoặc cần biểu quyết | Review code theo 3 góc độ song song |
| Orchestrator–workers | Không biết trước có bao nhiêu việc con | Research sâu, sửa nhiều file |
| Evaluator–optimizer | Có tiêu chí rõ để lặp cải tiến | Viết → chấm → sửa bản dịch |
| Human-in-the-loop | Hành động không đảo ngược được | Duyệt trước khi gửi email, deploy, xóa dữ liệu |

### Làm cho workflow dài chạy được trên cloud

- **Durable execution:** bọc mỗi lần gọi LLM/tool thành một activity có retry và lưu kết quả (Temporal, AWS Step Functions, Inngest). Khi worker chết giữa chừng, workflow tiếp tục từ bước cuối đã xong thay vì chạy lại từ đầu ([Temporal + OpenAI Agents SDK](https://temporal.io/blog/announcing-openai-agents-sdk-integration)).
- **Checkpoint state:** lưu transcript và artifact ra store bền vững (ví dụ `SessionStore` của Claude Agent SDK, session storage của AgentCore). Container là thứ có thể mất bất cứ lúc nào.
- **Chọn vòng đời session đúng:** ephemeral (1 container/1 task), long-running (container sống lâu, nhiều session), hybrid (ngủ khi idle, nạp lại từ store khi user quay lại) ([Claude Agent SDK hosting](https://code.claude.com/docs/en/agent-sdk/hosting)).
- **Giới hạn cứng:** đặt `max_turns`, ngân sách token, deadline theo wall-clock cho từng sub-agent; chia fan-out lớn thành batch nhỏ để tránh rate limit.
- **Idempotency:** mọi tool có side effect phải nhận idempotency key, vì retry là chuyện chắc chắn xảy ra.
- **Context engineering:** sub-agent chỉ trả về bản tóm tắt, không đổ toàn bộ lịch sử vào orchestrator; ghi kết quả trung gian ra file thay vì giữ trong context.
- **Human-in-the-loop bất đồng bộ:** workflow dừng ở trạng thái "chờ duyệt" (signal/callback), không giữ container chạy trong lúc chờ.

## 5. Kiểm tra kết quả đầu ra

Không có lớp kiểm tra nào bắt được mọi lỗi, nên cần xếp chồng nhiều lớp theo mô hình "Swiss cheese": eval tự động trước khi release, guardrail lúc chạy, monitoring production, A/B test, feedback người dùng và đọc transcript ([Anthropic – Demystifying evals for AI agents](https://anthropic.com/engineering/demystifying-evals-for-ai-agents)).

### Khái niệm cốt lõi

- **Task** — một bài test với input và tiêu chí thành công rõ ràng. **Trial** — một lần chạy; cần nhiều trial vì output không tất định.
- **Transcript** — toàn bộ tool call, suy luận, kết quả. **Outcome** — trạng thái cuối của môi trường: agent nói "đã xong" không tính, trạng thái DB/file mới tính.

### Ba loại grader

| Grader | Dùng cho | Ưu điểm | Nhược điểm |
| --- | --- | --- | --- |
| Code-based (deterministic) | Unit test, schema JSON, regex, kiểm tra trạng thái DB, tool có được gọi không | Nhanh, rẻ, tái lập được | Cứng nhắc, thiếu sắc thái |
| Model-based (LLM-as-judge) | Chất lượng văn bản, tính đúng ngữ cảnh, tuân thủ rubric | Linh hoạt với task mở | Không tất định, phải hiệu chuẩn với người |
| Human | Chuẩn vàng, hiệu chuẩn judge | Chính xác nhất | Đắt, chậm |

### Đo độ tin cậy

Với agent chạy tự động, chỉ số quan trọng thường là **pass^k** (mọi lần đều đúng) chứ không phải pass@k (ít nhất một lần đúng). Tỷ lệ 75% mỗi lần chỉ cho khoảng 42% khi yêu cầu đúng cả 3 lần.

```latex
\text{pass}^k = p^k \qquad 0.75^3 \approx 0.42
```

### Quy trình khuyến nghị

1. Bắt đầu với 20–50 task lấy từ lỗi thật (bug tracker, support queue, kiểm tra thủ công); mỗi task có lời giải tham chiếu chứng minh nó giải được.
2. Tách **capability eval** (pass rate thấp, để cải thiện) và **regression eval** (giữ gần 100%, chạy trong CI mỗi lần đổi prompt, model, tool).
3. Chấm **kết quả**, không chấm đúng từng bước; cho điểm từng phần với task nhiều phần.
4. Hiệu chuẩn LLM judge với chuyên gia, cho judge lựa chọn "Không xác định" để giảm chấm bừa.
5. Cô lập mỗi trial (sandbox sạch) để nhiễu hạ tầng không làm sai kết quả.
6. Đọc transcript định kỳ — đây là cách duy nhất biết eval có đo đúng thứ cần đo.

### Kiểm tra lúc chạy (runtime)

- **Guardrail đầu vào:** lọc prompt injection từ nội dung web/email/file mà agent đọc.
- **Guardrail đầu ra:** validate schema, chặn PII, kiểm tra chính sách trước khi trả kết quả.
- **Verifier step:** một agent/model khác kiểm tra kết quả trước hành động quan trọng; tốt hơn nữa là kiểm tra deterministic (chạy test, so sánh số liệu nguồn).
- **Online eval:** lấy mẫu trace production, chấm tự động hằng ngày để phát hiện drift; các nền tảng như Foundry và Gemini Enterprise Agent Platform đã tích hợp continuous evaluation.

## 6. Vận hành: chi phí, quan sát, rủi ro

Chi phí token thường lớn hơn chi phí container ít nhất một bậc: một container tối thiểu khoảng 0,05 USD/giờ, trong khi một session dài có thể tiêu vài USD tiền token ([Claude Agent SDK hosting](https://code.claude.com/docs/en/agent-sdk/hosting)). Vì vậy tối ưu nằm ở số vòng lặp và độ dài context, không phải ở hạ tầng.

- **Sizing:** điểm khởi đầu 1 GiB RAM, 5 GiB disk, 1 CPU mỗi agent; số agent/host = (RAM host − overhead) / RAM đỉnh mỗi session.
- **Routing:** với session dài, ghim session vào một container bằng consistent hashing theo `sessionId`.
- **Observability:** xuất trace/metric/log qua OpenTelemetry; theo dõi theo session: số turn, token, thời gian từng tool, tỷ lệ lỗi, `mirror_error` khi lưu state thất bại.

| Rủi ro | Giảm thiểu |
| --- | --- |
| Prompt injection từ dữ liệu ngoài | Sandbox, egress allowlist, tách quyền đọc/ghi, duyệt người cho hành động nhạy cảm |
| Lộ credential | Không đặt key trong container; proxy inject credential khi request đi ra; định danh riêng cho agent (Entra Agent ID, IAM theo session) |
| Rò rỉ dữ liệu giữa tenant | Thư mục, config, egress riêng mỗi tenant; tắt auto-memory dùng chung |
| Agent chạy vô hạn, đốt token | `max_turns`, ngân sách token, stall watchdog, cảnh báo chi phí theo session |
| Mất tiến độ khi container chết | Durable execution + checkpoint ra store bền vững |
| Chất lượng giảm khi đổi model/prompt | Regression eval trong CI, rollout theo trọng số (canary) |

## 7. Lộ trình đề xuất

1. **Giai đoạn 1 — Prototype có đo lường (2–4 tuần):** chọn 1 use case hẹp; viết agent bằng SDK; dựng 20–50 task eval từ case thật; bật tracing ngay từ đầu.
   - Cổng qua: pass rate trên bộ eval đạt ngưỡng team thống nhất; chi phí/task được đo.
2. **Giai đoạn 2 — Lên cloud nội bộ (4–6 tuần):** deploy lên managed runtime hoặc K8s; sandbox mỗi session; lưu state bền vững; egress proxy; human-in-the-loop cho hành động có side effect.
   - Cổng qua: regression eval chạy trong CI; dashboard chi phí và lỗi theo session.
3. **Giai đoạn 3 — Production & mở rộng:** canary rollout; online eval trên mẫu trace; mở dần sang workflow nhiều agent (orchestrator–workers) khi pipeline đơn đã ổn.

### Checklist trước khi go-live

- [ ] Mỗi session chạy trong sandbox cô lập, quyền tối thiểu
- [ ] Credential không nằm trong môi trường agent; egress có allowlist
- [ ] State/transcript được lưu ra store bền vững, resume được
- [ ] Có `max_turns`, ngân sách token, timeout cho từng sub-agent
- [ ] Tool có side effect dùng idempotency key
- [ ] Hành động không đảo ngược được có bước duyệt
- [ ] Bộ regression eval chạy trong CI, LLM judge đã hiệu chuẩn với người
- [ ] Tracing OpenTelemetry + cảnh báo chi phí, lỗi, drift
- [ ] Có lịch đọc transcript định kỳ và người chịu trách nhiệm bộ eval

## Nguồn tham khảo

- [Amazon Bedrock AgentCore – Use isolated sessions for agents](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-sessions.html)
- [Microsoft Foundry – Introducing the new hosted agents in Foundry Agent Service](https://devblogs.microsoft.com/foundry/introducing-the-new-hosted-agents-in-foundry-agent-service-secure-scalable-compute-built-for-agents)
- [Google Cloud – Memory Bank overview (Gemini Enterprise Agent Platform)](https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/overview)
- [The Decoder – Anthropic launches managed infrastructure for autonomous AI agents](https://the-decoder.com/anthropic-launches-managed-infrastructure-for-autonomous-ai-agents/)
- [Claude Agent SDK – Hosting the Agent SDK](https://code.claude.com/docs/en/agent-sdk/hosting)
- [Anthropic – Demystifying evals for AI agents](https://anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [Anthropic – Building effective agents](https://www.anthropic.com/research/building-effective-agents)
- [Temporal – Production-ready agents with the OpenAI Agents SDK + Temporal](https://temporal.io/blog/announcing-openai-agents-sdk-integration)
Note
Highlights (0)
✕
🔊
Write a note for this site…

