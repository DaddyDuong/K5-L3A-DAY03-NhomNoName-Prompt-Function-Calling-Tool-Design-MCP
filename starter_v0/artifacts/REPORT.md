# Day 03 Lab v2 Report — Research Agent

> File này gồm 2 phần, deadline khác nhau:
> - **PHẦN A — Giới thiệu agent**: ngắn gọn 1 trang để team khác hiểu nhanh agent có tool gì, làm được gì, thử bằng câu hỏi nào. **Xong trước 16:30** để làm tài liệu phụ trợ khi demo. Có thể làm thành poster HTML/SVG (`artifacts/poster.html` / `poster.svg`) để show cho team cùng zone.
> - **PHẦN B — Chi tiết / Bằng chứng**: bảng đầy đủ (v0–v3, failure, eval, chat) dựa trên log thật. **Có thể hoàn thiện sau buổi debate để nộp bài.**

## Team

- Team:
- Members:
- Provider/model:

---

# PHẦN A — Giới thiệu agent

## A1. Agent này làm được gì

> 1–2 câu mô tả agent dùng để làm gì (cho team khác hiểu nhanh).

Ví dụ: "Research agent: tìm tin theo từ khóa / theo tài khoản, đọc URL, tổng hợp thành digest, và gửi lên Telegram khi được xác nhận."

**Link dùng thử (deploy):**

> Dán link public để team khác mở thử ngay. Cách deploy nhanh bằng Cloudflare Tunnel xem README. Nếu deploy Vercel/Streamlit Cloud thì dán link đó.
>
> URL: 

## A2. Tool agent có

> Liệt kê các tool agent đang dùng (gồm tool mới nhóm tự thêm). Mỗi tool 1 dòng: tên + làm được gì.

| Tên tool | Làm được gì | Tool mới nhóm thêm? |
|---|---|---|
| clarify | hỏi lại người dùng khi thiếu thông tin | không |
| scope_check | phân loại yêu cầu (offline, theo từ khóa): in_scope / out_of_scope (toán, code, dịch, sáng tác) / mixed / meta / unclear, kèm hành động gợi ý; chỉ gọi khi không chắc yêu cầu có thuộc phạm vi | có |
|  |  |  |

## A3. Câu hỏi mẫu để thử

> 3–5 câu hỏi/yêu cầu mẫu để team khác tự thử agent ngay.

1.
2.
3.

---

# PHẦN B — Chi tiết / Bằng chứng

## B1. Version Evidence

Fill from `artifacts/version_log.csv` and `runs/*.json`.

> Metric = `case_accuracy` trên suite `base` (20 case), model gemini-3.5-flash. v0 và v1 lấy từ run thật. `artifacts/version_log.csv` hiện vẫn chưa có dòng nào (chỉ có header), nên số liệu dưới đây lấy trực tiếp từ `summary` trong `runs/*.json`. Dòng v2–v3 là kế hoạch sửa rút ra từ B2 (phần v1); cột Metric After và Run File điền sau khi chạy `run_eval.py`.

