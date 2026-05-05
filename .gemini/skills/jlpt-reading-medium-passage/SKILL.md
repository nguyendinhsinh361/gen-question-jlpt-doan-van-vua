---
name: jlpt-doan-van-vua
description: >
  Generate JLPT "đoạn văn vừa" (medium-passage / 中文読解) reading comprehension passages as
  styled HTML files and output CSV training data for AI fine-tuning. Each passage is a medium-length
  Japanese prose text (250–620 characters depending on level) testing the learner's ability to
  understand causal relations, reasoning, author's ideas, reference phrases, and key vocabulary
  via 2-3 multiple-choice questions per passage (N1/N2/N5 = 2 câu, N3/N4 = 3 câu).
  Skill này bao gồm TOÀN BỘ luồng: gen → QC loop (checklist PASS/FAIL) → sửa. Gen từng bài
  một, kiểm tra đến khi đạt chất lượng mới chuyển sang bài tiếp theo. Output chỉ gồm HTML +
  CSV (không có screenshot PNG).
  Skill này chỉ dành riêng cho dạng "đoạn văn vừa" (中文読解).
  Use this skill whenever the user wants to: gen bài đoạn văn vừa, tạo nội dung đoạn văn
  vừa, generate medium-passage reading comprehension, create JLPT 中文 passages,
  produce AI fine-tuning data for the đoạn văn vừa section of JLPT N1-N5,
  kiểm tra chất lượng, quality check, review bài, QC.
  Also trigger when the user mentions: gen bài đoạn văn vừa, tạo medium passage, generate
  JLPT 中文読解, medium reading passage N1/N2/N3/N4/N5.
---

# JLPT 中文 / Đoạn Văn Vừa — Workflow

> **Nguyên tắc cốt lõi:**
> 1. **Gen từng bài một** — không batch rồi QC sau
> 2. **Multi-question** — mỗi bài có 2-3 câu hỏi phủ các đoạn/ý KHÁC NHAU (N1/N2/N5=2, N3/N4=3)
> 3. **Agent tự QC** — đọc lại bài + toàn bộ câu hỏi, tự đánh giá từng mục, log PASS/FAIL
> 4. **1 FAIL = chưa xong** — sửa → QC lại → lặp đến khi ALL PASS
> 5. **KHÔNG có screenshot** — đoạn văn vừa không cần PNG

## Cấu trúc file

| File | Nội dung | Đọc khi |
|------|----------|---------|
| `SKILL.md` (file này) | Workflow + QC Checklist | Luôn đọc đầu tiên |
| `rules/content.md` | R1 chủ đề + R2 layout/char counts/Q count + R7 formats + R8 visual multi-question | Gen HTML |
| `rules/vocabulary.md` | R3 từ vựng/ngữ pháp + R4 furigana | Gen HTML + QC |
| `rules/questions.md` | R5 câu hỏi + R6 đáp án/6 bẫy + multi-question coverage | Gen Q&A + QC |
| `rules/technical.md` | R9 HTML template 720px + R10 clean HTML + R11 CSV multi-question | Gen HTML + CSV |
| `references/html-patterns.md` | Template chi tiết per level + marker conventions | Tra cứu khi gen HTML |
| `references/sample-analysis.md` | Phân tích định lượng data mẫu | Hiểu tần suất pattern |
| `scripts/process_html.py` | Xử lý HTML → CSV + count + validate + multi-question support | Gen CSV + QC |
| `scripts/fill_qa.py` | Điền Q&A vào CSV (quote an toàn, multi-question) | Sau khi gen Q&A |
| `scripts/load_references.py` | Load sample JSON để calibrate | BƯỚC 0 chuẩn bị |

## Outputs Per Passage

1. **Styled HTML** → `assets/html/doan_van_vua/{LEVEL}_{uuid}.html`
2. **Clean HTML** → CSV column `text_read` (no attributes, no `<rt>` content)
3. **CSV row** → `sheets/samples_v1.csv` với 2-3 câu hỏi populate (slot còn lại empty)

**KHÔNG có screenshot PNG.** CSV column `general_image` luôn `""`.

