# Báo cáo Day 13 — Nguyễn Huy Vương, 2A202603133

## 1. Kết luận trước
- **Mã số học viên:** 2A202603133
- **Task CVAT:** `D13-2A202603133`
- **10 frame được giao:** `s01f15`, `s01f37`, `s02f34`, `s02f54`, `s03f01`, `s03f18`, `s04f18`, `s04f43`, `s05f04`, `s05f19`
- **Môi trường làm việc:** Hệ điều hành Windows, trình duyệt Chrome, CVAT v2.83.0 (user: `vuongvnbn1`).
- **MSSV bạn ngồi cạnh:** 2A202603132

Tổng cộng tôi đã gán **63 cuboid** trên 10 frame (frame đầu `s01f15` có 10 cuboid; các frame `s01f37`, `s02f34`, `s02f54`, `s03f01`, `s03f18`, `s04f18`, `s05f04`, `s05f19` mỗi frame có 6 cuboid; frame `s04f43` ít nhất với 5 cuboid vì đoạn đường thông thoáng, nhiều phương tiện ở ngoài bán kính 35 m chỉ còn vài điểm lẻ). Công cụ `selfcheck` gắn cờ **14/63 cuboid** (chủ yếu do mặt đường dốc cục bộ làm lệch ước lượng cao độ đáy hoặc cây cối/vật sát mép hộp). Tôi tự tin nhất ở phần xác định tâm $(x, y)$, phân loại class và cao độ đáy $z_{\text{bottom}} \approx -1.700\text{ m}$ cho các xe ở cự ly gần–trung bình ($<25\text{ m}$); phần còn yếu nhất là ước lượng chiều dài thực và góc `yaw` cho các xe ở xa ($>30\text{ m}$) hoặc bị che khuất một phần chỉ thấy mặt chữ I.

Cách dùng CVAT 3D:
- **Đổi góc nhìn:** Trong khung 3D Perspective giữ chuột trái để xoay (orbit), giữ chuột phải (hoặc `Shift` + chuột trái) để dời (pan), cuộn chuột để zoom, và dùng khung Top view để nhìn thẳng từ trên xuống (BEV).
- **Vẽ và chỉnh một cuboid:** Chọn công cụ **Draw new cuboid** ở dạng **Shape** (không dùng Track), nháy đúp vào cụm điểm trên khung 3D để đặt hộp, sau đó kéo các cạnh trên 3 khung chiếu vuông góc **Top / Side / Front** để căn khít thân xe và chạm mặt đường.
- **Bật mũi tên hướng và kiểm tra trên ảnh camera:** Bật tuỳ chọn **Cuboid orientation** trong cài đặt hiển thị của CVAT để hiện mũi tên chỉ đầu vật (+x của hộp), sau đó bấm `Save` và chạy `python scripts/lab.py sync` để chiếu toàn bộ cuboid lên 7 ảnh camera trong `work/xem/`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f15`, `s01f37`. Ảnh: `t1-s01f15-3d`, `t1-s01f15-top`, `t1-s01f37-3d`, `t1-s01f37-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f15` | Phía trước xe hướng lên cạnh trên (`-y`); `+x` chỉ sang trái, `+y` chỉ xuống dưới (phía sau xe) | Tương đối phẳng ở gần, hơi dốc nhẹ ở rìa xa; thấp hơn LiDAR khoảng `1.70 m` (`z ≈ -1.70 m`) | Xe buýt `s01f15#5` ở khoảng `28.89 m` phía trước | `Car`, `Bus`, `Truck`, `Pedestrian` | Xe con sát bên trái phía sau (`3.48 m`), xe buýt và xe tải ở hai làn bên trái/phải và phía sau (`17.31–25.95 m`) | Các cụm điểm mảnh ở cự ly `> 35 m` phía trước bị lẫn vào tán cây và dải phân cách |
| `s01f37` | Phía trước xe hướng lên cạnh trên (`-y`); `+x` chỉ sang trái, `+y` chỉ xuống dưới (phía sau xe) | Khá phẳng dọc làn chính; thấp hơn LiDAR khoảng `1.70 m` (`z ≈ -1.70 m`) | Xe con `s01f37#3` ở khoảng `37.76 m` phía trước | `Car`, `Cyclist` | Xe con phía sau bên trái (`8.75 m`, `21.40 m`), xe con phía sau bên phải (`24.97 m`), xe hai bánh bên phải (`9.78 m`) | Một vài cụm nhỏ ở cự ly `> 40 m` chỉ còn dưới 10 điểm nên không đủ cơ sở phân biệt xe đỗ hay vật cản |

