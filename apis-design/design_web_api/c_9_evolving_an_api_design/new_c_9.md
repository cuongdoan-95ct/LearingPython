# Chương 9: Phát triển thiết kế API (Evolving an API Design)

Chương này giải thích cách sửa đổi một API mà không gây ảnh hưởng tiêu cực lớn đến người dùng, cách quản lý phiên bản API (versioning), và cách thiết kế API để dễ dàng mở rộng trong tương lai. Các chủ đề chính bao gồm:

1. **Thiết kế sự phát triển của API** (Designing API evolutions): Tránh các thay đổi phá vỡ (breaking changes) và đảm bảo API vẫn hoạt động ổn định.
2. **Quản lý phiên bản API** (Versioning an API): Phân biệt phiên bản API và phiên bản triển khai, chọn cách biểu diễn phiên bản phù hợp, và hiểu tác động của versioning.
3. **Thiết kế API với tính mở rộng** (Designing APIs with extensibility in mind): Tạo API có khả năng mở rộng dữ liệu, tương tác, luồng xử lý, và toàn bộ API.

Dưới đây là nội dung chi tiết của từng phần, được dịch sang tiếng Việt và giải thích rõ ràng với các ví dụ minh họa.

---

## 9.1 Thiết kế sự phát triển của API (Designing API evolutions)

Phần này (trang 215–229) tập trung vào cách sửa đổi một API mà không làm gián đoạn các ứng dụng khách (clients) đang sử dụng nó. Khi một API được phát hành, nó trở thành một hợp đồng giao diện (interface contract) giữa nhà cung cấp (provider) và người tiêu dùng (consumer). Mọi thay đổi cần được thực hiện cẩn thận để tránh các **thay đổi phá vỡ** (breaking changes), tức là những thay đổi khiến ứng dụng khách không còn hoạt động đúng như trước.

### Tránh các thay đổi phá vỡ trong dữ liệu đầu ra (Avoiding breaking changes in output data)

**Khái niệm**: Dữ liệu đầu ra (output data) là thông tin mà API trả về cho ứng dụng khách, ví dụ như phản hồi JSON từ một yêu cầu GET. Thay đổi dữ liệu đầu ra có thể làm hỏng ứng dụng khách nếu chúng phụ thuộc vào cấu trúc hoặc giá trị cụ thể. Các thay đổi phá vỡ bao gồm:

- Xóa một thuộc tính (property).
- Đổi tên một thuộc tính.
- Thay đổi kiểu dữ liệu của một thuộc tính (ví dụ, từ chuỗi sang số).
- Thay đổi định dạng hoặc ý nghĩa của dữ liệu (ví dụ, định dạng ngày từ `YYYY-MM-DD` sang `MM/DD/YYYY`).

**Nguyên tắc**: Để tránh phá vỡ, bạn cần đảm bảo tính tương thích ngược (backward compatibility), tức là các ứng dụng khách cũ vẫn hoạt động với API mới.

**Ví dụ từ sách (trang 215–220)**:

Giả sử bạn có một API trả về thông tin người dùng qua endpoint `/users/{id}`:

```json
{
  "id": "123",
  "name": "John Doe",
  "birthDate": "1980-01-01"
}
```

1. **Xóa thuộc tính**:
   - Nếu bạn xóa thuộc tính `birthDate`, ứng dụng khách dựa vào nó (ví dụ, để tính tuổi) sẽ gặp lỗi. Thay vào đó, nếu cần ngừng cung cấp `birthDate`, bạn có thể đánh dấu nó là **deprecated** (khuyến cáo không dùng nữa) trong tài liệu OpenAPI Specification (OAS):

     ```yaml
     birthDate:
       type: string
       format: date
       deprecated: true
     ```

     Sau một thời gian, khi chắc chắn không ai sử dụng `birthDate`, bạn mới xóa nó.

2. **Đổi tên thuộc tính**:
   - Nếu bạn đổi `name` thành `fullName`, ứng dụng khách sẽ không tìm thấy `name` và bị lỗi. Thay vào đó, bạn có thể giữ `name` và thêm `fullName` với cùng giá trị:

     ```json
     {
       "id": "123",
       "name": "John Doe",
       "fullName": "John Doe",
       "birthDate": "1980-01-01"
     }
     ```

     Sau đó, đánh dấu `name` là deprecated và thông báo cho người dùng chuyển sang `fullName`.

3. **Thay đổi kiểu dữ liệu**:
   - Nếu `id` từ chuỗi (`"123"`) đổi thành số (`123`), ứng dụng khách mong đợi chuỗi sẽ gặp lỗi. Giải pháp là giữ `id` là chuỗi và thêm một thuộc tính mới, ví dụ `numericId`:

     ```json
     {
       "id": "123",
       "numericId": 123,
       "name": "John Doe",
       "birthDate": "1980-01-01"
     }
     ```

