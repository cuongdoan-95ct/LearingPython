# Web API Design: The Missing Link

## Mục tiêu chính: **Cung cấp Các Thực Tiễn Tốt Nhất để Xây Dựng Giao Diện mà Các Nhà Phát Triển Yêu Thích**.

* **Tổng quan**: Không phải hướng dẫn API cho người mới bắt đầu, mà là sự phát triển của tư duy về API gần đây hơn.

### 1. Web APIs và REST

#### 1.1. Công việc của Nhà Thiết Kế APIs

* **Nhiệm vụ**: **Làm cho nhà phát triển ứng dụng thành công nhất có thể**.
* **Góc nhìn**: Suy nghĩ về các lựa chọn thiết kế từ góc nhìn của nhà phát triển ứng dụng.
* **Yếu tố then chốt**: Nhà phát triển ứng dụng.
* **Nguyên tắc chính**: Tối đa hóa năng suất và thành công của nhà phát triển ứng dụng.
* **Tầm quan trọng của thiết kế**: Thiết kế truyền đạt cách một cái gì đó sẽ được sử dụng.

#### 1.2. Web API là gì?

* Là **mô hình các yêu cầu và phản hồi HTTP** được sử dụng để truy cập một trang web được chuyên biệt hóa cho các chương trình máy tính.

#### 1.3. REST là gì?

* Tên cho **phong cách kiến trúc của chính HTTP**.
* **Bản chất**: HTTP là thực tế – REST là một tập hợp các ý tưởng thiết kế đã định hình nó.
* **Tầm quan trọng**: Giúp chúng ta hiểu cách suy nghĩ về HTTP và cách sử dụng nó.
* **Thực tế hiện đại**: Hầu hết các API sử dụng tập hợp con HTTP pha trộn với khái niệm RPC (ví dụ: "endpoints", "parameters").
* **Thuật ngữ**: **RESTful APIs** dùng nhiều khái niệm HTTP gốc hơn.
* **Apigee ủng hộ**: **Chỉ sử dụng HTTP mà không thêm các khái niệm bổ sung** nhiều nhất có thể.
    * **Lý do**:
        * Đặt mình vào vị trí người dùng (đã có kiến thức HTTP).
        * Tuân thủ các tiêu chuẩn và quy ước đã được thiết lập (HTTP là chuẩn tốt nhất).
    * **Ví dụ về tuân thủ chuẩn HTTP**:
        * **POST tạo tài nguyên**: Kèm **`Location` header** và **`201 Created`**.
        * **Kiểm tra đồng thời**: Dùng **`ETag` và `If-Match`**.
        * **Định dạng dữ liệu khác nhau**: Dùng **`HTTP Accept` header**.
    * **Lựa chọn thay thế**: Cung cấp **ngoài việc hỗ trợ cơ chế chuẩn**, không phải thay thế.
* **Chủ nghĩa tối giản về khái niệm**: Hạn chế sử dụng khái niệm HTTP để giữ sự đơn giản.
* **Giảm thiểu sự gắn kết (coupling)**: Giảm gắn kết giữa client và server. Khái niệm lớp từ RPC làm suy yếu gắn kết lỏng lẻo của HTTP.

### 2. HTTP và REST: Một mô hình thiết kế hướng dữ liệu

* Một REST API tập trung vào **các thực thể cơ bản của miền vấn đề** mà nó phơi bày.
    * **Ví dụ**: Tập hợp các con chó (`https://dogtracker.com/dogs`), các con chó riêng lẻ.
* **Tính đồng nhất (Uniformity)**: Khi sử dụng HTTP một cách tự nhiên, HTTP cung cấp API, bạn chỉ cần định nghĩa dữ liệu trong tài nguyên.
* **Ràng buộc giao diện đồng nhất (uniform interface constraint)**: Cho phép phần mềm phổ quát (trình duyệt, bot tìm kiếm) hoạt động với bất kỳ trang web nào.
* **So sánh với API hướng chức năng**: API hướng chức năng có nhiều sự thay đổi, không có cấu trúc rõ ràng, kiến thức khó tái sử dụng.

