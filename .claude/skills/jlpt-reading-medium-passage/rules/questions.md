# Rules: Câu hỏi & Đáp án (R5, R6)

> **Scope**: Đoạn văn vừa có **2-3 câu hỏi** per bài (N1/N2/N5 = 2 câu, N3/N4 = 3 câu). Mỗi câu hỏi cover MỘT đoạn/ý KHÁC NHAU của bài.

## R5. Câu hỏi (`question_label` + `question`)

### Nguyên tắc cốt lõi

- **100% căn cứ vào văn bản**: đáp án xác nhận rõ ràng bởi thông tin/ý trong bài. KHÔNG dựa kiến thức ngoài.
- **Không dùng ảnh trong câu hỏi** — `question_image_X` luôn empty.
- **2-3 câu hỏi KHÔNG trùng nhau**: mỗi câu test 1 đoạn/ý khác — không được 2 câu cùng hỏi 1 đoạn.

### 7 kiểu câu hỏi (`question_label`)

Labels từ `rules/mission.json` — **BẮT BUỘC dùng `question_` prefix**:

| `question_label`                   | Khi nào dùng | Keywords câu hỏi |
|------------------------------------|--------------|------------------|
| `question_content_match`           | Chọn câu phù hợp với nội dung | 最も合っているもの, 正しいもの, 本文の内容と合うもの |
| `question_reason_explanation`      | Hỏi lý do / nguyên nhân (nhân quả) | なぜ, どうして, ～のはなぜか, ～の理由は何か |
| `question_reference`               | Hỏi đại từ chỉ định hoặc cụm được gạch chân | 「①...」とあるが、何を指すか, これ/それ/その/この + どんなもの |
| `question_meaning_interpretation`  | Hỏi nghĩa của câu/cụm từ | どういうことか, どういう意味か, ～とはどういうことか |
| `question_content_mismatch`        | Chọn câu KHÔNG phù hợp | 合わないもの, 正しくないもの, 間違っているもの |
| `question_author_opinion`          | Hỏi quan điểm tác giả | 筆者の考え, 筆者は...について何と述べているか |
| `question_fill_in_the_blank`       | Điền từ vào ô trống | [ ① ] に入るもの, ( 1 ) に入る最も適当なもの |

### Distribution combo per level (từ data analysis)

Vì đoạn văn vừa có multi-question, mỗi level có combo ưu tiên:

| Level | Q Count | Combo khuyến nghị |
|-------|---------|-------------------|
| **N1** (95% có blank) | **2** | 1 `question_fill_in_the_blank` + 1 `question_reason_explanation` / `question_author_opinion` |
| **N2** (47% marker, 20% source) | **2** | 1 `question_reference` (có marker ①) + 1 `question_reason_explanation` hoặc `question_author_opinion` |
| **N3** (54% marker, 54% annotation) | **3** | 1 `question_reference` + 1 `question_reason_explanation` + 1 `question_content_match` |
| **N4** (17% marker) | **3** | 2 `question_content_match` + 1 `question_reference` hoặc `question_reason_explanation` |
| **N5** (2% marker, simple) | **2** | 2 `question_content_match`, HOẶC 1 `question_content_match` + 1 `question_reference` |

> **BẮT BUỘC**: Trong 1 bài, **KHÔNG dùng cùng 1 `question_label` cho tất cả câu**. Phải có ≥ 2 labels khác nhau.
> Ngoại lệ: N5 có thể 2 `question_content_match` (bài đơn giản).

### Marker — Câu hỏi khớp HTML

Nếu câu hỏi dùng `question_reference` với "「①...」とあるが":
- Trong HTML PHẢI có `<span class="marker">①</span><u>...</u>` đúng cụm được hỏi.
- Với bài N3 có 3 câu và 2 câu là `question_reference` → dùng `①` cho câu 1, `②` cho câu 2.

Nếu `question_fill_in_the_blank`:
- Trong HTML PHẢI có `[ ① ]` hoặc `( 1 )` đúng vị trí đang hỏi.

