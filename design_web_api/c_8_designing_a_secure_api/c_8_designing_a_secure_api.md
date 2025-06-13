### **Chương 8: Thiết kế API bảo mật**

### **8.1 Tổng quan về Bảo mật API (An Overview of API Security)**

Thiết kế API bảo mật đòi hỏi bạn phải có hiểu biết cơ bản về các nguyên tắc bảo mật API, cũng như ý nghĩa của chúng đối với cả nhà phát triển ứng dụng tiêu thụ API (API consumers) và nhà cung cấp API (API providers), và cả người dùng cuối (end-users).

1.  **8.1.1 Đăng ký người dùng (Registering a consumer)**
    *   Các API an toàn chỉ cho phép các **người dùng đã biết (known consumers)** sử dụng chúng.
    *   Khi nhà phát triển muốn sử dụng một API trong ứng dụng của họ (ứng dụng di động hoặc ứng dụng backend), họ phải đăng ký ứng dụng đó trước.
    *   Quá trình này thường được thực hiện thông qua **cổng thông tin dành cho nhà phát triển (developer portal)** của nhà cung cấp API, nơi cung cấp tài liệu, hướng dẫn, và các tài nguyên hữu ích khác.
    *   Trong quá trình đăng ký, nhà phát triển phải **chọn các "scopes" (phạm vi)** mà ứng dụng của họ sẽ sử dụng. Một scope tương ứng với một hoặc nhiều mục tiêu (goals) của API.
    *   **Ví dụ**: Ứng dụng "Boring Financial Dashboard" chỉ sử dụng scope `read accounts and transactions`, tương ứng với các mục tiêu `list accounts`, `read account`, và `list transactions`. Do đó, ứng dụng này chỉ được phép sử dụng ba mục tiêu đó. Trong khi đó, ứng dụng "Awesome Banking Application" sử dụng tất cả các scopes và được phép sử dụng tất cả các mục tiêu của API.
    *   Sau khi cấu hình, nhà phát triển sẽ nhận được một **Client ID** cho ứng dụng của họ, được sử dụng trong các bước tiếp theo.
2.  **8.1.2** Nhận thông tin xác thực
    - **Mô tả**: Sau khi có *client ID* và *client secret*, ứng dụng cần liên hệ với **máy chủ ủy quyền (authorization server)** để nhận *access token*. *Access token* là một chuỗi ký tự tạm thời, cho phép ứng dụng truy cập API trong một khoảng thời gian nhất định.
    - **Quy trình**:
      1. Ứng dụng gửi yêu cầu đến máy chủ ủy quyền, kèm theo *client ID* và *client secret*.
      2. Máy chủ xác minh thông tin xác thực và trả về *access token* nếu hợp lệ.
      3. *Access token* thường có thời hạn (ví dụ: 1 giờ) và có thể được làm mới (refresh) nếu cần.
    - **Ví dụ**: Tiếp tục ví dụ trên, ứng dụng di động của bạn gửi một yêu cầu HTTP đến máy chủ ủy quyền của mạng xã hội:
      ```http
      POST /oauth/token HTTP/1.1
      Host: api.socialnetwork.com
      Content-Type: application/x-www-form-urlencoded

      grant_type=client_credentials&client_id=abc123&client_secret=xyz789
      ```
      Máy chủ trả về:
      ```json
      {
        "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "token_type": "Bearer",
        "expires_in": 3600
      }
      ```
      *Access token* này sẽ được sử dụng để gọi API.

