### Chương 5: Thiết Kế Một API Đơn Giản và Dễ Hiểu

Chương 5 tập trung vào việc thiết kế một API đơn giản, dễ sử dụng và dễ hiểu ngay lập tức cho cả con người (nhà phát triển) và máy móc (ứng dụng). Một API "straightforward" (đơn giản và rõ ràng) giúp người dùng đạt được mục tiêu của họ mà không phải suy nghĩ quá nhiều về cách API hoạt động. Trong chương này, tác giả Arnaud Lauret giải thích cách thiết kế các biểu diễn dữ liệu (representations), tương tác (interactions), và luồng công việc (flows) để đảm bảo tính đơn giản và dễ sử dụng. Nội dung được chia thành ba phần chính:

1. **Thiết Kế Các Biểu Diễn Đơn Giản** (*Designing Straightforward Representations*)
2. **Thiết Kế Các Tương Tác Đơn Giản** (*Designing Straightforward Interactions*)
3. **Thiết Kế Các Luồng Công Việc Đơn Giản** (*Designing Straightforward Flows*)

Dưới đây, tôi sẽ giải thích chi tiết từng phần, kèm theo ví dụ cụ thể bằng tiếng Việt để minh họa các khái niệm.

---

### 5.1 Thiết Kế Các Biểu Diễn Đơn Giản (*Designing Straightforward Representations*)

Mục tiêu của phần này là tạo ra các biểu diễn dữ liệu (dữ liệu mà API trả về hoặc nhận vào) rõ ràng, dễ hiểu và dễ sử dụng. Biểu diễn là cách dữ liệu được trình bày trong các phản hồi (responses) hoặc yêu cầu (requests) của API. Một biểu diễn đơn giản giúp nhà phát triển nhanh chóng hiểu được dữ liệu và sử dụng nó mà không gặp khó khăn.

Phần này được chia thành ba khía cạnh chính:
- **Chọn tên rõ ràng** (*Choosing Crystal-Clear Names*)
- **Chọn kiểu dữ liệu và định dạng dễ sử dụng** (*Choosing Easy-to-Use Data Types and Formats*)
- **Chọn dữ liệu sẵn sàng để sử dụng** (*Choosing Ready-to-Use Data*)

#### 5.1.1 Chọn Tên Rõ Ràng (*Choosing Crystal-Clear Names*)

Tên của các thuộc tính (properties), tham số (parameters), và các yếu tố khác trong API phải rõ ràng, dễ hiểu, và phản ánh chính xác ý nghĩa của chúng. Tên không nên sử dụng thuật ngữ kỹ thuật nội bộ (jargon) hoặc các từ viết tắt khó hiểu, vì điều này có thể gây nhầm lẫn cho người dùng API.

**Nguyên tắc:**
- Sử dụng các tên mô tả ý nghĩa của dữ liệu một cách trực tiếp.
- Tránh sử dụng thuật ngữ nội bộ của tổ chức hoặc tên mang tính kỹ thuật không cần thiết.
- Đặt tên nhất quán, sử dụng các quy tắc đặt tên như camelCase hoặc snake_case, tùy thuộc vào tiêu chuẩn của API.

**Ví dụ:**
Giả sử bạn đang thiết kế một API cho một ứng dụng quản lý cửa hàng sách. Bạn cần trả về thông tin về một cuốn sách. Thay vì sử dụng tên như sau:

```json
{
  "bk_id": "12345",
  "bk_ttl": "Chiến Thắng",
  "pub_dt": "2023-01-15"
}
```

Hãy sử dụng các tên rõ ràng hơn:

```json
{
  "bookId": "12345",
  "title": "Chiến Thắng",
  "publicationDate": "2023-01-15"
}
```

