### Chương 11: Thiết kế API trong ngữ cảnh

#### 11.1 Điều chỉnh giao tiếp theo mục tiêu và bản chất của dữ liệu
Cho đến nay, sách chủ yếu đề cập đến các API web đồng bộ (synchronous), nơi người dùng gửi yêu cầu và nhận phản hồi ngay lập tức. Tuy nhiên, tùy thuộc vào bản chất của các mục tiêu và dữ liệu của API, cơ chế yêu cầu/phản hồi đồng bộ và đơn lẻ có thể không phải là cách biểu diễn hiệu quả nhất. Người thiết kế API cần có các công cụ khác ngoài yêu cầu/phản hồi đồng bộ để xử lý các trường hợp này.

*   **11.1.1 Quản lý các quy trình dài (Managing long processes)**
    *   Cơ chế yêu cầu/phản hồi đồng bộ không phải lúc nào cũng là lựa chọn tốt nhất để biểu diễn một mục tiêu, đặc biệt khi có **thời gian xử lý dài**.
    *   Ví dụ: Việc chuyển tiền quốc tế trong API Ngân hàng có thể yêu cầu **xác thực thủ công bởi con người** và tài liệu bổ sung tùy thuộc vào quốc gia, ngân hàng mục tiêu và số tiền chuyển.
    *   Nếu là đồng bộ, người dùng sẽ phải đợi hàng phút, thậm chí hàng giờ hoặc hàng ngày để nhận được phản hồi, điều này là không thể chấp nhận được.
    *   Trong những trường hợp như vậy, API cần cung cấp một mục tiêu để **tiếp nhận yêu cầu** (chẳng hạn, thông báo rằng yêu cầu đã được xử lý) và sau đó cung cấp một cách để **kiểm tra trạng thái xử lý yêu cầu này sau**. Việc cung cấp thông tin về thời điểm thực hiện yêu cầu tiếp theo (bằng cách tận dụng các tính năng giao thức hoặc đơn giản là trả về dữ liệu) mang lại lợi ích cho cả người dùng và nhà cung cấp bằng cách tránh các cuộc gọi không cần thiết.

*   **11.1.2 Thông báo sự kiện cho người dùng (Notifying consumers of events)**
    *   Khi người dùng cần được thông báo về các sự kiện ngay lập tức hoặc không thể liên tục kiểm tra trạng thái, cơ chế yêu cầu/phản hồi không hiệu quả.
    *   Sự kiện là một thông điệp được tạo bởi một hệ thống (nhà cung cấp) để thông báo cho các hệ thống khác (người dùng) rằng **điều gì đó đã xảy ra**.
    *   Hai cách phổ biến để thông báo sự kiện là **Webhook** và **WebSub**.
        *   **Webhook**: Khi một sự kiện xảy ra, nhà cung cấp sẽ gửi yêu cầu HTTP đến một URL được định cấu hình trước bởi người dùng.
        *   **WebSub**: Một giao thức công khai dựa trên webhook, cho phép các máy chủ đăng ký để nhận thông báo từ máy chủ khác.
    *   Thiết kế API webhook cần tuân thủ các nguyên tắc ẩn giấu quan điểm nhà cung cấp và đảm bảo tính dễ sử dụng, dễ phát triển.

*   **11.1.3 Truyền luồng sự kiện (Streaming event flows)**
    *   Đối với các trường hợp người dùng cần nhận **dữ liệu liên tục hoặc cập nhật theo thời gian thực**, chẳng hạn như giá cổ phiếu, truyền luồng là phù hợp hơn.
    *   **Server-Sent Events (SSE)** là một công nghệ cho phép máy chủ gửi cập nhật đến trình duyệt web qua kết nối HTTP một chiều. Dữ liệu sự kiện được cung cấp dưới dạng luồng, không còn là tài liệu tĩnh.
    *   **WebSocket** cung cấp giao tiếp hai chiều full-duplex qua một kết nối TCP duy nhất, phù hợp cho các ứng dụng tương tác cao như trò chuyện hoặc game.
    *   Khi thiết kế dữ liệu sự kiện cho luồng, thường tốt hơn là **cung cấp càng nhiều dữ liệu càng tốt** để người dùng không cần phải thực hiện các cuộc gọi API riêng biệt khác để lấy thông tin bổ sung.