## Số câu hỏi per level (BẮT BUỘC)

| Level | Q Count | Slots populate | Slots empty |
|-------|---------|----------------|-------------|
| N1    | 2       | `question_{1,2}` | `question_{3,4,5}` |
| N2    | 2       | `question_{1,2}` | `question_{3,4,5}` |
| N3    | 3       | `question_{1,2,3}` | `question_{4,5}` |
| N4    | 3       | `question_{1,2,3}` | `question_{4,5}` |
| N5    | 2       | `question_{1,2}` | `question_{3,4,5}` |

> **⛔ COVERAGE RULE**: 2-3 câu hỏi trong 1 bài PHẢI test các đoạn/ý KHÁC NHAU, không được trùng phủ 1 đoạn.
> **⛔ LABEL DIVERSITY**: ≥ 2 label khác nhau trong 1 bài (ngoại lệ: N5 có thể 2 `question_content_match`).

---

# WORKFLOW

## BƯỚC 0: CHUẨN BỊ (1 lần cho batch)

1. **Đọc `rules/rule_doc_hieu.md`** — **Bộ Tiêu Chí Đánh Giá Đọc Hiểu JLPT toàn diện** từ giáo viên (source-of-truth, 11 phần: 4 tiêu chí, 程度 ±, 書き下ろし/による, ①② 下線 空欄 注 ※, **文体の統一 (thể chia)**, furigana per level, 8 loại câu hỏi, 5 loại bẫy chuẩn (+ Single-side cho 統合理解), tiêu chí chi tiết per level).
   **Phần áp dụng trực tiếp cho dạng đoạn văn vừa (中文)**:
   - Phần 1 (Tổng quan & Nguyên tắc 程度) — biên ± per level
   - Phần 2 (Hình thức) — phân bổ ①② 下線 注 theo dạng bài; N4 中文 bắt đầu có ①②
   - **Phần 2.4 (Thể chia nhất quán 文体の統一)** — N1/N2/N3 dùng **普通形** (だ・である); N4/N5 dùng **ます形** (です・ます). Văn bản + câu hỏi + 4 đáp án phải **thống nhất thể chia**. N5 thêm **わかち書き** (khoảng trắng giữa các cụm từ).
   - Phần 3 (Furigana) — bảng quy tắc per level
   - Phần 4 (8 loại câu hỏi) — đặc biệt content_match, reference, reason_explanation, content_mismatch (N3), author_opinion (N2), fill_in (N1/N2)
   - Phần 5 (5 loại bẫy chuẩn)
   - **Phần 6.2 (N5 中文)**, **Phần 7.2 (N4 中文)**, **Phần 8.2 (N3 中文)**, **Phần 9.2 (N2 中文)**, **Phần 10.2 (N1 中文)** — tiêu chí chi tiết 4 chiều cho từng level
   - Phần 11 (Bảng so sánh tổng hợp) — tra cứu nhanh.
2. **Đọc rules skill**: `rules/content.md` + `rules/vocabulary.md` + `rules/technical.md` + `rules/questions.md`
3. **Đọc `rules/kanji_jlpt_sensei.csv`** — dùng để tra level từng kanji khi quyết định furigana
4. **Scan `sheets/samples_v1.csv` và `data/doan_van_vua_n*_clean.json`** — xem format, topic đã dùng → chọn format chưa/ít dùng
5. **Load 2-3 sample calibrate style**:
   ```bash
   python3 .claude/skills/jlpt-reading-medium-passage/scripts/load_references.py --level N3 --count 3
   ```
6. **Lập kế hoạch batch**: mỗi bài gán format + topic + combo question_label khác nhau theo distribution của level. Topic chọn **tiếng Anh** từ cột `en` của `rules/topic.json` (đa dạng ≥ 3 category trong batch > 5 bài).

---

## BƯỚC 1→5: LẶP CHO TỪNG BÀI

### BƯỚC 1: GEN HTML + CÁC CÂU HỎI

> Đọc: `rules/content.md` + `rules/vocabulary.md` + `rules/technical.md` + `rules/questions.md`
> Tham khảo: `references/html-patterns.md` cho template per level