**Giải thích:**
- `bk_id` không rõ ràng vì "bk" là viết tắt và không nói lên ý nghĩa cụ thể. `bookId` rõ ràng hơn vì nó chỉ ra rằng đây là ID của một cuốn sách.
- `bk_ttl` khó hiểu; `title` trực tiếp và dễ hiểu hơn.
- `pub_dt` không mô tả đầy đủ; `publicationDate` cho biết chính xác đây là ngày xuất bản.

**Ví dụ thực tế bằng tiếng Việt:**
Giả sử bạn xây dựng một API cho một ứng dụng bán hàng trực tuyến tại Việt Nam. Bạn muốn trả về thông tin đơn hàng. Thay vì:

```json
{
  "dh_ma": "DH001",
  "kh_ten": "Nguyen Van A",
  "ngay_dat": "2025-06-01"
}
```

Hãy dùng:

```json
{
  "orderId": "DH001",
  "customerName": "Nguyễn Văn A",
  "orderDate": "2025-06-01"
}
```

Điều này giúp nhà phát triển (người dùng API) dễ dàng hiểu rằng `orderId` là mã đơn hàng, `customerName` là tên khách hàng, và `orderDate` là ngày đặt hàng.

#### 5.1.2 Chọn Kiểu Dữ Liệu và Định Dạng Dễ Sử Dụng (*Choosing Easy-to-Use Data Types and Formats*)

Kiểu dữ liệu (data types) và định dạng (formats) phải dễ xử lý và phù hợp với mục đích của dữ liệu. Điều này giúp giảm thiểu lỗi và công sức khi nhà phát triển sử dụng API.

**Nguyên tắc:**
- Sử dụng các kiểu dữ liệu đơn giản như chuỗi (string), số (number), hoặc boolean thay vì các kiểu phức tạp không cần thiết.
- Sử dụng các định dạng chuẩn (standards) như ISO 8601 cho ngày giờ hoặc các định dạng phổ biến như JSON.
- Tránh các định dạng tùy chỉnh (custom formats) trừ khi thực sự cần thiết.

**Ví dụ:**
Thay vì trả về ngày xuất bản của một cuốn sách dưới dạng một chuỗi tùy chỉnh:

```json
{
  "publicationDate": "15-Jan-2023"
}
```

Hãy sử dụng định dạng chuẩn ISO 8601:

```json
{
  "publicationDate": "2023-01-15"
}
```

**Giải thích:**
- Định dạng ISO 8601 (`YYYY-MM-DD`) được hỗ trợ rộng rãi bởi các thư viện lập trình và dễ phân tích (parse) hơn so với định dạng tùy chỉnh như `15-Jan-2023`.
- Điều này giúp nhà phát triển không phải viết mã bổ sung để xử lý định dạng ngày tháng.

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng trực tuyến, bạn trả về giá trị tổng tiền đơn hàng. Thay vì:

```json
{
  "tong_tien": "1.000.000 VND"
}
```

Hãy dùng:

```json
{
  "totalAmount": 1000000,
  "currency": "VND"
}
```

**Giải thích:**
- `totalAmount` là một số (number) thay vì chuỗi, giúp dễ dàng thực hiện các phép tính toán học (như cộng, trừ).
- `currency` được tách riêng để chỉ định loại tiền tệ, tuân theo chuẩn ISO 4217.

#### 5.1.3 Chọn Dữ Liệu Sẵn Sàng Để Sử Dụng (*Choosing Ready-to-Use Data*)

Dữ liệu mà API cung cấp nên ở dạng sẵn sàng để sử dụng, giảm thiểu nhu cầu xử lý bổ sung từ phía người dùng API. Điều này có nghĩa là dữ liệu nên được định dạng hoặc cấu trúc sao cho phù hợp với cách nó sẽ được sử dụng.

**Nguyên tắc:**
- Cung cấp dữ liệu đã được xử lý hoặc chuyển đổi để phù hợp với nhu cầu của người dùng.
- Tránh trả về dữ liệu thô (raw data) mà yêu cầu nhà phát triển phải thực hiện các bước xử lý phức tạp.

