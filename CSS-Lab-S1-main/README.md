# Mô hình đe dọa: Cổng thông tin tra cứu điểm thi trực tuyến của sinh viên

## 1. Mô tả hệ thống và ba thành phần

Mô hình được xây dựng cho một **cổng thông tin tra cứu điểm thi trực tuyến**, nơi sinh viên có thể đăng nhập để xem kết quả học tập và thực hiện yêu cầu phúc khảo. Hệ thống được đơn giản hóa thành đúng ba thành phần chính:

1. **Trình duyệt của sinh viên (Frontend):** Giao diện web chạy trên trình duyệt, tiếp nhận thông tin đăng nhập, hiển thị kết quả học tập và hỗ trợ sinh viên gửi yêu cầu phúc khảo.

2. **Máy chủ ứng dụng Web API (Backend):** Đảm nhiệm việc xử lý các yêu cầu từ Frontend, xác thực người dùng bằng JWT, thực hiện kiểm tra quyền truy cập và gửi các truy vấn cần thiết đến cơ sở dữ liệu.

3. **Cơ sở dữ liệu PostgreSQL (Database):** Lưu giữ dữ liệu tài khoản, mật khẩu đã được băm, thông tin sinh viên và kết quả điểm thi.

Luồng dữ liệu cơ bản của hệ thống được mô hình hóa theo hướng:

**Trình duyệt sinh viên → Web API → PostgreSQL**

Trong đó, trình duyệt gửi yêu cầu đến máy chủ ứng dụng; máy chủ thực hiện xác thực, kiểm tra quyền và truy xuất dữ liệu tương ứng từ cơ sở dữ liệu.

---

## 2. Các mối đe dọa được ưu tiên xử lý

Mức độ ưu tiên của từng mối đe dọa được xác định dựa trên công thức:

**Mức rủi ro = Tác động × Khả năng xảy ra**

Với thang điểm từ 1 đến 5 cho mỗi yếu tố, ba mối đe dọa được đưa vào nhóm cần xử lý trước là:

| Mã | Mối đe dọa | Tác động | Khả năng | Điểm rủi ro |
|---|---|---:|---:|---:|
| **M01** | Khai thác SQL Injection để truy xuất dữ liệu điểm | 5 | 4 | **20** |
| **M02** | Truy cập điểm của sinh viên khác do lỗi IDOR | 4 | 5 | **20** |
| **M03** | Brute-force tài khoản quản trị viên | 4 | 4 | **16** |

### Cơ sở lựa chọn

**M01 – SQL Injection:**  
Các chức năng tra cứu của hệ thống phải tiếp nhận dữ liệu do người dùng cung cấp. Nếu dữ liệu đầu vào không được xử lý đúng cách, **khi tồn tại lỗ hổng SQL Injection**, kẻ tấn công có thể can thiệp vào câu truy vấn và truy cập trái phép dữ liệu trong cơ sở dữ liệu. Với hệ thống chứa bảng điểm của nhiều sinh viên, hậu quả có thể ảnh hưởng đồng thời đến tính bí mật và tính toàn vẹn của dữ liệu.

**M02 – IDOR:**  
Lỗi IDOR có thể xuất hiện khi Backend chỉ dựa vào mã định danh đối tượng được gửi từ phía người dùng mà không kiểm tra quyền sở hữu tương ứng. **Nếu cơ chế phân quyền không xác nhận sinh viên đang truy cập đúng dữ liệu của mình**, một tài khoản hợp lệ có thể lợi dụng việc thay đổi `student_id` để xem thông tin của người khác. Vì khả năng khai thác tương đối đơn giản và ảnh hưởng trực tiếp đến dữ liệu học tập, mối đe dọa này được đánh giá ở mức khả năng 5/5.

**M03 – Brute-force đăng nhập:**  
Trang đăng nhập là điểm tiếp xúc trực tiếp với người sử dụng. **Trong trường hợp API đăng nhập không có cơ chế giới hạn số lần thử**, kẻ tấn công có thể tự động gửi nhiều yêu cầu xác thực để tìm thông tin đăng nhập hợp lệ. Nếu tài khoản bị nhắm đến là tài khoản quản trị, hậu quả có thể mở rộng sang việc thay đổi dữ liệu hoặc cấu hình hệ thống.

