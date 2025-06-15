# The design of web api

## C1 What is API design?
### 1.1 What is an API?
##### 1.1.1 API là giao diện web cho phần mềm (An API is a web interface for software)
##### 1.1.2 API biến phần mềm thành các khối LEGO® (APIs turn software into LEGO® bricks)

### 1.2 Tại sao thiết kế API lại quan trọng (Why API design matters)
API được sử dụng bởi phần mềm, nhưng người xây dựng phần mềm đó là các nhà phát triển – con người.

##### 1.2.1 API công khai hoặc riêng tư là giao diện cho các nhà phát triển khác (A public or private API is an interface for other developers)
##### 1.2.2 API được tạo ra để ẩn đi chi tiết triển khai (An API is made to hide the implementation)
##### 1.2.3 Hậu quả khủng khiếp của việc thiết kế API kém (The terrible consequences of poorly designed APIs)

### 1.3 Các yếu tố của thiết kế API (The elements of API design)
Học thiết kế API yêu cầu tập trung vào giao diện, biết toàn bộ ngữ cảnh, và thể hiện sự đồng cảm với tất cả người dùng và phần mềm liên quan.

##### 1.3.1 Học các nguyên tắc vượt ra ngoài thiết kế giao diện lập trình (Learning the principles beyond programming interface design)
##### 1.3.2 Khám phá tất cả các khía cạnh của thiết kế API (Exploring all facets of API design)

## Designing APIs for Users
### Tại sao cần thiết kế API vì người dùng?
### Góc nhìn đúng đắn trong thiết kế
### Thiết kế giao diện của phần mềm
### Xác định Mục tiêu của API (Identifying an API’s Goals)
### Sử dụng API Goals Canvas

### Tránh Góc nhìn của Nhà cung cấp (Avoiding the Provider’s Perspective)
#### Ví dụ API Goals Canvas cho Shopping API (một phần)s

## Designing REST APIs
### 3.1 Để thiết kế một REST API, trước tiên cần hiểu cách nó hoạt động
#### Phân tích một lệnh gọi REST API
    - Phương thức HTTP: `GET` (truy xuất tài nguyên).
    - Đường dẫn (path): `/products/P123` (xác định một tài nguyên).
    - Mã trạng thái (status code): `200 OK` (thành công).
    - Phần thân phản hồi (response body): Chứa nội dung tài nguyên (ví dụ: dữ liệu JSON sản phẩm).
#### Các nguyên tắc cơ bản của HTTP
#### Các nguyên tắc cơ bản của REST API
### 3.1 Chuyển đổi các mục tiêu API thành một REST API

#### 3.2.1 Xác định tài nguyên và các mối quan hệ của chúng bằng API goals canvas
#### 3.2.2 Xác định hành động và các tham số, giá trị trả về của chúng bằng API goals canvas
#### 3.2.3 Thiết kế đường dẫn cho tài nguyên
#### 3.2.4 Biểu diễn hành động bằng HTTP

### 3.3 Thiết kế dữ liệu của API
#### Quy trình thiết kế dữ liệu
    1. Thiết kế khái niệm (concepts).
    2. Thiết kế phản hồi (responses) từ khái niệm.
    3. Thiết kế tham số (parameters) từ khái niệm hoặc phản hồi.
    4. Kiểm tra nguồn dữ liệu của tham số.
#### 3.3.1 Thiết kế khái niệm
#### 3.3.2 Thiết kế phản hồi từ khái niệm
#### 3.3.3 Thiết kế tham số từ khái niệm hoặc phản hồi
#### 3.3.4 Kiểm tra nguồn dữ liệu của tham số
#### 3.3.5 Thiết kế các tham số khác

### 3.4 Tìm sự cân bằng khi đối mặt với các thách thức thiết kế
#### 3.4.1 Ví dụ về đánh đổi trong REST
    - `POST /cart/check-out`: Thân thiện, nhưng không tuân thủ REST (dùng động từ trong đường dẫn).
    - `PATCH /cart`: Tuân thủ hơn (cập nhật trạng thái), nhưng có thể khó hiểu.
    - `POST /orders`: Tuân thủ nhất (tạo tài nguyên mới), nhưng không trực quan.
#### 3.4.2 Cân bằng giữa tính thân thiện với người dùng và sự tuân thủ

### 3.5 Hiểu lý do tại sao kiến trúc REST lại quan trọng đối với việc thiết kế bất kỳ loại API nào
Kiến trúc REST là bộ các nguyên tắc thiết kế hệ thống phân tán,
hữu ích khi thiết kế bất kỳ loại API nào, không chỉ REST API.

