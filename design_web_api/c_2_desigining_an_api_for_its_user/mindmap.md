# Thiết kế API từ góc độ người dùng (Designing APIs for Users)
* Tập trung vào việc **thiết kế API từ góc độ của người dùng API**. Mục tiêu chính là tạo ra các API dễ hiểu và dễ sử dụng.

### Tại sao cần thiết kế API vì người dùng?
* API không chỉ đơn thuần là phơi bày dữ liệu và khả năng của phần mềm.
* API được tạo ra **vì người dùng của nó để giúp họ đạt được mục tiêu của mình**.
* Ví dụ mục tiêu của người dùng: "chia sẻ ảnh", "thêm bạn", "liệt kê bạn bè".
* Những mục tiêu này tạo thành **bản thiết kế chức năng** cần thiết.

### Góc nhìn đúng đắn trong thiết kế
* Học hỏi từ thiết kế các giao diện người dùng hàng ngày.
* **Góc nhìn của người tiêu dùng (consumer's perspective)** là nền tảng của thiết kế API.
* Tập trung vào **"cách mọi thứ hoạt động"** (how things work) -> giao diện phức tạp.
* Tập trung vào **"những gì người dùng có thể làm"** (what users can do) -> giao diện đơn giản.
* Điều này đúng với cả API: tập trung vào cách phần mềm hoạt động sẽ dẫn đến thảm họa, tập trung vào những gì người dùng có thể làm sẽ giúp mọi thứ diễn ra suôn sẻ.

### Thiết kế giao diện của phần mềm
* API là **bảng điều khiển của phần mềm**.
* Ví dụ lò vi sóng: Bảng điều khiển là API, mạch điện là implementation.
* API là những gì người dùng nhìn thấy - một biểu diễn của những gì họ có thể làm.
* Implementation **ẩn đối với người dùng**.
* API cung cấp biểu diễn của các mục tiêu có thể đạt được.
* **Ví dụ pseudocode**: API "bật magnetron" (provider) phức tạp hơn API "làm nóng thức ăn" (consumer).
* **Độ phức tạp hay đơn giản của API phụ thuộc vào góc nhìn bạn tập trung khi thiết kế**. API phải được thiết kế từ **góc nhìn của người tiêu dùng**, không phải của nhà cung cấp.

### Xác định Mục tiêu của API (Identifying an API’s Goals)
* Bước đầu tiên và quan trọng nhất là xác định **các mục tiêu thực sự của API**.
* Cần có kiến thức **sâu sắc, chính xác và chi tiết** về:
    - **Ai** có thể sử dụng API?
    - **Họ có thể làm gì**?
    - **Họ làm điều đó như thế nào**?
    - **Họ cần gì** để làm điều đó?
    - **Họ nhận được gì** khi đổi lại?
* Phương pháp xác định mục tiêu:
    - **Xác định "What" và "How"**: "Người dùng muốn làm gì?" ("What") và "Họ làm điều đó như thế nào?" ("How").
    - **Xác định Inputs và Outputs**: Với mỗi mục tiêu, xác định thông tin đầu vào cần thiết và thông tin trả về.
    - **Xác định Mục tiêu bị thiếu**: Hỏi **"Đầu vào đến từ đâu?"** và **"Đầu ra được sử dụng như thế nào?"**.
    - **Xác định tất cả Người dùng**: Thêm câu hỏi **"Ai là người dùng?"** ngay từ đầu.

### Sử dụng API Goals Canvas
* Một công cụ để tổ chức thông tin thu thập được.
* Các cột của Canvas bao gồm:
    - **Whos** (Người dùng)
    - **Whats** (Họ có thể làm gì)
    - **Hows** (Họ làm như thế nào/các bước)
    - **Inputs (source)** (Đầu vào, nguồn gốc của nó)
    - **Outputs (usage)** (Đầu ra, cách nó được sử dụng)
    - **Goals** (Mục tiêu - công thức lại từ How + Inputs + Outputs)
* Việc điền canvas là một quá trình lặp đi lặp lại.

### Tránh Góc nhìn của Nhà cung cấp (Avoiding the Provider’s Perspective)
* Góc nhìn nhà cung cấp là **không thể tránh khỏi** và có thể xuất hiện ở mọi giai đoạn thiết kế.
* Biểu hiện qua việc **phơi bày các khía cạnh không phải là việc của người tiêu dùng**:
    - **Ảnh hưởng của dữ liệu (Data influences)**: API phản ánh cấu trúc hoặc tên của cơ sở dữ liệu nội bộ.
    - **Ảnh hưởng của code và logic nghiệp vụ (Code and business logic influences)**: API phơi bày cách dữ liệu được xử lý nội bộ.
    - **Ảnh hưởng của kiến trúc phần mềm (Software architecture influences)**: API phản ánh cấu trúc của các hệ thống backend tương tác.
    - **Ảnh hưởng của tổ chức nhân sự (Human organization influences)**: API phơi bày cấu trúc của các phòng ban trong công ty.
* Tất cả các khía cạnh này đều liên quan đến việc **phơi bày những gì không phải là việc của người tiêu dùng**.
* **Phát hiện trong API Goals Canvas**: Thêm câu hỏi: **"Tất cả những điều này có thực sự là việc của người tiêu dùng không?"**.

## Tóm tắt chương
* Để dễ hiểu và dễ dùng, API phải được thiết kế từ góc nhìn của người tiêu dùng.
* Thiết kế từ góc nhìn nhà cung cấp dẫn đến API khó hiểu và khó dùng.
* Danh sách mục tiêu đầy đủ và hướng tới người dùng là nền tảng vững chắc nhất cho API.
* Xác định người dùng, những gì họ làm, làm thế nào, cần gì và nhận gì là chìa khóa.

### Ví dụ minh họa từ các nguồn:

#### So sánh giao diện người dùng (UI) và API
* API giống như giao diện người dùng (UI) nhưng dành cho phần mềm.
* Con người sử dụng UI, Ứng dụng sử dụng API.
* **Ví dụ**: Ứng dụng di động mạng xã hội sử dụng API camera, API thư viện ảnh, và API remote (web API) của máy chủ mạng xã hội.
* Ứng dụng di động là **consumer**, máy chủ backend là **provider**.

#### Ví dụ về góc nhìn nhà cung cấp so với người tiêu dùng
* **Kitchen Radar 3000 API** (góc nhìn nhà cung cấp): Mục tiêu kiểu "bật magnetron".
* **Microwave Oven API** (góc nhìn người tiêu dùng): Mục tiêu "làm nóng thức ăn" với đầu vào "công suất" và "thời lượng".
* **Pseudocode so sánh**: Kitchen Radar (nhà cung cấp) cần nhiều dòng code phức tạp; Microwave Oven (người tiêu dùng) chỉ cần một dòng code đơn giản.

#### Ví dụ API Goals Canvas cho Shopping API (một phần)
* **Whos**: Customers, Admin.
* **Whats**: Buy products (Customers), Manage catalog (Admin).
* **Hows / Goals**:
    - Customers: Search for products, Add product to cart.
    - Admin: Add product to catalog.
* **Inputs (source)**:
    - Search for products: free query (provided by user).
    - Add product to cart: Product (search for products), cart (owned by user).
    - Add product to catalog: Catalog (owned by user), product (provided by user).
* **Outputs (usage)**:
    - Search for products: Products (add product to cart).
    - Add product to cart: Added product (search, get, update, delete, replace).
    - Add product to catalog: Added product (search, get, update, delete, replace).

#### Ví dụ tránh ảnh hưởng của dữ liệu/code/kiến trúc/tổ chức
* **Ảnh hưởng dữ liệu**: Thay vì `Read CUSA`, `Read CUSB`, cung cấp mục tiêu `Read customer`.
* **Ảnh hưởng code/logic**: Thay vì `List customer's addresses`, `Add address`, `Update address status`, cung cấp mục tiêu `Update customer's address`.
* **Ảnh hưởng kiến trúc**: Thay vì `Search for products` (chỉ mô tả), `Get product's price`, cung cấp mục tiêu `Search for products` trả về cả mô tả và giá trong một lần gọi.
* **Ảnh hưởng tổ chức**: Thay vì `Prepare order`, `Ship order`, các mục tiêu này được xử lý nội bộ sau khi consumer gọi `Check out cart`.