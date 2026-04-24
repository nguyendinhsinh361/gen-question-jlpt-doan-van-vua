# Rules: Từ vựng, Ngữ pháp & Furigana (R3, R4)

## R3. Trình độ kiến thức (Kanji, Từ vựng, Ngữ pháp)

### Nguyên tắc tổng quát

| Level | Kanji/Từ vựng cốt lõi | Ngữ pháp cốt lõi |
|-------|----------------------|-------------------|
| N5    | ~100 kanji + ~800 từ | ～です/ます, ～て, ～たい, trợ từ cơ bản |
| N4    | ~300 kanji + ~1500 từ | ～ている, ～てしまう, ～ばいい, ～ようと思う |
| N3    | ~650 kanji + ~3700 từ | ～ばかり, ～わけではない, ～ため, ～によって, ～にとって |
| N2    | ~1000 kanji + ~6000 từ | ～に伴い, ～に基づく, ～を踏まえて, ～とはいえ, ～にもかかわらず |
| N1    | ~2000 kanji + ~10000 từ | ～いかんにかかわらず, ～をもって, ～に先立ち, keigo cao |

### Golden principle — "THAY TỪ, KHÔNG RẮC FURIGANA"

Nếu bài N3 dùng quá nhiều kanji N2 → **viết lại bằng từ N3**, không phải rắc furigana bừa bãi cho từ N2.

**Furigana dùng khi không có cách nào thay thế khác** — ví dụ tên riêng, từ chuyên môn (`哲学`, `遺伝子`) không có từ N3 tương đương.

### Phân loại từ trong bài

1. **Từ khóa của bài** (key terms): từ trung tâm mà câu hỏi xoay quanh — nên **ở đúng level bài**, KHÔNG cần furigana. VD: trong bài N3 về 節約 (tiết kiệm) → chính từ 節約 phải là N3, không rắc furigana.

2. **Từ ngữ cảnh** (context words): từ phụ hỗ trợ, hay xuất hiện. VD: 家族, 学校, 食べる, 買う → giữ đúng level.

3. **Từ chuyên môn / thuật ngữ** (jargon): từ không thể thay thế, vượt level → **có thể thêm furigana** hoặc **thêm 注 annotation** giải thích.

### Ruby count density per level

| Level | Above-level words | Ruby `<ruby>` expected |
|-------|-------------------|------------------------|
| N5    | 0–1               | 0–3                    |
| N4    | 0–3               | 0–6                    |
| N3    | 0–5               | 0–10                   |
| N2    | 0–4               | 0–8                    |
| N1    | 0–2               | 0–4                    |

> **Nguyên tắc**: ≥ 80% từ vựng/ngữ pháp phải ở đúng level. Ruby chỉ cho phần vượt level không thể tránh.

---

## R4. Furigana — Quy tắc & Kanji lookup

### R4.1 Compound Word Rule — CẤM dạng "Ab"

**LUÔN viết nguyên bộ kanji** rồi đặt furigana bao toàn bộ. **TUYỆT ĐỐI KHÔNG** tách nửa kanji nửa hiragana.

Chỉ chọn 1 trong 2:

1. **Full kanji + furigana**: `<ruby>週間<rt>しゅうかん</rt></ruby>`
2. **Full hiragana** (khi ở level thấp): `しゅうかん`

**❌ CẤM**: `週かん`, `友だち` (ở N5 nên viết `ともだち` hoặc `<ruby>友達<rt>ともだち</rt></ruby>`), `拠てん`, `経けん`

**✅ Ngoại lệ Okurigana**: `<ruby>届<rt>とど</rt></ruby>く` — furigana chỉ phủ kanji, okurigana đứng riêng ngoài ruby.

### R4.2 Furigana Lookup Procedure

Bước 1: **Xác định level kanji** bằng `rules/jlpt_kanji.csv` (2150 kanji, mapped từ N5→N1).

Bước 2: **Nếu kanji > level bài** → thêm furigana:

```html
<ruby>割引<rt>わりびき</rt></ruby>
```

Bước 3: **Nếu kanji ≤ level bài** → KHÔNG thêm furigana (kể cả bài gen cho AI sau này).

### R4.3 Ví dụ áp dụng

| Từ | Level kanji | Bài N3 | Bài N4 | Bài N5 |
|----|-------------|--------|--------|--------|
| 割引 (わりびき) | N2 | `<ruby>割引<rt>わりびき</rt></ruby>` | `<ruby>割引<rt>わりびき</rt></ruby>` | → thay bằng `やすく` |
| 自転車 (じてんしゃ) | N4 | 自転車 (không furigana) | 自転車 (không furigana) | `<ruby>自転車<rt>じてんしゃ</rt></ruby>` |
| 経験 (けいけん) | N3 | 経験 (không furigana) | `<ruby>経験<rt>けいけん</rt></ruby>` | → thay bằng `こと・したこと` |
| 哲学 (てつがく) | N1 | `<ruby>哲学<rt>てつがく</rt></ruby>` | → không dùng từ này | → không dùng |

### R4.4 Furigana cho name riêng

Tên người/địa danh → **thêm furigana** nếu kanji có thể đọc nhiều cách:

- `<ruby>山田<rt>やまだ</rt></ruby>` (khó đọc sai)
- `<ruby>佐藤<rt>さとう</rt></ruby>`

Ở N5 + N4, tên riêng thường viết hiragana hết (`やまだ`, `さとう`).

### R4.5 Số lượng ruby cho đoạn văn vừa

Vì bài dài 250-620 chars, cho phép nhiều ruby hơn, nhưng vẫn không quá density target (xem R3 bảng).

**Dấu hiệu rắc furigana sai**:
- Bài có > 15 ruby tag → chắc chắn sai level bài
- Có ruby cho từ cơ bản như `食<rt>た</rt>べる`, `見<rt>み</rt>る`, `行<rt>い</rt>く` ở bài N3 trở lên → thừa
- Ruby tag kéo dài hơn 6 ký tự hiragana → từ quá vượt level, nên thay từ khác

### R4.6 Annotation `注` vs Furigana

Chọn một:

- **Furigana** cho từ có reading khó nhưng nghĩa dễ đoán từ ngữ cảnh
- **注 annotation** cho từ chuyên môn cần giải thích nghĩa

Ví dụ:
- `<ruby>憧れ<rt>あこがれ</rt></ruby>` → furigana OK (nghĩa dễ đoán)
- `普遍的` N2 term trong bài N3 → thêm `注1 普遍的：すべてのものに共通してあてはまる様子` → không cần furigana cho 的

Cả hai cùng lúc OK nếu cần:
```html
<ruby>普遍的<rt>ふへんてき</rt></ruby>（注1）
...
<div class="annotations">
    <p>注1 普遍的：すべてのものに共通してあてはまる様子</p>
</div>
```
