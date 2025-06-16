# Designing REST APIs

## Giới thiệu về REST API
Để thiết kế một REST API, trước tiên cần hiểu cách nó hoạt động.

### Phân tích một lệnh gọi REST API
* Người dùng (client) gửi yêu cầu HTTP (ví dụ: `GET /products/P123`) đến máy chủ (server).
* Yêu cầu bao gồm:
    - Phương thức HTTP: `GET` (truy xuất tài nguyên).
    - Đường dẫn (path): `/products/P123` (xác định một tài nguyên).
    - Yêu cầu có thể có phần thân (body) (để tạo, cập nhật).
* Máy chủ trả về phản hồi HTTP:
    - Mã trạng thái (status code): `200 OK` (thành công).
    - Phần thân phản hồi (response body): Chứa nội dung tài nguyên (ví dụ: dữ liệu JSON sản phẩm).

### Các nguyên tắc cơ bản của HTTP
* Sử dụng Request và Response.
* Đường dẫn (paths) xác định tài nguyên.
* Phương thức HTTP: GET (truy xuất), POST (tạo), PUT (cập nhật/thay thế), DELETE (xóa), PATCH (cập nhật một phần).
* Mã trạng thái HTTP: 200 OK, 404 Not Found, 400 Bad Request...
* Tiêu đề HTTP: Cung cấp siêu dữ liệu (Content-Type, Authorization).
* Phần thân yêu cầu/phản hồi: Chứa dữ liệu (JSON, XML).

### Các nguyên tắc cơ bản của REST API
* Cơ chế hoạt động tương tự trình duyệt web.
* Mục tiêu chức năng biểu diễn bằng hành động trên tài nguyên.
* Tài nguyên (Resources):
  * Mọi thứ được biểu diễn là tài nguyên, xác định bằng URI (ví dụ: `/users/123`).
* Phương thức HTTP: Sử dụng các phương thức HTTP tiêu chuẩn (CRUD).
* Không trạng thái (Stateless):
  * Mỗi yêu cầu chứa tất cả thông tin cần thiết
  * Máy chủ không lưu trạng thái phiên.
* Khả năng lưu trữ (Cacheability): Phản hồi có thể được lưu trữ.
* Phân tầng (Layered System): Hệ thống có thể có nhiều lớp mà không ảnh hưởng client.
* Truyền trạng thái biểu diễn (Representational State Transfer): Dữ liệu truyền dưới dạng biểu diễn trạng thái tài nguyên.

## Chuyển đổi các mục tiêu API thành một REST API
Quá trình thiết kế giao diện lập trình REST API là
chuyển đổi các mục tiêu API (từ API goals canvas)
thành cấu trúc REST API.

### 3.2.1 Xác định tài nguyên và các mối quan hệ của chúng bằng API goals canvas
Bước đầu tiên trong chuyển đổi. Nguồn: API goals canvas.
Để xác định tài nguyên (resources): Liệt kê tất cả
các danh từ mà động từ chính của mục tiêu áp dụng vào.
- Ví dụ: "thêm sản phẩm vào danh mục". `product`, `catalog` là tài nguyên.
- `free query` không phải tài nguyên vì động từ "tìm kiếm" không áp dụng trực tiếp.

Để xác định mối quan hệ giữa các tài nguyên:
Xem xét các mục tiêu có nhiều hơn một tài nguyên.
- Ví dụ: "thêm sản phẩm vào danh mục" -> `catalog` chứa nhiều `product`.
- Tập hợp tài nguyên (collection resource): Một tài nguyên chứa các tài nguyên khác cùng loại.

### 3.2.2 Xác định hành động và các tham số, giá trị trả về của chúng bằng API goals canvas
Bước thứ hai trong chuyển đổi.
Mục tiêu biểu diễn bằng các hành động trên tài nguyên.
Để xác định hành động (action): Lấy
động từ chính của mục tiêu và
liên kết nó với tài nguyên mà nó áp dụng.
- Ví dụ: "get product" -> `get` áp dụng cho `product`. "add product to catalog" -> `add` áp dụng cho `catalog`.

API goals canvas cung cấp danh sách đầy đủ các đầu vào (parameters) và đầu ra (returns).
Lọc đầu vào: Loại trừ tài nguyên mà hành động áp dụng.
Các đầu ra trở thành giá trị trả về của hành động.

### 3.2.3 Thiết kế đường dẫn cho tài nguyên
Đường dẫn được sử dụng để xác định các tài nguyên.
Cần đảm bảo tính thân thiện với người dùng.
Định dạng được áp dụng rộng rãi nhất: `/{tên số nhiều phản ánh loại mục của tập hợp}/{id của mục}` (ví dụ: `/products/{productId}`).

### 3.2.4 Biểu diễn hành động bằng HTTP
Ánh xạ các hành động này tới các phương thức HTTP tiêu chuẩn.
GET: Đọc hoặc truy xuất tài nguyên. (Ví dụ: `GET /products/{productId}`)
POST: Tạo tài nguyên mới, thêm mục vào tập hợp, kích hoạt hành động không CRUD.
(Ví dụ: `POST /products`)
PUT: Thay thế hoàn toàn tài nguyên, hoặc tạo tài nguyên mới với ID đã biết.
(Ví dụ: `PUT /products/{productId}`)
PATCH: Cập nhật một phần tài nguyên.
(Ví dụ: `PATCH /products/{productId}`)
DELETE: Xóa bỏ tài nguyên.
(Ví dụ: `DELETE /products/{productId}`)

