# HTML Patterns — Đoạn Văn Vừa

Template HTML cụ thể cho từng level, marker convention cho multi-question, annotation & source format.

## 1. Base Template

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{JP_TITLE}</title>
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
            max-width: 720px;      /* wider than short-passage (640) */
            margin: 0 auto;
            background: white;
            padding: 48px 56px;
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

## 2. Template Per Level

### N1 — 2 câu (95% có fill-in-the-blank)

Use case: phê bình triết học, luận điểm sâu. Spec focus: nhân quả + lý do.

```html
<div class="passage">
    <p>現代社会において、私たちは情報の<ruby>氾濫<rt>はんらん</rt></ruby>の中で生きている。
    その中で本質的なものを見抜く力こそ、現代人にとって最も必要とされる能力である。</p>

    <p>しかし、この能力は一朝一夕に身につくものではない。日々の<u>注意深い観察</u>と、
    他者との対話を通じた思考の深化が不可欠である。[ ① ]、現代教育が目指すべきは、
    単なる知識の<ruby>伝達<rt>でんたつ</rt></ruby>ではなく、思考する力の育成であろう。</p>

    <p>このような思考力は、一人では決して育たない。他者と意見を交わし、異なる視点に
    触れることで初めて<ruby>鍛<rt>きた</rt></ruby>えられるものである。</p>

    <div class="annotations">
        <p>注1 氾濫：overflow</p>
        <p>注2 伝達：transmission</p>
    </div>
    <p class="source">（山田太郎「現代思考論」による）</p>
</div>
```

Câu hỏi:
1. `[ ① ] に入る最も適当なものはどれか。` — fill_in_the_blank
2. `「注意深い観察」とあるが、筆者はなぜこれを重視しているか。` — reason_explanation

**Điểm chính**:
- 3 paragraph là sweet spot (có thể 4 nếu cần)
- Blank `[ ① ]` gần cuối để test kết luận
- Annotation cho thuật ngữ chuyên ngành
- Source line phổ biến (13%)
- 0-2 `<ruby>` (chỉ cho từ vượt N1)
- 1 `<u>` cho cụm được hỏi reason

### N2 — 2 câu (marker + annotation phổ biến)

Use case: tiểu luận đời sống, phê bình văn hóa nhẹ.

```html
<div class="passage">
    <p>最近、職場で年齢について話題になることが減った。以前は「もう何歳？」という質問が
    日常的だったが、<span class="marker">①</span><u>そのような質問</u>は今ではタブー視
    される傾向にある。</p>

    <p>これは、個人の<ruby>尊厳<rt>そんげん</rt></ruby>を重視する現代的価値観の
    <span class="marker">②</span><u>表れ</u>と言えるだろう。年齢はあくまでプライベート
    な情報であり、仕事の能力とは直接関係しないという考え方が広まっている。</p>

    <p>一方で、世代間のコミュニケーションが希薄になっているという指摘もある。注1
    年齢を話題にしないことで、相互理解の機会を失っているのではないだろうか。</p>

    <div class="annotations">
        <p>注1 希薄：thin, sparse</p>
    </div>
    <p class="source">（佐藤花子「現代社会と言葉」文春社による）</p>
</div>
```

Câu hỏi:
1. `①「そのような質問」とあるが、どんな質問を指すか。` — reference (marker ①)
2. `②「表れ」とあるが、何の表れか。` — reference (marker ②)

Hoặc mix:
1. `①「そのような質問」とあるが、なぜ今ではタブー視されるのか。` — reason_explanation
2. `筆者の考えとして最も適当なものはどれか。` — author_opinion

**Điểm chính**:
- 3 paragraph, văn formal
- 2 marker ①② cho 2 câu reference (47% data có marker)
- Annotation phổ biến (43%)
- Source line (20% data có)

### N3 — 3 câu (marker dày nhất + annotation)

Use case: thí nghiệm, bài báo khoa học đơn giản, thư có nội dung.

```html
<div class="passage">
    <p>ハトの視覚について興味深い実験が行われた。研究者は、ハトに複数の絵を見せ、
    特定の絵を選ばせる訓練をした。</p>

    <p>実験の手順はこうである。まず、<span class="marker">①</span><u>画面に二枚の絵を
    表示する</u>。一方は木の絵、もう一方は花の絵。ハトが木の絵をついばめば、餌が出る
    仕組みだ。</p>

    <p>数週間の訓練の後、ハトは木の絵を正確に選べるようになった。さらに驚くべきことに、
    訓練で使わなかった<span class="marker">②</span><u>別の木の絵</u>を見せても、正しく
    選択することができた。これは、ハトが「木」という注1<ruby>概念<rt>がいねん</rt>
    </ruby>を理解していることを示すのだろうか。</p>

    <p>研究者は、この結果を<u>注意深く解釈する</u>必要があると述べている。ハトが概念を
    持っているのか、それとも共通する特徴を学習しているだけなのか、まだ結論は出ていない。</p>

    <div class="annotations">
        <p>注1 概念：concept</p>
    </div>
</div>
```

