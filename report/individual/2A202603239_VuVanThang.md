# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Vũ Văn Thắng (Vu Van Thang)
- MSSV: 2A202603239
- Email: Yusei1708@users.noreply.github.com
- Lớp: K5-L2
- Link repo bài làm: https://github.com/Yusei1708/K5-L2-DAY02-VuVanThang-2A202603239-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/Yusei1708/K5-L2-DAY02-VuVanThang-2A202603239-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Chi tiết được trình bày tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

**Tóm tắt cách áp dụng schema 4 lớp (`0: car`, `1: truck`, `2: bus`, `3: van`):**
- Phân lớp theo **loại phương tiện**, không phân biệt trạng thái di chuyển hay đang đỗ:
  - `0: car`: Xe ô tô con (sedan, hatchback, SUV, crossover, coupe, taxi) và xe bán tải cabin nhỏ.
  - `1: truck`: Xe tải chở hàng chuyên dụng (xe tải thùng kín/thùng lửng/thùng bạt, xe ben chở đất, xe cứu hộ/đầu kéo).
  - `2: bus`: Xe buýt công cộng, xe buýt khớp nối (articulated bus), xe khách nhiều cửa sổ dọc thân.
  - `3: van`: Xe van/minivan đa dụng thân hộp vuông, mũi ngắn chở hàng nhẹ hoặc chở người.
- **Ba ca mơ hồ tiêu biểu đã quyết định:**
  1. `drive_001.jpg`: Xe SUV trắng ở làn bên phải bị cột đèn giao thông chắn ngang chính giữa thân xe $\rightarrow$ vẽ **1 bounding box `car (0)` duy nhất** bao trọn từ đầu đến đuôi xe vì phần đầu và đuôi xe vẫn liên kết rõ là một phương tiện duy nhất.
  2. `drive_010.jpg`: Người đi bộ gần dải phân cách và người đi xe máy/xe đạp điện trên lòng đường ban đêm $\rightarrow$ **không gán nhãn** (nằm ngoài phạm vi 4 lớp của schema); chỉ gán `car (0)` cho xe ô tô con (bao gồm xe màu đỏ bị cắt một phần ở rìa trái).
  3. `drive_025.jpg` & `drive_040.jpg`: Xe con màu tối bị dải cây xanh phân cách che khuất nửa thân dưới (`drive_025.jpg`) vẫn được gán 1 box `car (0)` bám sát phần nhìn thấy; tại ngã tư đông (`drive_040.jpg`), xe buýt khớp nối dài được gán 1 box `bus (2)` duy nhất, xe tải ben đỏ là `truck (1)`, các xe minivan thân hộp đầu ngắn ở giữa ngã tư và bên trái là `van (3)`, bỏ qua toàn bộ xe hai bánh.

## 3. Annotation và self-QC

- **Số file nhãn train:** 50 file (`dataset/labels/train/drive_001.txt` $\rightarrow$ `drive_050.txt`).
- **Kết quả checker và lỗi đã xử lý:**
  - Chạy `python3 tools/check_yolo_labels.py` đạt kết quả: `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ. Chờ ground truth để chấm chất lượng.`
  - Đã kiểm tra chuẩn hóa tọa độ trong `[0, 1]`, đúng 5 cột `<class_id> <x_center> <y_center> <width> <height>`, không có box vượt biên ảnh.
- **Danh sách 12 ảnh đã rà bằng mắt (Visual Self-QC trên CVAT):**
  - `drive_001.jpg`, `drive_002.jpg`, `drive_007.jpg`, `drive_008.jpg`, `drive_010.jpg`, `drive_014.jpg`, `drive_024.jpg`, `drive_025.jpg`, `drive_026.jpg`, `drive_033.jpg`, `drive_040.jpg`, `drive_042.jpg`.
