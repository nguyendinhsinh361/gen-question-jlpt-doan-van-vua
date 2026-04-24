# Rules: Nội dung, Layout, Format & Visual (R1, R2, R7, R8)

> **Scope**: Đoạn văn vừa (中文 / medium-passage) — văn xuôi Nhật 250-620 ký tự với **2-3 câu hỏi** trắc nghiệm.

## R1. Chủ đề & Văn phong theo level

> **NGUYÊN TẮC**: Đoạn văn vừa là văn xuôi có luận điểm/câu chuyện có chiều sâu — test khả năng **hiểu nhân quả / nắm ý chính / hiểu từ tham chiếu / hiểu ý tác giả** qua 2-3 câu hỏi phủ các đoạn khác nhau của bài.

| Level | Chủ đề | Văn phong | Cấu trúc câu |
|-------|--------|-----------|--------------|
| **N5** | Thư dài, nhật ký vài đoạn, giới thiệu bản thân, kể sự kiện đơn giản | Thân mật, nhiều hiragana, có full-width space `　` | ～です/～ます, ～てから, ～たい, ～てください |
| **N4** | Kỷ niệm, lịch học, câu chuyện đời sống, thư cho bạn bè, kế hoạch | Lịch sự đơn giản | ～ことができます, ～なければなりません, ～と思います, ～ので |
| **N3** | Essay nhẹ, bài báo, giải thích hiện tượng, thí nghiệm nhỏ, thư có nội dung | Nửa formal nửa conversational | ～について, ～によって, ～場合は, ～ために, ～にとって |
| **N2** | Tiểu luận, email công việc, phê bình văn hóa, phân tích xã hội | Formal, văn viết | ～に伴い, ～に基づき, ～を踏まえて, ～に限り, ～とはいえ |
| **N1** | Luận điểm triết học, phê bình sâu, phân tích logic phức tạp, email formal | Rất formal, văn viết cao cấp | ～いかんによらず, ～をもって, ～に先立ち, keigo cao |

### Topic tag — BẮT BUỘC

Tag chọn từ `rules/topic.json` — Phase 2 dùng **tiếng Việt** (khớp data mẫu & `smoke_test.csv`).

| Category | Ví dụ tag (vi) |
|----------|----------------|
| Đời sống & Tâm lý | `tâm lý học`, `cảm xúc`, `tư duy`, `động lực`, `phát triển bản thân`, `hạnh phúc`, `tuổi thơ`, `trải nghiệm cá nhân` |
| Xã hội & Quan hệ | `xã hội`, `gia đình`, `mối quan hệ`, `tình bạn`, `nuôi dạy con`, `truyền thống`, `phép lịch sự` |
| Giáo dục & Ngôn ngữ | `giáo dục`, `trường học`, `học tập`, `ngôn ngữ`, `giao tiếp`, `thư viện` |
| Kinh tế & Kinh doanh | `kinh tế`, `kinh doanh`, `thị trường`, `tiêu dùng`, `tài chính cá nhân` |
| Công việc | `công việc`, `sự nghiệp`, `nơi làm việc`, `email công việc`, `thông báo` |
| Khoa học & Công nghệ | `khoa học`, `sinh học`, `công nghệ`, `nghiên cứu`, `năng lượng` |
| Sức khỏe & Y học | `sức khỏe`, `y học`, `dinh dưỡng`, `tập luyện`, `căng thẳng` |
| Môi trường & Tự nhiên | `môi trường`, `tự nhiên`, `động vật`, `thời tiết`, `mùa`, `tái chế` |
| Văn hóa & Nghệ thuật | `văn hóa`, `nghệ thuật`, `âm nhạc`, `văn học`, `triết học`, `thể thao`, `du lịch` |
| Đời sống & Tiêu dùng | `đời sống thường nhật`, `mua sắm`, `thức ăn`, `nấu ăn`, `thú cưng` |
| Hạ tầng & Truyền thông | `giao thông`, `thành phố`, `truyền thông`, `báo chí` |

> **⚠️ KHÔNG dùng tag tiếng Anh hoặc tiếng Nhật.**
> Phải dùng tiếng Việt đúng cột `vi` của `rules/topic.json` (ví dụ: ✅ `văn hóa`, `kinh tế`, `đời sống thường nhật`).

Trong batch > 5 bài, chọn topic từ ≥ 3 category khác nhau để đa dạng.

---

## R2. Format văn bản & Độ dài

### Target character count (BẮT BUỘC)

Đo bằng `count_body_chars()` — đếm **ký tự visible trong body**, bỏ whitespace, bỏ `<rt>` furigana.

| Level | Target Range (P25–P75) | Hard Reject (< Min) | Min Sample | Max Sample |
|-------|------------------------|---------------------|------------|------------|
| N1    | **550–620**            | < 500 → gen lại     | 522        | 772        |
| N2    | **530–610**            | < 500 → gen lại     | 502        | 1209       |
| N3    | **380–500**            | < 350 → gen lại     | 355        | 984        |
| N4    | **490–610**            | < 450 → gen lại     | 457        | 819        |
| N5    | **270–310**            | < 250 → gen lại     | 254        | 378        |

