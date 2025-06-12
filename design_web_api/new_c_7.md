### Chương 7: Thiết Kế Một API Gọn Gàng và Có Tổ Chức (Designing a Concise and Well-Organized API)

Chương này tập trung vào việc tạo ra một API dễ sử dụng bằng cách đảm bảo nó **gọn gàng** (concise) và **có tổ chức** (well-organized). Một API gọn gàng tránh sự phức tạp không cần thiết, giảm số lượng tài nguyên, hành động, và dữ liệu dư thừa, trong khi một API có tổ chức được cấu trúc rõ ràng, dễ điều hướng và dễ bảo trì. Nội dung của chương được chia thành các phần chính: **Tạo API Gọn Gàng (Creating a Concise API)**, **Tổ Chức API (Organizing an API)**, và **Cân Bằng giữa Gọn Gàng và Có Tổ Chức (Balancing Conciseness and Organization)**. Tôi sẽ giải thích từng phần với các ví dụ cụ thể.

---

#### 7.1 Tạo API Gọn Gàng (Creating a Concise API)

**Mục tiêu**: Một API gọn gàng chỉ cung cấp những gì cần thiết để đáp ứng nhu cầu của người dùng, loại bỏ các tài nguyên, hành động, hoặc dữ liệu không cần thiết. Điều này giúp giảm độ phức tạp, cải thiện hiệu suất, và làm cho API dễ hiểu hơn.

##### 7.1.1 Giảm Số Lượng Tài Nguyên (Reducing the Number of Resources)

**Giải thích**: Một API không nên tạo ra quá nhiều tài nguyên riêng lẻ cho các khái niệm có thể được gộp lại. Thay vào đó, các tài nguyên nên được thiết kế để đại diện cho các thực thể cốt lõi (core entities) của hệ thống.

- **Ví dụ về tài nguyên dư thừa**: Giả sử bạn đang thiết kế một API cho một cửa hàng trực tuyến. Nếu bạn tạo các tài nguyên riêng biệt như `/pending-orders`, `/completed-orders`, và `/cancelled-orders`, điều này có thể làm API trở nên phức tạp. Thay vào đó, bạn có thể sử dụng một tài nguyên duy nhất `/orders` và sử dụng tham số lọc (filtering) để truy vấn các trạng thái khác nhau.
  - **Ví dụ cụ thể**: Thay vì:
    ```
    GET /pending-orders
    GET /completed-orders
    ```
    Hãy sử dụng:
    ```
    GET /orders?status=pending
    GET /orders?status=completed
    ```
    Phản hồi có thể là:
    ```json
    {
      "items": [
        { "id": 1, "status": "pending", "customer_id": 101, "total": 99.99 }
      ]
    }
    ```

- **Lợi ích**: Giảm số lượng tài nguyên giúp đơn giản hóa tài liệu API, giảm công việc bảo trì, và giúp người dùng dễ dàng ghi nhớ cấu trúc API.

##### 7.1.2 Hạn Chế Các Hành Động Không Cần Thiết (Limiting Unnecessary Actions)

**Giải thích**: API chỉ nên cung cấp các hành động (operations) thực sự cần thiết, tránh thêm các hành động trùng lặp hoặc không mang lại giá trị. Điều này thường liên quan đến việc sử dụng các phương thức HTTP một cách hợp lý và tránh các hành động tùy chỉnh không cần thiết.

- **Ví dụ về hành động dư thừa**: Nếu bạn có một tài nguyên `/orders` và cung cấp các hành động như `POST /create-order`, `POST /update-order-status`, và `POST /cancel-order`, thì điều này có thể không gọn gàng. Thay vào đó, hãy sử dụng các phương thức HTTP chuẩn:
  - Tạo đơn hàng: `POST /orders`
  - Cập nhật trạng thái: `PATCH /orders/{id}`
  - Hủy đơn hàng: `DELETE /orders/{id}` hoặc `PATCH /orders/{id}` với trạng thái `cancelled`.

- **Ví dụ cụ thể**:
  - Thay vì:
    ```
    POST /create-order
    {
      "customer_id": 101,
      "total": 99.99
    }
    ```
    Sử dụng:
    ```
    POST /orders
    {
      "customer_id": 101,
      "total": 99.99
    }
    ```
    Phản hồi:
    ```json
    {
      "id": 1,
      "customer_id": 101,
      "total": 99.99,
      "status": "pending"
    }
    ```
  - Thay vì:
    ```
    POST /update-order-status
    {
      "order_id": 1,
      "status": "completed"
    }
    ```
    Sử dụng:
    ```
    PATCH /orders/1
    {
      "status": "completed"
    }
    ```

