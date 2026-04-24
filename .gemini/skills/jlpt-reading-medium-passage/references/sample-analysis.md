# Sample Analysis — Đoạn Văn Vừa Reference Data

Phân tích định lượng dữ liệu mẫu thực tế ở `data/doan_van_vua_n{1-5}_clean.json` để gen agent biết **chính xác** mức độ dài, số câu hỏi, và pattern HTML cho từng level.

Số liệu chạy bằng `load_references.py --stats` + phân tích pattern.

## 1. Phân Bố Độ Dài (`jp_char_count`)

| Level | Samples | Min | P25 | P50 (median) | P75 | Avg | Max |
|-------|---------|-----|-----|--------------|-----|-----|-----|
| N1    | 91      | 522 | 556 | 589          | 624 | 596 | 772 |
| N2    | 87      | 502 | 534 | 569          | 612 | 606 | 1209|
| N3    | 74      | 355 | 384 | 436          | 502 | 468 | 984 |
| N4    | 29      | 457 | 495 | 541          | 613 | 556 | 819 |
| N5    | 34      | 254 | 276 | 286          | 301 | 294 | 378 |

**Kết luận**:
- **Target range** khuyến nghị = P25–P75 (± 10 chars)
- **Hard reject** = dưới Min thực tế
- **N4 dài hơn N3**: do N4 dùng nhiều hiragana, cùng nội dung nhưng nhiều ký tự hơn — đây là pattern đúng theo JLPT thật.
- N2 max 1209 là outlier (bài phê bình văn hoá dài bất thường) → skill chỉ cần đến 610.

## 2. Số Câu Hỏi Per Sample

| Level | 1q | 2q | 3q | 4q | 5q | Dominant |
|-------|----|----|----|----|----|----------|
| N1    | 0  | 27 (30%) | **64 (70%)** | 0  | 0  | 3 câu |
| N2    | 0  | 6 (7%)   | **76 (87%)** | 4 (5%) | 1 (1%) | 3 câu |
| N3    | 1 (1%) | 3 (4%) | **65 (88%)** | 5 (7%) | 0 | 3 câu |
| N4    | 0  | 2 (7%) | 8 (28%) | **19 (65%)** | 0 | 4 câu |
| N5    | 0  | **30 (88%)** | 2 (6%) | 2 (6%) | 0 | 2 câu |

**Mismatch giữa data & spec**:
- `rules/question_format.json` spec nói: N1=2, N2=2, N3=3, N4=3, N5=2
- Data thực tế cho thấy N1/N2 hay có 3 câu (legacy dataset noise)
- **Skill FOLLOW SPEC** — gen đúng số câu theo spec, không theo data

## 3. Pattern HTML Phổ Biến

### Pattern thô trong data gốc

| Pattern                  | N1    | N2    | N3    | N4    | N5    |
|--------------------------|-------|-------|-------|-------|-------|
| Có `<p>`                 | 100%  | 86%   | 72%   | 96%   | 94%   |
| Có `<br>`                | 21%   | 31%   | 35%   | 10%   | 14%   |
| Có `<ruby>`              | 0%    | 9%    | 13%   | 6%    | 2%    |
| Có `<span>`              | 5%    | 17%   | 22%   | 31%   | 5%    |
| Có `<u>` (underline)     | **72%** | **47%** | **33%** | 34% | 20% |
| Có 注 annotation         | 34%   | **43%** | **54%** | 6%    | 0%    |
| Có source line           | 13%   | **20%** | 9%    | 0%    | 0%    |
| Có marker ①②③④           | 7%    | **47%** | **54%** | 17%   | 2%    |
| Có blank `[ ]`/(1)       | **95%** | 17% | 6%    | 0%    | 0%    |

### Nhận xét Quan Trọng

1. **`<u>` rất phổ biến ở N1-N3 (33-72%)** — do nhiều câu hỏi reference/meaning cần gạch chân cụm từ. Skill dùng `<u>` cho mọi câu hỏi reference.

2. **Marker ①②③ lên tới 47-54% ở N2/N3** — VÌ có 2-3 câu hỏi/bài, mỗi câu reference có 1 marker riêng.

3. **Annotation 注 lên tới 54% ở N3** — bài có nhiều thuật ngữ cần giải thích. **Khuyến khích** thêm 注 cho N1-N3.

4. **Source line ở N2 lên 20%** — tiểu luận formal hay có. Khuyến khích.

5. **Blank `[ ]` ở N1 lên 95%** — đặc trưng N1 đoạn văn vừa. ĐÂY là key insight — N1 gần như luôn có fill_in_the_blank question.

6. **N4/N5 không có annotation/source** (≤6% & 0%) — không phù hợp level thấp.

## 4. Question Label Distribution (heuristic từ keyword detection)

Heuristic không catch được hết (reference detection yếu), nhưng reason_explanation consistent:

| Level | Total Q | reason | reference | fill_blank | content_match | author_op | meaning | mismatch |
|-------|---------|--------|-----------|------------|---------------|-----------|---------|----------|
| N1    | 246     | 15%    | —         | 11%        | 2%            | 5%        | 12%     | 0%       |
| N2    | 261     | 17%    | 8%        | 3%         | 13%           | 3%        | 2%      | 0%       |
| N3    | 222     | 18%    | 10%       | 5%         | 8%            | 2%        | 1%      | 1%       |
| N4    | 104     | 17%    | 0%        | 0%         | 6%            | 1%        | 0%      | 2%       |
| N5    | 74      | 18%    | 0%        | 0%         | 1%            | 0%        | 0%      | 0%       |

(Các câu không match pattern nào được coi là "reference implicit" — khoảng 50-70% các câu còn lại.)

### Distribution khuyến nghị cho mỗi bài

**N1 (2 câu/bài)** — với 95% có blank:
- Combo 1: 1 fill_in_the_blank + 1 reason_explanation (phổ biến nhất)
- Combo 2: 1 fill_in_the_blank + 1 author_opinion
- Combo 3: 1 fill_in_the_blank + 1 meaning_interpretation
- Combo 4: 1 reference + 1 reason_explanation

**N2 (2 câu/bài)** — marker 47%, annotation 43%:
- Combo 1: 1 reference (có marker ①) + 1 reason_explanation
- Combo 2: 1 reference + 1 content_match
- Combo 3: 1 reason_explanation + 1 author_opinion
- Combo 4: 1 reference + 1 meaning_interpretation

**N3 (3 câu/bài)** — marker 54%, annotation 54%:
- Combo 1: 1 reference (marker ①) + 1 reason_explanation + 1 content_match
- Combo 2: 1 reference (marker ①) + 1 reference (marker ②) + 1 reason_explanation
- Combo 3: 1 meaning_interpretation + 1 reason_explanation + 1 content_match

**N4 (3 câu/bài)** — ít marker, ít annotation:
- Combo 1: 1 content_match + 1 content_match + 1 reference
- Combo 2: 1 content_match + 1 reason_explanation + 1 content_match
- Combo 3: 2 content_match + 1 content_mismatch (test hiểu ngược)

**N5 (2 câu/bài)** — rất simple:
- Combo 1: 2 content_match (test các thông tin khác nhau trong bài)
- Combo 2: 1 content_match + 1 reason_explanation (ví dụ "なぜ...か。")

## 5. Chủ Đề Phổ Biến (từ đọc sample)

| Level | Chủ đề hay gặp |
|-------|----------------|
| N1    | Triết học ngôn ngữ (俳句, 文化), phê bình xã hội, hệ thống tư tưởng, email công việc formal |
| N2    | Tiểu luận về đời sống (年齢, 家事), phê bình văn hóa (礼儀作法), phân tích xã hội |
| N3    | Thí nghiệm (動物, 視覚), thư từ có nội dung (クラス会), giai thoại, bài báo ngắn |
| N4    | Kỷ niệm đời sống (夏休み, 買い物), thư cho bạn bè, câu chuyện đơn giản |
| N5    | Thư dài, nhật ký gia đình (うみに行った), thông báo chi tiết |

## 6. Câu Hỏi Phổ Biến Theo Level

### N1 — Formal, logic chặt

- `（　）に入る語彙は何か。` — fill_in_the_blank (rất phổ biến, dùng ký hiệu `（　）` hoặc `[ ]`)
- `それは具体的に何をいうか。` — reference (implicit, không có marker)
- `筆者がこの文書を書いた最もの目的はどれか。` — author_opinion
- `「このような考え」とあるが、どういうものか。` — meaning/reference

### N2 — Đa dạng, marker rất phổ biến

- `①「重要なキーワードだ」とあるが、なぜキーワードなのか。` — reason (có marker)
- `②「自分の年齢を恥じたりしていない」とあるが、なぜ恥じたりしないのか。` — reason (có marker)
- `筆者の考え方として最も適切なものはどれか。` — author_opinion
- `本文の内容と合うものはどれか。` — content_match

### N3 — Reference + reason nhiều

- `この手紙を受け取る人はどんな人か。` — content_match
- `この手紙を書いた人が次のクラス会の幹事に選ばれたのはなぜか。` — reason
- `この手紙を書いた一番の目的は何か。` — author_opinion
- `筆者は実験の使ったハトにどのように絵を見せたのか。` — content_match

### N4 — Content match đa số

- `夏休みに何をしましたか。` — content_match
- `みずうみで何をしましたか。` — content_match
- `お店で何をしましたか。` — content_match

### N5 — Rất đơn giản, hay dùng ア/イ/ウ ký hiệu

- `ア... イ... ウ...` — multiple fill-in or reference với ký hiệu katakana
- `正しいものはどれか` — content_match

## 7. Rule Tóm Lược Cho Gen Agent

1. **Độ dài**: Target P25–P75
2. **Số câu hỏi**: Theo SPEC (N1/N2/N5=2, N3/N4=3) — KHÔNG theo data noise
3. **Trong 1 bài**: ≥ 2 `question_label` khác nhau
4. **Các câu hỏi KHÔNG test cùng 1 đoạn** — phải test các phần khác nhau của bài
5. **Marker ①②③** cho mỗi câu reference — đặt trong HTML trùng vị trí cụm từ được hỏi
6. **Annotation 注1 注2** rất khuyến khích ở N1-N3 (data 34-54%)
7. **Source line** khuyến khích ở N1-N3 (13-20%); **KHÔNG** cho N4/N5
8. **N1 = hầu như luôn có fill_in_the_blank** — 95% data có `[ ]`/`（　）`
9. **HTML**: 3-5 paragraph cho N1/N2/N3/N4, 2-4 paragraph cho N5

## 8. So Sánh Với Đoạn Văn Ngắn

| Aspect | Đoạn văn ngắn | Đoạn văn vừa |
|--------|--------------|--------------|
| Char count range | 80–290 | 250–620 |
| Questions | 1 | 2 hoặc 3 |
| Paragraph count | 1–3 | 3–5 |
| Container width CSS | 640px | 720px |
| Marker `①②③` use rate | 3–7% | 7–54% |
| Underline `<u>` use rate | 1–5% | 20–72% |
| Annotation use rate | 0–40% | 6–54% |
| Fill-in-blank rate | 1–26% | 0–95% (N1 key) |
| Complexity per bài | Low | Medium |