**Ví dụ:**
Giả sử API trả về thông tin về một cuốn sách, bao gồm giá gốc và chiết khấu. Thay vì:

```json
{
  "originalPrice": 200000,
  "discountPercentage": 10
}
```

Hãy trả về thêm giá sau chiết khấu:

```json
{
  "originalPrice": 200000,
  "discountPercentage": 10,
  "finalPrice": 180000
}
```

**Giải thích:**
- Việc cung cấp `finalPrice` giúp nhà phát triển không phải tự tính toán giá sau chiết khấu (`originalPrice * (1 - discountPercentage / 100)`).
- Điều này làm cho API dễ sử dụng hơn, đặc biệt trong các ứng dụng hiển thị giá trực tiếp cho người dùng.

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, bạn trả về thông tin về một đơn hàng. Thay vì:

```json
{
  "items": [
    { "productId": "SP001", "quantity": 2, "unitPrice": 50000 },
    { "productId": "SP002", "quantity": 1, "unitPrice": 30000 }
  ]
}
```

Hãy cung cấp thêm tổng giá trị:

```json
{
  "items": [
    { "productId": "SP001", "quantity": 2, "unitPrice": 50000, "subtotal": 100000 },
    { "productId": "SP002", "quantity": 1, "unitPrice": 30000, "subtotal": 30000 }
  ],
  "totalAmount": 130000
}
```

**Giải thích:**
- `subtotal` cho mỗi mặt hàng và `totalAmount` cho toàn bộ đơn hàng giúp nhà phát triển dễ dàng hiển thị thông tin mà không cần tính toán thêm.
- Điều này đặc biệt hữu ích trong các ứng dụng thương mại điện tử tại Việt Nam, nơi người dùng mong muốn thấy tổng giá trị đơn hàng ngay lập tức.

---

### 5.2 Thiết Kế Các Tương Tác Đơn Giản (*Designing Straightforward Interactions*)

Tương tác (interactions) là cách người dùng (ứng dụng) gửi yêu cầu đến API và nhận phản hồi. Một tương tác đơn giản yêu cầu đầu vào (input) rõ ràng, phản hồi lỗi (error feedback) đầy đủ và thông tin, cũng như phản hồi thành công (success feedback) hữu ích.

Phần này được chia thành các khía cạnh sau:
- **Yêu cầu đầu vào đơn giản** (*Requesting Straightforward Inputs*)
- **Xác định tất cả các phản hồi lỗi có thể xảy ra** (*Identifying All Possible Error Feedbacks*)
- **Trả về phản hồi lỗi thông tin** (*Returning Informative Error Feedback*)
- **Trả về phản hồi lỗi đầy đủ** (*Returning Exhaustive Error Feedback*)
- **Trả về phản hồi thành công thông tin** (*Returning Informative Success Feedback*)

#### 5.2.1 Yêu Cầu Đầu Vào Đơn Giản (*Requesting Straightforward Inputs*)

Đầu vào (input) mà API yêu cầu nên đơn giản, chỉ bao gồm các tham số cần thiết và dễ hiểu. Điều này giúp giảm nguy cơ lỗi từ phía người dùng và làm cho API dễ sử dụng hơn.

**Nguyên tắc:**
- Chỉ yêu cầu các tham số thực sự cần thiết.
- Sử dụng các tên tham số rõ ràng và kiểu dữ liệu phù hợp.
- Cung cấp tài liệu rõ ràng về ý nghĩa và định dạng của các tham số.

**Ví dụ:**
Giả sử bạn thiết kế một API để tạo một đơn hàng mới. Thay vì yêu cầu:

```json
{
  "order_id": "DH001",
  "cust_id": "KH001",
  "items": [
    { "prod_id": "SP001", "qty": 2 },
    { "prod_id": "SP002", "qty": 1 }
  ],
  "order_timestamp": "2025-06-01T10:00:00Z"
}
```

