**3.1 Giới thiệu về REST API**

Để thiết kế một REST API, trước tiên cần hiểu cách nó hoạt động. Chúng ta sẽ phân tích một lệnh gọi REST API để lấy thông tin sản phẩm, như đã thấy ở phần giới thiệu chương (Hình 3.1). Lấy ví dụ `GET /products/P123`. Lệnh gọi này sử dụng giao thức HTTP.

*   **Phân tích một lệnh gọi REST API**:
    *   Khi người dùng muốn lấy thông tin sản phẩm có ID là P123 bằng cách sử dụng giao diện lập trình REST API.
    *   Người dùng (client) sẽ gửi một yêu cầu HTTP có dạng `GET /products/P123` đến máy chủ cung cấp API (server).
    *   Yêu cầu này bao gồm **phương thức HTTP** là `GET` và **đường dẫn (path)** là `/products/P123`. Đường dẫn này là một địa chỉ xác định một **tài nguyên (resource)** trên máy chủ; trong trường hợp này, nó xác định sản phẩm P123 trong tập hợp các sản phẩm (`products`). Phương thức HTTP (`GET`) cho biết người dùng muốn làm gì với tài nguyên này: `GET` có nghĩa là họ muốn **truy xuất (retrieve)** tài nguyên.
    *   Từ góc độ chức năng, yêu cầu này có nghĩa là "Chào, tôi có thể lấy thông tin về sản phẩm có ID là P123 được không?".
    *   Nhưng từ góc độ giao thức HTTP, nó có nghĩa là "Chào, tôi có thể lấy tài nguyên được xác định bởi đường dẫn `/products/P123` được không?".
    *   Yêu cầu HTTP có thể có phần thân (body) chứa nội dung của tài nguyên cần gửi đến máy chủ (ví dụ: để tạo, cập nhật hoặc thay thế tài nguyên). Phần thân này có thể là bất kỳ loại nào: tài liệu JSON, tệp văn bản, hoặc ảnh, v.v..
    *   Đáp lại, máy chủ sẽ trả về một **phản hồi HTTP**. Phản hồi này luôn chứa một **mã trạng thái (status code)** và cụm từ giải thích (reason phrase). Mã trạng thái này cho biết quá trình xử lý yêu cầu đã diễn ra như thế nào – thành công hay thất bại. Trong ví dụ này là `200 OK`, cho biết mọi thứ đều ổn. (Tài liệu lưu ý rằng việc trao đổi HTTP này đã được đơn giản hóa để chỉ tập trung vào các yếu tố liên quan).
    *   Phần đầu của phản hồi có thể được theo sau bởi một **phần thân phản hồi (response body)** chứa nội dung của tài nguyên đã được thao tác bởi yêu cầu. Trong ví dụ này, phần thân phản hồi chứa dữ liệu JSON mô tả sản phẩm được yêu cầu. Nội dung này cũng có thể là bất kỳ loại nào.