### Các mối đe dọa chưa được xử lý ngay

Các mối đe dọa từ **M04 đến M08** có điểm rủi ro thấp hơn, dao động từ 5 đến 12. Vì nguồn lực xử lý có giới hạn, nhóm tạm thời tập trung vào những rủi ro có điểm cao hơn trước.

- **M04 – Stored XSS:** Tác động chủ yếu tập trung vào người dùng hoặc tài khoản có liên quan đến nội dung độc hại. Một phần rủi ro có thể được hạn chế thông qua các cơ chế như CSP.
- **M05 – Nghe lén HTTP:** Rủi ro có thể giảm khi hệ thống được triển khai trong môi trường mạng có các biện pháp bảo vệ và được chuyển sang sử dụng HTTPS đầy đủ.
- **M06 và M08:** Hai mối đe dọa này đòi hỏi mức độ tiếp cận sâu hơn vào môi trường hạ tầng nên được đánh giá có khả năng xảy ra thấp hơn.
- **M07 – DoS tài nguyên:** Mối đe dọa chủ yếu ảnh hưởng đến khả năng phục vụ trong một khoảng thời gian nhất định và có thể được khôi phục bằng các biện pháp vận hành phù hợp.

Việc chưa xử lý các mối đe dọa trên **không có nghĩa là chúng được xem là không quan trọng**, mà là nhóm chấp nhận xử lý ở giai đoạn sau để ưu tiên nguồn lực cho các rủi ro có điểm cao hơn.

---

## 3. Ước lượng chi phí xử lý

Việc lựa chọn biện pháp xử lý được xem xét theo **Nguyên lý thứ bảy: chi phí bảo vệ cần tương xứng với giá trị của tài sản được bảo vệ**.

### M01 – Giảm thiểu SQL Injection

- **Chi phí dự kiến:** 16 giờ công kỹ sư, tương đương khoảng **4.000.000 VNĐ**.
- **Cơ sở ước lượng:** Khoảng 8 giờ dành cho việc rà soát và thay thế các truy vấn SQL trực tiếp bằng Prepared Statements hoặc ORM; 8 giờ còn lại dành cho việc xây dựng và chạy các kiểm thử tự động.
- **Cơ sở so sánh:** Dữ liệu điểm của toàn bộ sinh viên là tài sản có giá trị cao. Nếu xảy ra rò rỉ hoặc bị thay đổi trái phép, chi phí khắc phục và ảnh hưởng đến uy tín có thể lớn hơn nhiều so với chi phí triển khai biện pháp phòng vệ.

### M02 – Giảm thiểu lỗi IDOR

- **Chi phí dự kiến:** 20 giờ công kỹ sư, tương đương khoảng **5.000.000 VNĐ**.
- **Cơ sở ước lượng:** Khoảng 12 giờ để bổ sung cơ chế kiểm tra quyền truy cập, bảo đảm `student_id` được đối chiếu với danh tính trong JWT; 8 giờ tiếp theo dành cho kiểm thử các trường hợp truy cập hợp lệ và không hợp lệ.
- **Cơ sở so sánh:** Biện pháp này giúp hạn chế việc một sinh viên có thể truy cập dữ liệu học tập của người khác và bảo vệ thông tin của hơn 10.000 sinh viên. Chi phí triển khai thấp hơn đáng kể so với hậu quả tiềm ẩn của việc lộ dữ liệu trên diện rộng.

### M03 – Giảm thiểu Brute-force

- **Chi phí dự kiến:** 8 giờ công kỹ sư, tương đương khoảng **2.000.000 VNĐ**.
- **Cơ sở ước lượng:** 4 giờ để thiết lập Rate Limiting, giới hạn khoảng 5 lần thử đăng nhập mỗi phút tại API Gateway/Nginx; 4 giờ còn lại dành cho kiểm thử tải và kiểm tra khả năng hoạt động của cơ chế giới hạn.
- **Cơ sở so sánh:** Đây là biện pháp có chi phí triển khai tương đối thấp nhưng giúp giảm đáng kể khả năng kẻ tấn công thử nhiều mật khẩu liên tiếp nhằm chiếm quyền truy cập tài khoản quản trị.