Hãy đơn giản hóa:

```json
{
  "customerId": "KH001",
  "items": [
    { "productId": "SP001", "quantity": 2 },
    { "productId": "SP002", "quantity": 1 }
  ]
}
```

**Giải thích:**
- `order_id` không cần thiết vì API có thể tự động tạo ID cho đơn hàng.
- `order_timestamp` không cần thiết vì hệ thống có thể tự động ghi lại thời gian tạo đơn hàng.
- Các tên như `customerId`, `productId`, và `quantity` rõ ràng và dễ hiểu hơn so với `cust_id`, `prod_id`, và `qty`.

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, để đặt hàng, thay vì yêu cầu:

```json
{
  "ma_don_hang": "DH001",
  "ma_khach_hang": "KH001",
  "san_pham": [
    { "ma_sp": "SP001", "so_luong": 2 },
    { "ma_sp": "SP002", "so_luong": 1 }
  ],
  "thoi_gian_dat": "2025-06-01T10:00:00Z"
}
```

Hãy dùng:

```json
{
  "customerId": "KH001",
  "items": [
    { "productId": "SP001", "quantity": 2 },
    { "productId": "SP002", "quantity": 1 }
  ]
}
```

**Giải thích:**
- Loại bỏ `ma_don_hang` và `thoi_gian_dat` vì hệ thống có thể tự động xử lý.
- Sử dụng các tên như `customerId` và `items` để rõ ràng hơn và phù hợp với các ứng dụng thương mại điện tử tại Việt Nam.

#### 5.2.2 Xác Định Tất Cả Các Phản Hồi Lỗi Có Thể Xảy Ra (*Identifying All Possible Error Feedbacks*)

Một API đơn giản phải dự đoán và xác định tất cả các trường hợp lỗi có thể xảy ra, chẳng hạn như lỗi do đầu vào không hợp lệ, lỗi hệ thống, hoặc lỗi logic nghiệp vụ.

**Nguyên tắc:**
- Liệt kê tất cả các trường hợp lỗi, bao gồm:
  - **Lỗi cú pháp** (malformed request errors): Ví dụ, dữ liệu đầu vào không đúng định dạng JSON.
  - **Lỗi tham số không hợp lệ** (invalid parameter errors): Ví dụ, một tham số bắt buộc bị thiếu.
  - **Lỗi logic nghiệp vụ** (functional errors): Ví dụ, sản phẩm không tồn tại trong kho.
  - **Lỗi hệ thống** (server errors): Ví dụ, máy chủ gặp sự cố.
- Đảm bảo rằng mỗi lỗi có một mã trạng thái HTTP (HTTP status code) phù hợp và thông điệp lỗi rõ ràng.

**Ví dụ:**
Trong API đặt hàng, các lỗi có thể bao gồm:

- **400 Bad Request**: Thiếu tham số `customerId`.
- **400 Bad Request**: `quantity` nhỏ hơn 1.
- **404 Not Found**: `productId` không tồn tại.
- **500 Internal Server Error**: Máy chủ gặp sự cố khi xử lý đơn hàng.

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, các lỗi có thể là:

- **400 Bad Request**: Thiếu `customerId`.
- **400 Bad Request**: `quantity` của sản phẩm nhỏ hơn 1.
- **404 Not Found**: Sản phẩm với `productId` không có trong hệ thống.
- **503 Service Unavailable**: Hệ thống đang bảo trì.

#### 5.2.3 Trả Về Phản Hồi Lỗi Thông Tin (*Returning Informative Error Feedback*)

Phản hồi lỗi phải cung cấp thông tin chi tiết để nhà phát triển hiểu được nguyên nhân lỗi và cách khắc phục.

