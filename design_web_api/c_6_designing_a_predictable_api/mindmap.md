# Làm cho API của bạn dễ đoán (Making Your API Predictable)

## 6.1. Trở nên nhất quán (Being Consistent)
* Một thiết kế nhất quán giúp API trở nên trực quan bằng cách tận dụng kinh nghiệm trước đây của người dùng.

#### 6.1.1. Thiết kế dữ liệu nhất quán (Designing Consistent Data)
* Dữ liệu là cốt lõi của API.

* **Tên rõ ràng và nhất quán:**
    * Sử dụng cùng một tên cho cùng một khái niệm trên toàn bộ API.
    * **Ví dụ:** `customer_id` thay vì `client_id` hoặc `user_id`.
* **Kiểu dữ liệu và định dạng dễ sử dụng, nhất quán:**
    * Sử dụng các kiểu dữ liệu cơ bản (string, number, date, boolean).
    * Sử dụng cùng một định dạng cho cùng một loại dữ liệu.
    * **Ví dụ:** Chuỗi ISO 8601 cho ngày tháng.
* **Tổ chức dữ liệu nhất quán:**
    * Cấu trúc URL nhất quán (ví dụ: số nhiều cho collection).
    * Cấu trúc danh sách và đối tượng giống nhau.
    * **Ví dụ:**
        ```json
        {
          "items": [
            { "id": 1, "customer_id": 101, "total": 99.99 },
            { "id": 2, "customer_id": 102, "total": 149.99 }
          ]
        }
        ```

#### 6.1.2. Thiết kế các mục tiêu (Goals) nhất quán (Designing Consistent Goals)

* Hành vi của API được xác định bởi các mục tiêu.
* **Tên mục tiêu nhất quán:**
    * Sử dụng động từ nhất quán cho các mục tiêu cùng loại.
    * **Ví dụ:** `read account` và `read user information`.
* **Đầu vào nhất quán:**
    * Tên, kiểu dữ liệu, định dạng và cách tổ chức đầu vào nhất quán.
* **Phản hồi thành công nhất quán:**
    * Cấu trúc phản hồi thành công giống nhau.
    * **Ví dụ:** Mã trạng thái `201 Created` và dữ liệu của tài nguyên vừa tạo.
* **Phản hồi lỗi nhất quán:**
    * Mã trạng thái HTTP nhất quán để báo hiệu lỗi.
    * Dữ liệu thông báo lỗi nhất quán.
* **Luồng mục tiêu nhất quán:**
    * Các hành động nhạy cảm có cùng số lượng bước.

#### 6.1.3. Bốn cấp độ nhất quán (The Four Levels of Consistency)

* **Cấp độ 1: Nhất quán trong một API (Consistency within an API)**
    * Mọi lựa chọn thiết kế không gây ra mâu thuẫn trong API.
    * **Ví dụ:** Tất cả các tài nguyên sử dụng cấu trúc danh sách với trường `items`.
* **Cấp độ 2: Nhất quán giữa các API của một tổ chức/công ty/đội nhóm (Consistency across an organization/company/team’s APIs)**
    * Các API chia sẻ các tính năng chung.
    * **Ví dụ:** Sử dụng cùng một tên thuộc tính (ví dụ: `customer_id`).
* **Cấp độ 3: Nhất quán với (các) lĩnh vực của một API (Consistency with the domain(s) of an API)**
    * Tuân thủ các tiêu chuẩn hoặc quy ước phổ biến trong một lĩnh vực cụ thể.
    * **Ví dụ:** Sử dụng `/cart` cho giỏ hàng trong thương mại điện tử.
* **Cấp độ 4: Nhất quán với phần còn lại của thế giới (Consistency with the rest of the world)**
    * Sử dụng các tiêu chuẩn và quy ước chung đã được chấp nhận rộng rãi.
    * **Ví dụ:** Mã trạng thái HTTP `404 Not Found`.

#### 6.1.4. Sao chép người khác: Tuân thủ các quy tắc chung và tiêu chuẩn (Copying Others: Following Common Practices and Meeting Standards)

* **Tận dụng tiêu chuẩn:** Không cần phát minh lại.
* **Tuân thủ giao thức HTTP:** REST API dựa trên HTTP.
* **Các quy ước phổ biến:** Mẫu URL `/resources/{resourceId}`.
* **Sao chép các API nổi tiếng:** Đơn giản hóa và tạo sự quen thuộc.

#### 6.1.5. Nhất quán là khó và phải được thực hiện một cách khôn ngoan (Being Consistent is Hard and Must be Done Wisely)

* Duy trì sự nhất quán là khó.
* **Giải pháp:** Hướng dẫn thiết kế API (API Design Guidelines).
* **Cân bằng:** Đôi khi, sự nhất quán quá mức có thể làm API trở nên cứng nhắc.

## 6.2. Trở nên thích ứng (Being Adaptable)

### Cho phép người dùng chọn những gì họ muốn nhận.

#### 6.2.1. Cung cấp và chấp nhận các định dạng khác nhau (Providing and Accepting Different Formats)

* API có thể cung cấp dữ liệu ở nhiều định dạng (JSON, CSV, PDF).
* **Cách yêu cầu định dạng:**
    * Tham số truy vấn tùy chỉnh (`format=CSV`).
    * Thỏa thuận nội dung HTTP tiêu chuẩn (header `Accept`).

#### 6.2.2. Quốc tế hóa và địa phương hóa (Internationalizing and Localizing)

* **Ngôn ngữ:** Thông báo lỗi và nội dung bằng nhiều ngôn ngữ.
    * Sử dụng RFC 5646 (Language Tags).
    * Header `Accept-Language`.
* **Địa phương hóa:** Thích ứng với quy ước địa phương (hệ đo lường, định dạng ngày/số).

#### 6.2.3. Lọc, phân trang và sắp xếp (Filtering, Paginating, and Sorting)

* **Phân trang (Pagination):** Lấy một tập hợp con của danh sách lớn.
    * Tham số truy vấn `pageSize` và `page`.
* **Lọc (Filtering):** Chỉ định tiêu chí để nhận các mục cụ thể.
    * Tham số truy vấn (ví dụ: `category=restaurant`).
* **Sắp xếp (Sorting):** Yêu cầu dữ liệu được sắp xếp theo thứ tự cụ thể.

## 6.3. Khám phá (Being Discoverable)
### Cung cấp thông tin bổ sung.
#### 6.3.1. Cung cấp siêu dữ liệu (Providing Metadata)

* **Siêu dữ liệu phân trang:** `page` hiện tại và `totalPages`.
* **Siêu dữ liệu hành động:** Các hành động khả dụng trên một tài nguyên.
* Mục đích: "Tôi đang ở đâu và tôi có thể làm gì".

#### 6.3.2. Tạo API siêu phương tiện (Creating Hypermedia APIs)

* **Liên kết (Links):** URL trong các biểu diễn tài nguyên.
* **Khái niệm:** Hypermedia As The Engine Of Application State (HATEOAS).
* **Lợi ích:** Dễ khám phá và cập nhật API.

#### 6.3.3. Tận dụng giao thức HTTP (Taking Advantage of the HTTP Protocol)

* **Phương thức OPTIONS:** Xác định các phương thức HTTP khả dụng.
* **Header Link:** Chỉ ra các định dạng khác có sẵn cho tài nguyên.