1. **Gen `_id`** = `{LEVEL}_{uuid.uuid4().hex}` (full 32-char hex)
2. **Chọn format** từ R7 (`rules/content.md`) — 6 formats: essay/anecdote/advice/article/letter/diary.
3. **Chọn `tag`** (topic) — **tiếng Anh** từ cột `en` của `rules/topic.json`, đa dạng trong batch
4. **Chọn combo `question_label`** theo level (R5):
   - **N1** (2 câu): 1 `question_fill_in_the_blank` + 1 `question_reason_explanation`/`question_author_opinion`
   - **N2** (2 câu): 1 `question_reference` (marker ①) + 1 `question_reason_explanation`/`question_author_opinion`
   - **N3** (3 câu): 1 `question_reference` + 1 `question_reason_explanation` + 1 `question_content_match`
   - **N4** (3 câu): 2 `question_content_match` + 1 `question_reference`/`question_reason_explanation`
   - **N5** (2 câu): 2 `question_content_match`, HOẶC 1 `question_content_match` + 1 `question_reference`
5. **Gen HTML** theo rules → save `assets/html/doan_van_vua/{LEVEL}_{uuid}.html`
   - Container `max-width: 720px; margin: 0 auto`
   - `word-break: keep-all` (đảm bảo xuống dòng sạch ở ranh giới từ)
   - `<p>` thuần, không `<br>` giữa câu (trừ N5 letter/diary)
   - Marker ①②③ khớp với các câu hỏi `question_reference`/`question_fill_in_the_blank`
   - Furigana chỉ cho từ vượt level (tra `rules/kanji_jlpt_sensei.csv`)
   - **Minimum 3 paragraph** cho N1-N4 (cần đủ nội dung cho multi-question)
6. **Gen 2-3 câu hỏi + 4 đáp án mỗi câu** theo `rules/questions.md`:
   - 4 options ngăn cách `\n`, KHÔNG số thứ tự `1.`, `①`, `1)`
   - `correct_answer_i` = integer 1-4
   - Mỗi distractor PHẢI dùng info/ý THẬT từ bài (1 trong 6 loại bẫy: Reversal/Detail swap/Scope/Misinterpretation/Part of truth/Over-generalization)
   - Giải thích `explain_vn_i` + `explain_en_i` theo format 3 phần
7. **Tạo CSV row** bằng `process_html.py` hoặc `fill_qa.py` (⚠️ **dùng script, KHÔNG sửa CSV tay**):
   ```bash
   # Recommended cho multi-question: JSON
   python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
     --file assets/html/doan_van_vua/{LEVEL}_{uuid}.html \
     --csv sheets/samples_v1.csv \
     --tag "{topic_vn}" \
     --questions-json /tmp/qs.json
   ```
   Hoặc dùng `fill_qa.py` với flags per-question:
   > **⛔ KHÔNG ĐƯỢC sửa CSV bằng tay. Commas + newlines trong nội dung (`100,000円`, `A\nB\nC\nD`) sẽ làm vỡ cột.**
   ```bash
   python3 .claude/skills/jlpt-reading-medium-passage/scripts/fill_qa.py \
     --csv sheets/samples_v1.csv --row-id {LEVEL}_{uuid} --level N3 \
     --q1-label question_reference --q1 "..." --a1 "A\nB\nC\nD" --ca1 2 --evn1 "..." --een1 "..." \
     --q2-label question_reason_explanation --q2 "..." --a2 "A\nB\nC\nD" --ca2 3 --evn2 "..." --een2 "..." \
     --q3-label question_content_match --q3 "..." --a3 "A\nB\nC\nD" --ca3 1 --evn3 "..." --een3 "..."
   ```

---

### BƯỚC 2: ⛔ QC — AGENT TỰ ĐÁNH GIÁ CHECKLIST

