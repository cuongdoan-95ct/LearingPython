Chương 2 tập trung vào việc **thiết kế API từ góc độ của người dùng API**. Mục tiêu chính là tạo ra các API dễ hiểu và dễ sử dụng.

Dưới đây là các chi tiết chính từ chương này:

*   **Tại sao cần thiết kế API vì người dùng?**
    *   API không chỉ đơn thuần là phơi bày dữ liệu và khả năng của phần mềm.
    *   Giống như bất kỳ giao diện người dùng hàng ngày nào, API được tạo ra **vì người dùng của nó để giúp họ đạt được mục tiêu của mình**. Ví dụ về mục tiêu của người dùng với một API mạng xã hội có thể là "chia sẻ ảnh", "thêm bạn", hoặc "liệt kê bạn bè".
    *   Những mục tiêu này tạo thành **bản thiết kế chức năng** cần thiết để thiết kế một API hiệu quả.

*   **Góc nhìn đúng đắn trong thiết kế**
    *   Các nhà thiết kế API có thể học hỏi rất nhiều từ việc thiết kế các giao diện người dùng hàng ngày, dù là vật lý hay ảo.
    *   **Góc nhìn của người tiêu dùng (consumer's perspective)** - tức là quan điểm của người dùng API và phần mềm tiêu thụ API - là nền tảng của thiết kế API. Nó phải là kim chỉ nam cho nhà thiết kế trong suốt quá trình.
    *   Tập trung vào **"cách mọi thứ hoạt động"** (how things work) sẽ dẫn đến các giao diện phức tạp. Ví dụ là thiết bị Kitchen Radar 3000 với các nút và thông tin khó hiểu.
    *   Tập trung vào **"những gì người dùng có thể làm"** (what users can do) sẽ dẫn đến các giao diện đơn giản.
    *   Điều này đúng với cả API: tập trung vào cách phần mềm hoạt động sẽ dẫn đến thảm họa, tập trung vào những gì người dùng có thể làm sẽ giúp mọi thứ diễn ra suôn sẻ.

*   **Thiết kế giao diện của phần mềm**
    *   API là **bảng điều khiển của phần mềm**. Nó có thể được hiểu ngay từ cái nhìn đầu tiên hoặc là một bí ẩn hoàn toàn.
    *   Sử dụng ví dụ về lò vi sóng: Bảng điều khiển là API. Mạch điện bên trong là implementation (cách thực hiện). Người dùng tương tác qua bảng điều khiển (API) để đạt được mục tiêu (làm nóng thức ăn).
    *   API là những gì người dùng nhìn thấy - một biểu diễn của những gì họ có thể làm. Implementation (mã code chạy phía sau) là cách mọi thứ thực sự được thực hiện, nhưng nó **ẩn đối với người dùng**.
    *   API cung cấp biểu diễn của các mục tiêu có thể đạt được khi sử dụng nó. Mục tiêu có thể cần thông tin đầu vào (inputs).
    *   Ví dụ pseudocode: Một API được thiết kế từ góc nhìn provider ("bật magnetron") yêu cầu code phức tạp và dễ lỗi. Một API được thiết kế từ góc nhìn consumer ("làm nóng thức ăn") chỉ cần một dòng code đơn giản, khó lỗi.
    *   **Độ phức tạp hay đơn giản của API phụ thuộc vào góc nhìn bạn tập trung khi thiết kế**. API phải được thiết kế từ **góc nhìn của người tiêu dùng**, không phải của nhà cung cấp. Góc nhìn nhà cung cấp phơi bày cơ chế hoạt động nội bộ, làm cho API khó sử dụng. Góc nhìn người tiêu dùng ẩn cơ chế hoạt động nội bộ, làm cho API đơn giản.

*   **Xác định Mục tiêu của API (Identifying an API’s Goals)**
    *   Bước đầu tiên và quan trọng nhất là xác định những gì người dùng có thể đạt được khi sử dụng API – **xác định các mục tiêu thực sự của API**.
    *   Cần có kiến thức **sâu sắc, chính xác và chi tiết** về: **Ai** có thể sử dụng API? **Họ có thể làm gì**? **Họ làm điều đó như thế nào**? **Họ cần gì** để làm điều đó? **Họ nhận được gì** khi đổi lại?.
    *   Phương pháp xác định mục tiêu:
        *   **Xác định "What" và "How"**: Hỏi "Người dùng muốn làm gì?" ("What") và "Họ làm điều đó như thế nào?" ("How"). Phân rã "What" thành các bước ("How"), mỗi bước trở thành một mục tiêu của API. Ví dụ Shopping API: What = "Mua sản phẩm", How = "thêm sản phẩm vào giỏ hàng" rồi "thanh toán giỏ hàng". Mỗi bước này là một mục tiêu.
        *   **Xác định Inputs và Outputs**: Với mỗi mục tiêu, xác định thông tin đầu vào cần thiết ("What do they need?") và thông tin trả về ("What do they get in return?").
        *   **Xác định Mục tiêu bị thiếu (Identifying missing goals)**: Hỏi **"Đầu vào đến từ đâu?"** ("Where do the inputs come from?") và **"Đầu ra được sử dụng như thế nào?"** ("How are the outputs used?"). Điều tra nguồn gốc đầu vào và cách sử dụng đầu ra giúp phát hiện các bước hoặc mục tiêu bị thiếu trong luồng công việc của người dùng.
        *   **Xác định tất cả Người dùng (Identifying all users)**: Thêm câu hỏi **"Ai là người dùng?"** ("Who are the users?") ngay từ đầu để đảm bảo xác định đầy đủ các "What" khác nhau cho các loại người dùng khác nhau. "Người dùng" có thể là người dùng cuối, ứng dụng tiêu thụ, hoặc vai trò/profile của họ.

*   **Sử dụng API Goals Canvas**
    *   Đây là một công cụ (có thể vẽ trên bảng, giấy hoặc bảng tính) để tổ chức thông tin thu thập được.
    *   Các cột của Canvas bao gồm: **Whos** (Người dùng), **Whats** (Họ có thể làm gì), **Hows** (Họ làm như thế nào/các bước), **Inputs (source)** (Đầu vào, nguồn gốc của nó), **Outputs (usage)** (Đầu ra, cách nó được sử dụng), và **Goals** (Mục tiêu - công thức lại từ How + Inputs + Outputs).
    *   Việc điền canvas là một quá trình lặp đi lặp lại, cần tập trung vào từng phần và tinh chỉnh dần dần.

*   **Tránh Góc nhìn của Nhà cung cấp (Avoiding the Provider’s Perspective)**
    *   Góc nhìn nhà cung cấp là **không thể tránh khỏi** và có thể xuất hiện ở mọi giai đoạn thiết kế.
    *   Nó thường biểu hiện qua việc **phơi bày các khía cạnh không phải là việc của người tiêu dùng**:
        *   **Ảnh hưởng của dữ liệu (Data influences)**: API thiết kế phản ánh trực tiếp cấu trúc hoặc tên của cơ sở dữ liệu nội bộ (ví dụ: "đọc bảng CUSA", "đọc bảng CUSB"). Điều này làm API khó hiểu và khó sử dụng. **Cảnh báo**: Nếu danh sách mục tiêu và dữ liệu của API quá khớp với cơ sở dữ liệu của bạn (cấu trúc hoặc tên), bạn có thể đang thiết kế từ góc nhìn nhà cung cấp.
        *   **Ảnh hưởng của code và logic nghiệp vụ (Code and business logic influences)**: API phơi bày cách dữ liệu được xử lý nội bộ hoặc logic nghiệp vụ phức tạp (ví dụ: thay vì "cập nhật địa chỉ khách hàng", bạn cung cấp "liệt kê địa chỉ", "thêm địa chỉ", "cập nhật trạng thái địa chỉ"). Điều này buộc người dùng phải thực hiện nhiều bước phức tạp và có thể không an toàn.
        *   **Ảnh hưởng của kiến trúc phần mềm (Software architecture influences)**: API phản ánh cấu trúc của các hệ thống backend tương tác (ví dụ: cung cấp mục tiêu "tìm kiếm sản phẩm" chỉ trả về mô tả và mục tiêu "lấy giá sản phẩm" riêng biệt, vì dữ liệu nằm ở hai hệ thống khác nhau). Điều này buộc người dùng phải gọi nhiều API riêng lẻ để lấy thông tin đầy đủ.
        *   **Ảnh hưởng của tổ chức nhân sự (Human organization influences)**: API phơi bày cấu trúc của các phòng ban trong công ty (ví dụ: mục tiêu "chuẩn bị đơn hàng" và "vận chuyển đơn hàng" được phơi bày ra ngoài thay vì chỉ có "thanh toán giỏ hàng"). Điều này không liên quan và làm API khó hiểu.
    *   Tất cả các khía cạnh này của góc nhìn nhà cung cấp đều liên quan đến việc **phơi bày những gì không phải là việc của người tiêu dùng** thông qua API.
    *   **Phát hiện trong API Goals Canvas**: Để chắc chắn tránh góc nhìn nhà cung cấp, hãy thêm câu hỏi cuối cùng vào quá trình xác định mục tiêu: **"Tất cả những điều này có thực sự là việc của người tiêu dùng không?"**. Nếu câu trả lời là không, hãy xem xét lại thiết kế để ẩn các chi tiết nội bộ (dữ liệu, code/logic, kiến trúc, tổ chức nhân sự).

*   **Tóm tắt chương**: Để dễ hiểu và dễ dùng, API phải được thiết kế từ góc nhìn của người tiêu dùng. Thiết kế từ góc nhìn nhà cung cấp dẫn đến API khó hiểu và khó dùng. Danh sách mục tiêu đầy đủ và hướng tới người dùng là nền tảng vững chắc nhất cho API. Xác định người dùng, những gì họ làm, làm thế nào, cần gì và nhận gì là chìa khóa.

**Ví dụ minh họa từ các nguồn:**

*   **So sánh giao diện người dùng (UI) và API:**
    *   API giống như giao diện người dùng (UI) nhưng dành cho phần mềm.
    *   Con người sử dụng UI (trường nhập, nhãn, nút) để tương tác với ứng dụng.
    *   Ứng dụng sử dụng API (hàm, dữ liệu đầu vào/đầu ra) để tương tác với ứng dụng khác.
    *   Ví dụ: ứng dụng di động mạng xã hội sử dụng API camera, API thư viện ảnh, và API remote (web API) của máy chủ mạng xã hội.
    *   Ứng dụng di động là **consumer** (người tiêu dùng), máy chủ backend là **provider** (nhà cung cấp). Các công ty/đội ngũ phát triển cũng được gọi là consumer/provider.

*   **Ví dụ về góc nhìn nhà cung cấp so với người tiêu dùng:**
    *   **Kitchen Radar 3000 API** (góc nhìn nhà cung cấp): Mục tiêu kiểu "bật magnetron", "tắt magnetron". Phơi bày cách hoạt động nội bộ.
    *   **Microwave Oven API** (góc nhìn người tiêu dùng): Mục tiêu "làm nóng thức ăn" với đầu vào "công suất" và "thời lượng". Ẩn cách thực hiện (bật tắt magnetron).
    *   Pseudocode so sánh:
        *   Kitchen Radar (nhà cung cấp): cần nhiều dòng code phức tạp để mô phỏng cách lò vi sóng hoạt động.
        *   Microwave Oven (người tiêu dùng): `heat food at <power> for <duration>`. Chỉ một dòng code đơn giản.

*   **Ví dụ API Goals Canvas cho Shopping API (một phần):**
    *   **Whos**: Customers (Khách hàng), Admin (Quản trị viên).
    *   **Whats**: Buy products (Mua sản phẩm - cho Customers), Manage catalog (Quản lý danh mục - cho Admin).
    *   **Hows / Goals**:
        *   Customers: Search for products (Tìm sản phẩm), Add product to cart (Thêm sản phẩm vào giỏ hàng).
        *   Admin: Add product to catalog (Thêm sản phẩm vào danh mục).
    *   **Inputs (source)**:
        *   Search for products: free query (provided by user) - truy vấn tự do (do người dùng cung cấp).
        *   Add product to cart: Product (search for products) - sản phẩm (từ kết quả tìm kiếm), cart (owned by user) - giỏ hàng (của người dùng).
        *   Add product to catalog: Catalog (owned by user) - danh mục (của người dùng), product (provided by user) - sản phẩm (do người dùng cung cấp).
    *   **Outputs (usage)**:
        *   Search for products: Products (add product to cart) - danh sách sản phẩm (được dùng để thêm vào giỏ hàng).
        *   Add product to cart: Added product (search, get, update, delete, replace) - sản phẩm đã thêm (có thể tìm, lấy, cập nhật, xóa, thay thế).
        *   Add product to catalog: Added product (search, get, update, delete, replace) - sản phẩm đã thêm (có thể tìm, lấy, cập nhật, xóa, thay thế).
    *   (Canvas đầy đủ hơn sẽ bao gồm cả check out, list orders, check order status, và các mục tiêu liên quan khác)

*   **Ví dụ tránh ảnh hưởng của dữ liệu/code/kiến trúc/tổ chức:**
    *   **Ảnh hưởng dữ liệu**: Thay vì `Read CUSA`, `Read CUSB`, cung cấp mục tiêu `Read customer`.
    *   **Ảnh hưởng code/logic**: Thay vì `List customer's addresses`, `Add address`, `Update address status`, cung cấp mục tiêu `Update customer's address`.
    *   **Ảnh hưởng kiến trúc**: Thay vì `Search for products` (chỉ mô tả), `Get product's price`, cung cấp mục tiêu `Search for products` trả về cả mô tả và giá trong một lần gọi.
    *   **Ảnh hưởng tổ chức**: Thay vì `Prepare order`, `Ship order`, các mục tiêu này được xử lý nội bộ sau khi consumer gọi `Check out cart`.

Hy vọng những chi tiết này giúp bạn hiểu rõ hơn về nội dung chương 2 và cách thiết kế API vì người dùng dựa trên các nguồn bạn đã cung cấp.