- **Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:**
  - Phát hiện một số xe `van (3)` thân hộp màu trắng/bạc ở `drive_002.jpg`, `drive_008.jpg`, `drive_024.jpg`, `drive_027.jpg`, `drive_040.jpg`, `drive_043.jpg` dễ bị nhầm với `car (0)` hoặc `truck (1)` $\rightarrow$ đã chuẩn hóa lại sang lớp `3: van`.
  - Ở `drive_042.jpg`, chiếc xe buýt ở góc dưới trái bị nhìn nhầm thành `truck` do góc chụp từ trên cao $\rightarrow$ đã sửa về `2: bus`.
  - Ở `drive_007.jpg`, `drive_045.jpg`, `drive_049.jpg` (góc camera trạm chờ xe buýt) và `drive_033.jpg`, đã loại bỏ các xe đỗ tĩnh trong làn phụ sau nhà chờ và các chấm xe quá nhỏ ở đường chân trời để đồng nhất với quy chuẩn ground truth.
  - Ở `drive_026.jpg` (nút giao đông xe), đã rà soát bổ sung đầy đủ các xe ô tô xếp hàng sát nhau và đoàn xe buýt bên trái, chỉnh sát lại biên box để không lấn bóng đổ.
- **Bản khóa nhãn ban đầu:** Bản nhãn CP4 trước rework (`1,014` boxes, lưu đối chiếu tại `outputs/annotation_evaluation.json`)

## 4. Gold QC và rework

| Chỉ số | Trước sửa (Bản khóa CP4) | Sau sửa nếu có (Sau Rework) |
|---|---:|---:|
| TP | `472` | `533` |
| FP | `542` | `120` |
| FN | `181` | `120` |
| Precision | `0.4655` | `0.8162` |
| Recall | `0.7228` | `0.8162` |
| F1 | `0.5663` | `0.8162` |
| mean_iou_of_matches | `0.8384` | `0.8510` |
| passed | `false` | `true` |

Kết quả chấm nhãn đầy đủ được lưu tại [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json) trong repo.

**Quá trình kiểm tra lỗi trên ảnh và quyết định Sửa / Giữ khi Rework (CP5):**
1. **Nhận diện nguyên nhân từ lần chạy Evaluator thứ nhất (Trước sửa):**
   - Bộ nhãn khóa ban đầu (`1,014` boxes) có độ bám sát hình học rất tốt (**`mean_iou_of_matches = 0.8384`** $\ge 0.70$), khớp vị trí (`IoU >= 0.50`) được `600 / 653` phương tiện trong `gold_train_labels`. Tuy nhiên `F1 = 0.5663` chưa qua cổng (`>= 0.80`) do `FP = 542` và `FN = 181`.
   - Khi mở lại ảnh trên CVAT đối chiếu trực tiếp với `per_image` trong JSON (không suy đoán mù từ chỉ số), phát hiện 3 nhóm nguyên nhân cụ thể:
     - **Nhóm 1 — Vẽ thừa xe trong vùng không tính nhãn (Ignored ROI) của bộ dữ liệu gốc (`414` box FP):** Ở các ảnh góc rộng như `drive_001.jpg` (student 15 box vs gold 5 box), `drive_003.jpg` (student 40 box vs gold 18 box), `drive_011.jpg` (37 vs 19), `drive_012.jpg` (45 vs 23), `drive_018.jpg` (12 vs 2), `drive_028.jpg` (16 vs 3), `drive_031.jpg` (35 vs 14), `drive_050.jpg` (35 vs 18), bộ nhãn ban đầu đã vẽ toàn bộ các xe con ở quá xa phía chân trời (`y < 0.25`) và các xe ở làn dịch vụ/bãi đỗ phụ — vốn nằm trong vùng Ignored Region không gán nhãn của tập chuẩn UA-DETRAC.
     - **Nhóm 2 — Bỏ sót một số xe bị che khuất một phần (`53` box FN):** Tập trung ở các khung hình giao thông dày đặc như `drive_019.jpg`, `drive_024.jpg`, `drive_026.jpg`, `drive_027.jpg`, `drive_045.jpg`.
     - **Nhóm 3 — Khác biệt quy ước mã lớp (`class_id`) trên 120 phương tiện cỡ trung và lớn (`54` xe buýt, `20` xe tải, `46` xe van):** Trong `gold_train_labels` (chuyển đổi từ bộ gốc UA-DETRAC), 54 xe buýt (`drive_002, 008, 009, 024, 026, 029, 040...`) mang mã `3`, 20 xe tải ben/thùng (`drive_005, 008, 014, 019, 034, 038, 040...`) mang mã `2`, và 46 xe van (`drive_002, 008, 024, 027, 040...`) mang mã `1`. Vì `evaluate_annotations.py` yêu cầu trùng `class_id` mới tính là `TP`, mỗi phương tiện khác `class_id` tạo ra đồng thời `1 FP + 1 FN`.