#### 3.5.1 Giới thiệu kiến trúc REST
    1. Client/server separation (Tách biệt máy khách/máy chủ).
    2. Statelessness (Không trạng thái).
    3. Cacheability (Khả năng bộ nhớ đệm).
    4. Layered system (Hệ thống phân lớp).
    5. Code on demand (Mã theo yêu cầu): Tùy chọn.
    6. Uniform interface (Giao diện đồng nhất): Ràng buộc cơ bản nhất.
#### 3.5.2 Tác động của các ràng buộc REST đối với thiết kế API

## Mô tả API của bạn với Định dạng mô tả API

### 1. Định dạng mô tả API là gì? (What is an API description format?)
#### 1.1.1 Giới thiệu Đặc tả OpenAPI (OAS)
    - `openapi`: Phiên bản đặc tả.
    - `info`: Thông tin chung về API (tiêu đề, phiên bản).
    - `paths`: Định nghĩa các tài nguyên (URLs) và thao tác.
    - `parameters`: Tham số của thao tác.
    - `responses`: Các phản hồi của thao tác.
    - `content`: Nội dung phản h
    - `schema`: Cấu trúc dữ liệu của nội dung/tham số.
#### 1.1.2 Tại sao nên sử dụng định dạng mô tả API?
#### 1.1.3 Khi nào nên sử dụng định dạng mô tả API?

### 2. Mô tả tài nguyên và hành động của API bằng OAS (Describing API resources and actions with OAS)
#### 2.2.1 Tạo tài liệu OAS (Creating an OAS document)
#### 2.2.2 Mô tả tài nguyên (Describing a resource)
#### 2.2.3 Mô tả các thao tác trên tài nguyên (Describing operations on a resource)

### 3. Mô tả dữ liệu API bằng OpenAPI và JSON Schema (Describing API data with OpenAPI and JSON Schema)
OAS dựa vào đặc tả JSON Schema để mô tả dữ liệu (tham số, thân yêu cầu, thân phản hồi).

#### 3.3.1 Mô tả tham số truy vấn (Describing query parameters)
#### 3.3.2 Mô tả dữ liệu bằng JSON Schema (Describing data with JSON Schema)
#### 3.3.3 Mô tả phản hồi (Describing responses)
#### 3.3.4 Mô tả tham số thân yêu cầu (Describing body parameters)

### 4. Mô tả API hiệu quả bằng OAS (Describing an API effciently with OAS)
#### 4.4.1 Tái sử dụng các thành phần (Reusing components)
#### 4.4.2 Mô tả tham số đường dẫn (Describing path parameters)

## Thiết kế API rõ ràng
Designing Straightforward APIs

### 5.1 Thiết kế các biểu diễn rõ ràng (Designing Straightforward Representations)
#### 5.1.1 Chọn tên rõ ràng (Choosing Crystal-Clear Names)
#### 5.1.2 Chọn kiểu và định dạng dữ liệu dễ sử dụng (Choosing Easy-to-Use Data Types and Formats)
#### 5.1.3 Chọn dữ liệu sẵn sàng sử dụng (Choosing Ready-to-Use Data)

### 5.2 Thiết kế tương tác rõ ràng (Designing Straightforward Interactions)
Tương tác trong API bao gồm việc người dùng cung cấp đầu vào và nhận phản hồi.

#### 5.2.1 Yêu cầu đầu vào rõ ràng (Requesting Straightforward Inputs)
#### 5.2.2 Xác định tất cả các phản hồi lỗi có thể xảy ra (Identifying All Possible Error Feedbacks)
#### 5.2.3 Trả về phản hồi lỗi đầy đủ thông tin (Returning Informative Error Feedback)
#### 5.2.4 Trả về phản hồi lỗi toàn diện (Returning Exhaustive Error Feedback)
#### 5.2.5 Trả về phản hồi thành công đầy đủ thông tin (Returning Informative Success Feedback)

### 5.3 Thiết kế luồng rõ ràng (Designing Straightforward Flows)
#### 5.3.1 Xây dựng chuỗi mục tiêu rõ ràng (Building a Straightforward Goal Chain)
#### 5.3.2 Ngăn chặn lỗi (Preventing Errors)
#### 5.3.3 Tổng hợp mục tiêu (Aggregating Goals)
#### 5.3.4 Thiết kế luồng không trạng thái (Designing Stateless Flows)

