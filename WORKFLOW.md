# QUY TRÌNH — từ khóa cũ tới khóa mới có hướng dẫn sửa

Ghi lại để lần sau chạy lại được cho khóa khác mà không phải dựng lại từ đầu.

## Bốn tầng dữ liệu

| Tầng | Thư mục | Nội dung | Web |
|---|---|---|---|
| 1. Review | `review/*.review.json` | Đánh giá khóa cũ: điểm 7 tiêu chí, hành động 4 màu ở cấp chương/mục/bài | Trang 1–3 |
| 2. Khóa gộp | `merged/*.json` | Khóa mới: lấy bài nào từ khóa nào, nhãn `reuse`/`adapt`/`new` | `#/m/<key>` |
| 3. Sửa chi tiết | `merged/*.json` → trường `edits` | Từng chỗ sửa: nguyên văn đoạn gốc → đoạn thay thế | Mục "Sửa như thế nào" |
| 4. Giáo trình | `courseware/*.json` | Nội dung soạn thật: bài giảng, slide, kịch bản video | `#/g/<key>` |

Mỗi tầng độc lập — thêm file mới vào thư mục là web tự nạp, **không phải sửa code**.

## Chạy tầng 3 (sửa chi tiết) cho một khóa gộp

```bash
# 1. Xuất nguyên văn bài gốc của các bài `adapt`
./.venv/bin/python dump_adapt.py <tên-khóa>
#    → review/adapt-<tên-khóa>.md

# 2. Giao người/agent soạn edits, ghi ra merged/_edits<N>-<tên-khóa>.json
#    (chia nhiều mảnh cho nhiều người làm song song, không ghi đè nhau)

# 3. Ghép các mảnh vào khóa gộp
./.venv/bin/python apply_edits.py <tên-khóa>

# 4. Kiểm tra: mọi `before` phải khớp nguyên văn bài gốc
./.venv/bin/python build_merged.py       # báo lỗi nếu có chỗ không khớp

# 5. Dựng web
./.venv/bin/python build_web.py
```

## Chạy tầng 4 (giáo trình) cho một khóa

```bash
# Nhiều người soạn song song, mỗi người một mảnh
#   courseware/_part<N>-<tên>.json
./.venv/bin/python merge_parts.py <tên>      # ghép các mảnh
./.venv/bin/python build_courseware.py       # thống kê + cảnh báo thiếu kịch bản
./.venv/bin/python build_web.py
```

## Vì sao phải có bộ kiểm tra `before`

Đây là chỗ hỏng âm thầm nguy hiểm nhất. Nếu người soạn **tóm tắt** đoạn gốc thay
vì chép nguyên văn, thì:

- Web không bôi vàng được → biên tập viên không biết sửa chỗ nào
- Nhìn bằng mắt **không phát hiện ra** — file vẫn hợp lệ, vẫn hiển thị

`build_merged.py` đối chiếu từng `before` với nội dung thật đã crawl trong
`output/full/`. Không khớp thì báo ra console **và** hiện nhãn đỏ *"không tìm
thấy trong bài gốc"* ngay tại chỗ đó trên web.

Tương tự, `apply_edits.py` báo nếu `originCode` + `originActivityId` không trỏ
tới bài nào trong khóa gộp, và `build_merged.py` báo nếu `origin` trỏ tới id
không có thật trong khóa nguồn.

## Kinh nghiệm: agent hay chết giữa chừng

Trong quá trình làm, agent bị ngắt **hơn 10 lần** (máy ngủ, hoặc treo quá 600
giây). Cách duy nhất hiệu quả:

- **Bắt ghi đè file ra đĩa sau MỖI bài**, không dồn đến cuối
- Chia việc thành mảnh nhỏ, mỗi mảnh một file riêng
- Khi agent chết, `SendMessage` nối tiếp từ chỗ dở thay vì làm lại từ đầu —
  nhớ nói rõ **đã làm tới đâu** và **bài nào bỏ qua**

Nhờ vậy không mất bài nào dù bị ngắt liên tục.

## Kết quả hiện có

| Khóa gộp | Bài | Bài `adapt` có sửa chi tiết | Chỗ sửa |
|---|---:|---:|---:|
| Scratch lớp 3–5 | 110 | 32/32 | 87 |
| Python lớp 6–9 | 181 | 37/38 | 73 |
| Python lớp 10–12 | 222 | *(đang soạn)* | |

| Giáo trình chi tiết | Bài | Slide | Cảnh quay |
|---|---:|---:|---:|
| ScratchJr lớp 1–2 | 32 | 143 | 118 |