## Thiết kế dữ liệu của API
Thiết kế chi tiết dữ liệu được trao đổi qua tham số và giá trị trả về.

### Quy trình thiết kế dữ liệu
Thiết kế khái niệm (concepts).
Thiết kế phản hồi (responses) từ khái niệm.
Thiết kế tham số (parameters) từ khái niệm hoặc phản hồi.
Kiểm tra nguồn dữ liệu của tham số.

### 3.3.1 Thiết kế khái niệm
Đảm bảo tính thân thiện với người dùng và không để lộ các chi tiết nội bộ.
Bắt đầu bằng cách liệt kê các thuộc tính (properties) của khái niệm.
Phân tích tính dễ hiểu, liên quan, hữu ích của thuộc tính. Có thể cần đổi tên, xóa, tái cấu trúc.
Tên thuộc tính: Càng tự giải thích càng tốt.
Loại dữ liệu (type): Sử dụng các loại cơ bản, dễ di chuyển (`string`, `number`, `date`, `boolean`).
Xác định bắt buộc (required) hay tùy chọn (optional).
Thêm mô tả (description) nếu cần.

### 3.3.2 Thiết kế phản hồi từ khái niệm
Cùng một khái niệm có thể có các biểu diễn khác nhau trong các ngữ cảnh khác nhau.
Thiết kế phản hồi dựa trên khái niệm gốc nhưng điều chỉnh theo ngữ cảnh của hành động.
Ví dụ: `GET /product` trả về đầy đủ; `search for products` trả về tóm tắt.

### 3.3.3 Thiết kế tham số từ khái niệm hoặc phản hồi
Tương tự phản hồi, các tham số đầu vào có thể
có các biểu diễn khác nhau tùy ngữ cảnh.
Thiết kế tham số dựa trên khái niệm gốc
hoặc phản hồi nhưng điều chỉnh cho phù hợp
với thông tin người dùng cần cung cấp.
Tham số chỉ nên chứa dữ liệu cần thiết.

### 3.3.4 Kiểm tra nguồn dữ liệu của tham số
Người dùng phải có khả năng cung cấp tất cả
dữ liệu cần thiết (biết hoặc lấy từ API khác).
Giúp phát hiện các mục tiêu bị thiếu
hoặc vấn đề quan điểm của nhà cung cấp.
Quy trình kiểm tra:
- Với mỗi thuộc tính, xác định nguồn gốc.
- Nếu không rõ, có thể xóa hoặc thêm mục tiêu API mới.

### 3.3.5 Thiết kế các tham số khác
Áp dụng nguyên tắc thiết kế thân thiện với người dùng
và kiểm tra khả năng cung cấp dữ liệu.
- Ví dụ: tham số truy vấn tự do (`free-query`).

## 3.4 Tìm sự cân bằng khi đối mặt với các thách thức thiết kế
Đôi khi, biểu diễn tuân thủ mô hình lại không
thân thiện với người dùng.
Cần thực hiện các đánh đổi (trade-offs).

### 3.4.1 Ví dụ về đánh đổi trong REST
Mục tiêu "check out cart" trong API mua sắm.
Lựa chọn:
  - `POST /cart/check-out`: Thân thiện, nhưng không tuân thủ REST (dùng động từ trong đường dẫn).
  - `PATCH /cart`: Tuân thủ hơn (cập nhật trạng thái), nhưng có thể khó hiểu.
  - `POST /orders`: Tuân thủ nhất (tạo tài nguyên mới), nhưng không trực quan.

### 3.4.2 Cân bằng giữa tính thân thiện với người dùng và sự tuân thủ
Thách thức là tìm sự cân bằng phù hợp.
Nắm vững nguyên tắc cơ bản của kiểu API giúp tìm giải pháp gần mô hình.
Đòi hỏi thực hành, quan sát API khác, và trao đổi với người dùng/nhà thiết kế.

## 3.5 Hiểu lý do tại sao kiến trúc REST lại quan trọng đối với việc thiết kế bất kỳ loại API nào
Kiến trúc REST là bộ các nguyên tắc thiết kế
hệ thống phân tán, hữu ích
khi thiết kế bất kỳ loại API nào, không chỉ REST API.

### 3.5.1 Giới thiệu kiến trúc REST
Ra đời năm 2000 bởi Roy Fielding (trong quá trình làm việc với HTTP 1.1).
Mục tiêu:
- Tạo điều kiện xây dựng các hệ thống phân tán hiệu quả, có khả năng mở rộng và đáng tin cậy.

Sáu ràng buộc để được coi là RESTful:
1. Client/server separation (Tách biệt máy khách/máy chủ).
2. Statelessness (Không trạng thái).
3. Cacheability (Khả năng bộ nhớ đệm).
4. Layered system (Hệ thống phân lớp).
5. Code on demand (Mã theo yêu cầu): Tùy chọn.
6. Uniform interface (Giao diện đồng nhất): Ràng buộc cơ bản nhất.

### 3.5.2 Tác động của các ràng buộc REST đối với thiết kế API
Cung cấp nền tảng vững chắc cho thiết kế API.
Client/server separation và Layered system: Liên quan đến
nguyên tắc quan điểm của người dùng.
Giúp xây dựng API dễ hiểu, dễ sử dụng, tái sử dụng và tiến hóa.
Uniform interface: Liên quan đến sử dụng đường dẫn duy nhất và phương thức HTTP tiêu chuẩn.
Tạo giao diện đồng nhất, nhất quán.
Các ràng buộc khác (Statelessness, Cacheability, Code on demand) sẽ được khám phá chi tiết sau.