4. **Thay đổi định dạng**:
   - Nếu `birthDate` đổi từ `YYYY-MM-DD` sang `MM/DD/YYYY`, ứng dụng khách có thể hiểu sai ngày. Thay vì thay đổi định dạng, bạn có thể thêm một thuộc tính mới:

     ```json
     {
       "id": "123",
       "name": "John Doe",
       "birthDate": "1980-01-01",
       "birthDateUS": "01/01/1980"
     }
     ```

**Ví dụ bổ sung**:
Hãy tưởng tượng bạn đang quản lý một API thương mại điện tử trả về thông tin sản phẩm:

```json
{
  "productId": "abc123",
  "name": "Laptop XYZ",
  "price": 999.99
}
```

Nếu bạn muốn thay đổi `price` từ số (999.99) thành đối tượng để hỗ trợ nhiều loại tiền tệ:

```json
{
  "price": {
    "amount": 999.99,
    "currency": "USD"
  }
}
```

Điều này sẽ phá vỡ ứng dụng khách mong đợi `price` là số. Thay vào đó, bạn có thể:

```json
{
  "productId": "abc123",
  "name": "Laptop XYZ",
  "price": 999.99,
  "priceDetails": {
    "amount": 999.99,
    "currency": "USD"
  }
}
```

- Giữ `price` để tương thích ngược.
- Thêm `priceDetails` cho định dạng mới.
- Đánh dấu `price` là deprecated và khuyến khích dùng `priceDetails`.

**Lưu ý thực hành**:
- Luôn tài liệu hóa các thay đổi trong OAS, sử dụng thuộc tính `deprecated`.
- Thông báo trước cho người dùng qua tài liệu, email, hoặc cổng nhà phát triển (developer portal).
- Theo dõi việc sử dụng API (API analytics) để biết thuộc tính nào còn được dùng trước khi xóa.

### Tránh các thay đổi phá vỡ trong dữ liệu đầu vào và tham số (Avoiding breaking changes to input data and parameters)

**Khái niệm**: Dữ liệu đầu vào (input data) là thông tin ứng dụng khách gửi đến API, như tham số truy vấn (query parameters), tham số đường dẫn (path parameters), hoặc thân yêu cầu (request body). Các thay đổi phá vỡ ở đầu vào bao gồm:

- Yêu cầu một tham số bắt buộc mới.
- Xóa hoặc đổi tên tham số.
- Thay đổi kiểu hoặc định dạng dữ liệu của tham số.
- Thay đổi ý nghĩa của tham số.

**Nguyên tắc**: API phải chấp nhận dữ liệu đầu vào cũ và xử lý chúng đúng cách, đồng thời hỗ trợ các định dạng mới nếu cần.

**Ví dụ từ sách (trang 220–223)**:

Giả sử bạn có một endpoint POST `/users` để tạo người dùng, với thân yêu cầu:

```json
{
  "name": "John Doe",
  "email": "john@example.com"
}
```

1. **Thêm tham số bắt buộc mới**:
   - Nếu bạn thêm `phoneNumber` là bắt buộc:

     ```json
     {
       "name": "John Doe",
       "email": "john@example.com",
       "phoneNumber": "1234567890"
     }
     ```

     Các ứng dụng khách cũ không gửi `phoneNumber` sẽ nhận lỗi 400 Bad Request. Thay vào đó, làm `phoneNumber` tùy chọn:

     ```yaml
     phoneNumber:
       type: string
       required: false
     ```

     Nếu cần giá trị mặc định, API có thể gán giá trị mặc định (ví dụ, `null` hoặc chuỗi rỗng).

2. **Xóa hoặc đổi tên tham số**:
   - Nếu bạn đổi `email` thành `emailAddress`, ứng dụng khách gửi `email` sẽ gặp lỗi. Giải pháp là chấp nhận cả `email` và `emailAddress`, ánh xạ chúng nội bộ:

     ```json
     {
       "name": "John Doe",
       "email": "john@example.com",
       "emailAddress": "john@example.com"
     }
     ```

     Sau đó, đánh dấu `email` là deprecated.

3. **Thay đổi kiểu dữ liệu**:
   - Nếu `name` từ chuỗi đổi thành đối tượng (ví dụ, `{ "firstName": "John", "lastName": "Doe" }`), ứng dụng khách gửi chuỗi sẽ lỗi. Thay vào đó, giữ `name` và thêm `fullNameDetails`:

     ```json
     {
       "name": "John Doe",
       "fullNameDetails": {
         "firstName": "John",
         "lastName": "Doe"
       },
       "email": "john@example.com"
     }
     ```

**Ví dụ bổ sung**:
Trong API thương mại điện tử, endpoint POST `/orders` nhận:

```json
{
  "productId": "abc123",
  "quantity": 2
}
```

Nếu bạn muốn `quantity` đổi thành mảng để hỗ trợ nhiều sản phẩm:

```json
{
  "productId": "abc123",
  "quantities": [2]
}
```

Điều này sẽ phá vỡ ứng dụng khách cũ. Thay vào đó:

```json
{
  "productId": "abc123",
  "quantity": 2,
  "quantities": [2]
}
```

- Chấp nhận cả `quantity` (số) và `quantities` (mảng).
- Nếu chỉ có `quantity`, API chuyển nó thành `[quantity]`.
- Đánh dấu `quantity` là deprecated.

**Lưu ý thực hành**:
- Sử dụng JSON Schema trong OAS để định nghĩa các tham số tùy chọn.
- Xử lý nội bộ để ánh xạ dữ liệu cũ sang định dạng mới.
- Kiểm tra đầu vào để đảm bảo API không từ chối yêu cầu hợp lệ từ ứng dụng khách cũ.

### Tránh các thay đổi phá vỡ trong phản hồi thành công và lỗi (Avoiding breaking changes in success and error feedback)

**Khái niệm**: Phản hồi thành công (success feedback) và lỗi (error feedback) là các mã trạng thái HTTP (HTTP status codes) và nội dung phản hồi. Thay đổi phá vỡ ở đây bao gồm:

- Thay đổi mã trạng thái HTTP (ví dụ, từ 200 OK sang 201 Created).
- Thay đổi cấu trúc hoặc nội dung của phản hồi lỗi.
- Thay đổi ý nghĩa của phản hồi thành công.

**Nguyên tắc**: Giữ mã trạng thái và cấu trúc phản hồi ổn định, hoặc cung cấp các phản hồi bổ sung mà không xóa phản hồi cũ.

**Ví dụ từ sách (trang 223–225)**:

1. **Thay đổi mã trạng thái**:
   - Endpoint POST `/users` trả về 200 OK với thân phản hồi:

     ```json
     {
       "id": "123",
       "name": "John Doe"
     }
     ```

     Nếu bạn đổi thành 201 Created, ứng dụng khách mong đợi 200 có thể xử lý sai. Giải pháp là giữ 200 OK, nhưng thêm tiêu đề `Location` (thường đi với 201) để hỗ trợ ứng dụng mới:

     ```http
     HTTP/1.1 200 OK
     Location: /users/123
     Content-Type: application/json

     {
       "id": "123",
       "name": "John Doe"
     }
     ```

2. **Thay đổi phản hồi lỗi**:
   - Phản hồi lỗi ban đầu cho yêu cầu không hợp lệ:

     ```json
     {
       "error": "Invalid email"
     }
     ```

     Nếu bạn đổi thành:

     ```json
     {
       "code": "INVALID_EMAIL",
       "message": "Email is not valid"
     }
     ```

     Ứng dụng khách dựa vào `error` sẽ lỗi. Thay vào đó, kết hợp cả hai:

     ```json
     {
       "error": "Invalid email",
       "code": "INVALID_EMAIL",
       "message": "Email is not valid"
     }
     ```

     Đánh dấu `error` là deprecated.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, endpoint GET `/products/{id}` trả về 404 Not Found nếu sản phẩm không tồn tại:

```json
{
  "error": "Product not found"
}
```

Nếu bạn muốn cải thiện phản hồi lỗi:

```json
{
  "error": "Product not found",
  "code": "PRODUCT_NOT_FOUND",
  "details": "Product with ID abc123 does not exist"
}
```

- Giữ `error` để tương thích.
- Thêm `code` và `details` cho thông tin chi tiết hơn.
- Đánh dấu `error` là deprecated.

**Lưu ý thực hành**:
- Sử dụng các mã trạng thái HTTP tiêu chuẩn (RFC 7231) để tránh bất ngờ.
- Định nghĩa cấu trúc lỗi trong OAS, đảm bảo tương thích ngược.
- Theo dõi phản hồi nào được ứng dụng khách sử dụng nhiều nhất.

### Tránh các thay đổi phá vỡ trong mục tiêu và luồng xử lý (Avoiding breaking changes to goals and flows)

**Khái niệm**: Mục tiêu (goals) là các hành động mà API cho phép (ví dụ, tạo người dùng, lấy danh sách sản phẩm). Luồng xử lý (flows) là chuỗi các mục tiêu để hoàn thành một nhiệm vụ. Thay đổi phá vỡ ở đây bao gồm:

- Xóa một mục tiêu (endpoint).
- Thay đổi cách một mục tiêu hoạt động (ví dụ, yêu cầu thêm bước).
- Thay đổi luồng xử lý (ví dụ, yêu cầu gọi nhiều endpoint hơn).

**Nguyên tắc**: Giữ các mục tiêu và luồng cũ hoạt động, đồng thời cung cấp các mục tiêu hoặc luồng mới nếu cần.

**Ví dụ từ sách (trang 225–226)**:

1. **Xóa endpoint**:
   - Nếu bạn xóa endpoint GET `/users/list`, ứng dụng khách dựa vào nó sẽ lỗi. Thay vào đó, giữ endpoint cũ và thêm endpoint mới, ví dụ GET `/users`. Đánh dấu `/users/list` là deprecated trong OAS:

     ```yaml
     /users/list:
       get:
         deprecated: true
         ...
     ```