### 3. Các Yếu Tố Thiết Kế API

#### Các khía cạnh sau đây đều quan trọng và cùng nhau định nghĩa API của bạn:

* **Biểu diễn tài nguyên**: Định nghĩa các trường và liên kết.
* **Sử dụng HTTP headers**: Tiêu chuẩn và tùy chỉnh.
* **URL và URI template**: Định nghĩa giao diện truy vấn để định vị tài nguyên.
* **Hành vi bắt buộc của client**: Caching DNS, thử lại, chấp nhận các trường mới.

### 4. Thiết Kế Biểu Diễn (Representations)

#### Trong mô hình hướng dữ liệu như REST, tốt hơn nên bắt đầu với **thiết kế biểu diễn** trước khi thiết kế URL.

* **Biểu diễn (Representation)**: Dữ liệu được trả về/gửi đi khi một tài nguyên web được client truy xuất/gửi đến server.
* **Kiểu phương tiện (media types)**: Các định dạng khác nhau của biểu diễn (ví dụ: JSON).

#### 4.1. Sử dụng JSON

* **JSON (JavaScript Object Notation)**: Kiểu phương tiện chiếm ưu thế.
* **Lý do thành công**: Đơn giản, dễ ánh xạ tới cấu trúc dữ liệu lập trình.
* **Tiêu chuẩn thực tế (de facto standard)**.
* **Hạn chế**: Chỉ biểu diễn ít kiểu dữ liệu (Null, Boolean, Number, String). Ngày, giờ, URL không được hỗ trợ trực tiếp (biểu diễn dưới dạng chuỗi).
* **Giữ JSON đơn giản**.

#### 4.2. Biểu diễn các mối quan hệ trong liên kết (links)

* **Cấu trúc tự nhiên**: **Liên kết (link)**.
* **Liên quan đến**: **Hypermedia As The Engine Of Application State (HATEOAS)**.
* **Lợi ích**: **Cải thiện khả năng sử dụng và học hỏi** của tất cả các API.
* **Ví dụ**:
    * **Không có liên kết**: `{ "ownerID": "98765432" }` (phải dò tài liệu, tự tạo URL).
    * **Với liên kết rõ ràng**: Thêm thuộc tính URL như `ownerLink`, `dogsLink` (URL có sẵn, client dễ viết code, dùng code đa năng).
* Xu hướng: Nhiều API lớn (Google, GitHub) bao gồm liên kết trong dữ liệu.

#### 4.3. URI templates có còn cần thiết khi có links?

* **Có, chúng vẫn quan trọng**.
* **Liên kết**: Diễn tả nơi bạn có thể đi từ vị trí hiện tại (các con đường đã lát).
* **URI templates**: Giống lối tắt, cách đi thẳng đến đích cụ thể. Định vị tài nguyên khi không có liên kết trực tiếp hoặc đường dẫn quá dài.
* **Hữu ích nhất khi**: Chấp nhận các biến có giá trị dễ đọc đối với con người.
* **Chức năng**: Ngôn ngữ truy vấn chuyên biệt cho API.
* **Kết hợp**: Hai kỹ thuật bổ sung có giá trị.

#### 4.4. URL tuyệt đối so với tương đối trong biểu diễn

* **Vấn đề**: Lưu URL tuyệt đối trong DB có thể gây vấn đề trong các môi trường khác nhau.
* **Giải pháp hợp lý (Server-side)**: Server nên loại bỏ scheme và authority từ URL tự định danh trước khi lưu trữ và khôi phục khi được yêu cầu. Chấp nhận và tạo **URL tương đối** (đặc biệt là tuyệt đối đường dẫn, bắt đầu bằng `/`).
* **Đánh đổi của URL tương đối**: Đẩy gánh nặng cho client.
* **Ưu tiên**: Cung cấp URL tuyệt đối trên server sẽ tạo ra một API tốt hơn cho người dùng.

#### 4.5. Cách biểu diễn liên kết trong tài nguyên

* **Cách ưa thích**: Sử dụng một **cặp tên/giá trị JSON đơn giản**, ví dụ: `"ownerLink": "https://dogtracker.com/persons/98765432"`.
* **Mẫu phức tạp hơn**: Có, nhưng không yêu cầu.
* **Ví dụ từ GitHub**: URI Templates trong các giá trị liên kết (ví dụ: `"events_url": "https://api.github.com/users/octocat/events{/privacy}"`) truyền tải URL của một "họ" tài nguyên.

### 5. Thiết Kế URL

#### 5.1. Danh từ là tốt; động từ là xấu

* **Mô hình hướng dữ liệu**: Mỗi URL xác định một "thứ" (thing).
* **Quy tắc**: URL nên được hình thành từ **danh từ**.

#### 5.2. Các URL đã biết (Well-known URLs)

* Cần công bố ít nhất một URL đã biết để client có thể bắt đầu (ví dụ: `https://dogtracker.com/dogs`).

#### 5.3. Thiết kế URL thực thể (Entity URLs)

* **Client không nên phải xây dựng URL của các thực thể riêng lẻ**. URL của tài nguyên mới được tạo sẽ được trả về cho client.
* **URL mờ (opaque URLs)**: Không gây khó khăn cho máy tính nhưng con người thấy khó.
* **Mong muốn**: Định nghĩa **URL thực thể thân thiện với con người** (ví dụ: `https://dogtracker.com/dogs/a8098c1a`).
* **Dấu gạch chéo (`/`)**: Thường dùng để tách kiểu khỏi ID tài nguyên (`/{type}/{uuid}`).

#### 5.4. Web là phẳng (The web is flat)

* **Tránh mã hóa các hệ thống phân cấp sâu trong URL liên kết**.
* **URL truy vấn**: Có thể mã hóa phân cấp.
* **URL liên kết**: Mã hóa phân cấp là **ý tưởng tồi** (không ổn định, dễ hỏng liên kết).

#### 5.5. Giải pháp cho tình huống tiến thoái lưỡng nan về đổi tên

* **Vấn đề**: ID thân thiện với con người (tên) dễ dùng nhưng dễ hỏng liên kết khi đổi tên. ID do máy tạo (UUID) cho phép đổi tên tự do nhưng URL không thân thiện.
* **Giải pháp**:
    * **URL cho liên kết**: Dùng ID do máy tạo (mờ đối với client).
    * **URI templates cho truy vấn**: Dùng ID thân thiện với con người (ví dụ: `https://dogtracker.com/person/{name}`).

### 6. Thiết Kế URL Truy Vấn (Query URLs)

* **Mục tiêu**: Thiết kế các URL truy vấn có tính đều đặn và dễ đoán.

#### 6.1. Mô hình chung cho URL truy vấn

* **Mẫu ưa thích cho duyệt mối quan hệ**: `https://dogtracker.com/persons/{personId}/dogs`.
    * **Ưu tiên hơn**: `/search?type=Dog&owner={personId}`.
    * **Lý do**: Dễ đọc/trực quan, dễ triển khai, không cam kết ngôn ngữ truy vấn toàn diện.
    * Hữu ích cho chuỗi duyệt mối quan hệ (`/dogs/123456/owner/spouse`).
* **Biểu diễn mối quan hệ trong URL truy vấn**: Sử dụng các đoạn đường dẫn (path segments).
    * Mô hình chung: `/{relationship-name}[/{resource-id}]/.../{relationship-name}[/{resource-id}]`.
    * `POST` tới URL này có thể là cách viết tắt để tạo tài nguyên và thiết lập mối quan hệ.
* **Biểu diễn mối quan hệ đối xứng trong URL và biểu diễn**:
    * Nếu URL truy vấn ngụ ý mối quan hệ, hãy làm rõ trong biểu diễn tài nguyên bằng thuộc tính mối quan hệ có giá trị URL.
    * Ngược lại, nếu thuộc tính mối quan hệ có trong tài nguyên, hãy cung cấp URL truy vấn cho phép duyệt.
* **Giá trị của mô hình**: URL truy vấn dễ đoán, thống nhất với mô hình gốc duy nhất có liên kết của WWW.

