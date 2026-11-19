# Báo cáo Day 08 — 2A202603043 - Lê Việt Long

## 1. Model và recipe (Part 2–3)

| Lớp | Model và run | Ngưỡng | Kết quả quét trên Gold |
|---|---|---|---|
| person | YOLO26m · r05 | 0.3 | Recall tốt nhất với ngưỡng ≤ 0.5 là 14/16 (87.5%), dưới mục tiêu 90%. |
| bicycle | YOLO26m · r05 | 0.3 | 4 vật thể Gold, dưới mức tối thiểu 5 để chọn ngưỡng riêng. |
| motorbike | YOLO26m · r05 | 0.3 | Recall tốt nhất với ngưỡng ≤ 0.5 là 9/11 (81.8%). |
| car | YOLO26m · r05 | 0.3 | Recall tốt nhất với ngưỡng ≤ 0.5 là 6/7 (85.7%). |
| bus | YOLO26m · r05 | 0.3 | 1 vật thể Gold, dưới mức tối thiểu 5. |
| truck | YOLO26m · r05 | 0.3 | 0 vật thể Gold, dưới mức tối thiểu 5. |
| three_wheeler | YOLOE-26s · r06 | 0.15 | Recall 7/7 (100%), precision 7/9 (77.8%). |

Nguồn ngưỡng và số đo gồm [bảng quét](qc/threshold_sweep.md), [CSV](qc/threshold_sweep.csv) và [biểu đồ](qc/threshold_sweep.png). Theo [model log](model_log.csv), r05 và r06 chạy trên 36 ảnh ở confidence 0.05. r06 dùng prompt mặc định trong [taxonomy](taxonomy.json) cho lớp xe ba bánh gồm tuk-tuk, cyclo và cargo tricycle.

```bash
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

- Lệnh trên được tái chạy với đường dẫn đầu ra tạm và cho tệp trùng SHA-256 với bản merge hiện có, gồm 606 box trên 28 ảnh.
- Ở img_012.jpg, box YOLO26m gán car với confidence 0.8111 chồng lên nhãn Gold three_wheeler với IoU 0.917. Đổi ngưỡng chỉ giữ hoặc bỏ box. Recipe ưu tiên box YOLOE khi trùng giữa hai nguồn ở IoU ≥ 0.6, sau đó người duyệt kiểm lớp theo ảnh. Cùng ảnh có box three_wheeler điểm 0.1936 với IoU 0.79 so với Gold. Nguồn gồm pre-label r05, bản merge và [Gold](labels/gold_manual.json).
- Run r04 dùng SAM 2.1 Base (sam2.1_b.pt) theo cách box → mask, tạo 16 mask trên 2 ảnh, 5.36 giây/ảnh trên CPU. **CẦN XÁC NHẬN** lý do chọn phiên bản về chất lượng, tốc độ, RAM và quyền truy cập từ trải nghiệm thực tế. Nguồn [model log](model_log.csv) và work/runs.csv.
- Cùng 4 ảnh Gold được dùng để chọn ngưỡng và đánh giá. Precision và recall có thể lạc quan, còn ngưỡng có thể thay đổi nhiều khi thêm ảnh.

## 2. AI error profile (từ qc/review_stats.md)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 419 | 419/519 = 80.7% | Box máy |
| edited | 28 | 28/519 = 5.4% | Box máy |
| deleted | 72 | 72/519 = 13.9% | Box máy |
| added | 34 | 34/481 = 7.1% | Đối tượng sau duyệt |

- Profile tính trên 24 ảnh và đã loại 24 đối tượng quanh các bẫy. Script ghi nhận 8 trường hợp đổi lớp, nên số vật bị máy bỏ sót ước tính là 34 − 8 = 26. Đây là số đo trên tập được lọc theo vùng bẫy. COCO Batch chứa 492 annotation. Nguồn [review stats](qc/review_stats.md) và [Batch](labels/batch_reviewed.json).
- three_wheeler có tỉ lệ giữ nguyên thấp nhất theo lớp, 11/43 = 25.6%. Trong 43 box máy của lớp này có 8 box sửa và 24 box xóa, tương ứng 24/43 = 55.8% bị xóa. Sau duyệt có thêm 3 box lớp này. Ví dụ sai lớp ở img_012.jpg minh họa lỗi detector gốc, còn nguyên nhân của từng box bị xóa cần đối chiếu riêng. Nguồn [review stats](qc/review_stats.json) và r05.
- Trên 4 ảnh Gold, precision của máy là 41/74 = 55.4% (KTC 95% 44–66%) và recall là 41/46 = 89.1% (KTC 95% 77–95%). Có 41 TP, 33 FP và 5 FN. Nguồn [review stats](qc/review_stats.md).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Lần đầu bắt 9/12 = 75.0% (KTC 95% 47–91%), dưới ngưỡng 80%, tương ứng cần ít nhất 10/12. Kết quả theo loại là wrong_class 2/3 (66.7%), loose 2/3 (66.7%), missing 2/3 (66.7%) và extra 3/3 (100%). Nguồn [review stats](qc/review_stats.md).
- Ba bẫy lọt gồm img_034.jpg (wrong_class), img_029.jpg (loose) và img_018.jpg (missing). **CẦN XÁC NHẬN** lúc duyệt bạn đã kiểm đặc điểm phân lớp ở img_034, rà từng cạnh box ở img_029 và quét ảnh khi ẩn nhãn máy ở img_018 chưa.
- Accepted 419/519 = 80.7% và bắt bẫy 9/12 = 75.0% đo hai việc khác nhau. Học viên đối chiếu thao tác thực tế để xác định có bỏ bước nào hay không. Chưa có số liệu vòng rework. Nếu sửa sau khi biết đáp án, ghi là sửa có hướng dẫn. Nguồn [review stats](qc/review_stats.md).

## 4. Máy có đáng dùng không?

- **CẦN XÁC NHẬN** tổng phút thực đo cho gold_manual (4 ảnh) và batch_review (24 ảnh) trong [time log](time_log.csv). Phút/ảnh lần lượt là tổng phút Gold chia 4 và tổng phút Batch chia 24.
- **CẦN XÁC NHẬN** kết luận cá nhân sau khi có thời gian thực đo. Đối chiếu accepted 419/519 = 80.7% với 72 box phải xóa, 28 box phải chỉnh và 34 đối tượng phải thêm. Gold và Batch có thể khác độ khó và mật độ vật thể. Khi sang Batch, mức quen guideline và CVAT cũng có thể khác. Nguồn [review stats](qc/review_stats.md).
- **CẦN XÁC NHẬN** ba quy tắc giảm thiên kiến do học viên chọn. Có thể cân nhắc cách ghi nhận đã quét ảnh khi ẩn box, cách kiểm tra nhãn lớp và cách rà cạnh box hoặc vật sót. Với mỗi quy tắc, ghi cách kiểm tra việc thực hiện.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM | Click | Vùng mask và lỗi quan sát | Tệp |
|---|---|---|---|---|
| img_013.jpg | **CẦN XÁC NHẬN** checkpoint của lần click | 3 dấu trên ảnh (2 dương, 1 âm) | Mask phủ mui và thân xe, lan sang một phần người lái. Chưa xác nhận phần hàng hóa. | [tuktuk_img013.jpg](masks/tuktuk_img013.jpg) |

- Run r04 xác nhận SAM 2.1 Base cho box → mask. Phiên bản của ảnh click cần đối chiếu với lệnh hoặc log Part 5. **CẦN XÁC NHẬN** và bổ sung 2 vật khác, mỗi vật có ảnh mask, phiên bản, số click và nhận xét.
- Guideline v1.0 mục 4.2 yêu cầu bám phần nhìn thấy, xử lý lỗ lớn và tách polygon ở vùng tiếp giáp. Với img_013.jpg, đối chiếu riêng vùng người lái và khoảng hở dưới mui. Câu hỏi mở về kính nằm ở mục 6.5 của guideline.

## 6. Calibration (Part 6)

- Có 0 quyết định đã chốt trong [calibration_decisions.csv](qc/calibration_decisions.csv). Ba ca nháp và bằng chứng nằm trong [guideline v1.1](guidelines/guideline_v1.1_calibrated.md).
- **CẦN XÁC NHẬN** nguyên nhân từng ca, gồm lỗi detector hoặc thao tác review, và quy tắc nào thực sự cần bổ sung. Sau khi chốt, ghi mỗi quyết định vào CSV và đồng bộ quy tắc mới vào v1.1.

## 7. QC và quyết định (Part 6)

- Release artifact hiện có 24 ảnh, 492 đối tượng, chưa có ảnh được thay bằng bản QC fix theo [manifest](release/release_manifest.json).
- Mẫu gồm 5 ảnh ngẫu nhiên, 0 ca can_review, tổng 100 đối tượng. Danh sách khớp phương pháp mặc định với seed 42. Nếu học viên tự QC lô mình đã review, hai lần kiểm có thể lặp lại cùng điểm mù. Nguồn [mẫu QC](qc/qc_sample.csv) và scripts/qc_sample.py.
- Phiếu còn trống 5/5 ảnh. Bản nháp script đếm 0/100 = 0.0% đối tượng và 0/5 = 0.0% ảnh có lỗi được nhập. **CẦN XÁC NHẬN** lỗi thực tế và các ví dụ sau khi kiểm img_003.jpg, img_006.jpg, img_014.jpg, img_031.jpg và img_035.jpg. Cận trên Wilson 95% trong báo cáo là 43% cho trường hợp quan sát 0/5 ảnh lỗi. Ngay cả khi cả năm ảnh được xác nhận sạch, mẫu này vẫn quá nhỏ để chứng minh ngưỡng 2%. Nguồn [phiếu QC](qc/qc_review.csv) và [QC tự động](qc/qc_report_auto.md).
- Script kiểm cấu trúc, phạm vi ảnh và đếm lỗi đã nhập. Mức độ ôm sát, lớp đúng trong ảnh và vật cả người lẫn máy bỏ sót cần kiểm bằng mắt.
- **Quyết định cuối và ngày — CẦN XÁC NHẬN.** Xét Hold trước nếu guideline còn mơ hồ ảnh hưởng nhãn hoặc dữ liệu hỏng. Xét Rework khi vượt ngưỡng ảnh lỗi, lỗi lặp hoặc bắt bẫy dưới chuẩn. Release cần QC hoàn tất và đạt các điều kiện của lab. Bản nháp script gợi ý Rework vì lần đầu bắt 9/12 = 75.0% và còn ảnh chưa kiểm. Nguồn [hướng dẫn QC](../docs/CALIBRATION_QC.md) và [QC tự động](qc/qc_report_auto.md).
- Mở ảnh tại work/qc/review, ghi mỗi lỗi một dòng với người kiểm và ghi chú. Ảnh đã kiểm sạch ghi khong. Bỏ sót chưa có ID thì để trống object_id và mô tả trong note. Hoàn thành phiếu rồi chạy python scripts/qc_report.py.

## 8. Ghi chú cá nhân

- Dùng ChatGPT/Codex để đối chiếu số liệu, giải thích script và soạn báo cáo. Số được kiểm với đầu ra script, log và nhãn xuất. Học viên kiểm tra nhãn, nhận xét và tự xác nhận kết luận cuối.
- Môi trường học viên cung cấp gồm Windows 11, Git Bash, CVAT Docker và Python trong venv. Máy có GeForce RTX 3050 Ti Laptop GPU (4 GiB), còn work/runs.csv ghi các lần chạy trên CPU.
- **CẦN XÁC NHẬN** phiên bản CVAT, việc có xem pre-label hoặc hỏi AI trước khi hoàn tất Gold, việc có dùng cache ở bước nào, vướng mắc khi chạy lab và một thay đổi học viên muốn áp dụng lần sau.
