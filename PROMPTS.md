# Prompt — Gen bài Đoạn Văn Vừa (JLPT 中文読解)

## Cách dùng

Copy prompt bên dưới, thay `{số}` rồi paste vào Claude hoặc Gemini.

**Đặc thù: multi-question** — mỗi bài 2-3 câu phủ đoạn/ý KHÁC NHAU (N1/N2/N5 = 2 câu, N3/N4 = 3 câu).

> **🚨 ZERO-TOLERANCE QC**: Chỉ cần **1 tiêu chí FAIL** trong checklist QC của SKILL.md → **fix ngay hoặc gen lại** trước khi sang bài tiếp.

---

## Prompt

```
Đọc .claude/skills/jlpt-reading-medium-passage/SKILL.md rồi gen bài đọc hiểu đoạn văn vừa:
- N5: {số} bài | N4: {số} bài | N3: {số} bài | N2: {số} bài | N1: {số} bài

Lưu CSV: sheets/samples_v1.csv. HTML: assets/html/doan_van_vua/{LEVEL}_{uuid}.html.

═══ BƯỚC 0 — CHUẨN BỊ (1 lần) ═══
1. Đọc rules/rule_doc_hieu.md (rule giáo viên — source-of-truth, 11 phần). Áp dụng đặc biệt:
   - Phần 2.4 (Thể chia 文体の統一): N1/N2/N3 → 普通形 (だ・である); N4/N5 → ます形. Văn bản + câu hỏi + 4 đáp án (mọi câu) thống nhất 1 thể. N5 thêm わかち書き.
   - Phần 3 (Furigana per level), Phần 4 (8 loại Q), Phần 5 (5 loại bẫy chuẩn).
2. Đọc rules/content.md + vocabulary.md + technical.md + questions.md.
3. Đọc rules/kanji_jlpt_sensei.csv (2495 kanji) để tra furigana.
4. Load 2-3 sample: scripts/load_references.py --level {LEVEL} --count 2-3.
5. Scan sheets/samples_v1.csv xem topic + question_label combo đã dùng.

═══ BƯỚC 1→5 — LẶP CHO TỪNG BÀI ═══
1. Gen _id = {LEVEL}_{uuid32}; chọn format + topic + Q label combo chưa/ít dùng.
2. Tag = **tiếng Anh** từ cột `en` của rules/topic.json. TUYỆT ĐỐI không tiếng Việt/Nhật.
3. Gen HTML: container đúng spec, <p> thuần (không <br>), marker ①② khớp Q reference, furigana chỉ vượt level (cấm "Ab"), **thể chia đúng level (Phần 2.4)**.
4. Gen Q + 4 đáp án (newline \n, KHÔNG prefix); ≥ 2 unique question_label per bài; mỗi câu test đoạn/ý KHÁC NHAU; distractor ≥ 3 loại bẫy dùng info THẬT.
5. Tạo CSV row bằng scripts/process_html.py. Fill Q&A bằng scripts/fill_qa.py (KHÔNG sửa CSV tay).

═══ BƯỚC 2 — QC ZERO-TOLERANCE (BẮT BUỘC) ═══
Tự đánh giá checklist 4 phần trong SKILL.md, log PASS/FAIL:
- A. HTML + B. Content (chủ đề, từ vựng level, **thể chia nhất quán**) + C. Q&A (label, đáp án, explain VN+EN, self-solve khớp correct) + D. Multi-Q Coverage
- **1 FAIL = fix ngay hoặc gen lại → refresh CSV (nếu sửa HTML) → QC lại từ đầu**. CẤM bỏ qua.

═══ HARD REJECT (gen lại ngay) ═══
- Q count sai: N1=2, N2=2, N3=3, N4=3, N5=2 (slot dư phải empty)
- Char range ngoài (xem rules/content.md): N1 530–620 | N2 470–560 | N3 380–460 | N4 280–350 | N5 250–320
- 2 câu cùng test 1 đoạn/ý (vi phạm coverage)
- Marker ①② trong HTML không khớp Q reference (hoặc marker dư)
- <ruby> thiếu <rt> hoặc <rt> rỗng; furigana dạng "Ab"
- Thể chia trộn lẫn trong bài (vd N3 vừa だ vừa です)
- Tag tiếng Việt/Nhật; trong cùng level: trùng topic hoặc <2 unique label per bài

═══ CUỐI BATCH ═══
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_vua --csv sheets/samples_v1.csv
```
