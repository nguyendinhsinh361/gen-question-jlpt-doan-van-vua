# PROMPTS.md — Đoạn Văn Vừa

Prompt templates để gọi LLM gen content cho skill `jlpt-reading-medium-passage` (中文読解).

Khác Phase 1 (đoạn văn ngắn, 1 câu hỏi/bài), phase này yêu cầu **2-3 câu hỏi/bài** tuỳ level.

Cách dùng: copy prompt theo level, thay các `{PLACEHOLDER}` bằng giá trị thực tế, feed vào Claude/Gemini.

## 0. Common System Prompt (tất cả level)

```
Bạn là trợ lý chuyên gen dữ liệu JLPT 中文読解 (đoạn văn vừa).

Rule BẤT BIẾN:
1. Gen đoạn văn tiếng Nhật đúng Target Range của level (đếm ký tự không whitespace,
   không tính <rt>, <style>, <script>).
2. HTML template: <!DOCTYPE html>, Noto Sans JP qua Google Fonts,
   <div class="passage"> max-width 720px, chứa <p> với text-indent 1em.
   KHÔNG dùng <br> giữa câu (trừ N5 letter format).
3. Furigana chỉ cho từ VƯỢT level. Cấm dạng "Ab" (nửa kanji nửa hiragana).
4. Mỗi bài có NHIỀU câu hỏi theo spec:
   - N1: 2 câu
   - N2: 2 câu
   - N3: 3 câu
   - N4: 3 câu
   - N5: 2 câu
5. Trong 1 bài, ≥ 2 question_label khác nhau. Không lặp cùng 1 label cho tất cả câu.
6. Các câu hỏi trong 1 bài test các ĐOẠN/CỤM TỪ KHÁC NHAU. Không 2 câu cùng test 1 chỗ.
7. Nếu dùng marker ①②③ cho câu reference, MARKER phải xuất hiện trong HTML
   ngay trước cụm từ được hỏi, và câu hỏi dẫn bằng「①cụm từ」とあるが。
8. Distractor plausible, không có 2 đáp án cùng đúng. Đáp án answer được từ trong bài.
9. Giải thích VN + EN cho TỪNG CÂU (tại sao đáp án đúng + tại sao 3 đáp án kia sai).
10. Tên tác giả trong source line PHẢI tự chế, KHÔNG dùng tác giả có thật.

Output format: JSON với các field:
{
  "id": "{LEVEL}_{uuid32hex}",
  "level": "N3",
  "tag": "thí nghiệm",
  "html": "<!DOCTYPE html>...",
  "questions": [
    {
      "label": "question_reference",
      "question": "...",
      "answers": ["A", "B", "C", "D"],
      "correct": 2,
      "explain_vn": "...",
      "explain_en": "..."
    },
    {
      "label": "question_reason_explanation",
      "question": "...",
      "answers": ["A", "B", "C", "D"],
      "correct": 3,
      "explain_vn": "...",
      "explain_en": "..."
    }
  ]
}
```

## 1. Prompt N1 — Formal essay/phê bình + fill_in_the_blank

```
Gen 1 bài JLPT N1 đoạn văn vừa về chủ đề {TOPIC} (ví dụ: triết học ngôn ngữ,
phê bình văn hóa, xã hội hiện đại, email công việc formal).

Yêu cầu cụ thể:
- Độ dài: 550–620 ký tự (count không whitespace, không <rt>)
- Văn phong: rất formal (keigo, ～いかんによらず, ～をもって, ～に先立ち, ～をものともせず)
- Cấu trúc: 3-4 paragraph (KHÔNG dùng <br> giữa câu)
- Furigana: tối đa 4 cặp <ruby>/<rt> — chỉ cho từ vượt N1
- Khuyến nghị thêm <div class="annotations">注1 ...：...</div> (data N1 có 34%)
- Source line optional (data 13%)

BẮT BUỘC: 2 câu hỏi với ≥ 2 labels khác nhau.

**95% data N1 có fill_in_the_blank** — nên gen ít nhất 1 câu fill_in_the_blank.

Combo câu hỏi đề xuất (chọn 1):
  A. 1 fill_in_the_blank + 1 reason_explanation (phổ biến nhất)
  B. 1 fill_in_the_blank + 1 author_opinion
  C. 1 fill_in_the_blank + 1 meaning_interpretation
  D. 1 reference (marker ①) + 1 reason_explanation

Fill_in_the_blank pattern:
  HTML: <p>...[ ① ]...</p>
  Question: [ ① ]に入る最も適当なものはどれか。
  Đáp án: 4 cụm từ tiếng Nhật (1 cụm kanji/vocab phù hợp ngữ cảnh)

Reference pattern:
  HTML: <p>...<span class="marker">①</span><u>それ</u>...</p>
  Question: 「①それ」とあるが、何を指すか。

Reason pattern:
  Question: 筆者が...と考える最も大きな理由はどれか。

Author opinion pattern:
  Question: 筆者の主張として最も適切なものはどれか。

4 đáp án bắt buộc plausible — N1 yêu cầu distractor tinh vi (đúng 2/3 ý, sai 1 ý khó phát hiện).

Output JSON theo Common System Prompt, mảng "questions" có 2 phần tử.
```

