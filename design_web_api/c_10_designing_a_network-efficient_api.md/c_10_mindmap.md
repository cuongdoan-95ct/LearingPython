# Thiết kế API hiệu quả mạng (Designing a network-efficient API)

### 1. Tổng quan về các vấn đề truyền thông mạng

#### Hiệu quả truyền thông mạng là một yếu tố **quan trọng mà bất kỳ nhà thiết kế API nào cũng phải nắm rõ**.

* **Tầm quan trọng**: API không hiệu quả gây chậm giao diện người dùng, tiêu hao pin, tăng băng thông mạng, tăng chi phí hạ tầng.
* **Ví dụ minh họa**: Ứng dụng Ngân hàng Tuyệt vời trên di động với mạng 3G không tốt tiêu thụ API Ngân hàng được sửa đổi nhẹ.
* **Phân tích cuộc gọi API**:
    * **Độ trễ (Kết nối tới máy chủ)**: Khoảng **300 ms**.
    * **Gửi yêu cầu**: Rất nhanh (dưới 1 ms cho 100 byte).
    * **Xử lý phía máy chủ**: Giả định 20 ms.
    * **Tải xuống phản hồi**: Phụ thuộc kích thước và tốc độ (ví dụ: 32 KB mất 200 ms).
    * **Tổng thời gian một yêu cầu**: **520 ms**.
* **Phân tích vấn đề trong các trường hợp sử dụng**:
    * **Trường hợp đơn giản (1 chủ sở hữu, 1 tài khoản)**: **5 cuộc gọi API** (3 bước), tổng **1.2 giây** (vượt ngưỡng 500 ms chấp nhận được).
    * **Trường hợp phức tạp (4 chủ sở hữu, 8 tài khoản)**: **25 cuộc gọi API** (7 bước), tổng **4.2 giây**.
    * **Người dùng trung bình**: Khoảng **250 cuộc gọi API/phiên**, trao đổi **2 MB dữ liệu/phiên**, dẫn đến **60-90 MB dữ liệu/tháng**.
    * **Mối quan tâm của nhà cung cấp**: Chi phí dịch vụ đám mây dựa trên số lượng cuộc gọi, thời gian xử lý, khối lượng dữ liệu gửi đi.
* **Các khía cạnh của hiệu quả mạng**: Tốc độ, khối lượng dữ liệu và số lượng cuộc gọi.

### 2. Đảm bảo hiệu quả truyền thông mạng ở cấp độ giao thức

#### Các tối ưu hóa ở cấp độ giao thức có thể được thực hiện **mà không ảnh hưởng nhiều đến thiết kế lý tưởng** của API.

* **Kích hoạt nén và kết nối liên tục**:
    * **Nén (Compression)**: Giảm khối lượng dữ liệu (ví dụ: 310 KB xuống dưới 2 KB), tải xuống nhanh hơn, sử dụng ít dữ liệu hơn. Hầu hết thư viện HTTP hỗ trợ **minh bạch**.
    * **Kết nối liên tục (Persistent Connections)**: Giảm độ trễ bằng cách giữ kết nối mở (ví dụ: loại bỏ 6 * 300 ms độ trễ).
    * **HTTP/2**: Cung cấp **kết nối liên tục hiệu quả, các yêu cầu song song và truyền nhị phân**, **tương thích ngược** với HTTP/1.1 và hoàn toàn **minh bạch**.
    * **Vai trò của nhà thiết kế API**: Nắm rõ các tối ưu hóa này để đề xuất giải pháp hiệu suất, **lý tưởng nhất là ngay từ đầu**.
* **Kích hoạt bộ nhớ đệm (Caching) và yêu cầu có điều kiện (Conditional Requests)**:
    * **Mục tiêu**: **Giảm số lượng truyền thông** hoặc không truyền thông gì cả.
    * **Caching**: Lưu trữ phản hồi API để tái sử dụng, tránh các cuộc gọi không cần thiết.
    * **HTTP Caching (RFC 7234)**: API sử dụng `Cache-Control`. Người dùng sử dụng `If-None-Match` (với `ETag`) hoặc `If-Modified-Since` (với `Last-Modified`).
    * **Hạn chế của REST: Caching**: Yêu cầu phản hồi phải cho biết khả năng lưu vào bộ nhớ đệm. gRPC có thể không có caching gốc.
    * **Lựa chọn chính sách bộ nhớ đệm**: Vấn đề phức tạp, cần đánh giá từng mục tiêu và thuộc tính, thường đòi hỏi tư vấn từ đội an ninh và pháp lý.

### 3. Đảm bảo hiệu quả truyền thông mạng ở cấp độ thiết kế

#### **Thiết kế API cơ bản quyết định số lượng cuộc gọi** và lượng dữ liệu trao đổi giữa người dùng và nhà cung cấp.

