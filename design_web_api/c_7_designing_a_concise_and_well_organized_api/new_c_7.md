# Chương 7: Thiết Kế Một API Gọn Gàng và Có Tổ Chức Tốt

Chương 7 tập trung vào việc làm thế nào để thiết kế một API không chỉ dễ hiểu mà còn dễ sử dụng thông qua việc tổ chức và định cỡ các thành phần của API một cách hợp lý. Một API được tổ chức tốt sẽ giúp người dùng (nhà phát triển hoặc phần mềm tiêu thụ API) nhanh chóng nắm bắt cách hoạt động của nó và sử dụng nó một cách hiệu quả. Chương này được chia thành hai phần chính: **Tổ chức API** và **Định cỡ API**. Mỗi phần sẽ được giải thích chi tiết dưới đây, kèm theo ví dụ minh họa.

---

## 7.1 Tổ Chức Một API

Tổ chức API liên quan đến việc sắp xếp các thành phần của API (dữ liệu, phản hồi, và mục tiêu) sao cho chúng logic và dễ tiếp cận. Điều này giống như việc sắp xếp một ngôi nhà: mọi thứ cần được đặt ở đúng vị trí để người sử dụng có thể tìm thấy chúng mà không gặp khó khăn. Phần này chia thành ba khía cạnh: tổ chức dữ liệu, tổ chức phản hồi, và tổ chức mục tiêu.

### 7.1.1 Tổ Chức Dữ Liệu

**Tổ chức dữ liệu** là cách bạn cấu trúc và trình bày dữ liệu trong API để người dùng dễ dàng hiểu và sử dụng. Một API cần sử dụng các mô hình dữ liệu nhất quán, đặt tên rõ ràng và có cấu trúc hợp lý để tránh gây nhầm lẫn.

#### Nguyên Tắc Tổ Chức Dữ Liệu

1. **Nhóm Các Thuộc Tính Liên Quan**: Các thuộc tính có liên quan nên được nhóm lại với nhau để phản ánh mối quan hệ của chúng. Ví dụ, trong một API quản lý thông tin khách hàng, các thuộc tính như `firstName`, `lastName`, và `email` nên được nhóm trong một đối tượng `personalInfo` thay vì để riêng lẻ.

2. **Đặt Tên Rõ Ràng và Nhất Quán**: Tên của các thuộc tính nên phản ánh đúng nội dung của chúng và tuân theo một quy ước đặt tên nhất quán trong toàn bộ API. Ví dụ, sử dụng `camelCase` hoặc `snake_case` một cách thống nhất.

3. **Sử Dụng Cấu Trúc Phân Cấp Khi Cần Thiết**: Nếu dữ liệu phức tạp, hãy sử dụng các đối tượng lồng nhau để biểu diễn mối quan hệ. Tuy nhiên, cần tránh lồng quá sâu để không làm tăng độ phức tạp.

4. **Loại Bỏ Dữ Liệu Không Cần Thiết**: Chỉ bao gồm những thuộc tính thực sự cần thiết cho mục tiêu của API. Điều này giúp giảm kích thước payload và tăng hiệu suất.

#### Ví Dụ: Tổ Chức Dữ Liệu

Hãy xem xét một API để lấy thông tin về một đơn hàng. Dưới đây là một ví dụ về dữ liệu được tổ chức kém:

```json
{
  "orderId": "12345",
  "customerName": "Nguyen Van A",
  "customerEmail": "a.nguyen@example.com",
  "item1Name": "Laptop",
  "item1Price": 1000,
  "item2Name": "Mouse",
  "item2Price": 50,
  "orderDate": "2025-06-01"
}
```

Vấn đề với cấu trúc này:
- Các thuộc tính liên quan đến khách hàng (`customerName`, `customerEmail`) và sản phẩm (`item1Name`, `item1Price`, `item2Name`, `item2Price`) không được nhóm lại, làm cho dữ liệu khó đọc.
- Không có cấu trúc rõ ràng để biểu diễn danh sách các sản phẩm.

Dưới đây là phiên bản được tổ chức tốt hơn:

```json
{
  "orderId": "12345",
  "orderDate": "2025-06-01",
  "customer": {
    "name": "Nguyen Van A",
    "email": "a.nguyen@example.com"
  },
  "items": [
    {
      "name": "Laptop",
      "price": 1000
    },
    {
      "name": "Mouse",
      "price": 50
    }
  ]
}
```

Lợi ích của cấu trúc này:
- **Nhóm liên quan**: Thuộc tính của khách hàng được nhóm trong đối tượng `customer`, và các sản phẩm được nhóm trong mảng `items`.
- **Tính mở rộng**: Dễ dàng thêm các sản phẩm khác vào mảng `items` mà không cần thay đổi cấu trúc.
- **Rõ ràng**: Người dùng có thể dễ dàng hiểu mối quan hệ giữa các thuộc tính.

#### Ví Dụ Thực Tế (Bối Cảnh Việt Nam)

Giả sử bạn đang thiết kế một API cho một ứng dụng thương mại điện tử ở Việt Nam, nơi người dùng có thể xem thông tin đơn hàng bao gồm địa chỉ giao hàng. Một cấu trúc dữ liệu tốt có thể trông như sau:

```json
{
  "orderId": "DH001",
  "orderDate": "2025-06-01",
  "customer": {
    "fullName": "Trần Thị Bích",
    "phoneNumber": "0901234567",
    "email": "bich.tran@example.com"
  },
  "deliveryAddress": {
    "street": "123 Đường Láng",
    "ward": "Phường Láng Thượng",
    "district": "Quận Đống Đa",
    "city": "Hà Nội"
  },
  "items": [
    {
      "productId": "SP001",
      "name": "Áo Thun",
      "quantity": 2,
      "unitPrice": 150000
    },
    {
      "productId": "SP002",
      "name": "Quần Jean",
      "quantity": 1,
      "unitPrice": 500000
    }
  ],
  "totalAmount": 800000
}
```

Cấu trúc này:
- Nhóm các thông tin địa chỉ giao hàng vào đối tượng `deliveryAddress`, phản ánh đúng cách tổ chức địa chỉ ở Việt Nam (phường, quận, thành phố).
- Sử dụng đơn vị tiền tệ VND và đặt tên thuộc tính rõ ràng (`unitPrice`, `totalAmount`).
- Dễ dàng mở rộng để thêm các thuộc tính khác, như mã giảm giá hoặc phí vận chuyển.

---

### 7.1.2 Tổ Chức Phản Hồi

**Phản hồi** (feedback) bao gồm các phản hồi thành công (success feedback) và phản hồi lỗi (error feedback). Tổ chức phản hồi tốt đảm bảo rằng người dùng có thể dễ dàng xử lý các phản hồi từ API, bất kể kết quả là thành công hay thất bại.

#### Nguyên Tắc Tổ Chức Phản Hồi

1. **Phản Hồi Lỗi Nhất Quán**: Tất cả các lỗi nên được trả về theo một định dạng chung, bao gồm mã lỗi, thông báo, và chi tiết bổ sung nếu cần.

2. **Nhóm Các Lỗi Liên Quan**: Nếu một yêu cầu tạo ra nhiều lỗi (ví dụ, xác thực nhiều trường dữ liệu), hãy nhóm các lỗi này trong một danh sách để người dùng dễ xử lý.

3. **Phản Hồi Thành Công Rõ Ràng**: Phản hồi thành công nên bao gồm dữ liệu chính và bất kỳ siêu dữ liệu nào (metadata) cần thiết, như tổng số mục hoặc liên kết phân trang.

4. **Sử Dụng Mã Trạng Thái HTTP Phù Hợp**: Mã trạng thái HTTP (như 200 OK, 400 Bad Request) cần phản ánh đúng trạng thái của yêu cầu.

#### Ví Dụ: Tổ Chức Phản Hồi Lỗi

Giả sử bạn gửi một yêu cầu POST để tạo một đơn hàng, nhưng có lỗi trong dữ liệu đầu vào. Một phản hồi lỗi được tổ chức kém có thể trông như sau:

```json
{
  "error": "Invalid request"
}
```

Phản hồi này không cung cấp đủ thông tin để người dùng biết vấn đề nằm ở đâu. Một phản hồi lỗi được tổ chức tốt hơn sẽ là:

```json
{
  "status": 400,
  "error": {
    "code": "INVALID_INPUT",
    "message": "Dữ liệu đầu vào không hợp lệ",
    "details": [
      {
        "field": "customer.email",
        "issue": "Địa chỉ email không đúng định dạng"
      },
      {
        "field": "items[0].quantity",
        "issue": "Số lượng phải lớn hơn 0"
      }
    ]
  }
}
```

Lợi ích:
- **Cụ thể**: Chỉ rõ các trường bị lỗi (`customer.email`, `items[0].quantity`) và lý do.
- **Nhóm lỗi**: Tất cả lỗi được liệt kê trong mảng `details`, giúp xử lý từng lỗi một cách dễ dàng.
- **Mã trạng thái**: Sử dụng `400 Bad Request` để báo hiệu lỗi đầu vào.

#### Ví Dụ: Phản Hồi Thành Công

Đối với yêu cầu `GET /orders/DH001`, phản hồi thành công có thể là:

```json
{
  "status": 200,
  "data": {
    "orderId": "DH001",
    "orderDate": "2025-06-01",
    "customer": {
      "name": "Trần Thị Bích",
      "email": "bich.tran@example.com"
    },
    "items": [
      {
        "name": "Áo Thun",
        "quantity": 2,
        "unitPrice": 150000
      }
    ],
    "totalAmount": 300000
  },
  "meta": {
    "timestamp": "2025-06-01T15:00:00Z",
    "links": {
      "self": "/api/v1/orders/DH001",
      "cancel": "/api/v1/orders/DH001/cancel"
    }
  }
}
```

Cấu trúc này:
- **Dữ liệu chính**: Được đặt trong nhóm `data`.
- **Siêu dữ liệu**: Trong `meta`, cung cấp thông tin bổ sung như thời gian và các liên kết để thực hiện hành động khác (hủy đơn hàng).
- **Mã trạng thái**: Sử dụng `200 OK`.

---

### 7.1.3 Tổ Chức Mục Tiêu

**Mục tiêu** (goals) là các hành động mà API cho phép người dùng thực hiện, ví dụ: tạo đơn hàng, cập nhật thông tin khách hàng, hoặc xóa sản phẩm. Tổ chức mục tiêu tốt có nghĩa là nhóm chúng theo cách logic và dễ tìm, giúp người dùng nhanh chóng xác định hành động cần thực hiện.

#### Nguyên Tắc Tổ Chức Mục Tiêu

1. **Nhóm Theo Tài Nguyên**: Các mục tiêu liên quan đến cùng một tài nguyên (resource) nên được nhóm lại. Ví dụ, tất cả các hành động liên quan đến đơn hàng (tạo, xem, cập nhật, xóa) nên được đặt dưới cùng một đường dẫn cơ sở (`/orders`).

2. **Sử Dụng Đường Dẫn Rõ Ràng**: Đường dẫn (path) nên phản ánh cấu trúc tài nguyên, ví dụ: `/orders/{orderId}/items` để quản lý các mục trong một đơn hàng.

3. **Tách Biệt Các Mục Tiêu Theo Ý Nghĩa**: Các mục tiêu có ý nghĩa khác nhau nên được tách biệt rõ ràng, ngay cả khi chúng liên quan đến cùng một tài nguyên. Ví dụ, tạo một đơn hàng và hủy một đơn hàng là hai mục tiêu riêng biệt.

4. **Sử Dụng Siêu Dữ Liệu (Hypermedia)**: Cung cấp các liên kết trong phản hồi để hướng dẫn người dùng đến các mục tiêu tiếp theo.

#### Ví Dụ: Tổ Chức Mục Tiêu

Giả sử bạn có một API thương mại điện tử với các mục tiêu liên quan đến đơn hàng và khách hàng. Một cách tổ chức kém có thể là:

```
POST /createOrder
GET /getOrderDetails
DELETE /deleteOrder
PUT /updateCustomerInfo
GET /getCustomerInfo
```

Vấn đề:
- Không có cấu trúc rõ ràng, các mục tiêu không được nhóm theo tài nguyên.
- Đường dẫn không phản ánh mối quan hệ giữa các tài nguyên.

