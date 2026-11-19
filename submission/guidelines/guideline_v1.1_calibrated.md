# Label contract — Phố châu Á — Bổ sung v1.1 — 2A202603043 - Lê Việt Long

**Bản nháp chờ học viên xác nhận.** v1.0 hiện là bản áp dụng.

## 1. Câu hỏi mở

| ID | Nguồn | Câu hỏi và bằng chứng | Đề xuất để duyệt | Lý do và trạng thái |
|---|---|---|---|---|
| CAL-02 | cau_hoi_mo | Mục 6.7 v1.0 hỏi có nên dùng confidence để chấp nhận box. Ở img_012.jpg, r05 gán car điểm 0.8111 cho một xe có nhãn Gold three_wheeler. Bản merge cũng có box three_wheeler điểm 0.1936 khớp Gold (IoU 0.79). | Dùng confidence để lọc đầu vào. Khi duyệt, kiểm lớp và cạnh box theo ảnh ở mọi mức điểm. | Điểm số trong hai ví dụ không đủ để quyết định giữ nguyên. **CẦN XÁC NHẬN** cách ghi vào checklist. |

## 2. Ca khó từ lỗi máy và bẫy lọt

| ID | Nguồn | Ảnh và vấn đề | Đề xuất để duyệt | Lý do và trạng thái |
|---|---|---|---|---|
| CAL-01 | loi_may | img_012.jpg, r05 gán car cho một tuk-tuk, IoU với nhãn Gold three_wheeler là 0.917. | Áp dụng định nghĩa three_wheeler và mục 4.3 hiện hành. Kiểm đặc điểm xe khi car/truck chồng box YOLOE và đánh dấu can_review khi hình ảnh chưa đủ rõ. | v1.0 đã có lớp đúng và cách xử lý ca không chắc. **CẦN XÁC NHẬN** thao tác review cần bổ sung. |
| CAL-03 | bay_lot | img_029.jpg, review_stats.md ghi bẫy loose lọt lần đầu. | Rà lần lượt bốn cạnh theo phần xe nhìn thấy và xác nhận tiêu chí biên ở mục 4.1 trước khi giữ box. | Đây là bước áp dụng guideline hiện có. **CẦN XÁC NHẬN** bước đã bỏ sót và cách ghi dấu đã kiểm. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | **CẦN XÁC NHẬN**. Chờ duyệt ba ca nháp. | CAL-01, CAL-02, CAL-03 |

Sau khi duyệt, điền quyết định vào calibration_decisions.csv và ghi các quy tắc mới đã chốt vào mục tương ứng. Quy trình review có thể dẫn lại quy tắc v1.0 đang áp dụng.
