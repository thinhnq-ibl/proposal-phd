# proposal-phd
human mobility - OD flow generation 

**Đề tài:**  
Phát triển khung ước lượng ma trận OD đa phương thức tích hợp dữ liệu di động, flow và speed quan trắc: Nghiên cứu tại khu vực lõi TP. Hồ Chí Minh

## 1. GIỚI THIỆU

### 1.1. Tính cấp thiết & Khoảng trống nghiên cứu

- TP. Hồ Chí Minh là đô thị điển hình “motorbike-dominant” nhưng đang chuyển dịch (xe máy ~38% trên tuyến chính theo dữ liệu VDS mới nhất), giao thông hỗn hợp mạnh (xe máy, ô tô, xe tải), biến động cao theo thời gian.
- Phương pháp truyền thống (khảo sát hộ gia đình) tốn kém, ít cập nhật.
- Big data (Facebook Movement) và dữ liệu VDS (flow + speed) cung cấp thông tin luồng di chuyển, tốc độ, mở ra cơ hội ước lượng ma trận OD chính xác, cập nhật, phục vụ quy hoạch và điều hành giao thông thông minh.
- **Khoảng trống nghiên cứu**: Hầu hết các công trình chỉ dùng mobile data làm prior (gravity model) hoặc chỉ dùng flow; rất ít tích hợp speed constraint và regularization cho mạng hỗn hợp. Chưa có nghiên cứu nào áp dụng SPSA bilevel với multi-class static assignment trên dữ liệu thực tế Việt Nam, đặc biệt với tỷ lệ xe máy giảm dần trên tuyến chính.

### 1.2. Mục tiêu nghiên cứu

1. Xây dựng ma trận OD cho 2–3 nhóm phương tiện (motorbike ~38%, car/light vehicle ~29%, truck/heavy ~32%) tại 5 quận lõi (Quận 1, 3, 10, Tân Bình, Bình Thạnh), tích hợp đồng thời Facebook distance distribution, POI và VDS (flow + speed).
2. Phát triển khung hiệu chỉnh OD dựa trên:
   - Static Multi-class User Equilibrium
   - Bilevel optimization với thuật toán SPSA
   - Calibration BPR function sử dụng cả flow và speed
3. Đánh giá mức độ cải thiện accuracy & stability nhờ speed data và regularization, chứng minh tính ưu việt so với các phương pháp chỉ dùng flow hoặc prior đơn giản.

## 2. PHẠM VI VÀ NGUỒN DỮ LIỆU

### 2.1. Phạm vi không gian

- Khu vực nghiên cứu: 5 quận lõi TP. Hồ Chí Minh (Quận 1, 3, 10, Tân Bình, Bình Thạnh).
- Phân vùng: theo phường (32 vùng), ~1024 OD pairs.
- Tạm thời không xem xét các OD dài > 15km vì phạm vi các quận di chuyển không vượt quá quảng đường này.

### 2.2. Nguồn dữ liệu

| Loại dữ liệu          | Nguồn                  | Đặc điểm                                                                 |
|-----------------------|------------------------|--------------------------------------------------------------------------|
| Mạng lưới giao thông  | OSM                   | Đường, phân loại, POIs (8 nhóm: văn phòng, trường học, giải trí…)        |
| Dữ liệu di động nền   | Facebook Movement     | Phân phối khoảng cách di chuyển (0–1 km, 1–10 km, >10 km), dùng làm prior cho OD |
| Dữ liệu quan trắc     | Hệ thống VDS (76 trạm)| - Flow tổng hợp (PCU/h) + tốc độ trung bình (km/h)<br>- Phân loại theo Loại 1 (xe máy ~38,14%), Loại 2 (xe con/ô tô nhỏ ~29,43%), Loại 3–5 (xe tải/xe lớn ~32,43%)<br>- Dữ liệu tổng 24h ngày thường, tập trung tuyến đường chính<br>- PCU cao trên huyết mạch, speed biến động 29–47 km/h |

### 2.3. Lát cắt thời gian

- Chỉ xét ngày thường, tổng 24h, giảm complexity.
- Future work: phân tích giờ cao điểm và ngày cuối tuần.

