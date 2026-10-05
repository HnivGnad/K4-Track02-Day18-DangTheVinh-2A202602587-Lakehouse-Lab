# Hướng dẫn tự chụp ảnh bằng chứng

Không cần chạy lại nếu output trong `submission/notebooks/` vẫn còn. Mở từng notebook bằng Jupyter Lab/VS Code, thu gọn cửa sổ để ảnh thấy cả tên notebook và output. Không chụp `.env`, token, đường dẫn chứa bí mật hoặc dữ liệu thật.

## Danh sách ảnh đề nghị

1. `nb01_delta_log.png`: chụp cell hiển thị tên/nội dung commit JSON trong `_delta_log/`. Chụp thêm `nb01_schema.png` có dòng `BLOCKED by schema enforcement` và các PASS về `tier` nếu một ảnh không chứa đủ.
2. `nb02_optimize.png`: chụp cùng khung có `Files before 200`, `Files after 55`, `Speedup 9.1×` và phần deliverable PASS.
3. `nb03_time_travel.png`: chụp history cuối có RESTORE, `Total versions: 5` và `score<0: 0`.
4. `nb04_medallion.png`: chụp `Bronze 200,000`, `Silver 190,052` và Gold metrics `8 dates × 3 models = 24 rows`. Nếu bảng Gold quá dài, ưu tiên phần tổng kết ngay dưới bảng.
5. `nb05_iceberg.png`: chụp pruning `10×`, field ID giữ nguyên và partition specs `[1, 2]`; có thể dùng hai ảnh nếu font quá nhỏ.
6. `nb06_maintenance.png`: chụp block PASS cuối. Nên chụp thêm một ảnh có `200 → 11`, skip `90%`, `Orphans found: 3`, checkpoint và `20 → 3 snapshots`.
7. `nb07_vectors.png`: chụp block PASS và các số `200×`, `5.8×`, recall `0.904`, fidelity `1.000`, lakehouse `0` hit/external index `8` hit.
8. `nb08_agents.png`: chụp block PASS; thêm output có replay `1,578`, `5 turns → 1 catalog read`, `input_required` và các provenance partitions nếu cần.

## Cách mở notebook

Từ PowerShell tại repo:

```powershell
$env:JUPYTER_DATA_DIR = "$PWD\.jupyter\data"
$env:JUPYTER_CONFIG_DIR = "$PWD\.jupyter\config"
$env:JUPYTER_RUNTIME_DIR = "$PWD\.jupyter\runtime"
$env:IPYTHONDIR = "$PWD\.jupyter\ipython"
.\.venv\Scripts\python.exe -m jupyter lab --notebook-dir=submission/notebooks --no-browser
```

Sau khi chụp, lưu đúng tên vào thư mục này và kiểm tra ảnh mở được trước khi commit.

