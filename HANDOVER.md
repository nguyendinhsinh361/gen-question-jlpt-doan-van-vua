# HANDOVER — Đoạn Văn Vừa Skill

Tài liệu giao/nhận cho skill **jlpt-reading-medium-passage** (中文読解 / đoạn văn vừa). Đọc file này trước khi chạy batch gen hoặc khi bàn giao cho team mới.

## 1. Mục đích

Gen dữ liệu huấn luyện AI cho **dạng "đoạn văn vừa"** (中文読解) của JLPT N1-N5. Mỗi bài gồm:

- 1 đoạn văn Nhật độ dài trung bình (250–620 ký tự tuỳ level)
- **2 hoặc 3 câu hỏi** multiple-choice 4 đáp án (N1/N2/N5 = 2 câu, N3/N4 = 3 câu)
- Giải thích tiếng Việt + tiếng Anh cho từng câu

Khác Phase 1 (đoạn văn ngắn 80–290 chars, 1 câu hỏi) và Phase 0 (tìm thông tin, có screenshot PNG): **không cần screenshot**, chỉ HTML + Clean HTML + CSV, NHƯNG mỗi bài có **nhiều câu hỏi**.

## 2. Cấu trúc project

```
gen-question-doan-van-vua/
├── data/                                      # Sample JSON từ đề JLPT cũ
│   ├── doan_van_vua_n1_clean.json             # 91 samples
│   ├── doan_van_vua_n2_clean.json             # 87 samples
│   ├── doan_van_vua_n3_clean.json             # 74 samples
│   ├── doan_van_vua_n4_clean.json             # 29 samples
│   └── doan_van_vua_n5_clean.json             # 34 samples
├── .claude/skills/jlpt-reading-medium-passage/
│   ├── SKILL.md                               # Main skill definition
│   ├── scripts/
│   │   ├── process_html.py                    # Count + clean HTML + CSV upsert (multi-Q)
│   │   └── load_references.py                 # Pretty-print JSON cho gen agent
│   └── references/
│       ├── sample-analysis.md                 # Phân tích pattern per level
│       └── html-patterns.md                   # HTML template per level + marker strategy
├── .gemini/skills/jlpt-reading-medium-passage/    # Mirror identical của .claude/
├── assets/html/doan_van_vua/                  # Output HTML files (runtime)
├── sheets/                                     # Output CSV files (runtime)
├── rules/                                      # Schema & spec
│   ├── question_sheet.csv                     # 45-col CSV header
│   ├── question_format.json                   # Số câu hỏi per level
│   ├── kind_mission_mapping.json
│   ├── mission.json                           # Question label catalog
│   └── topic.json
├── HANDOVER.md                                 # (file này)
└── PROMPTS.md                                  # Prompt templates cho gen agent
```

## 3. Pipeline chuẩn

### Bước 1 — Load references (calibrate style)

```bash
cd /path/to/gen-question-doan-van-vua
python3 .claude/skills/jlpt-reading-medium-passage/scripts/load_references.py --stats
python3 .claude/skills/jlpt-reading-medium-passage/scripts/load_references.py --level N3 --count 2 --seed 42
```

Gen agent đọc 2 sample cùng level để học:
- Độ dài (P25–P75 của data)
- Chủ đề phổ biến
- Cấu trúc multi-question (marker ①②③, câu hỏi reference)

**KHÔNG bắt chước styling data gốc** — data có `<br>` và `<ruby>` sai quy tắc. Chỉ học **nội dung + question pattern**.

### Bước 2 — Gen HTML + câu hỏi từ LLM

LLM dùng prompt trong `PROMPTS.md` (template per level) để gen ra:
1. HTML file đầy đủ (có `<!DOCTYPE>`, Noto Sans JP CSS, `max-width: 720px`)
2. **2 hoặc 3 câu hỏi** tuỳ level, mỗi câu 4 đáp án + đáp án đúng
3. Giải thích VN + EN cho từng câu

Output khuyến nghị ở dạng JSON file `questions.json`:

```json
{
  "questions": [
    {
      "label": "question_reference",
      "question": "「①自分たちのシステム」とあるが、何を指すか。",
      "answers": ["A option", "B option", "C option", "D option"],
      "correct": 2,
      "explain_vn": "...",
      "explain_en": "..."
    },
    {
      "label": "question_reason_explanation",
      "question": "なぜ筆者は...と考えているか。",
      "answers": ["A", "B", "C", "D"],
      "correct": 3,
      "explain_vn": "...",
      "explain_en": "..."
    }
  ]
}
```

### Bước 3 — Save HTML