## Làm cho API của bạn dễ đoán (Making Your API Predictable)

### 6.1. Trở nên nhất quán (Being Consistent)
#### 6.1.1. Thiết kế dữ liệu nhất quán (Designing Consistent Data)
#### 6.1.2. Thiết kế các mục tiêu (Goals) nhất quán (Designing Consistent Goals)
#### 6.1.3. Bốn cấp độ nhất quán (The Four Levels of Consistency)
#### 6.1.4. Sao chép người khác: Tuân thủ các quy tắc chung và tiêu chuẩn (Copying Others: Following Common Practices and Meeting Standards)
#### 6.1.5. Nhất quán là khó và phải được thực hiện một cách khôn ngoan (Being Consistent is Hard and Must be Done Wisely)

### 6.2. Trở nên thích ứng (Being Adaptable)
#### 6.2.1. Cung cấp và chấp nhận các định dạng khác nhau (Providing and Accepting Different Formats)
#### 6.2.2. Quốc tế hóa và địa phương hóa (Internationalizing and Localizing)
#### 6.2.3. Lọc, phân trang và sắp xếp (Filtering, Paginating, and Sorting)

### 6.3. Khám phá (Being Discoverable)
#### 6.3.1. Cung cấp siêu dữ liệu (Providing Metadata)
#### 6.3.2. Tạo API siêu phương tiện (Creating Hypermedia APIs)
#### 6.3.3. Tận dụng giao thức HTTP (Taking Advantage of the HTTP Protocol)

## Thiết kế API ngắn gọn và có tổ chức tốt (Designing concise and well-organized APIs)
### 7.1 Tổ chức API (Organizing an API)
#### 7.1.1 Tổ chức dữ liệu (Organizing Data)
#### 7.1.2 Tổ chức phản hồi (Organizing Feedback)
#### 7.1.3 Tổ chức mục tiêu (Organizing Goals)

### 7.2 Xác định kích thước API (Sizing an API)
#### 7.2.1 Chọn mức độ chi tiết dữ liệu (Choosing Data Granularity)
#### 7.2.2 Chọn mức độ chi tiết mục tiêu (Choosing Goal Granularity)
#### 7.2.3 Chọn mức độ chi tiết API (Choosing API Granularity)

## Thiết kế API bảo mật
### 8.1 Tổng quan về Bảo mật API (An Overview of API Security)
#### 8.1.1 Đăng ký người dùng (Registering a consumer)
#### 8.1.2 Nhận thông tin xác thực
#### 8.1.3 Thực hiện cuộc gọi API (Making an API call)
#### 8.1.4 Nhìn nhận thiết kế API từ góc độ bảo mật (Envisioning API design from the perspective of security)

### 8.2 Phân vùng API để tạo điều kiện kiểm soát truy cập (Partitioning an API to Facilitate Access Control)
#### 8.2.1 Xác định các scopes chi tiết nhưng phức tạp (Defining flexible but complex fine-grained scopes)
#### 8.2.2 Xác định các scopes đơn giản nhưng ít linh hoạt hơn (Defining simple but less flexible coarse-grained scopes)
#### 8.2.4 Xác định scopes với định dạng mô tả API (Defining scopes with the API description format)

### 8.3 Thiết kế với kiểm soát truy cập (Designing with Access Control in Mind)
#### 8.3.1 Biết dữ liệu cần thiết để kiểm soát truy cập (Knowing what data is needed to control access)
#### 8.3.2 Điều chỉnh thiết kế khi cần thiết (Adapting the design when necessary)

### 8.4 Xử lý tài liệu nhạy cảm (Handling Sensitive Material)
#### 8.4.1 Xử lý dữ liệu nhạy cảm (Handling sensitive data)
#### 8.4.2 Xử lý mục tiêu nhạy cảm (Handling sensitive goals)
#### 8.4.3 Thiết kế phản hồi lỗi an toàn (Designing secure error feedback)
#### 8.4.4 Xác định các vấn đề về kiến trúc và giao thức (Identifying architecture and protocol issues)


## Evolving an API design

### 1. Thiết kế các tiến hóa API (Designing API evolutions)

#### Tránh các thay đổi gây lỗi trong dữ liệu đầu ra (Avoiding breaking changes in output data)

#### Tránh các thay đổi gây lỗi đối với dữ liệu đầu vào và tham số (Avoiding breaking changes to input data and parameters)

#### Tránh các thay đổi gây lỗi trong phản hồi thành công và lỗi (Avoiding breaking changes in success and error feedback)