### 2.4. Phân bố các loại phương tiện (dựa trên dữ liệu VDS ngày 02/04/2025)

Dựa trên dữ liệu quan trắc từ các trục đường chính trong khu vực lõi (Điện Biên Phủ, Ba Tháng Hai, Lý Thái Tổ, Cộng Hòa...):

- **Loại 1 (xe máy/mô tô)**: ~38,14% tổng lưu lượng xe.
- **Loại 2 (xe con/ô tô nhỏ)**: ~29,43% tổng lưu lượng xe.
- **Tổng phương tiện nhẹ (Loại 1 + 2)**: ~67,57%.
- **Loại 3–5 (xe tải nhỏ/trung bình, xe khách, container)**: ~32,43%.

**Nhận xét**:
- Xe máy không còn chiếm ưu thế tuyệt đối như các thống kê cũ (thường >70–80%), mà chỉ khoảng 38% trên tuyến chính được VDS quan trắc → phản ánh sự gia tăng xe con/ô tô cá nhân ở khu vực lõi.
- Tỷ trọng này hỗ trợ định nghĩa **2–3 lớp phương tiện** cho multi-class assignment: motorbike (~38%), car/light (~29%), truck/heavy (~32%).
- Dữ liệu VDS có thể under-represent xe máy trên đường nhỏ/làn phụ → cần kết hợp prior từ Facebook Movement + POI để cân bằng ma trận OD.

## 3. CÂU HỎI NGHIÊN CỨU

- RQ1: Làm thế nào tích hợp **đồng thời** POI + Facebook distance distribution + speed constraint để xây dựng OD prior và hiệu chỉnh ma trận OD đa phương thức ở đô thị motorbike-dominant nhưng đang chuyển dịch?
- RQ2: Việc áp dụng **regularized SPSA bilevel optimization** với static multi-class assignment và speed data có cải thiện đáng kể độ chính xác, stability và giải quyết được vấn đề ill-posedness so với các phương pháp truyền thống chỉ dùng flow?

## 4. PHƯƠNG PHÁP NGHIÊN CỨU

### 4.1. Xây dựng ma trận OD khởi tạo

- Gravity model: attraction = weighted POI (hoặc tổng số POI)
- Facebook Movement: dùng để calibrate distance distribution cho OD pair
- Kết quả: OD_prior cho 2–3 nhóm phương tiện (dựa trên tỷ trọng thực tế từ VDS)

### 4.2. Calibration của BPR function

- Sử dụng flow + speed từ VDS:
  - Travel time: ( t = L / speed )
  - Free-flow speed: 85th percentile hoặc OSM speed limit
- Fit BPR parameters α, β cho từng loại đường
- Outcome: link cost function calibrated, tăng độ tin cậy cho traffic assignment

### 4.3. Phân bổ giao thông & OD estimation

- Static Multi-class Assignment (UE)
  - 2–3 nhóm phương tiện (motorbike, car/taxi, truck) dựa trên tỷ trọng VDS
  - Flow & speed riêng cho từng link
- Bilevel Optimization với SPSA
  - Upper level: điều chỉnh OD → giảm flow + speed error
  - Lower level: chạy UE assignment
- Hàm mục tiêu (regularized):
$$J(\mathbf{OD}) = w_1 \sum_{l \in \mathcal{L}} \left( V_{l}^{\text{sim}} - V_{l}^{\text{obs}} \right)^2 + w_2 \sum_{l \in \mathcal{L}} \left( S_{l}^{\text{sim}} - S_{l}^{\text{obs}} \right)^2 + \lambda \| \mathbf{OD} - \mathbf{OD}_{\text{prior}} \|_2^2$$
- Điều kiện dừng: RMSE hội tụ hoặc đạt max iterations

### 4.4. Công cụ dự kiến

- Python: core language
- AequilibraE: UE assignment
- NumPy / SciPy: SPSA, dữ liệu
- QGIS: GIS visualization

## 5. TÍNH MỚI VÀ ĐÓNG GÓP

### 5.1. Tính mới

