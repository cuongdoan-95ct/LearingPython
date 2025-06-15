### 1. Tổng quan về Tài liệu API

Việc thiết kế API không chỉ dừng lại ở việc tạo ra các API dễ sử dụng; nó đòi hỏi phải xem xét toàn bộ ngữ cảnh mà API tồn tại, bao gồm giao diện, cách triển khai và cách sử dụng. **Tài liệu là một khía cạnh cực kỳ quan trọng** mà các nhà thiết kế API phải tham gia. Ngay cả những thiết kế đơn giản nhất cũng cần tài liệu để được hiểu rõ.

Các loại tài liệu API chính bao gồm:
*   **Tài liệu tham khảo (Reference Documentation)**: Đây là loại tài liệu API được biết đến nhiều nhất, liệt kê và mô tả các mục tiêu (goals) có sẵn cũng như các đầu vào và đầu ra của chúng.
*   **Hướng dẫn sử dụng (User Guide/Operating Manual)**: Nếu tài liệu tham khảo chỉ giống như một danh sách thành phần, thì hướng dẫn sử dụng sẽ giải thích **cách thức thực sự sử dụng API** để đạt được các trường hợp sử dụng cụ thể.
*   **Nhật ký thay đổi (Change Log)**: Tài liệu này chỉ ra các tính năng đã thay đổi hoặc được thêm vào, kể cả những thay đổi không gây lỗi (non-breaking changes).
*   **Thông số kỹ thuật triển khai (Implementation Specifications)**: Cung cấp thông tin chi tiết về những gì xảy ra "dưới vỏ bọc" của API, cần thiết cho những người thực hiện triển khai.

Mức độ tham gia của nhà thiết kế API vào việc tạo tài liệu phụ thuộc vào quy mô công ty/đội ngũ và loại API (nội bộ, đối tác, công khai). Việc tài liệu hóa API một cách cặn kẽ cũng là một cách để **kiểm tra thiết kế**: nếu không thể tài liệu hóa cách sử dụng hoặc triển khai API, đó có thể là dấu hiệu của một thiết kế không phù hợp.

### 2. Tạo Tài liệu Tham khảo (Reference Documentation)

Tài liệu tham khảo **liệt kê và mô tả từng thành phần có sẵn của giao diện API**. Đối với API, các thành phần này bao gồm các mục tiêu (goals), đầu vào, đầu ra (cho cả trường hợp thành công và lỗi), một mô tả API đơn giản và thông tin bảo mật.
*   Nên sử dụng **các định dạng mô tả API chuẩn như OpenAPI Specification (OAS)** để tạo tài liệu tham khảo, vì chúng giúp lưu trữ, quản lý phiên bản và tạo ra các bản trình bày thân thiện với con người một cách dễ dàng.
*   Các công cụ như **ReDoc** có thể tự động tạo tài liệu từ file OpenAPI.

#### 2.1. Tài liệu hóa các Mô hình Dữ liệu (Documenting Data Models)
Tài liệu tham khảo cho mô hình dữ liệu phải hiển thị các thuộc tính của dữ liệu (ví dụ: `source`, `destination`, `amount`, `date`, `occurrences`, `frequency`), kiểu dữ liệu của chúng (ví dụ: `string`, `number`, `integer`), và liệu chúng có bắt buộc hay không (`mandatory`/`required`).
*   Nó cần cung cấp **các mô tả chức năng và kỹ thuật hữu ích cùng với các ví dụ**.
    *   Ví dụ: các thuộc tính `source` và `destination` phải dài 15 ký tự và chỉ chứa chữ số (theo biểu thức chính quy `/^\d{15}$/`).
    *   `amount` là một số dương; `occurrences` phải nằm trong khoảng từ 2 đến 100.
    *   `frequency` có thể là "WEEKLY", "MONTHLY", "QUARTERLY", "YEARLY".
*   Các công cụ tài liệu có thể **tự động suy đoán các giá trị ví dụ** dựa trên các mô tả (ví dụ: ngày hôm nay cho thuộc tính `date`, một giá trị ngẫu nhiên từ enum cho `frequency`).
*   Việc thêm **các mô tả chi tiết, phù hợp** (bao gồm cả mô tả máy đọc được như định dạng/giá trị có thể có, và mô tả thân thiện với con người giải thích vai trò/mối quan hệ/cách sử dụng) và **các ví dụ** sẽ làm cho tài liệu trở nên hữu ích cho cả người dùng API và nhà cung cấp.
*   Nên tránh các mô tả "hiển nhiên" (ví dụ: "amount: số tiền chuyển khoản").
*   Nếu công cụ hỗ trợ, nên sử dụng các định dạng mô tả thân thiện với con người như Markdown.

