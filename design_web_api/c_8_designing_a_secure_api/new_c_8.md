# Chương 8: Thiết kế một API an toàn

Chương 8 tập trung vào việc thiết kế các API web với trọng tâm là bảo mật, một yếu tố quan trọng để đảm bảo rằng API không chỉ hoạt động tốt mà còn bảo vệ dữ liệu và hệ thống khỏi các mối đe dọa. Nội dung chương được chia thành các phần chính, bao gồm: tổng quan về bảo mật API, phân vùng API để kiểm soát truy cập, thiết kế với kiểm soát truy cập, xử lý dữ liệu nhạy cảm, và xác định các vấn đề về kiến trúc và giao thức. Dưới đây là chi tiết từng phần.

---

## 8.1 Tổng quan về bảo mật API (Trang 185)

### Khái niệm cơ bản
Bảo mật API không chỉ là việc áp dụng các biện pháp kỹ thuật như mã hóa mà còn là việc thiết kế API sao cho bảo mật được tích hợp từ đầu. Một API an toàn cần đảm bảo rằng:
- Chỉ những người dùng hoặc ứng dụng được ủy quyền mới có thể truy cập.
- Dữ liệu nhạy cảm được bảo vệ khỏi bị rò rỉ.
- Các hành động trên API (như thêm, xóa, sửa dữ liệu) được kiểm soát chặt chẽ.

Bảo mật API thường dựa trên các giao thức tiêu chuẩn như **OAuth 2.0**, một khuôn khổ phổ biến để quản lý quyền truy cập. OAuth 2.0 cho phép các ứng dụng (gọi là *consumer* - người tiêu dùng) truy cập vào API một cách an toàn mà không cần chia sẻ thông tin đăng nhập trực tiếp.

### Quy trình bảo mật cơ bản
Quy trình bảo mật API thường bao gồm ba bước chính:
1. **Đăng ký người tiêu dùng (Registering a consumer)**: Ứng dụng muốn sử dụng API phải đăng ký với nhà cung cấp API để nhận thông tin xác thực (credentials).
2. **Nhận thông tin xác thực (Getting credentials)**: Ứng dụng sử dụng thông tin xác thực để nhận *access token* từ máy chủ ủy quyền.
3. **Thực hiện API call (Making an API call)**: Ứng dụng sử dụng *access token* để gửi yêu cầu đến API.

#### 8.1.1 Đăng ký người tiêu dùng (Trang 185-186)
- **Mô tả**: Trước khi một ứng dụng có thể sử dụng API, nó phải được đăng ký với nhà cung cấp API. Quá trình này thường diễn ra thông qua một *developer portal* (cổng dành cho nhà phát triển), nơi người tiêu dùng cung cấp thông tin về ứng dụng của họ (ví dụ: tên ứng dụng, mục đích sử dụng).
- **Kết quả**: Sau khi đăng ký, ứng dụng nhận được một **client ID** và đôi khi là một **client secret**, là các thông tin xác thực dùng để xác minh danh tính của ứng dụng.
- **Ví dụ**: Giả sử bạn đang phát triển một ứng dụng di động muốn sử dụng API của một mạng xã hội để chia sẻ ảnh. Bạn truy cập cổng phát triển của mạng xã hội (như developers.facebook.com), tạo một ứng dụng mới, và nhận được:
  - Client ID: `abc123`
  - Client Secret: `xyz789`
  Những thông tin này sẽ được sử dụng trong bước tiếp theo để nhận *access token*.

#### 8.1.2 Nhận thông tin xác thực (Trang 186-188)
- **Mô tả**: Sau khi có *client ID* và *client secret*, ứng dụng cần liên hệ với **máy chủ ủy quyền (authorization server)** để nhận *access token*. *Access token* là một chuỗi ký tự tạm thời, cho phép ứng dụng truy cập API trong một khoảng thời gian nhất định.
- **Quy trình**:
  1. Ứng dụng gửi yêu cầu đến máy chủ ủy quyền, kèm theo *client ID* và *client secret*.
  2. Máy chủ xác minh thông tin xác thực và trả về *access token* nếu hợp lệ.
  3. *Access token* thường có thời hạn (ví dụ: 1 giờ) và có thể được làm mới (refresh) nếu cần.
