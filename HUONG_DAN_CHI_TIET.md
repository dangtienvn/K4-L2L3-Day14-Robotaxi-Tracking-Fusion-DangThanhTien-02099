# Hướng Dẫn Chi Tiết Lab 14 — Robotaxi: Tracking 3D & Camera–LiDAR Fusion

Tài liệu này tổng hợp đầy đủ hướng dẫn thực hành, quy trình setup, gán nhãn track 3D trên CVAT local và các **LƯU Ý QUAN TRỌNG** bắt buộc tuân thủ.

---

## 🚨 LƯU Ý QUAN TRỌNG (ĐỌC KỸ TRƯỚC KHI THỰC HÀNH)

> [!CAUTION]
> ### 1. KHÔNG COMMIT / UPDATE DỮ LIỆU BẢO MẬT VÀO REPO
> - Dữ liệu VinFast Robotaxi (PCD point cloud, ảnh camera gốc, annotation thô, file calibration `private/calib-diagnostic.json` và thư mục `private/frame-maps/`) **tuyệt đối KHÔNG được commit, upload hoặc chia sẻ** lên GitHub, VLearn hay mạng xã hội.
> - Thư mục `private/` đã được cấu hình trong `.gitignore`. **Không tự ý sửa `.gitignore` hay dùng `git add -f`** để đưa dữ liệu bảo mật lên repo công khai này.
> - Không sửa các giá trị calibration trong `private/calib-diagnostic.json` với mục đích cố tình làm ảnh chiếu khớp hơn.

> [!IMPORTANT]
> ### 2. NGUYÊN TẮC GIẢI NÉN FILE DỮ LIỆU ("GIẢI NÉN SAU")
> - **File ZIP upload CVAT (`day14-vinfast-cvat-upload.zip`):** **KHÔNG GIẢI NÉN!** Để nguyên dạng file `.zip` này để kéo trực tiếp vào ô *Select files* khi tạo Task 3D trên CVAT local.
> - **File Data Pack (`day14-coach-data-pack.zip`):** Giải nén **ở thư mục gốc của repo** để thu được thư mục `private/` (chứa `calib-diagnostic.json` và `frame-maps/`) phục vụ cho plugin overlay:
>   - *Windows PowerShell:* `Expand-Archive $HOME\Downloads\day14-coach-data-pack.zip -DestinationPath .`
>   - *Linux/macOS:* `unzip ~/Downloads/day14-coach-data-pack.zip`

> [!WARNING]
> ### 3. SETUP TẠO TASK TRÊN CVAT LOCAL ĐỂ KHÔNG BỊ LỖI LABEL & OVERLAY
> 1. **Add Label đúng tên:** Khi tạo Task trên CVAT (`http://localhost:8080`), bắt buộc phải bấm **Add label** và đặt tên label chính xác là `vehicles` (kiểu **Cuboid** hoặc **Any**). Nếu thiếu label này, job sẽ không vẽ được cuboid xe.
> 2. **Cấu hình Frame Step:** Giữ nguyên **Frame Step = 1** (mặc định). **KHÔNG đổi sang Step = 2** hay giá trị khác. Plugin CVAT Overlay v2.74.1 yêu cầu `step=1`; nếu khác 1, ảnh camera sẽ bị lệch mốc thời gian và overlay sẽ từ chối hiển thị.
> 3. **Chọn Workspace:** Khi mở Job, chuyển workspace sang **Standard 3D**.

---

## 🛠️ QUY TRÌNH THỰC HÀNH CHI TIẾT TỪ ĐẦU ĐẾN CUỐI

### Bước 1: Khởi động Môi trường CVAT Local & Python
1. Mở Docker Desktop trên máy tính.
2. Khởi chạy nhóm container CVAT (đã cài ở Day 2, phiên bản v2.74.1):
   - Mở terminal/PowerShell tại thư mục CVAT compose: `docker compose start` (hoặc start trực tiếp nhóm container trong Docker Desktop).
   - Truy cập giao diện CVAT tại: `http://localhost:8080`.
3. Kiểm tra môi trường Python:
   ```powershell
   python --version
   python -m pip install -r requirements.txt
   ```

---