> **ĐÂY LÀ BƯỚC QUAN TRỌNG NHẤT. KHÔNG ĐƯỢC BỎ QUA.**
>
> Agent phải **đọc lại** file HTML vừa gen + **TOÀN BỘ** câu hỏi/đáp án trong CSV,
> rồi **tự đánh giá từng mục** bên dưới. Log kết quả theo format:
>
> ```
> QC: {_id}  |  Level: N3  |  Q count: 3/3  |  Labels: [reference, reason, content_match]
> ────────────────────────────────
> [ 1] ✅ PASS — Char count (425 chars, range 380-500)
> [ 2] ❌ FAIL — Flow text (found 2x 。<br>)
> [ 3] ✅ PASS — Container CSS (720px, margin auto)
> ...
> ────────────────────────────────
> ⚠️ 1 FAIL → sửa rồi QC lại
> ```
>
> **⛔ KHÔNG ĐƯỢC tự PASS mà không đọc lại nội dung. Phải confirm từng mục cho TẤT CẢ câu hỏi.**

---

### BƯỚC 3: ⛔ CHECKLIST — TẤT CẢ PHẢI PASS

> **Quy tắc: 1 FAIL = chưa xong. Sửa → QC lại từ đầu → lặp đến khi ALL PASS.**
> **Tổng: 30 checks ở 4 phần (A HTML, B content, C questions/answers + C2 verify, D multi-question coverage).**

#### PHẦN A: HTML (10 checks)

Agent đọc lại file HTML và kiểm tra:

| # | Check | Cách verify | PASS nếu |
|---|-------|-------------|----------|
| 1 | **Char count** | Chạy `process_html.py --count-only --file ...` | Trong Target Range: N5 270-310, N4 490-610, N3 380-500, N2 530-610, N1 550-620 |
| 2 | **Không Hard Reject** | So với Hard Reject threshold | ≥ N5:250, N4:450, N3:350, N2:500, N1:500 |
| 3 | **Flow text** | Tìm `。<br>` trong HTML | Không có `。<br>` nào (trừ N5 letter/diary format) |
| 4 | **Container CSS** | Xem CSS | `max-width: 720px`, `margin: 0 auto`, `word-break: keep-all` (KHÔNG `auto-phrase`) |
| 5 | **`.passage` div** | Xem HTML structure | Có `<div class="passage">` bọc nội dung |
| 6 | **White background** | Xem CSS | `.passage` có `background: white`, body `#f9fafb` |
| 7 | **Furigana format** | Tìm ngoặc `漢字(かんじ)` hoặc `漢字【かんじ】` | Không có — tất cả furigana dùng `<ruby><rt>` |
| 8 | **Ruby có `<rt>` không rỗng** | Xem mọi `<ruby>...</ruby>` | Tất cả đều có `<rt>` chứa furigana **không rỗng** (vd `<ruby>興味<rt>きょうみ</rt></ruby>`). CẤM `<ruby>興味</ruby>` (thiếu rt) hoặc `<ruby>興味<rt></rt></ruby>` (rt rỗng). Auto-check: `process_html.py --validate` |
| 9 | **Ruby count** | Đếm số `<ruby>` | Trong ngưỡng: N5 0-3, N4 0-6, N3 0-10, N2 0-8, N1 0-4 |
| 10 | **Marker/annotation/source đúng level** | Xem có `<u>`, marker ①, `注`, source line, blank `[ ]` không | Phù hợp level (N4/N5 KHÔNG source, N5 KHÔNG annotation; N1 khuyến khích blank; xem R8) |

#### PHẦN B: NỘI DUNG & TỪ VỰNG (6 checks)

| # | Check | Cách verify | PASS nếu |
|----|-------|-------------|----------|
| 11 | **Chủ đề đúng level** | Đọc nội dung, đối chiếu R1 | Chủ đề phù hợp level (N5: nhật ký/thư; N1: luận triết học) |
| 12 | **Format đúng level** | Đối chiếu R7 | Format nằm trong 6 formats được phép cho level đó |
| 13 | **Nội dung logic + đủ depth cho multi-question** | Đọc toàn bài | Ý nhất quán, có ≥ 3 paragraph cho N1-N4, đủ nội dung phủ 2-3 câu hỏi khác nhau |
| 14 | **Không mơ hồ (test 2 cách hiểu)** | Đọc từng câu, thử hiểu theo cách 2 | Chỉ có DUY NHẤT 1 cách hiểu hợp lý cho từng câu hỏi |
| 15 | **Từ vựng đúng level** | Đọc từng từ, đối chiếu R3 | Key terms ≤ level, không dùng ngữ pháp vượt level |
| 16 | **Furigana đúng từ (tra CSV)** | Tra từng kanji trong `rules/kanji_jlpt_sensei.csv` | Mọi từ có kanji vượt level đều có `<ruby><rt>`. Không thừa furigana cho từ đúng level. Không dạng "Ab" (週かん) |

#### PHẦN C: CÂU HỎI & ĐÁP ÁN (10 checks — áp dụng cho TỪNG câu hỏi)

Agent đọc TOÀN BỘ câu hỏi + 4 đáp án từ CSV và đánh giá từng câu (Q1, Q2, Q3 nếu có):

| # | Check | Cách verify | PASS nếu |
|----|-------|-------------|----------|
| 17 | **Số câu hỏi đúng level** | Xem CSV đếm slot đã fill | N1/N2/N5: Q1+Q2 có content, Q3-Q5 empty. N3/N4: Q1+Q2+Q3 có content, Q4-Q5 empty |
| 18 | **question_label đúng intent (mỗi câu)** | Đối chiếu R5 với nội dung từng câu | Label có `question_` prefix, khớp với dạng câu hỏi thực tế |
| 19 | **≥ 2 label khác nhau trong bài** | Đếm unique labels | ≥ 2 labels khác nhau (ngoại lệ: N5 có thể 2 `question_content_match`) |
| 20 | **Marker khớp câu hỏi (mỗi câu)** | So marker trong HTML với câu hỏi | Mọi `question_reference` có `<u>` + marker ①/② trong HTML. Mọi `question_fill_in_the_blank` có `[ ① ]`/`( 1 )` trong HTML |
| 21 | **Answer format (mỗi câu)** | Xem 4 đáp án trong CSV | Đúng 4 options ngăn cách `\n`, KHÔNG `1.`/`①`/`1)` prefix. Độ dài tương đương (ratio < 2.0) |
| 22 | **correct_answer (mỗi câu + batch)** | Xem giá trị `correct_answer_i` | Integer 1-4. Scan batch: không lặp cùng vị trí ≥ 3 bài liên tiếp |
| 23 | **Paraphrase đáp án đúng (mỗi câu, N3+)** | So đáp án đúng với bài gốc | KHÔNG trùng cụm ≥ 4 từ liên tiếp (N3+) hoặc ≥ 6 từ (N4/N5) |
| 24 | **Distractor đa dạng bẫy (mỗi câu)** | Phân loại 3 distractor | ≥ 3 loại bẫy khác nhau trong (6: Reversal/Detail swap/Scope/Misinterpretation/Part of truth/Over-generalization) |
| 25 | **Distractor có căn cứ trong bài (mỗi câu)** | Với mỗi đáp án sai: trích được câu/vị trí trong bài để bác bỏ | KHÔNG bịa. Mỗi distractor dùng info/concept từ bài nhưng sai ngữ cảnh |
| 26 | **Explanations 3 phần (mỗi câu)** | Đọc `explain_vn_i` + `explain_en_i` | Có đủ 3 phần: đáp án đúng (trích vị trí) + đáp án sai (nêu loại bẫy) + tóm tắt. Cả VN và EN đầy đủ |

#### PHẦN C2: VERIFY ĐÁP ÁN (⛔ QUAN TRỌNG NHẤT) — 2 checks

> **Agent tự giải từng câu từ đầu — KHÔNG nhìn đáp án đã gen.**
> Đây là bước bắt lỗi distractor bịa, câu hỏi ambiguous, sai `correct_answer`.

| # | Check | Cách verify | PASS nếu |
|----|-------|-------------|----------|
| 27 | **Tự giải Q1→Q{n}** | Đọc bài + từng câu hỏi, tự chọn đáp án từ đầu (KHÔNG nhìn `correct_answer_i`) | TẤT CẢ kết quả tự chọn KHỚP với `correct_answer_i` trong CSV |
| 28 | **Distractor self-test (toàn bộ câu)** | Với TỪNG đáp án sai trong TỪNG câu: trích dẫn chính xác câu/vị trí trong bài dùng để bác bỏ | Mọi distractor đều trích được. Không trích được = BỊA → FAIL |

#### PHẦN D: MULTI-QUESTION COVERAGE — 2 checks

> **Đặc biệt riêng cho đoạn văn vừa**: 2-3 câu hỏi phải test các đoạn/ý khác nhau.

| # | Check | Cách verify | PASS nếu |
|----|-------|-------------|----------|
| 29 | **Mỗi câu hỏi test đoạn/ý KHÁC nhau** | Xác định paragraph/ý mà mỗi câu hỏi chỉ vào | Không 2 câu cùng hỏi 1 paragraph/ý/cụm từ reference. Q1 + Q2 (+ Q3) phủ ≥ 2 paragraph khác nhau |
| 30 | **Marker/blank khớp câu hỏi tương ứng** | Với mỗi `①`/`②`/`[ ① ]` trong HTML, confirm có câu hỏi hỏi về nó | Không có marker dư (không câu hỏi hỏi) và không câu hỏi nào hỏi marker không tồn tại |

---

### BƯỚC 4: SỬA & LẶP LẠI

> **⛔ Khi sửa HTML, CẬP NHẬT CSV — chạy lại `process_html.py --refresh` để cập nhật `text_read`, `jp_char_count` trong CSV.**
>
> **🚨 ĐẶC BIỆT khi sửa `<ruby>` thiếu/rỗng `<rt>`:** Đây là lỗi PHỔ BIẾN — agent hay chỉ sửa HTML mà QUÊN refresh CSV → CSV cột `text_read` vẫn chứa ruby hỏng → AI fine-tuning data BỊ HỎNG.
> Workflow BẮT BUỘC khi sửa ruby:
> 1. Sửa HTML: thay `<ruby>興味</ruby>` → `<ruby>興味<rt>きょうみ</rt></ruby>`
> 2. **BẮT BUỘC** chạy: `python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py --refresh --html-dir assets/html/doan_van_vua --csv sheets/samples_v1.csv`
> 3. Verify: `python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_vua --csv sheets/samples_v1.csv` — output PHẢI có dòng `✅ CSV ...: 0 row với broken ruby`. Nếu vẫn báo `🚫 CSV ... có N row với broken ruby` → CSV chưa sync, chạy lại `--refresh`.
>
> Không có screenshot nên KHÔNG cần chạy lại screenshot script.

| Nếu FAIL | Hành động | Sau đó |
|-----------|-----------|--------|
| #1, #2 (chars) | Bổ sung/cắt nội dung. Nếu Hard Reject → gen lại hoàn toàn | Chạy `--refresh` → QC lại |
| #3 (flow text) | Sửa `<br>` → `</p><p>` | Chạy `--refresh` → QC lại |
| #4, #5, #6 (CSS/structure) | Sửa CSS/structure HTML (720px) | Chạy `--refresh` → QC lại |
| #7, #8, #9 (ruby) | Sửa ruby tags | Chạy `--refresh` → QC lại |
| #10 (visual level) | Thêm/bớt annotation/source/marker theo R8 | Chạy `--refresh` → QC lại |
| #11-#15 | Gen lại nội dung (giữ _id) | Chạy `--refresh` → QC lại |
| #16 (furigana tra CSV) | Sửa ruby tags (tra lại `rules/kanji_jlpt_sensei.csv`) | Chạy `--refresh` → QC lại |
| #17 (số câu hỏi) | Thêm/xóa câu bằng `fill_qa.py` để đúng số slot | QC lại |
| #18, #19 (labels) | Sửa label trong `fill_qa.py` (dùng đủ `question_` prefix + đa dạng) | QC lại |
| #20 (marker ko khớp) | Sửa HTML (thêm/bớt marker) hoặc sửa câu hỏi | Chạy `--refresh` nếu sửa HTML → QC lại |
| #21, #22, #23 (đáp án) | Sửa đáp án bằng `fill_qa.py` | QC lại |
| #24 (distractor bẫy) | Viết lại distractor dùng 1 trong 6 loại bẫy | QC lại |
| #25 (distractor bịa) | Viết lại distractor dùng info thật từ bài | QC lại |
| #26 (explanation) | Viết lại explain 3 phần đầy đủ cho từng câu | QC lại |
| #27, #28 (self-solve) | Đáp án có thể sai → xem lại bài vs. đáp án | Sửa đáp án hoặc bài. QC lại |
| #29 (coverage trùng) | Sửa Q2/Q3 để hỏi đoạn khác hoặc thay nội dung bài để có đủ depth | QC lại |
| #30 (marker dư/thiếu) | Thêm/bớt marker trong HTML hoặc sửa câu hỏi | Chạy `--refresh` → QC lại |