**Nguyên tắc:**
- Sử dụng mã trạng thái HTTP phù hợp (ví dụ, 400 cho lỗi đầu vào, 404 cho tài nguyên không tìm thấy).
- Bao gồm một thông điệp lỗi rõ ràng và cụ thể.
- Nếu có thể, cung cấp thêm thông tin về các trường dữ liệu gây lỗi.

**Ví dụ:**
Thay vì trả về phản hồi lỗi chung chung:

```json
{
  "error": "Invalid request"
}
```

Hãy trả về:

```json
{
  "error": {
    "code": "INVALID_QUANTITY",
    "message": "Số lượng sản phẩm phải lớn hơn 0.",
    "details": {
      "productId": "SP001",
      "quantity": 0
    }
  }
}
```

**Giải thích:**
- `code` cung cấp một mã lỗi cụ thể để nhà phát triển có thể xử lý bằng mã lập trình.
- `message` giải thích rõ ràng vấn đề.
- `details` chỉ ra chính xác trường dữ liệu nào gây lỗi.

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, nếu khách hàng nhập số lượng không hợp lệ:

Thay vì:

```json
{
  "error": "Yêu cầu không hợp lệ"
}
```

Hãy dùng:

```json
{
  "error": {
    "code": "SO_LUONG_KHONG_HOP_LE",
    "message": "Số lượng sản phẩm phải lớn hơn 0.",
    "details": {
      "productId": "SP001",
      "quantity": 0
    }
  }
}
```

**Giải thích:**
- Phản hồi này giúp nhà phát triển tại Việt Nam hiểu rõ rằng số lượng sản phẩm là vấn đề và có thể sửa lỗi nhanh chóng.

#### 5.2.4 Trả Về Phản Hồi Lỗi Đầy Đủ (*Returning Exhaustive Error Feedback*)

Phản hồi lỗi phải bao gồm tất cả các lỗi xảy ra trong một yêu cầu, thay vì chỉ trả về lỗi đầu tiên. Điều này giúp nhà phát triển sửa tất cả các vấn đề cùng một lúc.

**Nguyên tắc:**
- Nếu có nhiều lỗi trong một yêu cầu, trả về tất cả chúng trong một phản hồi.
- Sử dụng cấu trúc dữ liệu (như mảng) để liệt kê các lỗi.

**Ví dụ:**
Giả sử một yêu cầu đặt hàng có nhiều lỗi:

Thay vì:

```json
{
  "error": "Thiếu customerId"
}
```

Hãy trả về:

```json
{
  "errors": [
    {
      "code": "MISSING_CUSTOMER_ID",
      "message": "Thiếu tham số customerId."
    },
    {
      "code": "INVALID_QUANTITY",
      "message": "Số lượng sản phẩm phải lớn hơn 0.",
      "details": {
        "productId": "SP001",
        "quantity": 0
      }
    }
  ]
}
```

**Giải thích:**
- Mảng `errors` liệt kê tất cả các lỗi, giúp nhà phát triển sửa cả `customerId` và `quantity` trong một lần.

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, nếu yêu cầu có nhiều lỗi:

Thay vì:

```json
{
  "error": "Thiếu mã khách hàng"
}
```

Hãy dùng:

```json
{
  "errors": [
    {
      "code": "THIEU_MA_KHACH_HANG",
      "message": "Thiếu tham số customerId."
    },
    {
      "code": "SO_LUONG_KHONG_HOP_LE",
      "message": "Số lượng sản phẩm phải lớn hơn 0.",
      "details": {
        "productId": "SP001",
        "quantity": 0
      }
    }
  ]
}
```

**Giải thích:**
- Phản hồi này giúp nhà phát triển tại Việt Nam sửa tất cả các lỗi trong một lần, tiết kiệm thời gian và công sức.

#### 5.2.5 Trả Về Phản Hồi Thành Công Thông Tin (*Returning Informative Success Feedback*)

Phản hồi thành công (success feedback) cũng cần cung cấp thông tin hữu ích, chẳng hạn như dữ liệu đã được tạo hoặc cập nhật, để nhà phát triển biết yêu cầu đã được xử lý như thế nào.