2. **Thay đổi luồng xử lý**:
   - Ban đầu, để tạo người dùng, ứng dụng khách gọi POST `/users`. Nếu bạn yêu cầu thêm bước xác thực qua endpoint `/auth/verify`, luồng cũ sẽ bị phá vỡ. Thay vào đó, giữ POST `/users` như cũ và cung cấp luồng mới với `/auth/verify` cho ứng dụng muốn bảo mật hơn.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, để đặt hàng, ứng dụng khách gọi POST `/orders`. Nếu bạn muốn thêm bước xác nhận giỏ hàng qua POST `/cart/confirm`, bạn có thể:

- Giữ POST `/orders` để tạo đơn hàng trực tiếp.
- Thêm POST `/cart/confirm` và POST `/orders/confirmed` cho luồng mới.
- Tài liệu hóa luồng mới và khuyến khích sử dụng nó.

**Lưu ý thực hành**:
- Sử dụng thuộc tính `deprecated` trong OAS để đánh dấu mục tiêu cũ.
- Cung cấp các endpoint thay thế trước khi xóa endpoint cũ.
- Kiểm tra luồng xử lý bằng cách mô phỏng hành vi của ứng dụng khách.

### Tránh vi phạm bảo mật và thay đổi phá vỡ (Avoiding security breaches and breaking changes)

**Khái niệm**: Thay đổi API có thể vô tình gây ra lỗ hổng bảo mật hoặc phá vỡ ứng dụng khách nếu không được xử lý đúng cách. Ví dụ, tăng yêu cầu bảo mật (như yêu cầu mã hóa mới) có thể làm ứng dụng khách cũ không hoạt động.

**Ví dụ từ sách (trang 226–228)**:

1. **Thay đổi giao thức bảo mật**:
   - Nếu API chuyển từ HTTP sang HTTPS, ứng dụng khách cũ sử dụng HTTP sẽ không kết nối được. Giải pháp là duy trì cả HTTP và HTTPS trong thời gian chuyển đổi, đồng thời thông báo cho người dùng nâng cấp lên HTTPS.

2. **Thay đổi phạm vi OAuth**:
   - Nếu bạn thay đổi phạm vi (scope) OAuth từ `read` thành `read:user`, ứng dụng khách sử dụng `read` sẽ bị từ chối truy cập. Thay vào đó, hỗ trợ cả `read` và `read:user` trong thời gian chuyển đổi, đánh dấu `read` là deprecated.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, nếu bạn yêu cầu tất cả yêu cầu phải có tiêu đề `Authorization` với token JWT, ứng dụng khách không gửi tiêu đề này sẽ lỗi. Giải pháp:

- Cho phép yêu cầu không có `Authorization` trong thời gian chuyển đổi, sử dụng cơ chế xác thực cũ (ví dụ, API key).
- Thông báo cho người dùng thêm `Authorization`.
- Sau thời gian chuyển đổi, bắt buộc `Authorization`.

**Lưu ý thực hành**:
- Kiểm tra bảo mật trước khi triển khai thay đổi (ví dụ, sử dụng OWASP guidelines).
- Theo dõi nhật ký truy cập để phát hiện ứng dụng khách bị ảnh hưởng.
- Cung cấp thời gian chuyển đổi dài (ví dụ, 6 tháng) cho các thay đổi bảo mật.

### Nhận thức về hợp đồng giao diện vô hình (Being aware of the invisible interface contract)

**Khái niệm**: Hợp đồng giao diện vô hình (invisible interface contract) là những kỳ vọng không được tài liệu hóa mà ứng dụng khách có về API. Ví dụ, ứng dụng khách có thể dựa vào một lỗi cụ thể hoặc một thuộc tính không được định nghĩa rõ trong tài liệu.

**Ví dụ từ sách (trang 228–229)**:
- API trả về lỗi 400 với thông điệp `"Invalid email"`. Ứng dụng khách phân tích chuỗi này để hiển thị thông báo. Nếu bạn đổi thông điệp thành `"Email is not valid"`, ứng dụng khách có thể không hiển thị đúng thông báo. Đây là một thay đổi phá vỡ vô hình.

**Giải pháp**:
- Tài liệu hóa mọi khía cạnh của API, kể cả lỗi, trong OAS.
- Theo dõi cách ứng dụng khách sử dụng API để phát hiện các kỳ vọng vô hình.
- Tránh thay đổi các chi tiết nhỏ (như thông điệp lỗi) mà không thông báo.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, phản hồi GET `/products/{id}` trả về thuộc tính `stock` (số lượng tồn kho). Dù tài liệu không cam kết `stock` luôn có, ứng dụng khách dựa vào nó để hiển thị trạng thái "Còn hàng". Nếu bạn xóa `stock`, ứng dụng khách sẽ gặp vấn đề. Giải pháp là giữ `stock` và thêm thuộc tính mới như `inventoryStatus`.

**Lưu ý thực hành**:
- Sử dụng công cụ phân tích API để phát hiện các thuộc tính hoặc hành vi không tài liệu hóa.
- Thử nghiệm với ứng dụng khách thực tế trước khi thay đổi.

