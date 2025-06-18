## Designing a network-efficient API
### 1. Tổng quan về các vấn đề truyền thông mạng

*   **Tầm quan trọng**: Hiệu quả truyền thông mạng là một yếu tố **quan trọng mà bất kỳ nhà thiết kế API nào cũng phải nắm rõ**. Các API không hiệu quả có thể dẫn đến giao diện người dùng chậm, tiêu hao pin trên thiết bị di động, và sử dụng băng thông mạng cao, gây tăng chi phí cho hạ tầng đám mây hoặc tắc nghẽn cho các hệ thống tại chỗ (on-premise).
*   **Ví dụ minh họa**: Chương này sử dụng một ví dụ về "Ứng dụng Ngân hàng Tuyệt vời" (Awesome Banking App) trên điện thoại di động kết nối mạng **3G không tốt** để minh họa các vấn đề khi tiêu thụ một "API Ngân hàng được sửa đổi nhẹ".
*   **Phân tích cuộc gọi API**:
    *   **Độ trễ (Kết nối tới máy chủ)**: Khoảng **300 ms**, có thể lên đến vài giây tùy thuộc vào loại mạng và chất lượng tín hiệu.
    *   **Gửi yêu cầu**: Rất nhanh (ví dụ: 100 byte mất chưa đến 1 ms).
    *   **Xử lý phía máy chủ**: Giả định 20 ms, thời gian thực tế phụ thuộc vào yêu cầu và khả năng của máy chủ.
    *   **Tải xuống phản hồi**: Phụ thuộc vào kích thước phản hồi và tốc độ tải xuống (ví dụ: 32 KB với 160 KB/s mất 200 ms).
    *   **Tổng thời gian**: Một yêu cầu đơn lẻ có thể mất **520 ms**.
*   **Phân tích vấn đề trong các trường hợp sử dụng**:
    *   **Trường hợp sử dụng đơn giản (1 chủ sở hữu, 1 tài khoản)**: Mất **5 cuộc gọi API** trong 3 bước, tổng cộng **1.2 giây**. Con số này đã vượt quá **500 ms** – ngưỡng chấp nhận được về độ trễ của não người.
    *   **Trường hợp sử dụng phức tạp (4 chủ sở hữu, 8 tài khoản)**: Mất **25 cuộc gọi API** trong 7 bước, tổng cộng **4.2 giây**.
    *   **Người dùng trung bình (2 chủ sở hữu, 2 tài khoản, duyệt tất cả dữ liệu)**: Khoảng **250 cuộc gọi API** mỗi phiên, trao đổi **2 MB dữ liệu**, dẫn đến **60-90 MB dữ liệu mỗi tháng**.
    *   **Mối quan tâm của nhà cung cấp**: Chi phí dịch vụ đám mây dựa trên số lượng cuộc gọi nhận được, thời gian xử lý và khối lượng dữ liệu gửi đi.
*   **Các khía cạnh của hiệu quả mạng**: Tốc độ, khối lượng dữ liệu và số lượng cuộc gọi.

### 2. Đảm bảo hiệu quả truyền thông mạng ở cấp độ giao thức

Các tối ưu hóa ở cấp độ giao thức có thể được thực hiện **mà không ảnh hưởng nhiều đến thiết kế lý tưởng** của API.

*   **Kích hoạt nén và kết nối liên tục**:
    *   **Nén (Compression)**: Giảm khối lượng dữ liệu (ví dụ: từ 310 KB xuống dưới 2 KB), dẫn đến tải xuống nhanh hơn và sử dụng ít dữ liệu hơn. Điều này mang lại lợi ích cho cả người dùng (pin, gói dữ liệu) và nhà cung cấp (tắc nghẽn mạng, hóa đơn đám mây). Hầu hết các thư viện HTTP tiêu chuẩn đều hỗ trợ tính năng này một cách **minh bạch**.
    *   **Kết nối liên tục (Persistent Connections)**: Giảm độ trễ bằng cách giữ kết nối mở cho các cuộc gọi tiếp theo. Ví dụ, có thể loại bỏ 6 * 300 ms độ trễ, giảm tổng thời gian từ 4.2 giây xuống 2.4 giây.
    *   **HTTP/2**: Cung cấp **kết nối liên tục hiệu quả, các yêu cầu song song và truyền nhị phân**, đồng thời **tương thích ngược** với HTTP/1.1, hoàn toàn **minh bạch** đối với thiết kế API.
    *   **Vai trò của nhà thiết kế API**: Cần nắm rõ các tối ưu hóa này để đề xuất giải pháp cho các vấn đề hiệu suất mạng, **lý tưởng nhất là ngay từ đầu**.