**Nguyên tắc:**
- Trả về dữ liệu liên quan đến yêu cầu, chẳng hạn như bản ghi vừa được tạo.
- Sử dụng mã trạng thái HTTP phù hợp (ví dụ, 201 Created cho việc tạo tài nguyên mới).
- Bao gồm các thông tin bổ sung nếu cần, như ID của tài nguyên mới.

**Ví dụ:**
Khi tạo một đơn hàng mới, thay vì chỉ trả về:

```json
{
  "message": "Đơn hàng đã được tạo."
}
```

Hãy trả về:

```json
{
  "order": {
    "orderId": "DH001",
    "customerId": "KH001",
    "items": [
      { "productId": "SP001", "quantity": 2 },
      { "productId": "SP002", "quantity": 1 }
    ],
    "orderDate": "2025-06-01T10:00:00Z",
    "totalAmount": 130000
  }
}
```

**Giải thích:**
- Phản hồi bao gồm toàn bộ thông tin về đơn hàng vừa được tạo, giúp nhà phát triển xác nhận rằng yêu cầu đã thành công và sử dụng dữ liệu ngay lập tức.

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, khi tạo đơn hàng:

Thay vì:

```json
{
  "message": "Đơn hàng đã được tạo thành công."
}
```

Hãy dùng:

```json
{
  "order": {
    "orderId": "DH001",
    "customerId": "KH001",
    "items": [
      { "productId": "SP001", "quantity": 2, "subtotal": 100000 },
      { "productId": "SP002", "quantity": 1, "subtotal": 30000 }
    ],
    "orderDate": "2025-06-01T10:00:00Z",
    "totalAmount": 130000,
    "currency": "VND"
  }
}
```

**Giải thích:**
- Phản hồi này cung cấp thông tin đầy đủ về đơn hàng, giúp nhà phát triển tại Việt Nam sử dụng dữ liệu để hiển thị trên giao diện người dùng hoặc xử lý các bước tiếp theo.

---

### 5.3 Thiết Kế Các Luồng Công Việc Đơn Giản (*Designing Straightforward Flows*)

Luồng công việc (flows) là chuỗi các yêu cầu API mà người dùng cần thực hiện để đạt được một mục tiêu lớn hơn. Một luồng công việc đơn giản giảm thiểu số lượng bước và đảm bảo rằng các bước dễ thực hiện.

Phần này bao gồm các khía cạnh sau:
- **Xây dựng chuỗi mục tiêu đơn giản** (*Building a Straightforward Goal Chain*)
- **Ngăn chặn lỗi** (*Preventing Errors*)
- **Tổng hợp các mục tiêu** (*Aggregating Goals*)
- **Thiết kế luồng không trạng thái** (*Designing Stateless Flows*)

#### 5.3.1 Xây Dựng Chuỗi Mục Tiêu Đơn Giản (*Building a Straightforward Goal Chain*)

Một chuỗi mục tiêu (goal chain) là tập hợp các yêu cầu API cần thiết để hoàn thành một nhiệm vụ. Chuỗi này nên có ít bước nhất có thể và các bước phải được sắp xếp logic.

**Nguyên tắc:**
- Giảm số lượng yêu cầu API cần thiết để đạt được mục tiêu.
- Đảm bảo các bước trong chuỗi được sắp xếp theo thứ tự hợp lý.
- Cung cấp thông tin trong phản hồi để hướng dẫn bước tiếp theo.

**Ví dụ:**
Giả sử bạn muốn đặt một đơn hàng. Một chuỗi mục tiêu phức tạp có thể là:

1. Gửi yêu cầu để lấy danh sách sản phẩm (`GET /products`).
2. Gửi yêu cầu để kiểm tra tồn kho (`GET /inventory/{productId}`).
3. Gửi yêu cầu để tạo đơn hàng (`POST /orders`).

