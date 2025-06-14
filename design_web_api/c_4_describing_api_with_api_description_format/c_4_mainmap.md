# Mô tả API của bạn với Định dạng mô tả API (Describing Your API with an API Description Format)
Giới thiệu về Định dạng mô tả API và cách sử dụng Đặc tả OpenAPI (OAS) để mô tả chi tiết API.

### 1. Định dạng mô tả API là gì? (What is an API description format?)
* Một định dạng dữ liệu dùng để mô tả một API, giống như một tệp văn bản có cấu trúc.
* Chứa **dữ liệu có cấu trúc**, có thể đọc và chuyển đổi bởi chương trình.
* **Ví dụ:** Mô tả mục tiêu "Thêm sản phẩm vào danh mục" bằng YAML.

#### 1.1.1 Giới thiệu Đặc tả OpenAPI (OAS)
* **Định dạng mô tả API REST độc lập với ngôn ngữ lập trình**, thúc đẩy bởi Sáng kiến OpenAPI (OAI).
* Trước đây là Đặc tả Swagger, phiên bản mới nhất (khi viết sách) là 3.0.
* Có thể viết bằng **YAML** (khuyến nghị vì dễ đọc) hoặc **JSON**.
* **Cấu trúc OAS cơ bản:**
    - `openapi`: Phiên bản đặc tả.
    - `info`: Thông tin chung về API (tiêu đề, phiên bản).
    - `paths`: Định nghĩa các tài nguyên (URLs) và thao tác.
    - `parameters`: Tham số của thao tác.
    - `responses`: Các phản hồi của thao tác.
    - `content`: Nội dung phản hồi.
    - `schema`: Cấu trúc dữ liệu của nội dung/tham số.

#### 1.1.2 Tại sao nên sử dụng định dạng mô tả API?
* **Lưu trữ phiên bản dễ dàng:** Tệp văn bản, dùng với Git.
* **Mô tả hiệu quả:** Định nghĩa các thành phần tái sử dụng.
* **Sử dụng công cụ chuyên dụng:** Trình soạn thảo (Swagger Editor), tạo tài liệu (Swagger UI, ReDoc).
* **Chia sẻ và tạo tài liệu:** Dễ dàng chia sẻ, tạo tài liệu tham khảo thân thiện.
* **Tạo mã và cấu hình công cụ:** Tạo khung sườn mã, SDK, cấu hình API gateway.

#### 1.1.3 Khi nào nên sử dụng định dạng mô tả API?
* **Không nên sử dụng khi:** Xác định mục tiêu API hoặc các khái niệm cơ bản.
* **Chắc chắn phải sử dụng khi:** Thiết kế biểu diễn có thể lập trình của mục tiêu và dữ liệu.
* Bắt đầu dùng OAS khi **thiết kế đường dẫn tài nguyên và chọn phương thức HTTP**.

### 2. Mô tả tài nguyên và hành động của API bằng OAS (Describing API resources and actions with OAS)

#### 2.2.1 Tạo tài liệu OAS (Creating an OAS document)
* Tài liệu tối thiểu: `openapi`, `info`, `paths` (ban đầu trống).
* **Lưu ý:** Phiên bản đặc tả và phiên bản API phải đặt trong dấu ngoặc kép.

#### 2.2.2 Mô tả tài nguyên (Describing a resource)
* Thêm đường dẫn tài nguyên vào thuộc tính `paths`.
* Có thể thêm thuộc tính `description` để mô tả tài nguyên.
* **Ví dụ:** `/products` với `description: The products catalog`.

#### 2.2.3 Mô tả các thao tác trên tài nguyên (Describing operations on a resource)
* Mỗi tài nguyên phải chứa ít nhất một thao tác (operation), mô tả bằng các **phương thức HTTP**.
* Mỗi thao tác có `summary` (mô tả ngắn) và `description` (mô tả chi tiết).
* Thuộc tính `responses` liệt kê các phản hồi có thể, xác định bằng mã trạng thái HTTP (đặt trong dấu ngoặc kép) và mỗi phản hồi phải có `description`.
* **Ví dụ:** `get` và `post` cho `/products`.

### 3. Mô tả dữ liệu API bằng OpenAPI và JSON Schema (Describing API data with OpenAPI and JSON Schema)

#### OAS dựa vào **đặc tả JSON Schema** để mô tả dữ liệu (tham số, thân yêu cầu, thân phản hồi).

#### 3.3.1 Mô tả tham số truy vấn (Describing query parameters)
* Thêm thuộc tính `parameters` vào thao tác.
* Mỗi tham số có `name`, `in: query`, và `schema`.
* `required` và `description` là tùy chọn.

#### 3.3.2 Mô tả dữ liệu bằng JSON Schema (Describing data with JSON Schema)
* Để mô tả đối tượng: `type: object`, liệt kê `properties`.
* Mỗi thuộc tính có `name` và `type`.
* Sử dụng danh sách `required` để chỉ định thuộc tính bắt buộc.
* Thêm `description` và `example` để rõ ràng hơn.
* Hỗ trợ đối tượng lồng nhau, mảng.

#### 3.3.3 Mô tả phản hồi (Describing responses)
* Dữ liệu trong thân phản hồi HTTP định nghĩa trong `content` của phản hồi.
* Chỉ định **kiểu media** (ví dụ: `application/json`).
* Mô tả schema của nội dung bằng JSON Schema.
* **Ví dụ:** Thao tác `GET /products` trả về `type: array` với `items` là schema sản phẩm.

#### 3.3.4 Mô tả tham số thân yêu cầu (Describing body parameters)
* Mô tả trong thuộc tính `requestBody` của thao tác.
* Có `content` với kiểu media và `schema` JSON Schema.
* **Ví dụ:** `POST /products` yêu cầu `name`, `price`, `supplierReference`.

### 4. Mô tả API hiệu quả bằng OAS (Describing an API effciently with OAS)

#### 4.4.1 Tái sử dụng các thành phần (Reusing components)
* Tránh lặp lại mô tả bằng cách sử dụng phần `components` ở cấp gốc.
* Định nghĩa `schemas`, `parameters`, `responses` có thể tái sử dụng.
* **Tham chiếu:** Sử dụng `$ref: "#/components/schemas/product"` để sử dụng lại.

#### 4.4.2 Mô tả tham số đường dẫn (Describing path parameters)
* Đối với đường dẫn chứa biến (ví dụ: `/products/{productId}`).
* `in: path`, `required: true`.
* `name` trong `parameters` phải khớp với tên trong `{}` trên đường dẫn.
* Có thể định nghĩa ở cấp `components.parameters` hoặc ở cấp tài nguyên nếu nhiều thao tác sử dụng.

### Tóm tắt Chương 4: Thiết kế API cơ bản

#### Bạn đã học được:
* API thực sự là gì.
* Cách xác định mục tiêu của API từ góc độ người dùng.
* Cách chuyển đổi mục tiêu thành biểu diễn có thể lập trình.
* **Cách mô tả chính thức biểu diễn có thể lập trình này bằng OAS**.

#### Các điểm chính cần nhớ:
* Định dạng mô tả API là cách đơn giản, có cấu trúc để mô tả và chia sẻ giao diện lập trình.
* Tài liệu mô tả API là tài liệu máy có thể đọc, có thể dùng để tạo tài liệu tham khảo API, v.v.
* Chỉ nên sử dụng định dạng mô tả API khi thiết kế biểu diễn có thể lập trình và dữ liệu.
* Luôn tận dụng các tính năng tài liệu của định dạng mô tả API, đặc biệt là định nghĩa các thành phần có thể tái sử dụng.