*   **Kích hoạt bộ nhớ đệm (Caching) và yêu cầu có điều kiện (Conditional Requests)**:
    *   **Mục tiêu**: **Giảm số lượng truyền thông** hoặc không truyền thông gì cả.
    *   **Caching**: Lưu trữ các phản hồi API để tái sử dụng, tránh các cuộc gọi không cần thiết. Có thể giảm đáng kể lưu lượng truy cập (ví dụ: giảm 46% cuộc gọi và 50% khối lượng dữ liệu trong một trường hợp).
    *   **HTTP Caching (RFC 7234)**: API có thể chỉ ra liệu phản hồi có thể được lưu vào bộ nhớ đệm và trong bao lâu bằng cách sử dụng tiêu đề `Cache-Control`. Người dùng có thể sử dụng các tiêu đề `If-None-Match` (với `ETag`) hoặc `If-Modified-Since` (với `Last-Modified`) cho các yêu cầu có điều kiện để kiểm tra xem dữ liệu họ có còn mới hay không.
    *   **Hạn chế của REST: Caching**: Yêu cầu phản hồi phải cho biết khả năng lưu vào bộ nhớ đệm. Tuy nhiên, người dùng có thể không thực sự lưu vào bộ nhớ đệm. Một số giao thức (ví dụ: gRPC) có thể không cung cấp tính năng caching gốc.
    *   **Lựa chọn chính sách bộ nhớ đệm**:
        *   Đây là một vấn đề phức tạp vì các chính sách (ví dụ: thời gian lưu trữ bộ nhớ đệm) cần được đánh giá cho từng mục tiêu và từng thuộc tính, thường đòi hỏi sự tư vấn từ các đội an ninh và pháp lý.

### 3. Đảm bảo hiệu quả truyền thông mạng ở cấp độ thiết kế

**Thiết kế API cơ bản quyết định số lượng cuộc gọi** mà người dùng cần thực hiện để đạt được mục tiêu của họ và lượng dữ liệu được trao đổi giữa người dùng và nhà cung cấp.

*   **Kích hoạt tính năng lọc (Filtering)**:
    *   Cho phép người dùng yêu cầu **chỉ dữ liệu họ thực sự cần**, cải thiện hiệu quả và khả năng sử dụng.
    *   **Phân trang dựa trên offset (Offset-based pagination)**: Tham số truy vấn `page` và `size`.
    *   **Phân trang dựa trên con trỏ (Cursor-based pagination)**: Sử dụng các giá trị con trỏ `before` và `after` (ID giao dịch) để truy xuất các giao dịch mới hoặc cũ hơn. Hiệu quả hơn đối với các danh sách cập nhật liên tục vì nó tránh trùng lặp.
    *   **Lọc theo tiêu chí**: Ví dụ: `category=restaurant` để lọc giao dịch.
*   **Lựa chọn dữ liệu phù hợp cho các biểu diễn danh sách**:
    *   **Biểu diễn tóm tắt (summarized) so với biểu diễn đầy đủ (complete)**.
    *   **Vấn đề**: Nếu dữ liệu tóm tắt (ví dụ: tên chủ sở hữu trong danh sách) không đủ, nó sẽ dẫn đến các cuộc gọi bổ sung (ví dụ: `read owner` để lấy tiêu đề).
    *   **Giải pháp**: **Bao gồm các thuộc tính đại diện và hữu ích hơn trong danh sách tóm tắt** để tránh các cuộc gọi bổ sung. Đôi khi, một biểu diễn **đầy đủ** trong danh sách lại hiệu quả hơn (ví dụ: giao dịch).
*   **Tổng hợp dữ liệu (Aggregating data)**:
    *   **Khái niệm**: Kết hợp dữ liệu liên quan chặt chẽ (ví dụ: chủ sở hữu + địa chỉ) vào một phản hồi duy nhất để giảm các cuộc gọi API.
    *   **Ví dụ**: Tổng hợp tài nguyên phụ là địa chỉ vào tài nguyên chủ sở hữu (`read owner` goal) **giảm số cuộc gọi từ 2 xuống 1**.
    *   **Tổng hợp sâu hơn**: Kết hợp tất cả dữ liệu tài khoản (ngoại trừ giao dịch) vào danh sách tài khoản, sau đó vào tài nguyên chủ sở hữu trong danh sách chủ sở hữu. Điều này có thể **giảm đáng kể độ trễ và khối lượng dữ liệu** (ví dụ: 17 cuộc gọi trong 6 bước, 2.2 giây, 58KB -> 1 cuộc gọi, 700ms, 52KB).
    *   **Đánh đổi**: Có thể **cản trở việc caching** (TTL của dữ liệu tổng hợp là giá trị nhỏ nhất của các phần của nó), các cuộc gọi đơn lẻ kéo dài làm **tăng nguy cơ mất kết nối** trên các mạng không ổn định. Có thể ảnh hưởng đến khả năng sử dụng bằng cách làm cho API phức tạp.