#### 2.2. Tài liệu hóa các Mục tiêu/Chức năng (Documenting Goals)
Tài liệu tham khảo của một mục tiêu mô tả **mục đích của nó, những gì cần thiết để sử dụng, phản hồi thành công/thất bại, và việc nó thuộc nhóm nào**.
*   Ví dụ: `POST /transfers` để tạo các giao dịch chuyển tiền (ngay lập tức, trì hoãn, định kỳ).
*   Tài liệu cần giải thích các khía cạnh chức năng (ví dụ: từng loại chuyển khoản, các thuộc tính chung/riêng).
*   **Cung cấp nhiều ví dụ yêu cầu (request samples)** cho các trường hợp sử dụng khác nhau (ví dụ: các tab "Immediate transfer", "Delayed transfer", "Recurring transfer") là điều bắt buộc để tăng tính thân thiện với người dùng.
*   Thông tin chi tiết về **các phản hồi có thể có**, bao gồm nhiều ví dụ cho thành công và mô tả toàn diện cho lỗi, là rất quan trọng.
*   **Việc tổ chức các mục tiêu** (ví dụ: sử dụng các danh mục như "Transfers" trong menu) là quan trọng và có thể được quản lý bằng OpenAPI tags.

#### 2.3. Tài liệu hóa Bảo mật (Documenting Security)
Tài liệu tham khảo phải chứa **thông tin về bảo mật**.
*   Ví dụ: API ngân hàng được bảo mật bằng **luồng implicit OAuth 2.0**, liệt kê các phạm vi (scopes) có sẵn (`transfer:create`, `transfer:read`, v.v.).
*   Nó cần nêu rõ **những phạm vi nào là cần thiết** cho các mục tiêu cụ thể (ví dụ: `transfer:create` hoặc `transfer:admin` cho chức năng `transfer money`).
*   Thông tin này thường được định nghĩa trong giai đoạn thiết kế API bằng cách sử dụng các sơ đồ bảo mật của OpenAPI, do đó yêu cầu ít nỗ lực bổ sung cho tài liệu cơ bản.

#### 2.4. Cung cấp Tổng quan về API (Providing an Overview of the API)
Tài liệu tham khảo cấp độ API lấy thông tin từ phần `info` trong file OpenAPI Specification.
*   Nó bao gồm **tên API (title), phiên bản, và một mô tả**.
*   Nên cung cấp **thông tin liên hệ** (tên đội ngũ, email, URL trang web dành cho nhà phát triển).
*   Một mô tả ngắn gọn về API là bắt buộc; nó giúp người dùng hiểu mục đích của API nếu tên gọi không đủ rõ ràng. Thông tin liên hệ là tùy chọn nhưng rất được khuyến khích để hỗ trợ người dùng.

#### 2.5. Tạo Tài liệu từ việc Triển khai: Ưu và Nhược điểm (Generating Documentation from the Implementation: Pros and Cons)
Tài liệu có thể được tạo ra từ **mã nguồn hoặc mã nguồn kèm theo các chú thích** (ví dụ: mục đích ban đầu của framework Swagger).
*   **Ưu điểm**: **Giữ cho phần triển khai và tài liệu được đồng bộ hóa**.
*   **Nhược điểm**:
    *   Việc tạo tài liệu chỉ dựa vào mã nguồn thường dẫn đến **tài liệu không đầy đủ**.
    *   Các framework chú thích có thể **thiếu tính linh hoạt** (ví dụ: không thể cung cấp các ví dụ phù hợp với ngữ cảnh cho các cấu trúc dữ liệu chung).
    *   Bao gồm tài liệu trong mã nguồn có nghĩa là **phải sửa đổi mã nguồn để sửa tài liệu**, điều này có thể gây ra vấn đề tùy thuộc vào vai trò của đội ngũ và độ tin cậy trong các triển khai tự động.
    *   Đòi hỏi mã nguồn phải được viết sớm, có khả năng **làm lộ góc nhìn của nhà cung cấp**.
*   Giữ tài liệu tách rời khỏi mã nguồn (bên ngoài) là một lựa chọn khác, nhưng nhược điểm chính là việc đồng bộ hóa.
*   Không có chiến lược nào là "tốt" hay "xấu" hoàn toàn; nên chọn cách phù hợp với tổ chức của bạn.

### 3. Tạo Hướng dẫn Sử dụng (Creating a User Guide)
Hướng dẫn sử dụng giải thích **cách thức thực sự sử dụng API**. Nó bao gồm cách sử dụng API tổng thể, các nguyên tắc của nó, và cách truy cập (đăng ký, lấy token).

#### 3.1. Tài liệu hóa các Trường hợp Sử dụng (Documenting Use Cases)
Hướng dẫn sử dụng mô tả **cách các mục tiêu API khác nhau có thể được kết hợp để đạt được một mục đích nào đó** (ví dụ: "Chuyển tiền vào tài khoản").
*   **Canvas mục tiêu API (API goals canvas)** từ giai đoạn thiết kế có thể được tái sử dụng trực tiếp cho các API nội bộ hoặc làm đầu vào thô cho các API đối tác/công khai.
*   Các trường hợp sử dụng có thể được mô tả bằng văn bản định dạng, biểu đồ hoặc hình ảnh.
*   Phần `info.description` của OpenAPI Specification có thể chứa văn bản định dạng Markdown và hình ảnh, mà các công cụ như ReDoc có thể hiển thị như các mục và tiểu mục trong menu.
*   **Biểu đồ** rất hữu ích ("một bức tranh đáng giá ngàn lời nói").