#### 6.2. Path parameters, hoặc matrix parameters

* Cú pháp thay thế: `/persons;5678/dogs`.
* Quy tắc tổng quát: `/{relationship-name}[;{selector}]/.../{relationship-name}[;{selector}]`.
* Ưu điểm: Gọn gàng về cú pháp, rõ ràng về khái niệm.

#### 6.3. Lọc các tập hợp (Filtering collections)

* Các mệnh đề truy vấn phức tạp hơn được đặt sau dấu `?` trong chuỗi truy vấn của URL (ví dụ: `/dogs?color=red&state=running`).
* Có thể kết hợp URL truy vấn dựa trên đường dẫn và tham số chuỗi truy vấn.

### 7. Các phản hồi không liên quan đến tài nguyên bền vững

* **Nguyên tắc**: **URL nên xác định tài nguyên trong phản hồi, không phải thuật toán xử lý**.
* **Từ góc nhìn của client**: Việc kết quả được tính toán hay truy xuất từ cơ sở dữ liệu là không liên quan.

### 8. Bao gồm thuộc tính `kind`

* Thuộc tính `kind` giúp client nhận ra liệu đây có phải là một đối tượng mà họ biết cách xử lý hay không.
* Lợi ích: Thêm khái niệm mới vào API mà không cần thay đổi phiên bản.
* **Sử dụng URL cho các kiểu** (ví dụ: `https://apigee.com/collections#Collection`) giúp tránh xung đột tên.

### 9. Cách biểu diễn các tập hợp (Collections)

* Các tập hợp là tài nguyên quan trọng (danh sách các thứ, nơi POST để tạo tài nguyên mới).
* Các kiểu phương tiện được chuẩn hóa bằng cách đăng ký ở một nơi trung tâm; các kiểu tài nguyên có thể sử dụng URL.

### 10. PATCH so với PUT để cập nhật

* **Sử dụng PATCH để cập nhật** vì nó giúp ích cho sự phát triển của API.
* **PUT**: Yêu cầu client thay thế toàn bộ nội dung (dễ vỡ).
* **PATCH**: Tránh được rủi ro này (ngữ nghĩa `UPDATE` của SQL).

### 11. API "chatty" (nhiều cuộc gọi nhỏ)

* **Quan niệm sai lầm phổ biến**: Myth rằng REST API là "chatty". **API "chatty" là do bạn đã thiết kế sai tài nguyên**, không phải do REST.
* **Chiến lược hữu ích**: Định nghĩa tất cả các tài nguyên liên quan đến lược đồ chuẩn hóa (đọc và cập nhật) và sau đó **thêm nhiều tài nguyên phi chuẩn hóa (denormalized resources) chỉ đọc** khi cần để hỗ trợ các client hiệu quả.

### 12. Phân trang và phản hồi một phần (Pagination and partial response)

* **Phản hồi một phần**: Cung cấp cho nhà phát triển ứng dụng **chỉ thông tin họ cần**.
* Apigee thích `limit` và `offset` cho phân trang.

### 13. Xử Lý Lỗi (Handling Errors)

* Chất lượng xử lý lỗi là một **phần quan trọng của trải nghiệm API** cho tất cả các client.
* **Tầm quan trọng**: Lỗi là công cụ quan trọng cung cấp ngữ cảnh và khả năng hiển thị, giúp nhà phát triển học và khắc phục sự cố.
* **Lời khuyên**: Học và sử dụng **các mã trạng thái HTTP tiêu chuẩn một cách thích hợp**.
* **Thông điệp trong phần thân phản hồi (payload)**: Nên **dài dòng (verbose) và sử dụng ngôn ngữ đơn giản**.
    * Bao gồm: `developerMessage`, `userMessage`, `errorCode`, `more info` (liên kết tài liệu).
    * Đối tượng thực sự: **user agent** (chương trình trung gian).

### 14. Mô hình hóa Hành Động (Modeling Actions)

#### Để kích hoạt một hành động mà vẫn tuân thủ mô hình hướng thực thể, dựa trên danh từ:

* **Tùy chọn 1**: Cung cấp một **thuộc tính trạng thái (state property)** của tiến trình và cho phép client đặt nó.
* **Tùy chọn 2**: Cho phép client **POST một yêu cầu hành động (action request) đến một URL liên quan**.
    * Ví dụ: Client **POST** một thực thể `StopRequest`, `StartRequest`, v.v.
* **Tùy chọn 3 (biến thể của 2)**: Cung cấp **các thuộc tính riêng biệt cho từng loại yêu cầu**.
    * Sự hiện diện/vắng mặt của thuộc tính truyền đạt tính hợp lệ của yêu cầu.

### 15. Xác Thực (Authentication)

* **OAuth2**: Tiêu chuẩn được sử dụng bởi hầu hết mọi Web API lớn. **Bạn nên sử dụng nó**.
* **Lợi ích của OAuth 2.0**:
    * Ứng dụng web/di động không cần chia sẻ mật khẩu.
    * Nhà cung cấp API có thể thu hồi token (cải thiện bảo mật).
    * Cải thiện bảo mật và trải nghiệm người dùng cuối tốt hơn.

### 16. Bổ Trợ bằng SDK (Software Development Kit)

* Hầu hết các lập trình viên client đều thích sử dụng một **SDK được xây dựng tốt** hơn là tiêu thụ trực tiếp API web.
* Lập trình viên có thể coi SDK là API, bỏ qua Web API cơ bản.
* **Tầm quan trọng của Web API cơ bản**: Ảnh hưởng lớn đến chi phí, độ tin cậy của SDK và số lượng SDK được tạo ra.
* Web API cũng hiển thị cho lập trình viên trong các tình huống gỡ lỗi và điều chỉnh hiệu suất.

### 17. Quản lý Phiên bản (Versioning)

#### Việc quản lý phiên bản API là một chủ đề gây tranh cãi.

#### 17.1. Cách thực hành phổ biến nhất

* Đặt **định danh phiên bản trong một đoạn đường dẫn của URL** (ví dụ: `/v1/dogs/12345678`).

#### 17.2. Không làm gì cả để quản lý phiên bản API là một cách tiếp cận thông minh

* Nhiều thay đổi có thể được thực hiện một cách **tương thích ngược**.
    * Ví dụ: An toàn khi thêm các thuộc tính mới, sử dụng **PATCH thay vì PUT để cập nhật**.
* Thay đổi cơ bản lớn sẽ làm hỏng client cũ, đòi hỏi API mới.
* Thường thì **loại thay đổi "ở giữa" không tồn tại**.
* Dễ dàng thêm quản lý phiên bản sau này (yêu cầu thiếu định danh phiên bản được coi là v1).
* **Nếu phân vân, an toàn không đưa quản lý phiên bản vào ngay từ đầu**.

#### 17.3. Liên kết (Links) và định danh phiên bản trong URL không phù hợp

* Nếu có quản lý phiên bản và sử dụng liên kết, **đặt thông tin phiên bản trong một header** (ví dụ: `Accept-Version`) là đơn giản hơn.
* Quản lý phiên bản bằng URL cùng với liên kết sẽ phức tạp hơn.
* Việc không triển khai quản lý phiên bản vẫn là đơn giản nhất.

### 18. Kết luận

* Thiết kế Web API đang không ngừng phát triển và ngày càng quan trọng về mặt thương mại.
* **Nguyên tắc cốt lõi**: Hạn chế bản thân nhiều nhất có thể với các khái niệm và kỹ thuật có trong các đặc tả HTTP và URI.
* **Hướng thực thể (entity-oriented)**: Tập trung vào dữ liệu và để HTTP cung cấp API đồng nhất.
* **Nỗ lực thiết kế API**: Phần lớn dành cho **đặc tả định dạng dữ liệu**.
* **Nỗ lực đáng kể khác**: Thiết kế URL thể hiện truy vấn.
* **Giá trị ngày càng tăng**: **Liên kết trong API** cho tất cả các client.
* **Quản lý phiên bản**: **"Không làm gì cả"** đã trở thành một lựa chọn hợp lý và hoạt động tốt.