### Thay đổi phá vỡ không phải lúc nào cũng là vấn đề (Introducing a breaking change is not always a problem)

**Khái niệm**: Trong một số trường hợp, thay đổi phá vỡ là cần thiết, đặc biệt nếu API có ít người dùng hoặc thay đổi mang lại lợi ích lớn. Tuy nhiên, cần quản lý cẩn thận.

**Ví dụ từ sách (trang 229)**:
- Nếu API của bạn chỉ có một ứng dụng khách nội bộ (private API), bạn có thể thực hiện thay đổi phá vỡ và cập nhật ứng dụng khách cùng lúc.
- Đối với public API, thay đổi phá vỡ cần thông báo trước và thời gian chuyển đổi dài.

**Ví dụ bổ sung**:
Trong API thương mại điện tử nội bộ, bạn quyết định đổi endpoint `/products/list` thành `/products` để đơn giản hóa. Vì chỉ có một ứng dụng khách nội bộ, bạn có thể thực hiện thay đổi này ngay sau khi cập nhật ứng dụng khách.

**Lưu ý thực hành**:
- Đối với public API, cung cấp ít nhất 6–12 tháng thông báo trước khi thực hiện thay đổi phá vỡ.
- Đối với private API, phối hợp với đội ngũ phát triển ứng dụng khách để giảm thiểu gián đoạn.

---

## 9.2 Quản lý phiên bản API (Versioning an API)

Phần này (trang 229–239) giải thích cách quản lý phiên bản API để hỗ trợ các thay đổi mà không làm gián đoạn ứng dụng khách. Phiên bản API (API versioning) khác với phiên bản triển khai (implementation versioning), và cần chọn cách biểu diễn phiên bản phù hợp với người dùng.

### Phân biệt phiên bản API và phiên bản triển khai (Contrasting API and implementation versioning)

**Khái niệm**: 
- **Phiên bản triển khai** (implementation versioning) là phiên bản của mã nguồn hoặc cơ sở hạ tầng phía sau API. Ví dụ, bạn nâng cấp máy chủ từ Java 8 lên Java 11, nhưng API vẫn hoạt động như cũ.
- **Phiên bản API** (API versioning) là phiên bản của hợp đồng giao diện, tức là cách ứng dụng khách tương tác với API (endpoint, tham số, phản hồi). Thay đổi hợp đồng giao diện yêu cầu phiên bản API mới.

**Ví dụ từ sách (trang 230–233)**:
- Một API có endpoint GET `/users/{id}` trả về:

  ```json
  {
    "id": "123",
    "name": "John Doe"
  }
  ```

  Nếu bạn nâng cấp cơ sở dữ liệu từ MySQL sang PostgreSQL mà không thay đổi phản hồi, đây là thay đổi triển khai, không cần phiên bản API mới.
- Nếu bạn thêm thuộc tính `email` vào phản hồi:

  ```json
  {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com"
  }
  ```

  Đây là thay đổi hợp đồng giao diện, có thể yêu cầu phiên bản API mới (tùy thuộc vào tính tương thích ngược).

**Ví dụ bổ sung**:
Trong API thương mại điện tử, nếu bạn tối ưu hóa thuật toán tìm kiếm sản phẩm nhưng giữ endpoint GET `/products` không đổi, đây là thay đổi triển khai. Nếu bạn đổi endpoint thành `/products/search`, đây là thay đổi API, cần phiên bản mới.

**Lưu ý thực hành**:
- Chỉ tạo phiên bản API mới khi có thay đổi phá vỡ hoặc thay đổi lớn trong hợp đồng giao diện.
- Sử dụng semantic versioning (ví dụ, v1.0.0, v1.1.0, v2.0.0) để quản lý phiên bản API.

### Chọn cách biểu diễn phiên bản từ góc độ người dùng (Choosing an API versioning representation from the consumer’s perspective)

**Khái niệm**: Cách biểu diễn phiên bản API (versioning representation) là cách bạn chỉ định phiên bản trong yêu cầu API, ví dụ qua URL, tiêu đề HTTP, hoặc tham số truy vấn. Cách này cần dễ hiểu và thuận tiện cho ứng dụng khách.

**Các phương pháp phổ biến** (trang 232–234):
1. **Phiên bản trong URL**:
   - Ví dụ: `https://api.example.com/v1/users/{id}`
   - Ưu điểm: Dễ thấy, dễ sử dụng, hỗ trợ tốt trong trình duyệt và công cụ như Postman.
   - Nhược điểm: Có thể làm URL phức tạp nếu có nhiều phiên bản.

2. **Phiên bản trong tiêu đề HTTP**:
   - Ví dụ: `Accept: application/vnd.example.v1+json`
   - Ưu điểm: Giữ URL sạch, phù hợp với REST principles.
   - Nhược điểm: Phức tạp hơn cho ứng dụng khách, cần cấu hình tiêu đề.

3. **Phiên bản trong tham số truy vấn**:
   - Ví dụ: `https://api.example.com/users/{id}?version=1`
   - Ưu điểm: Linh hoạt, dễ thêm vào yêu cầu.
   - Nhược điểm: Ít phổ biến, có thể gây nhầm lẫn với các tham số khác.

**Ví dụ từ sách**:
- API ban đầu là `GET /users/{id}` (v1). Khi thêm `email` và đổi cấu trúc phản hồi, bạn tạo phiên bản mới `GET /v2/users/{id}`:

  ```json
  {
    "userId": "123",
    "fullName": "John Doe",
    "email": "john@example.com"
  }
  ```

  Ứng dụng khách cũ tiếp tục dùng `/users/{id}`, còn ứng dụng mới dùng `/v2/users/{id}`.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, bạn có `GET /products` (v1). Khi đổi cấu trúc phản hồi sản phẩm, bạn tạo `GET /v2/products`:

```json
{
  "products": [
    {
      "id": "abc123",
      "name": "Laptop XYZ",
      "price": {
        "amount": 999.99,
        "currency": "USD"
      }
    }
  ]
}
```

Ứng dụng khách cũ vẫn dùng `/products`, còn ứng dụng mới dùng `/v2/products`.

**Lưu ý thực hành**:
- Phiên bản trong URL là lựa chọn phổ biến nhất vì tính đơn giản.
- Tránh thay đổi cách biểu diễn phiên bản (ví dụ, từ URL sang tiêu đề) vì điều này có thể phá vỡ ứng dụng khách.
- Tài liệu hóa rõ ràng cách truy cập các phiên bản trong OAS.

### Chọn mức độ chi tiết của phiên bản API (Choosing API versioning granularity)

**Khái niệm**: Mức độ chi tiết của phiên bản (versioning granularity) quyết định phạm vi áp dụng phiên bản, ví dụ toàn bộ API, một nhóm endpoint, hay một endpoint cụ thể.

**Các mức độ chi tiết** (trang 234–238):
1. **Phiên bản toàn bộ API**:
   - Áp dụng phiên bản cho tất cả endpoint (ví dụ, `/v1/*`).
   - Ưu điểm: Đơn giản, dễ quản lý.
   - Nhược điểm: Thay đổi nhỏ ở một endpoint cũng yêu cầu phiên bản mới cho toàn API.

2. **Phiên bản nhóm endpoint**:
   - Áp dụng phiên bản cho một nhóm endpoint liên quan (ví dụ, `/v1/users/*` và `/v2/products/*`).
   - Ưu điểm: Linh hoạt hơn, chỉ ảnh hưởng đến nhóm bị thay đổi.
   - Nhược điểm: Phức tạp hơn trong quản lý.

3. **Phiên bản từng endpoint**:
   - Áp dụng phiên bản cho từng endpoint cụ thể (ví dụ, `/users/{id}/v1`).
   - Ưu điểm: Rất chi tiết, chỉ ảnh hưởng endpoint bị thay đổi.
   - Nhược điểm: Rất phức tạp, khó duy trì.

**Ví dụ từ sách**:
- Nếu bạn đổi cấu trúc của `/users/{id}`, bạn có thể tạo `/v2/users/{id}` nhưng giữ `/products` ở v1. Đây là phiên bản nhóm endpoint.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, nếu bạn chỉ đổi `/orders` để hỗ trợ nhiều sản phẩm, bạn có thể tạo `/v2/orders` mà giữ `/products` và `/users` ở v1.

**Lưu ý thực hành**:
- Phiên bản toàn bộ API là phổ biến nhất cho public API để đơn giản hóa.
- Phiên bản nhóm endpoint phù hợp cho private API hoặc microservices.
- Tránh phiên bản từng endpoint trừ khi thực sự cần thiết.

### Hiểu tác động của phiên bản API ngoài thiết kế (Understanding the impact of API versioning beyond design)

**Khái niệm**: Phiên bản API không chỉ ảnh hưởng đến thiết kế mà còn đến triển khai, tài liệu, và hỗ trợ người dùng. Ví dụ, duy trì nhiều phiên bản tăng chi phí bảo trì.

**Ví dụ từ sách (trang 238–239)**:
- Nếu bạn hỗ trợ v1 và v2 của API, bạn cần:
  - Hai bộ mã nguồn hoặc logic xử lý riêng.
  - Tài liệu riêng cho v1 và v2.
  - Theo dõi việc sử dụng để biết khi nào có thể ngừng hỗ trợ v1.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, hỗ trợ `/v1/products` và `/v2/products` yêu cầu:
- Máy chủ xử lý cả hai phiên bản.
- Cổng nhà phát triển cung cấp tài liệu cho cả v1 và v2.
- Hệ thống giám sát để biết bao nhiêu ứng dụng khách còn dùng v1.

**Lưu ý thực hành**:
- Sử dụng API gateway để định tuyến yêu cầu đến phiên bản đúng.
- Đặt thời hạn ngừng hỗ trợ (sunset period) cho phiên bản cũ (ví dụ, 12 tháng).
- Theo dõi số liệu sử dụng để quyết định khi nào ngừng hỗ trợ phiên bản cũ.

---