| Version | Changed Artifact | Hypothesis | Metric Before | Metric After | Run File |
|---|---|---|---:|---:|---|
| v0 | baseline (`system_prompt.md` + `tools.yaml` starter, hash `v0+peb1c8179815b+t5b22a2476d41`) | Đo mốc ban đầu với gemini-3.5-flash. Prompt v0 cố tình xấu (đoán thay vì hỏi, tự gửi, chỉ chọn 1 tool), mô tả tool mơ hồ | — | 0.55 (11/20; routing 0.65, argument 0.55, multiturn 0.83) | `runs/v0_B_base_gemini_20261110T111122645086.json` |
| v1 | `system_prompt.md` + `tools.yaml` (hash `v1+pbbc09845307f+td67078e54c86`) | **Prompt:** thêm mục "Scope (highest priority)" ở đầu: câu ngoài phạm vi (toán, code, dịch, sáng tác) → từ chối bằng text, không gọi tool; câu hỏi về bản thân agent → trả lời thẳng; `send` không phải cách trả lời người dùng. **Tools:** mô tả `lookup.query` = chỉ từ khóa cốt lõi, không thêm từ chỉ thời gian; `send` = chỉ đăng bản tin lên Telegram; `policy` = chỉ cho câu hỏi về quy định nội bộ; thêm tool mới `scope_check` (chỉ gọi khi không chắc phạm vi). Kỳ vọng sửa R03, R08, R09, R14, M02 | 0.55 | **0.75** (15/20; routing 0.80, argument 0.75, multiturn 1.00) | `runs/v1_B_base_gemini_20261110T113910565355.json` |
| v2 | `system_prompt.md` | Xóa các câu v0 còn sót lại ("make a sensible guess", "pick a well-known account like Sam Altman", "assume a likely URL", "just go ahead and do it", "Pick one tool"). Thay bằng luật: thiếu handle/URL → `clarify(response_type="text")`; trước `send` → `clarify(response_type="yes_no")`, chỉ đặt `confirmed=true` sau khi người dùng nói "có"; yêu cầu nhiều nguồn → gọi song song nhiều tool. Kèm `tools.yaml`: `query` giữ nguyên cách viết của người dùng, không mở rộng viết tắt ("AI" ≠ "Artificial Intelligence"). Kỳ vọng sửa R03, R10, R11, R12, R13 | 0.75 | _(chưa chạy)_ | _(chưa chạy)_ |
| v3 | `system_prompt.md` (multi-turn) + `data/eval_group.json` | Củng cố luật multi-turn "lượt mới nhất ghi đè chủ đề, giữ các tham số không bị sửa". Chạy thêm 10 case group (B3) để kiểm tra fix có tổng quát hay chỉ khớp eval base | _(metric v2)_ | _(chưa chạy)_ | _(chưa chạy)_ |

**So sánh v0 → v1 theo từng case** (từ `results[*].result.passed`):

| Thay đổi | Case |
|---|---|
| Fail → Pass (4) | R08_out_of_scope, R09_no_tool_capability, R14_out_of_scope_coding, M02_carryover_timeframe |
| Vẫn fail (5) | R03_web_news_routing (lỗi khác), R10_missing_handle, R11_missing_url, R12_confirm_before_send, R13_parallel_web_and_tweets |
| Pass → Fail (regression) | không có |

## B2. Failure Analysis

Use actual failures from `results[*].result.failures`.

### B2.1. v0 (baseline)

Nguồn: `runs/v0_B_base_gemini_20261110T111122645086.json` (v0, suite `base`, gemini-3.5-flash). Kết quả: 11/20 pass — case_accuracy 0.55, tool_routing 0.65, argument 0.55, multiturn 0.83. 9 case fail như bảng dưới.