Tên file: `{LEVEL}_{uuid4().hex}.html`. Ví dụ: `N3_a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5.html`

```python
import uuid
filename = f"N3_{uuid.uuid4().hex}.html"
```

Save vào `assets/html/doan_van_vua/`.

### Bước 4 — Process + commit CSV (2 cách)

**Cách A (KHUYẾN NGHỊ) — JSON file chứa tất cả câu hỏi**:

```bash
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
    --file assets/html/doan_van_vua/N3_abc123...html \
    --csv sheets/samples_v1.csv \
    --tag "thí nghiệm" \
    --questions-json /tmp/qs.json
```

**Cách B — CLI flags (chỉ phù hợp khi 2 câu)**:

```bash
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
    --file assets/html/doan_van_vua/N2_abc123...html \
    --csv sheets/samples_v1.csv \
    --tag "phê bình văn hóa" \
    --q1-label question_reference --q1 "①...とあるが..." --a1 "A|B|C|D" --c1 2 --ev1 "..." --ee1 "..." \
    --q2-label question_reason_explanation --q2 "なぜ..." --a2 "A|B|C|D" --c2 3 --ev2 "..." --ee2 "..."
```

Script sẽ:
- Count `jp_char_count` từ full HTML (skip `<rt>`, whitespace)
- Extract clean HTML (bỏ attribute, collapse whitespace, bỏ `<rt>`)
- **Validate số câu hỏi vs spec** (N1/N2/N5=2, N3/N4=3) — warning, không block
- **Validate ≥ 2 labels khác nhau** nếu có ≥ 2 câu — warning
- **Hard-reject** nếu dưới threshold (N1/N2<500, N3<350, N4<450, N5<250) — exit 1, không commit CSV
- Cảnh báo nếu dưới Target Range
- Upsert row vào CSV (45 columns) theo `_id` = filename, populate `question_1..question_{n}`

### Bước 5 — Validate batch

```bash
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
    --validate --html-dir assets/html/doan_van_vua
```

Exit 0 = tất cả pass; 1 = có file fail (UNDER_TARGET / HARD_REJECT).

### Bước 6 — Refresh sau khi edit HTML

Giữ câu hỏi cũ, chỉ refresh `jp_char_count` + `text_read`:

```bash
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
    --refresh --html-dir assets/html/doan_van_vua --csv sheets/samples_v1.csv
```

## 4. Target ranges & số câu (BẮT BUỘC)

| Level | Target Range | Hard Reject | Số câu/bài | Combo đề xuất |
|-------|--------------|-------------|-----------|---------------|
| N1    | 550–620      | < 500       | **2**     | 1 fill_in_the_blank + 1 reference/reason |
| N2    | 530–610      | < 500       | **2**     | 1 reference + 1 reason_explanation |
| N3    | 380–500      | < 350       | **3**     | 1 reference + 1 reason + 1 content_match |
| N4    | 490–610      | < 450       | **3**     | 2 content_match + 1 reference |
| N5    | 270–310      | < 250       | **2**     | 2 content_match (hoặc 1 cm + 1 reference) |

> **LƯU Ý**: N4 target (490–610) cao hơn N3 (380–500) — N4 dùng nhiều hiragana, cùng nội dung dài ký tự hơn. Phản ánh đúng đề JLPT thật.

> **Data vs Spec**: Data gốc có noise (N1/N2 hay 3 câu, N4 hay 4 câu). **Skill LUÔN follow SPEC** ở bảng trên.

## 5. CSV Schema (45 columns)

Populate khác đoạn văn ngắn — có nhiều câu hỏi:

| Column | Value |
|--------|-------|
| `_id`  | `{LEVEL}_{uuid32hex}` |
| `level` | N1/N2/N3/N4/N5 |
| `tag`  | Topic label |
| `jp_char_count` | Result `count_body_chars()` |
| `kind` | Always `đoạn văn vừa` |
| `general_audio` | "" |
| `general_image` | "" (empty — no PNG) |
| `text_read` | Clean HTML |
| `text_read_vn` / `text_read_en` | "" |
| `question_label_1` | Label câu 1 |
| `question_1` | Câu hỏi 1 tiếng Nhật |
| `question_image_1` | "" |
| `answer_1` | `1. A\n2. B\n3. C\n4. D` |
| `correct_answer_1` | 1-4 |
| `explain_vn_1` / `explain_en_1` | Giải thích câu 1 |
| `question_label_2`..`explain_en_2` | Câu 2 (BẮT BUỘC với mọi level) |
| `question_label_3`..`explain_en_3` | Câu 3 (CHỈ N3/N4) |
| `question_4` / `question_5` | **""** (empty — vua tối đa 3 câu) |

