# Báo cáo kết quả thực thi

Môi trường lightweight được dùng cho toàn bộ tám notebook. Các số dưới đây lấy từ output đã lưu trong `submission/notebooks/`, không phải số liệu mô phỏng thêm sau khi chạy.

## Tổng quan

| Hạng mục | Kết quả |
|---|---:|
| Smoke test | 9/9 PASS |
| Pytest | 24/24 PASS trong 2.66 giây |
| Notebook runner | 8/8 PASS trong 27.7 giây |
| Notebook có cell lỗi | 0/8 |

## Kết quả theo notebook

### NB1 — Delta basics

- Ghi sai `age='thirty'` bị chặn bởi lỗi cast sang `Int64`.
- `_delta_log/` có commit JSON; schema evolution chỉ xảy ra khi dùng `schema_mode="merge"`.
- Cột `tier` được thêm và DuckDB đọc được hai nhóm tier.

Diễn giải: transaction log là nguồn sự thật của bảng; schema enforcement ngăn dữ liệu sai kiểu, còn evolution là hành động opt-in chứ không tự mở rộng schema.

### NB2 — OPTIMIZE và Z-order

- File dữ liệu: `200 → 55` (giảm khoảng 4 lần).
- Speedup đo được: `9.1×` (ngưỡng `≥ 3×`).
- Pruning ratio: `55×` (ngưỡng thay thế `≥ 10×`).

Diễn giải: compaction giảm chi phí mở file, còn Z-order làm vùng min/max của `user_id` có ích cho data skipping. Wall-clock phụ thuộc máy, nên pruning ratio là bằng chứng cơ chế ổn định hơn.

### NB3 — Time travel, MERGE và RESTORE

- MERGE 100K dòng hoàn tất trong `0.07 s`.
- History có `5` version và có transaction RESTORE.
- Sau RESTORE, số dòng `score < 0` bằng `0`.

Diễn giải: RESTORE tạo một commit mới trỏ bảng về trạng thái hợp lệ; nó không xóa lịch sử cũ, vì vậy audit trail vẫn giữ được MERGE và bad write.

### NB4 — Medallion

- Bronze: `200,000` dòng.
- Silver: `190,052` dòng; dedup loại `9,948` dòng.
- Gold: `24` dòng = `8` ngày × `3` model.
- Trên toàn bộ Gold: `p95 - p50` nhỏ nhất vẫn dương (`544.3 ms`), `cost_usd` nhỏ nhất là `13.4966384`, và `error_rate` nằm trong `[0.04217, 0.06184]`.

Diễn giải: Bronze bảo toàn đầu vào, Silver chuẩn hóa/khử trùng, Gold trả lời trực tiếp câu hỏi latency, chi phí và lỗi theo ngày/model.

### NB5 — Iceberg catalog

- Lọc một ngày chỉ đọc 1/10 file, pruning ratio `10×`.
- Metadata/data ratio của bộ dữ liệu nhỏ là `285.3%`.
- Đổi `latency_ms → latency_millis` vẫn giữ `field_id=4`.
- Hai partition spec `[1, 2]` cùng tồn tại và toàn bộ dòng vẫn đọc được.

Diễn giải: hidden partitioning suy ra partition từ predicate trên `ts`; field ID giúp rename không rewrite data. Metadata ratio lớn là hậu quả cố ý của file rất nhỏ, không đại diện bảng production có file 256–512 MB.

### NB6 — Maintenance

- Compaction: `200 → 11` file (`18×` ít hơn).
- Clustering cho point query skip `90%` file.
- Delta vacuum thu hồi `16.1 MB`.
- Tìm và xóa đúng `3` orphan Delta (`21.2 KB`).
- Checkpoint `00000000000000000099.checkpoint.parquet` và `_last_checkpoint` tồn tại.
- Iceberg snapshots: `20 → 3`; sweep xóa `17` manifest list bị mắc kẹt (`36.9 KB`).

Diễn giải: expiry/vacuum và orphan removal là hai việc khác nhau. Trong phiên bản thư viện của lab, Delta vacuum không thấy file chưa từng commit; PyIceberg expiry giảm metadata logic nhưng cần sweep riêng để thu hồi file vật lý.

### NB7 — Multimodal và vector

- Random-read amplification: `200×`.
- Embedding int8 nhỏ hơn float32 `5.8×` trên disk.
- Recall@10: `0.904`; topic fidelity: `1.000`.
- Sau erasure: bảng chính trả `0` hit nhưng external index cũ vẫn trả `8` hit.
- CDF phát ra `8` delete events.

Diễn giải: column pruning bảo vệ analytical scan, nhưng random row access vẫn phải đọc cả row group. External vector index chỉ là derived index và phải nhận delete/update qua CDF để không vi phạm lifecycle.

### NB8 — Agent trajectory và provenance

- Silver có `1,578` bước, partition theo `policy-v2` và `policy-v3`; Gold phủ cả hai policy.
- Replay ở version đã pin đọc đúng `1,578` bước.
- 5 lượt `list_tables` chỉ gây 1 catalog read.
- Lệnh phá hủy chưa xác nhận trả `input_required`; task mô phỏng poll đến hoàn tất.
- Có đủ bốn bucket trainable (`licensed`, `public_domain`, `scraped_optout_checked`, `synthetic`) và partition `UNCLASSIFIED`; `334` dòng UNCLASSIFIED bị loại.

Diễn giải: version pin là điều kiện tối thiểu cho reproducibility; cache giảm round-trip catalog. Lớp MCP và bucket provenance ở đây chỉ là mô phỏng offline, không phải authorization boundary hay chứng nhận pháp lý.

## Việc còn lại trước khi nộp

1. Tự chụp ảnh theo [screenshots/README.md](screenshots/README.md).
2. Xác nhận cách viết họ tên trong `INFO.md`.
3. Commit/push, mở PR và gửi repo URL + PR URL + commit SHA theo `docs/SUBMISSION.md`.