* **Kích hoạt tính năng lọc (Filtering)**:
    * Cho phép người dùng yêu cầu **chỉ dữ liệu họ thực sự cần**, cải thiện hiệu quả và khả năng sử dụng.
    * **Phân trang dựa trên offset**: Tham số truy vấn `page` và `size`.
    * **Phân trang dựa trên con trỏ**: Dùng `before` và `after` (ID giao dịch) để truy xuất giao dịch mới/cũ hơn. Hiệu quả hơn với danh sách cập nhật liên tục.
    * **Lọc theo tiêu chí**: Ví dụ: `category=restaurant`.
* **Lựa chọn dữ liệu phù hợp cho các biểu diễn danh sách**:
    * **Biểu diễn tóm tắt (summarized) so với biểu diễn đầy đủ (complete)**.
    * **Vấn đề**: Dữ liệu tóm tắt không đủ dẫn đến các cuộc gọi bổ sung.
    * **Giải pháp**: **Bao gồm các thuộc tính đại diện và hữu ích hơn trong danh sách tóm tắt**. Đôi khi biểu diễn **đầy đủ** hiệu quả hơn.
* **Tổng hợp dữ liệu (Aggregating data)**:
    * **Khái niệm**: Kết hợp dữ liệu liên quan chặt chẽ vào một phản hồi duy nhất để giảm các cuộc gọi API.
    * **Ví dụ**: Tổng hợp địa chỉ vào tài nguyên chủ sở hữu (`read owner` goal) **giảm số cuộc gọi từ 2 xuống 1**.
    * **Tổng hợp sâu hơn**: Kết hợp tất cả dữ liệu tài khoản vào danh sách tài khoản, sau đó vào tài nguyên chủ sở hữu. Có thể **giảm đáng kể độ trễ và khối lượng dữ liệu**.
    * **Đánh đổi**: Có thể **cản trở caching**, các cuộc gọi đơn lẻ kéo dài làm **tăng nguy cơ mất kết nối**. Có thể ảnh hưởng đến khả năng sử dụng.
* **Đề xuất các biểu diễn khác nhau (Content Negotiation)**:
    * Cho phép người dùng **chọn biểu diễn phù hợp nhất** (`summarized`, `complete`, `extended`) bằng tiêu đề `Accept` và kiểu phương tiện tùy chỉnh.
* **Kích hoạt mở rộng (Enabling Expansion)**:
    * Cho phép người dùng chỉ định **những tài nguyên phụ nào họ muốn bao gồm** trong phản hồi của tài nguyên chính (ví dụ: `GET /owners?_expand=accounts`).
    * Giảm số lượng cuộc gọi cần thiết để truy xuất cây dữ liệu.
* **Kích hoạt truy vấn (Enabling Querying)**:
    * Cho phép người dùng truy vấn dữ liệu **từng thuộc tính một** để giảm khối lượng dữ liệu (ví dụ: `GET /owners?_fields=id`).
    * **GraphQL**: Ngôn ngữ truy vấn cho API cho phép người dùng chỉ định **chính xác dữ liệu họ muốn** và thực hiện **nhiều truy vấn trong một cuộc gọi duy nhất**.
        * **Ưu điểm**: Lựa chọn dữ liệu linh hoạt cao, một cuộc gọi cho nhiều truy vấn.
        * **Nhược điểm**: Dùng POST (không có caching HTTP tiêu chuẩn), không có caching gốc.
* **Cung cấp dữ liệu và mục tiêu phù hợp hơn**:
    * Giao tiếp không hiệu quả có thể là dấu hiệu của thiết kế không đáp ứng đủ nhu cầu thực tế.
    * **Thêm dữ liệu liên quan**: Bao gồm dữ liệu phái sinh được sử dụng phổ biến trực tiếp (ví dụ: `balance` trong danh sách `transactions`).
    * **Thêm mục tiêu liên quan**: Cung cấp điểm truy cập trực tiếp cho các trường hợp sử dụng phổ biến (ví dụ: `GET /dashboards/me`).
    * **Đánh giá hiệu quả**: Kiểm tra thiết kế với các **trường hợp sử dụng thực tế và trường hợp biên (edge cases)**.
* **Tạo các lớp API khác nhau (API Layers)**:
    * **Khi nào nên nói "không"**: Đừng tối ưu hóa một API duy nhất mà hy sinh khả năng sử dụng và tái sử dụng cho tất cả người dùng.
    * **Giải pháp**: Xây dựng các API chuyên biệt (ví dụ: **BFF - Backend For Frontend**) dựa trên các API hiện có. Nhà cung cấp có thể cung cấp **"API trải nghiệm" (experience APIs)**.
    * **Các lớp API**:
        * **Experience APIs**: Tối ưu hóa cho ngữ cảnh/người dùng cụ thể (ví dụ: di động).
        * **Original/Not Specialized APIs**: Hướng tới người dùng, mục đích chung.
        * **System APIs**: Truy cập vào các hệ thống cốt lõi.