| Case ID | Failure Type | Actual Tool Calls | What Failed | Fix |
|---|---|---|---|---|
| R03_web_news_routing | wrong_arg_value | `lookup(query="AI news today", topic="news", timeframe="day", max_results=5)` | Đúng tool và topic/timeframe, nhưng `query` bị nhồi thêm "news today" (đã có trong topic/timeframe) → expected `"AI"` | `tools.yaml`: mô tả `query` = chỉ chủ đề cốt lõi, KHÔNG thêm từ chỉ thời gian/"tin tức" vì đã có `topic`/`timeframe` |
| R08_out_of_scope | out_of_scope | `send(text="Nguyên hàm của x^2 là x^3/3 + C ...")` | Câu toán ngoài phạm vi, đáng ra không gọi tool + từ chối; model tự giải rồi gọi `send` để "trả lời" | ✅ **Đã sửa ở v1.** `system_prompt.md`: thêm mục "Scope (highest priority — overrides the rules below)" ở đầu prompt, liệt kê phạm vi research (web news, tweet, đọc URL, arXiv, policy, đăng digest Telegram); câu ngoài phạm vi (toán, code, dịch, sáng tác) → không gọi tool nào, kể cả `send`/`scope_check`, trả lời bằng text, từ chối lịch sự, không giải bài. `tools.yaml`: `send` = "Đăng một BẢN TIN research lên Telegram… KHÔNG dùng send để trả lời người dùng". Kết quả v1: không gọi tool, trả lời "Xin lỗi bạn, yêu cầu giải bài toán/tích phân nằm ngoài phạm vi…" |
| R09_no_tool_capability | unnecessary_tool | `policy(query="quy định và khả năng trợ lý AI")` | Câu hỏi meta "Bạn là gì" phải trả lời thẳng; model lại tra tài liệu nội bộ | ✅ **Đã sửa ở v1.** `system_prompt.md` (mục Scope): câu hỏi về bản thân ("bạn là gì", "làm được gì") → trả lời thẳng bằng text, không gọi tool. `tools.yaml`: `policy` = "Tìm trong tài liệu QUY ĐỊNH NỘI BỘ… KHÔNG dùng cho câu hỏi 'bạn là gì / làm được gì'"; `scope_check` cũng ghi rõ không gọi cho câu meta. Kết quả v1: không gọi tool, tự giới thiệu là trợ lý nghiên cứu và liệt kê việc làm được |
| R10_missing_handle | missing_info | `timeline(screenname="sama", limit=5)` | Thiếu tài khoản nhưng model đoán bừa `sama` — do v0 prompt bảo "pick a well-known account like Sam Altman" | `system_prompt.md`: bỏ chỉ dẫn đoán; thiếu handle → `clarify(response_type="text")` |
| R11_missing_url | missing_info | `fetch(url="https://en.wikipedia.org/wiki/Artificial_intelligence")` | Nói "bài này" nhưng không có link; model bịa URL — do v0 prompt bảo "assume a likely URL" | `system_prompt.md`: không có URL → `clarify` xin link, không tự tạo URL. `tools.yaml`: `fetch` chỉ dùng URL người dùng đưa |
| R12_confirm_before_send | wrong_boundary | `send(confirmed=true, text="Bản tin Công nghệ & AI hôm nay: ...")` | Hành động ghi ra ngoài nhưng tự gửi, tự đặt `confirmed=true` và bịa nội dung bản tin — do v0 prompt bảo "just go ahead and do it" | `system_prompt.md`: trước khi `send` phải `clarify(response_type="yes_no")`. `tools.yaml`: `confirmed=true` chỉ khi người dùng đã trả lời "có" |
| R13_parallel_web_and_tweets | wrong_tool (missing call) + wrong_arg_value | `lookup(query="tin tức AI mới nhất", topic="news", timeframe="day")` | Yêu cầu 2 nguồn nhưng chỉ gọi `lookup`, thiếu `social_search(query="AI")`; `query` cũng bị thêm chữ thừa — do v0 prompt bảo "Pick one tool" | `system_prompt.md`: bỏ "pick one tool"; request nhiều nguồn → gọi song song nhiều tool. Kèm fix `query` như R03 |
| R14_out_of_scope_coding | out_of_scope | `send(text="Dưới đây là hàm Python tính Fibonacci ...")` | Câu coding ngoài phạm vi; model viết code rồi gọi `send` | ✅ **Đã sửa ở v1.** Cùng fix với R08: mục Scope trong `system_prompt.md` (viết code là ngoài phạm vi → từ chối bằng text, không gọi tool) + mô tả `send` mới trong `tools.yaml` (không dùng `send` để trả lời/gửi code). Kết quả v1: không gọi tool, trả lời "yêu cầu viết mã nguồn (code) nằm ngoài phạm vi hỗ trợ của mình" |
| M02_carryover_timeframe | wrong_arg_value | `lookup(query="robotics AI news", topic="news", timeframe="day")` | Carry đúng `topic`/`timeframe` qua 3 lượt, nhưng `query` trộn chủ đề cũ ("AI") + "news" dù lượt cuối nói "chỉ tìm robotics" | ✅ **Đã sửa ở v1**, chỉ bằng `tools.yaml`: mô tả `lookup.query` = "Chỉ từ khóa chủ đề/thực thể cốt lõi, ngắn gọn. KHÔNG chép nguyên câu hỏi. KHÔNG thêm từ chỉ thời gian vì đã có timeframe và topic". Chưa cần thêm luật multi-turn vào prompt như dự kiến ban đầu. Kết quả v1: `lookup(query="robotics", topic="news", timeframe="day")` |