*   **11.1.4 Xử lý nhiều phần tử (Processing multiple elements)**
    *   Khi người dùng cần thực hiện cùng một hành động trên nhiều tài nguyên, việc xử lý từng tài nguyên một có thể không hiệu quả về mặt mạng.
    *   API có thể cung cấp các mục tiêu cho phép **xử lý hàng loạt** (batch processing) nhiều phần tử trong một cuộc gọi duy nhất.
    *   Ví dụ: API Ngân hàng có thể cung cấp mục tiêu `update transactions` để cập nhật nhiều giao dịch cùng một lúc, nhận danh sách các cập nhật giao dịch và trả về trạng thái của từng giao dịch trong phản hồi duy nhất.
    *   Giao thức HTTP có mã trạng thái **207 Multi-Status** cho các phản hồi chứa kết quả của nhiều hoạt động.
    *   Dù giải pháp là gì, người dùng phải nhận được **cùng một dữ liệu**, bao gồm dữ liệu giao thức như headers hoặc mã trạng thái HTTP, như thể họ đã thực hiện các yêu cầu đơn lẻ. Cần xử lý các lỗi và điều khiển chung, ví dụ như giới hạn số lượng phần tử trong yêu cầu.

#### 11.2 Quan sát toàn bộ ngữ cảnh (Observing the full context)
Thiết kế API đòi hỏi phải xem xét đầy đủ ngữ cảnh mà API sẽ được sử dụng và cung cấp, để đảm bảo API đáp ứng tất cả các nhu cầu của người dùng một cách tốt nhất và có thể triển khai được bởi nhà cung cấp.