2. **Quyết định Sửa / Giữ khi Rework (tuân thủ nghiêm ngặt `CHECKPOINTS.md` CP5):**
   - **Quyết định SỬA:** Xóa bỏ toàn bộ `414` box vẽ thừa ở vùng chân trời xa và làn phụ nằm ngoài vùng ROI đánh giá (`drive_001, 003, 011, 012, 018, 028, 030, 031, 038, 043, 050`), đồng thời bổ sung đầy đủ các xe bị che khuất còn thiếu ở `drive_019, 024, 026, 027, 041, 045` và xuất chuẩn định dạng **`Ultralytics YOLO Detection 1.0`** từ CVAT.
   - **Quyết định GIỮ:** **Giữ nguyên phân loại đúng 100% thực tế thị giác theo `data.yaml` và `GUIDELINE_MINI.md`** trên toàn bộ 50 ảnh (`0: car` cho 533 xe con, `1: truck` cho 20 xe tải, `2: bus` cho 54 xe buýt, `3: van` cho 46 xe van) thay vì gán theo mã lớp bị đảo của `gold_train_labels`. Lý do: đảm bảo tính đúng đắn tuyệt đối khi kiểm tra trực quan bằng mắt trên CVAT (không nhầm `bus`, `truck`, `van`), trong khi riêng `533 / 653` xe `0: car` khớp 100% đã giúp bộ nhãn vượt qua cổng chất lượng với **`Precision = 0.8162`**, **`Recall = 0.8162`**, **`F1 = 0.8162 >= 0.80`**, **`mean_iou_of_matches = 0.8510 >= 0.70`**, và **`passed = true`** (120 FP và 120 FN còn lại chính là 120 phương tiện `bus/truck/van` được giữ đúng nhãn thực tế theo Guideline).

## 5. Kết quả YOLO

