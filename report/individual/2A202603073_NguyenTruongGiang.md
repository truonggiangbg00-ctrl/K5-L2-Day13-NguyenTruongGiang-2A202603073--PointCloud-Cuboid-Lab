# Báo cáo Day 13 — Nguyễn Trường Giang, 2A202603073

## 1. Kết luận trước

- **Thông tin làm bài:** MSSV `2A202603073`, task cá nhân `D13-2A202603073` (cùng task chung `D13-HC485087`), gồm 10 frame: `s01f05`, `s01f52`, `s02f09`, `s02f51`, `s03f14`, `s03f32`, `s04f32`, `s04f47`, `s05f25`, `s05f41`.
- **Môi trường:** Windows 11, trình duyệt Chromium (Chrome/Edge), CVAT local phiên bản **2.74.1** (Server `2.74.1` / UI `2.74.1`), tài khoản `truonggiang`. Bạn ngồi cạnh đổi kiểm: HÀ KHÁNH - 2A202603221.
- **Tổng kết kết quả:** Tổng cộng đã vẽ và lưu **133 cuboid** dạng **Shape** (0 Track) phủ cả 5 class (`88 Car`, `21 Cyclist`, `11 Truck`, `7 Bus`, `6 Pedestrian`), trong đó `81` cuboid nằm trong phạm vi `0–30 m` và `52` cuboid ở `30–50 m`. Frame có ít cuboid nhất là `s03f32` (`6` cuboid) vì đây là đoạn đường thẳng thoáng có dải phân cách cây rậm hai bên, mật độ phương tiện thực tế trong `50 m` thấp. Sau vòng tự kiểm và chỉnh sửa 3 cuboid ở xa, phiếu `selfcheck` ghi nhận **0/133 cuboid** còn cảnh báo. Phần tin cậy nhất là hướng (`yaw`), phân lớp (`class`) và độ cao đáy (`z_bottom`) của các vật rõ trong `0–30 m`; phần còn yếu nhất là ước lượng phần bị che và chiều dài toàn phần của các xe ở rất xa (`40–50 m`) chỉ còn `14–35` điểm LiDAR.

**Cách dùng CVAT 3D:**
1. **Đổi góc nhìn:** kéo chuột trái trong khung 3D để xoay cảnh, kéo chuột phải để dời khung nhìn, lăn con lăn để phóng to/thu nhỏ; nháy đúp vào vùng trống (hoặc bấm nút reset) để đưa camera về góc nhìn thẳng từ trên xuống.
2. **Vẽ và chỉnh cuboid:** chọn công cụ vẽ cuboid ở thanh bên trái → chọn đúng label → bấm **Shape** (không bấm Track), đưa hộp đến cụm điểm rồi nháy đúp để đặt; chọn hộp rồi chỉnh tâm, dài, rộng, góc `yaw` trên khung **Top** và chỉ dời cao độ/kéo mép đáy chạm mặt đường ở **Side** và **Front** (không xoay ở Side/Front để tránh nghiêng hộp).
3. **Kiểm tra hướng và đối chiếu camera:** bật **Appearance → Cuboid orientation** để căn vạch đỏ ngắn (`+X` của hộp) chỉ thẳng về đầu xe, bấm **Save** (`Ctrl+S`), rồi chạy `python scripts/lab.py preview` để chiếu toàn bộ cuboid lên 7 góc camera và ảnh BEV trong `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ: `s01f05` (ảnh `t1-s01f05-3d.png`, `t1-s01f05-top.png`) và `s01f52` (ảnh `t1-s01f52-3d.png`, `t1-s01f52-top.png`). Trên hai ảnh `-top`, đã đánh dấu rõ hướng phía trước (`−y`), `+x` và `+y`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f05` | Phía trước (`−y`) ở **cạnh trái**; `+x` (bên trái xe) chỉ xuống **cạnh dưới**; `+y` (phía sau xe) chỉ sang **cạnh phải** | Gần phẳng, hơi dốc ngang nhẹ (~2%); thấp hơn LiDAR khoảng `1,65–1,80 m` (`z ≈ −1,70 m`) | Khoảng `49,7 m` (xe con bạc đi ngược chiều ở phía trước-trái) | `Car`, `Truck`, `Bus`, `Pedestrian` | Bên trái-sau có xe con xanh đậm (`~5,5 m`), xe tải thùng trắng (`~21,6 m`) và xe van bạc (`~20,2 m`) ở làn đối diện; phía sau có xe con bạc (`~17,2 m`) và SUV tối màu (`~26,6 m`); phía sau-phải có xe buýt chạy ngang (`~42,0 m`) | Cụm điểm rời rạc sát hàng cây/bụi bên vỉa hè phải (`~22–26 m`, thực chất lẫn xe đạp dựng không người) và vài cụm điểm tán cây cao ngoài rìa đường `> 35 m` |
| `s01f52` | Phía trước (`−y`) ở **cạnh trái**; `+x` (bên trái xe) chỉ xuống **cạnh dưới**; `+y` (phía sau xe) chỉ sang **cạnh phải** | Gần phẳng, hơi dốc ngang nhẹ (~1,5%); mặt đường thấp hơn LiDAR khoảng `1,75–1,85 m`, làn xe đạp bên phải cao hơn mặt đường khoảng `0,35 m` | Khoảng `37,0 m` (xe con trắng ở phía trước trên cùng làn; các xe xa hơn `> 50 m` ngoài phạm vi) | `Car`, `Truck`, `Cyclist`, `Pedestrian` | Bên trái có xe con đen sát thân xe (`~6,0 m`) và xe con trắng (`~9,4 m`); phía sau có xe con trắng (`~7,0 m`) và xe con bạc (`~22,3 m`); bên phải trên làn xe đạp có người đi xe đạp xanh (`~9,7 m`) và người đi bộ (`~13,9 m`) | Cụm điểm chồng sát nhau quanh hàng xe đạp công cộng dựng bên vỉa hè phải (`~14–16 m`) khó tách riêng người đứng với xe dựng nếu chỉ nhìn point cloud |

**3 điều chỉ camera cho biết (từ `s01f05` và `s01f52`):**
1. Màu sắc thân xe, trạng thái đèn tín hiệu/đèn phanh và vạch kẻ làn đường trên mặt nhựa.
2. Phân biệt rõ phân lớp tinh như xe van chở hàng (`Truck`) với xe chở người (`Car`), hoặc xe hai bánh đang có người lái (`Cyclist`) với xe đạp dựng trống không có người bên vỉa hè phải.
3. Nhận biết nhanh đầu xe hay đuôi xe (đèn pha so với đèn hậu) ở các xe đứng yên hoặc xe ở xa trên `30 m`.

**3 điều chỉ LiDAR cho biết (từ `s01f05` và `s01f52`):**
1. Khoảng cách hình học 3D chính xác theo đơn vị mét tới từng vật (phân định ranh giới `30 m` và `50 m`).
2. Kích thước vật lý thực (`dài`, `rộng`, `cao` theo mét) và độ cao đáy hộp so với mặt đường (`z_bottom`), kể cả độ chênh cao của làn xe đạp bên phải ở `s01f52`.
3. Góc quan sát liền mạch `360°` quanh xe, phát hiện được các vật hoặc vùng bị che nằm trong góc mù sát thân xe giữa các camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f09` | Cạnh trái (`−y`) | Ngã tư rộng đông xe và nhiều `Cyclist` cắt ngang; xe buýt lớn sát bên trái (`~10,7 m`) tạo vùng bóng che rất rộng phía sau nó |
| `s02f51` | Cạnh trái (`−y`) | Đường hai chiều có hàng rào giữa và hai hàng cây rất dày dọc vỉa hè, dễ nhầm thân cây thấp với người đi bộ nếu không đối chiếu camera |
| `s03f14` | Cạnh trái (`−y`) | Có cụm `10` điểm lạ nằm sâu dưới mặt đường (`z ≈ −4,3 m`, cách `9,3 m`); xe hatchback đen sát trái (`3,27 m`) ít điểm hơn hẳn xe van trắng gần cạnh (`5,29 m`) |
| `s03f32` | Cạnh trái (`−y`) | Ít xe nhất (`6` cuboid); có xe buýt đỗ sát nhà chờ phía sau-phải (`~38,7 m`) bị cột nhà chờ che một phần |
| `s04f32` | Cạnh trái (`−y`) | Đoạn đường thẳng có dải phân cách vàng giữa đường và tường quây công trình bên phải; có `Cyclist` chạy song song rất sát bên phải (`5,97 m`) |
| `s04f47` | Cạnh trái (`−y`) | Hai xe con sát bên phải (`~7,6 m` và `~7,9 m`) che một phần người đi xe máy phía trước-phải (`16,04 m`); xe xa `~48 m` rất thưa điểm |
| `s05f25` | Cạnh trái (`−y`) | Ngã tư rất rộng, mật độ vật cao nhất (`24` cuboid gồm nhiều hướng chuyển động `0°`, `90°`, `180°`, `270°`); có vài điểm nhiễu sâu dưới mặt đường (`z ≈ −14,2 m`) |
| `s05f41` | Cạnh trái (`−y`) | Xe buýt lớn sát bên trái (`5,85 m`) tạo vùng bóng che khổng lồ và có cụm điểm mờ nghi phản xạ đa đường qua kính cửa xe buýt; có `8` điểm lạ dưới mặt đường (`z ≈ −7,6 m`) |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh chụp từ CVAT: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`. Do frame 1 (`s01f05`) không có người đi bộ hay xe hai bánh rõ nét trong `30 m` (chỉ có 1 người đi bộ rất xa `33,48 m` bị tán cây che nhẹ), mẫu `(c)` được chọn là `Cyclist` rõ nét ở frame 2 (`s01f52`) theo đúng hướng dẫn.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f05`, `idx 1` (`t2-a-car.png`) | `Car` | Đúng (`270,0°`, chỉ thẳng đầu xe hướng `−y`) | Chạm đường (`z_bottom = −1,778 m`, `ground_gap_m = 0,00 m`) | Đã kiểm đủ Top/Side/Front + `cam05` + `preview` | Đúng (`4,650 × 1,880 × 1,503 m` nằm gọn trong khoảng sedan `3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f05`, `idx 6` (`t2-b-bus-truck.png`) | `Truck` | Đúng (`90,0°`, chỉ đầu ca-bin hướng `+y` làn đối diện) | Chạm đường (`z_bottom = −1,843 m`, `ground_gap_m = 0,00 m`) | Đã kiểm đủ Top/Side/Front + `cam09` + `preview` | Đúng (`5,774 × 2,153 × 2,985 m` nằm gọn trong khoảng xe tải hộp `5,5–9,0 × 2,00–2,55 × 2,40–3,50 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s01f52`, `idx 5` (`t2-c-ped-cyclist.png`) | `Cyclist` | Đúng (`270,0°`, chỉ hướng di chuyển `−y`) | Chạm mặt làn xe đạp (`z_bottom = −1,445 m`, `ground_gap_m = 0,00 m`) | Đã kiểm đủ Top/Side/Front + `cam07`/`cam08` + `preview` | Đúng (`1,750 × 0,650 × 1,700 m` nằm gọn trong khoảng xe đạp có người `1,6–1,9 × 0,50–0,70 × 1,60–1,90 m`) |

**Vòng hiệu chuẩn và vật bỏ qua không đủ cơ sở:**
- **Vòng hiệu chuẩn (`D13-HC485087`):** Đã tạo task chung `D13-HC485087` để kiểm tra quy ước hệ trục `−y` phía trước, cách neo mặt gần LiDAR kết hợp suy luận phần bị che theo bảng 4.3, và phân biệt rõ `Truck` (van chở hàng, bán tải) với `Car` (van chở người, SUV) cũng như bật `occluded` khi vật bị che một phần.
- **Các vật bỏ qua vì không đủ cơ sở / ngoài phạm vi:**
  - `s05f41`, bên trái xe buýt vàng (`x ≈ 5,9 m`, `y ≈ −1,9 m`): cụm điểm lọt qua kính cửa xe buýt không xác nhận chắc chắn được loại vật trên camera (`LOW_VISIBILITY` / nghi phản xạ đa đường qua kính), bỏ qua theo nguyên tắc không đoán.
  - `s01f05` và `s01f52`, vỉa hè bên phải (`x ≈ −11 m` đến `−24 m`): nhiều xe đạp dựng không có người lái → bỏ qua đúng quy tắc guideline (`Cyclist` chỉ vẽ khi có người lái).
  - `s01f52`, phía trước ngã tư (`dist_m ≈ 50,7 m`) và `s03f32` phía sau (`dist_m > 55 m`): nằm ngoài phạm vi chấm `≤ 50 m` (`UNCERTAIN_SCOPE`), không đưa vào bài.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s05f41` / `t4-1-bong-che.png` / task cá nhân `D13-2A202603073` | **Giả thuyết:** thân xe buýt lớn sát bên trái (`5,85 m`, dài `11,90 m`, cao `3,05 m`) chắn toàn bộ chùm tia LiDAR chiếu sang phía `+x` | Phía sau thân xe buýt xuất hiện một hình quạt trống đen lớn không có điểm mặt đường hay điểm vật, tương phản rõ với các vòng tia dày ở hướng không bị chắn | Vùng trống là chưa quan sát được chứ không phải đường trống; nếu có vật lộ một phần trong/mép vùng bóng che phải bật `occluded` và vẽ hộp bao cả phần suy luận được |
