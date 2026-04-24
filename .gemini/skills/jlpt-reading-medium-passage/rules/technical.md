# Rules: HTML Template, Clean HTML, CSV Schema (R9, R10, R11)

> **Scope**: Đoạn văn vừa (medium-passage). **KHÔNG có screenshot PNG** — `general_image` luôn empty. Mỗi bài có **2-3 câu hỏi** (N1/N2/N5=2, N3/N4=3) populate vào cùng 1 row CSV.

## R9. HTML Template

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{JP_TITLE_VỪA}</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700&display=swap');
        body {
            font-family: 'Noto Sans JP', sans-serif;
            background: #f9fafb;
            color: #111827;
            line-height: 1.9;
            word-break: keep-all;
            line-break: strict;
            overflow-wrap: break-word;
            margin: 0;
            padding: 40px 20px;
        }
        .passage {
            max-width: 720px;
            margin: 0 auto;
            background: white;
            padding: 40px 56px;
            border: 1px solid #e5e7eb;
            border-radius: 6px;
            font-size: 16px;
        }
        .passage p {
            margin: 0 0 1em 0;
            text-indent: 1em;
        }
        .passage .no-indent { text-indent: 0; }
        .marker { font-weight: bold; color: #1e40af; }
        .annotations {
            margin-top: 2em;
            padding-top: 1em;
            border-top: 1px dashed #d1d5db;
            font-size: 0.9em;
            color: #374151;
            line-height: 1.7;
        }
        .annotations p { margin: 0.3em 0; text-indent: 0; }
        .source {
            margin-top: 1.2em;
            text-align: right;
            font-size: 0.88em;
            color: #4b5563;
            text-indent: 0;
        }
        ruby { ruby-align: center; ruby-position: over; vertical-align: baseline; }
        ruby rt { font-size: 0.55em; color: #374151; letter-spacing: 0.02em; line-height: 1; vertical-align: top; }
        u { text-decoration: underline; text-decoration-thickness: 1.5px; }
    </style>
</head>
<body>
<div class="passage">
    {BODY_CONTENT}
</div>
</body>
</html>
```

### Key specs (MUST match exactly)

| Element | Value |
|---------|-------|
| Container width | `max-width: 720px` |
| Container margin | `margin: 0 auto` (căn giữa) |
| Container padding | `40px 56px` |
| Background body | `#f9fafb` (light gray) |
| Container background | `#fff` với `border: 1px solid #e5e7eb` |
| `word-break` | `keep-all` (xuống dòng ở ranh giới từ) |
| `text-align` | default (left) — KHÔNG justify |
| Tailwind CDN | ❌ KHÔNG dùng — CSS inline hết |
| Screenshot PNG | ❌ KHÔNG có |

### Template N1 — Essay/phê bình 3-5 paragraph, 2 câu hỏi, có blank + marker

```html
<div class="passage">
    <p>現代社会において、「幸福」という概念は個人の<ruby>主観<rt>しゅかん</rt></ruby>（注1）に
    大きく左右される。ある者は物質的な豊かさを、ある者は精神的な充足を幸福と呼ぶ。しかし、
    両者に共通するのは、自らの価値観を明確に持っているという点であろう。</p>
    <p>心理学者の研究によれば、<span class="marker">①</span><u>他者の基準に従って生きる人々</u>は、
    一時的に満たされたように見えても、長期的には<ruby>虚無感<rt>きょむかん</rt></ruby>に
    苛まれることが多いという。</p>
    <p>つまり、真の幸福とは、外部から与えられるものではなく、[ ② ]を通じて内側から
    築き上げるものなのである。</p>
    <div class="annotations">
        <p>注1 主観：個人の内面から見た見方・感じ方</p>
    </div>
    <p class="source">（山田太郎「幸福論の現在」文化新聞による）</p>
</div>
```

**Question 1** (`question_reference`): 「①他者の基準に従って生きる人々」とあるが、どのような人々か。
**Question 2** (`question_fill_in_the_blank`): [ ② ] に入る最も適当なものを選びなさい。

### Template N3 — Essay nhẹ 3-4 paragraph, 3 câu hỏi, có marker ①②

```html
<div class="passage">
    <p><ruby>節約<rt>せつやく</rt></ruby>は、ただお金を使わないことだと思う人が多い。
    しかし、本当の節約とは、<span class="marker">①</span><u>必要なものと不要なものを区別する力</u>
    のことである。</p>
    <p>例えば、安いからといって使わない服をたくさん買うのは、節約ではなく無駄づかいだ。
    一方、少し高くても長く使える物を選ぶのは、結果として節約になる。</p>
    <p>私が<span class="marker">②</span><u>この考え方</u>を学んだのは、祖母からだった。
    祖母は物を大切にし、一つの道具を何十年も使い続けた。</p>
    <p>節約は生活の質を下げることではない。むしろ、大切なものを見極める知恵なのである。</p>
</div>
```

**Question 1** (`question_reference`): 「①必要なものと不要なものを区別する力」とは、どのような力か。
**Question 2** (`question_reference`): 「②この考え方」とあるが、どのような考え方か。
**Question 3** (`question_content_match`): この文章の内容と合っているものはどれか。

### Template N5 — Diary/letter format có `<br>` (ngoại lệ)

```html
<div class="passage">
    <p class="no-indent">きょうは　日よう日です。<br>
    あさ、わたしは　こうえんへ　さんぽに　いきました。<br>
    こうえんには　たくさんの　人が　いました。<br>
    子どもたちは　あそんでいました。<br>
    おとなは　ベンチに　すわっていました。</p>
    <p class="no-indent">さんぽの　あとで、カフェで　コーヒーを　のみました。<br>
    コーヒーは　あたたかくて　おいしかったです。<br>
    たのしい　一日でした。</p>
</div>
```

Chi tiết template per format xem `references/html-patterns.md`.

---

## R10. Clean HTML

### Clean HTML (`text_read`)

Strip all attributes, classes, and excess whitespace cho CSV column `text_read`. Clean HTML **GIỮ** `<ruby>` tag nhưng **BỎ** nội dung `<rt>` — nghĩa là chỉ có kanji gốc + okurigana, không có furigana trong CSV.

```python
class CleanHTMLExtractor(HTMLParser):
    SKIP_TAGS = ('style', 'script', 'rt')   # bỏ furigana trong CSV text_read
    def __init__(self):
        super().__init__()
        self.result, self.skip_depth = [], 0
        self.in_body, self.body_done = False, False
    def handle_starttag(self, tag, attrs):
        if tag == 'body': self.in_body = True; return
        if not self.in_body or self.body_done: return
        if tag in self.SKIP_TAGS: self.skip_depth += 1; return
        if self.skip_depth > 0: return
        self.result.append(f'<{tag}>')
    def handle_endtag(self, tag):
        if tag == 'body': self.body_done = True; return
        if not self.in_body or self.body_done: return
        if tag in self.SKIP_TAGS: self.skip_depth -= 1; return
        if self.skip_depth > 0: return
        self.result.append(f'</{tag}>')
    def handle_data(self, data):
        if not self.in_body or self.body_done or self.skip_depth > 0: return
        self.result.append(data)

def clean_html(full_html):
    ext = CleanHTMLExtractor()
    ext.feed(full_html)
    raw = ''.join(ext.result)
    raw = re.sub(r'\s+', ' ', raw)
    raw = re.sub(r'\s*<', '<', raw)
    raw = re.sub(r'>\s*', '>', raw)
    raw = re.sub(r'<(\w+)></\1>', '', raw)
    return raw.strip()
```

### Character count (`count_body_chars()`)

```python
from html.parser import HTMLParser
import re

class BodyTextExtractor(HTMLParser):
    def __init__(self):
        super().__init__()
        self.texts, self.skip_depth, self.in_body = [], 0, False
    def handle_starttag(self, tag, attrs):
        if tag == 'body': self.in_body = True
        if tag in ('rt', 'style', 'script'): self.skip_depth += 1
    def handle_endtag(self, tag):
        if tag in ('rt', 'style', 'script'): self.skip_depth -= 1
    def handle_data(self, d):
        if self.in_body and self.skip_depth == 0: self.texts.append(d)

def count_body_chars(html_string):
    ext = BodyTextExtractor()
    ext.feed(html_string)
    return len(re.sub(r'[ \t\n\r\u3000]', '', ''.join(ext.texts)))
```

Rules:
- Count từ **full HTML file**, KHÔNG phải clean HTML.
- Skip `<rt>` (furigana), `<style>`, `<script>` content.
- Remove all whitespace: space, tab, newline, full-width space (`　`).
- Numbers, punctuation, Latin chars ALL count.

### ⛔ KHÔNG có screenshot

Đoạn văn vừa KHÔNG chụp PNG. CSV column `general_image` LUÔN empty string (`""`). Không cần Playwright / viewport / `screenshot.py` — skill này KHÔNG bundle screenshot script.

---

## R11. CSV Schema & File Naming

45 columns matching `rules/question_sheet.csv`:

| Column | Value cho đoạn văn vừa |
|--------|------------------------|
| `_id` | `{LEVEL}_{uuid.uuid4().hex}` — 32-char hex |
| `level` | `N1`, `N2`, `N3`, `N4`, `N5` |
| `tag` | Topic tiếng Việt từ `rules/topic.json` (VD: `tâm lý học`, `kinh tế`, `văn hóa`) |
| `jp_char_count` | Result of `count_body_chars()` |
| `kind` | **Always `đoạn văn vừa`** |
| `general_audio` | `""` (empty) |
| `general_image` | **`""` (empty — KHÔNG có PNG)** |
| `text_read` | Clean HTML (no attributes, no `<rt>` content, collapsed whitespace) |
| `text_read_vn` | `""` (empty) |
| `text_read_en` | `""` (empty) |

### Câu hỏi — số lượng tùy level

| Level | Cần populate | Cần empty |
|-------|--------------|-----------|
| **N1** | `question_{1,2}` | `question_{3,4,5}` |
| **N2** | `question_{1,2}` | `question_{3,4,5}` |
| **N3** | `question_{1,2,3}` | `question_{4,5}` |
| **N4** | `question_{1,2,3}` | `question_{4,5}` |
| **N5** | `question_{1,2}` | `question_{3,4,5}` |

Với mỗi slot `i` được populate:

| Field | Value |
|-------|-------|
| `question_label_i` | Một trong 7 labels (xem `rules/questions.md` R5) |
| `question_i` | Câu hỏi tiếng Nhật |
| `question_image_i` | `""` (empty) |
| `answer_i` | 4 options ngăn cách `\n`, KHÔNG số thứ tự |
| `correct_answer_i` | Integer `1`-`4` |
| `explain_vn_i` | Giải thích VN 3 phần (xem `rules/questions.md` R6) |
| `explain_en_i` | Giải thích EN 3 phần (cùng nội dung với VN) |

Với mỗi slot `j` KHÔNG dùng (j > n): tất cả 7 cột `question_label_j`, `question_j`, `question_image_j`, `answer_j`, `correct_answer_j`, `explain_vn_j`, `explain_en_j` = `""`.

### File Naming

All files và CSV `_id` column dùng cùng ID: `{LEVEL}_{uuid}`

- **Pattern**: `{LEVEL}_{uuid}.html` — e.g. `N3_a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5.html`
- **Level prefix UPPERCASE**: `N1`, `N2`, `N3`, `N4`, `N5`
- **UUID**: 32-char hex từ `uuid.uuid4().hex` (full, không cắt)
- **_id in CSV** = cùng value = filename without extension

```python
import uuid
def gen_id(level: str) -> str:
    return f"{level}_{uuid.uuid4().hex}"
```

### ⛔ KHÔNG BAO GIỜ sửa CSV bằng tay

> **Nội dung câu hỏi thường chứa commas (ví dụ `100,000円`, `それは、そうだ`) + newlines trong `answer_i` → sẽ vỡ cột CSV khi mở bằng text editor thô.**
> **LUÔN dùng `scripts/fill_qa.py`** để điền Q&A — script tự quote đúng mọi trường hợp, tự validate label có `question_` prefix.
> Hoặc dùng `scripts/process_html.py` với `--questions-json` hoặc `--q1`/`--q2`/... args.

---

## QC Automation Scripts

```python
import re
from pathlib import Path

def check_html(html_path: str, level: str) -> dict:
    """Kiểm tra tự động các tiêu chí đo được cho đoạn văn vừa."""
    html = Path(html_path).read_text(encoding='utf-8')
    results = {}

    # TC1: Character count (min AND max)
    char_count = count_body_chars(html)
    char_range = {
        "N1": (550, 620), "N2": (530, 610), "N3": (380, 500),
        "N4": (490, 610), "N5": (270, 310)
    }
    hard_reject = {"N1": 500, "N2": 500, "N3": 350, "N4": 450, "N5": 250}
    lo, hi = char_range[level]
    results["TC1_chars"] = {
        "count": char_count, "range": f"{lo}-{hi}",
        "pass": lo <= char_count <= hi + 30,
        "hard_reject": char_count < hard_reject[level]
    }

    # TC2: Flow text (no 。<br>) — ngoại lệ N5 letter
    br_in_prose = len(re.findall(r'。\s*<br\s*/?>', html))
    results["TC2_flow_text"] = {"br_in_prose": br_in_prose, "pass": br_in_prose == 0 or level == "N5"}

    # TC3: Container CSS (720px cho đoạn văn vừa)
    has_max_width = bool(re.search(r'max-width:\s*720px', html))
    has_margin_auto = bool(re.search(r'margin:\s*0\s+auto', html))
    results["TC3_container"] = {"pass": has_max_width and has_margin_auto}

    # TC4a: Ruby count per level
    ruby_count = len(re.findall(r'<ruby>', html))
    ruby_max = {"N1": 4, "N2": 8, "N3": 10, "N4": 6, "N5": 3}
    results["TC4_ruby_count"] = {"count": ruby_count, "max": ruby_max[level],
                                 "pass": ruby_count <= ruby_max[level] + 2}

    # TC4b: Wrong furigana format (check parentheses)
    paren_furigana = re.findall(r'[\u4e00-\u9fff]+[(][ぁ-ん]+[)]', html)
    bracket_furigana = re.findall(r'[\u4e00-\u9fff]+【[ぁ-ん]+】', html)
    results["TC4_furigana_format"] = {
        "paren_found": paren_furigana,
        "bracket_found": bracket_furigana,
        "pass": len(paren_furigana) == 0 and len(bracket_furigana) == 0
    }

    # TC4c: Ruby without rt
    ruby_blocks = re.findall(r'<ruby>(.*?)</ruby>', html, re.DOTALL)
    ruby_without_rt = [b for b in ruby_blocks if '<rt>' not in b]
    results["TC4_ruby_has_rt"] = {
        "missing_rt": ruby_without_rt,
        "pass": len(ruby_without_rt) == 0
    }

    # TC5: Marker consistency — nếu có marker ①②③ thì phải có câu hỏi reference tương ứng
    markers = re.findall(r'<span class="marker">([①②③④])</span>', html)
    results["TC5_markers"] = {"count": len(markers), "markers": markers}

    return results


def check_csv_row(csv_row: dict, level: str) -> dict:
    """Kiểm tra row CSV cho đoạn văn vừa — số câu hỏi đúng level."""
    expected_q = {"N1": 2, "N2": 2, "N3": 3, "N4": 3, "N5": 2}[level]
    filled = sum(1 for i in range(1, 6) if csv_row.get(f"question_{i}", "").strip())
    empty_beyond = all(
        csv_row.get(f"question_{i}", "") == ""
        for i in range(expected_q + 1, 6)
    )
    return {
        "question_count": filled,
        "expected": expected_q,
        "count_pass": filled == expected_q,
        "empty_beyond_pass": empty_beyond,
        "kind_pass": csv_row.get("kind") == "đoạn văn vừa",
        "general_image_pass": csv_row.get("general_image") == "",
    }
```

---

## Bundled Scripts

```bash
# Đếm ký tự:
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py --count-only --file <html-file>

# Validate batch (Target Range + broken ruby):
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_vua

# Full pipeline qua JSON (RECOMMENDED cho multi-question):
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
    --file <html-file> \
    --csv sheets/samples_v1.csv \
    --tag "kinh tế" \
    --questions-json /tmp/qs.json

# Full pipeline qua CLI flags (phù hợp cho 2 câu N1/N2/N5):
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
    --file <html-file> \
    --csv sheets/samples_v1.csv \
    --tag "tâm lý học" \
    --q1-label question_reference --q1 "..." --a1 "A|B|C|D" --c1 2 --ev1 "..." --ee1 "..." \
    --q2-label question_reason_explanation --q2 "..." --a2 "A|B|C|D" --c2 3 --ev2 "..." --ee2 "..."

# Safe Q&A filling (khuyến khích cho agent — tránh vỡ CSV):
python3 .claude/skills/jlpt-reading-medium-passage/scripts/fill_qa.py \
    --csv sheets/samples_v1.csv \
    --row-id N3_abcdef... \
    --level N3 \
    --q1-label question_reference \
    --q1 "..." \
    --a1 "Option 1
Option 2
Option 3
Option 4" \
    --ca1 2 \
    --evn1 "..." \
    --een1 "..." \
    --q2-label question_reason_explanation \
    --q2 "..." \
    --a2 "..." \
    --ca2 3 \
    --evn2 "..." \
    --een2 "..." \
    --q3-label question_content_match \
    --q3 "..." \
    --a3 "..." \
    --ca3 1 \
    --evn3 "..." \
    --een3 "..."

# Refresh CSV sau khi sửa HTML (giữ Q&A):
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py \
    --refresh \
    --file assets/html/doan_van_vua/{LEVEL}_{uuid}.html \
    --csv sheets/samples_v1.csv

# Load reference samples để calibrate style:
python3 .claude/skills/jlpt-reading-medium-passage/scripts/load_references.py --level N3 --count 3
```