- **Lợi ích**: Giảm số lượng hành động giúp API dễ hiểu hơn và tuân thủ các nguyên tắc RESTful, đồng thời giảm khối lượng mã phía máy khách (client).

##### 7.1.3 Loại Bỏ Dữ Liệu Dư Thừa (Removing Redundant Data)

**Giải thích**: API nên chỉ trả về các trường dữ liệu cần thiết, tránh bao gồm thông tin không được yêu cầu. Điều này có thể được thực hiện thông qua cơ chế **field selection** (lựa chọn trường) hoặc thiết kế dữ liệu tối giản.

- **Ví dụ về dữ liệu dư thừa**: Nếu `GET /orders/1` trả về toàn bộ thông tin về đơn hàng, bao gồm cả thông tin chi tiết của khách hàng, ngay cả khi người dùng chỉ cần ID và tổng giá trị, thì điều này là không cần thiết.
  - **Phản hồi không gọn gàng**:
    ```json
    {
      "id": 1,
      "customer": {
        "id": 101,
        "name": "Nguyen Van A",
        "email": "a@example.com",
        "address": "123 Hanoi"
      },
      "total": 99.99,
      "status": "pending",
      "created_at": "2025-06-11"
    }
    ```
  - **Phản hồi gọn gàng**:
    ```json
    {
      "id": 1,
      "customer_id": 101,
      "total": 99.99
    }
    ```

- **Field Selection**: Cho phép người dùng chỉ định các trường họ muốn thông qua tham số `fields`.
  - **Ví dụ cụ thể**:
    ```
    GET /orders/1?fields=id,total
    ```
    Phản hồi:
    ```json
    {
      "id": 1,
      "total": 99.99
    }
    ```

- **Lợi ích**: Giảm dữ liệu trả về giúp cải thiện hiệu suất (ít băng thông hơn) và làm cho phản hồi dễ đọc hơn.

---

#### 7.2 Tổ Chức API (Organizing an API)

**Mục tiêu**: Một API có tổ chức được cấu trúc rõ ràng, dễ điều hướng, và dễ bảo trì. Điều này bao gồm việc sắp xếp tài nguyên, sử dụng phiên bản hóa (versioning), và cung cấp tài liệu rõ ràng.

##### 7.2.1 Sắp Xếp Tài Nguyên Theo Hệ Thống Phân Cấp (Organizing Resources Hierarchically)

**Giải thích**: Tài nguyên nên được sắp xếp theo một hệ thống phân cấp hợp lý, phản ánh mối quan hệ giữa các thực thể. Điều này giúp người dùng dễ dàng tìm thấy tài nguyên và hiểu cách chúng liên kết với nhau.

- **Ví dụ về hệ thống phân cấp**: Trong một API cửa hàng, bạn có thể tổ chức tài nguyên như sau:
  - `/customers`: Quản lý khách hàng.
  - `/customers/{id}/orders`: Quản lý đơn hàng của một khách hàng cụ thể.
  - `/orders`: Quản lý tất cả đơn hàng.
  - `/orders/{id}/items`: Quản lý các mục trong một đơn hàng.

- **Ví dụ cụ thể**:
  - Để lấy tất cả đơn hàng của khách hàng có ID 101:
    ```
    GET /customers/101/orders
    ```
    Phản hồi:
    ```json
    {
      "items": [
        { "id": 1, "total": 99.99, "status": "pending" },
        { "id": 2, "total": 149.99, "status": "completed" }
      ]
    }
    ```
  - Để lấy các mục trong đơn hàng số 1:
    ```
    GET /orders/1/items
    ```
    Phản hồi:
    ```json
    {
      "items": [
        { "product_id": 501, "quantity": 2, "price": 49.99 }
      ]
    }
    ```

- **Lợi ích**: Hệ thống phân cấp rõ ràng giúp người dùng dự đoán được đường dẫn tài nguyên và hiểu mối quan hệ giữa chúng.

##### 7.2.2 Sử Dụng Phiên Bản Hóa (Versioning the API)

**Giải thích**: Phiên bản hóa giúp quản lý các thay đổi trong API mà không làm gián đoạn người dùng hiện tại. Một API có tổ chức thường bao gồm phiên bản hóa trong đường dẫn hoặc tiêu đề để đảm bảo khả năng tương thích ngược (backward compatibility).

- **Cách phiên bản hóa**:
  - **Trong đường dẫn**: Bao gồm phiên bản trong URL, ví dụ `/v1/orders`.
  - **Trong tiêu đề**: Sử dụng tiêu đề `Accept` với phiên bản, ví dụ `Accept: application/vnd.example.v1+json`.