*   **Đề xuất các biểu diễn khác nhau (Content Negotiation)**:
    *   Cho phép người dùng **chọn biểu diễn phù hợp nhất** với nhu cầu của họ (ví dụ: `summarized`, `complete`, `extended`) bằng cách sử dụng tiêu đề `Accept` và các kiểu phương tiện tùy chỉnh (ví dụ: `application/vnd.bankingapi.extended+json`).
    *   Cung cấp sự linh hoạt cho các nhu cầu khác nhau của người dùng (ví dụ: ứng dụng di động cần `extended` so với ứng dụng máy tính cần `summarized`).
*   **Kích hoạt mở rộng (Enabling Expansion)**:
    *   Cho phép người dùng chỉ định **những tài nguyên phụ nào họ muốn bao gồm** trong phản hồi của tài nguyên chính (ví dụ: `GET /owners?_expand=accounts`).
    *   Giảm số lượng cuộc gọi cần thiết để truy xuất cây dữ liệu.
*   **Kích hoạt truy vấn (Enabling Querying)**:
    *   Cho phép người dùng truy vấn dữ liệu **từng thuộc tính một** để giảm khối lượng dữ liệu (ví dụ: `GET /owners?_fields=id`). Thường sử dụng các tham số truy vấn `fields` hoặc `properties`.
    *   **GraphQL**: Một ngôn ngữ truy vấn cho API cho phép người dùng chỉ định **chính xác dữ liệu họ muốn** và thực hiện **nhiều truy vấn trong một cuộc gọi duy nhất**.
        *   **Ưu điểm**: Lựa chọn dữ liệu linh hoạt cao, một cuộc gọi cho nhiều truy vấn.
        *   **Nhược điểm**: Sử dụng phương thức POST, nên cơ chế caching HTTP tiêu chuẩn (như GET) không áp dụng được. Bản thân GraphQL không có caching gốc; tùy thuộc vào người dùng.
*   **Cung cấp dữ liệu và mục tiêu phù hợp hơn**:
    *   Giao tiếp không hiệu quả có thể là dấu hiệu cho thấy thiết kế không đáp ứng đủ nhu cầu thực tế của người dùng.
    *   **Thêm dữ liệu liên quan**: Bao gồm dữ liệu phái sinh được sử dụng phổ biến trực tiếp (ví dụ: `balance` trong danh sách `transactions`) để tránh các cuộc gọi bổ sung.
    *   **Thêm mục tiêu liên quan**: Cung cấp các điểm truy cập trực tiếp cho các trường hợp sử dụng phổ biến (ví dụ: `GET /dashboards/me` cho dữ liệu bảng điều khiển).
    *   **Đánh giá hiệu quả**: Kiểm tra thiết kế với các **trường hợp sử dụng thực tế và trường hợp biên (edge cases)**, không chỉ các trường hợp giả định cơ bản.
*   **Tạo các lớp API khác nhau (API Layers)**:
    *   **Khi nào nên nói "không"**: Đừng tối ưu hóa một API duy nhất mà hy sinh khả năng sử dụng và tái sử dụng cho tất cả người dùng. Nhu cầu rất cụ thể có thể dẫn đến các API phức tạp, không thể tái sử dụng.
    *   **Giải pháp**: Xây dựng các API chuyên biệt (ví dụ: **BFF - Backend For Frontend**) dựa trên các API hiện có. Các nhà cung cấp có thể cung cấp **"API trải nghiệm" (experience APIs)** được tối ưu hóa cho các ngữ cảnh cụ thể (chức năng hoặc kỹ thuật).
    *   **Các lớp API**:
        *   **Experience APIs**: Tối ưu hóa cho các ngữ cảnh/người dùng cụ thể (ví dụ: di động).
        *   **Original/Not Specialized APIs**: Hướng tới người dùng, mục đích chung.
        *   **System APIs**: Truy cập vào các hệ thống cốt lõi (như "magnetron" trong ví dụ lò vi sóng).

Hy vọng thông tin chi tiết này giúp bạn hiểu rõ hơn về chương 10.