- **Ví dụ**: Tiếp tục ví dụ trên, ứng dụng di động của bạn gửi một yêu cầu HTTP đến máy chủ ủy quyền của mạng xã hội:
  ```http
  POST /oauth/token HTTP/1.1
  Host: api.socialnetwork.com
  Content-Type: application/x-www-form-urlencoded

  grant_type=client_credentials&client_id=abc123&client_secret=xyz789
  ```
  Máy chủ trả về:
  ```json
  {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600
  }
  ```
  *Access token* này sẽ được sử dụng để gọi API.

#### 8.1.3 Thực hiện API call (Trang 188-189)
- **Mô tả**: Với *access token*, ứng dụng có thể gửi yêu cầu đến API. *Access token* thường được gửi trong tiêu đề HTTP `Authorization` của yêu cầu.
- **Quy trình**:
  1. Ứng dụng gửi yêu cầu HTTP đến **máy chủ tài nguyên (resource server)**, kèm theo *access token*.
  2. Máy chủ tài nguyên xác minh *access token* với máy chủ ủy quyền.
  3. Nếu *access token* hợp lệ, máy chủ tài nguyên xử lý yêu cầu và trả về dữ liệu.
- **Ví dụ**: Ứng dụng của bạn muốn tải lên một bức ảnh lên mạng xã hội. Yêu cầu HTTP sẽ như sau:
  ```http
  POST /photos HTTP/1.1
  Host: api.socialnetwork.com
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  Content-Type: multipart/form-data

  [Dữ liệu ảnh]
  ```
  Nếu *access token* hợp lệ, máy chủ sẽ trả về:
  ```json
  {
    "photo_id": "12345",
    "status": "uploaded"
  }
  ```
  Nếu không hợp lệ, máy chủ có thể trả về mã lỗi như `401 Unauthorized` hoặc `403 Forbidden`.

#### 8.1.4 Nhìn nhận thiết kế API từ góc độ bảo mật (Trang 189-191)
- **Mô tả**: Khi thiết kế API, bạn cần luôn nghĩ đến bảo mật ngay từ đầu. Điều này bao gồm:
  - Xác định **ai** có quyền truy cập vào API (người tiêu dùng, ứng dụng, hoặc người dùng cuối).
  - Xác định **những gì** họ có thể làm (quyền truy cập vào tài nguyên hoặc hành động cụ thể).
  - Đảm bảo rằng các yêu cầu được xác minh đúng cách và dữ liệu nhạy cảm được bảo vệ.
- **Nguyên tắc**: Sử dụng **least privilege principle** (nguyên tắc quyền tối thiểu), nghĩa là chỉ cấp quyền vừa đủ để thực hiện nhiệm vụ, không hơn.
- **Ví dụ**: Trong API mạng xã hội, bạn có thể thiết kế để:
  - Cho phép ứng dụng chỉ tải lên ảnh (`POST /photos`) nhưng không được xóa ảnh (`DELETE /photos`) nếu không cần thiết.
  - Chỉ cho phép truy cập vào ảnh của người dùng hiện tại, không phải ảnh của người khác.

---

## 8.2 Phân vùng API để kiểm soát truy cập (Trang 191-199)

### Khái niệm
Phân vùng API là quá trình chia API thành các phần nhỏ hơn (hoặc các phạm vi - *scopes*) để kiểm soát quyền truy cập một cách chi tiết. Điều này giúp đảm bảo rằng người tiêu dùng chỉ có thể truy cập vào các tài nguyên hoặc hành động mà họ được phép.

### 8.2.1 Phạm vi chi tiết (Fine-grained scopes) (Trang 192-194)
- **Mô tả**: Phạm vi chi tiết cho phép kiểm soát quyền truy cập ở mức rất cụ thể, ví dụ: quyền đọc, ghi, hoặc xóa trên từng tài nguyên riêng lẻ.
- **Ưu điểm**:
  - Linh hoạt, phù hợp với các hệ thống phức tạp.
  - Cho phép cấp quyền rất cụ thể, giảm nguy cơ lạm quyền.