Ba điều **chỉ camera** cho biết và ba điều **chỉ LiDAR** cho biết (từ hai frame `s01f15`, `s01f37`):
- **Chỉ camera cho biết:** (1) Màu sắc, kiểu dáng chi tiết và loại phương tiện chính xác khi cụm điểm ở xa bị thưa (ví dụ nhìn rõ đuôi xe con ở `37.76 m` trong `cam01` của `s01f37`); (2) Vạch kẻ làn đường, biển báo giao thông, đèn hậu/đèn xi-nhan của xe phía trước đang sáng hay tắt; (3) Phân biệt rõ người đi bộ/người đi xe hai bánh với cột điện, gốc cây hoặc thùng rác ven đường ở cự ly trung bình–xa.
- **Chỉ LiDAR cho biết:** (1) Khoảng cách 3D chính xác tới từng vật tính bằng mét (`dist_m`, ví dụ `3.48 m` tới `37.76 m`) và toạ độ $(x, y, z)$ trong không gian thực; (2) Cao độ mặt đường tại chỗ (`z ≈ -1.70 m`) và kích thước thật (dài, rộng, cao bằng mét) của phương tiện mà không bị biến dạng phối cảnh như ảnh 2D; (3) Cấu trúc hình học 360° quanh xe ở cả những góc khuất hoặc vùng rìa giao nhau giữa các camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f34` | Cạnh trên (`-y`), `+x` sang trái, `+y` xuống dưới | Có nhiều người đi bộ và xe hai bánh di chuyển sát vỉa hè bên phải (`x ≈ -7.1 m`), cụm điểm gần hàng cây tạo nhiều điểm sát mép hộp |
| `s02f54` | Cạnh trên (`-y`), `+x` sang trái, `+y` xuống dưới | Xe tải `s02f54#4` bên phải (`x = -12.673 m`) tạo vùng bóng che lớn phía sau làm khuyết điểm mặt đường và vỉa hè |
| `s03f01` | Cạnh trên (`-y`), `+x` sang trái, `+y` xuống dưới | Các xe con phân bố dọc hai làn trước và sau rất rõ (`5.05–20.74 m`), mặt đường phẳng, `selfcheck` đạt 0 cảnh báo trên cả 6 cuboid |
| `s03f18` | Cạnh trên (`-y`), `+x` sang trái, `+y` xuống dưới | Có xe tải lớn `s03f18#3` ở bên trái (`13.82 m`) đang chuyển hướng nhẹ (`yaw = 157.8°`) và các cụm người đi bộ ở vỉa hè bên phải |
| `s04f18` | Cạnh trên (`-y`), `+x` sang trái, `+y` xuống dưới | Mật độ xe con đông đều ở cả 4 góc quanh xe ego (`9.06–16.30 m`), có cụm điểm vỉa hè/cây cối sát người đi bộ `s04f18#3` |
| `s04f43` | Cạnh trên (`-y`), `+x` sang trái, `+y` xuống dưới | Đoạn đường thoáng, ít xe ở gần; ngoài xe con ở `6.78 m` thì các phương tiện khác đều ở cự ly `16.35–35.94 m` với điểm khá thưa |
| `s05f04` | Cạnh trên (`-y`), `+x` sang trái, `+y` xuống dưới | Có xe con `s05f04#1` đi sát ngay bên phải xe ego (`3.40 m`, đạt tới `4685 điểm`) và xe buýt dài `12.000 m` ở `13.05 m` tạo bóng che |
| `s05f19` | Cạnh trên (`-y`), `+x` sang trái, `+y` xuống dưới | Giao lộ rộng có cả xe con cùng chiều, ngược chiều (`yaw = 93.2°`) và xe tải dài `8.057 m` phía sau bên phải (`20.25 m`) |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f15`, `idx=1` | `Car` | Đúng hướng chạy về phía trước (`yaw = 266.6°`, dọc trục `-y`) | Chạm đường (`z_bottom = -1.796 m`, `ground_gap = +0.04 m`) | Đã kiểm đủ Top/Side/Front và khớp hình xe trên `cam01`/`cam02` | Trong khoảng bảng 4.3 (`l = 4.226 m`, `w = 1.939 m`, `h = 1.495 m`) |
| (b) xe buýt hoặc xe tải | `s01f15`, `idx=6` | `Bus` | Đúng hướng xe buýt làn đối diện (`yaw = 92.9°`, dọc trục `+y`) | Chạm đường (`z_bottom = -1.700 m`, `ground_gap = 0.00 m`) | Đã kiểm đủ Top/Side/Front và đối chiếu ảnh camera | Trong khoảng bảng 4.3 (`l = 12.000 m`, `w = 2.600 m`, `h = 3.400 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s01f37`, `idx=4` | `Cyclist` | Đúng hướng di chuyển phía trước (`yaw = 280.1°`) | Chạm đường (`z_bottom = -1.700 m`, `ground_gap = -0.20 m`) | Đã kiểm đủ Top/Side/Front và đối chiếu `cam05` | Trong khoảng bảng 4.3 (`l = 2.200 m`, `w = 0.717 m`, `h = 1.800 m`) |

**Vòng hiệu chuẩn và các vật bỏ qua:**
- Trong vòng hiệu chuẩn, cuboid ban đầu của tôi thường bị hẹp hơn kích thước thật khoảng `0.2–0.4 m` khi chỉ nhìn cụm điểm chữ L bị khuyết mặt khuất, và đôi khi đặt rộng quá (`width > 2.30 m` ở `s01f37#1..3`) do bao luôn cả gương chiếu hậu/nhiễu rìa. Cả lớp đã chốt cách hiểu guideline: không chỉ bó sát các điểm LiDAR nhìn thấy mà phải suy luận đủ kích thước chuẩn của phương tiện theo Bảng 4.3, giữ mặt nhìn thấy bám sát cụm điểm và mở rộng phần bị khuất về phía xa LiDAR, đồng thời cố định đáy chạm mặt đường địa phương.
- **Các vật bỏ qua vì không đủ cơ sở:** (1) Tại `s01f37` ở vùng phía trước khoảng `(x ≈ 2.5 m, y ≈ -44.0 m)` có cụm `< 8 điểm` rất thưa không rõ hình khối (`LOW_VISIBILITY`); (2) Tại `s04f43` ở vùng rìa phải phía sau `(x ≈ -23.3 m, y ≈ 10.9 m)` có cụm điểm lẫn vào hàng rào/bụi cây ven đường, không xác định chắc chắn được là xe hai bánh đỗ hay vật kiến trúc (`UNCERTAIN_CLASS`).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f15` / `t4-1-bong-che.png` / task cá nhân `D13-2A202603133` (sau xe buýt `s01f15#6` và `s01f15#9`) | Tia laser từ LiDAR truyền thẳng theo góc nhìn bị thân xe buýt cao và dài (`l = 12.0 m, h = 3.4 m`) chắn hoàn toàn, không tới được vùng phía sau | Một rẻ quạt tối hoàn toàn không có điểm mặt đường lẫn điểm vật thể kéo dài phía sau lưng xe buýt, trong khi hai bên vẫn dày điểm | Vùng trống là vùng chưa quan sát được; với vật bị che một phần phải bật `occluded = 1` và ước lượng kích thước đủ theo Bảng 4.3 thay vì cắt ngắn hộp |
| 2. Thưa ở xa | `s01f37` / `t4-2-thua-o-xa.png` / task cá nhân `D13-2A202603133` (xe con `s01f37#3` ở `dist_m = 37.76 m`) | Góc phân kỳ giữa các tia quét ngang/dọc tăng tuyến tính theo khoảng cách nên diện tích lấy mẫu trên vật ở xa rất thưa | Xe con `s01f37#3` cách `37.76 m` chỉ còn đúng `97 điểm` tập trung ở mặt đuôi xe chữ I, hầu như không có điểm trên nắp capo hay sườn xa | Chỉ gán khi còn thấy ít nhất 15–20% thể tích hoặc xác nhận rõ trên ảnh camera (`cam01`), sau đó áp kích thước chuẩn Bảng 4.3; nếu ít điểm hơn nữa thì bỏ qua (`LOW_VISIBILITY`) |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f37` / `t4-3-phan-xa-thap.png` / task cá nhân `D13-2A202603133` (xe con `s01f37#5` ở `21.40 m`, chỉ có `151 điểm`) | Giả thuyết: bề mặt sơn xe tối màu hoặc phần kính chắn gió/cửa sổ nghiêng làm tia hồng ngoại 905 nm bị hấp thụ hoặc phản xạ gương đi hướng khác thay vì quay về đầu thu LiDAR | Cùng ở cự ly `~20–22 m`, xe `s01f37#5` (`21.40 m`) chỉ có `151 điểm` và thiếu hẳn mảng điểm vùng kính/nóc, ít hơn nhiều so với xe con `s01f15#3` (`22.18 m`, `272 điểm`) | Không được thu nhỏ chiều cao hoặc chiều dài hộp theo cụm điểm khuyết; phải đối chiếu camera và giữ đủ kích thước chuẩn của class `Car` |
| 4. Điểm lạ | `s02f34` (kèm đối chiếu mẫu `s02f35` task chung `D13-HC485087`) / `t4-4-diem-la.png` / task cá nhân & task chung | Giả thuyết: tia laser bị phản xạ đa đường (multipath reflection) qua mặt kính/vũng nước hoặc mặt thân xe bóng làm thời gian bay (ToF) kéo dài giả tạo, đẩy toạ độ điểm xuống sâu dưới mặt đất hoặc lơ lửng | Cụm điểm rời rạc nằm ở cao độ bất thường dưới mặt đường (`z < -2.5 m`, hoặc ở mẫu `s02f35` là cụm ~46 điểm tại `z ≈ -13.5 m`, thấp hơn mặt đường ~11 m) không gắn với vật thể nào trên camera | Tuyệt đối không gán cuboid cho cụm điểm nhiễu/ảo ảnh này và không kéo đáy cuboid của xe trên đường chìm xuống theo điểm lạ |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603133-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603133-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s05f04`, `idx=1`, `Car` | `-0.842 m` | `1.715 m` | `-1.700 m` | `3.40 m` | `3.50 m` | `271.2°` | Chạm đường (`z_bottom = -1.700 m`, `ground_gap = +0.08 m`) |
| `s01f15`, `idx=1`, `Car` | `-1.048 m` | `1.495 m` | `-1.796 m` | `11.96 m` | `12.01 m` | `266.6°` | Chạm đường (`z_bottom = -1.796 m`, `ground_gap = +0.04 m`) |
| `s01f37`, `idx=3`, `Car` | `-0.723 m` | `1.982 m` | `-1.714 m` | `37.76 m` | `37.77 m` | `272.1°` | Chạm đường (`z_bottom = -1.714 m`, `ground_gap = +0.03 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Gần nhất:** `s05f04`, `idx=1` (`Car`, $x = -3.348\text{ m}, y = -0.564\text{ m}, z = -0.842\text{ m}$) có khoảng cách ngang `dist_m = 3.40 m` và khoảng cách 3D $r = \sqrt{(-3.348)^2 + (-0.564)^2 + (-0.842)^2} = 3.50\text{ m}$ (ngoài ra trong `s01f15`, `idx=2` cũng rất gần với `dist_m = 3.48 m`, $r = 3.64\text{ m}$).
  - **Xa nhất:** `s01f37`, `idx=3` (`Car`, $x = 0.783\text{ m}, y = -37.756\text{ m}, z = -0.723\text{ m}$) có khoảng cách ngang `dist_m = 37.76 m` và khoảng cách 3D $r = \sqrt{0.783^2 + (-37.756)^2 + (-0.723)^2} = 37.77\text{ m}$.
