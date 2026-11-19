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

- Gán tay Gold mất 30 phút/4 ảnh = 7,5 phút/ảnh. Duyệt Batch có pre-label mất 90 phút/24 ảnh = 3,75 phút/ảnh. Nguồn [time log](time_log.csv).
- Pre-label có ích để gán nhãn sơ bộ. Batch mất 3,75/7,5 = 50% thời gian mỗi ảnh so với Gold trong lần thực hiện này. Hai nhóm có thể khác về độ khó và mật độ vật thể, và mức độ quen guideline, CVAT khi sang Batch cũng có thể ảnh hưởng thời gian. Tôi vẫn cần duyệt kỹ, đặc biệt với xe ba bánh vì chỉ 11/43 = 25,6% box của lớp này được giữ nguyên. Nguồn [time log](time_log.csv) và [review stats](qc/review_stats.md).
- Ba quy tắc giảm thiên kiến áp dụng khi duyệt:
  1. Ẩn nhãn máy, quét toàn ảnh để tìm vật bị bỏ sót, rồi đánh dấu đã quét trước khi bật lại nhãn.
  2. Trước khi giữ từng box, đối chiếu lớp với đặc điểm nhìn thấy của vật ở mọi mức confidence.
  3. Rà đủ bốn cạnh theo phần vật thể nhìn thấy và chỉnh cạnh bị lỏng trước khi giữ box.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM | Click | Vùng mask và lỗi quan sát | Tệp |
|---|---|---|---|---|
| img_013.jpg, xe có người lái | SAM 2.1 Base | 3 (2 dương, 1 âm) | Mask phủ mui và thân xe, lan sang một phần người lái. Chưa xác định được phần hàng hóa từ ảnh. | [tuktuk_img013_sam21.jpg](masks/tuktuk_img013_sam21.jpg) |
| img_004.jpg, xe xanh lá | SAM 2.1 Base | 4 (2 dương, 2 âm) | Mask phủ mui, kính và thân xe. Người đứng bên phải ở ngoài mask. Biên quanh các khe ở hông xe còn gồ ghề. | [tuktuk_xanh_la_img004.jpg](masks/tuktuk_xanh_la_img004.jpg) |
| img_004.jpg, xe xanh dương | SAM 2.1 Base | 4 (2 dương, 2 âm) | Mask phủ mui, thân và bánh xe. Người đứng bên phải ở ngoài mask. Mảng vật sáng ở hông xe bị tô một phần và vùng ghế có các lỗ nhỏ. | [tuktuk_xanh_duong_img004.jpg](masks/tuktuk_xanh_duong_img004.jpg) |

- Checkpoint `sam2.1_b.pt` lấy từ lệnh Part 5 dùng để tạo ba ảnh trên. Số điểm được đối chiếu với chấm xanh và đỏ trên ảnh đầu ra.
- Theo guideline v1.0 mục 4.2, mask cần bám biên phần nhìn thấy, để hở lỗ lớn và tách các vật tiếp giáp. Vùng mask lấn sang người lái trong img_013.jpg cần tách khỏi mask xe. Các lỗ nhỏ trên vùng ghế trong img_004.jpg cần đối chiếu tiêu chí diện tích của guideline.

## 6. Calibration (Part 6)

- Đã chốt 3 quyết định trong [calibration_decisions.csv](qc/calibration_decisions.csv): CAL-01 xử lý sai lớp xe ba bánh, CAL-02 chốt cách dùng confidence và CAL-03 chốt thao tác kiểm box lỏng.
- CAL-02 giải quyết câu hỏi mở 6.7. Dùng confidence để lọc đầu vào và kiểm lớp, các cạnh box bằng mắt trước khi giữ. Quy tắc được ghi tại mục 1 của [guideline v1.1](guidelines/guideline_v1.1_calibrated.md).
- CAL-01 là lỗi phân lớp của detector và áp dụng mục 4.3 của v1.0. CAL-03 là bẫy box lỏng lọt trong lần duyệt đầu và áp dụng mục 4.1 của v1.0.

## 7. QC và quyết định (Part 6)

- Release artifact hiện có 24 ảnh, 492 đối tượng, chưa có ảnh được thay bằng bản QC fix theo [manifest](release/release_manifest.json).
- Mẫu gồm 5 ảnh ngẫu nhiên, 0 ca can_review, tổng 100 đối tượng. Danh sách khớp phương pháp mặc định với seed 42. Nếu học viên tự QC lô mình đã review, hai lần kiểm có thể lặp lại cùng điểm mù. Nguồn [mẫu QC](qc/qc_sample.csv) và scripts/qc_sample.py.
- Lê Việt Long kiểm và xác nhận cả 5 ảnh mẫu sạch. Kết quả là 0/100 = 0.0% đối tượng lỗi và 0/5 = 0.0% ảnh có lỗi, nên mẫu không có loại lỗi để xếp hạng. Cận trên Wilson 95% cho 0/5 ảnh lỗi là 43%. Mẫu quá nhỏ để chứng minh tỉ lệ lỗi theo ảnh của cả lô đạt ngưỡng 2%. Nguồn [phiếu QC](qc/qc_review.csv) và [QC tự động](qc/qc_report_auto.md).
- Script kiểm cấu trúc, phạm vi ảnh và đếm lỗi đã nhập. Mức độ ôm sát, lớp đúng trong ảnh và vật cả người lẫn má  y bỏ sót cần kiểm bằng mắt.
- **Quyết định đã chốt: Rework, ngày 2026-11-19.** Hai tệp nhãn qua kiểm tra cấu trúc và ba ca calibration đã được chốt. Mẫu QC ghi nhận 0/5 = 0.0% ảnh lỗi. Tỉ lệ bắt bẫy lần đầu là 9/12 = 75.0%, dưới ngưỡng 80%, nên lô cần duyệt lại. Nguồn [hướng dẫn QC](../docs/CALIBRATION_QC.md) và [QC tự động](qc/qc_report_auto.md).
- Lượt Rework sẽ áp dụng checklist đã chốt, rà lại lớp xe, biên box và vật bị bỏ sót. Xuất các ảnh đã sửa vào `qc_fixed.json` nếu có thay đổi, dựng lại dữ liệu đầu ra và kiểm QC sau sửa. Số bắt bẫy lần đầu được giữ làm kết quả của vòng duyệt ban đầu.

## 8. Ghi chú cá nhân

- Dùng ChatGPT/Codex để đối chiếu số liệu, giải thích script và soạn báo cáo. Số được kiểm với đầu ra script, log và nhãn xuất. Học viên kiểm tra nhãn, nhận xét và tự xác nhận kết luận cuối.
- Môi trường học viên cung cấp gồm Windows 11, Git Bash, CVAT Docker và Python trong venv. Máy có GeForce RTX 3050 Ti Laptop GPU (4 GiB), còn work/runs.csv ghi các lần chạy trên CPU.
- **CẦN XÁC NHẬN** phiên bản CVAT, việc có xem pre-label hoặc hỏi AI trước khi hoàn tất Gold, việc có dùng cache ở bước nào, vướng mắc khi chạy lab và một thay đổi học viên muốn áp dụng lần sau.
