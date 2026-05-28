# Prompt — Gen bài Đoạn Văn Vừa (JLPT 中文読解)

Copy prompt bên dưới, thay `{số}` rồi paste vào Claude hoặc Gemini.

## Prompt

```
Đọc .claude/skills/jlpt-reading-medium-passage/SKILL.md và tuân thủ đầy đủ workflow + 5 GATE.

Gen bài đọc hiểu đoạn văn vừa (N1/N2/N5 = 2 câu, N3/N4 = 3 câu):
- N5: {số} | N4: {số} | N3: {số} | N2: {số} | N1: {số}

Lưu CSV: sheets/samples_v1.csv
Lưu HTML: assets/html/doan_van_vua/{LEVEL}_{uuid}.html
```

---

## Prompt QC hậu kỳ

Chạy QC trên CSV đã gen — auto-check scripts + LLM review + auto-fix tối đa 3 vòng.

```
Đọc .claude/skills/jlpt-reading-medium-passage-post-qc/SKILL.md và chạy QC đầy đủ theo workflow.

CSV cần QC: sheets/samples_v1.csv

Phạm vi (chọn 1):
- ALL: toàn bộ CSV
- LEVEL: chỉ rows có level = {N1|N2|N3|N4|N5}
- ID: chỉ row có _id = {LEVEL}_{uuid}

Quy trình BẮT BUỘC:
1. BƯỚC 1 — Auto-check: chạy post_qc.py + check_furigana + check_spacing + check_csv_fields + check_answer_punctuation
2. BƯỚC 2 — LLM review: L1-L16 (đặc thù 中文: L10 N3 markers ①②③ bắt buộc, L11 pattern Q per level, L12 N3 không 注)
3. BƯỚC 3 — Cross-batch: B1-B5 (topic/label/content diversity, label distribution %, (中略) rate)
4. BƯỚC 4 — Auto-fix: row FAIL → sửa tối thiểu phần lỗi (KHÔNG gen lại toàn bộ), lặp tối đa 3 vòng

Báo cáo theo format trong SKILL.md.
```

---

## Prompt với topic chỉ định

Chỉ định topic cho từng level. Topic dùng tiếng Anh từ cột `en` của `rules/topic.json`.

```
Đọc .claude/skills/jlpt-reading-medium-passage/SKILL.md và tuân thủ đầy đủ workflow + 5 GATE.

Gen bài đọc hiểu đoạn văn vừa với số bài + topic chỉ định cho từng level (N1/N2/N5 = 2 câu/bài, N3/N4 = 3 câu/bài):
- N5: 2 bài | topic: family
- N4: 3 bài | topic: travel
- N3: 3 bài | topic: environment
- N2: 2 bài | topic: psychology
- N1: 2 bài | topic: education

Quy tắc:
- Topic PHẢI có trong cột `en` của `rules/topic.json` — kiểm tra trước, không có → DỪNG báo user.
- CSV field `tag` của mỗi row = topic của level đó.
- Nếu nhiều bài cùng level → giữ chung topic NHƯNG mỗi bài câu hỏi cover các đoạn KHÁC NHAU.

Lưu CSV: sheets/samples_v1.csv
Lưu HTML: assets/html/doan_van_vua/{LEVEL}_{uuid}.html
```
