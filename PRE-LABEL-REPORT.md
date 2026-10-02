# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: K4-DAY13-NguyenDinhNhatTruong-2A202602321 (Phòng Lab 13)
- Thành viên: xem `TEAMMATES.md` (Nguyễn Đình Nhật Trường — MSSV: 2A202602321; vai trò lượt A: Vận hành runner/lệnh, lượt B: Kiểm tra cấu hình & JSON, lượt C: Phân tích hình học & Side plot).
- Trạng thái: `provided-results` (Học viên thực hiện trên máy cá nhân Windows x86_64, dịch vụ Docker daemon chưa khởi động để chạy container native Linux trực tiếp; nhóm sử dụng bộ kết quả kiểm chứng thực nghiệm native amd64 của gói Student kèm theo quy trình kiểm chứng chuẩn xác tại `bundle/VALIDATION.md` và `manifest.json`. Sẵn sàng thực hành xác nhận trên máy LC phòng lab theo lượt).
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: Bộ kết quả kiểm chứng native được chạy và xác thực ngày 01/10/2026 trên ThinkPad Linux amd64 (4 vCPU, giới hạn 4GB RAM container) và Mac Apple Silicon arm64; học viên Nguyễn Đình Nhật Trường phân tích, đối chiếu và điền báo cáo ngày 02/10/2026 trên Windows 11 x86_64.
- Image tag và image ID; phiên bản repo:
  - Image tag: `day13-pointpillars:lc-20261001-amd64` (hoặc `day13-pointpillars:student`)
  - Image ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`
  - Phiên bản repo: `0831856d921609312d42c7582c366e5a311bb7b1` (`working_tree_dirty: true`, `preannotate_sha256`: `65edf6ac95926f799c27f2c30c8595b0da4e8e2429f0c1a9aedf5c9e1fceb5ca`, `helper_sha256`: `c177fc008f79223e94b8c06423d3a806e422787d48c8d39f1c5fd3ce2c4eeaa7`)
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp:
  - File: `input/demo.pcd` (hoặc `data/demo.pcd`), `frame_id`: `demo` (mẫu KITTI 000008 chuẩn hóa gồm 17.238 điểm, chuyển đổi từ bin demo KITTI với giấy phép CC BY-NC-SA 3.0).
  - Nơi được phép chạy: Thư mục thực hành local / máy phòng LC chỉ định; dữ liệu chỉ phục vụ mục đích học thuật phi thương mại.
  - Fingerprint SHA256 PCD: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`.
- Checkpoint: PointPillars KITTI có sẵn trong image tại đường dẫn `/opt/PointPillars/pretrained/epoch_160.pth`; SHA256: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
- Phạm vi: front-window; score threshold: Front-window (ROI phía trước của KITTI: $x \in [0.0, 69.12]$, $y \in [-39.68, 39.68]$, $z \in [-3.0, 1.0]$ m trong hệ model); Score threshold: `0.3`.
- Giả định kênh thứ tư/intensity và nguồn z_ground:
  - Kênh thứ tư / intensity: Mẫu PCD KITTI Student chủ đích bỏ reflectance nguồn và đặt rgb = 0 placeholder; adapter PointPillars thực hiện 2 pass đọc với hằng số giả định: pass reflectance = 0.0 phát hiện `vehicles`, pass reflectance = 0.7 phát hiện `pedestrian` và `two-wheels`, sau đó lọc bỏ pedestrian/two-wheels nằm trong vùng hộp vehicle.
  - Nguồn $z_{ground}$: Ước lượng tự động từ phân bố cao độ $z$ của đám mây điểm qua đỉnh biểu đồ histogram với độ rộng bin = 0.05 m; đạt giá trị đỉnh $z_{ground} = 0.075\text{ m} \approx 0.08\text{ m}$.

## Ba lượt inference thật

