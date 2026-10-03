---
day: "D14"
title: "Lab 14 — Robotaxi B: giữ track 3D qua thời gian và đối chiếu Camera–LiDAR"
description: "Khởi động bằng một job ngắn, hoàn thiện track lõi 66 frame, làm thêm job ngắn theo tốc độ, kiểm nội suy và giải thích discrepancy."
outcomes:
  - "Giữ identity, class và kích thước của một vật rắn qua sequence; đặt thêm keyframe khi cần."
  - "Kiểm tra năm nhóm lỗi temporal bằng sequence và bằng chứng trước/sau."
  - "Phân biệt bất nhất do annotation, FOV, occlusion, sparse points và nghi lỗi calibration; ghi hành động có căn cứ."
prerequisites:
  - "Đã fit cuboid bằng Top → Side → Front trong Lab 13."
  - "Có tài khoản CVAT và job Robotaxi cùng guideline do Lab Coach giao."
  - "Có quyền đọc repo calibration của lớp; Coach chuẩn bị môi trường phép chiếu trước buổi học."
requiredTools:
  - "CVAT của lớp và trình xem ảnh; Python dùng cho phần baseline hoặc demo của Coach."
  - "Repo K4-L23-Day14-Calibration; báo cáo QC và overlay do Coach cung cấp từ bản đã Save."
commonErrors:
  - "Box co theo frame thưa → quay lại kích thước đã xác lập ở frame có đủ evidence."
  - "Chỉ xem keyframe → kiểm thêm toàn bộ frame nội suy của track được giao."
  - "Overlay lệch nhiều object → kiểm cặp dữ liệu và phép chiếu trước khi sửa cuboid."
requiresSubmission: false
workMode: "individual"
---

# Lab 14 — Robotaxi B: giữ track 3D qua thời gian và đối chiếu Camera–LiDAR

**Thời lượng:** 4 giờ thực hành. **Hình thức:** cá nhân. **Nộp bài:** Save rồi chuyển job sang `completed` trên CVAT như Day 12. Coach thu annotation và review riêng. Với 3D, Coach giữ native annotation JSON từ API và bản `Datumaro 3D 1.0` đã kiểm mapping cho QC/overlay. VLearn dùng để đọc hướng dẫn; không nộp ZIP tại đây.

Bạn tiếp tục từ cuboid của Lab 13 đến một object đi qua nhiều frame. Bài hôm nay có một nhiệm vụ lõi: **hoàn thiện một track 3D dài, rồi chứng minh quyết định giữ, sửa hoặc escalate bằng QC và hình ảnh**. Bạn nhận một ngân hàng **20 job**, có object lõi và thứ tự case được giao ngẫu nhiên trong từng mức. Phần bắt buộc là job khởi động B01, track lõi J01 và bằng chứng của nó; sau đó phần lớn người làm thêm được 4–6 job ngắn, người nhanh làm tiếp. Không ai phải làm hết 20 job.

