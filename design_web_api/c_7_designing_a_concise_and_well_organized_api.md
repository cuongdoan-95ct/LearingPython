Chào bạn, tôi sẽ cung cấp cho bạn thông tin chi tiết về Chương 7: "Thiết kế API ngắn gọn và có tổ chức tốt" (Designing a Concise and Well-Organized API) dựa trên các nguồn đã cung cấp và cuộc trò chuyện của chúng ta. Mục tiêu của chương này là giúp bạn thiết kế các API mà người dùng có thể dễ dàng hiểu và sử dụng, tránh gây choáng ngợp hoặc khó hiểu.

### **Chương 7: Thiết kế API ngắn gọn và có tổ chức tốt**

Chương này tập trung vào hai khía cạnh chính để đảm bảo API của bạn dễ sử dụng: **tổ chức API** và **kích thước API**.

---

### **7.1 Tổ chức API (Organizing an API)**

Giống như việc sắp xếp các nút trên một chiếc điều khiển TV, khả năng sử dụng của API phụ thuộc vào cách các yếu tố của nó được nhóm và sắp xếp hợp lý. Một API có thể trở nên khó sử dụng hoặc hoàn toàn trực quan tùy thuộc vào cách dữ liệu, phản hồi và mục tiêu của nó được tổ chức.

*   **7.1.1 Tổ chức dữ liệu (Organizing Data)**
    Thiết kế một API có tổ chức tốt bắt đầu từ dữ liệu của nó.
    *   **Nhóm dữ liệu (Grouping Data)**:
        *   Bạn nên nhóm các thuộc tính liên quan lại với nhau. Điều này có thể được thực hiện bằng cách sử dụng các tiền tố chung hoặc tạo các cấu trúc con (substructures).
        *   **Ví dụ**: Thay vì có các thuộc tính riêng lẻ như `overdraftProtection` (bảo vệ thấu chi) và `limit` (giới hạn), bạn có thể nhóm chúng lại thành một cấu trúc con như sau:
            ```json
            {
              "overdraftProtection": {
                "active": true,
                "limit": 100
              }
            }
            ```
        *   Cách này giúp cung cấp cái nhìn rõ ràng hơn về các thuộc tính bắt buộc (required) hoặc tùy chọn (optional).
    *   **Sắp xếp dữ liệu (Sorting Data)**:
        *   Sau khi nhóm dữ liệu, bạn nên **sắp xếp các thuộc tính từ quan trọng nhất đến ít quan trọng nhất** trong mỗi nhóm, và sau đó sắp xếp các nhóm theo mức độ quan trọng. Điều này cải thiện khả năng đọc cho người dùng.
        *   **Ví dụ**: Trong một đối tượng tài khoản ngân hàng, các thuộc tính có thể được sắp xếp như sau, từ quan trọng hơn đến ít quan trọng hơn:
            ```json
            {
              "typeName": "checking",
              "type": 2,
              "safeToSpend": 600,
              "balance": 500,
              "overdraftProtection": {
                "active": true,
                "limit": 100
              },
              "age": 3
            }
            ```

*   **7.1.2 Tổ chức phản hồi (Organizing Feedback)**
    Một API có tổ chức tốt cung cấp phản hồi có tổ chức tốt.
    *   Sử dụng **mã trạng thái HTTP** (HTTP status codes) thích hợp để cung cấp phản hồi có thông tin (ví dụ: `201 Created` cho tạo thành công, `202 Accepted` cho yêu cầu được chấp nhận nhưng xử lý sau, `400 Bad Request` cho lỗi yêu cầu).
    *   Tổ chức phản hồi để tạo điều kiện thuận lợi cho việc giải thích bằng cách:
        *   Tận dụng cách tổ chức phản hồi của giao thức cơ bản.
        *   Tạo cách tổ chức phản hồi của riêng bạn.
        *   **Sắp xếp nhiều lỗi từ quan trọng nhất đến ít quan trọng nhất**.
        *   **Ví dụ**: Thay vì trả về từng lỗi một, API có thể trả về một phản hồi `400 Bad Request` chứa một danh sách tất cả các lỗi chi tiết, được sắp xếp theo mức độ nghiêm trọng:
            ```json
            {
              "message": "Invalid request",
              "errors": [
                {
                  "source": "amount",
                  "type": "BUSINESS_RULE",
                  "message": "Amount exceeds safe to spend"
                },
                {
                  "source": "destination",
                  "type": "MISSING_MANDATORY_PARAMETER",
                  "message": "Destination is mandatory"
                },
                {
                  "source": "date",
                  "type": "BAD_FORMAT_OR_TYPE",
                  "message": "Date must use ISO 8601 YYY-MM-DD format"
                }
              ]
            }
            ```

*   **7.1.3 Tổ chức mục tiêu (Organizing Goals)**
    Các mục tiêu của API cũng cần được tổ chức tốt.
    *   **Tổ chức ảo (Virtual Organization)**: Sử dụng các định dạng mô tả API như **OpenAPI Specification (OAS)** để nhóm các mục tiêu một cách ảo.
        *   Thêm thuộc tính `tags` (thẻ) vào mỗi hoạt động (operation) để nhóm chúng thành các danh mục (ví dụ: "Account" cho tài khoản, "Transfer" cho chuyển khoản).
        *   Một hoạt động có thể thuộc nhiều danh mục nếu cần.
        *   **Nhóm các mục tiêu dựa trên quan điểm chức năng**, không nhất thiết phải dựa trên cấu trúc URL.
            *   **Ví dụ**: Các hoạt động liên quan đến `/transfers` (chuyển khoản) và `/beneficiaries` (người thụ hưởng) đều có thể thuộc danh mục "Transfer" nếu chúng liên quan đến chức năng chuyển tiền.
        *   **Sắp xếp các danh mục** bằng cách định nghĩa một danh sách `tags` ở cấp độ gốc của tài liệu OAS, với `name` và `description` cho mỗi tag. Thứ tự trong danh sách này sẽ xác định cách các danh mục được hiển thị (ví dụ: "Account" trước "Transfer" nếu người dùng thường quan tâm đến tài khoản trước khi chuyển tiền).
        *   **Sắp xếp các hoạt động bên trong mỗi nhóm** (ví dụ: thứ tự chuẩn cho các phương thức HTTP như GET, POST, DELETE).
    *   **Tổ chức vật lý (Physical Organization)**: Nhóm các tài nguyên bằng cách sử dụng các đường dẫn (paths) có cấu trúc.
        *   **Ví dụ**: Thêm các tiền tố đường dẫn như `/account/` hoặc `/transfer/` vào URL để nhóm các tài nguyên:
            *   Thay vì `/accounts/{id}` và `/beneficiaries/{id}`, bạn có thể có `/account/accounts/{id}` và `/transfer/beneficiaries/{id}`.
        *   Cách này có thể khiến các đường dẫn dễ đoán hơn, nhưng đôi khi cũng có thể làm chúng ít đơn giản hơn.

---

### **7.2 Xác định kích thước API (Sizing an API)**

Các nguyên tắc "Less is more" (ít hơn là tốt hơn) và "a place for everything and everything in its place" (mọi thứ đều có chỗ và mọi thứ đều ở đúng chỗ) rất quan trọng trong thiết kế API. Mỗi khía cạnh của API, bao gồm dữ liệu và mục tiêu, nên được định kích thước một cách khôn ngoan. Đôi khi, một API có vẻ là một khối duy nhất có thể cần được chia thành nhiều API nhỏ hơn.

*   **7.2.1 Chọn mức độ chi tiết dữ liệu (Choosing Data Granularity)**
    Mức độ chi tiết dữ liệu có hai khía cạnh: **số lượng thuộc tính** và **chiều sâu cấu trúc**.
    *   **Số lượng thuộc tính (Number of Properties)**:
        *   Các thuộc tính được cung cấp phải phù hợp về mặt chức năng trong ngữ cảnh mà chúng được sử dụng.
        *   Quá nhiều thuộc tính, dù có liên quan, cũng không làm cho API dễ sử dụng. Nếu có **hơn 20 thuộc tính**, bạn nên xem xét việc tổ chức lại hoặc xem xét lại từng thuộc tính.
        *   API nên yêu cầu lượng dữ liệu tối thiểu để đảm bảo khả năng sử dụng.
    *   **Chiều sâu cấu trúc (Depth)**:
        *   Bạn nên cố gắng **không vượt quá ba cấp độ chiều sâu** trong cấu trúc dữ liệu, vì nhiều hơn có thể làm phức tạp việc thao tác, lập trình và đọc tài liệu.
    *   Cần có sự cân bằng giữa tổ chức và kích thước dữ liệu.

*   **7.2.2 Chọn mức độ chi tiết mục tiêu (Choosing Goal Granularity)**
    *   **Tránh việc đưa quá nhiều dữ liệu vào một mục tiêu duy nhất** mà khiến nó khó quản lý.
        *   **Ví dụ**: Thay vì một mục tiêu `get bank account` (lấy tài khoản ngân hàng) trả về cả thông tin tài khoản và tất cả lịch sử giao dịch, tốt hơn nên có một mục tiêu riêng để lấy danh sách giao dịch (ví dụ: `GET /accounts/{id}/transactions`). Điều này giúp tránh việc phải xử lý lượng lớn dữ liệu không cần thiết.
    *   **Tránh các mục tiêu "tất cả trong một"** (does-it-all goals) bao gồm nhiều mục tiêu con, làm cho chúng phức tạp cho cả nhà thiết kế và người dùng.
        *   **Ví dụ**: Nếu có thể cập nhật địa chỉ của chủ tài khoản thông qua mục tiêu cập nhật tài khoản ngân hàng, điều này có thể trở nên phức tạp nếu tài khoản có nhiều thuộc tính khác cũng có thể cập nhật. Tốt hơn nên có một mục tiêu `update address` độc lập.
    *   Mức độ chi tiết của một mục tiêu được xác định bởi **ngữ cảnh và khả năng sử dụng**.

*   **7.2.3 Chọn mức độ chi tiết API (Choosing API Granularity)**
    *   Nếu các nhóm mục tiêu có thể **hoàn toàn độc lập về chức năng**, bạn nên xem xét việc **chia một API lớn thành các API nhỏ hơn, có chức năng cụ thể hơn**.
    *   Các API nhỏ hơn thì dễ quản lý hơn và có thể được tái sử dụng độc lập trong các ngữ cảnh khác nhau.
    *   Việc chia nhỏ này có thể được thực hiện ngay từ giai đoạn xác định mục tiêu ban đầu.
    *   **Ví dụ**: Một "Banking API" tổng thể có thể được chia thành hai API nhỏ hơn, chuyên biệt hơn như "Bank Account API" (API Tài khoản Ngân hàng) và "Money Transfer API" (API Chuyển tiền).

Việc áp dụng các nguyên tắc về tổ chức và định kích thước này sẽ giúp API của bạn không chỉ hoạt động hiệu quả mà còn dễ hiểu và dễ sử dụng hơn rất nhiều cho các nhà phát triển.