- **Ví dụ cụ thể**:
  - Phiên bản 1 của API:
    ```
    GET /v1/orders
    ```
    Phản hồi:
    ```json
    {
      "items": [
        { "id": 1, "customer_id": 101, "total": 99.99 }
      ]
    }
    ```
  - Phiên bản 2 (với trường mới `order_date`):
    ```
    GET /v2/orders
    ```
    Phản hồi:
    ```json
    {
      "items": [
        { "id": 1, "customer_id": 101, "total": 99.99, "order_date": "2025-06-11" }
      ]
    }
    ```

- **Lợi ích**: Phiên bản hóa giúp API phát triển mà không phá vỡ các ứng dụng khách hiện có, đồng thời giữ cho API có tổ chức khi thêm các tính năng mới.

##### 7.2.3 Cung Cấp Tài Liệu Rõ Ràng (Providing Clear Documentation)

**Giải thích**: Tài liệu là một phần quan trọng của một API có tổ chức. Tài liệu nên mô tả rõ ràng các tài nguyên, hành động, tham số, và phản hồi, đồng thời cung cấp các ví dụ thực tế.

- **Nội dung tài liệu**:
  - Danh sách tài nguyên và đường dẫn (endpoints).
  - Các phương thức HTTP được hỗ trợ.
  - Tham số yêu cầu (query parameters, body).
  - Cấu trúc phản hồi và mã trạng thái.
  - Ví dụ yêu cầu/phản hồi.

- **Ví dụ cụ thể**:
  Tài liệu cho tài nguyên `/orders`:
  ```
  ## GET /v1/orders
  Lấy danh sách đơn hàng.

  ### Tham số
  - status (tùy chọn): Lọc theo trạng thái (pending, completed, cancelled).
  - page (tùy chọn): Số trang, mặc định là 1.
  - page_size (tùy chọn): Số mục mỗi trang, mặc định là 50.

  ### Phản hồi
  - 200 OK: Danh sách đơn hàng.
  - 400 Bad Request: Tham số không hợp lệ.

  ### Ví dụ
  **Yêu cầu**:
  ```
  GET /v1/orders?status=pending&page=1&page_size=10
  ```

  **Phản hồi**:
  ```json
  {
    "items": [
      { "id": 1, "customer_id": 101, "total": 99.99, "status": "pending" }
    ],
    "meta": {
      "page": 1,
      "page_size": 10,
      "total": 1
    }
  }
  ```
  ```

- **Lợi ích**: Tài liệu rõ ràng giúp người dùng nhanh chóng hiểu và sử dụng API, giảm số lượng câu hỏi hỗ trợ.

---

#### 7.3 Cân Bằng giữa Gọn Gàng và Có Tổ Chức (Balancing Conciseness and Organization)

**Mục tiêu**: Một API lý tưởng phải đạt được sự cân bằng giữa việc giữ cho nó gọn gàng (ít tài nguyên, hành động, dữ liệu) và có tổ chức (dễ điều hướng, bảo trì). Quá tập trung vào một khía cạnh có thể làm tổn hại đến khía cạnh kia.

##### 7.3.1 Tránh Gọn Gàng Quá Mức (Avoiding Over-Conciseness)

**Giải thích**: Nếu API quá gọn gàng, nó có thể trở nên khó hiểu hoặc thiếu tính linh hoạt. Ví dụ, gộp tất cả các hành động vào một tài nguyên duy nhất có thể làm mất đi sự rõ ràng.

- **Ví dụ về gọn gàng quá mức**: Thay vì có các tài nguyên riêng biệt như `/orders` và `/invoices`, bạn gộp tất cả vào `/documents` với một tham số `type`:
  ```
  GET /documents?type=order
  GET /documents?type=invoice
  ```
  Điều này có thể làm cho API khó hiểu, vì các tài nguyên khác nhau (`orders`, `invoices`) có ngữ nghĩa và thuộc tính khác nhau.

- **Cách cân bằng**: Duy trì các tài nguyên riêng biệt khi chúng đại diện cho các thực thể khác nhau, nhưng sử dụng các tham số để xử lý các biến thể nhỏ.
  - **Ví dụ cụ thể**:
    Thay vì `/documents`, sử dụng:
    ```
    GET /orders
    GET /invoices
    ```
    Nhưng cho phép lọc trong từng tài nguyên:
    ```
    GET /orders?status=pending
    ```

##### 7.3.2 Tránh Tổ Chức Quá Phức Tạp (Avoiding Over-Organization)

**Giải thích**: Một API có quá nhiều lớp tổ chức (như quá nhiều tài nguyên con hoặc phiên bản) có thể trở nên phức tạp và khó sử dụng.

- **Ví dụ về tổ chức quá mức**: Nếu bạn tạo một hệ thống phân cấp quá sâu:
  ```
  GET /customers/{customer_id}/orders/{order_id}/items/{item_id}/details
  ```
  Điều này có thể làm cho API khó điều hướng. Thay vào đó, hãy giữ hệ thống phân cấp ở mức hợp lý:
  ```
  GET /orders/{order_id}/items
  ```

- **Cách cân bằng**: Giữ hệ thống phân cấp tài nguyên ở mức 2-3 cấp độ và sử dụng các tham số để xử lý các chi tiết bổ sung.
  - **Ví dụ cụ thể**:
    Thay vì:
    ```
    GET /customers/101/orders/1/items/501/details
    ```
    Sử dụng:
    ```
    GET /orders/1/items?product_id=501
    ```

##### 7.3.3 Sử Dụng Phản Hồi Từ Người Dùng (Using Feedback from Users)

**Giải thích**: Để đạt được sự cân bằng, hãy thu thập phản hồi từ người dùng API (nhà phát triển, ứng dụng khách) để hiểu những gì họ thấy khó khăn hoặc thiếu rõ ràng. Điều này giúp tinh chỉnh thiết kế API.

- **Ví dụ cụ thể**: Giả sử người dùng phàn nàn rằng tài nguyên `/orders` trả về quá nhiều dữ liệu, làm chậm ứng dụng của họ. Bạn có thể thêm hỗ trợ `fields`:
  ```
  GET /orders/1?fields=id,total
  ```
  Hoặc nếu họ thấy hệ thống phân cấp `/customers/{id}/orders` khó sử dụng, bạn có thể cung cấp một đường dẫn thay thế:
  ```
  GET /orders?customer_id=101
  ```

- **Lợi ích**: Phản hồi giúp điều chỉnh API để vừa gọn gàng vừa có tổ chức, đáp ứng nhu cầu thực tế.

---

### Tổng Kết Chương 7

Chương 7 nhấn mạnh rằng một API gọn gàng và có tổ chức là yếu tố then chốt để cải thiện trải nghiệm người dùng và khả năng bảo trì. Các điểm chính bao gồm:
- **Tạo API Gọn Gàng**: Giảm số lượng tài nguyên, hành động, và dữ liệu không cần thiết bằng cách sử dụng lọc, phương thức HTTP chuẩn, và field selection.
- **Tổ Chức API**: Sắp xếp tài nguyên theo hệ thống phân cấp hợp lý, sử dụng phiên bản hóa, và cung cấp tài liệu rõ ràng.
- **Cân Bằng**: Tránh gọn gàng quá mức (làm mất rõ ràng) hoặc tổ chức quá phức tạp (làm khó sử dụng), và sử dụng phản hồi để tinh chỉnh.

**Ví dụ tổng hợp**: Một API cửa hàng có thể được thiết kế như sau:
- **Gọn gàng**:
  - Tài nguyên: `/orders`, `/customers`, `/products` (thay vì `/pending-orders`, `/completed-orders`).
  - Hành động: `POST /orders`, `PATCH /orders/{id}` (thay vì `POST /create-order`, `POST /update-order-status`).
  - Dữ liệu: Hỗ trợ `GET /orders/1?fields=id,total`.
- **Có tổ chức**:
  - Hệ thống phân cấp: `/customers/{id}/orders`, `/orders/{id}/items`.
  - Phiên bản hóa: `/v1/orders`.
  - Tài liệu:
    ```
    ## POST /v1/orders
    Tạo đơn hàng mới.

    ### Body
    ```json
    {
      "customer_id": 101,
      "total": 99.99
    }
    ```

    ### Phản hồi
    - 201 Created: Đơn hàng được tạo.
    ```json
    {
      "id": 1,
      "customer_id": 101,
      "total": 99.99,
      "status": "pending"
    }
    ```
    ```
- **Cân bằng**: Sử dụng `/orders?status=pending` thay vì `/pending-orders`, nhưng giữ `/invoices` riêng nếu nó có thuộc tính khác biệt.

Hy vọng nội dung chi tiết này giúp bạn hiểu rõ cách thiết kế một API gọn gàng và có tổ chức theo chương 7 của *The Design of Web APIs*. Nếu bạn cần thêm ví dụ, giải thích sâu hơn về một phần cụ thể, hoặc muốn tôi tạo một mindmap cho chương này, hãy cho tôi biết!