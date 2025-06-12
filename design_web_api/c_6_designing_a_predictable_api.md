### 6.1. Trở nên nhất quán (Being Consistent)

Một thiết kế nhất quán giúp API trở nên trực quan bằng cách tận dụng kinh nghiệm trước đây của người dùng. Sự không nhất quán sẽ gây ra các biến thể hoặc mâu thuẫn, làm cho giao diện khó hiểu và khó sử dụng hơn.

Sự nhất quán có nghĩa là các yếu tố của API (dữ liệu, mục tiêu, hành vi) được thiết kế theo cách giống nhau trên toàn bộ API, hoặc thậm chí giữa các API khác nhau trong cùng một tổ chức.

#### 6.1.1. Thiết kế dữ liệu nhất quán (Designing Consistent Data)

Dữ liệu là cốt lõi của API. Ý nghĩa, tên, kiểu, định dạng và cấu trúc của tài nguyên, tham số, phản hồi và thuộc tính của chúng phải nhất quán để giúp người dùng dễ dàng hiểu, không phải đoán hoặc học lại cách xử lý dữ liệu ở các phần khác nhau của API.

*   **Tên rõ ràng và nhất quán:**
    *   Nếu một API sử dụng từ `customer_id` trong một tài nguyên (resource), thì nó nên sử dụng `customer_id` ở mọi nơi, thay vì sử dụng `client_id` hoặc `user_id` ở những tài nguyên khác. Sự không nhất quán trong cách đặt tên có thể gây nhầm lẫn.
Ví dụ cụ thể: Trong một API quản lý cửa hàng, nếu tài nguyên /orders trả về trường customer_id, nhưng tài nguyên /invoices trả về client_id cho cùng một khái niệm (ID của khách hàng), thì điều này sẽ làm khó người dùng. Thay vào đó, hãy sử dụng customer_id ở cả hai tài nguyên.
*   **Kiểu dữ liệu và định dạng dễ sử dụng, nhất quán:**
    *   Sử dụng các kiểu dữ liệu cơ bản, phổ biến giữa các ngôn ngữ lập trình như chuỗi (string), số (number), ngày (date) hoặc boolean.
    *   Một khi người dùng đã thấy một thuộc tính ngày tháng (`balanceDate`) được biểu thị bằng chuỗi **ISO 8601** (ví dụ: "2018-03-23"), họ sẽ mong đợi tất cả các thuộc tính ngày tháng khác (`creationDate`, `executionDate`) cũng là chuỗi ISO 8601, chứ không phải Unix timestamp (ví dụ: `1423267200`) hoặc định dạng `YYYY-DD-MM` khác.
    *   **Sự đồng nhất toàn cầu trong thiết kế** là quan trọng.
*   **Tổ chức dữ liệu nhất quán:**
    *   Cấu trúc **URL** của API phải nhất quán. Ví dụ, nếu bạn sử dụng quy ước số nhiều cho các collection như `/accounts/{accountNumber}`, thì bạn nên tuân thủ quy ước này cho tất cả các collection khác (ví dụ: `/transfers/{transferId}` thay vì `/transfer/{transferId}`).
    *   Các danh sách (lists) hoặc các đối tượng (objects) nên có cấu trúc giống nhau. Cách tổ chức các phần tử trong collection cũng phải nhất quán. Nếu tất cả các tài nguyên collection của bạn được biểu thị bằng một đối tượng chứa thuộc tính `items` là một mảng, thì đừng thiết kế một collection khác dưới dạng một mảng đơn giản trực tiếp. Ví dụ, nếu danh sách các đơn hàng trả về một mảng các đối tượng với các trường `id`, `customer_id`, và `total`, thì danh sách các hóa đơn cũng nên có cấu trúc tương tự nếu phù hợp.
  - **Ví dụ cụ thể**: Phản hồi từ `GET /orders` có thể là:
    ```json
    {
      "items": [
        { "id": 1, "customer_id": 101, "total": 99.99 },
        { "id": 2, "customer_id": 102, "total": 149.99 }
      ]
    }
    ```
    Nếu `GET /invoices` trả về:
    ```json
    {
      "invoices": [
        { "invoice_id": 1, "client_id": 101, "amount": 99.99 },
        { "invoice_id": 2, "client_id": 102, "amount": 149.99 }
      ]
    }
    ```
    thì đây là sự không nhất quán. Thay vào đó, `GET /invoices` nên trả về:
    ```json
    {
      "items": [
        { "id": 1, "customer_id": 101, "total": 99.99 },
        { "id": 2, "customer_id": 102, "total": 149.99 }
      ]
    }
    ```
#### 6.1.2. Thiết kế các mục tiêu (Goals) nhất quán (Designing Consistent Goals)

Hành vi của API được xác định bởi các mục tiêu của nó. Tất cả các mục tiêu này phải nhất quán.

*   **Tên mục tiêu nhất quán:**
    *   Các mục tiêu cùng loại hành động nên sử dụng động từ nhất quán. Ví dụ, `read account` và `get user information` là không nhất quán; nên đổi thành `read account` và `read user information`. Đối với **REST API**, việc sử dụng giao thức **HTTP** sẽ giúp đạt được sự nhất quán này một cách tự động, vì cả hai mục tiêu đều sẽ được biểu thị bằng yêu cầu `GET /resource-path`.
*   **Đầu vào nhất quán:**
    *   Đầu vào của các mục tiêu cũng phải nhất quán về tên, kiểu dữ liệu, định dạng và cách tổ chức. Ví dụ, khi liệt kê giao dịch của tài khoản trong một khoảng thời gian, nên sử dụng `fromDate=2015-02-07` và `toDate=2015-03-17` thay vì các định dạng không nhất quán.

*   **Phản hồi thành công**: Các phản hồi thành công (success responses) nên có cấu trúc giống nhau. Ví dụ, nếu một hành động tạo (`POST`) trả về mã trạng thái HTTP `201 Created` và dữ liệu của tài nguyên vừa tạo, thì tất cả các hành động tạo khác cũng nên làm như vậy.
    *   **Ví dụ cụ thể**: Phản hồi từ `POST /orders` có thể là:
    ```json
    {
      "id": 3,
      "customer_id": 103,
      "total": 199.99
    }
    ```
    với mã trạng thái `201 Created`. Tương tự, `POST /invoices` cũng nên trả về:
    ```json
    {
      "id": 3,
      "customer_id": 103,
      "total": 199.99
    }
    ```
    với mã trạng thái `201 Created`, thay vì trả về một cấu trúc khác hoặc mã trạng thái khác như `200 OK`.
*   **Phản hồi lỗi nhất quán:**
    *   Bạn phải trả về các mã trạng thái **HTTP** nhất quán để báo hiệu lỗi. Dữ liệu thông báo lỗi cũng phải nhất quán. Nếu bạn đã định nghĩa các mã lỗi chung như `MISSING_MANDATORY_PROPERTY`, hãy luôn sử dụng mã này trên toàn bộ **API** của bạn.
*   **Luồng mục tiêu nhất quán:**
    *   Nếu các hành động nhạy cảm trước đây (ví dụ: chuyển tiền) bao gồm hai bước (`control <action>` và `do <action>`), thì bất kỳ hành động nhạy cảm mới nào cũng phải được biểu thị bằng các mục tiêu có cùng hành vi.

#### 6.1.3. Bốn cấp độ nhất quán (The Four Levels of Consistency)

Có bốn cấp độ nhất quán có thể áp dụng cho việc thiết kế **API**:

*   **Cấp độ 1: Nhất quán trong một API (Consistency within an API)**
    *   Đảm bảo rằng mọi lựa chọn thiết kế không gây ra biến thể hoặc mâu thuẫn trong **API**. Người dùng phải thấy một giao diện có quy tắc chung.
    *   **Ví dụ**: Trong một API, tất cả các tài nguyên nên sử dụng cấu trúc danh sách với trường `items`. Ví dụ, `GET /orders` và `GET /invoices` đều trả về dữ liệu trong trường `items`.
*   **Cấp độ 2: Nhất quán giữa các API của một tổ chức/công ty/đội nhóm (Consistency across an organization/company/team’s APIs)**
    *   Người dùng **API** không quan tâm liệu các **API** đó được thiết kế bởi một hay nhiều nhà thiết kế. Họ chỉ quan tâm rằng các **API** chia sẻ các tính năng chung để họ có thể hiểu và sử dụng dễ dàng.
    *   Chia sẻ các tính năng chung (như tổ chức dữ liệu, kiểu dữ liệu, định dạng) giúp tăng cường khả năng tương tác giữa các **API**.
    *   **Ví dụ**: Nếu công ty của bạn sử dụng `customer_id` trong một API, thì tất cả các API khác của công ty cũng nên sử dụng `customer_id`, thay vì `user_id` hoặc `client_id`.
*   **Cấp độ 3: Nhất quán với (các) lĩnh vực của một API (Consistency with the domain(s) of an API)**
    *   Tuân thủ các tiêu chuẩn hoặc quy ước phổ biến trong một lĩnh vực cụ thể. Ví dụ, nếu bạn cần tính toán khoảng cách trong **API** điều hướng hàng hải, bạn sẽ sử dụng hải lý chứ không phải dặm hoặc kilomet.
    *   **Ví dụ**: Trong lĩnh vực thương mại điện tử, nhiều API sử dụng tài nguyên `/cart` để quản lý giỏ hàng. Nếu API của bạn cũng sử dụng `/cart`, thì điều này giúp người dùng quen thuộc hơn.
*   **Cấp độ 4: Nhất quán với phần còn lại của thế giới (Consistency with the rest of the world)**
    *   Sử dụng các tiêu chuẩn và quy ước chung đã được chấp nhận rộng rãi. Điều này không chỉ giúp **API** của bạn dễ đoán cho người dùng mới mà còn đơn giản hóa công việc của nhà thiết kế **API**.
    *   **Ví dụ**: Sử dụng mã trạng thái HTTP như `404 Not Found` để biểu thị tài nguyên không tồn tại là một thông lệ chung trên toàn thế giới.

#### 6.1.4. Sao chép người khác: Tuân thủ các quy tắc chung và tiêu chuẩn (Copying Others: Following Common Practices and Meeting Standards)

*   **Tận dụng tiêu chuẩn:** Không cần phải phát minh lại những gì đã có. Có hàng ngàn tiêu chuẩn (ví dụ: **ISO 7000** cho các biểu tượng như Play/Pause, **E.164** cho số điện thoại, **ISO 8601** cho ngày tháng, **ISO 4217** cho tiền tệ) mà bạn có thể sử dụng trong **API** của mình. Việc này giúp người dùng dễ hiểu **API** của bạn hơn, tăng cường khả năng tương tác và đơn giản hóa công việc thiết kế.
*   **Tuân thủ giao thức HTTP:** **REST API** dựa trên giao thức **HTTP**, và bản thân **HTTP** đã cung cấp một khuôn khổ nhất quán và dễ đoán (ví dụ: các phương thức **HTTP** tiêu chuẩn, mã trạng thái **HTTP**).
*   **Các quy ước phổ biến:** Mặc dù không phải tất cả đều được tiêu chuẩn hóa, nhiều **REST API** sử dụng mẫu **URL** phổ biến như `/resources/{resourceId}` (sử dụng tên số nhiều cho collection và ID của từng tài nguyên).
*   **Sao chép các API nổi tiếng:** Bạn có thể sao chép các thiết kế từ các **API** nổi tiếng đã được nhiều người biết đến để đơn giản hóa công việc của mình và giúp người dùng cảm thấy quen thuộc khi sử dụng **API** của bạn lần đầu.

#### 6.1.5. Nhất quán là khó và phải được thực hiện một cách khôn ngoan (Being Consistent is Hard and Must be Done Wisely)

*   **Thách thức của sự nhất quán:** Rất khó để duy trì sự nhất quán, đặc biệt khi có nhiều nhà thiết kế **API** làm việc trên nhiều **API** khác nhau hoặc theo thời gian một mình bạn cũng có thể quên các quy ước đã định.
*   **Giải pháp:** Cần định nghĩa chính thức các quy tắc thiết kế trong một tài liệu được gọi là "Hướng dẫn thiết kế **API**" (API Design Guidelines) hoặc "Hướng dẫn phong cách **API**" (API Design Style Guide).
*   **Cân bằng**: Đôi khi, sự nhất quán quá mức có thể làm API trở nên cứng nhắc. Ví dụ, áp dụng cùng một cấu trúc phản hồi cho mọi tài nguyên có thể không phù hợp nếu một số tài nguyên cần dữ liệu đặc biệt.
    *   **Ví dụ**: Nếu tài nguyên `/orders` trả về danh sách đơn hàng với trường `items`, nhưng tài nguyên `/statistics` trả về dữ liệu thống kê (không phải danh sách), thì việc ép buộc `/statistics` sử dụng trường `items` có thể không hợp lý. Trong trường hợp này, sự nhất quán cần được cân nhắc dựa trên ngữ cảnh.

### 6.2. Trở nên thích ứng (Being Adaptable)

Một cách khác để làm cho **API** dễ đoán là cho phép người dùng chọn những gì họ muốn nhận, làm cho **API** trở nên thích ứng hơn.

#### 6.2.1. Cung cấp và chấp nhận các định dạng khác nhau (Providing and Accepting Different Formats)

*   **Đa dạng định dạng:** **API** có thể cung cấp dữ liệu ở nhiều định dạng khác nhau (ví dụ: **JSON**, **CSV**, **PDF**) tùy thuộc vào nhu cầu của người dùng.
*   **Cách yêu cầu định dạng:**
    *   Sử dụng **tham số truy vấn tùy chỉnh** (Custom query parameter): Ví dụ, thêm `format=CSV` vào **URL** để yêu cầu dữ liệu **CSV**: `GET /accounts/{accountId}/transactions?format=CSV`.
    *   Sử dụng **thỏa thuận nội dung HTTP tiêu chuẩn** (Standard HTTP content negotiation): Đây là cơ chế mạnh mẽ hơn, cho phép người dùng chỉ định định dạng mong muốn bằng cách thêm header `Accept` vào yêu cầu **HTTP** (ví dụ: `Accept: text/csv`). Nếu **API** hỗ trợ, máy chủ sẽ phản hồi với header `Content-type: text/csv` và dữ liệu **CSV**.
    *   Nếu định dạng không được hỗ trợ, máy chủ nên trả về trạng thái **HTTP** `406 Not Acceptable`.

#### 6.2.2. Quốc tế hóa và địa phương hóa (Internationalizing and Localizing)

*   **Ngôn ngữ:** **API** có thể cung cấp các thông báo lỗi và nội dung văn bản khác bằng nhiều ngôn ngữ khác nhau.
    *   Sử dụng **RFC 5646** (Language Tags) để xác định ngôn ngữ một cách chính xác (ví dụ: `fr-FR` cho tiếng Pháp Pháp, `en-US` cho tiếng Anh Mỹ).
    *   Sử dụng header `Accept-Language` trong yêu cầu **HTTP** để người dùng chỉ định ngôn ngữ mong muốn (ví dụ: `Accept-Language: fr-FR`). Máy chủ sẽ phản hồi với header `Content-Language` tương ứng. Nếu ngôn ngữ không được hỗ trợ, trả về `406 Not Acceptable` với mã lỗi rõ ràng (ví dụ: `UNSUPPORTED_LANGUAGE`).
*   **Địa phương hóa (Localization):** Bao gồm việc thích ứng với các quy ước địa phương như hệ đo lường (đế quốc/mét), định dạng ngày/số và kích thước giấy.
*   **Lưu ý:** Việc thêm tính năng quốc tế hóa vào một **API** hiện có có thể dễ dàng về mặt giao diện nhưng có thể phức tạp nếu phần triển khai không được xây dựng với mục đích đó từ đầu.

#### 6.2.3. Lọc, phân trang và sắp xếp (Filtering, Paginating, and Sorting)

*   **Phân trang (Pagination):** Cho phép người dùng lấy một tập hợp con của danh sách lớn.
    *   Sử dụng các tham số truy vấn như `pageSize` (số lượng mục trên mỗi trang) và `page` (số trang) (ví dụ: `GET /accounts/A1/transactions?page=1&size=25`).
*   **Lọc (Filtering):** Cho phép người dùng chỉ định tiêu chí để nhận các mục cụ thể.
    *   Sử dụng tham số truy vấn (ví dụ: `category=restaurant` để lọc giao dịch theo danh mục).
*   **Sắp xếp (Sorting):** Cho phép người dùng yêu cầu dữ liệu được sắp xếp theo một thứ tự cụ thể.
    *   Sử dụng tham số truy vấn (ví dụ: sắp xếp giao dịch theo số tiền giảm dần).

### 6.3. Khám phá (Being Discoverable)

**API** có thể được thiết kế để dễ khám phá bằng cách cung cấp thông tin bổ sung, giống như một cuốn sách có mục lục.

#### 6.3.1. Cung cấp siêu dữ liệu (Providing Metadata)

*   **Siêu dữ liệu phân trang:** Cung cấp thông tin như `page` hiện tại và `totalPages` trong phản hồi để người dùng biết có bao nhiêu trang và họ đang ở đâu.
*   **Siêu dữ liệu hành động:** Liệt kê các hành động khả dụng trên một tài nguyên. Ví dụ, một danh sách các yêu cầu chuyển tiền có thể bao gồm một thuộc tính `actions` cho biết liệu một yêu cầu chuyển tiền có thể bị hủy bỏ hay không (`cancel`).
*   Mục đích của siêu dữ liệu là giúp người dùng hiểu "Tôi đang ở đâu và tôi có thể làm gì".

#### 6.3.2. Tạo API siêu phương tiện (Creating Hypermedia APIs)

*   **Liên kết (Links):** Cung cấp các **URL** trong các biểu diễn tài nguyên (ví dụ: thuộc tính `href`) để liên kết đến các tài nguyên liên quan. Điều này cho phép người dùng điều hướng **API** mà không cần biết trước cấu trúc **URL**.
    *   Ví dụ, khi liệt kê các tài khoản, **API** có thể trả về một liên kết `/accounts/1234567` cho một tài khoản cụ thể. Khi truy vấn tài khoản đó, phản hồi có thể bao gồm một liên kết `/accounts/1234567/transactions` để truy cập các giao dịch của tài khoản.
*   **Khái niệm:** Điều này được gọi là **Hypermedia As The Engine Of Application State (HATEOAS)**, một ràng buộc cơ bản của phong cách kiến trúc **REST**.
*   **Lợi ích:** Tạo điều kiện thuận lợi cho việc khám phá **API** và cập nhật **API**.

#### 6.3.3. Tận dụng giao thức HTTP (Taking Advantage of the HTTP Protocol)

*   **Phương thức OPTIONS:** Sử dụng phương thức **HTTP** `OPTIONS` trên một tài nguyên để xác định các phương thức **HTTP** khả dụng trên tài nguyên đó (ví dụ: `Allow: GET, DELETE`). Điều này cung cấp thông tin về các mục tiêu **API**.
*   **Header Link:** Sử dụng header `Link` trong phản hồi để chỉ ra các định dạng khác có sẵn cho tài nguyên (ví dụ: một danh sách giao dịch có thể có sẵn ở định dạng **PDF** và **CSV** thông qua các liên kết trong header `Link`).
*   **Lưu ý:** Mặc dù hữu ích, các tính năng này có thể không được biết đến rộng rãi và cần được tài liệu hóa cẩn thận để tránh gây nhầm lẫn cho người dùng.

Hy vọng những thông tin chi tiết này giúp bạn hiểu rõ hơn về nội dung của Chương 6.