Câu hỏi:
1. `①「画面に二枚の絵を表示する」とあるが、これはどのような実験の一部か。` — reference (marker)
2. `②「別の木の絵」とあるが、これが選ばれたことから何がわかるか。` — reference (marker)
3. `この実験の結果について、筆者はどのように考えているか。` — author_opinion / reason

**Điểm chính**:
- 4 paragraph là thường gặp
- 2 marker ①② cho 2 câu reference đầu
- Câu 3 là tổng hợp (author opinion)
- Annotation rất khuyến khích (54% data)
- Không bắt buộc source line (9%)

### N4 — 3 câu (ít marker, ít annotation)

Use case: kỷ niệm đời sống, thư cho bạn bè, câu chuyện ngày thường.

```html
<div class="passage">
    <p>私は先月、友達と山登りに行きました。天気がとても良くて、空気もきれいでした。</p>

    <p>朝早く家を出て、電車に乗りました。山の駅に着いたのは９時ごろでした。駅から山の
    入り口まで、歩いて３０分かかりました。</p>

    <p>山を登るのは大変でした。途中でお弁当を食べました。おにぎりとお茶です。とても
    おいしかったです。</p>

    <p>山の上から見える景色はすばらしかったです。遠くに海も見えました。友達と一緒に
    写真をたくさん撮りました。</p>

    <p>また行きたいと思います。次は桜の季節に行くつもりです。</p>
</div>
```

Câu hỏi:
1. `この人は誰と山登りに行きましたか。` — content_match
2. `山の上から何が見えましたか。` — content_match
3. `この人は、次はいつ山に行きたいと思っていますか。` — content_match

**Điểm chính**:
- 4-5 paragraph, câu ngắn hơn
- Đa số content_match questions
- KHÔNG annotation (6% data)
- KHÔNG source line (0% data)
- Ít marker (17%)

### N5 — 2 câu (rất simple)

Use case: thư dài, nhật ký vài đoạn, thông báo chi tiết.

```html
<div class="passage">
    <p>きのう、わたしは　おばあちゃんの　いえに　いきました。</p>

    <p>おばあちゃんは　７０さいです。げんきで、まいにち　こうえんを　さんぽします。
    にわで　はなを　そだてて　います。</p>

    <p>おばあちゃんは　りょうりが　じょうずです。わたしは　てんぷらを　たべました。
    とても　おいしかったです。</p>

    <p>ひるごはんの　あと、おばあちゃんと　さんぽに　いきました。こうえんで、
    ねこを　みました。</p>

    <p>ゆうがた、わたしは　いえに　かえりました。また　おばあちゃんの　いえに　
    いきたいです。</p>
</div>
```

Câu hỏi:
1. `この人は、きのう　どこへ　いきましたか。` — content_match
2. `おばあちゃんは　なにが　じょうずですか。` — content_match

**Điểm chính**:
- 4-5 paragraph, câu rất ngắn
- Full-width space `　` giữa cụm từ
- 0-1 `<ruby>`
- KHÔNG annotation, KHÔNG source
- Có thể dùng letter format với `<br>` nếu là thư

## 3. Marker Strategy Cho Multi-Question

### Pattern 1 — 2 câu, cả 2 đều reference (N2/N3 phổ biến)

```html
<p>...<span class="marker">①</span><u>cụm từ 1</u>...</p>
<p>...<span class="marker">②</span><u>cụm từ 2</u>...</p>
```

Câu hỏi: `①「cụm từ 1」とあるが、〜` + `②「cụm từ 2」とあるが、〜`

### Pattern 2 — 2 câu, 1 reference + 1 reason (N1/N2)

```html
<p>...<u>cụm từ được hỏi reference</u>...</p>
<p>...<span class="marker">※</span>原因của hiện tượng X...</p>
```

Câu hỏi: `「cụm từ」とあるが、〜` + `なぜ X なのか。`

### Pattern 3 — 3 câu, 2 reference + 1 tổng hợp (N3)

```html
<p>Paragraph 1 với ①<u>cụm 1</u></p>
<p>Paragraph 2 với ②<u>cụm 2</u></p>
<p>Paragraph 3 — conclusion without marker</p>
```

Câu hỏi: ①, ②, và 1 câu tổng hợp từ đoạn 3.

### Pattern 4 — N4/N5: không marker, content_match đa số

