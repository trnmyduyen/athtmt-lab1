# Mô hình đe dọa: Cổng thông tin tra cứu điểm thi trực tuyến của sinh viên

## 1. Mô tả hệ thống và ba thành phần

Hệ thống được lựa chọn là **Cổng thông tin tra cứu điểm thi trực tuyến của sinh viên**, phục vụ nhu cầu xem điểm tổng kết học kỳ và gửi đơn phúc khảo. Hệ thống gồm đúng ba thành phần cốt lõi:
1. **Trình duyệt sinh viên (Frontend):** Ứng dụng web Single Page Application chạy trên trình duyệt của sinh viên, chịu trách nhiệm thu thập thông tin đăng nhập, hiển thị bảng điểm và cho phép gửi đơn phúc khảo.
2. **Máy chủ ứng dụng Web API (Backend):** Máy chủ dịch vụ xử lý logic nghiệp vụ, xác thực danh tính sinh viên qua JWT, kiểm tra phân quyền và định tuyến truy vấn tới cơ sở dữ liệu.
3. **Cơ sở dữ liệu PostgreSQL (Database):** Lưu trữ tập trung thông tin tài khoản người dùng, hàm băm mật khẩu, thông tin cá nhân và bảng điểm học tập của toàn bộ sinh viên.

---

## 2. Ba mối đe dọa được ưu tiên xử lý và lập luận lựa chọn

Dựa trên ma trận rủi ro tính theo tích `Tác động × Khả năng` (thang điểm 1–5), ba mối đe dọa có thứ hạng cao nhất được lựa chọn để xử lý ngay gồm:

| Mã | Tên mối đe dọa | Tác động | Khả năng | Tích rủi ro |
| :---: | :--- | :---: | :---: | :---: |
| **M01** | Trích xuất toàn bộ bảng điểm qua SQL Injection | 5 | 4 | **20** |
| **M02** | Xem trộm điểm sinh viên khác qua lỗi phân quyền (IDOR) | 4 | 5 | **20** |
| **M03** | Dò quét vét cạn (Brute-force) tài khoản quản trị viên | 4 | 4 | **16** |

### Lý do chọn ba mối này:
* **M01 (SQL Injection):** Điểm cuối tra cứu điểm là giao diện công khai tiếp nhận dữ liệu từ người dùng. Nếu bị tấn công SQLi, kẻ xấu có thể đọc hoặc xóa sạch toàn bộ cơ sở dữ liệu điểm thi, dẫn đến việc sụp đổ hoàn toàn tính bí mật và toàn vẹn của hệ thống.
* **M02 (IDOR - Lỗi phân quyền mức đối tượng):** Bất kỳ sinh viên nào sau khi đăng nhập hợp lệ đều có thể dễ dàng thay đổi tham số trên URL để xem trộm điểm của bạn bè. Khả năng xảy ra đạt mức tối đa (5/5) do thao tác tấn công rất đơn giản nhưng gây ảnh hưởng nghiêm trọng đến quyền riêng tư của hàng ngàn sinh viên.
* **M03 (Brute-force đăng nhập):** Giao diện đăng nhập đối mặt trực tiếp với Internet. Nếu thiếu cơ chế giới hạn tần suất (Rate Limiting), kẻ tấn công tự động hóa có thể nhanh chóng chiếm quyền kiểm soát tài khoản quản trị viên và can thiệp sửa đổi kết quả thi.

### Thừa nhận phần bị bỏ lại (Trade-offs):
Các mối đe dọa còn lại (**M04 đến M08**) có mức độ rủi ro thấp hơn (tích từ 5 đến 12) nên tạm thời được chấp nhận hoãn xử lý trong đợt đầu nhằm tối ưu hóa nguồn lực phát triển:
* **M04 (Stored XSS)** và **M05 (Nghe lén HTTP):** M04 chỉ ảnh hưởng cục bộ tới tài khoản khảo thí và có thể bù đắp một phần nhờ cơ chế CSP; M05 được giảm thiểu đáng kể vì hệ thống triển khai trên mạng trường học có tường lửa nội bộ và sẽ được nâng cấp HTTPS ở giai đoạn sau.
* **M06, M07, M08:** M06 và M08 yêu cầu kẻ tấn công phải có quyền truy cập sâu vào hạ tầng nội bộ (khả năng xảy ra rất thấp: 1–2/5); M07 (DoS tài nguyên) chỉ ảnh hưởng tính sẵn sàng trong thời gian ngắn và có thể phục hồi bằng cách khởi động lại dịch vụ.

---

## 3. Ước lượng chi phí xử lý theo Nguyên lý thứ bảy

Theo Nguyên lý thứ bảy (*Chi phí bảo vệ phải tương xứng với giá trị tài sản cần bảo vệ*), ngân sách bỏ ra để khắc phục ba mối đe dọa được tính toán như sau:

1. **Mối đe dọa M01 (Chống SQL Injection):**
   * **Chi phí ước lượng:** **16 giờ công kỹ sư** (tương đương khoảng 4.000.000 VNĐ).
   * **Căn cứ:** 8 giờ kỹ sư rà soát mã nguồn chuyển toàn bộ câu lệnh thô sang Prepared Statements / ORM, và 8 giờ viết kiểm thử tự động.
   * **So với tài sản:** Toàn bộ bảng điểm của nhà trường nếu bị rò rỉ sẽ gây thiệt hại uy tín và chi phí khắc phục ước tính vượt quá 200.000.000 VNĐ. Chi phí bỏ ra chỉ chiếm ~2% giá trị rủi ro tiềm ẩn.

2. **Mối đe dọa M02 (Chống lỗi phân quyền IDOR):**
   * **Chi phí ước lượng:** **20 giờ công kỹ sư** (tương đương khoảng 5.000.000 VNĐ).
   * **Căn cứ:** 12 giờ lập trình bổ sung middleware kiểm tra `student_id` từ token JWT khớp với bản ghi được truy vấn, và 8 giờ kiểm thử luồng phân quyền.
   * **So với tài sản:** Bảo vệ bí mật thông tin học tập cho hơn 10.000 sinh viên, ngăn ngừa các khiếu nại pháp lý và khủng hoảng truyền thông có thể tốn kém hàng trăm triệu đồng.

3. **Mối đe dọa M03 (Chống Brute-force đăng nhập):**
   * **Chi phí ước lượng:** **8 giờ công kỹ sư** (tương đương khoảng 2.000.000 VNĐ).
   * **Căn cứ:** 4 giờ cấu hình giới hạn tần suất (Rate Limiting) tối đa 5 lần thử/phút tại API Gateway/Nginx, và 4 giờ kiểm thử tải.
   * **So với tài sản:** Chi phí rất nhỏ nhưng loại bỏ hoàn toàn nguy cơ chiếm đoạt quyền quản trị cơ sở dữ liệu điểm thi.