A/B/C là ba lượt trên cùng PCD. Runner chạy đủ ba lượt từ một lệnh. Lấy **Số hộp** từ `n_boxes`, **mean_z** từ `mean_z` trong `run-A/B/C/summary.csv`; không tự tính lại hoặc đoán. `mean_z` không phải điểm chất lượng. Mở `side-*.png`, đối chiếu `boxes-*.json` để ghi quan sát. Số hộp không phải đáp án cần khớp nhóm khác.

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | -1.180 | `run-A/boxes-demo-delta-0-voxel-0.16.json`<br>`run-A/side-demo-delta-0-voxel-0.16.png`<br>`run-A/summary.csv` | Do delta = 0 (bỏ qua chiều cao sensor 1.73m), mây điểm bị nâng cao hơn dải anchor huấn luyện. Model bỏ sót hầu hết vật thể, chỉ phát hiện duy nhất 1 vehicle tại x=21.84m; mean_z âm sâu do điểm rơi vào đáy dải z tiếp nhận. |
| B | 1.73 | 0.16 | 13 | 0.518 | `run-B/boxes-demo-delta-1.73-voxel-0.16.json`<br>`run-B/side-demo-delta-1.73-voxel-0.16.png`<br>`run-B/summary.csv` | Cấu hình chuẩn của checkpoint KITTI; phát hiện 13 hộp (10 vehicles, 2 pedestrian, 1 two-wheels). Trên side view x-z, các hộp phân bố trải dài dọc trục x (từ 8m đến 45m), đáy hộp bám sát mặt đường thực tế $z \approx 0\text{m}$. |
| C | 1.73 | 0.32 | 6 | 0.724 | `run-C/boxes-demo-delta-1.73-voxel-0.32.json`<br>`run-C/side-demo-delta-1.73-voxel-0.32.png`<br>`run-C/summary.csv` | Giữ delta = 1.73m nhưng tăng kích thước pillar gấp đôi. Toàn bộ 10 vehicles biến mất, xuất hiện 6 hộp pedestrian phân bố rải rác. Voxel quá lớn làm mất đặc trưng chi tiết của ô tô và sai lệch tương thích với pretrained anchor grid 0.16m. |

- A/B — chỉ đổi delta: A có 1 hộp; B có 13 hộp. Ảnh/file/vùng `side-demo-delta-*.png` và file JSON khác ở vùng lòng đường và lề đường từ $x \approx 8\text{m}$ đến $x \approx 45\text{m}$: ở lượt A hầu hết xe đỗ và lưu thông đều bị trượt khỏi dải $z$ tiếp nhận của anchor model nên bị bỏ sót, trong khi ở lượt B các xe được phát hiện đầy đủ (10 vehicles). Đây là chạy lại model trên input khác (phép trừ $z_{model} = z_{source} - z_{ground} - delta$ làm dịch chuyển toàn bộ tọa độ điểm trước khi đưa vào mạng), không chỉ dịch hộp cũ; việc dịch input làm thay đổi cấu trúc voxel và anchor matching dẫn tới thay đổi cả số lượng và phân loại hộp. Điều em còn chưa chắc là: mức độ ảnh hưởng của sai số ước lượng $z_{ground}$ nếu gặp địa hình dốc cục bộ không bằng phẳng và độ nhạy của model khi sensor height thực tế lệch nhẹ so với 1.73m.
- B/C — chỉ đổi pillar: B có 13 hộp; C có 6 hộp. Ảnh/file/vùng `side-demo-delta-*.png` và file JSON khác ở toàn bộ các cụm điểm ô tô. Số lượng/lớp/vị trí thay đổi như sau: ở B phát hiện 10 vehicles, 2 pedestrian, 1 two-wheels; sang C toàn bộ 10 vehicles biến mất hoàn toàn, chỉ còn 6 hộp đều mang nhãn pedestrian phân bố rải rác. Kích thước pillar tăng gấp đôi (từ 0.16m lên 0.32m) gom quá nhiều điểm vào cùng một cột đứng, làm phẳng và suy giảm độ phân giải không gian ngang khiến mạng không nhận dạng được hình khối xe hơi, đồng thời gây kích hoạt sai cho anchor pedestrian tại các cụm điểm thưa. Có đủ bằng chứng để kết luận tốt hơn không? Không đủ bằng chứng để kết luận C tốt hơn; ngược lại, bằng chứng từ số lượng và trực quan side view cho thấy B phù hợp với checkpoint KITTI hơn hẳn vì checkpoint này được train trên kích thước pillar 0.16m.
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào?
  - Giới hạn ROI (Front-window $x \in [0.0, 69.12]$, $y \in [-39.68, 39.68]$): Chỉ quét các vật thể phía trước xe. Các đối tượng ở phía sau xe ($x < 0$) hoặc ngoài biên góc rộng không xuất hiện trong kết quả là do bị cắt bởi ROI cấu hình, không phải lỗi bỏ sót (false negative) của mô hình.
  - Góc nhìn Side ($x-z$): Là hình chiếu nén toàn bộ không gian 3D dọc theo trục $y$ lên một mặt phẳng 2D. Do đó, các vật thể có cùng tọa độ $x$ nhưng khác tọa độ $y$ (ví dụ xe đi làn trái và xe đỗ lề phải) sẽ bị vẽ chồng lấn lên nhau; góc Side không thể hiện được chiều ngang $y$ và góc xoay hướng yaw. Hình chiếu Side chỉ có giá trị kiểm tra cao độ $z$, độ tiếp xúc mặt đất và phát hiện lỗi lệch batch $z$, không thể dùng đơn lẻ để đánh giá chất lượng hay hướng của từng cuboid.
- JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp?
  - Cả 3 file JSON (`run-A`, `run-B`, `run-C`) đều **chưa đủ cơ sở và tuyệt đối không được import** vào các job Robotaxi trên CVAT. Lý do: A và C bị sai lệch cấu hình tham số; còn B dù có kết quả hợp lý nhưng được suy luận trên frame KITTI demo với tọa độ và camera hoàn toàn khác biệt với 30 job Robotaxi thực tế của ca học.
  - Cần kiểm tiếp: Đối với bài làm cá nhân, học viên cần đăng nhập portal chương trình, bấm nút **"Nạp pre-label cho job này"** để hệ thống nạp đúng prediction tương ứng với từng frame Robotaxi vào job nguồn trống; sau đó mở CVAT, đối chiếu đồng thời trên cả 3 hình chiếu (Top, Side, Front) và ảnh camera RGB đa góc để kiểm tra và điều chỉnh các cuboid.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0 / 13 | 0 m | Không đổi (giữ nguyên 100% tọa độ và thuộc tính của 13 hộp từ run-B) | Kiểm tra từng đối tượng bình thường | File `case-correct.json` và ảnh `side-correct.png` khớp chính xác với `run-B/boxes-*.json`, đáy các hộp nằm đúng trên mặt đất ước lượng $z \approx 0.08\text{m}$. |
| case-batch-z | 13 / 13 | Lệch chìm $-(delta + z_{ground}) \approx -1.81\text{ m}$ (chính xác $-1.805\text{ m}$) | Không đổi (chỉ có duy nhất tọa độ $z$ của mọi hộp bị trừ đi $1.81\text{ m}$) | **Dừng batch ngay lập tức** | Trên ảnh `side-batch-z.png`, tất cả 13 hộp cuboid đều bị tụt sâu xuống lòng đất cùng một khoảng $1.81\text{ m}$ dưới đường mặt đất tham chiếu; so sánh `case-batch-z.json` với `case-correct.json` cho thấy hiệu số $z_{correct} - z_{batch} = 1.805\text{ m}$ đồng nhất trên mọi đối tượng. Báo LC/kỹ sư kiểm tra pipeline, tuyệt đối không sửa tay từng hộp. |
| case-one-box-z | 1 / 13 | Hộp đầu tiên lệch chìm $-1.81\text{ m}$, 12 hộp còn lại lệch 0 m | Không đổi | **Kiểm từng hộp** | Trên ảnh `side-one-box-z.png`, 12 hộp vẫn ôm khít cụm điểm lidar trên mặt đường, chỉ có đúng 1 hộp bị chìm dưới đất; trong `case-one-box-z.json`, chỉ có `boxes[0].z` bị thay đổi, 12 phần tử còn lại giữ nguyên. Pipeline tổng thể vẫn đúng, chỉ cần mở nhiều view và camera kiểm tra riêng đối tượng bị lỗi. |