| Yếu tố                  | Nghiên cứu trước đây                          | Nghiên cứu này (novelty)                              |
|-------------------------|-----------------------------------------------|-------------------------------------------------------|
| Nguồn dữ liệu           | Chỉ Facebook/Gravity hoặc chỉ VDS flow        | Tích hợp đồng thời Facebook distance + POI + flow + speed |
| Phân loại phương tiện   | Giả định xe máy chiếm >70–80%                 | Dựa trên dữ liệu thực tế VDS mới: motorbike ~38%, car ~29%, heavy ~32% |
| Phương pháp tối ưu      | Bilevel gradient-based hoặc chỉ single-class  | SPSA bilevel + multi-class static assignment           |
| Regularization & speed  | Thường không có hoặc chỉ đơn giản             | Regularized objective với speed constraint → giải quyết ill-posedness |

→ Đây là khung OD estimation **đầu tiên** áp dụng cho đô thị đang phát triển với tỷ lệ xe máy giảm dần trên tuyến chính, sử dụng dữ liệu thực tế mới nhất.

### 5.2. Đóng góp

- Học thuật: Khung regularized multi-class OD estimation mới, có thể áp dụng cho các thành phố Đông Nam Á.
- Thực tiễn: OD matrix cho 5 quận lõi → input cho quy hoạch, mô phỏng ùn tắc, kịch bản hạ tầng TP.HCM.

## 6. GIẢ THUYẾT NGHIÊN CỨU

- H1: Tích hợp Facebook distance + speed constraint + regularization cải thiện accuracy & stability vượt trội so với gravity-only hoặc flow-only.
- H2: Multi-class SPSA bilevel cho phép recover OD đáng tin cậy từ dữ liệu sparse trong mạng hỗn hợp.

## 7. KẾ HOẠCH THỰC HIỆN

| Giai đoạn | Thời gian       | Nội dung chính                                                                 |
|-----------|-----------------|--------------------------------------------------------------------------------|
| 1         | Tháng 1–6       | GIS, OSM, zoning 32 vùng, chuẩn hóa VDS flow + speed, thu thập Facebook Movement |
| 2         | Tháng 7–15      | Gravity OD prior + POI weighting, calibration BPR từ flow & speed, static multi-class assignment |
| 3         | Tháng 16–27     | SPSA bilevel optimization, chạy thực nghiệm, đánh giá RMSE, viết bài báo 1 (Q2) |
| 4         | Tháng 28–36     | Kiểm chứng với dữ liệu khác (nếu có), viết bài báo quốc tế 2 (Q1/Q2), hoàn thiện luận án, bảo vệ |

## 8. HẠN CHẾ & HƯỚNG MỞ RỘNG

**Hạn chế:**
- Chỉ xét ngày thường, tổng 24h
- Speed aggregated, không phân loại theo phương tiện chi tiết ở một số đoạn
- Sensor coverage chưa 100%, xe máy có thể under-represent trên đường phụ

**Hướng mở rộng:**
- Phân tích giờ cao điểm & cuối tuần
- Dynamic assignment hoặc time-of-day OD
- So sánh với dữ liệu di động độ phân giải cao hơn

## 6. Giới hạn nghiên cứu khi chuyển đoạn thành điểm

- **Ưu điểm:**
  - Giảm độ phức tạp của bài toán (ít biến, ít tham số cần calibrate).
  - Phù hợp với bản chất dữ liệu VDS vốn là điểm đo.
  - Vẫn giữ được thông tin quan trọng (flow + speed trung bình).

- **Hạn chế:**
  - Không mô phỏng chi tiết travel time trên toàn đoạn, chỉ phản ánh trạng thái tại điểm.
  - Tốc độ trung bình tại điểm có thể bị nhiễu bởi vị trí sensor (gần nút giao, đèn tín hiệu).
  - Khả năng mở rộng sang phân tích động (theo giờ cao điểm) có thể bị hạn chế nếu chỉ dựa vào điểm.

- **Chiến lược khắc phục:**
  - Ghi rõ trong luận án rằng đây là lựa chọn chiến lược để giảm độ phức tạp.
  - Đề xuất hướng mở rộng: kết hợp dữ liệu đoạn dài hoặc dữ liệu di động độ phân giải cao hơn để bổ sung travel time chi tiết.