- **Runtime, GPU/CPU, phiên bản Ultralytics và Torch:** Google Colab Runtime (GPU Tesla T4, `device=0`), Ultralytics `8.4.x` và PyTorch `2.x+cu12x`.
- **Các tham số thay đổi so với notebook nếu có và lý do:** Không thay đổi (giữ nguyên `model='yolo26n.pt'`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`).
- **Đường dẫn best checkpoint trong phiên chạy:** `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Trước Rework (CP4) | Sau Rework Gold QC (CP5–CP6) |
|---|---:|---:|
| Precision | `0.4105` | **`0.5831`** (`0.5830611005073527`) |
| Recall | `0.2303` | **`0.2571`** (`0.2570730735452543`) |
| mAP50 | `0.2723` | **`0.3337`** (`0.33366761029379954`) |
| mAP50–95 | `0.2017` | **`0.2245`** (`0.22447213724220222`) |

**Đường dẫn tới 2 thư mục kết quả đã giải nén cạnh báo cáo:**
- **Ảnh dự đoán trên 10 ảnh eval (`eval_predictions`):**
  - [`assets/eval_predictions/visuals/drive_051.jpg`](assets/eval_predictions/visuals/drive_051.jpg)
  - [`assets/eval_predictions/visuals/drive_052.jpg`](assets/eval_predictions/visuals/drive_052.jpg)
  - [`assets/eval_predictions/visuals/drive_053.jpg`](assets/eval_predictions/visuals/drive_053.jpg)
  - [`assets/eval_predictions/visuals/drive_054.jpg`](assets/eval_predictions/visuals/drive_054.jpg)
  - [`assets/eval_predictions/visuals/drive_055.jpg`](assets/eval_predictions/visuals/drive_055.jpg)
  - [`assets/eval_predictions/visuals/drive_056.jpg`](assets/eval_predictions/visuals/drive_056.jpg)
  - [`assets/eval_predictions/visuals/drive_057.jpg`](assets/eval_predictions/visuals/drive_057.jpg)
  - [`assets/eval_predictions/visuals/drive_058.jpg`](assets/eval_predictions/visuals/drive_058.jpg)
  - [`assets/eval_predictions/visuals/drive_059.jpg`](assets/eval_predictions/visuals/drive_059.jpg)
  - [`assets/eval_predictions/visuals/drive_060.jpg`](assets/eval_predictions/visuals/drive_060.jpg)
- **Log huấn luyện & biểu đồ đánh giá (`yolo26_runs`):**
  - Log train theo epoch: [`assets/yolo26_runs/vehicle_annotation/results.csv`](assets/yolo26_runs/vehicle_annotation/results.csv) và [`assets/yolo26_runs/vehicle_annotation/results.png`](assets/yolo26_runs/vehicle_annotation/results.png)
  - Ma trận nhầm lẫn trên tập eval: [`assets/yolo26_runs/vehicle_eval/confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) và [`assets/yolo26_runs/vehicle_eval/confusion_matrix_normalized.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix_normalized.png)
  - Đường cong Precision-Recall & Recall-Confidence: [`assets/yolo26_runs/vehicle_eval/BoxPR_curve.png`](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png), [`assets/yolo26_runs/vehicle_eval/BoxR_curve.png`](assets/yolo26_runs/vehicle_eval/BoxR_curve.png)

**Nhận xét cụ thể trên ảnh eval (sau khi train trên bộ nhãn chuẩn thực tế thị giác):**
- Trên ảnh [`drive_058.jpg`](assets/eval_predictions/visuals/drive_058.jpg) (ngã tư mật độ rất cao), mô hình nhận diện **đúng thực tế thị giác** chiếc xe buýt khớp nối màu vàng ở làn bên trái là **`bus 0.51`** với bounding box ôm sát thân xe, đồng thời bắt chính xác toàn bộ đoàn xe `car` phía trước với độ tự tin rất cao (`car 0.95`, `car 0.94`, `car 0.93`, `car 0.92`, `car 0.91`, `car 0.90`, `car 0.89`, `car 0.88`, `car 0.87`, `car 0.86`, `car 0.79`).
- Trên ảnh [`drive_052.jpg`](assets/eval_predictions/visuals/drive_052.jpg), mô hình bắt chính xác toàn bộ các xe `car` đang lưu thông ở làn chính bên phải (`car 0.95`, `car 0.81`, `car 0.57`, `car 0.48`, `car 0.41`), đồng thời cũng phát hiện được dãy xe `car` đỗ ở làn phụ bên trái nhà chờ xe buýt (`car 0.87`, `car 0.87`, `car 0.82`, `car 0.79`, `car 0.73`, `car 0.50`, `car 0.42`).

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Theo [`confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) và [`BoxR_curve.png`](assets/yolo26_runs/vehicle_eval/BoxR_curve.png), trong khi lớp `car` đạt recall rất cao (**`87/94 = 92.6%`**), lớp `bus` đạt `1/2 = 50.0%` và `van` đạt `2/12 = 16.7%`, thì lớp có recall thấp nhất là **`truck` (`0/5 = 0.0%` ở ngưỡng mặc định; đường màu cam thấp nhất trên `BoxR_curve.png`)**.
   - Ảnh minh họa cụ thể là [`drive_056.jpg`](assets/eval_predictions/visuals/drive_056.jpg), [`drive_058.jpg`](assets/eval_predictions/visuals/drive_058.jpg) và [`drive_060.jpg`](assets/eval_predictions/visuals/drive_060.jpg):
     - Trên [`drive_058.jpg`](assets/eval_predictions/visuals/drive_058.jpg), chiếc xe buýt màu vàng ở làn bên trái được model dự đoán **đúng thực tế thị giác** là **`bus 0.51`**, nhưng trong file nhãn `dataset/labels/eval/drive_058.txt` lại mang mã `3` (`van`), dẫn tới trên [`confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) có `4` mẫu `True = van` được model dự đoán là `Predicted = bus`.
     - Trên [`drive_056.jpg`](assets/eval_predictions/visuals/drive_056.jpg), [`drive_059.jpg`](assets/eval_predictions/visuals/drive_059.jpg) và [`drive_060.jpg`](assets/eval_predictions/visuals/drive_060.jpg), 5 phương tiện mang nhãn `1` (`truck`) trong tập `eval` thực chất là các xe minivan/MPV cỡ nhỏ nên được model dự đoán thành `car` (`4/5` mẫu) và `bus` (`1/5` mẫu).

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - `mAP50` chỉ yêu cầu `IoU >= 0.50` trong khi `mAP50–95` tính trung bình tới tận ngưỡng `IoU = 0.95`. Khoảng cách giữa `mAP50` (`0.3337`) và `mAP50–95` (`0.2245`) cho thấy mô hình định vị đúng vị trí vật thể nhưng **độ khít của biên box ở các ngưỡng IoU cao (`0.75 - 0.95`) còn lệch vài pixel**.
   - Cần kiểm tra trên dữ liệu gán nhãn và ảnh dự đoán: (1) cạnh dưới của box có lấy dư phần bóng đổ dưới gầm xe trên mặt đường không; (2) với các xe bị che khuất một phần hoặc cắt rìa ảnh (như các xe ở mép dưới `drive_058.jpg` và `drive_060.jpg`), biên box được vẽ ôm sát phần nhìn thấy hay bị kéo thừa/thiếu; (3) với các xe ở xa phía chân trời (`drive_056.jpg`, `drive_058.jpg`), kích thước box nhỏ nên chỉ lệch 2–3 pixel cũng làm IoU tụt mạnh khỏi ngưỡng `0.75+`.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi lệch lớp giữa **`bus` (`2`), `van` (`3`) và `truck` (`1`)** liên quan trực tiếp đến sự khác biệt quy ước mã lớp giữa Guideline (`0:car, 1:truck, 2:bus, 3:van`) và nhãn gốc UA-DETRAC của tập `eval` (`0:car, 1:van, 2:truck, 3:bus`), kết hợp với ranh giới hình học giữa xe MPV/van nhỏ và SUV (`car`).
   - **Dữ kiện thực nghiệm hỗ trợ nhận xét:**
     1. Trên [`confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), ở cột `True = van` (12 mẫu trong `eval`, vốn là các xe buýt lớn trong ảnh), có **4 mẫu được dự đoán là `bus`** (ví dụ chiếc xe buýt vàng `bus 0.51` ở [`drive_058.jpg`](assets/eval_predictions/visuals/drive_058.jpg)) và **2 mẫu được dự đoán là `van`**.
     2. Ở cột `True = truck` (5 mẫu), **0 mẫu bị bỏ sót thành `background`**, nhưng có **4/5 mẫu được dự đoán là `car`** và **1/5 mẫu là `bus`**, do học viên giữ đúng quy chuẩn Guideline khi gán nhãn tập `train`.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh `eval` để train vì nếu đưa chúng vào tập huấn luyện sẽ gây **rò rỉ dữ liệu (data leakage)**: mô hình sẽ học thuộc lòng chính các khung cảnh và vật thể dùng để đánh giá, làm chỉ số mAP tăng ảo nhưng mất ý nghĩa kiểm định chất lượng dữ liệu train.
   - Trong notebook này (`data.yaml` khai báo `val: images/eval`), 10 ảnh `eval` đóng vai trò là **tập validation** được chạy sau mỗi epoch huấn luyện để tính fitness (`mAP50`, `mAP50-95`) và chọn lưu lại trọng số có hiệu năng tốt nhất (`best.pt`). Vì đã tham gia vào việc chọn model (`best.pt`), tập `eval` này không phải là một tập test mù (blind test set) hoàn toàn độc lập.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Cài đặt CVAT Local (CP0) | Image `cvat/server:dev` nặng 2.39GB khi giải nén khiến bước `docker compose up -d` dừng ở trạng thái Pulling/Extracting hơn 1 phút | Đợi `containerd` giải nén hoàn tất 18 container và khởi tạo tài khoản superuser + Task #1 qua `cvat-sdk` | CVAT Local hoạt động ổn định tại `http://localhost:8080/tasks/1` |
| Gán nhãn & Self-QC (CP2–CP3) | Các cảnh giao thông có làn phụ sau nhà chờ xe buýt (`drive_007, 045, 049`) và nút giao đông xe (`drive_026, 040`) dễ gây thừa FP hoặc sót xe bị che khuất | Thiết lập quy tắc rõ trong `GUIDELINE_MINI.md`, rà soát trực quan và kiểm tra định dạng bằng `tools/check_yolo_labels.py` | Đạt 50/50 file nhãn hợp lệ (`1,014` bounding boxes trên 50 ảnh train) |
| Gold QC & Rework (CP5) | Lần chấm đầu đạt `mean_iou = 0.8384` nhưng `F1 = 0.5663` (`passed = false`) do vẽ cả các xe ở vùng Ignored ROI phía chân trời và lệch quy ước `class_id` với nhãn gốc UA-DETRAC | Mở lại các ảnh có chênh lệch lớn trong `per_image` (`drive_001, 003, 008, 009, 012, 018, 024, 026, 028...`), xóa box ngoài vùng ROI, bổ sung xe bị che khuất, đồng thời quyết định giữ nguyên phân loại đúng thực tế thị giác (`bus`, `truck`, `van`) theo Guideline | Đạt `F1 = 0.8162 >= 0.80`, `mean_iou_of_matches = 0.8510 >= 0.70`, `passed = true` |
| Huấn luyện YOLO26n trên Colab (CP6) | Sau khi chuẩn hóa vùng ROI và giữ đúng nhãn thực tế thị giác (`0:car, 1:truck, 2:bus, 3:van`), chạy lại pipeline huấn luyện trên Google Colab | Kiểm tra [`confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) và ảnh dự đoán [`drive_058.jpg`](assets/eval_predictions/visuals/drive_058.jpg) | `Precision` tăng từ `0.4105` lên **`0.5831`**, `mAP50` tăng từ `0.2723` lên **`0.3337`**, xe buýt vàng ở `drive_058.jpg` được dự đoán đúng là **`bus 0.51`** |

## 7. Tự đánh giá theo rubric

- [x] **Hoàn thành và định dạng YOLO (10/10):** Đủ 50 file `.txt` trong `dataset/labels/train/` xuất chuẩn định dạng `Ultralytics YOLO Detection 1.0` từ CVAT, chạy qua `tools/check_yolo_labels.py` đạt 50/50.
- [x] **Đầy đủ vật thể (20/20):** Đã bao phủ đầy đủ 653 phương tiện thuộc 4 lớp trong vùng ROI đánh giá, bao gồm các xe bị che khuất một phần ở các nút giao đông đúc.
- [x] **Độ bám sát và hình học box (20/20):** Box bám sát phần thân xe nhìn thấy được, không lấy bóng đổ hoặc gộp nhiều xe (`mean_iou_of_matches = 0.8510 >= 0.80`).
- [x] **Phân lớp nhất quán (15/15):** Nhất quán 100% đúng thực tế thị giác theo schema `0: car`, `1: truck`, `2: bus`, `3: van` trên toàn bộ 50 ảnh.
- [x] **Mini guideline và quyết định mơ hồ (10/10):** Đã hoàn thiện [`GUIDELINE_MINI.md`](../../GUIDELINE_MINI.md) với định nghĩa 4 lớp, quy tắc box và 4 ca mơ hồ cụ thể.
- [x] **Gold-set annotation QC (15/15):** Đã chạy `tools/evaluate_annotations.py` với `gold_train_labels`, ghi nhận đầy đủ kết quả Trước sửa (`F1 = 0.5663, mean_iou = 0.8384`) và Sau sửa (`F1 = 0.8162, mean_iou = 0.8510, passed = true`), giải thích rõ quyết định Sửa (vùng ROI / xe sót) và Giữ (nhãn thực tế `bus/truck/van` theo Guideline) và lưu [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json).
- [x] **Pipeline YOLO và phân tích (10/10):** Đã chạy hoàn tất trên Google Colab, trích xuất đủ 2 thư mục `assets/eval_predictions/` và `assets/yolo26_runs/` (không chứa file `.pt`), điền đủ metrics và phân tích sâu cả 4 câu hỏi.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ gán nhãn & huấn luyện:** CVAT Local (`v2.82.1` qua Docker Compose), Google Colab (GPU T4) chạy Ultralytics `yolo26n.pt`, và script kiểm tra `tools/check_yolo_labels.py`.
- **Sử dụng AI hỗ trợ:** Không dùng.
- **Tài liệu tham khảo:** Tài liệu hướng dẫn trong repo bài lab (`README.md`, `GUIDE.md`, `CHECKPOINTS.md`, `RUBRIC.md`, `SUBMISSION.md`) và tài liệu Ultralytics YOLO.
