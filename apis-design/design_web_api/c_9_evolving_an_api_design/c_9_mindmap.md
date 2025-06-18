# Evolving an API design

## 1. Thiết kế các tiến hóa API (Designing API evolutions)

### Thiết kế các tiến hóa API đòi hỏi sự cẩn trọng đặc biệt để tránh những thay đổi gây lỗi (breaking changes). ``Một thay đổi gây lỗi là một thay đổi sẽ gây ra vấn đề cho người tiêu dùng nếu họ không cập nhật mã của mình``.

### Tránh các thay đổi gây lỗi trong dữ liệu đầu ra (Avoiding breaking changes in output data)
* Đổi tên thuộc tính (Renaming a property)
* Di chuyển thuộc tính (Moving a property)
* Xóa thuộc tính bắt buộc (Removing a mandatory property)
* Thay đổi ý nghĩa của thuộc tính (Changing a property's meaning)
* Thay đổi loại hoặc định dạng thuộc tính (Changing a property's type or format)
* Thay đổi đặc điểm thuộc tính (Modifying (decreasing) characteristics)
* Xóa giá trị khỏi enum (Removing values from enums)
* Thêm giá trị mới vào enum (Adding new values to enum)
* ``Cách an toàn để thay đổi dữ liệu đầu ra``:
    - Chỉ thêm các thuộc tính mới.
    - Thêm một thuộc tính mới để chỉ ra một tình trạng đặc biệt.
    - Thêm một thuộc tính mở rộng chứa dữ liệu cũ và mới.

### Tránh các thay đổi gây lỗi đối với dữ liệu đầu vào và tham số (Avoiding breaking changes to input data and parameters)
* Đổi tên thuộc tính (Renaming a property)
* Xóa thuộc tính (Removing a property)
* Thay đổi định dạng hoặc loại (Modifying format or type)
* Thay đổi đặc điểm thuộc tính (Modifying (decreasing) characteristics)
* Xóa giá trị khỏi enum (Removing values from enums)
* Thay đổi ý nghĩa của thuộc tính (Changing a property's meaning)
* Thêm một thuộc tính bắt buộc mới (Adding a mandatory property)
* ``Cách an toàn để thay đổi dữ liệu đầu vào``:
    - Chỉ thêm các thuộc tính tùy chọn.
    - Biến một thuộc tính bắt buộc hiện có thành tùy chọn.
    - Tăng đặc điểm của thuộc tính.

### Tránh các thay đổi gây lỗi trong phản hồi thành công và lỗi (Avoiding breaking changes in success and error feedback)
* Thay đổi cấu trúc dữ liệu lỗi (Modifying error data structure)
* Thay đổi mã trạng thái HTTP (Modifying HTTP status codes)
* Thay đổi ý nghĩa của mã trạng thái HTTP (Changing HTTP status code meaning)

### Tránh các thay đổi gây lỗi đối với mục tiêu và luồng (Avoiding breaking changes to goals and flows)
* Đổi tên hoặc xóa mục tiêu (Renaming or removing goals)
* Thêm mục tiêu bắt buộc mới vào luồng (Adding a new mandatory goal to a flow)

### Tránh các vi phạm bảo mật và thay đổi gây lỗi (Avoiding security breaches and breaking changes)
* Thay đổi cách thức lấy token (Modifying how tokens are acquired)
* Xóa thông tin nhạy cảm khỏi token truy cập (Removing sensitive data from access tokens)
* Thay đổi phạm vi (scopes)

### Lưu ý về hợp đồng giao diện vô hình (Being aware of the invisible interface contract)
* Người dùng API có thể dựa vào các ``hành vi có thể quan sát được của hệ thống`` không được mô tả rõ ràng trong tài liệu.
* Thay đổi hiệu suất của API có thể gây lỗi hết thời gian chờ (timeout).
* ``Luật Hyrum`` (Hyrum's law): "Với đủ số lượng người dùng của một API, bất kể bạn hứa gì trong hợp đồng: tất cả các hành vi có thể quan sát được của hệ thống của bạn sẽ bị ai đó phụ thuộc vào".

### Khi nào một thay đổi gây lỗi không phải là vấn đề? (Introducing a breaking change is not always a problem)
* Trong bối cảnh ``API nội bộ`` hoặc ``API riêng tư``, việc giới thiệu các thay đổi gây lỗi có thể chấp nhận được.
* Đối với các ``API công cộng``, các thay đổi gây lỗi chắc chắn không phải là một lựa chọn.

## 2. Phiên bản hóa API (Versioning an API)

## Khi một thay đổi gây lỗi là không thể tránh khỏi, ``phiên bản hóa API là một cách khôn ngoan để xử lý tình huống đó``.

### Phiên bản hóa API so với phiên bản hóa triển khai (Contrasting API and implementation versioning)
* ``Phiên bản của API`` phát triển dựa trên ``những thay đổi đối với hợp đồng giao diện``.
* ``Phiên bản của việc triển khai`` liên quan đến cách mã nguồn phát triển và có thể thay đổi mà không ảnh hưởng đến phiên bản API.
* Người dùng API chủ yếu chỉ quan tâm đến các ``thay đổi phiên bản lớn`` báo hiệu các thay đổi gây lỗi.

### Chọn cách biểu diễn phiên bản API từ góc độ người dùng (Choosing an API versioning representation from the consumer's perspective)
* ``Trong đường dẫn`` (Path versioning): `GET /v1/transfers`.
* ``Trong tên miền hoặc tên miền phụ`` (Domain or subdomain versioning): `api.bankingcompany.com`.
* ``Trong tham số truy vấn`` (Query parameter): `GET /transfers?version=2`.
* ``Trong header tùy chỉnh`` (Custom header): `Version: 2`.
* ``Sử dụng đàm phán nội dung`` (Content negotiation): `Accept: application/vnd.bankingapi.v2+json`.
* ``Dựa trên cấu hình người dùng`` (Consumer's configuration).
* Mỗi cách đều có ưu và nhược điểm.

### Chọn mức độ chi tiết của phiên bản API (Choosing API versioning granularity)
* ``Cấp độ API`` (API-level versioning): Phiên bản hóa toàn bộ API. (Thực hành phổ biến nhất cho REST API).
* ``Cấp độ tài nguyên`` (Resource-level versioning): Phiên bản hóa mỗi tài nguyên riêng lẻ.
* ``Cấp độ mục tiêu/hoạt động`` (Goal/operation-level versioning): Phiên bản hóa mỗi mục tiêu/hoạt động riêng lẻ.
* ``Cấp độ dữ liệu/thông điệp`` (Data/message-level versioning): Phiên bản hóa dữ liệu trong phần thân yêu cầu và phản hồi.
* Có thể ``kết hợp các mức độ chi tiết khác nhau`` tùy thuộc vào bối cảnh.

### Hiểu tác động của phiên bản hóa API ngoài thiết kế (Understanding the impact of API versioning beyond design)
* ``Tác động đến người dùng``: Giới thiệu một thay đổi gây lỗi có thể khiến người dùng không hài lòng.
* ``Tác động đến triển khai và quản lý sản phẩm``: Hỗ trợ nhiều phiên bản API đòi hỏi công sức triển khai và bảo trì bổ sung.

## 3. Thiết kế API có khả năng mở rộng (Designing APIs with extensibility in mind)

## ``Khả năng mở rộng (extensibility)`` là một nguyên tắc thiết kế phần mềm, nơi việc triển khai xem xét sự phát triển trong tương lai, nhằm giảm thiểu tác động đến các chức năng hệ thống hiện có khi thêm mới hoặc sửa đổi. Bằng cách thiết kế cẩn thận dữ liệu, tương tác và luồng, và chọn mức độ chi tiết phù hợp cho phiên bản hóa, chúng ta có thể thiết kế API có khả năng mở rộng, giúp dễ dàng phát triển và giảm rủi ro thay đổi gây lỗi.

### Thiết kế dữ liệu có khả năng mở rộng (Designing extensible data)
* Sử dụng các envelope dữ liệu (data envelopes).
* Cung cấp dữ liệu tự mô tả (self-descriptive data).
* Sử dụng danh sách cho các thuộc tính tương tự (Using lists for similar properties).
* Chọn loại thuộc tính cẩn thận (Choose property types carefully).

### Thiết kế tương tác có khả năng mở rộng (Designing extensible interactions)
* Bỏ qua các tham số không xác định hoặc không hợp lệ (Ignoring unknown or invalid parameters).
* Áp dụng giới hạn mềm (Soft limits).
* Không hy sinh sự chính xác để tránh lỗi.

### Thiết kế luồng có khả năng mở rộng (Designing extensible flows)
* Độc lập hóa các bước trong luồng.
* Sử dụng các đầu vào và đầu ra được chấp nhận rộng rãi.
* Tránh sự phụ thuộc vào luồng UI cụ thể.

### Thiết kế API có khả năng mở rộng (Designing extensible APIs)
* Xây dựng các API nhỏ hơn.
* Đánh giá từng mục tiêu mới.