## 2. Prompt N2 — Tiểu luận, marker + annotation nhiều

```
Gen 1 bài JLPT N2 đoạn văn vừa về chủ đề {TOPIC} (ví dụ: 年齢, 家事, 礼儀作法,
phân tích xã hội, email công việc, văn hóa).

Yêu cầu cụ thể:
- Độ dài: 530–610 ký tự
- Văn phong: formal, văn viết (～に伴い, ～に基づき, ～を踏まえて, ～に限り, ～とはいえ)
- Cấu trúc: 3-4 paragraph (KHÔNG dùng <br>)
- Furigana: tối đa 6 cặp — chỉ cho từ N1
- Marker ①②③④ phổ biến (data 47%) — nên có ít nhất 1 marker
- Annotation 注 khuyến nghị (data 43%) — nên thêm 1-2 chú thích
- Source line optional (data 20%, hơi cao hơn các level khác)

BẮT BUỘC: 2 câu hỏi với ≥ 2 labels khác nhau.

Combo đề xuất:
  A. 1 reference (marker ①) + 1 reason_explanation
  B. 1 reference (marker ①) + 1 content_match
  C. 1 reason_explanation + 1 author_opinion
  D. 1 reference (marker ①) + 1 meaning_interpretation

Reference with marker:
  HTML: <p><span class="marker">①</span><u>古典的な勘違い</u>の最たるものは、...</p>
  Question: ①「古典的な勘違い」とあるが、どういうことか。

Reason pattern:
  Question: 筆者はなぜ...と述べているのか。

Content_match:
  Question: 本文の内容と合うものはどれか。

4 đáp án bắt buộc plausible — N2 yêu cầu distractor sai ở 1 giải thích/nuance, không hiển nhiên.

Output JSON với 2 questions.
```

## 3. Prompt N3 — Thí nghiệm/thư từ/giai thoại, marker + annotation cao

```
Gen 1 bài JLPT N3 đoạn văn vừa về chủ đề {TOPIC} (ví dụ: thí nghiệm động vật,
thí nghiệm thị giác, thư từ có nội dung, giai thoại xã hội, bài báo ngắn).

Yêu cầu cụ thể:
- Độ dài: 380–500 ký tự
- Văn phong: nửa formal nửa conversational (～について, ～によって, ～場合は, ～ために)
- Cấu trúc: 3-5 paragraph (KHÔNG <br>)
- Furigana: tối đa 10 cặp — cho từ N2+
- Marker ①②③ phổ biến (data 54%) — với 3 câu, nên có ≥ 1 marker
- Annotation 注 rất phổ biến (data 54%) — khuyến nghị 1-2 注

BẮT BUỘC: 3 câu hỏi với ≥ 2 labels khác nhau.

Combo đề xuất:
  A. 1 reference (marker ①) + 1 reason_explanation + 1 content_match
  B. 1 reference (marker ①) + 1 reference (marker ②) + 1 reason_explanation
  C. 1 meaning_interpretation + 1 reason_explanation + 1 content_match

Dual-marker example:
  HTML: ...<span class="marker">①</span><u>このような方法</u>...<span class="marker">②</span><u>その結果</u>...
  Q1: 「①このような方法」とあるが、どのような方法か。
  Q2: 「②その結果」とあるが、どのような結果か。

Ví dụ câu hỏi N3 phổ biến:
  - この手紙を書いた一番の目的は何か。
  - この手紙を書いた人が次のクラス会の幹事に選ばれたのはなぜか。
  - 筆者は実験の使ったハトにどのように絵を見せたのか。

4 đáp án bắt buộc plausible — N3 distractor sai ở 1 nuance (đảo ngược, nhân-quả, thời gian).

Output JSON với 3 questions.
```

