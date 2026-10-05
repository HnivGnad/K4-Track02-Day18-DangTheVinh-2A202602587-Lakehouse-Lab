# Reflection

Anti-pattern tôi quan tâm nhất là **small-file problem**. Một pipeline observability dễ ghi mỗi micro-batch thành nhiều file nhỏ để giảm độ trễ, nhưng đổi lại metadata phình to, query phải mở quá nhiều object và cold start ngày càng chậm. Kết quả NB2 và NB6 làm rủi ro này rất rõ: compaction giảm 200 xuống 55 file ở bài tối ưu, và 200 xuống 11 file ở bài maintenance; file skipping sau clustering đạt 90%. Vì vậy, với hệ thống tôi thiết kế, writer phải có mục tiêu kích thước file, giới hạn số partition theo cardinality thấp, và theo dõi đồng thời số file, kích thước trung vị cùng chi phí GET/LIST. `OPTIMIZE` chỉ là lưới an toàn; sửa cấu hình writer và lịch micro-batch mới là biện pháp gốc. Retention, checkpoint và orphan sweep cũng phải chạy theo lịch có cảnh báo, không dùng `VACUUM 0` ngoài dữ liệu scratch.

Phạm vi hỗ trợ AI được khai báo tại [AI_USAGE.md](AI_USAGE.md). Tôi chịu trách nhiệm kiểm tra lại mã, output và phần giải thích trước khi nộp.