Một cách tổ chức tốt hơn:

```
POST /orders
GET /orders/{orderId}
PUT /orders/{orderId}
DELETE /orders/{orderId}
POST /orders/{orderId}/cancel
GET /customers/{customerId}
PUT /customers/{customerId}
```

Lợi ích:
- **Nhóm theo tài nguyên**: Tất cả các mục tiêu liên quan đến đơn hàng được đặt dưới `/orders`, và khách hàng dưới `/customers`.
- **Rõ ràng**: Đường dẫn như `/orders/{orderId}/cancel` chỉ rõ hành động hủy thuộc về một đơn hàng cụ thể.
- **Nhất quán**: Sử dụng các phương thức HTTP chuẩn (POST, GET, PUT, DELETE).

#### Ví Dụ Thực Tế (Bối Cảnh Việt Nam)

Trong một ứng dụng giao hàng thực phẩm ở Việt Nam, bạn có thể tổ chức các mục tiêu như sau:

```
POST /orders
GET /orders/{orderId}
PUT /orders/{orderId}
DELETE /orders/{orderId}
POST /orders/{orderId}/cancel
GET /restaurants
GET /restaurants/{restaurantId}/menu
POST /orders/{orderId}/track
```

- **Tài nguyên**: Đơn hàng (`/orders`), nhà hàng (`/restaurants`), và thực đơn (`/menu`).
- **Mục tiêu cụ thể**: `POST /orders/{orderId}/track` cho phép theo dõi trạng thái giao hàng, một tính năng phổ biến ở Việt Nam.
- **Hypermedia**: Phản hồi từ `GET /orders/{orderId}` có thể bao gồm liên kết đến `/orders/{orderId}/track` hoặc `/orders/{orderId}/cancel`.

---

## 7.2 Định Cỡ Một API

Định cỡ API liên quan đến việc quyết định mức độ chi tiết (granularity) của dữ liệu, mục tiêu, và toàn bộ API. Một API được định cỡ tốt sẽ cân bằng giữa tính đơn giản và tính linh hoạt, đảm bảo rằng nó không quá phức tạp nhưng vẫn đáp ứng được nhu cầu của người dùng.

### 7.2.1 Lựa Chọn Độ Chi Tiết Của Dữ Liệu

**Độ chi tiết của dữ liệu** (data granularity) là mức độ chi tiết của thông tin được trả về hoặc yêu cầu trong một API. Dữ liệu có thể được thiết kế ở mức độ chi tiết cao (fine-grained) hoặc thô (coarse-grained).

#### Nguyên Tắc Lựa Chọn Độ Chi Tiết Dữ Liệu

1. **Cân Bằng Hiệu Suất và Tiện Dụng**: Dữ liệu chi tiết cao cho phép người dùng chỉ lấy những gì họ cần, nhưng có thể yêu cầu nhiều yêu cầu API hơn, làm giảm hiệu suất. Dữ liệu thô giảm số lượng yêu cầu nhưng có thể trả về dữ liệu không cần thiết.

2. **Phù Hợp Với Nhu Cầu Người Dùng**: Xác định xem người dùng cần toàn bộ dữ liệu hay chỉ một phần nhỏ. Ví dụ, một ứng dụng di động có thể chỉ cần thông tin cơ bản, trong khi một hệ thống backend cần dữ liệu đầy đủ.

3. **Hỗ Trợ Lọc và Mở Rộng**: Cung cấp các tham số để lọc hoặc mở rộng dữ liệu, cho phép người dùng kiểm soát độ chi tiết.

#### Ví Dụ: Độ Chi Tiết Dữ Liệu

Giả sử bạn có một API để lấy thông tin khách hàng. Một API chi tiết cao có thể chia nhỏ dữ liệu thành nhiều điểm cuối:

```
GET /customers/{customerId}/personalInfo
GET /customers/{customerId}/orders
GET /customers/{customerId}/preferences
```

Một API thô hơn có thể trả về tất cả thông tin trong một yêu cầu:

```
GET /customers/{customerId}
```

Phản hồi:

```json
{
  "customerId": "C001",
  "personalInfo": {
    "name": "Nguyen Van A",
    "email": "a.nguyen@example.com"
  },
  "orders": [
    {
      "orderId": "DH001",
      "totalAmount": 300000
    }
  ],
  "preferences": {
    "language": "vi",
    "notifications": true
  }
}
```

Để cân bằng, bạn có thể cung cấp một API thô với khả năng lọc:

```
GET /customers/{customerId}?fields=personalInfo,orders
```

Phản hồi chỉ bao gồm các trường được yêu cầu:

```json
{
  "customerId": "C001",
  "personalInfo": {
    "name": "Nguyen Van A",
    "email": "a.nguyen@example.com"
  },
  "orders": [
    {
      "orderId": "DH001",
      "totalAmount": 300000
    }
  ]
}
```

#### Ví Dụ Thực Tế (Bối Cảnh Việt Nam)

Trong một ứng dụng ví điện tử ở Việt Nam, bạn có thể thiết kế API để lấy thông tin tài khoản người dùng:

```
GET /accounts/{accountId}?fields=balance,transactions
```

Phản hồi:

```json
{
  "accountId": "TK001",
  "balance": 5000000,
  "transactions": [
    {
      "transactionId": "GD001",
      "amount": -200000,
      "description": "Thanh toán tại Shopee",
      "date": "2025-06-01"
    },
    {
      "transactionId": "GD002",
      "amount": 1000000,
      "description": "Nạp tiền từ Vietcombank",
      "date": "2025-06-02"
    }
  ]
}
```

- **Lọc**: Tham số `fields` cho phép người dùng chỉ lấy số dư (`balance`) và lịch sử giao dịch (`transactions`), phù hợp với ứng dụng di động có băng thông hạn chế.
- **Đơn vị tiền tệ**: Sử dụng VND, phù hợp với bối cảnh Việt Nam.

---

### 7.2.2 Lựa Chọn Độ Chi Tiết Của Mục Tiêu

**Độ chi tiết của mục tiêu** (goal granularity) là mức độ chi tiết của các hành động mà API cung cấp. Mục tiêu chi tiết cao chia nhỏ các hành động thành nhiều bước, trong khi mục tiêu thô kết hợp nhiều hành động vào một.

#### Nguyên Tắc Lựa Chọn Độ Chi Tiết Mục Tiêu

1. **Đáp Ứng Kịch Bản Sử Dụng**: Mục tiêu nên phù hợp với cách người dùng thực hiện công việc. Ví dụ, nếu người dùng thường cập nhật nhiều thuộc tính cùng lúc, hãy cung cấp một mục tiêu thô.

2. **Tránh Quá Nhiều Bước**: Mục tiêu chi tiết cao có thể yêu cầu nhiều yêu cầu API, làm tăng độ phức tạp và giảm hiệu suất.

3. **Cân Nhắc Tính Linh Hoạt**: Mục tiêu chi tiết cao cung cấp tính linh hoạt hơn, nhưng mục tiêu thô thường đơn giản hơn cho người dùng.

#### Ví Dụ: Độ Chi Tiết Mục Tiêu

Giả sử bạn có một API để cập nhật thông tin đơn hàng. Một thiết kế chi tiết cao có thể chia nhỏ thành:

```
PUT /orders/{orderId}/status
PUT /orders/{orderId}/deliveryAddress
PUT /orders/{orderId}/items
```

Một thiết kế thô hơn sẽ kết hợp tất cả vào một:

```
PUT /orders/{orderId}
```

Body của yêu cầu:

```json
{
  "status": "confirmed",
  "deliveryAddress": {
    "street": "123 Đường Láng",
    "city": "Hà Nội"
  },
  "items": [
    {
      "productId": "SP001",
      "quantity": 2
    }
  ]
}
```

Để cân bằng, bạn có thể cung cấp cả hai tùy chọn:
- `PUT /orders/{orderId}` cho cập nhật toàn bộ.
- `PATCH /orders/{orderId}/status` để chỉ cập nhật trạng thái.

#### Ví Dụ Thực Tế (Bối Cảnh Việt Nam)

Trong một ứng dụng đặt vé xe khách ở Việt Nam, bạn có thể thiết kế:

```
PUT /bookings/{bookingId}
PATCH /bookings/{bookingId}/seat
```

