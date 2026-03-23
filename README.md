# proposal-phd
human mobility - OD flow generation 

**Đề tài:** Phát triển khung ước lượng ma trận OD đa phương thức tích hợp dữ liệu di động, lưu lượng và tốc độ quan trắc hai chiều: Nghiên cứu tại khu vực lõi TP. Hồ Chí Minh

## 1. GIỚI THIỆU

### 1.1. Tính cấp thiết & Khoảng trống nghiên cứu
TP. Hồ Chí Minh đang trải qua sự chuyển dịch cơ cấu phương tiện mạnh mẽ. Dữ liệu VDS mới nhất cho thấy xe máy chỉ còn chiếm khoảng **38%** lưu lượng trên các tuyến chính khu vực lõi, phản ánh sự gia tăng nhanh chóng của ô tô cá nhân. Tuy nhiên, các phương pháp ước lượng ma trận OD truyền thống vẫn dựa nặng nề vào khảo sát hộ gia đình tốn kém hoặc các giả định lỗi thời về tỉ lệ xe máy (>80%). 

**Khoảng trống nghiên cứu:** * Hầu hết các công trình hiện nay chỉ sử dụng dữ liệu di động làm thông tin tiên nghiệm (prior) đơn lẻ hoặc chỉ dựa vào lưu lượng (flow). 
* Rất ít nghiên cứu tích hợp đồng thời ràng buộc tốc độ (**speed constraint**) và cơ chế hiệu chỉnh (**regularization**) cho mạng lưới hỗn hợp với các cung đường ngắn đặc thù đô thị. 
* Chưa có khung nghiên cứu nào giải quyết triệt để bài toán khớp nối dữ liệu quan trắc hai chiều (**bi-directional flow**) vào mô hình phân bổ tĩnh đa lớp tại Việt Nam.

### 1.2. Mục tiêu nghiên cứu
1. Xây dựng ma trận OD khởi tạo cho 3 nhóm phương tiện (Motorbike ~38%, Car ~29%, Truck/Heavy ~32%) tại 5 quận lõi (Q.1, 3, 10, Tân Bình, Bình Thạnh) tích hợp từ Facebook Movement và POI.
2. Phát triển thuật toán hiệu chỉnh OD dựa trên tiệm cận **Entropy Maximization** kết hợp **Weighted Least Squares (WLS)** để xử lý dữ liệu từ 76 trạm VDS.
3. Đánh giá độ tin cậy của mô hình thông qua chỉ số GEH và sự bảo toàn cấu trúc phân bố chuyến đi (Trip Length Distribution).

---

## 2. PHẠM VI VÀ NGUỒN DỮ LIỆU

### 2.1. Phạm vi không gian & Phân vùng
* **Khu vực:** 5 quận lõi TP.HCM, phân chia thành **32 vùng (zones)** theo đơn vị hành chính phường.
* **Mạng lưới:** Trích xuất từ OpenStreetMap (OSM), tập trung vào các trục huyết mạch nơi đặt sensor.

### 2.2. Nguồn dữ liệu (Input)
* **Dữ liệu nền:** Facebook Movement (phân phối khoảng cách di chuyển) và dữ liệu POI (8 nhóm chức năng) để tạo ma trận Prior.
* **Dữ liệu quan trắc:** 76 trạm VDS cung cấp lưu lượng PCU tổng hợp 24h và vận tốc trung bình ($km/h$). Đặc thù dữ liệu ghi nhận trên mặt cắt ngang hai chiều.

---

## 3. PHƯƠNG PHÁP NGHIÊN CỨU

### 3.1. Xây dựng Ma trận OD Tiên nghiệm (Prior OD)
Sử dụng mô hình Gravity cải biên, trong đó hàm chi phí khoảng cách được hiệu chỉnh bởi dữ liệu Facebook Movement.
$$OD_{ij}^{prior} = \alpha \cdot P_i \cdot A_j \cdot f(d_{ij})$$
*Trong đó $P_i, A_j$ là khả năng phát sinh/hấp dẫn dựa trên trọng số POI.*

### 3.2. Thuật toán Ước lượng Entropy Maximization (EM)
Để phù hợp với các cung quan trắc ngắn và giảm độ phức tạp tính toán của các bài toán Bilevel, nghiên cứu áp dụng phương pháp tối ưu hóa lồi một cấp:

**Khớp nối dữ liệu hai chiều:** Xây dựng ma trận đóng góp gộp $P^{agg}$ từ kết quả phân bổ tĩnh (Static Assignment). Đối với trạm VDS $l$ án ngữ trên cung thuận ($a$) và nghịch ($b$):
$$P_{l,ij}^{agg} = P_{a,ij} + P_{b,ij}$$

**Hàm mục tiêu (Objective Function):**
$$\min J(OD) = \sum_{ij} OD_{ij} \left( \ln \frac{OD_{ij}}{OD_{ij}^{prior}} - 1 \right) + \frac{1}{2} \sum_{l \in \mathcal{L}} w_l \left( V_l^{sim} - V_l^{obs} \right)^2$$
*Với:*
* $V_l^{sim} = \sum_{ij} P_{l,ij}^{agg} \cdot OD_{ij}$
* $w_l$: Trọng số tin cậy tỉ lệ thuận với vận tốc quan trắc $S_l^{obs}$.

### 3.3. Công cụ thực hiện
* **Ngôn ngữ:** Python (Pandas, NumPy, SciPy).
* **Mô phỏng:** AequilibraE (xây dựng ma trận $P$ và chạy Assignment).
* **GIS:** QGIS và OSMnx để xử lý mạng lưới.

---

## 4. TÍNH MỚI VÀ ĐÓNG GÓP

| Yếu tố | Nghiên cứu truyền thống | Đề xuất của luận án |
| :--- | :--- | :--- |
| **Dữ liệu** | Đơn nguồn hoặc giả định tĩnh | Đa nguồn: FB + POI + VDS (Flow & Speed) |
| **Cấu trúc đường** | Thường giả định cung dài | Tối ưu cho cung ngắn, mật độ nút giao dày |
| **Hướng di chuyển** | Tách chiều khiên cưỡng | Gộp ma trận đóng góp (Aggregated Matrix) |
| **Giải thuật** | Bilevel/SPSA (dễ nhiễu) | Entropy Scaling (ổn định, hội tụ nhanh) |

---

## 5. KẾ HOẠCH THỰC HIỆN

1. **Giai đoạn 1 (Tháng 1-6):** Số hóa mạng lưới, Mapping 76 trạm VDS vào Link ID, chuẩn hóa dữ liệu PCU hai chiều.
2. **Giai đoạn 2 (Tháng 7-15):** Xây dựng ma trận Prior. Chạy Assignment khởi tạo để trích xuất ma trận xác suất $P$.
3. **Giai đoạn 3 (Tháng 16-27):** Thực hiện tối ưu hóa Entropy. Đánh giá sai số GEH và hiệu chỉnh trọng số $w_l$ theo Speed.
4. **Giai đoạn 4 (Tháng 28-36):** Viết bài báo quốc tế (Q1/Q2) và hoàn thiện luận án.

---

## 6. HẠN CHẾ & HƯỚNG MỞ RỘNG
* **Hạn chế:** Chưa xét đến biến động theo giờ (Time-of-day). Việc gộp hai chiều có thể làm mờ tính hướng tâm trong giờ cao điểm.
* **Mở rộng:** Phát triển mô hình cho các khung giờ cao điểm sáng/chiều khi có dữ liệu tách hướng chi tiết hơn.