Nhận xét: phần lớn lỗi (R10, R11, R12, R13) do chính `system_prompt.md` v0 bảo model đoán, tự gửi và chỉ chọn một tool. Nhóm lỗi thứ hai là mô tả tool quá mơ hồ ở `tools.yaml` (R03, R08, R09, R14, M02): `send`/`policy` không có ranh giới dùng, còn `query` không có quy ước. Routing cơ bản (R01, R02, R04) và đổi tên → handle (R01, R05, M01, M03, M05) thì đã đúng từ v0.

### B2.2. v1

Nguồn: `runs/v1_B_base_gemini_20261110T113910565355.json` (v1, suite `base`, gemini-3.5-flash, hash `v1+pbbc09845307f+td67078e54c86`). Kết quả: 15/20 pass — case_accuracy 0.75, tool_routing 0.80, argument 0.75, multiturn 1.00. `failure_counts`: wrong_tool 2, missing_info 2, wrong_boundary 1; `observed_mismatch_counts`: missing_tool_call 4, wrong_arg_value 1. 5 case fail như bảng dưới.

| Case ID | Failure Type | Actual Tool Calls | What Failed | Fix |
|---|---|---|---|---|
| R03_web_news_routing | wrong_arg_value | `lookup(query="Artificial Intelligence", topic="news", timeframe="day", max_results=5)` | Đã hết nhồi "news today" (fix v1 có tác dụng), nhưng model tự mở rộng viết tắt "AI" → "Artificial Intelligence". Grader so khớp chính xác nên báo `query: expected 'AI', got 'Artificial Intelligence'` | `tools.yaml`: mô tả `query` thêm "giữ nguyên từ khóa như người dùng viết, KHÔNG mở rộng/dịch viết tắt". Lỗi này đúng về ngữ nghĩa, nên cần review thủ công (xem B6) |
| R10_missing_handle | missing_info | `timeline(screenname="sama", limit=5)` | Vẫn đoán `sama` thay vì `clarify`. Câu v0 "pick a well-known account like Sam Altman" vẫn còn trong prompt, v1 chỉ thêm mục Scope ở đầu chứ không xóa câu này | `system_prompt.md` (v2): xóa câu đoán handle, thêm luật thiếu tài khoản → `clarify(response_type="text")` |
| R11_missing_url | missing_info | `fetch(url="https://techcrunch.com")` | Vẫn bịa URL (đổi từ Wikipedia sang TechCrunch). Nguyên nhân: câu v0 "assume a likely URL and read it" chưa bị xóa | `system_prompt.md` (v2): xóa câu "assume a likely URL"; không có link → `clarify` xin link. `tools.yaml`: `fetch.url` chỉ nhận URL người dùng đưa |
| R12_confirm_before_send | wrong_boundary | `send(text="Bản tin Nghiên cứu & Công nghệ AI: ...", confirmed=true)` | Vẫn tự gửi, tự đặt `confirmed=true` và bịa nội dung bản tin. Mô tả `send` mới nói "là hành động ghi ra ngoài" nhưng prompt vẫn có câu "just go ahead and do it", và mô tả `confirmed` chưa đổi | `system_prompt.md` (v2): xóa "just go ahead"; trước `send` bắt buộc `clarify(response_type="yes_no")`. `tools.yaml`: `confirmed=true` chỉ khi người dùng đã xác nhận "có" ở lượt trước |
| R13_parallel_web_and_tweets | wrong_tool (missing call) | `lookup(query="AI", topic="news", timeframe="day")` | `query` đã đúng "AI" (fix v1 có tác dụng), nhưng vẫn chỉ gọi 1 tool, thiếu `social_search(query="AI")` — câu "Pick one tool" của v0 vẫn còn | `system_prompt.md` (v2): xóa "Pick one tool"; yêu cầu nhiều nguồn (web + tweet) → gọi song song nhiều tool |

Nhận xét v1:

- **Fix của v1 có tác dụng đúng chỗ đã sửa.** Mục Scope + giới hạn `send`/`policy` sửa được cả 3 case phạm vi (R08, R14 từ chối bằng text, R09 tự giới thiệu, đều không gọi tool). Quy ước `query` sửa được M02 (`query="robotics"`) và phần args của R13 (`query="AI"`). Multiturn lên 1.00. Không có regression nào, và `scope_check` không bị gọi thừa ở case nào.
- **4/5 lỗi còn lại cùng một nguyên nhân:** v1 chỉ *thêm* mục Scope vào đầu prompt mà chưa *xóa* các chỉ dẫn xấu của v0 (đoán handle, đoán URL, tự gửi, chọn một tool). Model vẫn làm theo các câu đó, nên R10, R11, R12, R13 fail y như v0. Đây là việc cần làm ở v2.
- **R03 là lỗi khác loại:** hành vi đã đúng về ngữ nghĩa ("Artificial Intelligence" = "AI") nhưng grader so khớp chuỗi chính xác. Cần sửa mô tả `query` để giữ nguyên từ khóa, và ghi nhận đây là case nên được review thủ công.

## B3. Team Eval Cases

List the 10 cases added to `data/eval_group.json` (5 single turn + 5 multi turn).

> Lưu ý: tại thời điểm viết, `data/eval_group.json` vẫn đang rỗng (`"cases": []`) và chưa có run nào với suite `group`. 10 case dưới đây được thiết kế từ các lỗi ở B2 để kiểm tra fix có tổng quát hay không (không chỉ khớp đúng câu trong `eval_base`). Cột **Result** cần điền sau khi thêm case vào file và chạy `run_eval.py` với suite group.