## 9.3 Thiết kế API với tính mở rộng (Designing APIs with extensibility in mind)

Phần này (trang 239–245) giải thích cách thiết kế API để dễ dàng mở rộng trong tương lai, giảm thiểu nhu cầu thay đổi phá vỡ. Tính mở rộng (extensibility) áp dụng cho dữ liệu, tương tác, luồng xử lý, và toàn bộ API.

### Thiết kế dữ liệu mở rộng (Designing extensible data)

**Khái niệm**: Dữ liệu mở rộng là dữ liệu có thể thêm thuộc tính hoặc giá trị mới mà không phá vỡ ứng dụng khách. Điều này tuân theo **Postel’s Law**: “Be conservative in what you send, be liberal in what you accept” (Gửi dữ liệu hạn chế, chấp nhận dữ liệu linh hoạt).

**Nguyên tắc**:
- Cho phép thêm thuộc tính mới trong phản hồi mà không bắt buộc ứng dụng khách xử lý chúng.
- Chấp nhận các thuộc tính không xác định trong dữ liệu đầu vào.
- Sử dụng định dạng linh hoạt như JSON thay vì XML hoặc CSV cứng nhắc.

**Ví dụ từ sách (trang 240–243)**:
- Phản hồi GET `/users/{id}` ban đầu:

  ```json
  {
    "id": "123",
    "name": "John Doe"
  }
  ```

  Để mở rộng, bạn có thể thêm thuộc tính `email` mà không phá vỡ ứng dụng khách:

  ```json
  {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com"
  }
  ```

  Ứng dụng khách cũ bỏ qua `email`, còn ứng dụng mới sử dụng nó.

- Trong yêu cầu POST `/users`, nếu ứng dụng khách gửi thuộc tính không xác định:

  ```json
  {
    "name": "John Doe",
    "unknownField": "some value"
  }
  ```

  API nên bỏ qua `unknownField` thay vì trả lỗi.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, phản hồi GET `/products/{id}`:

```json
{
  "id": "abc123",
  "name": "Laptop XYZ",
  "price": 999.99
}
```

Để mở rộng, thêm `tags`:

```json
{
  "id": "abc123",
  "name": "Laptop XYZ",
  "price": 999.99,
  "tags": ["electronics", "laptop"]
}
```

Ứng dụng khách cũ bỏ qua `tags`, còn ứng dụng mới sử dụng nó để lọc sản phẩm.

**Lưu ý thực hành**:
- Trong JSON Schema, không đặt `additionalProperties: false` để cho phép thuộc tính mới.
- Kiểm tra ứng dụng khách có bỏ qua thuộc tính không xác định hay không.
- Tránh thay đổi ý nghĩa của thuộc tính hiện có.

### Thiết kế tương tác mở rộng (Designing extensible interactions)

**Khái niệm**: Tương tác mở rộng là các hành động (như gọi endpoint) có thể hỗ trợ tham số hoặc hành vi mới mà không phá vỡ ứng dụng khách.

**Nguyên tắc**:
- Cho phép tham số tùy chọn mới.
- Hỗ trợ các giá trị mới cho tham số kiểu liệt kê (enum).
- Xử lý các tham số không hợp lệ một cách linh hoạt.

**Ví dụ từ sách (trang 243–244)**:
- Endpoint GET `/users` có tham số `sort`:

  ```http
  GET /users?sort=name
  ```

  Để mở rộng, thêm giá trị mới cho `sort`:

  ```http
  GET /users?sort=email
  ```

  Ứng dụng khách cũ vẫn dùng `sort=name`, còn ứng dụng mới dùng `sort=email`.

- Nếu ứng dụng khách gửi tham số không hợp lệ:

  ```http
  GET /users?sort=invalid
  ```

  API nên trả về phản hồi mặc định (ví dụ, không sắp xếp) thay vì lỗi.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, GET `/products` có tham số `category`:

```http
GET /products?category=electronics
```

Để mở rộng, thêm tham số `tag`:

```http
GET /products?category=electronics&tag=laptop
```

Ứng dụng khách cũ bỏ qua `tag`, còn ứng dụng mới lọc sản phẩm theo cả `category` và `tag`.

**Lưu ý thực hành**:
- Định nghĩa tham số tùy chọn trong OAS.
- Xử lý tham số không hợp lệ bằng giá trị mặc định.
- Theo dõi tham số nào được sử dụng để biết nhu cầu mở rộng.

### Thiết kế luồng xử lý mở rộng (Designing extensible flows)

**Khái niệm**: Luồng xử lý mở rộng là chuỗi các mục tiêu có thể thêm bước mới mà không phá vỡ ứng dụng khách cũ.

**Nguyên tắc**:
- Cho phép ứng dụng khách bỏ qua các bước mới.
- Cung cấp endpoint tổng hợp (aggregate endpoints) để đơn giản hóa luồng mới.