- **Thô**: `PUT /bookings/{bookingId}` cho phép cập nhật toàn bộ thông tin đặt vé (ghế, thời gian, thông tin hành khách).
- **Chi tiết**: `PATCH /bookings/{bookingId}/seat` cho phép đổi ghế mà không ảnh hưởng đến các thông tin khác, một nhu cầu phổ biến khi đặt vé xe.

---

### 7.2.3 Lựa Chọn Độ Chi Tiết Của API

**Độ chi tiết của API** (API granularity) là phạm vi chức năng mà toàn bộ API bao phủ. Một API chi tiết cao tập trung vào một tập hợp nhỏ các chức năng, trong khi một API thô cung cấp nhiều chức năng hơn.

#### Nguyên Tắc Lựa Chọn Độ Chi Tiết API

1. **Phù Hợp Với Bối Cảnh**: Một API cho một ứng dụng cụ thể (như ứng dụng di động) có thể chi tiết hơn, trong khi một API backend cần thô hơn để hỗ trợ nhiều chức năng.

2. **Cân Nhắc Tái Sử Dụng**: API chi tiết cao dễ tái sử dụng trong các ngữ cảnh khác nhau, nhưng API thô giảm số lượng API cần quản lý.

3. **Hỗ Trợ Tích Hợp**: API thô có thể đơn giản hóa việc tích hợp, nhưng API chi tiết cao cung cấp tính linh hoạt hơn.

#### Ví Dụ: Độ Chi Tiết API

Một API chi tiết cao có thể chỉ quản lý đơn hàng:

```
POST /orders
GET /orders/{orderId}
```

Một API thô hơn có thể bao gồm cả đơn hàng, khách hàng, và sản phẩm:

```
POST /orders
GET /orders/{orderId}
GET /customers/{customerId}
GET /products/{productId}
```

Để cân bằng, bạn có thể tách thành nhiều API nhỏ hơn nhưng liên quan, ví dụ:
- API Đơn Hàng: `/orders`
- API Khách Hàng: `/customers`
- API Sản Phẩm: `/products`

#### Ví Dụ Thực Tế (Bối Cảnh Việt Nam)

Trong một ứng dụng bán hàng nông sản ở Việt Nam, bạn có thể thiết kế:

- **API chi tiết**: 
  - `/products` để quản lý danh sách nông sản (gạo, rau, trái cây).
  - `/orders` để quản lý đơn hàng.
- **API thô**: Một API tổng hợp `/marketplace` bao gồm cả sản phẩm, đơn hàng, và chương trình khuyến mãi.

Ví dụ yêu cầu:

```
GET /marketplace/products?category=vegetables
```

Phản hồi:

```json
{
  "products": [
    {
      "productId": "SP001",
      "name": "Rau Muống",
      "price": 20000,
      "unit": "kg"
    },
    {
      "productId": "SP002",
      "name": "Cải Thảo",
      "price": 30000,
      "unit": "kg"
    }
  ]
}
```

---

## Kết Luận

Chương 7 nhấn mạnh rằng một API được thiết kế tốt không chỉ dừng lại ở việc cung cấp chức năng mà còn phải được tổ chức và định cỡ một cách hợp lý. Bằng cách tổ chức dữ liệu, phản hồi, và mục tiêu một cách logic, cùng với việc chọn độ chi tiết phù hợp, bạn có thể tạo ra một API dễ sử dụng và hiệu quả. Các ví dụ thực tế trong bối cảnh Việt Nam (thương mại điện tử, giao hàng thực phẩm, ví điện tử, bán nông sản) minh họa cách áp dụng các nguyên tắc này vào các tình huống thực tế.

Hãy thực hành bằng cách thiết kế một API nhỏ cho một ứng dụng quen thuộc trong bối cảnh Việt Nam (ví dụ: ứng dụng đặt lịch khám bệnh hoặc quản lý quán cà phê). Áp dụng các nguyên tắc tổ chức và định cỡ từ chương này, và kiểm tra xem API của bạn có dễ hiểu và dễ sử dụng không. Nếu bạn cần thêm hướng dẫn hoặc ví dụ cụ thể, hãy cho tôi biết!