### Bước 2: Chuẩn bị Dữ liệu & Giải nén Data Pack
1. Nhận gói `day14-coach-data-pack.zip` từ Lab Coach.
2. Giải nén đúng vào **thư mục gốc repo**:
   ```powershell
   Expand-Archive $HOME\Downloads\day14-coach-data-pack.zip -DestinationPath .
   ```
3. Đảm bảo cấu trúc thư mục sau khi giải nén xuất hiện thư mục `private/` nằm cùng cấp với `README.md`:
   ```text
   K4-L2L3-Day14-Robotaxi-Tracking-Fusion-.../
   ├── README.md
   ├── HUONG_DAN_CHI_TIET.md
   ├── private/
   │   ├── calib-diagnostic.json
   │   └── frame-maps/
   ├── scripts/
   └── ...
   ```

---

### Bước 3: Tạo Task 3D trên CVAT Local Không Lỗi
1. Đăng nhập CVAT (`http://localhost:8080`) -> chọn tab **Tasks** -> bấm nút **+** -> **Create a new task**.
2. **Name:** Đặt tên dạng `Day14_<Tên_bạn>` (ví dụ: `Day14 DangThanhTien`).
3. **Labels:** Click **Add label** -> Nhập Name: `vehicles` -> Chọn type `Cuboid` (hoặc `Any`) -> Bấm **Continue**.
4. **Select files:** Chọn tab **My computer** -> Kéo thả trực tiếp file `day14-vinfast-cvat-upload.zip` (**file ZIP chưa giải nén**).
5. **Advanced configuration:** Đảm bảo **Frame step = 1**.
6. Bấm **Submit & Open** -> Chờ CVAT giải nén và tạo task (khoảng vài phút) -> Nhấn vào Job để mở.
7. Đảm bảo Workspace ở góc trên là **Standard 3D**. Panel camera hiển thị các ảnh `image_0` … `image_7`.

---

### Bước 4: Kích hoạt Plugin Overlay Camera
Overlay giúp chiếu trực tiếp 3D Cuboid từ point cloud lên ảnh camera trước (`image_1` / `CAM_P_F`) ngay trong CVAT local.

1. Đảm bảo container CVAT đang chạy (tên container mặc định `cvat_ui`). Kiểm tra bằng lệnh:
   ```powershell
   docker ps --format "{{.Names}}" | findstr cvat_ui
   ```
2. Bật overlay từ thư mục gốc repo:
   - *Windows (PowerShell / CMD):*
     ```powershell
     python scripts\cvat-overlay\overlay.py up
     ```
   - *Linux / macOS:*
     ```bash
     python3 scripts/cvat-overlay/overlay.py up
     ```
3. Tải lại trang CVAT bằng phím tắt **Ctrl + Shift + R** (macOS: **Cmd + Shift + R**).
4. Quan sát góc `image_1`: Nét chiếu cuboid màu xanh sẽ xuất hiện ôm lấy xe tương ứng khi bạn vẽ/chỉnh cuboid 3D.

---

### Bước 5: Quy trình Gán Nhãn Track 3D Dài (J01 - 66 Frames)

1. **Xác định Object Mục tiêu:** Đọc chỉ định xe mục tiêu từ Lab Coach.
2. **Xác lập Kích thước Tham chiếu (L/W/H):**
   - Bấm `F` để duyệt tới frame xe có **nhiều điểm LiDAR nhất** (frame đủ evidence).
   - Fit cuboid tỉ mỉ trên 3 view phụ (**Top**, **Side**, **Front**):
     - **Top:** Kéo thân hộp trùng tâm cụm điểm, kéo điểm đỏ ôm sát ranh giới, kéo chấm xanh lá xoay song song thân xe.
     - **Side & Front:** Kéo điểm đỏ dưới chạm mặt đường, điểm đỏ trên chạm nóc xe.
   - Mở phần **DETAILS** trên thẻ track, ghi lại 3 chỉ số **Length (L)**, **Width (W)**, **Height (H)** vào file `docs/personal-notes.txt`.
3. **Tạo Track từ Frame Đầu Tiên (Sử dụng Track, KHÔNG dùng Shape):**
   - Lùi về frame đầu tiên xe xuất hiện (`D`).
   - Chọn công cụ Cuboid -> Label `vehicles` -> Chọn kiểu **Track** -> Click chọn cụm điểm của xe.
   - Nhập chính xác bộ 3 số L/W/H tham chiếu vào DETAILS của keyframe đầu tiên.