**Lệnh refresh CSV sau khi sửa HTML:**
```bash
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
  --refresh \
  --file assets/html/doan_van_vua/{LEVEL}_{uuid}.html \
  --csv sheets/samples_v1.csv
```

> **Vòng lặp: sửa → refresh CSV (nếu sửa HTML) → quay lại BƯỚC 2 (QC lại TẤT CẢ) → nếu còn FAIL thì lặp lại.**
> **Tối đa 5 vòng. Sau 5 vòng vẫn FAIL → báo lỗi cho user, KHÔNG bỏ qua.**

---

### BƯỚC 5: ✅ HOÀN THÀNH → BÀI TIẾP THEO

Chỉ khi **TẤT CẢ 30 checks PASS** → log:
```
🎉 ALL PASSED (30/30) — {_id} hoàn thành — {n} câu hỏi ({labels})
```
→ Chuyển sang bài tiếp theo (quay lại BƯỚC 1).

---

## BƯỚC CUỐI: VERIFY BATCH (sau khi gen xong TẤT CẢ bài)

Sau khi hoàn thành toàn bộ batch, chạy verify toàn bộ:

```bash
# 1. Validate tất cả file HTML (char count + broken ruby)
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
  --validate --html-dir assets/html/doan_van_vua

# 2. Đếm số rows trong CSV + check số câu hỏi
python3 -c "
import csv
expected_q = {'N1': 2, 'N2': 2, 'N3': 3, 'N4': 3, 'N5': 2}
with open('sheets/samples_v1.csv', 'r', encoding='utf-8') as f:
    rows = list(csv.DictReader(f))
print(f'Total rows: {len(rows)}')
bad = 0
for r in rows:
    lv = r.get('level')
    want = expected_q.get(lv, 0)
    got = sum(1 for i in range(1, 6) if r.get(f'question_{i}', '').strip())
    if got != want:
        bad += 1
        print(f\"  ❌ {r['_id']} ({lv}): {got} câu, expected {want}\")
print(f'Multi-question OK: {len(rows) - bad}/{len(rows)}')
for level in ['N1','N2','N3','N4','N5']:
    n = sum(1 for r in rows if r.get('level') == level)
    print(f'  {level}: {n}')
"
```

### Batch-level checklist