## 6. Quality Gates

Trước khi coi 1 batch là xong:

- [ ] 100% file qua Hard Reject threshold
- [ ] ≥ 80% file nằm trong Target Range
- [ ] **100% row có đúng số câu hỏi per level** (N1/N2/N5=2, N3/N4=3)
- [ ] **Trong 1 bài, ≥ 2 `question_label` khác nhau**
- [ ] Các câu hỏi trong 1 bài KHÔNG test cùng 1 đoạn (tránh duplicate nội dung)
- [ ] Marker `①②③` trong HTML khớp với câu hỏi reference
- [ ] Batch ≥ 5 bài có ≥ 3 `tag` khác nhau
- [ ] 0 file có furigana dạng "Ab"
- [ ] 100% file có `general_image = ""`
- [ ] 100% row có `kind = "đoạn văn vừa"`
- [ ] 0 row có `question_4` / `question_5` non-empty
- [ ] Mỗi câu hỏi có 4 đáp án trong `answer_X`, 1 `correct_answer_X` là 1–4
- [ ] `explain_vn_X` + `explain_en_X` đều non-empty cho mọi câu

## 7. Edge cases & pitfalls

1. **Chỉ gen 1 câu hỏi** (như đoạn văn ngắn) — SAI, đoạn văn vừa BẮT BUỘC multi-question.
2. **Gen 3 câu cho N1/N2, 2 câu cho N3/N4** — SAI, phải follow spec.
3. **2 câu hỏi cùng test 1 đoạn** — SAI, phải test các đoạn/cụm từ khác nhau.
4. **Marker ①②③ không match câu hỏi** — câu `①「cụm từ」とあるが` phải có `<span class="marker">①</span><u>cụm từ</u>` trong HTML.
5. **Char count dưới target** vì chỉ có 1-2 paragraph — phải **3-5 paragraph** cho N1-N4 (N5 có thể 3-4).
6. **Distractor quá dễ** — đoạn văn vừa đòi hỏi distractor chi tiết hơn đoạn văn ngắn.
7. **Source line cho N4/N5** — KHÔNG, data gốc 0%.
8. **Annotation 注 cho N5** — KHÔNG, data gốc 0%.
9. **Data gốc có `<br>`** — KHÔNG bắt chước (trừ N5 letter format).
10. **Correct index conversion** — JSON `correctAnswer` là 0-based, CSV `correct_answer_X` là 1-based. `load_references.py` hiển thị cả hai.
11. **Dạng "Ab" (週かん, 友だち)** — tuyệt đối không. Full kanji + ruby HOẶC full hiragana.
12. **Container width** = 720px (rộng hơn 640 của đoạn văn ngắn). Đừng nhầm.

## 8. Sample patterns theo data (tóm tắt, chi tiết xem `references/sample-analysis.md`)

| Pattern | N1 | N2 | N3 | N4 | N5 |
|---------|----|----|----|----|----|
| Có `<u>` underline | **72%** | 47% | 33% | 34% | 20% |
| Có marker ①②③ | 7% | **47%** | **54%** | 17% | 2% |
| Có 注 annotation | 34% | 43% | **54%** | 6% | 0% |
| Có source line | 13% | **20%** | 9% | 0% | 0% |
| Có fill_in_blank `[ ]` | **95%** | 17% | 6% | 0% | 0% |

**Key insights**:
- **N1 = hầu như luôn có fill_in_the_blank** (95% data)
- **N2/N3 = marker + annotation thường xuyên** — multi-Q reference dùng nhiều
- **N4/N5 = simple content match**, không marker/annotation/source

## 9. Tương lai (Phase 3+)

- Phase 3: `đoạn văn dài` (N1 ~1000 chars, N3 ~550) — 3-4 câu/bài
- Phase 4: `đọc hiểu chủ đề` (N1 ~1000, N2 ~900) — 3 câu/bài
- Phase 5: `đọc hiểu tổng hợp` (N1/N2 ~600) — 2 đoạn A+B, 2 câu so sánh
- Post-pipeline: bổ sung `text_read_vn` / `text_read_en` nếu cần bản dịch
- CSV consolidation: merge CSVs của 5 skill thành 1 `sheets/master.csv` để train

## 10. Liên hệ

- Skill owner: Nguyễn Đình Sinh <sinhnd@eupgroup.net>
- Phase trước: `../gen-question-doan-van-ngan/` (đoạn văn ngắn, 1 câu hỏi)
- Phase 0 reference: `../gen-question-jlpt/` (tìm thông tin, có screenshot)
- Master plan: `../PLAN_5_DANG_DOC_HIEU.md`