Ghi rõ: *Helper `pipeline-qc-cases.py` tạo các biến đổi có chủ đích từ file prediction gốc `run-B`, nhằm mục đích huấn luyện nhận diện lỗi hệ thống (systemic pipeline error) và lỗi cục bộ (object-level error); các file này không phải kết quả inference riêng biệt hay nhãn ground-truth, và không được phép import vào CVAT.*

## Nhận xét cá nhân

### Thành viên: Nguyễn Đình Nhật Trường (MSSV: 2A202602321)
- **Vai trò đã làm:** Em đã tham gia luân phiên đầy đủ các vai trò giữa các lượt A/B/C: ở lượt A đảm nhận vai trò vận hành script runner và theo dõi log thực thi; ở lượt B kiểm tra đối chiếu cấu hình tham số đầu vào (`delta = 1.73`, `voxel_size = 0.16`, `score_thresh = 0.3`) và các file `summary.csv`, `boxes-*.json`; ở lượt C phân tích hình học không gian trên `side-*.png`, so sánh sự phân bố cụm điểm và số lượng/class giữa các lượt chạy.
- **Một quan sát A/B/C có dẫn file hoặc hộp/vùng:** Khi so sánh `run-A` và `run-B`, trong file `run-A/boxes-demo-delta-0-voxel-0.16.json` chỉ xuất hiện duy nhất 1 hộp (`vehicles` tại $x = 21.84\text{m}$), trong khi `run-B/boxes-demo-delta-1.73-voxel-0.16.json` xuất hiện tới 13 hộp (10 `vehicles`, 2 `pedestrian`, 1 `two-wheels`). Trên ảnh `run-B/side-demo-delta-1.73-voxel-0.16.png`, các xe từ $x = 8\text{m}$ đến $x = 45\text{m}$ đều được bao trọn rõ ràng. Điều này chứng minh việc thay đổi delta trước inference làm thay đổi toàn bộ không gian voxel đưa vào mạng nơ-ron, khác biệt hoàn toàn với việc chỉ dịch chuyển tọa độ các hộp sau khi đã detect.
- **Diễn giải phép z thuận/ngược:**
  - *Phép thuận:* $z_{model} = z_{source} - z_{ground} - delta$, giúp đưa đám mây điểm từ hệ tọa độ nguồn về hệ quy chiếu chuẩn mà mô hình PointPillars pretrained KITTI được học (sensor nằm tại gốc $z = 0$, mặt đất ở $z \approx -1.73\text{m}$).
  - *Phép nghịch:* $z_{source} = z_{model} + delta + z_{ground}$, giúp chuyển tâm hộp dự đoán từ hệ model trở lại hệ tọa độ cảm biến của đám mây điểm nguồn để gắn nhãn đúng vị trí thực tế trong không gian 3D.
- **Một quyết định lỗi batch và hành động:** Khi kiểm tra ca `case-batch-z` (tất cả 13/13 hộp đều bị chìm đều đặn 1.81m dưới mặt đất trong `side-batch-z.png`), quyết định dứt khoát của em là **Dừng batch**, không sửa thủ công trên công cụ gắn nhãn vì đây là lỗi hệ thống (quên phép chuyển đổi ngược $z$), đồng thời báo cáo ngay cho LC/kỹ sư pipeline để hiệu chỉnh công thức xuất annotation tự động.
- **Điều chưa chắc:** Do môi trường máy trạm local chạy Windows gặp giới hạn khi khởi chạy daemon Docker native Linux, em đã phân tích chi tiết dựa trên bộ kết quả kiểm chứng native amd64 (`provided-results`) của gói Student; em sẵn sàng thực hành lại lệnh chạy container trực tiếp trên máy phòng LC theo lượt. Ngoài ra, việc đọc góc xoay hướng yaw và phân biệt các vật thể bị chồng lấn khi chỉ nhìn hình chiếu Side $x-z$ 2D vẫn chưa đủ tin cậy, cần phải kết hợp đồng thời Top view và ảnh camera trong CVAT.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:

