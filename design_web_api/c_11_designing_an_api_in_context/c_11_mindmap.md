# Chương 11: Thiết kế API trong ngữ cảnh
Việc thiết kế API đòi hỏi phải xem xét toàn bộ ngữ cảnh mà API tồn tại.

## 11.1 Điều chỉnh giao tiếp theo mục tiêu và bản chất của dữ liệu
* Cơ chế yêu cầu/phản hồi đồng bộ và đơn lẻ không phải lúc nào cũng hiệu quả nhất.

* ``11.1.1 Quản lý các quy trình dài (Managing long processes)``
    * ``Vấn đề``: Thời gian xử lý dài (phút, giờ, ngày) khiến yêu cầu/phản hồi đồng bộ không phù hợp (ví dụ: chuyển tiền quốc tế).
    * ``Giải pháp``:
        * API tiếp nhận yêu cầu và thông báo đã được xử lý.
        * Cung cấp cách để ``kiểm tra trạng thái xử lý yêu cầu sau``.
        * Cung cấp thông tin về thời điểm thực hiện yêu cầu tiếp theo để tránh cuộc gọi không cần thiết.

* ``11.1.2 Thông báo sự kiện cho người dùng (Notifying consumers of events)``
    * ``Vấn đề``: Người dùng cần thông báo ngay lập tức hoặc không thể liên tục kiểm tra trạng thái.
    * ``Sự kiện``: Thông điệp từ nhà cung cấp thông báo ``điều gì đó đã xảy ra``.
    * ``Cách phổ biến``:
        * ``Webhook``: Nhà cung cấp gửi yêu cầu HTTP đến URL cấu hình trước của người dùng.
        * ``WebSub``: Giao thức công khai dựa trên webhook.
    * ``Thiết kế``: Tuân thủ nguyên tắc ẩn giấu quan điểm nhà cung cấp, dễ sử dụng, dễ phát triển.

* ``11.1.3 Truyền luồng sự kiện (Streaming event flows)``
    * ``Trường hợp``: Cần nhận ``dữ liệu liên tục hoặc cập nhật theo thời gian thực`` (ví dụ: giá cổ phiếu).
    * ``Công nghệ``:
        * ``Server-Sent Events (SSE)``: Máy chủ gửi cập nhật một chiều qua HTTP.
        * ``WebSocket``: Giao tiếp hai chiều full-duplex qua một kết nối TCP duy nhất.
    * ``Thiết kế dữ liệu sự kiện``: Cung cấp ``càng nhiều dữ liệu càng tốt`` để tránh cuộc gọi bổ sung.

* ``11.1.4 Xử lý nhiều phần tử (Processing multiple elements)``
    * ``Vấn đề``: Thực hiện cùng hành động trên nhiều tài nguyên, từng cái một không hiệu quả.
    * ``Giải pháp``: Cung cấp mục tiêu cho phép ``xử lý hàng loạt`` (batch processing) nhiều phần tử trong một cuộc gọi duy nhất.
    * ``Ví dụ``: `update transactions` để cập nhật nhiều giao dịch.
    * ``HTTP``: Mã trạng thái ``207 Multi-Status`` cho phản hồi nhiều hoạt động.
    * ``Yêu cầu``: Người dùng phải nhận được ``cùng dữ liệu`` (bao gồm headers, mã trạng thái HTTP) như yêu cầu đơn lẻ. Cần xử lý lỗi và điều khiển chung (giới hạn phần tử).

## 11.2 Quan sát toàn bộ ngữ cảnh (Observing the full context)
* Thiết kế API đòi hỏi phải xem xét đầy đủ ngữ cảnh để đáp ứng nhu cầu và có thể triển khai được.

* ``11.2.1 Nhận thức về các thực hành và giới hạn hiện có của người dùng (Being aware of consumers' existing practices and limitations)``
    * ``Phù hợp với``: ``Thực hành hiện có`` và ``giới hạn kỹ thuật`` của người dùng.
    * ``Ví dụ``: Sử dụng ``XML tài chính chuẩn ISO 20022`` thay vì JSON nếu phần mềm COTS của khách hàng quen với XML.
    * ``Giới hạn kỹ thuật khác``: Không thể thêm headers, chỉ dùng GET/POST, giới hạn mạng di động.
    * ``Quan trọng``: ``Thể hiện sự đồng cảm``, nói chuyện với người dùng.
    * ``Cẩn trọng``: Không hy sinh tính khả dụng/tái sử dụng. Xem xét ``lớp API khác nhau`` hoặc BFF.

* ``11.2.2 Cẩn thận xem xét các giới hạn của nhà cung cấp (Carefully considering the provider’s limitations)``
    * Tránh phơi bày quan điểm nội bộ nhưng xem xét ``khả năng triển khai``.
    * ``Giới hạn kỹ thuật``: Thời gian phản hồi dài của hệ thống bên dưới, khả năng mở rộng/tính sẵn có, hạn chế mạng (firewall).
    * ``Lưu ý``: Nhiều giới hạn kỹ thuật là "giả", có thể giải quyết bằng nỗ lực triển khai.
    * ``Vai trò nhà thiết kế``: ``Hiểu sâu sắc`` toàn bộ chuỗi để phát hiện và giải quyết sớm.
    * ``Nguyên tắc``: Vẫn phải ``che giấu quan điểm của nhà cung cấp`` càng nhiều càng tốt.

## 11.3 Lựa chọn phong cách API theo ngữ cảnh (Choosing an API style according to the context)
* Lựa chọn công cụ (phong cách API) phải dựa trên ``ngữ cảnh và nhu cầu``, không phải thói quen, phổ biến hay sở thích cá nhân.

* ``11.3.1 Đối lập các API dựa trên tài nguyên, dữ liệu và chức năng (Contrasting resource-, data-, and function-based APIs)``
    * ``REST (Resource-based)``:
        * ``Dựa trên tài nguyên``, tận dụng HTTP.
        * ``Ưu điểm``: Nhất quán cao, dễ đoán, tận dụng tính năng HTTP (caching, conditional requests, SSE).
        * ``Nhược điểm``: Vẫn cần thiết kế cẩn thận.
    * ``gRPC (Function-based)``:
        * ``RPC (Remote Procedure Call)``: Phơi bày các hàm.
        * ``Ưu điểm``: Giao tiếp hiệu quả qua HTTP/2 (binary transport, multiplexing), phù hợp microservices, tạo SDK tự động.
        * ``Nhược điểm``: Ít tiêu chuẩn hóa, không có caching/conditional requests sẵn có, khó kiểm soát độ phức tạp truy vấn.
    * ``GraphQL (Data-based)``:
        * ``Ngôn ngữ truy vấn dữ liệu`` và runtime. Người dùng yêu cầu chính xác dữ liệu cần.
        * ``Ưu điểm``: Linh hoạt cao, giảm thiểu cuộc gọi/dữ liệu trao đổi (over-fetching/under-fetching).
        * ``Nhược điểm``: Giống RPC trong tạo/sửa đổi dữ liệu (mutations), không có caching tích hợp, khó kiểm soát độ phức tạp truy vấn.
    * ``Kết luận``: ``Không có phong cách nào tốt hơn``, phụ thuộc nhu cầu/ngữ cảnh. ``Chọn REST theo mặc định``; cân nhắc GraphQL/gRPC cho nhu cầu rất đặc thù.

* ``11.3.2 Suy nghĩ vượt ra ngoài các API dựa trên yêu cầu/phản hồi và HTTP (Thinking beyond request/response- and HTTP-based APIs)``
    * API dựa trên yêu cầu/phản hồi và HTTP không phải là cách duy nhất.
    * ``Hệ thống dựa trên sự kiện (Event-based systems)``: Thông báo sự kiện bằng webhook, WebSub (dựa trên HTTP) hoặc các hệ thống nhắn tin (RabbitMQ).
    * ``Lưu ý``: Giao tiếp dựa trên HTTP không phải là lựa chọn duy nhất, đôi khi nên tránh.

## Tóm lại:

Việc thiết kế API đòi hỏi phải xem xét toàn bộ ngữ cảnh, bao gồm các phương thức giao tiếp khác nhau, các giới hạn của người dùng và nhà cung cấp, và lựa chọn phong cách API phù hợp nhất với tình huống cụ thể, chứ không phải dựa vào các xu hướng hoặc sở thích cá nhân.