```html
<p>Paragraph 1: thông tin A</p>
<p>Paragraph 2: thông tin B</p>
<p>Paragraph 3: thông tin C</p>
```

Mỗi câu hỏi test 1 paragraph khác nhau (tránh overlap).

## 4. Source Line Format

Format chuẩn: `（[fake_author]「[fake_title]」[media?]による）`

| Level | Tần suất data | Nên dùng |
|-------|---------------|----------|
| N1    | 13%           | Có — bài triết học/phê bình |
| N2    | **20%**       | **Rất nên** — tiểu luận |
| N3    | 9%            | Có khi có — bài formal |
| N4    | 0%            | **KHÔNG** |
| N5    | 0%            | **KHÔNG** |

Fake author examples:
```
山田太郎, 佐藤花子, 鈴木一郎, 田中美和子, 高橋健太, 伊藤由美,
中村明, 小林さとし, 加藤けいこ, 吉田あきら, 山口まり, 斎藤ひろし
```

Fake title/media:
```
「現代社会論」, 「暮らしの中の言葉」, 「日本語の風景」, 「文化を読む」
新聞, 雑誌, 文春社, 朝日, 月刊〇〇
```

## 5. 注 Annotation Format

```html
<div class="annotations">
    <p>注1 〜〜〜：〜〜〜</p>
    <p>注2 〜〜〜：〜〜〜</p>
</div>
```

| Level | Tần suất | Khuyến nghị |
|-------|----------|-------------|
| N1    | 34%      | Có — 1-2 chú thích |
| N2    | **43%**  | **Rất nên** — 1-3 chú thích |
| N3    | **54%**  | **Rất nên** — 1-3 chú thích |
| N4    | 6%       | Ít khi — chỉ nếu thật sự cần |
| N5    | 0%       | **KHÔNG** |

## 6. Paragraph Count Guidelines

| Level | Paragraph count | Lý do |
|-------|-----------------|-------|
| N1    | 3–4             | Văn logic chặt, mỗi đoạn 1 ý |
| N2    | 3–4             | Tương tự N1 |
| N3    | 3–5             | Nhiều thí nghiệm/câu chuyện có flow nhiều bước |
| N4    | 4–5             | Câu ngắn nên chia nhỏ |
| N5    | 4–5             | Câu rất ngắn, mỗi ý 1 đoạn |

## 7. Visual Elements Catalog

| Element | HTML | Khi nào dùng |
|---------|------|--------------|
| Gạch chân | `<u>…</u>` | Cụm được hỏi (reference/meaning) |
| Marker số | `<span class="marker">①</span>`, `②`, `③` | Bài có ≥ 2 câu reference |
| Furigana | `<ruby>漢字<rt>かんじ</rt></ruby>` | Chỉ cho từ vượt level |
| Blank | `[ ① ]`, `（ ）`, `　ａ　` | fill_in_the_blank (N1 đặc biệt) |
| Chú thích | `<div class="annotations">` | N1-N3 (khuyến khích) |
| Source | `<p class="source">` | N1-N3 (essay/phê bình) |
| Full-width space | `　` | N5 giữa cụm từ |

## 8. Cheatsheet

| Feature | N1 | N2 | N3 | N4 | N5 |
|---------|-----|-----|-----|-----|-----|
| Paragraphs | 3-4 | 3-4 | 3-5 | 4-5 | 4-5 |
| `<ruby>` count | 0-4 | 0-8 | 0-10 | 0-6 | 0-3 |
| `<u>` gạch chân | Có | Có (47%) | Có (33%) | Có (34%) | Hiếm |
| Marker ①②③ | 7% | **47%** | **54%** | 17% | 2% |
| Annotation | 34% | **43%** | **54%** | 6% | 0% |
| Source line | 13% | **20%** | 9% | 0% | 0% |
| Fill-in-blank | **95%** | 17% | 6% | 0% | 0% |
| Full-width `　` | No | No | No | Hiếm | **Có** |

## 9. Common Mistakes

1. **Chỉ có 1 câu hỏi** — SAI, phải 2 hoặc 3
2. **2 câu hỏi test cùng 1 paragraph** — SAI, mỗi câu phải test đoạn khác
3. **Marker trong HTML không match question** — phải khớp
4. **Thêm source line cho N4/N5** — KHÔNG (0% data)
5. **Thêm annotation cho N5** — KHÔNG (0% data)
6. **Gen 2 câu cho N3/N4 (spec yêu cầu 3)** hoặc ngược lại — SAI
7. **Tất cả câu cùng label** — bắt buộc ≥ 2 label khác nhau
8. **Dùng tên tác giả thật** — KHÔNG, luôn tự chế
9. **Char count < Hard Reject** — phải gen lại, không chỉnh sửa nhỏ
