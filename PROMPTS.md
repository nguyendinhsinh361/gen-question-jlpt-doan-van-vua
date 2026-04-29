# Prompt — Gen bài Đoạn Văn Vừa (JLPT 中文読解)

## Cách dùng

Copy prompt bên dưới, thay `{số}` rồi paste vào Claude hoặc Gemini.
SKILL.md chứa workflow + checklist QC. rules/ chứa chi tiết. Prompt chỉ cần nói **cái gì** và **bao nhiêu**.

**Đặc thù dạng này: multi-question** — mỗi bài 2-3 câu hỏi phủ các đoạn/ý KHÁC NHAU (N1/N2/N5 = 2 câu, N3/N4 = 3 câu).

---

## Prompt ngắn (khuyên dùng)

```
Đọc .claude/skills/jlpt-reading-medium-passage/SKILL.md rồi gen bài đọc hiểu đoạn văn vừa:
- N5: {số} bài | N4: {số} bài | N3: {số} bài | N2: {số} bài | N1: {số} bài

Lưu CSV vào sheets/samples_v1.csv. HTML lưu vào assets/html/doan_van_vua/{LEVEL}_{uuid}.html.
Làm đúng theo SKILL.md — từng bài một, đọc rules/ trước khi gen.

⛔ Q COUNT BẮT BUỘC theo level (đọc spec):
- N1 = 2 câu | N2 = 2 câu | N3 = 3 câu | N4 = 3 câu | N5 = 2 câu
- Slot không dùng phải để empty (vd N1 chỉ fill question_{1,2}, question_{3,4,5} = "")

⛔ COVERAGE RULE (multi-question): các câu trong 1 bài PHẢI test các đoạn/ý KHÁC NHAU (không 2 câu cùng hỏi 1 đoạn). Marker ①② trong HTML phải khớp câu hỏi reference.

⛔ ĐA DẠNG — BẮT BUỘC:
1. Đọc rules/rule_doc_hieu.md (rule giáo viên — section 3-5 áp dụng trực tiếp) + rules/content.md (chủ đề + char range) + rules/questions.md (label distribution per level).
2. Scan sheets/samples_v1.csv xem topic + question_label combo đã dùng.
3. Trong cùng level: KHÔNG trùng topic; dùng ≥ 2 question_label khác nhau per bài (label diversity).
4. Tag **tiếng Anh** từ cột `en` của `rules/topic.json` (vd: family, economics, culture). TUYỆT ĐỐI không tiếng Việt/Nhật.

⛔ CHAR RANGE — Hard Reject phải gen lại (xem chi tiết trong rules/content.md):
N1 530–620 | N2 470–560 | N3 380–460 | N4 280–350 | N5 250–320

⛔ FURIGANA — chỉ cho từ VƯỢT level. Cấm dạng "Ab". Tra rules/jlpt_kanji.csv.

Sau khi gen xong mỗi bài, tự QC checklist trong SKILL.md (HTML + CSV + multi-question coverage → log PASS/FAIL). 1 FAIL = sửa → QC lại. Tất cả PASS mới sang bài tiếp.
Điền Q&A bằng scripts/fill_qa.py (KHÔNG sửa CSV bằng tay).
Sửa HTML = chạy lại process_html.py --refresh.
Verify cuối: python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_vua
```

---

## Prompt có thêm ràng buộc (khi cần kiểm soát chất lượng)

```
Đọc .claude/skills/jlpt-reading-medium-passage/SKILL.md rồi gen bài đọc hiểu đoạn văn vừa:
- N5: {số} bài | N4: {số} bài | N3: {số} bài | N2: {số} bài | N1: {số} bài

Lưu CSV vào sheets/samples_v1.csv. HTML lưu vào assets/html/doan_van_vua/{LEVEL}_{uuid}.html.
Trước khi gen:
1. Đọc rules/rule_doc_hieu.md (rule giáo viên — source-of-truth cho vocab/grammar/distractor)
2. Đọc rules/content.md + rules/vocabulary.md + rules/technical.md + rules/questions.md
3. Đọc rules/jlpt_kanji.csv để tra level kanji
4. Đọc 1-2 sample: scripts/load_references.py --level {LEVEL} --count 2
5. Scan sheets/samples_v1.csv xem topic + label combo nào đã dùng

⛔ Q COUNT BẮT BUỘC: N1=2, N2=2, N3=3, N4=3, N5=2. Slot dư phải empty.

⛔ MULTI-QUESTION COVERAGE:
- Mỗi câu test đoạn/ý KHÁC NHAU (không 2 câu trùng đoạn)
- Marker ①② trong HTML khớp câu hỏi reference (① ↔ Q reference 1, ② ↔ Q reference 2)
- Không có marker dư (trong HTML mà không câu hỏi nào hỏi)

⛔ ĐA DẠNG CHỦ ĐỀ + LABEL:
- Trong cùng level: KHÔNG trùng topic; mỗi bài ≥ 2 question_label khác nhau
- Cross-level: ưu tiên topic chưa xuất hiện
- Tag **tiếng Anh** từ cột `en` của `rules/topic.json` (TUYỆT ĐỐI không tiếng Việt/Nhật)

⛔ FURIGANA ZERO-TOLERANCE:
- Kanji vượt level PHẢI có <ruby><rt>
- Cấm dạng "Ab". Chọn 1 trong 2: full kanji + furigana HOẶC full hiragana
- Density per level theo R4 trong rules/vocabulary.md

⛔ ĐÁP ÁN — 4 options newline-separated, KHÔNG prefix "1.", "①", "1)".

Yêu cầu chất lượng câu hỏi:
- Question_label dùng prefix `question_`. ≥ 2 unique labels per bài
- Distractor đa dạng ≥ 3 loại bẫy. Mỗi distractor PHẢI dùng info thật từ bài
- Paraphrase: đáp án đúng KHÔNG copy nguyên văn ≥ 4 từ liên tiếp (N1) / 5 từ (N2-N5)
- Self-solve verify: tự giải từng câu, KHỚP correct_answer_i
- Explanation 3 phần (VN + EN)

Sau khi gen xong mỗi bài, BẮT BUỘC tự QC theo checklist trong SKILL.md:
- Phần A HTML + B Content + C Questions/Answers + D Multi-Q Coverage
- 1 FAIL = sửa → refresh CSV (nếu sửa HTML) → QC lại

Lưu ý kỹ thuật:
- Điền Q&A bằng scripts/fill_qa.py (KHÔNG edit CSV tay)
- Refresh CSV sau sửa HTML: process_html.py --refresh
- Verify cuối: process_html.py --validate --html-dir assets/html/doan_van_vua
```
