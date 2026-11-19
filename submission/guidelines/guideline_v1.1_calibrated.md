# Label contract — Phố châu Á — Bổ sung v1.1 — 2A202603043 - Lê Việt Long

## 1. Câu hỏi mở đã chốt

| ID | Nguồn | Câu hỏi và bằng chứng | Quy tắc đã chốt | Lý do |
|---|---|---|---|---|
| CAL-02 | cau_hoi_mo | Mục 6.7 v1.0 hỏi có nên dùng confidence để chấp nhận box. Ở img_012.jpg, r05 gán car điểm 0.8111 cho một xe có nhãn Gold three_wheeler. Bản merge cũng có box three_wheeler điểm 0.1936 khớp Gold (IoU 0.79). | Dùng confidence để lọc đầu vào. Trước khi giữ box, kiểm lớp và các cạnh theo ảnh ở mọi mức điểm. | Hai ví dụ cho thấy cần đối chiếu điểm với lớp và hình học thực tế. |

## 2. Ca khó từ lỗi máy và bẫy lọt

| ID | Nguồn | Ảnh và vấn đề | Cách xử lý đã chốt | Quy tắc áp dụng |
|---|---|---|---|---|
| CAL-01 | loi_may | img_012.jpg, r05 gán car cho một tuk-tuk, IoU với nhãn Gold three_wheeler là 0.917. | Gán three_wheeler khi nhận diện rõ xe ba bánh. Khi chưa chắc lớp, chọn lớp tin nhất và đánh dấu can_review. | v1.0 mục 4.3. |
| CAL-03 | bay_lot | img_029.jpg, review_stats.md ghi bẫy loose lọt lần đầu. | Rà đủ bốn cạnh theo phần vật thể nhìn thấy và chỉnh các cạnh bị lỏng. | v1.0 mục 4.1 khoản 1–2. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt cách dùng confidence khi duyệt box và dẫn lại hai quy tắc v1.0 cho các ca sai lớp, box lỏng. | CAL-02, CAL-01, CAL-03 |