- **Nhược điểm**:
  - Tăng độ phức tạp trong quản lý và triển khai.
  - Có thể khó khăn cho người tiêu dùng khi phải chọn đúng phạm vi.
- **Ví dụ**: Trong API mạng xã hội, bạn có thể định nghĩa các phạm vi như:
  - `photos.read`: Cho phép đọc danh sách ảnh.
  - `photos.write`: Cho phép tải lên hoặc chỉnh sửa ảnh.
  - `photos.delete`: Cho phép xóa ảnh.
  Một ứng dụng chỉ cần tải ảnh để hiển thị sẽ chỉ được cấp phạm vi `photos.read`.

#### Minh họa bằng OpenAPI Specification (OAS)
Để mô tả phạm vi chi tiết trong OAS, bạn có thể định nghĩa chúng trong phần `securitySchemes`:
```yaml
components:
  securitySchemes:
    oauth2:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: https://api.socialnetwork.com/oauth/token
          scopes:
            photos.read: Read access to photos
            photos.write: Write access to photos
            photos.delete: Delete access to photos
```

### 8.2.2 Phạm vi thô (Coarse-grained scopes) (Trang 195-197)
- **Mô tả**: Phạm vi thô cấp quyền cho một nhóm hành động hoặc tài nguyên lớn hơn, thay vì chi tiết từng hành động.
- **Ưu điểm**:
  - Đơn giản hơn để triển khai và quản lý.
  - Dễ hiểu hơn cho người tiêu dùng.
- **Nhược điểm**:
  - Ít linh hoạt, có thể cấp quá nhiều quyền không cần thiết.
- **Ví dụ**: Thay vì các phạm vi chi tiết như trên, bạn có thể định nghĩa một phạm vi thô:
  - `photos`: Cho phép đọc, ghi, và xóa ảnh.
  Điều này đơn giản hơn nhưng có thể không an toàn nếu ứng dụng không cần tất cả các quyền.

#### Minh họa bằng OAS
```yaml
components:
  securitySchemes:
    oauth2:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: https://api.socialnetwork.com/oauth/token
          scopes:
            photos: Full access to photos
```

### 8.2.3 Chọn chiến lược phạm vi (Trang 197-198)
- **Mô tả**: Việc chọn giữa phạm vi chi tiết và thô phụ thuộc vào nhu cầu của API và người tiêu dùng. Một số chiến lược:
  - **Phạm vi chi tiết**: Dùng cho các API công khai hoặc khi cần kiểm soát chặt chẽ.
  - **Phạm vi thô**: Dùng cho các API nội bộ hoặc khi muốn đơn giản hóa.
  - **Kết hợp**: Kết hợp cả hai để cân bằng giữa linh hoạt và đơn giản.
- **Ví dụ**: API mạng xã hội có thể cung cấp cả hai:
  - `photos` (thô) cho các ứng dụng nội bộ.
  - `photos.read`, `photos.write` (chi tiết) cho các ứng dụng bên thứ ba.

### 8.2.4 Định nghĩa phạm vi với OAS (Trang 198-200)
- **Mô tả**: OAS cho phép bạn mô tả các phạm vi bảo mật trong phần `securitySchemes` và áp dụng chúng cho các hoạt động cụ thể.
- **Ví dụ**: Để yêu cầu phạm vi `photos.read` cho hành động đọc ảnh:
  ```yaml
  paths:
    /photos:
      get:
        summary: Get list of photos
        security:
          - oauth2: [photos.read]
        responses:
          '200':
            description: List of photos
            content:
              application/json:
                schema:
                  type: array
                  items:
                    type: object
                    properties:
                      photo_id: { type: string }
                      url: { type: string }
  ```

---

## 8.3 Thiết kế với kiểm soát truy cập (Trang 200-202)