Mở [repo calibration Day 14](https://github.com/VinUni-AI20k/K4-L23-Day14-Calibration/tree/main) trước khi tải hoặc chạy mã. Repo cung cấp ví dụ một point cloud và một ảnh camera trước. Sequence thực hành dùng mẫu VinFast `3D_Lidar_sample_data`: **66 frame liên tiếp, mỗi frame có 8 ảnh ngữ cảnh**. Coach đã đối chiếu byte: PCD và ảnh trước của repo chính là frame 0 và `image_1.jpg` trong mẫu này. Quan hệ ấy nối ví dụ phép chiếu với dữ liệu lớp; nó chưa chứng minh calibration đúng trên mọi frame.

| Phút | Việc làm | Xong khi |
|---|---|---|
| 0–15 | Đăng nhập CVAT, mở danh sách job. Coach demo 10 phút trên màn chiếu theo [hướng dẫn bằng hình](huong-dan-hinh.md). | Mở được workspace 3D của B01 và J01 |
| 15–35 | **Khởi động B01** (6 frame) theo mục "Làm một track từ đầu đến cuối" của hướng dẫn bằng hình. | B01 đã Save và completed |
| 35–110 | **Track lõi J01** (66 frame). Mốc con: phút 55 có L/W/H tham chiếu; phút 85 có keyframe đầu, cuối và chỗ đổi hướng; phút 110 đã bấm `F` qua đủ 66 frame. | J01 đã Save; Coach thu bản độc lập |
| 110–160 | **Job ngắn** theo slot, bỏ qua B01. Khoảng 8–10 phút/job; nhờ bạn cùng cặp xem một job. | Mỗi job Save và completed; thường được 4–6 job |
| 160–185 | **Dừng mở job mới.** Coach chiếu baseline phép chiếu; bạn đọc QC và overlay của J01, ghi một ca bình thường và một ca khó. | Hai dòng discrepancy trong phiếu |
| 185–210 | Review với bạn cùng cặp và Coach; quyết định sửa, giữ hay escalate. | Ít nhất một quyết định có evidence |
| 210–230 | Rework J01 và job bị góp ý; Save, tải lại trang để kiểm. | Bản cuối đã Save |
| 230–240 | Chuyển các job đã xong sang completed, báo Coach phần đã làm và ca còn mở. | Danh sách Jobs hiện completed |

Khi trễ mốc:

- **J01 chưa xong ở phút 110:** vẫn Save ngay để Coach có bản độc lập, rồi làm tiếp J01 tối đa tới phút 130 và bỏ phần job ngắn. Phút 130 vẫn chưa xong thì Save, ghi frame cuối đã kiểm và gọi Coach.
- **Một job ngắn quá 15 phút:** Save, ghi chỗ kẹt vào phiếu, sang job kế tiếp.
- **Kẹt một thao tác quá 5 phút:** gọi Coach, đừng ngồi đoán.

Tự chạy repo calibration (mục "Chạy phép chiếu mẫu") là phần làm thêm cho người đã xong sớm; mọi người vẫn xem baseline qua phần Coach chiếu ở phút 160.

## Chuẩn bị đúng job và ghi nguồn bằng chứng

**Đầu ra:** bạn biết object nào mình chịu trách nhiệm, sequence nào đang dùng và đâu là dữ liệu demo.

### Tôi sẽ làm trên dữ liệu nào?

Lab Coach giao một job Robotaxi và một object mục tiêu có đoạn đủ điểm để xác lập kích thước, cùng đoạn khó hơn để kiểm tính ổn định. Bạn chịu trách nhiệm toàn đoạn của object đó. Xem object lân cận để kiểm identity hoặc lệch cả cảnh.

Bạn dùng job cá nhân và tài khoản từ Day 12–13. Coach chỉ định object bằng ảnh/vị trí trong cảnh; annotation nguồn được giữ riêng. Mỗi job chứa sequence liên tục để bạn xem keyframe và frame nội suy; bộ của bạn có một job dài và các job ngắn.

Dữ liệu giữ dải xa để luyện quyết định khi điểm thưa. Demo trong `artifacts.zip` giúp đọc cảnh báo; đối chiếu cảm biến dùng ảnh thật VinFast.

### Tôi bắt đầu trong CVAT thế nào?

1. Đăng nhập CVAT của lớp, mở job Day 14 được giao. Ghi task/job và phạm vi vào phiếu cá nhân.
2. Tìm object mục tiêu theo ảnh/vị trí. Xem một frame đủ điểm và một đoạn khó. Bài yêu cầu một track, không gán nhãn toàn cảnh.
3. Đọc mapping Coach cấp: job dài có index 0–65; job ngắn bắt đầu lại từ 0. Filename giữ timestamp gốc, nên index cục bộ khác frame nguồn. Ảnh `image_1` dùng cho camera trước theo correspondence ở frame nguồn 0.
4. Mở phiếu ghi chú cho frame fit, L/W/H, keyframe, phát hiện temporal và discrepancy. Coach thu/review riêng; bạn không phải export hoặc nộp CSV lên VLearn.

Chưa quen workspace 3D thì mở [hướng dẫn bằng hình](huong-dan-hinh.md): chỗ tạo Track, fit trên Top/Side/Front, đọc keyframe/outside/occluded và Save → completed.

**Checkpoint:** tìm được object và frame fit, đọc được mapping PCD/ảnh/calibration. Nếu lỗi truy cập hoặc thiếu dữ liệu, báo Coach; không dùng demo thay evidence thật. Khi đầu vào đủ, chuyển sang đọc baseline phép chiếu.

## Làm theo bộ job ngẫu nhiên và chọn mức tiếp theo

**Đầu ra:** hoàn thành track lõi với evidence chung, rồi làm các lượt mở rộng phù hợp tốc độ và giữ chất lượng từng job.

### Tôi nhận 20 job thì phải làm đủ 20 mới xong bài không?

Bộ 20 job là ngân hàng thực hành đã được sắp theo mức. **Chuẩn chung bắt buộc là job track dài J01 và evidence của nó**: frame fit đa view, L/W/H có lý do, keyframe, quét nội suy, self-QC, đối chiếu một ca bình thường và một ca khó, cùng quyết định sau review. Học viên cần hỗ trợ vẫn làm đúng chuẩn ấy với checkpoint và demo thêm. Số job mở rộng phản ánh lượng thực hành bạn đã làm; nó không tự thay thế bằng chứng về track lõi hoặc biến thành thang điểm riêng.

Một job ngắn là một đoạn **6, 8 hoặc 10 frame liên tiếp**, với một object mục tiêu. Job dài dùng 66 frame dữ liệu. Vì vậy “20 job” trong bài này không phải 20 sequence dài giống nhau. Bạn tập thêm thao tác và quyết định trên các object/đoạn khác, giữ thời gian cuối buổi cho fusion và rework. Khi đến mốc dừng mở job mới, hãy xử lý bài đã bắt đầu thay vì chạy tiếp để tăng số completed.

### Vì sao giao ngẫu nhiên nhưng vẫn chia mức?

Coach ngẫu nhiên object của job lõi và thứ tự các case trong mỗi mức, giữ cùng cơ cấu độ khó giữa các bộ. Các bạn có thể nhận thứ tự khác nhau, nên phải đọc đúng object và mapping của mình. Ngẫu nhiên không có nghĩa lấy bất kỳ job nào trong danh sách hoặc đổi guideline theo bộ. Một lần kết thúc job cũng không kết thúc identity thật của object ở dataset: ranh giới job ngắn chỉ là phạm vi thực hành đã giao.

| Lượt | Thành phần bổ sung | Tổng job | Trọng tâm |
|---|---|---|---|
| 1 | J01 dài + 4 job cơ bản, 6 frame/job | 5 | Fit, track, kích thước và nội suy |
| 2 | 2 job cơ bản + 3 job trung gian, 8 frame/job trung gian | 10 | Frame thưa/xa, heading, FOV và ảnh ngữ cảnh |
| 3 | 3 trung gian + 2 chẩn đoán, 10 frame/job chẩn đoán | 15 | Identity, gap và quyết định giữ/sửa |
| 4 | 1 trung gian + 4 chẩn đoán | 20 | So giả thuyết, giải thích ambiguity/escalation |

### Tôi chuyển lượt bằng tín hiệu nào?

1. **Làm B01 rồi J01.** B01 là job khởi động 6 frame để quen thao tác; hướng dẫn bằng hình chụp đúng job này nên nó không tính là bằng chứng fit độc lập. Sau đó làm J01. Đọc object được giao, chọn frame fit và hoàn thiện track trên toàn đoạn. Save bản độc lập trước phản hồi. Khi cần hỗ trợ, nhờ kiểm frame fit; đừng bỏ job lõi để lấy nhiều completed ngắn.
2. **Làm các mini-job theo thứ tự bộ của mình.** Mỗi job vẫn tạo Track, giữ kích thước có evidence, dịch/xoay các mốc và xem toàn bộ frame ở giữa. Đọc local frame và source frame đúng mapping; không nối số ID qua hai job chỉ vì cùng filename nguồn.
3. **Self-check trước khi completed.** Kiểm đúng object/class, geometry, ID/keyframe/nội suy và ca khó. Save, mở lại kiểm, ghi ca sửa/giữ/escalate rồi completed. Nhờ bạn cùng cặp xem một mini-job mỗi lượt; giữ quyết định của mình trước trao đổi.
4. **Đi lượt kế khi bài hiện tại đã được xử lý.** Có lỗi xác minh thì rework; thiếu evidence thì ghi blocker/người nhận. Bạn không cần chờ Coach duyệt từng mini-job; Coach spot-check và review riêng. Nếu lỗi lặp lại, quay về checkpoint thay vì mở tiếp job khó.
5. **Dừng mở job mới ở phút 160.** Ghi job đã bắt đầu, job đã completed và phần còn mở. Dành 80 phút cuối cho fusion, review, rework và Save/completed theo bảng giờ ở đầu bài. Job chưa bắt đầu trong ngân hàng không được ghi là đã làm hoặc tự coi là lỗi annotation.

**Checkpoint:** bạn chỉ ra được J01, lượt đang làm, job đã lưu và một quyết định có evidence. Phần lớn người làm thêm được 4–6 job ngắn, người nhanh làm tiếp; người cần hỗ trợ có đường kiểm từng mốc; mọi người đều giữ chuẩn lõi. Coach ghi riêng khối lượng và chất lượng, không dùng số completed để suy ra annotation đúng. Báo Coach cả thời gian và blocker thực tế.

## Chạy phép chiếu mẫu và đọc giới hạn của calibration

**Đầu ra:** một ảnh LiDAR chiếu lên camera từ dữ liệu mẫu của repo và ghi chú về phạm vi mà ảnh này kiểm chứng.

### Tại sao chạy ví dụ trước khi sửa track?

Khi overlay không khớp, có thể cuboid đặt sai, nhưng cũng có thể bạn đang đọc sai phép biến đổi hoặc ghép nhầm file. Chạy ví dụ có sẵn giúp kiểm môi trường và hiểu hình chiếu trông như thế nào trước khi dùng ảnh để nhận xét annotation. Ví dụ này chỉ có point cloud và ảnh camera. Nó chưa chứng minh track của bạn đúng, vì không có annotation 3D hay box YOLO kèm trong `data/sample/`.

Một điểm trong LiDAR được lưu bằng mét. Điểm tương ứng trên ảnh được lưu bằng pixel. Muốn đặt chúng vào cùng một phép đối chiếu, script phải đưa điểm từ LiDAR sang ego, rồi từ ego sang camera. Khi điểm đã ở hệ camera, phép chiếu dùng chiều sâu Z và đặc tính ống kính để tính vị trí trên ảnh. Vì phép chiếu chia cho Z, vật ở xa chiếm ít pixel hơn dù kích thước thật của vật không thay đổi.

### Intrinsic và extrinsic quyết định điều gì?

Trong repo này, **intrinsic** nằm ở `data/calib/Intrinsics.txt`: kích thước ảnh, ma trận camera K và hệ số méo. **Extrinsic** nằm ở `data/calib/Extrinsics.txt`: các ma trận 4×4 đưa tọa độ giữa các hệ. Hai file có đuôi `.txt` nhưng nội dung là JSON. Bạn đọc các thông số đã được cấp; bài không yêu cầu tự đo calibration.

Điểm dễ nhầm là các ma trận trong cùng file extrinsic không dùng chung một chiều. README quy định `LIDAR_*` là sensor → ego, còn `CAM_*` là ego → camera. Chuỗi của script là `T_ego2cam × T_lidar2ego`. Không nghịch đảo thêm ma trận camera chỉ vì thấy công thức khác trên mạng. Với dữ liệu khác, phải đọc lại manifest thay vì mang quy ước này sang mặc định.

### Tôi cần quan sát gì trên ảnh baseline?

1. Theo demo của Coach, hoặc từ terminal trong repo calibration đã tải qua link đầu bài, cài đúng dependency rồi chạy test của repo. Test kiểm các hàm trên dữ liệu nhỏ, giúp phát hiện lỗi môi trường hoặc logic đã được test; nó không xác nhận calibration của toàn bộ sequence thật.

   ```bash
   python -m pip install -r requirements.txt
   python -m pytest -q
   ```

2. Chạy ví dụ gốc với camera `CAM_P_F` và LiDAR `LIDAR_TOP`:

   ```bash
   python src/lidar_to_image_projection.py \
     --pcd data/sample/point_cloud.pcd \
     --image data/sample/front.jpg \
     --extrinsics data/calib/Extrinsics.txt \
     --intrinsics data/calib/Intrinsics.txt \
     --lidar LIDAR_TOP --camera CAM_P_F \
     --out output/front_projection.jpg
   ```

3. Mở `output/front_projection.jpg`. Tìm mặt đường, cạnh tòa nhà và một vùng có xe. Ghi một vùng bạn thấy hợp lý và một vùng cần kiểm thêm; không dùng số điểm được chiếu như thước đo annotation đúng.
4. Đọc mục 7 của README và ghi vào provenance khoản hiệu chỉnh tịnh tiến mà script đang cộng vào `CAM_P_F` (giá trị ghi trong README). Đây là hiệu chỉnh tác giả fit trên một ảnh; bạn không sửa giá trị để làm bài mình đẹp hơn và không sao chép sang calibration demo.

### Có ảnh kết quả là đã kiểm calibration xong chưa?

Chưa. Ảnh baseline cho biết các đầu vào mẫu có thể được đọc và chiếu trong môi trường của bạn. Chấm màu xuất hiện trên mặt đường hoặc xe là evidence cần quan sát, nhưng một ảnh đẹp chưa đủ để xác nhận nguồn gốc tọa độ, đồng bộ cảm biến hoặc độ đúng trên frame khác. README cũng nêu rõ mức bù đó chưa được chứng minh từ bản vẽ lắp đặt và chỉ được fit cho camera trước.

**Checkpoint:** mở được ảnh baseline do bạn chạy hoặc Coach cung cấp, ghi phiên bản repo và hiệu chỉnh đang dùng. Bạn cần phân biệt được “script chạy được” với “calibration đã được kiểm cho sequence của tôi”. Phần tiếp theo quay lại point cloud thật trong CVAT để xác lập track; ảnh mẫu của repo không quyết định kích thước track đó.

## Hoàn thiện một track từ frame có đủ evidence

**Đầu ra:** một track cá nhân có kích thước tham chiếu, keyframe có lý do và kiểm tra trên toàn bộ đoạn được giao.

### Vì sao không fit từ frame xa nhất?

Ở frame xa, point cloud có thể chỉ giữ lại một phần bề mặt xe. Nếu bạn co cuboid theo vài điểm đó, nhãn đang mô tả phần nhìn thấy thay vì kích thước vật rắn. Khi xe đến gần, box lại lớn lên. Mỗi frame có thể trông vừa cụm điểm, nhưng cả sequence sẽ dạy một quan hệ sai: xe thay đổi kích thước theo khoảng cách.

Bạn chọn frame đủ evidence để xác lập L/W/H, sau đó kiểm lại kích thước ấy trên các frame khác. “Frame dày” là frame giúp nhìn ranh giới object rõ; không phải cứ frame cuối hoặc gần nhất thì được dùng. Một cụm dày nhưng dính hai object cũng có thể gây fit sai. Trình tự Top → Side → Front của Lab 13 vẫn giúp kiểm footprint, chiều cao, đáy và bề ngang trước khi chốt.

Các số 4,19 × 2,04 × 1,82 m trong slide thuộc ví dụ track 94. Bạn không gõ bộ số ấy cho mọi xe. Ghi kích thước được xác lập từ object thực của mình, tên frame tham chiếu và lý do chọn frame. Nếu sau đó thấy frame tham chiếu fit sai, sửa kích thước nhất quán trên các mốc có liên quan và ghi rework; không giữ một kích thước sai chỉ để cột drift bằng 0.

### Track và keyframe khác shape rời thế nào?

**Track** nối nhiều trạng thái của cùng một object bằng identity. **Keyframe** là mốc bạn đặt hoặc chỉnh; tool nội suy trạng thái giữa các mốc. **Shape** rời mô tả một frame và không tự bảo đảm cùng identity ở frame kế tiếp. Vì thế, nhiều box cùng label không đủ chứng minh chúng là một track.

Ở đoạn giữa hai keyframe, kiểm tâm, heading và L/W/H trên các frame nội suy. Hai mốc khác kích thước có thể tạo đoạn box co giãn; đừng giả định tool giữ kích thước thay mình. Cách làm của lab là xác lập một kích thước đáng tin, rồi ưu tiên dịch tâm và xoay heading ở các mốc tiếp theo. Khi object rẽ, thêm mốc để quỹ đạo nội suy theo được cụm điểm. Không xóa hàng loạt keyframe chỉ để giảm cảnh báo: các mốc ấy có thể đang giữ đúng đường đi hoặc thời điểm che khuất.

Theo guideline Robotaxi được giao, `occluded` ghi trạng thái bị che; `outside` đánh dấu object đã ra khỏi phạm vi track/không còn cơ sở theo dõi theo rule. Một lần tạm ít điểm hoặc bị che chưa tự động là Exit. Khi không chắc có cùng object, ghi ca cần phân xử với Coach thay vì cố giữ ID bằng cách nối hai vật khác nhau.

### Tôi sẽ thao tác theo thứ tự nào?

1. **Ghi quyết định ban đầu.** Bắt đầu với J01; áp dụng lại quy trình này cho mỗi mini-job. Quan sát object mục tiêu độc lập, ghi frame định dùng để fit và lý do trước khi xem cách sửa của bạn khác. Task bắt đầu trống; Coach giữ bản nguồn riêng. Sau lần dựng track đầu tiên, nhấn Save để Coach có thể thu bản trước review.
2. **Xác lập cuboid tham chiếu trong CVAT.** Chọn frame đủ điểm, dùng Top → Side → Front và kiểm label. Nếu tạo mới, chọn `Draw new cuboid → Track`. Ghi L/W/H, heading và frame tham chiếu vào phiếu cá nhân; giữ một ảnh chụp có frame và ID hoặc chỉ trực tiếp cho Coach.
3. **Mở rộng hoặc sửa track qua thời gian.** Dịch tâm và xoay heading theo evidence của từng đoạn. Đặt thêm keyframe tại chỗ đổi hướng, Enter/Exit hoặc nơi nội suy lệch. Bạn đang làm trên task Day 14 mới. Không lấy các shape rời của job Day 13 để mặc định chúng là cùng identity trong sequence này.
4. **Kiểm đoạn thưa hoặc bị che.** Dùng kích thước tham chiếu khi evidence hỗ trợ giữ cùng vật rắn. Không tạo ID mới chỉ vì tạm mất điểm; cũng không nối hai object khi identity chưa chắc. Phân biệt `outside` với trạng thái occlusion theo guideline được giao, và ghi ca thiếu evidence để Lab Coach phân xử.
5. **Lưu để review.** Duyệt hết đoạn được giao, gồm frame nội suy, rồi nhấn Save. Báo task/job và track cho Coach để thu snapshot trước phản hồi. Tiếp tục giữ trạng thái làm việc đến khi đã rework và kiểm bản cuối.

### Khi nào track đủ để chuyển sang QC?

Bạn có thể chỉ ra một object xuyên suốt đoạn, một kích thước tham chiếu và lý do cho các mốc quan trọng. Mỗi frame thưa cần được giải thích bằng evidence của track, thay vì một box nhỏ hơn. Với heading, hãy xem đầu–đuôi và hướng chuyển động qua nhiều frame; hướng chuyển động một mình không đủ giải mọi ca xe quay đầu hoặc đi lùi.

**Checkpoint:** có track đã Save và frame tham chiếu truy vết được. Bạn đã xem mọi frame trong phạm vi track của mình, kể cả các frame do tool nội suy. Một đoạn identity còn mơ hồ phải được ghi lại, không bị giấu bằng việc merge hoặc đổi ID. QC tiếp theo sẽ giúp tìm nơi cần xem lại và kiểm xem bản export có giữ được hành vi vừa thấy trong UI hay không.

## Self-QC temporal và review trước khi rework

**Đầu ra:** đọc được QC trước/sau do Coach cung cấp, giữ một nhận xét review và cách xử lý phát hiện của track mình.

### Script chỉ ra lỗi hay chỉ ra nơi cần nhìn?

Script `cvat3d_fusion.py qc` đọc annotation export và thống kê theo track. Nó hữu ích để tìm kích thước thay đổi, góc nhảy hoặc tâm box dịch nhiều. Tuy nhiên, script không nhìn object trong ảnh và không biết toàn bộ ý nghĩa của chuyển động. Hai xe gần nhau có thể kích hoạt cờ `id_switch_risk` dù ID vẫn đúng. Một object đi vào giữa sequence có thể bị báo `fragmentation_suspect` dù đó là một lần Enter hợp lệ.

Các mặc định như drift trên 5%, đổi heading trên 25° giữa hai bản ghi hoặc khoảng cách hai tâm dưới 2,5 m là **heuristic của toolkit**. Chúng giúp bạn chọn ca xem lại; đây không phải ngưỡng đạt của học viên hoặc mức lỗi nghiệp vụ. Nếu kích thước thay đổi ít hơn 5%, script có thể im lặng trong khi quy tắc vật rắn của bài vẫn cần được kiểm. Ngược lại, nhiều cảnh báo không tự động có nghĩa là nhiều nhãn sai.

Đặc biệt, QC chỉ kiểm các item có trong file JSON, không tự tạo frame nội suy bị thiếu. Nếu export chỉ chứa keyframe, kết quả không thể chứng minh frame ở giữa đã đúng. Khoảng dịch tâm và đổi góc cũng cần đọc cùng khoảng cách frame: hai bản ghi cách nhiều frame không tương đương hai frame liên tiếp. Vì thế bạn phải giữ phạm vi export và đối chiếu số frame được kiểm.

### Năm nhóm lỗi temporal xuất hiện ra sao?

Trong lab, bạn xem identity trước/sau các ca xe sát nhau để tìm **ID switch**; xem lần kết thúc và bắt đầu để tìm **fragmentation**; so L/W/H để tìm **dimension drift**; kiểm hướng đầu xe để tìm **orientation drift hoặc flip**; và xem box ở frame ít điểm để tìm **co box hoặc nhảy box do sparsity**. Script hỗ trợ một số dấu hiệu, còn quyết định cuối cần sequence và guideline.

Một track không có cờ vẫn cần xem bằng mắt. Chẳng hạn, hai track có thể đổi object mà tâm box không nhảy quá xa. Sparsity cũng không có một cờ riêng tự chứng minh “đã xử lý đúng”. Review phải quay lại frame thực, chứ không dừng ở việc đọc CSV. Bạn giữ nhận xét ban đầu trước khi nhận phản hồi để thấy mình đã thay đổi quyết định ở đâu.

### Tôi chạy và ghi QC thế nào?

1. Nhấn Save sau lần dựng track đầu tiên. Ghi task/job, track, phạm vi và mốc đã kiểm để Coach thu đúng revision. Coach export và chạy `cvat3d_fusion.py qc` riêng; bạn không phải tải source annotation.
2. Đọc báo cáo hoặc các dòng cờ Coach cung cấp cho track của mình. Đối chiếu `track_id`, `n_frames`, `L_drift`, `W_drift`, `H_drift` với phạm vi đã làm. Nếu ID export khác ID CVAT, dùng mapping do Coach xác nhận, không ghép theo số gần giống.
3. Tự xem frame được báo, cùng frame trước/sau và frame ngoài keyframe. Ghi quan sát ban đầu: object nào, thay đổi gì, evidence nào hỗ trợ giữ hoặc sửa. Chỉ nhận trách nhiệm cho object được giao; object lân cận hỗ trợ kiểm identity hoặc lệch cả cảnh.
4. Nhờ một bạn xem một đoạn rủi ro sau self-QC. Bài vẫn nộp cá nhân. Trao đổi sau khi đã ghi quyết định ban đầu; giữ cả nhận xét của người review và lý do đồng ý/không đồng ý để Coach thu riêng.
5. Sửa ca đã xác minh hoặc ghi lý do giữ/escalate. Nhấn Save khi annotation thay đổi; Coach thu bản sau và tạo lại QC từ cùng revision. Bạn kiểm các ca vừa sửa ngay trong CVAT, không chờ số cờ để quyết định thay mình.

### Nếu cờ vẫn còn thì có nộp được không?

Điều cần có là một cách xử lý truy vết được. Cờ có thể được sửa, được giải thích là tình huống hợp lệ hoặc được escalate vì chưa đủ evidence. Bạn không đổi ngưỡng script để làm cờ biến mất. Nếu `qc_flags.csv` trống, Coach ghi rõ phạm vi đã chạy và không có cảnh báo; toolkit có thể tạo file rỗng không có header. Bạn vẫn cần giải thích đoạn khó đã tự xem.

**Checkpoint:** từ báo cáo Coach cung cấp, bạn xác định đúng track và lần theo một phát hiện đến frame thực. Nhận xét của bạn khác giúp kiểm tính nhất quán, không tự biến nhãn thành đúng. Coach giữ QC trước/sau tương ứng với hai snapshot. Nếu bạn đã sửa sau báo cáo gần nhất, báo lại để Coach thu bản mới trước khi đối chiếu fusion.

## Đối chiếu Camera–LiDAR và ghi discrepancy có căn cứ

**Đầu ra:** một tập overlay trên ảnh thật ứng với track, cùng quyết định sửa, giữ hoặc escalate cho các ca đã xem.

### Tôi có đang xem ảnh đúng frame không?

Trước khi đọc hình chiếu, kiểm manifest của sequence thật. Tên frame trong CVAT, `items[].id` trong export và tên ảnh/PCD có thể khác nhau. Lab Coach cung cấp mapping rõ để bạn biết ảnh nào thuộc frame nào. Không chọn ảnh chỉ vì filename có cùng vài chữ số. Một ảnh nhầm thời điểm vẫn có thể có cùng chiếc xe và tạo overlay có vẻ gần đúng.

Coach dùng runtime map chính xác để ghép annotation, ảnh và PCD; CLI overlay dừng nếu thiếu media, mapping mơ hồ hoặc kích thước ảnh sai. Bảng frame-map công khai giúp định vị case, không thay cho runtime map riêng có đường dẫn media. Bạn vẫn phải mở kết quả và kiểm lại ảnh gốc, local frame, camera cùng nguồn calibration trước khi nhận xét sự khớp; chạy được script không chứng minh calibration đúng.

### Tôi dùng calibration nào cho sequence thật?

Coach chuẩn bị overlay từ PCD/ảnh thật và annotation đã lưu của bạn. Overlay của Coach **không** dùng nguyên chuỗi baseline của repo. PCD mẫu đã được tịnh tiến về gốc ego nhưng chưa xoay, nên profile chỉ giữ phần xoay yaw của `LIDAR_TOP`, bỏ tịnh tiến của nó và bỏ khoản bù của script; `CAM_P_F` dùng ego → camera gốc của repo. Coach đã kiểm chuỗi này trên 66 frame bằng ba cách độc lập (ICP giữa các PCD, khớp cạnh LiDAR–ảnh, so cuboid chiếu với box phát hiện): lệch còn khoảng 2 px trên ảnh camera trước. Script baseline của repo vẫn cộng khoản bù đó; bạn ghi lại điều đó như một khác biệt giữa hai công cụ, không sửa số để ảnh của mình khớp.

Trường `pcd_frame` cho biết toolkit có đưa điểm/cuboid từ LiDAR sang ego hay không. Chọn `lidar` thì có bước ấy; chọn `ego` thì bỏ qua. Profile hiện tại để `lidar`, với `sensor2ego` chỉ chứa phần xoay nói trên. Học viên không tự đổi các số này để làm ảnh khớp.

Toolkit của artifacts dùng pinhole Brown–Conrady với tối đa năm hệ số. Repo hỗ trợ thêm camera fisheye và các trường hợp pinhole khác. Trong lab, đọc overlay camera trước trên ảnh 1920 × 1536. Các ảnh ngữ cảnh khác giúp nhận diện vật; chưa dùng chúng với profile camera trước. Khi không rõ camera model, hệ tọa độ hoặc timing, giữ quan sát thật và ghi giới hạn của kết luận.

### Tôi phân loại một chỗ không khớp bằng cách nào?

Ảnh hỗ trợ nhận class, số object và đầu–đuôi xe. Kích thước, tâm và fit hình học vẫn dựa trên point cloud cùng evidence của track. Khi camera không thấy xe, chưa thể kết luận xe không tồn tại: nó có thể ngoài FOV hoặc bị che riêng ở sensor đó. Khi camera thấy rõ nhưng cloud thưa, giữ kích thước đã xác lập nếu identity được hỗ trợ; không suy khoảng cách và cuboid mới chỉ từ pixel.

Lệch cùng chiều trên nhiều object và nhiều frame là dấu hiệu cần điều tra pipeline, gồm calibration hoặc ghép dữ liệu. Đó chưa phải phép đo tự chứng minh calibration sai. Một box lệch riêng có thể là annotation, nhưng vẫn cần kiểm object, heading và cặp file. Bạn ghi điều quan sát được trước, giả thuyết sau; dùng `chưa đủ evidence` khi chưa phân biệt được các nguyên nhân.

1. **Kiểm đầu vào thật.** Mở ảnh của frame tham chiếu và một frame khó của track; đối chiếu index CVAT, filename gốc, camera và resolution với mapping. Đọc ghi chú correction/hệ tọa độ của profile trước khi nhận xét sự lệch.
2. **Đọc overlay bản đã Save.** Coach cấp ảnh overlay và các dòng `projection.csv` ứng với snapshot cá nhân. Kiểm task/job/track/revision; nếu bạn sửa sau thời điểm thu, báo để tạo lại ảnh. Không đọc overlay annotation nguồn như kết quả bài của mình.
3. **Đối chiếu hình và bảng.** Xem frame dày, frame thưa và đoạn dễ nhầm identity. Các status `fully_visible`, `partial`, `out_of_fov` chỉ kiểm vị trí góc box trên ảnh, không là nhãn che khuất thực tế; xác nhận FOV bằng ảnh và geometry.
4. **Ghi discrepancy trước khi sửa.** Trong phiếu cá nhân, ghi frame/camera/track, quan sát 2D, quan sát 3D, nguyên nhân nghi ngờ, evidence và hành động. Coach thu vào log riêng. Phân biệt annotation, FOV, occlusion, sparse points, nghi calibration/pipeline và chưa đủ evidence.
5. **Thực hiện hành động có căn cứ.** Lỗi annotation thì sửa trong job 3D và Save; Coach tạo lại QC/overlay liên quan. Ca FOV, che khuất hoặc thưa điểm thì ghi lý do giữ nhãn theo guideline. Nghi lỗi hệ thống thì báo Coach cùng frame/evidence; không kéo cuboid khỏi point cloud hoặc chỉnh calibration để che lệch.

### Tôi có cần tạo task 2D mới không?

Bằng chứng tối thiểu là overlay thật và ghi chú truy vết được đến track/frame; Coach giữ CSV riêng. Bạn có thể xem ảnh bằng trình xem ảnh trên máy. Nếu Lab Coach đã tạo task 2D review, dùng Issue trên ảnh để nhận xét và ghi Issue ID vào log; đó là một bản sao phục vụ review, không phải nơi vẽ lại nhãn 3D. Bạn không cần tự tạo thêm task để hoàn thành bài.

**Checkpoint:** người review lần được ít nhất một ca bình thường và một ca khó từ ghi chú đến ảnh thật và cuboid. Nếu đầu vào thật thiếu, báo rõ blocker và phần đã làm cho Coach; evidence demo không được ghi là đã hoàn thành fusion thật. Cách ghi giới hạn này giúp Lab Coach phân biệt lỗi học viên với lỗi gói dữ liệu.

## Rework, Save và completed trên CVAT

**Đầu ra:** annotation cuối đã lưu trong job cá nhân; Coach có thể thu và review cùng evidence của bạn.

### Save và completed xác nhận điều gì?

**Save** lưu annotation đang làm lên CVAT. Trạng thái **completed** báo rằng bạn đã kết thúc lần làm bài và sẵn sàng để Coach thu. Hai thao tác có trách nhiệm khác nhau: chuyển trạng thái không thay thế kiểm rằng thay đổi đã được lưu. Completed cũng không phải điểm số hay kết luận annotation đúng; Coach vẫn xem identity, geometry, nội suy và cách xử lý ca khó.

Coach thu bản trước/sau rework và phản hồi riêng. Giữ một quyết định sau review cùng frame/evidence và lý do; ca còn mở cần có người nhận escalation.

### Tôi kết thúc bài thế nào?

1. Kiểm lại track trong CVAT: đúng object/label, L/W/H có căn cứ, heading hợp lý và đã xem toàn bộ frame của đoạn. Xem lại một frame nội suy cùng một ca khó sau rework.
2. Hoàn thiện ghi chú gồm task/job/track, phạm vi, frame tham chiếu, L/W/H, keyframe quan trọng, một quyết định sau review và discrepancy còn mở. Ghi thêm case/lượt, job đã bắt đầu và đã completed. Coach thu riêng; không nộp ZIP lên VLearn.
3. Nhấn **Save** và chờ thao tác lưu hoàn tất. Tải lại job để xác nhận cuboid/keyframe mới nhất còn đúng. Nếu có lỗi lưu hoặc mất dữ liệu, báo Coach trước khi chuyển trạng thái.
4. Chuyển từng job đã làm xong sang **completed** như Day 12. Kiểm trạng thái trong danh sách Jobs; báo bộ job, phần chưa xong và ca cần theo dõi cho Coach. Khi Coach yêu cầu rework, mở lại theo quyền được cấp, sửa, Save và completed lại.

**Checkpoint cuối:** job hiển thị completed và mở lại giữ annotation cuối. Coach thu/review riêng. Nếu bị chặn bởi đầu vào/tool, báo phần đã làm và blocker trước khi đóng job. Mốc nộp theo thông báo lớp.

Nếu cần hỗ trợ, nhờ kiểm checkpoint trên J01. Nếu còn thời gian trước phút 160, đi tiếp lượt ngẫu nhiên đã giao; ở mức cao, giải thích thêm một cờ có thể false positive hoặc một giả định phép chiếu.