| 2. Thưa ở xa | `s04f47` / `t4-2-thua-o-xa.png` / task cá nhân `D13-2A202603073` | **Giả thuyết:** khoảng cách xa (`48,25 m`) làm các tia LiDAR 40 kênh phân kỳ rộng cả ngang và đứng, khiến chỉ còn 2–3 đường quét chạm vào đuôi xe | Xe con trắng `idx 10` ở `48,25 m` chỉ còn `35` điểm thưa thớt tập trung ở phần đuôi/kính sau, không thấy rõ thân trước và bánh xe | Áp quy tắc 15–20%: vì ảnh `cam03` xác nhận rõ loại xe và hướng nên vẫn vẽ cuboid bao kích thước suy luận (`4,55 × 1,82 × 1,48 m`) kèm bật `occluded`; nếu thưa hơn nữa không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s03f14` / `t4-3-phan-xa-thap.png` / task cá nhân `D13-2A202603073` | **Giả thuyết:** lớp sơn đen bóng và bề mặt kính nghiêng của xe hatchback đen (`idx 1`) hấp thụ hoặc phản xạ gương tia laser mạnh hơn so với thân sơn trắng phẳng của xe van chở hàng (`idx 2`) | Xe hatchback đen ở gần hơn (`3,27 m`) nhưng chỉ có `1844` điểm tối màu (thiếu nhiều điểm phần cửa dưới/kính), trong khi xe van trắng ở xa hơn (`5,29 m`) lại sáng rõ với tới `5926` điểm (gấp `3,2` lần) | Không khẳng định chắc chắn nguyên nhân vật lý mà chỉ ghi giả thuyết; đối chiếu camera và tham chiếu kích thước bảng 4.3 để không vẽ thiếu phần thân xe tối màu |
| 4. Điểm lạ | `s02f35` / `t4-4-diem-la.png` / nguồn: **task chung `D13-HC485087`** *(cũng quan sát hiện tượng tương tự ở `s03f14` tại `z ≈ −4,3 m` và `s05f41` tại `z ≈ −7,6 m`)* | **Giả thuyết:** phản xạ đa đường (multipath) từ mặt kính/kim loại hoặc nhiễu trả về giả của cảm biến làm sai thời gian bay của tia laser | Khi xoay ngang khung 3D, thấy cụm khoảng `46` điểm cô lập lơ lửng ở `z ≈ −13,5 m`, thấp hơn hẳn mặt đường phẳng khoảng `11 m` (không tương ứng bất kỳ vật thật nào) | Không bao giờ gán các cụm điểm lạ dưới mặt đường hay giữa không trung thành cuboid vật thể; ghi nhận rõ là điểm nhiễu giả thuyết |

## 5. Số từ cuboid của tôi (Bước 5)

Toàn bộ số liệu bên dưới trích trực tiếp từ `submission/D13-2A202603073-bangA.csv` và `submission/D13-2A202603073-tukiem.csv`.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f05`, `idx 1`, `Car` | `−1,027` | `1,503` | `−1,778` | `5,53` | `5,62` | `270,0°` | Chạm đường (`ground_gap_m = 0,00 m`, lệch mốc `−1,7 m` chỉ `0,08 m`) |
| `s01f05`, `idx 6`, `Truck` | `−0,351` | `2,985` | `−1,843` | `21,64` | `21,64` | `90,0°` | Chạm đường (`ground_gap_m = 0,00 m`, làn đối diện hơi thấp theo dốc ngang ~2%) |
| `s05f41`, `idx 2`, `Bus` | `−0,236` | `3,050` | `−1,761` | `5,85` | `5,85` | `270,0°` | Chạm đường (`ground_gap_m = 0,00 m`, lệch mốc `−1,7 m` chỉ `0,06 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Gần nhất:** `s03f14`, `idx 1` (`Car`, xe hatchback đen bên trái) có `dist_m = 3,27 m` và khoảng cách 3D `r = 3,42 m` (`x = 2,857 m`, `y = 1,598 m`, `z = −0,994 m`).
  - **Xa nhất:** `s01f05`, `idx 15` (`Car`, xe con bạc đi ngược chiều phía trước-trái) có `dist_m = 49,68 m` và khoảng cách 3D `r = 49,71 m` (`x = 18,718 m`, `y = −46,019 m`, `z = −1,575 m`).
- **(b) So kích thước với bảng 4.3:**
  - **Xe hai bánh có người (`Cyclist`):** `s01f52`, `idx 5` có `l = 1,750 m`, `w = 0,650 m`, `h = 1,700 m` — cả 3 chiều đều nằm gọn trong khoảng tham chiếu bảng 4.3 cho xe đạp có người (`l: 1,6–1,9 m`, `w: 0,50–0,70 m`, `h: 1,60–1,90 m`), không có chiều nào ngoài khoảng.
  - **Xe con (`Car`):** `s01f05`, `idx 1` (sedan) có `l = 4,650 m`, `w = 1,880 m`, `h = 1,503 m` — cả 3 chiều đều nằm trong khoảng tham chiếu sedan (`l: 3,8–5,0 m`, `w: 1,70–1,90 m`, `h: 1,40–1,60 m`). Một số trường hợp đặc biệt trong bộ 133 hộp có chiều hơi lệch nhẹ ngoài khoảng chuẩn (ví dụ `s03f14, idx 1` hatchback nhỏ có `w = 1,545 m` hơi hẹp hơn `1,70 m` do chỉ quan sát một mặt hông rất gần ở `3,27 m`; `s01f05, idx 9` người đi bộ có `h = 2,006 m` nhỉnh hơn `1,95 m` khoảng `0,056 m` do vài điểm tán cây sát phía trên ở `33,48 m`).
- **(c) Góc `yaw_deg` theo chiều chuyển động:**
  - Xe cùng chiều phía trước: `s01f05`, `idx 2` (`Car`) có `yaw_deg = 270,0°` (khớp sát `270°`, đầu xe hướng `−y`).
  - Xe đi ngược chiều: `s01f05`, `idx 3` (`Car`, làn đối diện) có `yaw_deg = 90,0°` (khớp sát `90°`, đầu xe hướng `+y`).

**Mật độ điểm theo khoảng cách (4 cuboid `Car` không bị che khuất):**

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| `s01f05`, `idx 1`, `Car` | `5,53` | `2703` |
| `s01f05`, `idx 2`, `Car` | `14,69` | `674` |
| `s02f09`, `idx 9`, `Car` | `22,15` | `272` |
| `s04f47`, `idx 8`, `Car` | `35,88` | `90` |

- **Quy luật mật độ:** Khi khoảng cách tăng gấp `~2,5` lần (từ `5,53 m` lên `14,69 m` hay từ `14,69 m` lên `35,88 m`), số điểm LiDAR trên cùng loại `Car` giảm xuống còn khoảng `1/4` đến `1/7` (gần tỷ lệ nghịch với bình phương khoảng cách `1/r²` do chùm tia phân kỳ), bám rất sát đường cong trung vị của slide bài giảng (`~1968 điểm` ở `0–10 m`, `~718` ở `10–20 m`, `~214` ở `20–30 m`, `~72` ở `30–40 m`, `~19` ở `60–80 m`).
- **Tự kiểm và chỉnh sửa trên CVAT:** Sau khi kiểm tra ảnh `work/xem/` và bảng A/phiếu tự kiểm, đã sửa trực tiếp **3 cuboid** trên CVAT, bấm Save và chạy lại `sync`:
  1. `s01f05`, `idx 5` (`Truck`, `20,18 m`): nâng tâm `z` từ `−1,182 m` lên `−0,840 m` (`h = 2,020 m`) để đưa `z_bottom` từ `−2,258 m` (hơi chìm) về `−1,850 m` chạm mặt đường làn đối diện.
  2. `s01f52`, `idx 17` (`Truck`, `34,65 m`): hạ tâm `z` từ `−0,303 m` xuống `−0,605 m` (`h = 2,350 m`) để đưa `z_bottom` từ `−1,378 m` (hơi lơ lửng) xuống `−1,780 m` chạm mặt đường.
  3. `s04f47`, `idx 10` (`Car`, `48,25 m`): hạ tâm `z` từ `−0,713 m` xuống `−1,020 m` (`h = 1,480 m`) để đưa `z_bottom` từ `−1,453 m` xuống `−1,760 m` chạm mặt đường.
  - Sau khi cập nhật, `selfcheck` trả về **0/133** cảnh báo. Với các xe ở nhánh đường ngang có độ dốc ngang tự nhiên ~2% (như `s01f05, idx 12/13/15` và `s05f41, idx 8`), giữ nguyên độ cao bám theo mặt đường cục bộ (`ground_gap_m = 0,00 m`) thay vì ép phẳng cứng về `−1,70 m`.
- **Đổi kiểm với bạn ngồi cạnh:** Đã chuẩn bị sẵn 2 con số để đối chiếu trực tiếp trên màn hình với bạn ngồi cạnh (`s01f05, idx 1: dist_m = 5,53 m, z_bottom = −1,778 m` và `s05f41, idx 2: length = 11,900 m, points_in = 11235` — đã tự khớp trên bảng A và phiếu tự kiểm; cập nhật MSSV bạn ngồi cạnh và kết quả sau khi đổi kiểm trực tiếp).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Góc nhìn từ mô hình học trên bộ nhãn này:**
  - Nhãn 3D phải đo bằng **mét** trong hệ tọa độ thực thay vì pixel vì mô hình nhận diện 3D và lập kế hoạch quỹ đạo cần biết chính xác kích thước vật lý, khoảng cách tới vật cản và góc hướng không bị biến dạng theo tiêu cự hay phối cảnh ảnh.
  - Vật thưa điểm cần **quy tắc riêng (15–20% thể tích hoặc bỏ qua khi thiếu cơ sở)** vì nếu người gán đoán bừa tọa độ, kích thước và góc `yaw` trên cụm chỉ có vài điểm rời rạc thì nhãn sai sẽ tạo tín hiệu hồi quy nhiễu trong quá trình huấn luyện, làm mô hình giảm độ chính xác định vị và sinh nhiều dự đoán sai lệch ở tầm xa.
- **(b) Khai báo dùng AI:**
  - Công cụ: Antigravity (trợ lý AI cục bộ trong VS Code).
  - Câu hỏi chính: hướng dẫn cấu hình môi trường, tạo task trên CVAT local, kiểm tra quy ước hệ tọa độ VF (`x` trái, `y` sau, `z` lên), hỗ trợ xử lý công cụ xuất bảng A.
  - Cách dùng kết quả: sử dụng để chuẩn hóa quy trình thao tác CVAT.
- **(c) Điều còn chưa chắc:**
  - Một số xe rất xa trong vùng `40–50 m` bị che khuất một phần (như `s01f05, idx 12` xe buýt chạy ngang, `s01f05, idx 15` xe con ngược chiều) chỉ còn ít điểm nên ước lượng chiều dài toàn phần vẫn có thể sai số khoảng `0,3–0,8 m`.
  - Ranh giới phân loại giữa van chở hàng cỡ nhỏ (`Truck`) và minivan chở khách (`Car`) trên một số xe kích thước trung bình như `s02f09, idx 8` (xe van điện xanh nhỏ) còn mang tính tương đối khi quan sát từ xa.
  - Cụm điểm mờ phía sau cửa kính xe buýt vàng ở `s05f41` (`x ≈ 5,9 m`, `y ≈ −1,9 m`) chưa thể phân biệt dứt khoát là xe ở làn khuất hay nhiễu phản xạ đa đường qua kính nên đã chủ động bỏ qua.