### 8.3.1 Biết dữ liệu cần thiết để kiểm soát truy cập (Trang 200-201)
- **Mô tả**: Để kiểm soát truy cập, API cần biết thông tin về người dùng hoặc ứng dụng đang gửi yêu cầu. Thông tin này thường được lấy từ *access token* hoặc các tham số trong yêu cầu.
- **Ví dụ**: Trong API mạng xã hội, để chỉ cho phép người dùng xem ảnh của chính họ:
  - *Access token* chứa thông tin về ID người dùng (ví dụ: `user_id: 123`).
  - API kiểm tra xem `user_id` trong token có khớp với chủ sở hữu của ảnh hay không.
  ```http
  GET /photos?user_id=123 HTTP/1.1
  Host: api.socialnetwork.com
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  ```
  API sẽ từ chối yêu cầu nếu `user_id` không khớp.

### 8.3.2 Điều chỉnh thiết kế khi cần thiết (Trang 201-202)
- **Mô tả**: Đôi khi, thiết kế API cần được điều chỉnh để hỗ trợ kiểm soát truy cập. Ví dụ: thay vì yêu cầu người tiêu dùng cung cấp `user_id` trong mỗi yêu cầu, API có thể tự động lấy thông tin này từ *access token*.
- **Ví dụ**: Ban đầu, API yêu cầu:
  ```http
  GET /photos?user_id=123 HTTP/1.1
  ```
  Sau khi điều chỉnh, API chỉ cần:
  ```http
  GET /photos HTTP/1.1
  ```
  Và lấy `user_id` từ *access token*. Điều này làm giảm nguy cơ người tiêu dùng gửi sai `user_id`.

---

## 8.4 Xử lý dữ liệu nhạy cảm (Trang 203-211)

### 8.4.1 Xử lý dữ liệu nhạy cảm (Trang 203-206)
- **Mô tả**: Dữ liệu nhạy cảm (như thông tin cá nhân, số thẻ tín dụng) cần được bảo vệ trong quá trình truyền tải và lưu trữ. Các biện pháp bao gồm:
  - **Mã hóa (Encryption)**: Sử dụng HTTPS (TLS) để mã hóa dữ liệu khi truyền.
  - **Che giấu (Masking)**: Chỉ trả về một phần dữ liệu nhạy cảm.
  - **Không lưu trữ**: Tránh lưu trữ dữ liệu nhạy cảm nếu không cần thiết.
- **Ví dụ**: Trong API mạng xã hội, khi trả về thông tin người dùng:
  - Thay vì:
    ```json
    {
      "user_id": "123",
      "email": "user@example.com",
      "phone": "1234567890"
    }
    ```
  - Trả về:
    ```json
    {
      "user_id": "123",
      "email": "u***@example.com",
      "phone": "*******890"
    }
    ```
  Điều này giảm nguy cơ rò rỉ dữ liệu.

### 8.4.2 Xử lý mục tiêu nhạy cảm (Sensitive goals) (Trang 207-209)
- **Mô tả**: Một số hành động (goals) của API có thể nhạy cảm, ví dụ: xóa tài khoản người dùng hoặc chuyển tiền. Những hành động này cần được bảo vệ bằng các biện pháp như:
  - Yêu cầu xác nhận bổ sung (như mã OTP).
  - Giới hạn phạm vi quyền truy cập.
- **Ví dụ**: Để xóa một tài khoản, API có thể yêu cầu:
  ```http
  DELETE /account HTTP/1.1
  Host: api.socialnetwork.com
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  X-OTP: 123456
  ```
  Nếu thiếu mã OTP, API trả về lỗi `403 Forbidden`.

### 8.4.3 Thiết kế phản hồi lỗi an toàn (Trang 209-211)
- **Mô tả**: Phản hồi lỗi không được tiết lộ thông tin nhạy cảm (như chi tiết lỗi hệ thống hoặc thông tin người dùng). Thay vào đó, phản hồi lỗi nên chung chung nhưng đủ thông tin để người tiêu dùng xử lý.
- **Ví dụ**:
  - **Không an toàn**:
    ```json
    {
      "error": "Database error: User not found in table users where id=123"
    }
    ```
    Phản hồi này tiết lộ cấu trúc cơ sở dữ liệu và ID người dùng.
  - **An toàn**:
    ```json
    {
      "error": "Invalid user ID"
    }
    ```
    Phản hồi này không tiết lộ thông tin nhạy cảm.