Nếu `question_meaning_interpretation`:
- Cụm từ hỏi nên có `<u>...</u>` (không bắt buộc marker ①② nếu chỉ 1 câu meaning).

### Paraphrasing test (N3+)

Đáp án đúng KHÔNG được copy-paste 1 cụm ≥ 4 từ liên tiếp từ bài.

**SAI**: Bài viết `「この考え方は昔から多くの哲学者や作家が語ってきた普遍的な価値観である」`, đáp án chỉ chép `「多くの哲学者や作家が語ってきた」`.

**ĐÚNG**: Paraphrase — `「長い間、多くの思想家や文学者が述べてきたもの」`.

N4/N5 có thể nhẹ hơn nhưng vẫn nên tránh copy > 6 từ liên tiếp.

### Câu hỏi phải độc lập

Mỗi câu hỏi đứng độc lập — không tham chiếu câu hỏi khác (không có `前の問題で言及した...`).

---

## R6. Đáp án (`answer_X`, `correct_answer_X`, `explain_vn_X`, `explain_en_X`)

### R6.1 Format 4 đáp án

```
answer_X = "Option A\nOption B\nOption C\nOption D"
```

- **Ngăn cách**: `\n` (newline) giữa các option
- **KHÔNG prefix** `1.`, `①`, `1)` — chỉ nội dung thuần
- **Đúng 4 option** per câu (không 3 không 5)
- Độ dài: tương đương nhau (không có 1 option ngắn bất thường)
- `correct_answer_X` = integer 1-4 (1-based)

**Trong CSV**, `answer_X` sẽ được lưu dưới dạng 1 cell đa dòng:

```csv
"選択肢1
選択肢2
選択肢3
選択肢4"
```

Dùng `fill_qa.py` để auto-quote.

### R6.2 Nguyên tắc đáp án

- **Đáp án đúng**: xác nhận rõ bởi thông tin/ý trong bài, PHẢI paraphrase (N3+).
- **Đáp án sai (distractor)**: PHẢI dùng **thông tin/ý THẬT từ bài** nhưng sai ngữ cảnh/hiểu lầm. **NGHIÊM CẤM bịa thông tin không có trong bài**.
- **Đa dạng vị trí đúng**: trong batch ≥ 10 bài, `correct_answer` phân bố đều 1/2/3/4 (không lệch quá 40% về 1 con số).

### R6.3 Các loại bẫy (distractor traps) — BẮT BUỘC đa dạng

Trong 4 đáp án (1 đúng + 3 sai), 3 sai PHẢI dùng ≥ 3 loại bẫy khác nhau:

| Loại bẫy | Mô tả | Ví dụ |
|----------|-------|-------|
| **① Reversal** | Đảo ngược logic/điều kiện/quan hệ nhân-quả từ bài | Bài nói "A dẫn đến B" → distractor nói "B dẫn đến A" |
| **② Detail swap** | Đổi chi tiết (subject/object/số lượng/thời gian) | Bài: "筆者は日本料理を好む" → distractor: "筆者は中国料理を好む" |
| **③ Scope** | Mở rộng/thu hẹp phạm vi | Bài: "ある場合" → distractor: "どんな場合も" |
| **④ Misinterpretation** | Hiểu sai nghĩa từ/cụm từ chỉ định | `この考え方` ám chỉ X, distractor diễn giải là Y |
| **⑤ Part of truth** | Chỉ đúng 1 phần của bài, bỏ sót ý quan trọng | Bài có 2 luận điểm A+B → distractor chỉ nói A (thiếu B) |
| **⑥ Over-generalization** | Áp dụng cho mọi người/mọi trường hợp trong khi bài chỉ nói 1 đối tượng | Bài: "筆者は" → distractor: "日本人は全員" |

### R6.4 Self-test cho distractor (trước khi finalize)

Với mỗi distractor, tự hỏi:

1. **Textual evidence test**: Distractor này có dùng info/ý xuất hiện trong bài không?
   - KHÔNG → đang bịa → sửa
2. **Trap type test**: Distractor này thuộc loại bẫy nào? Đã match 1 trong 6 loại chưa?
   - KHÔNG → đang random → sửa
3. **Plausibility test**: Người đọc nhanh có thể chọn nhầm đáp án này không?
   - KHÔNG (quá xa) → đáp án quá dễ loại → sửa cho gần hơn
   - CÓ NHƯNG đọc kỹ lại thấy sai → OK
4. **Refutation test**: Trích được câu cụ thể từ bài để bác bỏ distractor không?
   - KHÔNG → đang bịa → sửa
5. **Only-one-correct test**: Có thể có 2 đáp án cùng đúng không?
   - CÓ → sửa cho rõ chỉ 1 đáp án đúng

### R6.5 Format explanation (3 phần VN + EN)

Mỗi câu có `explain_vn_X` và `explain_en_X` viết theo cùng cấu trúc 3 phần:

**Phần 1 — Đáp án đúng**: Nêu đáp án đúng là số mấy + trích câu/đoạn trong bài xác nhận + giải thích paraphrase nếu có.

**Phần 2 — Đáp án sai**: Giải thích TẠI SAO từng đáp án sai. Nêu rõ loại bẫy (reversal / detail swap / scope / misinterpretation / part of truth / over-generalization) và chỉ ra info/ý trong bài khiến đáp án đó sai.

**Phần 3 — Tóm tắt**: 1 câu tóm tắt ý chính của câu hỏi này (helpful để AI học).

### R6.6 Ví dụ đáp án + explanation

**Bài N3** (giả tưởng): Tác giả viết về việc trẻ em thời nay không biết cảm ơn, vì được chiều từ nhỏ.

**Question 1** (`question_reason_explanation`):
> なぜ筆者は「今の子どもたちは感謝を知らない」と考えているか。

**Answers**:
```
小さい頃から何でも与えられすぎているから
学校で感謝の大切さを教えないから
親が忙しくて子どもと話す時間がないから
子どもたち自身の性格が変わったから
```

**correct_answer**: 1

**explain_vn**:
```
Đáp án đúng: 1. Bài viết: "小さい頃から親にすべてを与えられている子は、何かをもらうことを当たり前に感じる" — xác nhận lý do là được chiều từ nhỏ. Đáp án 1 paraphrase chính xác.
Đáp án sai:
- 2 (Scope): Bịa — bài không đề cập đến trường học.
- 3 (Part of truth): Bài có nhắc bố mẹ bận, nhưng đó không phải lý do chính theo tác giả.
- 4 (Over-generalization): Bài không nói tính cách trẻ em thay đổi, chỉ nói về cách nuôi dạy.
Tóm tắt: Câu hỏi test khả năng xác định nguyên nhân chính được tác giả đưa ra (không phải lý do phụ hoặc kiến thức ngoài bài).
```

**explain_en**:
```
Correct answer: 1. Article states: "Children who receive everything from parents since childhood come to take receiving as a matter of course" — confirms the reason is being spoiled from young. Option 1 is a precise paraphrase.
Wrong answers:
- 2 (Scope): Fabricated — article doesn't mention schools.
- 3 (Part of truth): Article mentions busy parents but not as the main reason author cites.
- 4 (Over-generalization): Article doesn't claim children's personalities have changed, only about how they're raised.
Summary: Question tests identifying the main cause explicitly given by the author (not secondary reasons or outside knowledge).
```

### R6.7 Đa câu trong 1 bài — Tránh trùng

**BẮT BUỘC**:
- Câu Q1 test đoạn/ý khác hẳn Q2 (và Q3 nếu có)
- Nếu Q1 hỏi paragraph 1-2 thì Q2 phải hỏi paragraph 3-5
- Không được 2 câu cùng test `この現象` (cùng reference)
- Không được 2 câu đều là `question_reason_explanation` test cùng 1 nguyên nhân