## 4. Prompt N4 — Đời sống đơn giản, content_match nhiều

```
Gen 1 bài JLPT N4 đoạn văn vừa về chủ đề {TOPIC} (ví dụ: kỷ niệm đời sống,
夏休み, 買い物, thư cho bạn bè, câu chuyện đơn giản).

Yêu cầu cụ thể:
- Độ dài: 490–610 ký tự (N4 DÀI vì nhiều hiragana)
- Văn phong: lịch sự đơn giản (～ことができます, ～なければなりません, ～と思います, ～ので)
- Cấu trúc: 4-5 paragraph, câu ngắn (KHÔNG <br>)
- Furigana: tối đa 6 cặp — chỉ cho từ N3+
- KHÔNG annotation (data 6%)
- KHÔNG source line (data 0%)
- Marker ①②③ optional (data 17%) — có thể 0-1 marker

BẮT BUỘC: 3 câu hỏi với ≥ 2 labels khác nhau.

Combo đề xuất:
  A. 2 content_match + 1 reference
  B. 1 content_match + 1 reason_explanation + 1 content_match
  C. 2 content_match + 1 content_mismatch (test hiểu ngược)

Content_match pattern:
  Question: 夏休みに何をしましたか。
  Question: みずうみで何をしましたか。
  Question: お店で何をしましたか。

Distribution: chọn 3 khía cạnh khác nhau của bài (mỗi paragraph 1 khía cạnh)
để hỏi 3 câu → mỗi câu test 1 đoạn/ý riêng.

4 đáp án — N4 distractor sai ở 1 chi tiết (thời gian, đối tượng, hành động).

Output JSON với 3 questions.
```

## 5. Prompt N5 — Thư dài/nhật ký, 2 câu content_match

```
Gen 1 bài JLPT N5 đoạn văn vừa về chủ đề {TOPIC} (ví dụ: thư dài cho bạn,
nhật ký gia đình, うみに行った, thông báo chi tiết, kỳ nghỉ đơn giản).

Yêu cầu cụ thể:
- Độ dài: 270–310 ký tự
- Văn phong: rất thân mật, nhiều hiragana, câu ngắn (～です/～ます, ～てから, ～たい)
- Có thể dùng full-width space 　 giữa các cụm
- Cấu trúc: 3-4 paragraph (hoặc letter format với <br>)
- Furigana: tối đa 3 cặp
- KHÔNG annotation, KHÔNG source line, KHÔNG marker (data 2%)

BẮT BUỘC: 2 câu hỏi với ≥ 2 labels khác nhau.

Combo đề xuất:
  A. 2 content_match (test 2 thông tin khác nhau trong bài) + ≥ 2 labels có thể
     dùng cùng label nếu test 2 cụm khác nhau... NHƯNG skill yêu cầu ≥ 2 labels,
     nên dùng combo B hoặc C:
  B. 1 content_match + 1 reason_explanation (ví dụ "なぜ...か。")
  C. 1 content_match + 1 reference (kiểu ア/イ/ウ label)

OPTION — Letter format (được phép <br>):
  <div class="passage">
    <p class="no-indent">「ヤンさんへ<br>
    せんしゅうの　にちようび、かぞくで　うみに　いきました。<br>
    ...<br>
    　　　大川　ひろし」</p>
  </div>

OPTION — Multi-paragraph narrative:
  <div class="passage">
    <p>せんしゅうの　にちようび、かぞくで　うみに　いきました。</p>
    <p>うみは　とても　きれいでした。わたしは　およぎました。</p>
    <p>いもうとは　すなで　あそびました。おとうとは　おさかなを　とりました。</p>
    <p>おひる、みんなで　おべんとうを　たべました。とても　たのしかったです。</p>
  </div>

Câu hỏi phổ biến:
  大川さんは　うみで　なにを　しましたか。
  わたしは　どうして　たのしかったですか。

4 đáp án — N5 distractor sai rõ (thông tin ngược hoặc không có trong bài).

Output JSON với 2 questions.
```

## 6. Batch Prompt — Gen nhiều bài 1 lần

```
Gen {N} bài JLPT {LEVEL} đoạn văn vừa. Yêu cầu đa dạng:

1. Topic: chọn từ {N_TOPIC} nhóm khác nhau:
   - Đời sống (nhật ký, thư từ, kỷ niệm, gia đình)
   - Xã hội (văn hóa, truyền thống, môi trường, ngôn ngữ)
   - Công việc (email công việc, nơi làm việc)
   - Giáo dục (trường học, học tập, nghiên cứu)
   - Khoa học (thí nghiệm, công nghệ, y học)
   - Kinh tế (tiêu dùng, thương mại)
   - Văn học (tiểu luận, phê bình, triết học) — chỉ N1/N2

2. Số câu hỏi per bài: BẮT BUỘC đúng spec
   - N1: 2 câu  | N2: 2 câu  | N3: 3 câu  | N4: 3 câu  | N5: 2 câu

3. question_label: Mỗi bài ≥ 2 labels khác nhau.
   Batch-level: ≥ 3 labels khác nhau trong cả batch.
   Distribution đề xuất theo level (xem PROMPTS.md section {LEVEL}).

4. Mỗi bài có _id riêng: {LEVEL}_{uuid32hex}.

5. Độ dài nằm trong Target Range.

6. Các câu hỏi trong cùng bài phải test các ĐOẠN/CỤM TỪ khác nhau.

Batch size khuyến nghị: 5-8 bài/lần (mỗi bài 2-3 câu hỏi, workload cao hơn đoạn văn ngắn).

Output: array của {N} JSON objects theo Common System Prompt.
```

## 7. Fix Prompt — Khi bài fail validate

### Case: UNDER_TARGET

```
Bài {ID} có {CHARS} ký tự, dưới Target Range của {LEVEL} ({LO}-{HI}).

Bổ sung thêm {NEEDED} ký tự bằng một trong các cách (theo thứ tự ưu tiên):
1. Thêm 1 câu văn vào paragraph giữa (mở rộng ý, không đưa ý mới)
2. Thêm 1 paragraph mới ở giữa bài (nối ý đang phát triển)
3. Thêm 1 chú thích 注1/注2 ở <div class="annotations"> (N1-N3)
4. Mở rộng 1 option đáp án

KHÔNG thêm paragraph có ý trái ngược — phải flow nối tiếp.
Giữ nguyên: _id, tất cả questions, đáp án đúng.

Output: JSON object mới (cùng schema, cùng _id, cùng questions), đã chỉnh độ dài.
```

### Case: HARD_REJECT

```
Bài {ID} có {CHARS} ký tự, DƯỚI Hard Reject của {LEVEL} ({THRESHOLD}).
KHÔNG chỉnh sửa — GEN LẠI TỪ ĐẦU.

Yêu cầu: {LEVEL} Target Range {LO}-{HI} ký tự.
Topic: {TOPIC}.
Số câu: {Q_COUNT} (spec).
Labels đề xuất: {LABEL_COMBO}.

Output: JSON object mới hoàn toàn.
```

### Case: Sai số câu hỏi (warning)

```
Bài {ID} đang có {ACTUAL_Q} câu hỏi, nhưng {LEVEL} yêu cầu {EXPECTED_Q} câu.

Fix:
- Nếu dư câu: bỏ câu kém nhất (test cùng đoạn với câu khác).
- Nếu thiếu câu: thêm câu test đoạn CHƯA được hỏi, với label khác các câu hiện có.

Giữ nguyên: _id, HTML, đáp án các câu còn lại.

Output: JSON object mới, "questions" có đúng {EXPECTED_Q} phần tử.
```

### Case: 2 câu cùng label (warning)

```
Bài {ID} có {ACTUAL_Q} câu hỏi nhưng tất cả cùng label = {LABEL}.
Skill yêu cầu ≥ 2 labels khác nhau.

Fix: thay 1 câu bằng label khác, test 1 đoạn chưa hỏi trong bài.
Label thay thế đề xuất: {SUGGESTED_LABEL}.

Giữ nguyên: _id, HTML, các câu hỏi còn lại.

Output: JSON object mới.
```

### Case: Dạng "Ab"

```
Bài {ID} có dạng "Ab" (nửa kanji nửa hiragana) sai quy tắc furigana.

Cụm vi phạm: "{VIOLATION}" (ví dụ: 週かん, 友だち)

Fix: chọn 1 trong 2 cách:
1. Full kanji + <ruby>: <ruby>週間<rt>しゅうかん</rt></ruby>
2. Full hiragana: しゅうかん

Giữ nguyên phần còn lại, chỉ sửa cụm vi phạm.

Output: JSON object mới (cùng _id, cùng questions, HTML đã fix).
```

### Case: Marker không match câu hỏi

```
Bài {ID} có câu hỏi「{Q_NUM}」tham chiếu marker {MARKER} nhưng HTML không có marker đó.

Fix: thêm marker vào HTML:
  <span class="marker">{MARKER}</span><u>{PHRASE}</u>
ngay trước hoặc bao quanh cụm từ được hỏi.

Hoặc: đổi câu hỏi không dùng marker, chỉ quote cụm từ:
  「{PHRASE}」とあるが、〜

Giữ nguyên các phần còn lại.

Output: JSON object mới.
```

## 8. Quality Check Prompt (gọi sau batch)

```
Kiểm tra chất lượng batch {N} bài JLPT {LEVEL} đoạn văn vừa. Cho mỗi bài, đánh giá:

1. Độ dài (target {LO}-{HI}):                PASS / FAIL ({CHARS} chars)
2. Số câu hỏi = {EXPECTED_Q}:                 PASS / FAIL ({ACTUAL_Q})
3. ≥ 2 labels khác nhau trong bài:            PASS / FAIL
4. Các câu hỏi test đoạn KHÁC NHAU:           PASS / FAIL
5. Marker ①②③ match câu hỏi (nếu có):         PASS / FAIL
6. Furigana đúng quy tắc (không "Ab"):        PASS / FAIL
7. Mỗi câu chỉ 1 đáp án đúng:                 PASS / FAIL
8. Distractor plausible level-phù hợp:        PASS / FAIL
9. Câu hỏi answer được từ trong bài:          PASS / FAIL
10. explain_vn + explain_en non-empty:         PASS / FAIL

Batch-level:
- ≥ 3 question_label khác nhau trong batch?    YES / NO
- ≥ 3 tag (topic) khác nhau?                   YES / NO
- Tất cả _id unique?                           YES / NO
- {N} bài, tổng {N}×{EXPECTED_Q} = {TOTAL_Q} câu hỏi — đúng?   YES / NO

Output: bảng markdown 1 row per bài + summary batch-level.
```

## 9. Variables reference

| Placeholder | Giá trị mẫu |
|-------------|-------------|
| `{LEVEL}`   | N1, N2, N3, N4, N5 |
| `{TOPIC}`   | thí nghiệm, phê bình văn hóa, nhật ký, ... |
| `{N}`       | Số bài (thường 5, 8) |
| `{N_TOPIC}` | Số nhóm topic (thường 3) |
| `{LO}, {HI}` | Target Range |
| `{THRESHOLD}` | Hard Reject |
| `{CHARS}`   | Char count thực tế |
| `{NEEDED}`  | Số ký tự cần bổ sung |
| `{ID}`      | `{LEVEL}_{uuid32hex}` |
| `{VIOLATION}` | Cụm vi phạm rule |
| `{Q_COUNT}` / `{EXPECTED_Q}` | Số câu hỏi spec (2 hoặc 3) |
| `{ACTUAL_Q}` | Số câu thực tế (để fix warning) |
| `{LABEL_COMBO}` | Combo labels đề xuất (xem section level) |
| `{SUGGESTED_LABEL}` | Label thay thế (khi fix duplicate) |
| `{Q_NUM}` | 1, 2, 3 (số thứ tự câu hỏi) |
| `{MARKER}` | ①, ②, ③, ④ |
| `{PHRASE}` | Cụm từ được gạch chân |
| `{TOTAL_Q}` | Tổng câu hỏi kỳ vọng cả batch |