### 8.4.4 Xác định vấn đề kiến trúc và giao thức (Trang 211-212)
- **Mô tả**: Một số vấn đề bảo mật xuất phát từ kiến trúc hoặc giao thức, ví dụ:
  - Sử dụng HTTP thay vì HTTPS.
  - Không xác minh *access token* đúng cách.
  - Cho phép các phương thức HTTP không cần thiết (như `DELETE` ở nơi không nên).
- **Ví dụ**: Nếu API cho phép `DELETE /photos` mà không kiểm tra quyền, kẻ tấn công có thể xóa ảnh của người khác. Để khắc phục, API nên:
  - Yêu cầu phạm vi `photos.delete`.
  - Kiểm tra xem người dùng có quyền xóa ảnh hay không.

---

## Áp dụng vào thực tế: Ví dụ thiết kế API an toàn

Hãy xem xét một API quản lý tài khoản ngân hàng. Dưới đây là cách áp dụng các nguyên tắc từ Chương 8 để thiết kế API này.

### 1. Đăng ký và xác thực
- Ứng dụng di động của ngân hàng đăng ký qua cổng phát triển để nhận *client ID* và *client secret*.
- Khi người dùng đăng nhập, ứng dụng gửi yêu cầu đến máy chủ ủy quyền:
  ```http
  POST /oauth/token HTTP/1.1
  Host: api.bank.com
  Content-Type: application/x-www-form-urlencoded

  grant_type=password&client_id=app123&client_secret=secret456&username=user&password=pass
  ```
  Nhận được *access token*:
  ```json
  {
    "access_token": "abc123token",
    "expires_in": 3600
  }
  ```

### 2. Phân vùng API
- Định nghĩa các phạm vi:
  - `accounts.read`: Xem thông tin tài khoản.
  - `accounts.transfer`: Thực hiện chuyển tiền.
- Trong OAS:
  ```yaml
  components:
    securitySchemes:
      oauth2:
        type: oauth2
        flows:
          password:
            tokenUrl: https://api.bank.com/oauth/token
            scopes:
              accounts.read: View account details
              accounts.transfer: Perform transfers
  ```

### 3. Kiểm soát truy cập
- API chỉ trả về thông tin tài khoản của người dùng hiện tại, lấy từ *access token*:
  ```http
  GET /accounts HTTP/1.1
  Host: api.bank.com
  Authorization: Bearer abc123token
  ```
  Trả về:
  ```json
  [
    {
      "account_id": "12345",
      "balance": 1000.00
    }
  ]
  ```

### 4. Xử lý dữ liệu nhạy cảm
- Che giấu số tài khoản:
  ```json
  {
    "account_id": "****12345",
    "balance": 1000.00
  }
  ```
- Yêu cầu xác nhận OTP cho chuyển tiền:
  ```http
  POST /transfers HTTP/1.1
  Host: api.bank.com
  Authorization: Bearer abc123token
  X-OTP: 654321
  Content-Type: application/json

  {
    "to_account": "67890",
    "amount": 500.00
  }
  ```

### 5. Phản hồi lỗi an toàn
- Nếu OTP sai:
  ```json
  {
    "error": "Invalid OTP"
  }
  ```
  Không tiết lộ chi tiết như "OTP không khớp với giá trị trong cơ sở dữ liệu".

---

## Kết luận
Chương 8 cung cấp một hướng dẫn toàn diện về cách thiết kế API an toàn, từ việc đăng ký người tiêu dùng, phân vùng quyền truy cập, đến xử lý dữ liệu nhạy cảm và phản hồi lỗi. Các nguyên tắc như **least privilege**, sử dụng **OAuth 2.0**, và mã hóa dữ liệu là nền tảng để đảm bảo bảo mật. Bằng cách áp dụng các kỹ thuật này, bạn có thể tạo ra một API không chỉ mạnh mẽ mà còn an toàn trước các mối đe dọa.

Nếu bạn cần thêm ví dụ cụ thể hoặc muốn tôi giải thích chi tiết hơn về bất kỳ phần nào, hãy cho tôi biết!