- [ ] Mỗi bài có `_id` unique, đúng format `{LEVEL}_{uuid}`
- [ ] `kind` = `đoạn văn vừa` trong tất cả rows
- [ ] `general_image` = `""` (empty) — KHÔNG có PNG
- [ ] `general_audio` = `""` (empty)
- [ ] Char count trong Target Range cho mọi bài
- [ ] Không bài nào dưới Hard Reject threshold
- [ ] Furigana chỉ cho từ vượt level, không dạng "Ab", mọi `<ruby>` có `<rt>`
- [ ] Ruby tags count ≤ expected per level
- [ ] **Mỗi bài có đúng số câu hỏi theo level** (N1/N2/N5=2, N3/N4=3) — các slot khác empty
- [ ] `question_label_i` có `question_` prefix (7 labels hợp lệ)
- [ ] Trong 1 bài có ≥ 2 label khác nhau (ngoại lệ N5)
- [ ] Mỗi câu hỏi có 4 đáp án ngăn cách `\n` (KHÔNG số thứ tự)
- [ ] `correct_answer_i` phân bố đều 1-4 trong batch
- [ ] Distractor dùng info từ bài (không bịa), ≥ 3 loại bẫy khác nhau per câu
- [ ] `explain_vn_i` + `explain_en_i` đủ 3 phần cho mọi câu
- [ ] Marker trong text khớp câu hỏi (`①` ↔ Q1 reference, `[ ① ]` ↔ Q fill_blank...)
- [ ] **Multi-question coverage**: Q1/Q2/Q3 test các đoạn/ý khác nhau trong từng bài
- [ ] `text_read` clean — không attribute, không class, không `<rt>` content
- [ ] `<p>` thuần, không `<br>` giữa câu (trừ N5 letter/diary)
- [ ] Annotation (注) giải thích bằng **tiếng Nhật đơn giản**, KHÔNG tiếng Anh/Việt
- [ ] Trong batch, tag đa dạng ≥ 3 category (nếu batch > 5)

---

## Reference Data & Samples

Data mẫu có sẵn trong `data/`:

| Level | File | Samples |
|-------|------|---------|
| N1 | `doan_van_vua_n1_clean.json` | ~100 |
| N2 | `doan_van_vua_n2_clean.json` | ~150 |
| N3 | `doan_van_vua_n3_clean.json` | ~130 |
| N4 | `doan_van_vua_n4_clean.json` | ~110 |
| N5 | `doan_van_vua_n5_clean.json` | ~95 |

Load bằng:
```bash
# Stats tất cả levels
python3 .claude/skills/jlpt-reading-medium-passage/scripts/load_references.py --stats

# 3 random samples N3
python3 .claude/skills/jlpt-reading-medium-passage/scripts/load_references.py --level N3 --count 3
```

**LƯU Ý khi đọc data gốc**:
- Data gốc DÙNG `<br>` nhiều — thói quen xấu. Output HTML skill KHÔNG theo.
- Data gốc có `<span>` bọc paragraph — output KHÔNG cần.
- Data gốc N1 thường có 3 câu hỏi — skill follow spec `EXPECTED_Q_COUNT` (N1=2), không bắt chước data.
- Data gốc dùng ruby không đều (N4 thường rắc ruby thừa) — output follow `rules/kanji_jlpt_sensei.csv`.

Chi tiết phân tích từng level xem `references/sample-analysis.md`.

---

## Common errors (dạng đoạn văn vừa hay gặp)

1. **Chỉ gen 1 câu hỏi** — SAI, đoạn văn vừa bắt buộc 2-3 câu theo level
2. **Gen thừa (4-5 câu) cho N1 vì bắt chước data** — SAI, N1 spec chỉ 2 câu
3. **Tất cả câu dùng cùng 1 label** — SAI, cần ≥ 2 labels khác nhau
4. **2 câu cùng hỏi 1 đoạn** — SAI, coverage rule yêu cầu đoạn khác nhau
5. **Marker `①` trong bài nhưng không có câu hỏi reference** — marker vô nghĩa
6. **Câu hỏi hỏi `①` nhưng HTML không có marker** — câu hỏi không thể trả lời
7. **N4/N5 có source line** — 0% data thực tế, không phù hợp level
8. **Distractor quá dễ** — đoạn văn vừa đòi hỏi distractor dùng ≥ 3 loại bẫy đa dạng
9. **Quên `question_` prefix trong label** — label phải là `question_content_match`, không phải `content_match`

---

## Cảnh báo bảo mật dữ liệu

> **🚫 KHÔNG ĐƯỢC GHI VÀO THƯ MỤC `rules/`** — `rules/question_sheet.csv`, `rules/topic.json`, `rules/kanji_jlpt_sensei.csv`, `rules/question_format.json`, `rules/mission.json`, `rules/rule_doc_hieu.md` là file tham chiếu, chỉ đọc. Mọi dữ liệu gen phải ghi vào `sheets/samples_v1.csv`.