#### 3.2. Tài liệu hóa Bảo mật (Documenting Security)
Hướng dẫn sử dụng phải bao gồm lời khuyên về **cách đăng ký làm nhà phát triển, đăng ký ứng dụng tiêu thụ, và lấy token** bằng các luồng OAuth hoặc các hệ thống bảo mật khác.

#### 3.3. Cung cấp Tổng quan về các Hành vi và Nguyên tắc chung (Providing an Overview of Common Behaviors and Principles)
Hướng dẫn sử dụng API cũng có thể chứa thông tin về **tất cả các hành vi và nguyên tắc chung của API**:
*   Cách xử lý lỗi.
*   Các định dạng dữ liệu và ngôn ngữ được hỗ trợ.
*   Cách xử lý phân trang.
*   Nói chung, **mọi thứ chung cho các mục tiêu API và đáng được đề cập** để tạo điều kiện sử dụng API.

#### 3.4. Suy nghĩ vượt ra ngoài Tài liệu Tĩnh (Thinking Beyond Static Documentation)
Tài liệu tĩnh không phải là lựa chọn duy nhất. Các API công khai được đánh giá cao thường có **cổng thông tin nhà phát triển động (dynamic developer portals)** với tài liệu chất lượng cao.
*   Các tính năng như nút "Try It!" cho phép người dùng gọi API trực tiếp từ tài liệu, với cổng thông tin nhà phát triển xử lý bảo mật tự động.
*   Một số hướng dẫn sử dụng cho phép **kiểm thử các trường hợp sử dụng theo từng bước** ngay trong cổng thông tin nhà phát triển.

### 4. Cung cấp Thông tin đầy đủ cho Người Triển khai (Providing Adequate Information to Implementers)
Những người triển khai cần một **mô tả chi tiết về hợp đồng giao diện** và **những gì xảy ra "dưới vỏ bọc"** của API.
*   Các vấn đề có thể phát sinh do thông tin không đầy đủ về việc **ánh xạ hợp đồng API với các hệ thống nền tảng** và **các kiểm soát bảo mật dự kiến**.
*   Các file mô tả API (như OAS) có thể được tăng cường với:
    *   **Mô tả chi tiết thuộc tính**: ví dụ: chỉ định số chữ số thập phân cho `amount` theo tiêu chuẩn ISO 4217.
    *   **Liên kết đến tài liệu bên ngoài** (ví dụ: tiêu chuẩn ISO 4217) bằng cách sử dụng `externalDocs`.
    *   **Các extension tùy chỉnh (vendor extensions)** bắt đầu bằng `x-` (ví dụ: `x-implementation`) để cung cấp chi tiết dành riêng cho nhà cung cấp (ví dụ: nguồn dữ liệu, kiểm soát bảo mật) mà người dùng API không nhìn thấy.
*   Những người triển khai cần **tài liệu hướng dẫn về phía nhà cung cấp** về ánh xạ dữ liệu, ánh xạ lỗi (chuyển đổi lỗi nội bộ thành lỗi hiển thị cho người dùng), dữ liệu và kiểm soát bảo mật, và các hành vi dự kiến dựa trên các quy tắc nội bộ.
*   Thông tin này có thể nằm trong file mô tả API hoặc được cung cấp riêng biệt.
*   Hướng dẫn chung và đào tạo cho nhà phát triển cũng rất quan trọng.

### 5. Tài liệu hóa các Bản cập nhật và việc Ngừng sử dụng (Documenting Evolutions and Retirement)
Các thay đổi (gây lỗi hoặc không gây lỗi) chắc chắn sẽ xảy ra và **phải được tài liệu hóa**.
*   **Nhật ký thay đổi** hữu ích cho người dùng (các tính năng mới, các phần bị lỗi thời, hoặc đã ngừng sử dụng) và các đội ngũ dự án (tổng quan về các thay đổi sắp tới).
*   Các nhà thiết kế API, với tư cách là người tạo ra các thay đổi, là những người phù hợp nhất để liệt kê chúng.
*   Một nhật ký thay đổi nên nêu rõ **các phần tử đã được thêm, sửa đổi, lỗi thời (deprecated), hoặc ngừng sử dụng (retired)** (thuộc tính mô hình dữ liệu, tham số, phản hồi, phạm vi bảo mật).
*   Có thể được bao gồm trong phần `info.description` của file OpenAPI bằng cách sử dụng Markdown.
*   OpenAPI hỗ trợ **chỉ định các phần tử lỗi thời** bằng cờ `deprecated: true` trên mục tiêu hoặc tham số.
*   Header **`Sunset` (RFC 8594)** có thể được sử dụng trong các phản hồi API để thông báo khi một tài nguyên sẽ trở nên không phản hồi, giúp người dùng có thời gian để cập nhật.