> **🚫 HARD REJECT**: Nếu `count_body_chars()` **thấp hơn Hard Reject threshold**, bài **PHẢI gen lại từ đầu**. Không chấp nhận, không chỉnh sửa nhỏ — gen lại hoàn toàn.
> **⚠️ UNDER TARGET**: Dưới Target Range nhưng ≥ Hard Reject → bổ sung 1 câu văn hoặc 1 `注` annotation. Không tự thêm whitespace.
> **⚠️ OVER TARGET**: Cho phép dài hơn tới +30 chars; quá nhiều thì cảnh báo.

**Lưu ý N4 > N3**: N4 target cao hơn vì N4 dùng nhiều hiragana, cùng nội dung nhưng nhiều ký tự hơn — pattern đúng theo JLPT thật.

### Số câu hỏi per bài (BẮT BUỘC)

Dựa trên `rules/question_format.json` (spec JLPT chính thức):

| Level | Q Count |
|-------|---------|
| N1    | **2**   |
| N2    | **2**   |
| N3    | **3**   |
| N4    | **3**   |
| N5    | **2**   |

- CSV columns `question_1..question_{n}` populate đủ.
- Các column `question_{n+1}..question_5` = "" (empty string).
- **Data gốc có noise** (N1 thường có 3 câu) — skill luôn follow spec trên, không bắt chước data.

### Flow text (KHÔNG `<br>` giữa câu)

> **⛔ LỖI PHỔ BIẾN**: Data gốc dùng `<br>` thay `<p>` (thói quen xấu).
> **Output KHÔNG dùng `<br>` giữa câu** — dùng `<p>` thuần, text flow liên tục.

**Quy tắc:**
- **ĐÚNG**: `<p>Câu 1。Câu 2。Câu 3。</p>` — 1 paragraph 1 `<p>`
- **SAI → REJECT**: `<p>Câu 1。<br>Câu 2。</p>` — có `。<br>`
- **Ngắt paragraph** chỉ khi chuyển ý hoàn toàn khác
- **Xuống hàng trong source code** chỉ để dễ đọc; HTML parser sẽ collapse whitespace

**Ngoại lệ — N5 letter format** (cho phép `<br>` để giữ layout thư):

```html
<p class="no-indent">「やまださんへ<br>
こんにちは。<br>
きのう、えいがを　みに　いきました。<br>
とても　たのしかったです。<br>
　　　たなか　ゆき」</p>
```

### CSS layout bắt buộc

- **Container**: `max-width: 720px`, `margin: 0 auto`, white background trên light gray body `#f9fafb`
- **Body**: `word-break: keep-all`, `line-break: strict`, `overflow-wrap: break-word`
  (`keep-all` đảm bảo xuống dòng ở ranh giới từ, tránh cắt kanji compound)
- **Paragraph**: `<p>` với `text-indent: 1em` (chuẩn văn Nhật)
- **Font**: Noto Sans JP qua Google Fonts (KHÔNG dùng Tailwind CDN)
- **Đặc biệt N5**: thêm full-width space `　` giữa các cụm từ (mô phỏng đề N5 thật)

Template chi tiết xem `rules/technical.md` R9.

### Test mơ hồ (BẮT BUỘC)

> **Mỗi đoạn văn phải có DUY NHẤT 1 cách hiểu hợp lý cho mỗi câu hỏi.**
> Sau khi viết xong, đọc lại: "Câu hỏi này có thể hiểu theo cách thứ 2 không?" Nếu có → sửa lại.
> **Đặc biệt quan trọng với multi-question**: 2-3 câu hỏi trong 1 bài KHÔNG được test cùng 1 đoạn/ý.

---

## R7. Các dạng bài (document formats)

Đoạn văn vừa có 6 dạng chính:

| # | Format | Level | Đặc điểm | Ví dụ chủ đề |
|---|--------|-------|----------|--------------|
| 1 | **Essay/commentary** | N1, N2, N3 | 3-5 paragraph văn nghị luận, có thể có source line | 芸術, 文化, 言語, 人生 |
| 2 | **Anecdote/story** | N2, N3, N4 | Câu chuyện đời thường có rút ra bài học, nhiều marker ①② | 家族, 子育て, 友情, 旅行 |
| 3 | **Advice/tip** | N3, N4 | Lời khuyên/tips có lý giải (≥ 3 đoạn) | 食事, 健康, 学習, 仕事 |
| 4 | **Article/news** | N2, N3 | Bài báo 3-5 đoạn, có thể có annotation 注 | 環境, 技術, 社会 |
| 5 | **Letter/email** | N1-N5 | Thư dài (N5 dùng `<br>`), email công việc (N2/N1) | お礼, ご報告, 業務連絡 |
| 6 | **Diary/note** | N4, N5 | Nhật ký nhiều đoạn hoặc note dài | 旅行日記, 一日の記録 |