| Case ID | What It Tests | Expected Tool/Behavior | Result |
|---|---|---|---|
| G01_query_no_time_words | (single) "Tin về Nvidia trong tháng này" — `query` chỉ là chủ đề, thời gian đưa vào `timeframe` (từ R03) | `lookup(query="Nvidia", topic="news", timeframe="month")` | chưa chạy |
| G02_out_of_scope_translate | (single) "Dịch câu này sang tiếng Anh: Tôi thích học máy" — ngoài phạm vi, không được lạm dụng `send` (từ R08/R14) | `no_tool=true`, behavior `refuse` | chưa chạy |
| G03_missing_handle_vague | (single) "Xem anh ấy vừa đăng gì trên X" — không rõ "anh ấy" là ai, không đoán (từ R10) | `clarify(response_type="text")` | chưa chạy |
| G04_send_needs_confirm | (single) "Gửi bản tóm tắt tin AI vừa rồi lên kênh Telegram" — hành động ghi phải xác nhận (từ R12) | `clarify(response_type="yes_no")`, không gọi `send` | chưa chạy |
| G05_parallel_web_and_user | (single) "Tìm tin OpenAI hôm nay trên web và xem tweet mới nhất của Sam Altman" — 2 nguồn, 2 tool (từ R13) | `lookup(query="OpenAI", topic="news", timeframe="day")` + `timeline(screenname="sama")` | chưa chạy |
| GM01_topic_override | (multi) "Tin AI tuần này?" → "Đổi sang chip bán dẫn" → "Chỉ chủ đề chip thôi" — lượt mới ghi đè chủ đề, giữ timeframe (từ M02) | `lookup(query="chip bán dẫn", topic="news", timeframe="week")` | chưa chạy |
| GM02_clarify_then_handle | (multi) "Tóm tắt 3 tweet mới nhất" → "Của Andrej Karpathy" → "Đúng 3 tweet nhé" — clarify rồi điền handle, giữ `limit` (từ R10/M01) | `timeline(screenname="karpathy", limit=3)` | chưa chạy |
| GM03_send_after_confirm | (multi) "Đăng bản tin AI hôm nay lên Telegram" → "Nội dung: GPT-6 Cyber ra mắt" → "Có, gửi đi" — chỉ gửi sau khi người dùng xác nhận (từ R12) | `send(text≈"GPT-6 Cyber ra mắt", confirmed=true)` | chưa chạy |
| GM04_url_after_clarify | (multi) "Đọc bài này giúp mình" → "https://deepmind.google/blog" → "Chỉ đọc link đó" — không bịa URL, dùng đúng link được đưa (từ R11/M04) | `fetch(url="https://deepmind.google/blog")` | chưa chạy |
| GM05_add_second_source | (multi) "Tin robotics hôm nay trên web" → "Tìm thêm cả tweet về robotics" → "Gộp cả hai nguồn" — lượt sau bổ sung nguồn, gọi 2 tool (từ R13/M02) | `lookup(query="robotics", topic="news", timeframe="day")` + `social_search(query="robotics")` | chưa chạy |

## B4. Live Chat Evidence

Use `transcripts/*.transcript.json`.

| Turn | User Request | Tool Calls | Version Evidence | Outcome |
|---|---|---|---|---|
|  |  |  |  |  |

## B5. Bonus Evidence

Only fill if your team did bonus.

| Bonus | Evidence File | What Worked | Risk / Guardrail |
|---|---|---|---|
| send (Telegram) |  |  |  |
| arXiv/company policy |  |  |  |
| UI |  |  |  |
| Tool unit tests (`tests/`) | `pytest` output |  |  |

Tool unit tests: số test đã viết, số bug tìm được trong tool có sẵn (nếu có), và một ví dụ lỗi mà unit test bắt được nhưng eval không bắt được.

## B6. Reflection

- Which fixes belonged in `system_prompt.md`?
- Which fixes belonged in `tools.yaml`?
- Which failure needed manual review instead of automatic grading?
- What would you improve next?

## B7. Small Local Model Challenge (bonus)

Only fill if your team did the local model challenge.

| Version | Model (provider) | Prompt/Tools | case_accuracy | tool_routing | argument | multiturn | Run File |
|---|---|---|---|---|---|---|---|
| v3 | (cloud) | v3 |  |  |  |  |  |
| v3-... | (ollama) | v3 |  |  |  |  |  |
| v3-... | (ollama) | v3 |  |  |  |  |  |
| v4-local |  |  |  |  |  |  |  |
| v5-local |  |  |  |  |  |  |  |

Lỗi đặc thù của model nhỏ (đếm từ run JSON):

| Failure Pattern | Count | Example Case | Fix Tried | Helped? |
|---|---|---|---|---|
| wrong / hallucinated tool |  |  |  |  |
| invalid JSON args (`_invalid_json_arguments`) |  |  |  |  |
| no tool call / answered directly |  |  |  |  |
| missed `clarify` |  |  |  |  |
| repeated / unnecessary calls |  |  |  |  |

- Cái gì giúp model nhỏ gọi tool đúng hơn nhất? Bằng chứng?
- Bản prompt tối ưu cho local chạy trên model cloud thì sao?
- Khi nào nhóm sẽ chọn model local thay vì cloud cho agent này?