*   **11.2.1 Nhận thức về các thực hành và giới hạn hiện có của người dùng (Being aware of consumers' existing practices and limitations)**
    *   API nên được thiết kế để phù hợp với **thực hành hiện có của người dùng** và **các giới hạn kỹ thuật** của họ.
    *   Ví dụ: Một API xác minh chi tiết ngân hàng cho khách hàng doanh nghiệp trong ngành tài chính có thể cần sử dụng các tin nhắn **XML tài chính chuẩn ISO 20022** thay vì JSON tùy chỉnh, bởi vì phần mềm COTS (Commercial Off-The-Shelf) của họ đã quen thuộc với định dạng đó. Việc này tuân theo nguyên tắc "Nhập gia tùy tục" (When in Rome, do as the Romans do).
    *   Các giới hạn kỹ thuật khác có thể bao gồm:
        *   Không thể thêm headers vào yêu cầu HTTP.
        *   Chỉ có thể sử dụng các phương thức HTTP **GET hoặc POST**.
        *   Giới hạn về khả năng mạng (đặc biệt đối với ứng dụng di động).
    *   Quan trọng là phải **thể hiện sự đồng cảm với người dùng** mục tiêu, nói chuyện và thảo luận với họ về thiết kế để hiểu rõ nhu cầu và giới hạn của họ.
    *   Tuy nhiên, không nên làm điều này bằng cách hy sinh tính khả dụng và khả năng tái sử dụng của API. Nếu có nhu cầu quá đặc thù, hãy xem xét tạo **các lớp API khác nhau** hoặc để người dùng tự tạo API backend-for-frontend (BFF) của riêng họ.

*   **11.2.2 Cẩn thận xem xét các giới hạn của nhà cung cấp (Carefully considering the provider’s limitations)**
    *   Tránh phơi bày quan điểm nội bộ của nhà cung cấp nhưng vẫn cần xem xét những gì đang xảy ra "đằng sau API" để đề xuất một thiết kế có thể thực sự triển khai được.
    *   Các giới hạn kỹ thuật có thể bao gồm:
        *   **Thời gian phản hồi dài** của hệ thống bên dưới.
        *   Khả năng mở rộng hoặc tính sẵn có của các hệ thống bên dưới.
        *   Hạn chế mạng (ví dụ: firewall chặn một số phản hồi HTTP).
    *   **Thường thì các giới hạn kỹ thuật là "giả"** và có thể được giải quyết bằng một chút nỗ lực trong việc triển khai mà không ảnh hưởng lớn đến thiết kế API.
    *   Người thiết kế API phải có **hiểu biết sâu sắc** về toàn bộ chuỗi từ người dùng đến triển khai thực tế để phát hiện các giới hạn kỹ thuật sớm nhất có thể và giải quyết chúng.
    *   Mặc dù phải cân nhắc các giới hạn của nhà cung cấp, người thiết kế vẫn phải **che giấu quan điểm của nhà cung cấp** càng nhiều càng tốt để cung cấp các API dễ hiểu và dễ sử dụng.

#### 11.3 Lựa chọn phong cách API theo ngữ cảnh (Choosing an API style according to the context)
Việc lựa chọn công cụ (phong cách API) phải dựa trên **ngữ cảnh và nhu cầu**, chứ không phải dựa trên thói quen, sự phổ biến hoặc sở thích cá nhân.

*   **11.3.1 Đối lập các API dựa trên tài nguyên, dữ liệu và chức năng (Contrasting resource-, data-, and function-based APIs)**
    *   Có ba cách chính để tạo API web hiện nay:
        *   **REST (Resource-based)**:
            *   **Dựa trên tài nguyên** (resource-oriented) và tận dụng giao thức HTTP.
            *   Các mục tiêu được biểu diễn bằng việc sử dụng các phương thức HTTP chuẩn trên các tài nguyên và kết quả được biểu diễn bằng các mã trạng thái HTTP chuẩn.
            *   Ví dụ: GET `/owners/123` để đọc thông tin chủ sở hữu.
            *   Ưu điểm: Tính nhất quán cao (uniform interface), dễ đoán, tận dụng các tính năng HTTP như caching, conditional requests, SSE.
            *   Nhược điểm: Vẫn cần thiết kế cẩn thận để tránh API tệ.
        *   **gRPC (Function-based)**:
            *   Là viết tắt của Remote Procedure Call (RPC). API RPC đơn giản là **phơi bày các hàm**.
            *   Ví dụ: `listOwners()`, `saveOwner()`, `updateOwner()`.
            *   Ưu điểm: Giao tiếp hiệu quả qua HTTP/2 (binary transport, multiplexing), phù hợp cho microservices, có thể tạo SDK tự động.
            *   Nhược điểm: Ít tiêu chuẩn hóa hơn về cách đặt tên hàm và biểu diễn kết quả, không có cơ chế caching/conditional requests sẵn có. Khó kiểm soát độ phức tạp của truy vấn từ người dùng, có thể gây quá tải hệ thống.
        *   **GraphQL (Data-based)**:
            *   Là một **ngôn ngữ truy vấn dữ liệu** (query language) và runtime cho các API. Người dùng có thể yêu cầu chính xác dữ liệu họ cần trong một yêu cầu duy nhất.
            *   Ví dụ: Truy vấn để lấy thông tin chủ sở hữu và tài khoản liên quan.
            *   Ưu điểm: Linh hoạt cao trong việc lấy dữ liệu, giảm thiểu các cuộc gọi API không cần thiết và lượng dữ liệu trao đổi (over-fetching/under-fetching).
            *   Nhược điểm: Giống RPC trong việc tạo/sửa đổi dữ liệu (mutations), không có cơ chế caching tích hợp mạnh mẽ như HTTP, khó kiểm soát độ phức tạp của truy vấn từ người dùng, có thể gây quá tải hệ thống.
    *   **Không có phong cách nào tốt hơn phong cách nào**; tất cả phụ thuộc vào nhu cầu và ngữ cảnh.
    *   Quy tắc chung là **chọn REST theo mặc định** vì nó phổ biến và đáp ứng hầu hết các nhu cầu. GraphQL hoặc gRPC nên được xem xét khi REST không đáp ứng được các nhu cầu rất đặc thù (ví dụ: truy vấn dữ liệu phức tạp, hiệu quả mạng cao cho microservices).

*   **11.3.2 Suy nghĩ vượt ra ngoài các API dựa trên yêu cầu/phản hồi và HTTP (Thinking beyond request/response- and HTTP-based APIs)**
    *   Người thiết kế API phải nhận thức rằng API dựa trên yêu cầu/phản hồi và HTTP không phải là cách duy nhất để kích hoạt giao tiếp giữa các ứng dụng.
    *   **Hệ thống dựa trên sự kiện (Event-based systems)**: Người cung cấp có thể thông báo cho người dùng về các sự kiện bằng cách sử dụng webhook, WebSub (dựa trên HTTP) hoặc các hệ thống nhắn tin như RabbitMQ.
    *   Đối với mục đích nội bộ, việc kết nối trực tiếp nhà cung cấp và người dùng với các công cụ nhắn tin có thể hiệu quả hơn.
    *   Quan trọng là phải nhớ rằng giao tiếp dựa trên HTTP không phải là lựa chọn duy nhất, và trong một số ngữ cảnh nhất định, nó thực sự nên được tránh bằng mọi giá.

Chương 11 kết thúc với thông điệp rằng việc thiết kế API đòi hỏi phải xem xét toàn bộ ngữ cảnh, bao gồm các phương thức giao tiếp khác nhau, các giới hạn của người dùng và nhà cung cấp, và lựa chọn phong cách API phù hợp nhất với tình huống cụ thể, chứ không phải dựa vào các xu hướng hoặc sở thích cá nhân.