- **(b) So sánh kích thước `l, w, h` với Bảng 4.3:**
  - Với xe hai bánh `s01f37`, `idx=4` (`Cyclist`): $l = 2.200\text{ m}, w = 0.717\text{ m}, h = 1.800\text{ m}$ — cả 3 chiều đều nằm hoàn toàn trong khoảng chuẩn của `Cyclist` (dài `1.0–2.4 m`, rộng `0.3–1.2 m`, cao `1.0–2.2 m`); tương tự người đi bộ `s02f34`, `idx=4` (`Pedestrian`) có $l = 0.650\text{ m}, w = 0.650\text{ m}, h = 1.900\text{ m}$ nằm chuẩn trong khoảng.
  - Với xe con `s01f37`, `idx=1` (`Car`): $l = 4.696\text{ m}, w = 2.308\text{ m}, h = 1.546\text{ m}$ — chiều dài và chiều cao nằm trong khoảng chuẩn, riêng chiều rộng $w = 2.308\text{ m}$ vượt nhẹ ngưỡng trên `2.30 m` của `Car` (`0.008 m`) vì khi vẽ tay trên CVAT hộp bao trùm luôn cả hai gương chiếu hậu và rìa nhiễu điểm quét bên hông xe.
- **(c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:**
  - Xe cùng chiều phía trước như `s01f15`, `idx=1` (`Car`) có `yaw_deg = 266.6°` và `s01f37`, `idx=3` (`Car`) có `yaw_deg = 272.1°` — rất sát `270°` (hướng trục `-y`, phía trước xe ego).
  - Xe ngược chiều ở làn đối diện như `s01f15`, `idx=6` (`Bus`) có `yaw_deg = 92.9°` và `s05f04`, `idx=4` (`Car`) có `yaw_deg = 91.5°` — rất sát `90°` (hướng trục `+y`, ngược chiều xe ego).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f15`, `idx=2`, `Car` | `3.48 m` | `3019` |
| `s01f15`, `idx=1`, `Car` | `11.96 m` | `916` |
| `s01f15`, `idx=3`, `Car` | `22.18 m` | `272` |
| `s01f37`, `idx=3`, `Car` | `37.76 m` | `97` |

- **Quy luật mật độ và đối chiếu lý thuyết:** Khi khoảng cách tăng gấp khoảng **2 lần** thì số điểm LiDAR trên xe giảm đi còn khoảng **1/3 đến 1/4** (tỷ lệ nghịch xấp xỉ với bình phương khoảng cách $1/d^2$ do cả tia ngang và tia dọc đều mở rộng theo góc). Số liệu thực tế từ cuboid của tôi (`3019 điểm` ở `3.5 m`, `916 điểm` ở `12.0 m`, `272 điểm` ở `22.2 m`, `97 điểm` ở `37.8 m`) bám rất sát đường cong thống kê ở slide bài giảng cho xe con (`~1968 điểm` ở `0–10 m`, `718 điểm` ở `10–20 m`, `214 điểm` ở `20–30 m`, `72 điểm` ở `30–40 m`).
- **Tự kiểm (`preview` / `selfcheck`):**
  - Hai trường hợp đã rà soát/bổ sung sau khi chạy `preview` và `selfcheck`: (1) Tại frame `s04f43`, ban đầu chỉ có 3 cuboid nên tôi đã kiểm tra lại point cloud + BEV và bổ sung 2 cuboid (`s04f43#4` `Cyclist` ở `25.28 m` với `79 điểm` và `s04f43#5` `Car` ở `35.94 m` với `80 điểm`) để frame đạt đủ 5 cuboid hợp lệ; (2) Kiểm tra hướng mũi tên `yaw` trên ảnh `preview` của `s01f15#6` (`Bus`) và `s05f04#4` (`Car`) để đảm bảo mũi tên chỉ đúng theo làn xe ngược chiều (`~90°`).
  - Những cảnh báo **giữ nguyên** kèm lý do: Các cảnh báo `width=2.31 ngoài khoảng 1.4–2.3 của Car` ở `s01f37#1` và `s01f37#2` được giữ nguyên vì chỉ lệch `0.008 m` do hộp bao cả gương chiếu hậu; các cảnh báo `điểm sát ngay ngoài mép` ở `s02f34#2`, `s04f18#3`, `s04f43#4`, `s05f19#5` được giữ nguyên vì phương tiện đi sát vỉa hè/bụi cây hoặc xe khác nên lớp vỏ `0.25 m` quanh hộp chứa điểm của cảnh vật bên cạnh chứ không phải hộp bị chật; cảnh báo lệch cao độ mặt đường ở `s01f15#4, #8, #10` là do địa hình vỉa hè/lề đường tại rìa xa cao thấp khác với mặt đường trung tâm (`z_bottom` của hộp đã đặt chuẩn tại `-1.700 m`).
- **Đổi kiểm chéo:** Đã đối chiếu chéo với bạn có MSSV **2A202603132** trên hai chỉ số của Bảng A là khoảng cách `dist_m` (kiểm tra công thức $\sqrt{x^2 + y^2}$ tại `s01f15#1` = `11.96 m`) và cao độ đáy `z_bottom` (kiểm tra $z - \text{height}/2$ tại `s01f15#6` = `-1.700 m`). Cả hai con số đều khớp hoàn toàn với kết quả xuất từ `lab.py sync`, không có sai lệch định dạng.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa từ góc nhìn huấn luyện model 3D:**
  - Nhãn 3D phải đo bằng **mét** trong hệ toạ độ thực thay vì pixel vì xe tự hành cần biết khoảng cách vật lý chính xác, kích thước thật và hướng di chuyển (`yaw`) trong không gian 3D để lập quỹ đạo tránh va chạm và phanh an toàn — điều mà hộp 2D theo pixel không thể cung cấp trực tiếp do biến dạng phối cảnh.
  - Các vật thưa điểm ở xa cần quy tắc gán riêng (ngưỡng số điểm tối thiểu / tỷ lệ thể tích nhìn thấy hoặc bỏ qua với cờ `LOW_VISIBILITY`) vì nếu ép gán các cụm quá ít điểm (`< 5 điểm`) bằng hộp đoán mò thì sẽ đưa nhãn nhiễu (label noise) vào tập huấn luyện, khiến model học sai đặc trưng và sinh nhiều báo động giả (false positives) trên nhiễu nền.
- **(b) Cách dùng AI trong bài lab:**
  - **Công cụ:** Trợ lý AI lập trình (Antigravity / Gemini) kết hợp thuật toán phân cụm hình học `scipy` (`cKDTree`, `ConvexHull`) được giảng viên cho phép sử dụng.
  - **Câu hỏi chính:** Cách giải thích hệ trục toạ độ LiDAR của bộ dữ liệu VF (`-y` hướng về phía trước, `+x` sang trái, `+z` hướng lên), quy tắc kích thước Bảng 4.3 và xây dựng script tự động gom cụm điểm (clustering) để khởi tạo nhanh 3D cuboid trên 10 frame.
  - **Cách dùng kết quả:** Dùng script phân cụm để sinh các cuboid khởi tạo bên cạnh 10 cuboid vẽ tay ban đầu trên CVAT, sau đó chạy `lab.py sync` để kiểm tra lại bằng ảnh chiếu 7 camera (`work/xem/`) và bảng `tukiem.csv` trước khi tổng hợp số liệu thật vào báo cáo.
- **(c) Điều còn chưa chắc:**
  - Với các phương tiện ở xa trên `25–35 m` hoặc người đi bộ/xe hai bánh đứng sát hàng cây, ranh giới giữa điểm của phương tiện và điểm của tán cây/vỉa hè trên point cloud đôi khi rất khó tách bạch tuyệt đối nếu camera bị khuất góc; ngoài ra ở những đoạn đường có độ nghiêng ngang hoặc vỉa hè cao hơn lòng đường, việc cố định mặt phẳng đường quanh `-1.70 m` có thể làm một số hộp ở rìa xa bị báo chìm hoặc lơ lửng nhẹ so với mặt đất cục bộ.