Thay vào đó, bạn có thể đơn giản hóa:

1. Gửi yêu cầu để tạo đơn hàng (`POST /orders`), trong đó API tự động kiểm tra tồn kho và trả về kết quả.

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, để đặt hàng:

Thay vì:

1. `GET /san-pham` để lấy danh sách sản phẩm.
2. `GET /kho-hang/{ma-sp}` để kiểm tra số lượng tồn kho.
3. `POST /don-hang` để tạo đơn hàng.

Hãy dùng:

1. `POST /don-hang` với dữ liệu:

```json
{
  "customerId": "KH001",
  "items": [
    { "productId": "SP001", "quantity": 2 },
    { "productId": "SP002", "quantity": 1 }
  ]
}
```

API sẽ kiểm tra tồn kho và trả về phản hồi:

```json
{
  "order": {
    "orderId": "DH001",
    "status": "success",
    "totalAmount": 130000
  }
}
```

**Giải thích:**
- Chỉ cần một yêu cầu thay vì ba, giảm độ phức tạp và thời gian xử lý.

#### 5.3.2 Ngăn Chặn Lỗi (*Preventing Errors*)

API nên được thiết kế để ngăn chặn lỗi trước khi chúng xảy ra, chẳng hạn bằng cách cung cấp các tham số mặc định hoặc kiểm tra đầu vào trước khi xử lý.

**Nguyên tắc:**
- Sử dụng các giá trị mặc định hợp lý.
- Kiểm tra đầu vào ngay từ đầu để tránh lỗi sau này.
- Hướng dẫn người dùng cách tránh lỗi thông qua tài liệu hoặc phản hồi.

**Ví dụ:**
Trong API đặt hàng, nếu người dùng quên chỉ định số lượng, API có thể mặc định `quantity` là 1 thay vì trả về lỗi.

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, nếu yêu cầu là:

```json
{
  "customerId": "KH001",
  "items": [
    { "productId": "SP001" }
  ]
}
```

Thay vì trả về lỗi:

```json
{
  "error": "Thiếu số lượng sản phẩm"
}
```

API có thể mặc định `quantity` là 1 và xử lý yêu cầu:

```json
{
  "order": {
    "orderId": "DH001",
    "items": [
      { "productId": "SP001", "quantity": 1 }
    ]
  }
}
```

**Giải thích:**
- Điều này giúp ngăn chặn lỗi và làm cho API thân thiện hơn với người dùng.

#### 5.3.3 Tổng Hợp Các Mục Tiêu (*Aggregating Goals*)

Tổng hợp các mục tiêu (aggregating goals) có nghĩa là kết hợp nhiều thao tác vào một yêu cầu API duy nhất, giảm số lượng yêu cầu và cải thiện hiệu suất.

**Nguyên tắc:**
- Kết hợp các thao tác liên quan vào một endpoint duy nhất.
- Đảm bảo rằng endpoint tổng hợp vẫn dễ hiểu và không quá phức tạp.

**Ví dụ:**
Thay vì yêu cầu nhà phát triển thực hiện hai yêu cầu:

1. `POST /orders` để tạo đơn hàng.
2. `POST /payments` để thanh toán.

Bạn có thể cung cấp một endpoint tổng hợp:

```json
POST /orders-and-payments
{
  "customerId": "KH001",
  "items": [
    { "productId": "SP001", "quantity": 2 }
  ],
  "payment": {
    "method": "credit_card",
    "amount": 100000
  }
}
```

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, thay vì:

1. `POST /don-hang` để tạo đơn hàng.
2. `POST /thanh-toan` để thanh toán.

Hãy cung cấp:

```json
POST /dat-hang-va-thanh-toan
{
  "customerId": "KH001",
  "items": [
    { "productId": "SP001", "quantity": 2 }
  ],
  "payment": {
    "method": "momo",
    "amount": 100000
  }
}
```

Phản hồi:

```json
{
  "order": {
    "orderId": "DH001",
    "status": "paid",
    "totalAmount": 100000
  }
}
```

**Giải thích:**
- Endpoint này kết hợp tạo đơn hàng và thanh toán, phù hợp với các ứng dụng thương mại điện tử tại Việt Nam, nơi người dùng thường muốn hoàn tất giao dịch trong một bước.

#### 5.3.4 Thiết Kế Luồng Không Trạng Thái (*Designing Stateless Flows*)

Luồng không trạng thái (stateless flows) có nghĩa là mỗi yêu cầu API phải chứa đầy đủ thông tin cần thiết để xử lý, không phụ thuộc vào trạng thái của các yêu cầu trước đó. Điều này làm cho API dễ mở rộng và đáng tin cậy hơn.

**Nguyên tắc:**
- Mỗi yêu cầu phải độc lập, chứa tất cả thông tin cần thiết.
- Tránh lưu trữ trạng thái trên máy chủ giữa các yêu cầu, trừ khi thực sự cần thiết.
- Sử dụng các token hoặc ID để duy trì ngữ cảnh nếu cần.

**Ví dụ:**
Thay vì yêu cầu nhà phát triển thực hiện các bước tuần tự và phụ thuộc vào trạng thái máy chủ:

1. `POST /start-order` để bắt đầu đơn hàng.
2. `POST /add-item` để thêm sản phẩm (phụ thuộc vào trạng thái từ bước 1).
3. `POST /complete-order` để hoàn tất.

Hãy thiết kế một yêu cầu không trạng thái:

```json
POST /orders
{
  "customerId": "KH001",
  "items": [
    { "productId": "SP001", "quantity": 2 }
  ]
}
```

**Ví dụ thực tế bằng tiếng Việt:**
Trong API bán hàng, thay vì:

1. `POST /bat-dau-don-hang` để tạo phiên đơn hàng.
2. `POST /them-san-pham` để thêm sản phẩm.
3. `POST /hoan-tat-don-hang` để hoàn tất.

Hãy dùng:

```json
POST /don-hang
{
  "customerId": "KH001",
  "items": [
    { "productId": "SP001", "quantity": 2 }
  ]
}
```

Phản hồi:

```json
{
  "order": {
    "orderId": "DH001",
    "status": "success",
    "totalAmount": 100000
  }
}
```

**Giải thích:**
- Yêu cầu này độc lập, không cần trạng thái trước đó, phù hợp với các hệ thống phân tán tại Việt Nam, nơi tính mở rộng là quan trọng.

---

### Tổng Kết

Chương 5 của *The Design of Web APIs* nhấn mạnh tầm quan trọng của việc thiết kế một API đơn giản, dễ hiểu và dễ sử dụng. Bằng cách tập trung vào:

- **Biểu diễn đơn giản**: Sử dụng tên rõ ràng, kiểu dữ liệu chuẩn, và dữ liệu sẵn sàng để sử dụng.
- **Tương tác đơn giản**: Yêu cầu đầu vào tối thiểu, cung cấp phản hồi lỗi đầy đủ và thông tin, cũng như phản hồi thành công hữu ích.
- **Luồng công việc đơn giản**: Giảm số bước, ngăn chặn lỗi, tổng hợp mục tiêu, và thiết kế luồng không trạng thái.

API của bạn sẽ trở nên thân thiện hơn với nhà phát triển, giảm thời gian tích hợp và cải thiện trải nghiệm người dùng. Các ví dụ thực tế bằng tiếng Việt, như API bán hàng trực tuyến, minh họa cách áp dụng các nguyên tắc này trong bối cảnh Việt Nam, đặc biệt cho các ứng dụng thương mại điện tử hoặc quản lý đơn hàng.

Nếu bạn cần thêm ví dụ hoặc giải thích chi tiết hơn về bất kỳ phần nào, hãy cho tôi biết!