3.  **8.1.3 Thực hiện cuộc gọi API (Making an API call)**
    *   Để thực hiện cuộc gọi API, người dùng (consumer) phải gửi một yêu cầu tới API (qua một kênh bảo mật) cùng với **"access token"** đã nhận được trước đó.
    *   Khi API nhận được yêu cầu, nó sẽ liên hệ với **"authorization server"** để xác thực access token.
    *   Nếu token hợp lệ, authorization server sẽ trả về dữ liệu đính kèm với nó, bao gồm **ID của người dùng cuối (end user's ID)**, Client ID và các scopes đã được cấp quyền.
    *   Triển khai API trước tiên kiểm tra xem mục tiêu được yêu cầu (ví dụ: `list accounts`) có thuộc một trong các scopes đã được cấp quyền cho ứng dụng hay không.
    *   Sau đó, dựa trên **ID của người dùng cuối đính kèm với access token**, việc triển khai sẽ lọc kết quả và chỉ trả về các tài khoản mà người dùng cuối đó được phép truy cập. Điều này đảm bảo rằng ngay cả khi ứng dụng có quyền truy cập rộng hơn, dữ liệu trả về vẫn được giới hạn cho người dùng cụ thể.

4.  **8.1.4 Nhìn nhận thiết kế API từ góc độ bảo mật (Envisioning API design from the perspective of security)**
    *   **Kênh bảo mật**: Mọi giao tiếp giữa người dùng và nhà cung cấp nên diễn ra qua một kênh bảo mật để ngăn chặn việc đánh chặn dữ liệu. Tuy nhiên, điều này không liên quan trực tiếp đến thiết kế API mà là về hạ tầng.
    *   **Kiểm soát truy cập ứng dụng**: Chỉ những người dùng đã đăng ký mới được phép truy cập API, và họ chỉ nên được phép sử dụng những phần API mà họ thực sự cần và mà người dùng cuối đã cấp quyền.
        *   **Xác định scopes (a)**: Việc phân chia API thành các nhóm mục tiêu (scopes) để cấp quyền truy cập chọn lọc là công việc của nhà thiết kế API, bởi vì các nhóm mục tiêu này phải có ý nghĩa đối với cả nhà phát triển và người dùng cuối.
        *   **Điều chỉnh hành vi theo người dùng cuối (b)**: Việc triển khai cần biết người dùng cuối là ai để điều chỉnh hành vi của API theo quyền của người dùng cuối cụ thể. Nhà thiết kế API nên ghi nhớ điều này vì nó có thể ảnh hưởng đến thiết kế API.
        *   **Nguyên tắc**: Sử dụng **least privilege principle** (nguyên tắc quyền tối thiểu), nghĩa là chỉ cấp quyền vừa đủ để thực hiện nhiệm vụ.
    *   **Xử lý tài liệu nhạy cảm (Sensitive material)**:
        *   **Dữ liệu nhạy cảm (c)**: Nhà thiết kế API phải xem xét liệu có nên thực sự hiển thị dữ liệu nhạy cảm qua API hay không.
        *   **Mục tiêu nhạy cảm (d)**: Tương tự, một số mục tiêu API có thể rất nhạy cảm (ví dụ: kích hoạt các hành động có hậu quả nghiêm trọng) và cần được xem xét cẩn thận.
    *   Tóm lại, để tạo API bảo mật theo thiết kế, nhà thiết kế API phải quan tâm đến **kiểm soát truy cập ứng dụng, kiểm soát truy cập người dùng cuối và dữ liệu/mục tiêu nhạy cảm**.

### **8.2 Phân vùng API để tạo điều kiện kiểm soát truy cập (Partitioning an API to Facilitate Access Control)**

Việc phân vùng API thành các nhóm mục tiêu (scopes) là cần thiết để kiểm soát truy cập.

*   **Nguyên tắc đặc quyền tối thiểu (principle of least privilege)**: Bằng cách giới hạn quyền truy cập của người dùng chỉ vào các mục tiêu mà họ thực sự cần, bạn giảm thiểu khả năng xảy ra một cuộc tấn công. Với các API web được phơi bày trên internet, càng ít cửa mở càng tốt.
*   **Ví dụ**: Một công ty có thể cung cấp các tính năng API khác nhau yêu cầu các gói đăng ký trả phí khác nhau, do đó không muốn người dùng không trả phí truy cập các tính năng đó một cách tự do.

1.  **8.2.1 Xác định các scopes chi tiết nhưng phức tạp (Defining flexible but complex fine-grained scopes)**
    *   Cách tiếp cận là **xác định một scope cho mỗi mục tiêu (goal)** của API.
    *   **Ví dụ**: Ứng dụng "PayFriend App" không thể liệt kê các tài khoản (`list accounts`) nhưng có thể kích hoạt chuyển tiền (`transfer money`).
    *   Ưu điểm: Cấu hình kiểm soát truy cập **rất linh hoạt**.
    *   Nhược điểm: Nếu có nhiều mục tiêu, danh sách scopes sẽ rất dài và có thể **gây choáng ngợp hoặc khó chịu** cho cả nhà phát triển và người dùng cuối khi chọn scopes.

2.  **8.2.2 Xác định các scopes đơn giản nhưng ít linh hoạt hơn (Defining simple but less flexible coarse-grained scopes)**
    *   Thay vì dựa trên mục tiêu, có thể **nhóm các mục tiêu thành các danh mục (categories)** và sử dụng chúng làm scopes.
    *   **Ví dụ**: Nhóm các mục tiêu thành "Account" và "Transfer".
    *   **Nhược điểm**: Một scope quá bao quát (coarse-grained) có thể cấp quyền truy cập không mong muốn vào các mục tiêu quan trọng.
    *   Để tốt hơn, Lauret gợi ý dựa các scopes vào **"whats" (những gì người dùng có thể làm)** mà bạn đã xác định trong API goals canvas (Chương 2).
    *   **Ví dụ**: Thay vì scopes chi tiết, có thể có `manage transfers` hoặc `manage beneficiaries`.

3.  **8.2.3** Chọn chiến lược phạm vi (Trang 197-198)
- **Mô tả**: Việc chọn giữa phạm vi chi tiết và thô phụ thuộc vào nhu cầu của API và người tiêu dùng. Một số chiến lược:
  - **Phạm vi chi tiết**: Dùng cho các API công khai hoặc khi cần kiểm soát chặt chẽ.
  - **Phạm vi thô**: Dùng cho các API nội bộ hoặc khi muốn đơn giản hóa.
  - **Kết hợp**: Kết hợp cả hai để cân bằng giữa linh hoạt và đơn giản.
- **Ví dụ**: API mạng xã hội có thể cung cấp cả hai:
  - `transfers` (thô) cho các ứng dụng nội bộ.
  - `transfers.read`, `transfers.craete` (chi tiết) cho các ứng dụng bên thứ ba.

4.  **8.2.4 Xác định scopes với định dạng mô tả API (Defining scopes with the API description format)**
    *   **OpenAPI Specification (OAS)** cho phép bạn mô tả scopes.
    *   Scopes được định nghĩa trong phần `components.securitySchemes` của tài liệu OAS, với `type` là `oauth2` và mô tả các `flows` (ví dụ: `implicit`) và danh sách `scopes` với tên và mô tả.
    *   **Ví dụ**:
        ```yaml
        components:
          securitySchemes:
            BankingAPIScopes:
              type: oauth2
              flows:
                implicit:
                  authorizationUrl: "https://auth.bankingcompany.com/authorize"
                  scopes:
                    "beneficiary:create": Create beneficiaries
                    "beneficiary:read": List beneficiaries
                    "beneficiary:delete": Delete beneficiaries
                    "beneficiary:manage": Create, list, and delete beneficiaries
        ```
    *   Sau đó, bạn **liên kết một mục tiêu (goal) với các scopes** cần thiết trong phần `security` của mỗi `operation`.
    *   **Ví dụ**: Mục tiêu `list beneficiaries` (GET /beneficiaries) được bảo vệ bởi scopes `beneficiary:read` và `beneficiary:manage`.
        ```yaml
        paths:
          /beneficiaries:
            get:
              tags:
                - Transfer
              description: Gets beneficiaries list
              security:
                - BankingAPIScopes:
                    - "beneficiary:read"
                    - "beneficiary:manage"
              responses:
                "200":
                  description: The beneficiaries list
        ```
    *   Redoc (công cụ hiển thị tài liệu API) tự động thêm menu "Authentication" và hiển thị các scopes cần thiết cho mỗi mục tiêu.

### **8.3 Thiết kế với kiểm soát truy cập (Designing with Access Control in Mind)**

1.  **8.3.1 Biết dữ liệu cần thiết để kiểm soát truy cập (Knowing what data is needed to control access)**
    *   Ngay cả khi một ứng dụng được phép liệt kê tài khoản, nó chỉ nên được phép liệt kê các tài khoản thuộc về người dùng cuối của nó.
    *   **Ví dụ**: Khi "Awesome Banking Application" yêu cầu `list accounts`, việc triển khai sử dụng **ID của người dùng cuối đính kèm với access token** để lọc và chỉ trả về các tài khoản thuộc về người dùng đó. Điều này là một **phần ẩn của hợp đồng giao diện API (hidden part of the API interface contract)** mà nhà thiết kế API phải biết để đảm bảo thiết kế hoạt động và an toàn.

2.  **8.3.2 Điều chỉnh thiết kế khi cần thiết (Adapting the design when necessary)**
    *   Khi có sự thay đổi trong ngữ cảnh sử dụng (ví dụ: ứng dụng của cố vấn ngân hàng cần thực hiện chuyển tiền thay mặt khách hàng), có thể cần **sửa đổi thiết kế API** để truyền tải thông tin chính xác về người dùng cuối.
    *   **Ví dụ**: Nếu cố vấn ngân hàng yêu cầu chuyển tiền cho một khách hàng, API cần biết ID khách hàng nào đang được xử lý. Có thể thêm một thuộc tính `customerId` tùy chọn vào request body (POST /transfers) hoặc tạo một đường dẫn tài nguyên mới như `POST /customers/{customerId}/transfers`. Lựa chọn thứ hai có ý nghĩa hơn nếu cố vấn cũng cần liệt kê các giao dịch của khách hàng thông qua `GET /customers/{customerId}/transfers`.
    *   Việc sử dụng API goals canvas và phương pháp xác định tất cả người dùng, những gì họ làm, cách họ làm, và đặc biệt là những gì họ cần để làm, sẽ giúp bạn xử lý các câu hỏi về thiết kế API so với bảo mật API một cách dễ dàng và gần như liền mạch.

### **8.4 Xử lý tài liệu nhạy cảm (Handling Sensitive Material)**

Khi thiết kế API, bạn phải xác định xem dữ liệu hoặc khả năng được yêu cầu, cung cấp hoặc thực hiện thông qua API có liên quan đến **tài liệu nhạy cảm (sensitive material)** hay không. Nếu có, bạn phải đảm bảo rằng nó thực sự cần thiết và sau đó tạo ra thiết kế an toàn nhất có thể.

1.  **8.4.1 Xử lý dữ liệu nhạy cảm (Handling sensitive data)**
    *   **Xác định dữ liệu nhạy cảm**: Bước đầu tiên là xác định dữ liệu nhạy cảm. Điều này đôi khi rõ ràng (ví dụ: số thẻ ghi nợ đầy đủ, mã CVV, tên chủ thẻ, tên người dùng và mật khẩu), nhưng đôi khi không rõ ràng. Luôn kiểm tra với các chuyên gia bảo mật (CISO, DPO) và bộ phận pháp lý.
    *   **Chọn cách thể hiện phù hợp**:
        *   **(1) Xóa dữ liệu nhạy cảm không cần thiết**: Ví dụ, nếu mục tiêu chỉ là cung cấp thông tin chi tiết về tài khoản và thẻ liên kết, không cần cung cấp số thẻ đầy đủ và CVV.
        *   **(2) Thay thế bằng một hình thức không nhạy cảm**: Nếu không thể xóa, hãy thay thế dữ liệu nhạy cảm bằng một phiên bản không nhạy cảm.
            *   **Ví dụ**: Cắt bớt số thẻ (chỉ hiển thị 4 số cuối).
            *   Thay thế giá trị nhạy cảm (`averageMonthlyBill`) bằng một cờ không nhạy cảm (`vip` flag).
            *   Sử dụng **định danh không có ý nghĩa (non-meaningful identifier)** như ID mờ (opaque ID) (ví dụ: `c4ca76a1` thay vì `5678`) cho đường dẫn tài nguyên (`/cards/c4ca76a1`).
        *   **(3) Thay thế nhiều phần dữ liệu nhạy cảm**: Có thể thay thế nhiều mảnh dữ liệu nhạy cảm (ví dụ: `merchant` và `address`) bằng một thuộc tính không nhạy cảm (`type`).
        *   **(4) Mã hóa giá trị (Encrypting value)**: Mã hóa giá trị để nó không còn nhạy cảm khi hiển thị.

2.  **8.4.2 Xử lý mục tiêu nhạy cảm (Handling sensitive goals)**
    *   Có hai loại mục tiêu nhạy cảm: **thao tác với dữ liệu nhạy cảm** và **kích hoạt các hành động có hậu quả nhạy cảm**.
    *   Câu hỏi đầu tiên là: "Mục tiêu này có thực sự cần thiết không?" Nếu không, việc không đưa nó vào API là cách đơn giản nhất.
    *   **Kiểm soát truy cập mục tiêu hiển thị dữ liệu nhạy cảm**:
        *   **(1) Truy cập qua một mục tiêu chuyên biệt (dedicated goal)**: Tạo một mục tiêu riêng biệt để truy cập dữ liệu nhạy cảm.
        *   **(2) Truy cập dựa trên scope (scope-based access)**: Chỉ những người dùng có scope cụ thể (ví dụ: `show sensitive card data`) mới có thể truy cập dữ liệu nhạy cảm thông qua một mục tiêu chung.
        *   **(3) Truy cập dựa trên quyền (permission-based access)**: Dựa vào quyền của người dùng cuối để kiểm tra xem họ có được phép truy cập dữ liệu nhạy cảm hay không.
        *   **(4) Kết hợp các phương pháp**.
    *   **Xử lý các hành động nhạy cảm**: Khi một hành động là nhạy cảm (ví dụ: chặn thẻ), tốt hơn là nên **tạo các mục tiêu riêng biệt, được xác định rõ ràng và chi tiết** (`block card`) thay vì kết hợp nó vào một mục tiêu chung (`update card`). Điều này giúp kiểm soát truy cập dễ dàng hơn và giữ cho API dễ hiểu và dễ sử dụng.

3.  **8.4.3 Thiết kế phản hồi lỗi an toàn (Designing secure error feedback)**
    *   **Không cung cấp thông tin nhạy cảm trong phản hồi lỗi**: Tránh trả về các thông báo lỗi chứa thông tin nhạy cảm (ví dụ: `Impossible to update the 123412341234124 card` nếu `123412341234124` là số thẻ đầy đủ).
    *   **Sử dụng mã trạng thái HTTP phù hợp**:
        *   `401 Unauthorized`: Khi không cung cấp token hoặc token không hợp lệ.
        *   `403 Forbidden`: Khi người dùng không có quyền truy cập scope yêu cầu (ví dụ: `Consumer has not been granted the "read card" scope` với mã lỗi `SCOPE`) hoặc không có quyền truy cập vào một tài nguyên cụ thể (ví dụ: `End user is not allowed to access this card` với mã lỗi `PERMISSIONS`).
        *   `5XX (Internal Server Error)`: Đối với các lỗi máy chủ không mong muốn, **không bao giờ cung cấp thông tin chi tiết về ngăn xếp kỹ thuật (technical stack)** (ví dụ: stack trace, phiên bản phần mềm, địa chỉ máy chủ). Bạn có thể cung cấp một ID lỗi để điều tra thêm.

4.  **8.4.4 Xác định các vấn đề về kiến trúc và giao thức (Identifying architecture and protocol issues)**
    *   **Đường dẫn và tham số truy vấn**: Luôn thận trọng khi đặt bất kỳ thông tin nhạy cảm nào vào các **tham số đường dẫn (path parameters) hoặc tham số truy vấn (query parameters)**.
    *   **Lý do**: Các proxy hoặc bộ cân bằng tải (load balancers) có thể ghi lại mọi cuộc gọi HTTP, bao gồm cả URL và tham số truy vấn, và những nhật ký này có thể được hiển thị trong các công cụ giám sát.
    *   **Ví dụ**: `GET /accounts?customerLastName=Smith` có thể bị ghi lại và làm lộ thông tin nhạy cảm của khách hàng.
    *   **Kiểm tra rò rỉ dữ liệu**: Luôn kiểm tra xem có bất kỳ rò rỉ dữ liệu tiềm ẩn nào do giao thức cơ bản hoặc kiến trúc được xây dựng cho API hay không để điều chỉnh thiết kế API cho phù hợp.

Việc áp dụng các nguyên tắc này sẽ giúp bạn thiết kế các API không chỉ mạnh mẽ về chức năng mà còn an toàn và đáng tin cậy.