*   **Các nguyên tắc cơ bản của HTTP**:
    *   Giao thức HTTP được sử dụng cho cả việc duyệt web (bởi con người) và các API web (bởi phần mềm).
    *   HTTP sử dụng các **yêu cầu** và **phản hồi**. Máy khách (client) gửi yêu cầu đến máy chủ (server) và máy chủ gửi phản hồi trở lại.
    *   Các **phương thức HTTP** (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`) cho biết hành động mà máy khách muốn thực hiện.
    *   **Đường dẫn (paths)** xác định tài nguyên mà hành động sẽ áp dụng.

*   **Các nguyên tắc cơ bản của REST API**:
    *   REST API sử dụng giao thức HTTP để cho phép phần mềm tương tác với phần mềm.
    *   Cơ chế hoạt động của REST API tương tự như cách trình duyệt web tương tác với máy chủ web.
    *   Trong REST API, các mục tiêu chức năng được biểu diễn bằng các **hành động (actions)** được thực hiện trên **tài nguyên (resources)**.
    *   Các phương thức HTTP như `GET`, `POST`, `PUT`, `PATCH`, và `DELETE` được sử dụng để ánh xạ tới các hành động tiêu chuẩn trên tài nguyên (thường được gọi là các chức năng CRUD: Create, Read, Update, Delete - Tạo, Đọc, Cập nhật, Xóa).
    *   Đường dẫn (paths) trong REST API được sử dụng để xác định các tài nguyên này.

**3.2 Chuyển đổi các mục tiêu API thành một REST API**

Quá trình thiết kế giao diện lập trình REST API là việc chuyển đổi các mục tiêu API (đã xác định bằng API goals canvas) và các đầu vào, đầu ra của chúng thành các cấu trúc của REST API. Tài liệu đề xuất một phương pháp đơn giản gồm bốn bước cho quá trình này (xem Hình 3.5):
1.  Xác định tài nguyên và các mối quan hệ của chúng.
2.  Xác định hành động, các tham số và giá trị trả về của chúng.
3.  Thiết kế đường dẫn cho tài nguyên.
4.  Biểu diễn hành động bằng HTTP.

*   **3.2.1 Xác định tài nguyên và các mối quan hệ của chúng bằng API goals canvas**:
    *   Đây là **bước đầu tiên** trong việc chuyển đổi mục tiêu API thành REST API.
    *   Tài liệu nguồn cho việc này là **API goals canvas** (xem Chương 2), mô tả người dùng là ai, họ có thể làm gì, làm như thế nào, cần gì làm đầu vào và nhận gì làm đầu ra.
    *   Để xác định các **tài nguyên (resources)** tiềm năng, bạn nên xem xét các mục tiêu API trong canvas và **liệt kê tất cả các danh từ mà động từ chính của mục tiêu áp dụng**.
        *   Ví dụ: Trong mục tiêu "thêm sản phẩm vào danh mục" (`add a product to the catalog`), 'sản phẩm' (`product`) và 'danh mục' (`catalog`) được xác định là tài nguyên vì động từ "thêm" (`add`) áp dụng cho chúng.
        *   Tuy nhiên, trong "tìm kiếm sản phẩm trong danh mục bằng truy vấn tự do" (`search for products in the catalog using a free query`), 'truy vấn tự do' (`free query`) là một danh từ nhưng không phải tài nguyên vì động từ chính "tìm kiếm" (`search`) không áp dụng trực tiếp cho nó; động từ này áp dụng cho tài nguyên 'danh mục' (`catalog`).
    *   Để xác định **mối quan hệ giữa các tài nguyên**, bạn xem xét các mục tiêu có đề cập đến **nhiều hơn một tài nguyên**.
        *   Các mục tiêu như "thêm sản phẩm vào danh mục" hoặc "tìm kiếm sản phẩm trong danh mục" gợi ý một mối quan hệ trong đó tài nguyên danh mục (`catalog`) **"chứa nhiều"** tài nguyên sản phẩm (`product`).
        *   Một tài nguyên chứa các tài nguyên khác cùng loại được gọi là **tài nguyên tập hợp (collection resource)**.
        *   Tài nguyên cũng có thể liên kết với các tài nguyên khác.
    *   Ví dụ minh họa bằng API goals canvas cho phần quản lý danh mục (Hình 3.6): Liệt kê các mục tiêu như "Add product to catalog", "Get product's information", "Update product's information", "Replace product", "Delete product", "Search for products". Phân tích động từ/danh từ chính: 'add' áp dụng cho 'product' và 'catalog'; 'get', 'update', 'replace', 'delete' áp dụng cho 'product'; 'search' áp dụng cho 'catalog'. Từ đó xác định tài nguyên là `catalog` và `product`. Mối quan hệ là `catalog` chứa nhiều `product`. (Hình 3.7 minh họa phân tích này, Hình 3.8 minh họa mối quan hệ).

*   **3.2.2 Xác định hành động và các tham số, giá trị trả về của chúng bằng API goals canvas**:
    *   Đây là **bước thứ hai** trong quá trình chuyển đổi.
    *   Trong REST API, các mục tiêu được biểu diễn bằng **các hành động được thực hiện trên tài nguyên**.
    *   Để xác định một **hành động (action)**, bạn lấy **động từ chính** của mục tiêu và liên kết nó với **tài nguyên** mà nó áp dụng.
        *   Đối với các mục tiêu chỉ liên quan đến một tài nguyên (ví dụ: "get product"), động từ ('get') áp dụng trực tiếp cho tài nguyên đó ('product').
        *   Đối với các mục tiêu liên quan đến nhiều tài nguyên (ví dụ: "add product to catalog" hoặc "search for products in catalog"), động từ được liên kết với **tài nguyên chính** (ví dụ: 'catalog') đang được sử dụng hoặc sửa đổi. Các tài nguyên khác liên quan thường trở thành tham số hoặc giá trị trả về.
    *   **API goals canvas** cung cấp danh sách đầy đủ các **đầu vào (parameters)** và **đầu ra (returns)** cho mỗi mục tiêu.
    *   Khi xác định tham số, bạn cần **lọc các đầu vào** được liệt kê trong canvas để loại trừ tài nguyên mà hành động được áp dụng. Ví dụ: Khi áp dụng hành động 'add' cho tài nguyên 'catalog', 'thông tin sản phẩm' (`product information`) là một tham số đầu vào.
    *   Các đầu ra được liệt kê trong canvas trở thành **giá trị trả về** của hành động. Ví dụ: Hành động 'search' trên tài nguyên 'catalog' trả về 'các tài nguyên sản phẩm phù hợp' (`matching product resources`).
    *   Ví dụ minh họa (Hình 3.9, Hình 3.10, Hình 3.11): Phân tích các mục tiêu quản lý danh mục để xác định hành động và liên kết chúng với tài nguyên `catalog` hoặc `product`. Ví dụ: "Add product to catalog" -> hành động `add` áp dụng cho `catalog`, cần `product info` làm tham số, trả về `added product`. "Get product" -> hành động `get` áp dụng cho `product`, không cần tham số, trả về `product info`.

*   **3.2.3 Thiết kế đường dẫn cho tài nguyên**:
    *   Đường dẫn (paths) trong REST API được sử dụng để **xác định các tài nguyên**.
    *   Việc thiết kế đường dẫn rất quan trọng và cần đảm bảo tính **thân thiện với người dùng**.
    *   Nên sử dụng các đường dẫn chỉ rõ tài nguyên mà chúng đại diện, ví dụ `/catalog` cho tài nguyên danh mục và `/product-{productId}` cho tài nguyên sản phẩm.
    *   **Định dạng được áp dụng rộng rãi nhất** cho đường dẫn tài nguyên trong REST API là `/{tên số nhiều phản ánh loại mục của tập hợp}/{id của mục}` (ví dụ: `/products/{productId}`). Cách này thể hiện mối quan hệ phân cấp giữa các tài nguyên và sử dụng tên số nhiều cho các tài nguyên tập hợp (collection resources).
    *   Ví dụ (Hình 3.13): So sánh các cách đặt tên đường dẫn khác nhau và nhấn mạnh việc sử dụng tên số nhiều và chỉ rõ tài nguyên.

*   **3.2.4 Biểu diễn hành động bằng HTTP**:
    *   Sau khi xác định tài nguyên và hành động, bước tiếp theo là **ánh xạ các hành động này tới các phương thức HTTP tiêu chuẩn**.
    *   **GET**: Được sử dụng để **đọc (read)** hoặc **truy xuất (retrieve)** tài nguyên. Yêu cầu `GET` không nên có phần thân và không nên gây ra thay đổi trạng thái trên máy chủ (trừ các thay đổi liên quan đến logging hoặc caching). Ví dụ: Hành động "get product" được ánh xạ thành `GET /products/{productId}`. (Hình 3.17).
    *   **POST**: Được sử dụng để **tạo (create)** một tài nguyên mới trong một tập hợp, hoặc **thêm (add)** một mục vào một tập hợp, hoặc **kích hoạt (trigger)** một hành động không dễ dàng ánh xạ tới các phương thức CRUD tiêu chuẩn. Tham số yêu cầu được truyền trong phần thân yêu cầu. Ví dụ: Hành động "add product to catalog" được ánh xạ thành `POST /products`. (Hình 3.18).
    *   **PUT**: Được sử dụng để **thay thế (replace)** hoàn toàn một tài nguyên hiện có, hoặc **tạo (create)** một tài nguyên mới với định danh đã biết trước. Tham số yêu cầu được truyền trong phần thân yêu cầu. Ví dụ: Hành động "replace product" được ánh xạ thành `PUT /products/{productId}`. (Hình 3.20).
    *   **PATCH**: Được sử dụng để **cập nhật một phần (partially update)** tài nguyên. Tham số yêu cầu (chỉ chứa các thuộc tính cần cập nhật) được truyền trong phần thân yêu cầu. Ví dụ: Hành động "update product's information" được ánh xạ thành `PATCH /products/{productId}`. (Hình 3.19).
    *   **DELETE**: Được sử dụng để **xóa bỏ (remove/delete)** tài nguyên. Ví dụ: Hành động "delete product" được ánh xạ thành `DELETE /products/{productId}`. (Hình 3.16).
    *   Tài liệu cung cấp một bảng tổng hợp các phương thức HTTP phổ biến ngoài CRUD, bao gồm `POST`, `GET`, `PUT`, `PATCH`, `DELETE` và các mục đích sử dụng phổ biến của chúng (Bảng 3.1).
    *   Tài liệu cũng cung cấp một "cheat sheet" tổng hợp quá trình chuyển đổi từ mục tiêu sang REST API, bao gồm nhận diện tài nguyên, hành động, thiết kế đường dẫn và chọn phương thức HTTP (Hình 3.21).

**3.3 Thiết kế dữ liệu của API**

Sau khi chuyển đổi mục tiêu thành tài nguyên và hành động được biểu diễn bằng đường dẫn và phương thức HTTP, bước tiếp theo là **thiết kế chi tiết dữ liệu** sẽ được trao đổi thông qua các **tham số (parameters)** và **giá trị trả về (returns)**. Quá trình này giống như thiết kế cấu trúc dữ liệu trong lập trình. Tài liệu đề xuất một quy trình gồm các bước (xem Hình 3.22):
1.  Thiết kế khái niệm (concepts).
2.  Thiết kế phản hồi (responses) từ khái niệm.
3.  Thiết kế tham số (parameters) từ khái niệm hoặc phản hồi.
4.  Kiểm tra nguồn dữ liệu của tham số.

*   **3.3.1 Thiết kế khái niệm**:
    *   Các khái niệm đã xác định (tài nguyên) sẽ được trao đổi giữa người dùng và nhà cung cấp thông qua tham số và giá trị trả về.
    *   Khi thiết kế một cấu trúc dữ liệu cho khái niệm, phải đảm bảo tính **thân thiện với người dùng** và **không để lộ các chi tiết nội bộ** của nhà cung cấp.
    *   Bắt đầu bằng cách **liệt kê các thuộc tính (properties)** của khái niệm. Ví dụ với tài nguyên sản phẩm, các thuộc tính có thể là `reference`, `name`, `price`, `dateAdded`, `unavailable`, `warehouses`, `description`, `supplier`.
    *   Đối với mỗi thuộc tính, cần phân tích xem nó có **dễ hiểu** không, có **thật sự liên quan đến người dùng** không và có **thật sự hữu ích** không. Có thể cần **đổi tên**, **xóa bỏ**, hoặc **tái cấu trúc** các thuộc tính để đảm bảo tính thân thiện với người dùng. Ví dụ: xóa thuộc tính `warehouses` vì không liên quan đến người dùng; đổi tên `unavailable` thành `definitelyOutOfStock` để rõ ràng hơn.
    *   Thông tin quan trọng nhất về một thuộc tính là **tên** của nó; tên càng tự giải thích càng tốt.
    *   Ngoài tên, cần xác định **loại dữ liệu (type)** cho mỗi thuộc tính. Nên sử dụng các **loại dữ liệu cơ bản, dễ di chuyển** giữa các ngôn ngữ lập trình như `string`, `number`, `date`, hoặc `boolean`.
    *   Cần xác định rõ thuộc tính đó là **bắt buộc (required)** hay **tùy chọn (optional)**. Trạng thái bắt buộc/tùy chọn có thể thay đổi tùy ngữ cảnh.
    *   Có thể thêm **mô tả (description)** tùy chọn khi tên và loại dữ liệu không đủ để mô tả rõ ràng thuộc tính. Mô tả này có thể bao gồm thông tin bổ sung không thể hiện rõ qua tên hoặc loại.
    *   Ví dụ chi tiết thuộc tính sản phẩm (Hình 3.25): Liệt kê các thuộc tính như `reference` (string, yes, Unique ID), `name` (string, yes, Product's name), `price` (number, yes, Price in USD), v.v. Có thể có thuộc tính phức tạp là đối tượng (`supplier`). (Hình 3.23 minh họa quá trình thiết kế khái niệm, Hình 3.24 minh họa các đặc điểm của thuộc tính).

*   **3.3.2 Thiết kế phản hồi từ khái niệm**:
    *   Cùng một khái niệm có thể có **các biểu diễn khác nhau** trong các ngữ cảnh khác nhau khi được trả về trong phản hồi.
    *   Thiết kế phản hồi dựa trên khái niệm gốc nhưng được **điều chỉnh theo ngữ cảnh của hành động**.
    *   Ví dụ: Hành động "get product" có thể trả về thông tin sản phẩm **đầy đủ**, trong khi hành động "search for products" có thể chỉ trả về **thông tin tóm tắt** (ví dụ: chỉ `reference`, `name`, `price` và tên nhà cung cấp). Điều này phụ thuộc vào việc người dùng cần những thông tin gì trong từng ngữ cảnh.
    *   (Hình 3.26 minh họa khái niệm sản phẩm xuất hiện trong các ngữ cảnh hành động khác nhau, Hình 3.27 minh họa các biểu diễn khác nhau của sản phẩm trong phản hồi).

*   **3.3.3 Thiết kế tham số từ khái niệm hoặc phản hồi**:
    *   Tương tự như phản hồi, các tham số đầu vào cũng có thể có **các biểu diễn khác nhau** tùy thuộc vào ngữ cảnh của hành động.
    *   Thiết kế tham số dựa trên khái niệm gốc hoặc biểu diễn phản hồi nhưng được **điều chỉnh cho phù hợp với thông tin mà người dùng cần cung cấp**.
    *   Ví dụ: Đối với hành động "add product", người dùng không cần cung cấp `reference` vì nó sẽ được tạo bởi backend. Đối với "update product" hoặc "replace product", người dùng cần cung cấp `reference` (thường là trong đường dẫn). Trong tất cả các trường hợp này, người dùng có thể không cần cung cấp đầy đủ thông tin nhà cung cấp (`supplier` object) mà chỉ cần `supplierReference`.
    *   **Tham số chỉ nên chứa dữ liệu cần thiết**; nó không nên bao gồm dữ liệu mà backend xử lý độc quyền.
    *   (Hình 3.28 minh họa khái niệm sản phẩm xuất hiện trong các ngữ cảnh tham số khác nhau, Hình 3.29 minh họa các biểu diễn khác nhau của sản phẩm trong tham số).

*   **3.3.4 Kiểm tra nguồn dữ liệu của tham số**:
    *   Người dùng **phải có khả năng cung cấp tất cả dữ liệu** cần thiết cho một tham số, hoặc vì họ đã biết thông tin đó, hoặc vì họ có thể lấy nó từ một mục tiêu API khác.
    *   Việc kiểm tra này là rất quan trọng vì nó giúp **phát hiện các mục tiêu bị thiếu** hoặc các vấn đề liên quan đến việc để lộ **quan điểm của nhà cung cấp**.
    *   Quy trình kiểm tra (Hình 3.30): Đối với mỗi thuộc tính trong tham số, xác định nguồn gốc của nó. Nếu nó đến từ người dùng hoặc từ một mục tiêu API đã xác định (có thể cần cập nhật phản hồi của mục tiêu đó để bao gồm thông tin này) thì ổn. Nếu không có nguồn gốc rõ ràng, có thể thuộc tính đó không thực sự cần thiết (xóa nó) hoặc cần thêm một mục tiêu API mới để cung cấp thông tin đó.

*   **3.3.5 Thiết kế các tham số khác**:
    *   Một số tham số, như tham số truy vấn tự do (`free-query`) trong hành động "search for products", không dựa trên các khái niệm đã xác định trước.
    *   Đối với các tham số này (query parameters hoặc body parameters không dựa trên concept), nguyên tắc thiết kế vẫn giữ nguyên: **chọn biểu diễn thân thiện với người dùng** và **kiểm tra khả năng cung cấp dữ liệu** của người dùng.

**3.4 Tìm sự cân bằng khi đối mặt với các thách thức thiết kế**

Khi lựa chọn một kiểu API cụ thể (ví dụ: REST), đôi khi sẽ gặp phải những hạn chế hoặc khó khăn trong việc biểu diễn một mục tiêu nào đó theo đúng mô hình của kiểu đó. Đôi khi, biểu diễn tuân thủ mô hình lại không thân thiện với người dùng. Trong những trường hợp như vậy, không có giải pháp hoàn hảo và người thiết kế API sẽ phải **thực hiện các đánh đổi (trade-offs)**.

*   **3.4.1 Ví dụ về đánh đổi trong REST**:
    *   Lấy ví dụ mục tiêu "check out cart" trong API mua sắm.
    *   Có thể có nhiều cách để biểu diễn mục tiêu này trong REST, mỗi cách có ưu và nhược điểm riêng về sự tuân thủ mô hình REST và tính thân thiện với người dùng.
    *   Các lựa chọn có thể bao gồm:
        *   `POST /cart/check-out`: Sử dụng động từ trong đường dẫn (`check-out`), điều này đi ngược lại nguyên tắc REST khuyến khích sử dụng danh từ trong đường dẫn. Tuy nhiên, nó có thể rất thân thiện với người dùng vì dễ hiểu.
        *   `PATCH /cart`: Coi việc thanh toán là một cập nhật trạng thái của tài nguyên giỏ hàng (`cart`). Cách này tuân thủ mô hình REST hơn, nhưng có thể hơi khó hiểu hoặc khó sử dụng hơn.
        *   `POST /orders`: Coi việc thanh toán là việc tạo ra một tài nguyên mới là đơn hàng (`order`). Cách này có thể được xem là tuân thủ mô hình REST nhất, nhưng có thể không trực quan cho người dùng ("thanh toán giỏ hàng" được biểu diễn bằng "tạo đơn hàng").

*   **3.4.2 Cân bằng giữa tính thân thiện với người dùng và sự tuân thủ**:
    *   Thách thức là tìm ra **sự cân bằng phù hợp** giữa việc tuân thủ nghiêm ngặt mô hình của kiểu API đã chọn và đảm bảo tính thân thiện với người dùng.
    *   Việc nắm vững các nguyên tắc cơ bản của kiểu API giúp tìm ra các giải pháp gần nhất với mô hình.
    *   Tuy nhiên, việc tìm ra giải pháp tốt nhất đòi hỏi **thực hành**, **quan sát** các API khác, và quan trọng nhất là **trao đổi** với người dùng (developers) và các nhà thiết kế API khác.
    *   (Hình 3.34 minh họa sự cân bằng giữa tính thân thiện với người dùng và sự tuân thủ kiểu API).

**3.5 Hiểu lý do tại sao kiến trúc REST lại quan trọng đối với việc thiết kế bất kỳ loại API nào**

REST API được áp dụng rộng rãi, nhưng lý do quan trọng hơn khiến nó được chọn làm ví dụ chính trong sách là vì nó dựa trên **kiến trúc REST (REST architectural style)**, một bộ các nguyên tắc thiết kế hệ thống phân tán. Những nguyên tắc này rất hữu ích và quan trọng cần biết khi **thiết kế bất kỳ loại API nào**, không chỉ REST API.

*   **3.5.1 Giới thiệu kiến trúc REST**:
    *   Kiến trúc REST được giới thiệu bởi Roy Fielding vào năm 2000 trong luận án tiến sĩ của ông. Ông đã phát triển nó trong quá trình làm việc trên giao thức HTTP 1.1.
    *   Mục tiêu của kiến trúc REST là **tạo điều kiện xây dựng các hệ thống phân tán hiệu quả, có khả năng mở rộng và đáng tin cậy**. Một hệ thống phân tán bao gồm các thành phần phần mềm nằm trên các máy tính khác nhau, làm việc cùng nhau và giao tiếp qua mạng. (Hình 3.35 minh họa một hệ thống phân tán).
    *   Để được coi là **RESTful**, kiến trúc phần mềm cần tuân thủ **sáu ràng buộc** sau:
        1.  **Client/server separation (Tách biệt máy khách/máy chủ)**: Phải có sự tách biệt rõ ràng về trách nhiệm giữa các thành phần máy khách (ví dụ: ứng dụng di động) và máy chủ API khi chúng làm việc và giao tiếp cùng nhau.
        2.  **Statelessness (Không trạng thái)**: Tất cả thông tin cần thiết để thực thi một yêu cầu đều nằm trong chính yêu cầu đó. Máy chủ không lưu trữ ngữ cảnh (context) của máy khách trong một phiên (session) giữa các yêu cầu.
        3.  **Cacheability (Khả năng bộ nhớ đệm)**: Phản hồi đối với một yêu cầu phải cho biết liệu nó có thể được lưu trữ (để máy khách có thể tái sử dụng thay vì gọi lại cùng một yêu cầu) và trong bao lâu.
        4.  **Layered system (Hệ thống phân lớp)**: Khi một máy khách tương tác với máy chủ, nó chỉ nhận biết về máy chủ đó chứ không biết về hạ tầng ẩn đằng sau. Máy khách chỉ nhìn thấy một lớp của hệ thống.
        5.  **Code on demand (Mã theo yêu cầu)**: Máy chủ có thể chuyển mã thực thi đến máy khách (ví dụ: JavaScript). Ràng buộc này là **tùy chọn**.
        6.  **Uniform interface (Giao diện đồng nhất)**: Tất cả các tương tác phải được định hướng bởi khái niệm tài nguyên được xác định, được thao tác thông qua biểu diễn trạng thái tài nguyên và các phương thức tiêu chuẩn. Các tương tác cũng phải cung cấp tất cả siêu dữ liệu (metadata) cần thiết để hiểu các biểu diễn và biết có thể làm gì với tài nguyên đó. Đây là ràng buộc cơ bản nhất của REST và là nguồn gốc của tên gọi Representational State Transfer (Chuyển đổi trạng thái theo biểu diễn).

*   **3.5.2 Tác động của các ràng buộc REST đối với thiết kế API**:
    *   Các ràng buộc của kiến trúc REST có ý nghĩa lớn đối với REST API nói riêng và bất kỳ loại API nào nói chung. Chúng cung cấp những nền tảng vững chắc cho việc thiết kế API.
    *   **Client/server separation** và **Layered system** liên quan chặt chẽ đến nguyên tắc **quan điểm của người dùng** (consumer's perspective). Nhà cung cấp API không được giao công việc của mình cho người dùng API (ví dụ: yêu cầu người dùng bật/tắt magnetron trong ví dụ Lò vi sóng). Người dùng API chỉ biết về giao diện API, không biết về chi tiết triển khai bên trong (như ví dụ nhà hàng). (Hình 3.36 minh họa mối liên hệ này). Hai ràng buộc này, cùng với việc tập trung vào quan điểm của người dùng, giúp xây dựng API dễ hiểu, dễ sử dụng, tái sử dụng và tiến hóa.
    *   **Uniform interface** liên quan đến việc sử dụng các đường dẫn **duy nhất** và các **phương thức HTTP tiêu chuẩn** để biểu diễn hành động. Điều này tạo ra một giao diện **đồng nhất**, nhất quán trong bản thân API và với các giao diện khác. Ví dụ: `POST /books` và `POST /recipes` đều có cùng ý nghĩa là "tạo cái gì đó". (Hình 3.37 minh họa giao diện đồng nhất).
    *   Các ràng buộc khác như **Statelessness**, **Cacheability**, và **Code on demand** sẽ được khám phá chi tiết hơn trong các chương sau của sách. (Bảng 3.2 liệt kê các chương và phần mô tả từng ràng buộc REST).

Tóm lại, Chương 3 cung cấp nền tảng về cách chuyển đổi các yêu cầu chức năng (từ mục tiêu API) thành một giao diện lập trình cụ thể (với ví dụ REST API). Nó hướng dẫn cách xác định tài nguyên và hành động, ánh xạ chúng sang đường dẫn và phương thức HTTP, và thiết kế chi tiết cấu trúc dữ liệu. Chương cũng nhấn mạnh tầm quan trọng của việc tìm kiếm sự cân bằng trong thiết kế và giới thiệu kiến trúc REST cùng các ràng buộc của nó như những nguyên tắc thiết kế API quan trọng, áp dụng cho cả các kiểu API khác.

Hy vọng phần giải thích chi tiết này bằng tiếng Việt đã đáp ứng được yêu cầu của bạn và cung cấp cái nhìn sâu sắc hơn về nội dung Chương 3.