# Thiết kế API ngắn gọn và có tổ chức tốt (Designing concise and well-organized APIs)
Tập trung vào hai khía cạnh chính để đảm bảo
API của bạn dễ sử dụng: tổ chức API và kích thước API.

## 7.1 Tổ chức API (Organizing an API)
Khả năng sử dụng của API phụ thuộc vào cách các yếu tố của nó được nhóm và sắp xếp hợp lý.

#### 7.1.1 Tổ chức dữ liệu (Organizing Data)
Thiết kế API có tổ chức tốt bắt đầu từ dữ liệu.
- Nhóm dữ liệu (Grouping Data):
  - Nhóm các thuộc tính liên quan lại với nhau bằng tiền tố chung hoặc cấu trúc con.
  - Ví dụ: `overdraftProtection` có `active` và `limit`.
  - Cung cấp cái nhìn rõ ràng hơn về thuộc tính bắt buộc/tùy chọn.
- Sắp xếp dữ liệu (Sorting Data):
  - Sắp xếp các thuộc tính từ quan trọng nhất đến ít quan trọng nhất trong mỗi nhóm.
  - Sắp xếp các nhóm theo mức độ quan trọng.
  - Ví dụ: `typeName`, `type`, `safeToSpend`, `balance`, `overdraftProtection`, `age`.

#### 7.1.2 Tổ chức phản hồi (Organizing Feedback)
Đảm bảo người dùng dễ dàng xử lý các phản hồi.
Phản Hồi Lỗi Nhất Quán: Tất cả lỗi trả về theo định dạng chung (mã lỗi, thông báo, chi tiết).
Nhóm Các Lỗi Liên Quan: Nếu nhiều lỗi, nhóm chúng trong một danh sách, sắp xếp từ quan trọng nhất.
    - Ví dụ: Phản hồi `400 Bad Request` chứa danh sách lỗi chi tiết.
Phản Hồi Thành Công Rõ Ràng: Bao gồm dữ liệu chính và siêu dữ liệu cần thiết (tổng số mục, phân trang).
Sử Dụng Mã Trạng Thái HTTP Phù Hợp: Mã trạng thái (200 OK, 400 Bad Request) phản ánh đúng trạng thái yêu cầu.

#### 7.1.3 Tổ chức mục tiêu (Organizing Goals)
* Các mục tiêu của API cũng cần được tổ chức tốt.
* Tổ chức ảo (Virtual Organization): Sử dụng OpenAPI Specification (OAS) để nhóm mục tiêu.
    - Thêm thuộc tính `tags` vào mỗi hoạt động (ví dụ: "Account", "Transfer").
    - Nhóm các mục tiêu dựa trên quan điểm chức năng.
    - Sắp xếp các danh mục bằng cách định nghĩa danh sách `tags` ở cấp gốc tài liệu OAS.
    - Sắp xếp các hoạt động bên trong mỗi nhóm (ví dụ: thứ tự chuẩn cho các phương thức HTTP).
* Tổ chức vật lý (Physical Organization): Nhóm các tài nguyên bằng cách sử dụng các đường dẫn (paths) có cấu trúc.
    - Ví dụ: Thêm tiền tố đường dẫn như `/account/` hoặc `/transfer/`.

## 7.2 Xác định kích thước API (Sizing an API)
Mỗi khía cạnh của API, bao gồm dữ liệu và mục tiêu,
nên được định kích thước một cách khôn ngoan.

#### 7.2.1 Chọn mức độ chi tiết dữ liệu (Choosing Data Granularity)
* Mức độ chi tiết dữ liệu có hai khía cạnh: số lượng thuộc tính và chiều sâu cấu trúc.
* Số lượng thuộc tính (Number of Properties):
    - Thuộc tính phải phù hợp về mặt chức năng trong ngữ cảnh sử dụng.
    - Hơn 20 thuộc tính cần xem xét lại việc tổ chức hoặc từng thuộc tính.
    - Yêu cầu lượng dữ liệu tối thiểu.
* Chiều sâu cấu trúc (Depth):
    - Không nên vượt quá ba cấp độ chiều sâu trong cấu trúc dữ liệu.
    - Cần có sự cân bằng giữa tổ chức và kích thước dữ liệu.

#### 7.2.2 Chọn mức độ chi tiết mục tiêu (Choosing Goal Granularity)
* Tránh việc đưa quá nhiều dữ liệu vào một mục tiêu duy nhất.
    - Ví dụ: Thay vì `get bank account` trả về cả tài khoản và lịch sử giao dịch, nên có mục tiêu riêng `GET /accounts/{id}/transactions`.
* Tránh các mục tiêu "tất cả trong một" (does-it-all goals).
    - Ví dụ: Thay vì cập nhật địa chỉ qua mục tiêu cập nhật tài khoản chung, nên có mục tiêu `update address` độc lập.
* Mức độ chi tiết của một mục tiêu được xác định bởi ngữ cảnh và khả năng sử dụng.

#### 7.2.3 Chọn mức độ chi tiết API (Choosing API Granularity)
* Nếu các nhóm mục tiêu có thể hoàn toàn độc lập về chức năng, nên xem xét việc chia một API lớn thành các API nhỏ hơn, có chức năng cụ thể hơn.
* Các API nhỏ hơn dễ quản lý và tái sử dụng độc lập.
* Việc chia nhỏ có thể thực hiện từ giai đoạn xác định mục tiêu ban đầu.
* Ví dụ: "Banking API" có thể chia thành "Bank Account API" và "Money Transfer API".