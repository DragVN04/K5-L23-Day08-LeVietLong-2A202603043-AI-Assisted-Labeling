# Thông tin học viên

- Họ và tên: Lê Việt Long
- MSHV: 2A202603043
- Lớp: L2
- GitHub: DragVN04
- Repo hiện tại: [K5-L23-Day08-LeVietLong-2A202603043-AI-Assisted-Labeling](https://github.com/DragVN04/K5-L23-Day08-LeVietLong-2A202603043-AI-Assisted-Labeling)
- Commit chốt bài: **CẦN XÁC NHẬN** sau khi chốt và commit.
- Máy: Windows 11, NVIDIA GeForce RTX 3050 Ti Laptop GPU (4 GiB). Log model ghi thiết bị chạy là CPU.
- CVAT Docker: **CẦN XÁC NHẬN** phiên bản.

## Kết quả tóm tắt

- Gold 4 ảnh, 46 đối tượng. Batch đã xuất 24 ảnh, 492 đối tượng. Nguồn [Gold](submission/labels/gold_manual.json) và [Batch](submission/labels/batch_reviewed.json).
- Accepted 419/519 = 80.7%, edited 28/519 = 5.4%, deleted 72/519 = 13.9%, added 34/481 = 7.1%. Profile loại vùng quanh bẫy. Nguồn [review stats](submission/qc/review_stats.md).
- Máy trên Gold đạt precision 41/74 = 55.4% và recall 41/46 = 89.1%. Bắt bẫy lần đầu 9/12 = 75.0%. Nguồn [review stats](submission/qc/review_stats.md).
- Gán tay Gold mất 30 phút/4 ảnh = 7,5 phút/ảnh. Duyệt Batch mất 90 phút/24 ảnh = 3,75 phút/ảnh. Nguồn [time log](submission/time_log.csv).
- QC ghi nhận 0/5 = 0.0% ảnh có lỗi và 0/100 = 0.0% đối tượng lỗi. Cận trên Wilson 95% theo ảnh là 43%. Nguồn [QC tự động](submission/qc/qc_report_auto.md).
- Quyết định cuối: **Rework**, ngày 2026-11-19. Lần đầu bắt 9/12 = 75.0% bẫy, dưới ngưỡng 80%. Nguồn [báo cáo](submission/REPORT.md) và [QC tự động](submission/qc/qc_report_auto.md).