**Ví dụ từ sách (trang 244–245)**:
- Luồng cũ để tạo người dùng: POST `/users`.
- Luồng mới yêu cầu xác thực: POST `/auth/verify` rồi POST `/users/verified`.

  Ứng dụng khách cũ vẫn dùng POST `/users`, còn ứng dụng mới dùng luồng xác thực.

**Ví dụ bổ sung**:
Trong API thương mại điện tử, luồng cũ để đặt hàng: POST `/orders`. Luồng mới yêu cầu xác nhận giỏ hàng: POST `/cart/confirm` rồi POST `/orders/confirmed`. Giữ POST `/orders` để tương thích với ứng dụng cũ.

**Lưu ý thực hành**:
- Tài liệu hóa các luồng mới trong user guide.
- Cung cấp endpoint tổng hợp để giảm số bước trong luồng mới.
- Kiểm tra luồng cũ vẫn hoạt động sau khi thêm luồng mới.

### Thiết kế API mở rộng (Designing extensible APIs)

**Khái niệm**: Một API mở rộng là API có thể thêm endpoint, tài nguyên, hoặc chức năng mới mà không ảnh hưởng đến ứng dụng khách cũ.

**Nguyên tắc**:
- Sử dụng cấu trúc tài nguyên linh hoạt (ví dụ, `/resources/{type}`).
- Hỗ trợ hypermedia (HATEOAS) để ứng dụng khách tự khám phá tài nguyên mới.
- Giữ các endpoint cũ ổn định.

**Ví dụ từ sách (trang 245)**:
- API có `/users`. Để mở rộng, thêm `/groups` mà không ảnh hưởng đến `/users`. Ứng dụng khách cũ tiếp tục dùng `/users`, còn ứng dụng mới khám phá `/groups` qua hypermedia:

  ```json
  {
    "id": "123",
    "name": "John Doe",
    "links": [
      { "rel": "self", "href": "/users/123" },
      { "rel": "group", "href": "/groups/456" }
    ]
  }
  ```

**Ví dụ bổ sung**:
Trong API thương mại điện tử, bạn có `/products`. Để mở rộng, thêm `/reviews`:

```json
{
  "id": "abc123",
  "name": "Laptop XYZ",
  "links": [
    { "rel": "self", "href": "/products/abc123" },
    { "rel": "reviews", "href": "/products/abc123/reviews" }
  ]
}
```

Ứng dụng khách cũ bỏ qua `reviews`, còn ứng dụng mới theo liên kết để lấy đánh giá.

**Lưu ý thực hành**:
- Sử dụng hypermedia để hỗ trợ khám phá tài nguyên mới.
- Đảm bảo tài nguyên mới không xung đột với tài nguyên cũ.
- Theo dõi việc sử dụng hypermedia để biết ứng dụng khách có tận dụng nó hay không.

---

## Tổng kết và áp dụng thực tế

Chương 9 cung cấp một hướng dẫn chi tiết để phát triển API một cách bền vững, tránh gián đoạn cho ứng dụng khách, và chuẩn bị cho các thay đổi trong tương lai. Dưới đây là các điểm chính để áp dụng thực tế:

1. **Tránh thay đổi phá vỡ**:
   - Luôn giữ tương thích ngược bằng cách giữ các thuộc tính, tham số, mã trạng thái, và endpoint cũ.
   - Sử dụng thuộc tính `deprecated` trong OAS để đánh dấu các phần không còn khuyến khích.
   - Thông báo trước và cung cấp thời gian chuyển đổi dài.

2. **Quản lý phiên bản**:
   - Chỉ tạo phiên bản mới khi cần thiết (thay đổi phá vỡ).
   - Sử dụng phiên bản trong URL cho tính đơn giản.
   - Đặt thời hạn ngừng hỗ trợ cho phiên bản cũ.

3. **Thiết kế mở rộng**:
   - Cho phép thêm thuộc tính và tham số mới mà không phá vỡ.
   - Hỗ trợ hypermedia để khám phá tài nguyên mới.
   - Xử lý dữ liệu và tham số không xác định một cách linh hoạt.

**Ví dụ thực tế áp dụng**:
Giả sử bạn đang phát triển một API quản lý nhà hàng với endpoint GET `/menu` trả về:

```json
{
  "items": [
    {
      "id": "m1",
      "name": "Pizza Margherita",
      "price": 10.99
    }
  ]
}
```

- **Thay đổi phá vỡ**: Nếu bạn muốn đổi `price` thành `priceDetails`, giữ cả hai:

  ```json
  {
    "items": [
      {
        "id": "m1",
        "name": "Pizza Margherita",
        "price": 10.99,
        "priceDetails": {
          "amount": 10.99,
          "currency": "USD"
        }
      }
    ]
  }
  ```

- **Phiên bản API**: Nếu bạn đổi cấu trúc hoàn toàn, tạo `/v2/menu` và giữ `/menu` cho v1.
- **Mở rộng**: Thêm `categories` mà không phá vỡ:

  ```json
  {
    "items": [
      {
        "id": "m1",
        "name": "Pizza Margherita",
        "price": 10.99,
        "categories": ["pizza", "vegetarian"]
      }
    ]
  }
  ```