#### Tránh các thay đổi gây lỗi đối với mục tiêu và luồng (Avoiding breaking changes to goals and flows)

#### Tránh các vi phạm bảo mật và thay đổi gây lỗi (Avoiding security breaches and breaking changes)

#### Lưu ý về hợp đồng giao diện vô hình (Being aware of the invisible interface contract)
#### Khi nào một thay đổi gây lỗi không phải là vấn đề? (Introducing a breaking change is not always a problem)

### 2. Phiên bản hóa API (Versioning an API)

#### Phiên bản hóa API so với phiên bản hóa triển khai (Contrasting API and implementation versioning)

#### Chọn cách biểu diễn phiên bản API từ góc độ người dùng (Choosing an API versioning representation from the consumer's perspective)

#### Chọn mức độ chi tiết của phiên bản API (Choosing API versioning granularity)

#### Hiểu tác động của phiên bản hóa API ngoài thiết kế (Understanding the impact of API versioning beyond design)

### 3. Thiết kế API có khả năng mở rộng (Designing APIs with extensibility in mind)

#### Thiết kế dữ liệu có khả năng mở rộng (Designing extensible data)

#### Thiết kế tương tác có khả năng mở rộng (Designing extensible interactions)

#### Thiết kế luồng có khả năng mở rộng (Designing extensible flows)

#### Thiết kế API có khả năng mở rộng (Designing extensible APIs)


## C 10 Thiết kế API hiệu quả mạng (Designing a network-efficient API)
### 1. Tổng quan về các vấn đề truyền thông mạng
Hiệu quả truyền thông mạng là một yếu tố quan trọng mà bất kỳ nhà thiết kế API nào cũng phải nắm rõ.

* Tầm quan trọng:
* Phân tích cuộc gọi API:
* Phân tích vấn đề trong các trường hợp sử dụng:
* Các khía cạnh của hiệu quả mạng: Tốc độ, khối lượng dữ liệu và số lượng cuộc gọi.

### 2. Đảm bảo hiệu quả truyền thông mạng ở cấp độ giao thức
Các tối ưu hóa ở cấp độ giao thức có thể được thực hiện mà không ảnh hưởng nhiều đến thiết kế lý tưởng của API.

* Kích hoạt nén và kết nối liên tục:
* Kích hoạt bộ nhớ đệm (Caching) và yêu cầu có điều kiện (Conditional Requests):

### 3. Đảm bảo hiệu quả truyền thông mạng ở cấp độ thiết kế
Thiết kế API cơ bản quyết định số lượng cuộc gọi và lượng dữ liệu trao đổi giữa người dùng và nhà cung cấp.
* Kích hoạt tính năng lọc (Filtering):
* Lựa chọn dữ liệu phù hợp cho các biểu diễn danh sách:
* Tổng hợp dữ liệu (Aggregating data):
* Đề xuất các biểu diễn khác nhau (Content Negotiation):
* Kích hoạt mở rộng (Enabling Expansion):
* Kích hoạt truy vấn (Enabling Querying):
* Cung cấp dữ liệu và mục tiêu phù hợp hơn:
* Tạo các lớp API khác nhau (API Layers):

## Chương 11: Thiết kế API trong ngữ cảnh
### 11.1 Điều chỉnh giao tiếp theo mục tiêu và bản chất của dữ liệu
Cơ chế yêu cầu/phản hồi đồng bộ và đơn lẻ không phải lúc nào cũng hiệu quả nhất.

* 11.1.1 Quản lý các quy trình dài (Managing long processes)
* 11.1.2 Thông báo sự kiện cho người dùng (Notifying consumers of events)
* 11.1.3 Truyền luồng sự kiện (Streaming event flows)
* 11.1.4 Xử lý nhiều phần tử (Processing multiple elements)

### 11.2 Quan sát toàn bộ ngữ cảnh (Observing the full context)
Thiết kế API đòi hỏi phải xem xét đầy đủ ngữ cảnh để đáp ứng nhu cầu và có thể triển khai được.
* 11.2.1 Nhận thức về các thực hành và giới hạn hiện có của người dùng (Being aware of consumers' existing practices and limitations)
* 11.2.2 Cẩn thận xem xét các giới hạn của nhà cung cấp (Carefully considering the provider’s limitations)

### 11.3 Lựa chọn phong cách API theo ngữ cảnh (Choosing an API style according to the context)
Lựa chọn công cụ (phong cách API) phải dựa trên ngữ cảnh và nhu cầu, không phải thói quen, phổ biến hay sở thích cá nhân.
* 11.3.1 Đối lập các API dựa trên tài nguyên, dữ liệu và chức năng (Contrasting resource-, data-, and function-based APIs)
* 11.3.2 Suy nghĩ vượt ra ngoài các API dựa trên yêu cầu/phản hồi và HTTP (Thinking beyond request/response- and HTTP-based APIs)

## Chương 12: Tài liệu hóa API của bạn (Documenting Your API)
### 1. Tổng quan về Tài liệu API
Tài liệu là một khía cạnh cực kỳ quan trọng mà các nhà thiết kế API phải tham gia.

### 2. Tạo Tài liệu Tham khảo (Reference Documentation)
#### 2.1. Tài liệu hóa các Mô hình Dữ liệu (Documenting Data Models)
#### 2.2. Tài liệu hóa các Mục tiêu/Chức năng (Documenting Goals)
#### 2.3. Tài liệu hóa Bảo mật (Documenting Security)
#### 2.4. Cung cấp Tổng quan về API (Providing an Overview of the API)
#### 2.5. Tạo Tài liệu từ việc Triển khai: Ưu và Nhược điểm (Generating Documentation from the Implementation: Pros and Cons)
    * Ưu điểm:
    * Nhược điểm:

### 3. Tạo Hướng dẫn Sử dụng (Creating a User Guide)
Hướng dẫn sử dụng giải thích cách thức thực sự sử dụng API.
#### 3.1. Tài liệu hóa các Trường hợp Sử dụng (Documenting Use Cases)
#### 3.2. Tài liệu hóa Bảo mật (Documenting Security)
#### 3.3. Cung cấp Tổng quan về các Hành vi và Nguyên tắc chung (Providing an Overview of Common Behaviors and Principles)
#### 3.4. Suy nghĩ vượt ra ngoài Tài liệu Tĩnh (Thinking Beyond Static Documentation)

### 4. Cung cấp Thông tin đầy đủ cho Người Triển khai (Providing Adequate Information to Implementers)
Những người triển khai cần mô tả chi tiết về hợp đồng giao diện và những gì xảy ra "dưới vỏ bọc" của API.
* Vấn đề:
* Tăng cường file mô tả API (OAS):
* Tài liệu hướng dẫn về phía nhà cung cấp:
* Quan trọng:

### 5. Tài liệu hóa các Bản cập nhật và việc Ngừng sử dụng (Documenting Evolutions and Retirement)
Các thay đổi (gây lỗi hoặc không gây lỗi) chắc chắn sẽ xảy ra và phải được tài liệu hóa.

* Nhật ký thay đổi:
* Người tạo thay đổi:
* Nội dung nhật ký thay đổi:
* Vị trí:
* OpenAPI:
* Header `Sunset` (RFC 8594):

## Chương 13: Phát triển API (Growing APIs)
### 13.1 Vòng đời API (The API lifecycle)
#### Vòng đời API mô tả cách một API được sinh ra, tồn tại và cuối cùng là ngừng hoạt động.

### 13.2 Xây dựng hướng dẫn thiết kế API (Building API design guidelines)
#### Việc xác định các hướng dẫn là điều bắt buộc để đảm bảo tính nhất quán trong và giữa các API của tổ chức/nhóm.

* Lợi ích:
* Cấu trúc thành ba lớp:
    * Hướng dẫn tham chiếu (Reference guidelines):
    * Hướng dẫn trường hợp sử dụng (Use case guidelines):
    * Hướng dẫn quy trình thiết kế (Design process guidelines):
* Quá trình xây dựng:

### 13.3 Đánh giá API (Reviewing APIs)

#### API cần được đánh giá ở các giai đoạn khác nhau của vòng đời để đảm bảo chúng hoạt động như dự định.

* Các loại đánh giá:
    * Thách thức và phân tích nhu cầu (Challenging and analyzing needs):
    * Kiểm tra thiết kế (Linting the design):
    * Đánh giá thiết kế từ quan điểm của nhà cung cấp (Reviewing the design from the provider’s perspective):
    * Đánh giá thiết kế từ quan điểm của người tiêu dùng (Reviewing the design from the consumer’s perspective):
    * Xác minh việc triển khai (Verifying the implementation):

### 13.4 Giao tiếp và Chia sẻ (Communicating and Sharing)

#### Các nhà thiết kế API không làm việc một mình; họ cộng tác với nhiều vai trò khác nhau.

* Tính nhất quán:
* Khả năng tìm kiếm:
* Mục tiêu:
