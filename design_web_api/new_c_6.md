### Chương 6: Thiết Kế Một API Có Tính Dự Đoán (Designing a Predictable API)

Chương này tập trung vào việc làm thế nào để tạo ra một API dễ hiểu và dễ sử dụng, đến mức người dùng (cả con người lẫn máy móc) có thể đoán được cách hoạt động của API mà không cần quá nhiều tài liệu hướng dẫn. Một API có tính dự đoán là một API mà người dùng có thể sử dụng một cách trực giác, dựa trên sự nhất quán (consistency), khả năng thích ứng (adaptability), và khả năng khám phá (discoverability). Dưới đây là nội dung chi tiết của chương 6, được chia theo các phần chính: **Sự Nhất Quán (Being Consistent)**, **Khả Năng Thích Ứng (Being Adaptable)**, và **Khả Năng Khám Phá (Being Discoverable)**.

---

#### 6.1 Sự Nhất Quán (Being Consistent)

**Mục tiêu**: Một API nhất quán giúp người dùng dễ dàng hiểu và dự đoán cách hoạt động của nó. Sự nhất quán có nghĩa là các yếu tố của API (dữ liệu, mục tiêu, hành vi) được thiết kế theo cách giống nhau trên toàn bộ API, hoặc thậm chí giữa các API khác nhau trong cùng một tổ chức.

##### 6.1.1 Thiết Kế Dữ Liệu Nhất Quán (Designing Consistent Data)

**Giải thích**: Dữ liệu trong API cần được trình bày theo cách thống nhất về cách đặt tên, định dạng, và cấu trúc. Điều này giúp người dùng không phải đoán hoặc học lại cách xử lý dữ liệu ở các phần khác nhau của API.

- **Ví dụ về đặt tên**: Nếu một API sử dụng từ `customer_id` trong một tài nguyên (resource), thì nó nên sử dụng `customer_id` ở mọi nơi, thay vì sử dụng `client_id` hoặc `user_id` ở những tài nguyên khác. Sự không nhất quán trong cách đặt tên có thể gây nhầm lẫn.
  - **Ví dụ cụ thể**: Trong một API quản lý cửa hàng, nếu tài nguyên `/orders` trả về trường `customer_id`, nhưng tài nguyên `/invoices` trả về `client_id` cho cùng một khái niệm (ID của khách hàng), thì điều này sẽ làm khó người dùng. Thay vào đó, hãy sử dụng `customer_id` ở cả hai tài nguyên.
  
- **Ví dụ về định dạng**: Nếu ngày tháng được định dạng theo chuẩn ISO 8601 (`YYYY-MM-DD`) trong một phản hồi (response), thì tất cả các trường ngày tháng trong API cũng nên sử dụng định dạng này.
  - **Ví dụ cụ thể**: Nếu API trả về `created_date: "2025-06-11"` trong tài nguyên `/orders`, nhưng lại trả về `invoice_date: "06/11/2025"` trong tài nguyên `/invoices`, thì người dùng sẽ phải viết mã để xử lý hai định dạng khác nhau. Để nhất quán, cả hai trường nên sử dụng định dạng `YYYY-MM-DD`.

- **Cấu trúc dữ liệu**: Các danh sách (lists) hoặc các đối tượng (objects) nên có cấu trúc giống nhau. Ví dụ, nếu danh sách các đơn hàng trả về một mảng các đối tượng với các trường `id`, `customer_id`, và `total`, thì danh sách các hóa đơn cũng nên có cấu trúc tương tự nếu phù hợp.
  - **Ví dụ cụ thể**: Phản hồi từ `GET /orders` có thể là:
    ```json
    {
      "items": [
        { "id": 1, "customer_id": 101, "total": 99.99 },
        { "id": 2, "customer_id": 102, "total": 149.99 }
      ]
    }
    ```
    Nếu `GET /invoices` trả về:
    ```json
    {
      "invoices": [
        { "invoice_id": 1, "client_id": 101, "amount": 99.99 },
        { "invoice_id": 2, "client_id": 102, "amount": 149.99 }
      ]
    }
    ```
    thì đây là sự không nhất quán. Thay vào đó, `GET /invoices` nên trả về:
    ```json
    {
      "items": [
        { "id": 1, "customer_id": 101, "total": 99.99 },
        { "id": 2, "customer_id": 102, "total": 149.99 }
      ]
    }
    ```

**Lợi ích**: Sự nhất quán trong dữ liệu giúp giảm thời gian học hỏi và giảm lỗi khi tích hợp API.

---

##### 6.1.2 Thiết Kế Mục Tiêu Nhất Quán (Designing Consistent Goals)

**Giải thích**: Các mục tiêu (goals) của API, tức là các hành động mà API cho phép (như tạo, đọc, cập nhật, xóa), cũng cần được thiết kế nhất quán. Điều này bao gồm cách các hành động được biểu diễn (bằng các phương thức HTTP, đường dẫn tài nguyên) và cách chúng trả về phản hồi.

- **Phương thức HTTP**: Sử dụng các phương thức HTTP một cách nhất quán. Ví dụ, `GET` để lấy dữ liệu, `POST` để tạo, `PUT` hoặc `PATCH` để cập nhật, và `DELETE` để xóa.
  - **Ví dụ cụ thể**: Nếu `GET /orders/{id}` được sử dụng để lấy thông tin một đơn hàng, thì `GET /invoices/{id}` cũng nên được sử dụng để lấy thông tin một hóa đơn. Không nên sử dụng `POST /getInvoice` cho mục đích này, vì điều đó không nhất quán với chuẩn REST và làm người dùng khó đoán.

- **Phản hồi thành công**: Các phản hồi thành công (success responses) nên có cấu trúc giống nhau. Ví dụ, nếu một hành động tạo (`POST`) trả về mã trạng thái HTTP `201 Created` và dữ liệu của tài nguyên vừa tạo, thì tất cả các hành động tạo khác cũng nên làm như vậy.
  - **Ví dụ cụ thể**: Phản hồi từ `POST /orders` có thể là:
    ```json
    {
      "id": 3,
      "customer_id": 103,
      "total": 199.99
    }
    ```
    với mã trạng thái `201 Created`. Tương tự, `POST /invoices` cũng nên trả về:
    ```json
    {
      "id": 3,
      "customer_id": 103,
      "total": 199.99
    }
    ```
    với mã trạng thái `201 Created`, thay vì trả về một cấu trúc khác hoặc mã trạng thái khác như `200 OK`.

- **Phản hồi lỗi**: Các phản hồi lỗi cũng cần nhất quán về cấu trúc và mã trạng thái. Ví dụ, nếu lỗi xác thực trả về mã `400 Bad Request` với thông điệp lỗi cụ thể, thì tất cả các lỗi xác thực khác cũng nên làm như vậy.
  - **Ví dụ cụ thể**: Nếu `POST /orders` trả về lỗi:
    ```json
    {
      "error": {
        "code": "INVALID_INPUT",
        "message": "customer_id is required"
      }
    }
    ```
    với mã `400 Bad Request`, thì `POST /invoices` cũng nên trả về lỗi tương tự với cấu trúc và mã trạng thái giống nhau.

---

##### 6.1.3 Bốn Cấp Độ Nhất Quán (The Four Levels of Consistency)

**Giải thích**: Sự nhất quán có thể được áp dụng ở bốn cấp độ khác nhau:
1. **Trong chính API đó**: Các tài nguyên, hành động, và dữ liệu trong cùng một API phải nhất quán.
2. **Giữa các API trong cùng tổ chức**: Các API do một công ty hoặc tổ chức phát triển nên tuân theo các quy tắc thiết kế chung.
3. **Giữa các API trong cùng lĩnh vực**: Các API trong cùng ngành (ví dụ: thương mại điện tử, tài chính) nên tuân theo các thông lệ chung của ngành.
4. **Giữa các API nói chung**: Một số tiêu chuẩn và thông lệ chung (như chuẩn REST, mã trạng thái HTTP) được áp dụng trên toàn thế giới.

- **Ví dụ cấp độ 1**: Trong một API, tất cả các tài nguyên nên sử dụng cấu trúc danh sách với trường `items`. Ví dụ, `GET /orders` và `GET /invoices` đều trả về dữ liệu trong trường `items`.
- **Ví dụ cấp độ 2**: Nếu công ty của bạn sử dụng `customer_id` trong một API, thì tất cả các API khác của công ty cũng nên sử dụng `customer_id`, thay vì `user_id` hoặc `client_id`.
- **Ví dụ cấp độ 3**: Trong lĩnh vực thương mại điện tử, nhiều API sử dụng tài nguyên `/cart` để quản lý giỏ hàng. Nếu API của bạn cũng sử dụng `/cart`, thì điều này giúp người dùng quen thuộc hơn.
- **Ví dụ cấp độ 4**: Sử dụng mã trạng thái HTTP như `404 Not Found` để biểu thị tài nguyên không tồn tại là một thông lệ chung trên toàn thế giới.

---

##### 6.1.4 Sao Chép Người Khác: Tuân Theo Các Thông Lệ Chung và Tiêu Chuẩn (Copying Others: Following Common Practices and Meeting Standards)

**Giải thích**: Thay vì tự tạo ra các quy tắc mới, API nên tuân theo các tiêu chuẩn và thông lệ chung đã được chấp nhận rộng rãi. Điều này giúp người dùng dễ dàng làm quen với API của bạn, vì họ đã quen với các mẫu tương tự từ các API khác.

- **Tiêu chuẩn**: Sử dụng các định dạng chuẩn như ISO 8601 cho ngày tháng, ISO 4217 cho mã tiền tệ, hoặc E.164 cho số điện thoại.
  - **Ví dụ cụ thể**: Thay vì sử dụng định dạng ngày tháng tùy chỉnh như `DD-MM-YYYY`, hãy sử dụng ISO 8601 (`2025-06-11`). Điều này giúp API của bạn tương thích với các hệ thống khác và dễ hiểu hơn.

- **Thông lệ chung**: Nhiều API sử dụng các mẫu đường dẫn như `/resources/{id}` để truy cập một tài nguyên cụ thể, hoặc sử dụng các tham số như `page` và `page_size` để phân trang.
  - **Ví dụ cụ thể**: Để phân trang, API của bạn nên sử dụng `GET /orders?page=2&page_size=20` thay vì một tham số tùy chỉnh như `GET /orders?offset=20&limit=20`. Cách đầu tiên là thông lệ phổ biến hơn trong các API REST.

- **Ví dụ thực tế**: API của GitHub sử dụng các tiêu chuẩn REST và mã trạng thái HTTP phổ biến. Nếu bạn thiết kế một API để quản lý kho mã nguồn, việc sao chép các mẫu của GitHub (như `GET /repos/{owner}/{repo}`) sẽ giúp người dùng dễ dàng làm quen với API của bạn.

---

##### 6.1.5 Nhất Quán Là Khó và Phải Được Thực Hiện Một Cách Thông Minh (Being Consistent Is Hard and Must Be Done Wisely)

**Giải thích**: Việc duy trì sự nhất quán không phải lúc nào cũng dễ dàng, đặc biệt khi API phát triển hoặc khi có nhiều nhóm phát triển cùng làm việc. Cần có các hướng dẫn thiết kế API (API design guidelines) và công cụ để đảm bảo sự nhất quán.

- **Thách thức**: Các nhóm phát triển khác nhau có thể vô tình tạo ra các mẫu không nhất quán nếu không có hướng dẫn chung. Ví dụ, một nhóm có thể sử dụng `total_amount`, trong khi nhóm khác sử dụng `amount_total`.
- **Giải pháp**: Tạo một bộ hướng dẫn thiết kế API, sử dụng các công cụ như linter (kiểm tra mã tự động) để đảm bảo rằng các tài nguyên, hành động, và dữ liệu tuân theo các quy tắc nhất quán.
  - **Ví dụ cụ thể**: Sử dụng công cụ như Spectral hoặc OpenAPI linter để kiểm tra xem tất cả các trường ngày tháng có tuân theo định dạng ISO 8601 hay không.

- **Cân bằng**: Đôi khi, sự nhất quán quá mức có thể làm API trở nên cứng nhắc. Ví dụ, áp dụng cùng một cấu trúc phản hồi cho mọi tài nguyên có thể không phù hợp nếu một số tài nguyên cần dữ liệu đặc biệt.
  - **Ví dụ cụ thể**: Nếu tài nguyên `/orders` trả về danh sách đơn hàng với trường `items`, nhưng tài nguyên `/statistics` trả về dữ liệu thống kê (không phải danh sách), thì việc ép buộc `/statistics` sử dụng trường `items` có thể không hợp lý. Trong trường hợp này, sự nhất quán cần được cân nhắc dựa trên ngữ cảnh.

---

#### 6.2 Khả Năng Thích Ứng (Being Adaptable)

**Mục tiêu**: Một API thích ứng là một API có thể đáp ứng các nhu cầu khác nhau của người dùng, chẳng hạn như cung cấp dữ liệu ở nhiều định dạng, hỗ trợ quốc tế hóa, hoặc cho phép người dùng tùy chỉnh phản hồi.

##### 6.2.1 Cung Cấp và Chấp Nhận Các Định Dạng Khác Nhau (Providing and Accepting Different Formats)

**Giải thích**: API nên hỗ trợ nhiều định dạng dữ liệu (như JSON, XML, CSV) để phù hợp với các ứng dụng khách khác nhau. Điều này được thực hiện thông qua cơ chế **content negotiation** (đàm phán nội dung), sử dụng tiêu đề HTTP `Accept` và `Content-Type`.

- **Content Negotiation**: Người dùng có thể yêu cầu định dạng dữ liệu mong muốn bằng tiêu đề `Accept`. API sẽ trả về dữ liệu theo định dạng được yêu cầu nếu nó hỗ trợ.
  - **Ví dụ cụ thể**: Một yêu cầu `GET /orders` với tiêu đề `Accept: application/json` sẽ trả về dữ liệu JSON:
    ```json
    {
      "items": [
        { "id": 1, "customer_id": 101, "total": 99.99 }
      ]
    }
    ```
    Nếu yêu cầu có tiêu đề `Accept: text/csv`, API có thể trả về:
    ```csv
    id,customer_id,total
    1,101,99.99
    ```

- **Tham số format**: Một số API sử dụng tham số truy vấn `format` để chỉ định định dạng, thay vì tiêu đề `Accept`.
  - **Ví dụ cụ thể**: `GET /orders?format=csv` sẽ trả về dữ liệu ở định dạng CSV.

- **Chấp nhận định dạng đầu vào**: API cũng nên chấp nhận dữ liệu đầu vào ở các định dạng khác nhau, được chỉ định bằng tiêu đề `Content-Type`.
  - **Ví dụ cụ thể**: Khi gửi yêu cầu `POST /orders` với tiêu đề `Content-Type: application/json`, dữ liệu có thể là:
    ```json
    {
      "customer_id": 103,
      "total": 199.99
    }
    ```
    Nếu API cũng hỗ trợ XML, thì yêu cầu với `Content-Type: application/xml` có thể là:
    ```xml
    <order>
      <customer_id>103</customer_id>
      <total>199.99</total>
    </order>
    ```

- **Lỗi không hỗ trợ định dạng**: Nếu API không hỗ trợ định dạng được yêu cầu, nó nên trả về mã trạng thái `406 Not Acceptable` hoặc `415 Unsupported Media Type`.
  - **Ví dụ cụ thể**: Nếu yêu cầu `GET /orders` với `Accept: application/pdf` nhưng API không hỗ trợ PDF, thì phản hồi có thể là:
    ```json
    {
      "error": {
        "code": "UNSUPPORTED_FORMAT",
        "message": "PDF format is not supported"
      }
    }
    ```
    với mã `406 Not Acceptable`.

---

##### 6.2.2 Quốc Tế Hóa và Địa Phương Hóa (Internationalizing and Localizing)

**Giải thích**: API nên hỗ trợ quốc tế hóa (internationalization) và địa phương hóa (localization) để phục vụ người dùng từ các khu vực và ngôn ngữ khác nhau. Điều này bao gồm việc trả về nội dung bằng ngôn ngữ phù hợp và định dạng dữ liệu theo chuẩn địa phương (như tiền tệ, ngày tháng).

- **Ngôn ngữ**: Sử dụng tiêu đề `Accept-Language` hoặc tham số truy vấn `language` để chỉ định ngôn ngữ mong muốn.
  - **Ví dụ cụ thể**: Yêu cầu `GET /products` với `Accept-Language: vi-VN` có thể trả về mô tả sản phẩm bằng tiếng Việt:
    ```json
    {
      "items": [
        { "id": 1, "name": "Điện thoại thông minh", "price": 999.99 }
      ]
    }
    ```
    Với `Accept-Language: en-US`, phản hồi sẽ là:
    ```json
    {
      "items": [
        { "id": 1, "name": "Smartphone", "price": 999.99 }
      ]
    }
    ```

- **Định dạng địa phương**: Dữ liệu như tiền tệ, ngày tháng, hoặc số nên được điều chỉnh theo địa phương. Ví dụ, tiền tệ có thể được định dạng theo mã ISO 4217 (VND, USD), và số có thể sử dụng dấu phân cách phù hợp (dấu phẩy hoặc dấu chấm).
  - **Ví dụ cụ thể**: Với `Accept-Language: vi-VN`, giá sản phẩm có thể là:
    ```json
    {
      "id": 1,
      "name": "Điện thoại thông minh",
      "price": "999,99 VND"
    }
    ```
    Với `Accept-Language: en-US`, giá sẽ là:
    ```json
    {
      "id": 1,
      "name": "Smartphone",
      "price": "$999.99"
    }
    ```

- **Tiêu chuẩn**: Sử dụng các mã ngôn ngữ (ISO 639) và mã quốc gia (ISO 3166) để đảm bảo tính nhất quán.
  - **Ví dụ cụ thể**: Thay vì sử dụng `lang=vn`, hãy sử dụng `language=vi-VN` theo chuẩn RFC 5646.

---

##### 6.2.3 Lọc, Phân Trang, và Sắp Xếp (Filtering, Paginating, and Sorting)

**Giải thích**: API nên cung cấp khả năng lọc (filtering), phân trang (paginating), và sắp xếp (sorting) để người dùng có thể tùy chỉnh dữ liệu trả về theo nhu cầu của họ. Điều này cải thiện hiệu suất và trải nghiệm người dùng.

- **Lọc**: Cho phép người dùng chỉ định các điều kiện để lọc dữ liệu.
  - **Ví dụ cụ thể**: Để lấy các đơn hàng của một khách hàng cụ thể, API có thể hỗ trợ `GET /orders?customer_id=101`, trả về:
    ```json
    {
      "items": [
        { "id": 1, "customer_id": 101, "total": 99.99 }
      ]
    }
    ```

- **Phân trang**: Chia dữ liệu thành các trang nhỏ để tránh tải quá nhiều dữ liệu cùng lúc.
  - **Ví dụ cụ thể**: Yêu cầu `GET /orders?page=2&page_size=10` sẽ trả về trang thứ hai với tối đa 10 đơn hàng:
    ```json
    {
      "items": [
        { "id": 11, "customer_id": 101, "total": 100 },
        { "id": 12, "customer_id": 102, "total": 150 }
      ],
      "meta": {
        "page": 2,
        "page_size": 10,
        "total_pages": 5
      }
    }
    ```

- **Sắp xếp**: Cho phép người dùng để chỉ định thứ tự sắp xếp dữ liệu.
  - **Ví dụ cụ thể**: `GET /orders?sort_by=total&order=desc` sẽ trả về các đơn hàng được sắp xếp theo tổng giá trị giảm dần:
    ```json
    {
      "items": [
        { "id": 2, "customer_id": 102, "total": 150 },
        { "id": 1, "customer_id": 101, "total": 100 }
      ]
    }
    ```

- **Tiêu chuẩn**: Sử dụng các tham số phổ biến như `page`, `page_size`, `sort_by`, và `order` (với giá trị `asc` hoặc `desc`).

---

#### 6.3 Khả Năng Khám Phá (Being Discoverable)

**Mục tiêu**: Một API có khả năng khám phá là một API mà người dùng có thể tự tìm hiểu cách sử dụng mà không cần tài liệu chi tiết. Điều này được đạt được thông qua việc cung cấp siêu dữ liệu (metadata), sử dụng API hypermedia, và tận dụng giao thức HTTP.

##### 6.3.1 Cung cấp Siêu Dữ liệu (Providing Metadata)

**Giải thích**: Siêu dữ liệu cung cấp thông tin về tài nguyên hoặc trạng thái của API, giúp người dùng hiểu rõ hơn về dữ liệu và các hành động có thể thực hiện.

- **Siêu dữ liệu về tài nguyên**: Bao gồm thông tin như tổng số mục trong danh sách, thông tin phân trang, hoặc thời gian cập nhật.
  - **Ví dụ cụ thể**: Phản hồi từ `GET /orders` có thể bao gồm siêu dữ liệu:
    ```json
    {
      "items": [
        { "id": 1, "customer_id": 101, "total": 99.99 }
      ],
      "meta": {
        "total": 100,
        "page": 1,
        "page_size": 50,
        "last_updated": "2025-06-10T22:41:00"
      }
    }
    ```

- **Siêu dữ liệu về hành động**: Cung cấp thông tin về các hành động có thể thực hiện trên tài nguyên.
  - **Ví dụ cụ thể**: Phản hồi có thể bao gồm các liên kết (links) đến các hành động liên quan:
    ```json
    {
      "id": 1",
      "customer_id": 101,
      "links": {
        "self": "http://api.example.com/v1/orders/1",
        "cancel": "http://api.example.com/v1/orders/1/cancel"
      }
    }
    ```

---

##### 6.3.2 Tạo API Hypermedia (Creating Hypermedia APIs)

**Giải thích**: API hypermedia (HATEOAS - Hypermedia as the Engine of Application State) cho phép API tự mô tả bằng cách cung cấp các liên kết đến các tài nguyên hoặc hành động liên quan. Điều này giúp người dùng hoặc máy móc tự động khám phá các chức năng của API.

- **Liên kết đến tài nguyên liên quan**: Mỗi tài nguyên có thể bao gồm các liên kết đến các tài nguyên khác hoặc các hành động.
  - **Ví dụ cụ thể**: Phản hồi từ `GET /orders/1` có thể là:
    ```json
    {
      "id": 1,
      "customer_id": 101,
      "total": 99.99,
      "links": [
        { "rel": "self", "href": "https://api.example.com/v1/orders/1" },
        { "rel": "customer", "href": "https://api.example.com/v1/customers/101" },
        { "rel": "cancel", "href": "https://api.example.com/v1/orders/1/cancel", "method": "POST" }
      ]
    }
    ```
    Các liên kết này cho phép người dùng biết rằng họ có thể lấy thông tin khách hàng hoặc hủy đơn hàng mà không cần tra cứu tài liệu.

- **HATEOAS**: Trong một API hypermedia, trạng thái của ứng dụng được điều hướng thông qua các liên kết. Điều này làm cho API giống như một trang web, nơi người dùng điều hướng bằng cách nhấp vào các liên kết.
  - **Ví dụ cụ thể**: Một ứng dụng khách có thể bắt đầu từ điểm gốc `/api/v1`:
    ```json
    {
      "links": [
        { "rel": "orders", "href": "/api/v1/orders" },
        { "rel": "customers", "href": "/api/v1/customers" }
      ]
    }
    ```
    Từ đây, ứng dụng có thể tự động khám phá để các tài nguyên `/orders` và `/customers`.

- **Định dạng Hypermedia**: Sử dụng các định dạng như Siren hoặc JSON-LD để cung cấp các liên kết phong phú hơn.
  - **Ví dụ cụ thể (Siren)**:
    ```json
{
      {
        "class": ["order"],
        "properties": {
          "id": 1,
          "customer_id": 101,
          "total": 99.99
        },
        "links": [
          { "rel": "self", "href": "/api/v1/orders/1" },
          { "rel": "cancel", "href": "/api/v1/orders/1/cancel", "method": "POST" }
        ]
      }
    }
    ```

---

##### 6.3.3 Tận Dụng Giao Thức HTTP (Taking Advantage of the HTTP Protocol)

**Giải thích**: Giao thức HTTP cung cấp nhiều tính năng giúp API dễ khám phá hơn, chẳng hạn như các phương thức, mã trạng thái, và tiêu đề.

- **Phương thức OPTIONS**: Phương thức `OPTIONS` cho phép người dùng khám phá các phương thức HTTP được hỗ trợ trên một tài nguyên.
  - **Ví dụ cụ thể**: Yêu cầu `OPTIONS /orders/1` có thể trả về tiêu đề:
    ```
    Allow: GET, POST, PATCH, DELETE
    ```
    Điều này cho biết người dùng có thể sử dụng các phương thức GET`, `POST`, và `PATCH` trên tài nguyên này.

- **Tiêu đề Link**: Tiêu đề `Link` có thể cung cấp các liên kết liên quan đến tài nguyên.
  - **Ví dụ cụ thể**: Phản hồi từ `GET /orders/1` có thể bao gồm tiêu đề:
    ```
    Link: </api/v1/customers/101>; rel="customer"
    ```

- **Mã trạng thái**: Sử dụng mã trạng thái HTTP một cách ý nghĩa để cung cấp thông tin về kết quả của hành động.
  - **Ví dụ cụ thể**: Nếu yêu cầu `DELETE /orders/1` trả về `204 No Content`, thì người dùng biết rằng đơn hàng đã được xóa thành công mà không cần kiểm tra thêm.

---

### Tổng Kết Chương 6

Chương 6 nhấn mạnh rằng một API dự đoán được là một API dễ sử dụng và dễ hiểu, đạt được thông qua:
- **Sự nhất quán**: Dữ liệu và mục tiêu phải được thiết kế thống nhất về cách đặt tên, định dạng, và cấu trúc.
- **Khả năng thích ứng**: API nên hỗ trợ nhiều định dạng, quốc tế hóa, và các tính năng như lọc, phân trang, và sắp xếp.
- **Khả năng khám phá**: API nên cung cấp siêu dữ liệu, sử dụng hypermedia, và tận dụng giao thức HTTP để giúp người dùng tự tìm hiểu cách sử dụng.

**Ví dụ tổng hợp**: Một API quản lý cửa hàng có thể được thiết kế như sau để đảm bảo tính dự đoán:
- **Nhất quán**: Tất cả tài nguyên sử dụng `customer_id`, định dạng ngày tháng ISO 8601, và cấu trúc danh sách với trường `items`.
- **Thích ứng**: Hỗ trợ JSON và CSV (`GET /orders?format=csv`), quốc tế hóa (`Accept-Language: vi-VN`), và phân trang (`page`, `page_size`).
- **Khám phá**: Trả về siêu dữ liệu (`meta`) và liên kết hypermedia:
  ```json
  {
    "items": [
      { "id": 1", "customer_id": 101, "total": 99.99 }
    ],
    "meta": {
      "total": 1,
      "page": 1,
      "page_size": 50
    },
    "links": [
      { "rel": "self", "href": "/api/v1/orders" },
      { "rel": "next", "href": "/api/v1/orders?page=2" }
    ]
  }
  ```

Hy vọng nội dung chi tiết này giúp bạn hiểu rõ cách thiết kế một API có tính dự đoán theo chương 6 của *The Design of Web APIs*. Nếu bạn cần thêm ví dụ hoặc giải thích sâu hơn về bất kỳ phần nào, hãy cho mình biết!