4. **Duyệt qua Toàn bộ Sequence & Đặt Keyframe:**
   - Bấm `F` duyệt qua từng frame (0 đến 65).
   - **NGUYÊN TẮC VẬT RẮN:** Giữ nguyên kích thước L/W/H. Ở các frame nội suy, **chỉ dịch chuyển thân hộp và xoay heading (chấm xanh lá)** khi hộp bị lệch cụm điểm.
   - Đặt **Keyframe** (sao đặc) tại: frame đầu, frame cuối, và các điểm xe đổi hướng/tốc độ.
   - **Thuộc tính quan trọng:**
     - Bật `occluded` (`Q`): Khi xe bị vật khác che khuất một phần hoặc toàn bộ.
     - Bật `outside` (`O`): Khi xe đi ra khỏi vùng quan sát / kết thúc track.

---

### Bước 6: Self-QC Kiểm Tra 5 Nhóm Lỗi Temporal
Trong quá trình gán nhãn, cần đối chiếu tự kiểm tra 5 nhóm lỗi sau:
1. **ID Switch:** Đảm bảo 1 xe giữ nguyên 1 Track ID duy nhất từ đầu đến cuối sequence, không bị nhảy ID sang xe bên cạnh.
2. **Fragmentation:** Không cắt rời track thành nhiều track ngắn khi xe tạm thời bị che khuất ngắn.
3. **Dimension Drift:** Không để kích thước box co giãn bất thường theo khoảng cách xa/gần (tuân thủ kích thước vật rắn).
4. **Orientation Flip:** Kiểm tra hướng đầu xe (chấm xanh) không bị lộn ngược 180° so với hướng di chuyển thật.
5. **Box Shrinkage/Jump do Sparsity:** Ở dải xa ít điểm point cloud, duy trì box theo kích thước tham chiếu và quỹ đạo nội suy, không thu nhỏ box vừa khít với vài điểm còn lại.

---

### Bước 7: Đối chiếu Camera–LiDAR (Fusion & Discrepancy)
1. Quan sát hình chiếu cuboid màu xanh trên ô camera `image_1`.
2. Đối chiếu giữa 3D Point Cloud và ảnh 2D:
   - Xác nhận đúng class (`vehicles`), loại xe và hướng xe.
   - Kiểm tra các trường hợp không khớp (discrepancy): Phân biệt do lỗi annotation, xe ngoài FOV, bị occlusion che lấp, hay do điểm LiDAR quá thưa.
3. Ghi lại các ca bình thường và ca khó vào phiếu cá nhân `docs/personal-notes.txt`.

---

### Bước 8: Save & Nộp Bài (Completed)
1. Sau khi chỉnh sửa xong, bấm **Save** (`Ctrl + S`).
2. Tải lại trang (F5) để kiểm tra đảm bảo toàn bộ annotation và keyframe đã được lưu thành công trên CVAT.
3. Nhấp vào Menu góc trái CVAT -> Chọn **Change job state** -> Chọn **completed**.
4. Báo với Lab Coach về danh sách job đã hoàn thành. *(Không export ZIP để nộp lên VLearn)*.

---

## 📌 BẢNG TRA CỨU PHÍM TẮT & THUỘC TÍNH CVAT 3D

| Phím tắt | Thao tác / Thuộc tính | Ý nghĩa / Khi nào dùng |
| :---: | :--- | :--- |
| `F` | Next Frame | Chuyển sang frame kế tiếp |
| `D` | Previous Frame | Quay lại frame phía trước |
| `K` | Toggle Keyframe | Bật/tắt mốc Keyframe (sao đặc) |
| `Q` | Occluded | Bật/tắt trạng thái Xe bị che khuất |
| `O` | Outside | Bật/tắt trạng thái Xe ra khỏi góc nhìn / kết thúc track |
| `L` | Lock | Khóa box để tránh lỡ tay kéo lệch |
| `H` | Hide | Ẩn/hiện box trên màn hình |
| `N` | Re-draw | Vẽ lại cuboid mới với cùng label/thiết lập |
| `Ctrl + S` | Save | Lưu annotation lên máy chủ CVAT local |
