# Prompt — Gen bài Đoạn Văn Vừa (JLPT 中文読解)

## Cách dùng

Copy prompt bên dưới, thay `{số}` rồi paste vào Claude hoặc Gemini.

**Đặc thù: multi-question** — N1/N2/N5 = 2 câu, N3/N4 = 3 câu. Mỗi câu phủ đoạn/ý KHÁC NHAU.

> **🚨 ZERO-TOLERANCE WORKFLOW**: SKILL.md có **5 GATE bắt buộc** (0→1, 1→2, 2→3, 3→4, 4→5). Mỗi gate phải log `GATE X→Y PASSED` mới được sang bước tiếp. **1 mục FAIL = sửa/gen lại → QC TỪ ĐẦU**, đến khi 30/30 PASS mới hoàn thành.

---

## Prompt

```
Đọc .claude/skills/jlpt-reading-medium-passage/SKILL.md rồi gen bài đọc hiểu đoạn văn vừa:
- N5: {số} bài | N4: {số} bài | N3: {số} bài | N2: {số} bài | N1: {số} bài

Lưu CSV: sheets/samples_v1.csv. HTML: assets/html/doan_van_vua/{LEVEL}_{uuid}.html.

🔒 5 GATE bắt buộc — KHÔNG QUA = KHÔNG SANG BƯỚC TIẾP. Log explicit GATE X→Y PASSED.

═══ BƯỚC 0 — CHUẨN BỊ (1 lần) → GATE 0→1 ═══
Đọc đầy đủ:
- rules/rule_doc_hieu.md (Phần 2.4 thể chia, Phần 5 — 7 loại bẫy, Phần 6.2–10.2 per level)
- rules/{content,vocabulary,technical,questions}.md + rules/kanji_jlpt_sensei.csv (2495 kanji)
- Load 2-3 sample: scripts/load_references.py --level {LEVEL} --count 2
- Scan sheets/samples_v1.csv để biết topic + Q label combo đã dùng
GATE 0→1: tick 6/6 → log "GATE 0→1 PASSED".

═══ BƯỚC 1 — GEN HTML + 2-3 Q+A → GATE 1→2 ═══
1. _id = {LEVEL}_{uuid32}
2. Tag = **tiếng Anh** từ rules/topic.json
3. Gen HTML đúng spec, marker ①② khớp Q reference, **thể chia đúng Phần 2.4**, furigana chỉ vượt level (cấm "Ab")
4. Gen Q + 4 đáp án (newline \n, KHÔNG prefix); ≥ 2 unique label per bài; mỗi câu test đoạn KHÁC NHAU; distractor ≥ 3 loại bẫy
5. Tạo CSV bằng scripts/process_html.py + scripts/fill_qa.py (KHÔNG sửa CSV tay)
GATE 1→2: tick 5/5 → log "GATE 1→2 PASSED".

═══ BƯỚC 2-3 — QC 30 MỤC → GATE 2→3 + GATE 3→4 ═══
GATE 2→3: cam kết check ĐẦY ĐỦ 30 mục → log "GATE 2→3 PASSED".
Đánh giá 30 mục: A. HTML + B. Content (thể chia, từ vựng level) + C. Q&A (label/đáp án/explain VN+EN/self-solve khớp) + D. Multi-Q Coverage.
**Self-solve cho TỪNG câu hỏi**: tự giải KHÔNG nhìn correct.
GATE 3→4: liệt kê mục FAIL + diagnosis → log "GATE 3→4 PASSED — fix list".

═══ BƯỚC 4-5 — SỬA + LẶP → GATE 4→5 ═══
- Fix HTML → `process_html.py --refresh`. Fix Q&A → fill_qa.py
- ≥ 50% FAIL hoặc self-solve FAIL hoặc char Hard Reject → **GEN LẠI** (giữ _id)
- Quay lại GATE 2→3 → QC 30/30 TỪ ĐẦU (KHÔNG chỉ check mục đã sửa)
- Tối đa 5 vòng → vẫn FAIL → báo user, KHÔNG sang bài tiếp
GATE 4→5: 30/30 PASS + --validate clean → log "🎉 ALL PASSED (30/30) + GATE 4→5 PASSED" → bài tiếp.

═══ HARD REJECT (gen lại ngay) ═══
- Q count sai: N1=2, N2=2, N3=3, N4=3, N5=2 (slot dư phải empty)
- Char range ngoài (xem rules/content.md): N1 530–620 | N2 470–560 | N3 380–460 | N4 280–350 | N5 250–320
- 2 câu cùng test 1 đoạn (vi phạm coverage); marker ①② không khớp Q hoặc dư
- <ruby> thiếu <rt> rỗng; furigana "Ab"; thể chia trộn lẫn
- Tag tiếng Việt/Nhật; trùng topic; <2 unique label per bài

═══ CUỐI BATCH ═══
python3 .claude/skills/jlpt-reading-medium-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_vua --csv sheets/samples_v1.csv
```