Mỗi bài phải có đủ nội dung để trả lời 2-3 câu hỏi phủ các đoạn khác nhau — vì vậy **minimum 3 paragraph** cho N1-N4.

### Source line convention (giống các phase khác)

Format: `（[fake author]「[fake title]」による）` hoặc `（[author]「[title]」[media]による）`

- Author: tên Nhật tự chế (2-4 chữ, họ + tên), VD: `山田太郎`, `佐藤花子`, `田中美咲`, `鈴木健一`
- **⛔ TUYỆT ĐỐI KHÔNG** dùng tên tác giả thật (`村上春樹`, `夏目漱石`, `芥川龍之介`, `太宰治`...)
- Title Nhật tự chế ngắn gọn phù hợp nội dung

### Tần suất source line theo data (khuyến nghị)

| Level | Tần suất thực tế | Có nên thêm? |
|-------|------------------|--------------|
| N1    | 13%              | Đôi khi (essay) |
| N2    | 20%              | **Nên thêm** cho tiểu luận formal |
| N3    | 9%               | Đôi khi |
| N4    | 0%               | Không |
| N5    | 0%               | Không |

---

## R8. Visual elements cho multi-question

### Marker ①②③④

**Tần suất data**: N1=7%, N2=47%, **N3=54%**, N4=17%, N5=2%.

Với đoạn văn vừa multi-question, **marker dùng nhiều** — vì mỗi câu hỏi `question_reference` cần 1 marker riêng.

**Quy tắc**:
- Mỗi câu `question_reference` → 1 marker riêng trong bài (`①`, `②`, `③`)
- Marker đứng **ngay trước** cụm từ bị hỏi, kèm `<u>...</u>`:
  ```html
  <p>...<span class="marker">①</span><u>自分たちのシステム</u>を構築することが重要...</p>
  ...
  <p>そして、<span class="marker">②</span><u>この現象</u>は今も続いている...</p>
  ```
- Question 1: `「①自分たちのシステム」とあるが、何を指すか。`
- Question 2: `「②この現象」とあるが、どのような現象か。`

### Underline `<u>`

**Tần suất data**: N1=72%, N2=47%, N3=33%, N4=34%, N5=20% — rất phổ biến.

Lý do: mỗi câu hỏi reference/meaning đều cần gạch chân cụm từ bị hỏi. Skill dùng `<u>` cho MỌI câu hỏi `question_reference` và `question_meaning_interpretation`.

### Annotation `注`

**Tần suất data**: N1=34%, N2=43%, **N3=54%**, N4=6%, N5=0%.

Bài dài hơn → nhiều thuật ngữ cần giải thích. **Khuyến khích** thêm 1-2 `注` cho N1-N3.

Format:
```html
<div class="annotations">
    <p>注1 憧れ（あこがれ）：強く心を引きつけられること</p>
    <p>注2 普遍的（ふへんてき）：すべてのものに共通してあてはまる様子</p>
</div>
```

- Đặt **ngay trước** `<p class="source">` (nếu có)
- Mỗi 注 một `<p>` riêng, dùng full-width `：`
- Số thứ tự `注1`, `注2`, `注3`

### Blank `[ ]` hoặc `( 1 )` — ĐẶC TRƯNG N1

**Tần suất N1 = 95%** — gần như bài N1 nào cũng có 1 chỗ trống fill_in_the_blank.

Format:
```html
<p>...人間の幸福というものは、単なる快楽の積み重ねではなく、[ ① ]であると言えよう。</p>
```

Question: `[ ① ] に入る最も適当なものを選びなさい。`

**Quy tắc**:
- N1: **KHUYẾN KHÍCH** 1 chỗ trống (95% data)
- N2-N5: KHÔNG bắt buộc, nếu có thì dùng format `[ ① ]` hoặc `( 1 )`

### Cheatsheet — Visual elements per level

| Level | Marker ①② | `<u>` | 注 | Source | Blank `[ ]` |
|-------|-----------|-------|-----|--------|-------------|
| N1    | Đôi khi   | **Có (72%)** | Có (34%) | Đôi khi (13%) | **Có (95%)** |
| N2    | **Có (47%)** | **Có (47%)** | **Có (43%)** | **Có (20%)** | Ít (17%) |
| N3    | **Có (54%)** | Có (33%) | **Có (54%)** | Đôi khi (9%) | Ít (6%) |
| N4    | Ít (17%)  | Có (34%) | Ít (6%) | Không | Không |
| N5    | Không     | Ít (20%) | Không | Không | Không |

### Common errors

1. ❌ Có marker `①` trong bài nhưng KHÔNG có câu hỏi reference tương ứng → marker vô nghĩa
2. ❌ Câu hỏi hỏi `①...とあるが` nhưng bài không có marker `①` → câu hỏi không thể trả lời
3. ❌ 2-3 câu hỏi trong 1 bài đều test cùng 1 đoạn → thiếu coverage
4. ❌ N4/N5 mà có source line (data 0%) → không phù hợp level
5. ❌ Source line dùng tên tác giả thật → vi phạm IP
