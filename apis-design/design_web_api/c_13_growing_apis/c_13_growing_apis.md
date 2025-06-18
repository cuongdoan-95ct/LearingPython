## chương 13: Growing APIs (Phát triển API)
### 13.1 Vòng đời API (The API lifecycle)

Vòng đời API mô tả **cách một API được sinh ra, tồn tại và cuối cùng là ngừng hoạt động**. Nó chạy song song với các vòng đời khác trong hệ thống.

Các **giai đoạn chính** trong vòng đời API bao gồm:
*   **Phân tích (Analyze)**: Trong giai đoạn này, một công ty/tổ chức/nhóm/cá nhân xem xét việc cung cấp một API có thể mang lại lợi ích kinh doanh hoặc kỹ thuật. Các chủ đề được khám phá bao gồm mục tiêu của API, nhu cầu mà nó cần đáp ứng, đối tượng người tiêu dùng mà nó nhắm đến, ai cần nó, và những lợi ích mà nó mang lại.
*   **Thiết kế (Design)**: Ý tưởng từ giai đoạn Phân tích được nghiên cứu sâu hơn và chuyển thành một hợp đồng giao diện lập trình.
*   **Thực hiện (Implement)**: Một ứng dụng thể hiện hợp đồng này được xây dựng.
*   **Xuất bản (Publish)**: API được cung cấp cho người tiêu dùng mục tiêu.
*   **Chạy (Run)**: API hoạt động.
*   **Phát triển (Evolve)**: API có thể phát triển để cung cấp các tính năng mới hoặc cải tiến các tính năng hiện có.
*   **Ngừng hoạt động (Retire)**: API có thể ngừng hoạt động khi một phiên bản mới giới thiệu các thay đổi phá vỡ (breaking changes) và thay thế phiên bản cũ, hoặc nếu API không thành công hay không còn cần thiết.

Quá trình "Phân tích, Thiết kế, Thực hiện" là một **quá trình lặp đi lặp lại**; bạn có thể cần quay lại các giai đoạn trước tùy theo những khám phá mới, câu hỏi mới, hoặc đơn giản là thay đổi ý tưởng về một giải pháp thiết kế.

Các nhà thiết kế API cần **làm việc chặt chẽ với các bên liên quan**, chủ sản phẩm, người viết tài liệu kỹ thuật, nhà phát triển hoặc kiểm thử, và người tiêu dùng (trực tiếp hoặc thông qua đội ngũ quan hệ nhà phát triển API). Họ cũng tham gia vào việc tạo tài liệu, thực hiện các đánh giá (từ phía nhà cung cấp và người tiêu dùng), và truyền thông về API.

Các tổ chức hiếm khi chỉ xây dựng một API duy nhất, không thay đổi; thay vào đó, họ thường **tạo ra nhiều API liên tục phát triển**. Điều này yêu cầu các nhà thiết kế API phải làm việc cùng nhau để xây dựng một bộ API nhất quán cho tổ chức.

### 13.2 Xây dựng hướng dẫn thiết kế API (Building API design guidelines)

Việc xác định các hướng dẫn, một tập hợp các quy tắc được sử dụng bởi tất cả các nhà thiết kế, là **điều bắt buộc để đảm bảo tính nhất quán** trong và giữa các API của tổ chức/nhóm. Nó giúp tránh lãng phí thời gian vào các cuộc tranh luận vô tận và tập trung vào việc cung cấp các API dễ hiểu, dễ sử dụng, đáp ứng nhu cầu của người tiêu dùng. Hướng dẫn cũng là một công cụ tuyệt vời để hỗ trợ các nhà thiết kế API mới bắt đầu.

Các hướng dẫn thiết kế API có thể được cấu trúc thành **ba lớp khác nhau**:
*   **Hướng dẫn tham chiếu (Reference guidelines)**: Tập trung vào mô tả các nguyên tắc cơ bản của thiết kế API. Chúng có thể bao gồm các phương thức HTTP, mã trạng thái, tiêu đề có thể sử dụng và khi nào sử dụng; định dạng đường dẫn tài nguyên; định dạng dữ liệu lỗi; và cách xử lý phân trang. Các hướng dẫn này cũng nên cung cấp định nghĩa rõ ràng và dùng chung về từ vựng được sử dụng khi thiết kế API (ví dụ: API là gì, tài nguyên hay tập hợp là gì, đường dẫn là gì, phiên bản là gì).
*   **Hướng dẫn trường hợp sử dụng (Use case guidelines)**: Giải thích cách áp dụng các nguyên tắc cơ bản thông qua các trường hợp sử dụng khác nhau. Chúng cung cấp "công thức" hoặc giải pháp sẵn có (ví dụ: cách tạo một phần tử, các tham số dự kiến, phản hồi). Loại hướng dẫn này đặc biệt quan trọng đối với người mới bắt đầu và cũng hữu ích cho các nhà thiết kế API có kinh nghiệm.
*   **Hướng dẫn quy trình thiết kế (Design process guidelines)**: Cung cấp các phương pháp, công cụ và quy trình để thiết kế API (ví dụ: canvas thiết kế, danh sách kiểm tra, các buổi đào tạo do các nhà thiết kế API có kinh nghiệm cung cấp). Chúng có thể bao gồm các cân nhắc rộng hơn ngoài hợp đồng giao diện, như chi tiết về kiến trúc phần mềm và nguyên tắc triển khai.

Quá trình xây dựng hướng dẫn cần:
*   **Bắt đầu nhỏ và chính xác**: Chỉ bao gồm các chủ đề cơ bản, cần thiết một cách đơn giản và trực tiếp, hướng tới sự đầy đủ và chính xác.
*   **Phát triển, thích nghi và sửa chữa**: Chỉ thêm nội dung đã được chứng minh thực tế. Hướng dẫn cần được phép phát triển để bao gồm các nội dung mới liên quan đến các tình huống ban đầu không được đề cập. Chúng cũng có thể cần được sửa chữa nếu một số quy tắc, trường hợp sử dụng hoặc nội dung khác được tiết lộ là gây khó khăn khi áp dụng hoặc bất tiện về lâu dài.
*   **Xây dựng tập thể, không giáo điều**: Cần truyền thông và quảng bá các hướng dẫn, giải thích lý do tồn tại và tránh giáo điều. Việc xây dựng tập thể sẽ giúp các nhà thiết kế API chấp nhận và tham gia vào quá trình này.

### 13.3 Đánh giá API (Reviewing APIs)

API cần được **đánh giá (xác thực, phân tích, xem xét kỹ lưỡng, kiểm tra, v.v.)** ở các giai đoạn khác nhau của vòng đời để đảm bảo chúng hoạt động như dự định. Các nhà thiết kế API thường tham gia tích cực hoặc ít nhất có tiếng nói trong tất cả các cuộc đánh giá này.

Các loại đánh giá bao gồm:
*   **Thách thức và phân tích nhu cầu (Challenging and analyzing needs)**:
    *   Đây là bước **quan trọng nhất và cần được thực hiện sớm nhất có thể**.
    *   Mục tiêu là xác định rõ ràng nhu cầu thực sự của người dùng và tìm ra các giải pháp phù hợp, có thể triển khai và an toàn, thay vì chỉ chấp nhận các yêu cầu ban đầu.
    *   Kỹ thuật như **"5 Whys"** (hỏi "tại sao?" năm lần) có thể giúp đi sâu vào phân tích và tìm ra gốc rễ của nhu cầu.
    *   Cần xem xét ngữ cảnh của người tiêu dùng và nhà cung cấp, cũng như các yếu tố bảo mật.
*   **Kiểm tra thiết kế (Linting the design)**:
    *   Bao gồm việc **kiểm tra lỗi trong thiết kế**, xác minh rằng nó tuân thủ các hướng dẫn thiết kế và nhất quán với các yếu tố hiện có (như các API khác hoặc tiêu chuẩn ngành). Đồng thời, kiểm tra bảo mật và tài liệu của API.
    *   Nó kiểm tra các khía cạnh của hợp đồng giao diện, bao gồm tài liệu, luồng dữ liệu (đảm bảo người tiêu dùng có thể cung cấp các tham số cần thiết), và tính nhất quán với các yếu tố đã tồn tại (tên, kiểu dữ liệu, mô hình dữ liệu, hành vi, và các tiêu chuẩn như ISO 8601, ISO 4217, E.164).
    *   Quá trình này có thể được tự động hóa một phần nhưng vẫn cần sự tham gia của con người.
    *   Linting **chỉ xác thực hình thức, không phải nội dung**, nghĩa là nó chỉ đảm bảo thiết kế tuân thủ các quy tắc, chứ không đảm bảo API đáp ứng tất cả các yêu cầu hoặc có tính khả dụng cao.
*   **Đánh giá thiết kế từ quan điểm của nhà cung cấp (Reviewing the design from the provider’s perspective)**:
    *   Xác nhận rằng thiết kế API **thực sự đáp ứng tất cả các yêu cầu của nhà cung cấp**, bao gồm tính bảo mật, khả năng triển khai và khả năng mở rộng.
    *   Sử dụng API goals canvas và tài liệu chi tiết để hỗ trợ quá trình đánh giá.
    *   Kiểm tra luồng mục tiêu, hành vi, dữ liệu trả về, xử lý lỗi, hiệu suất, và khả năng mở rộng của từng yếu tố.
*   **Đánh giá thiết kế từ quan điểm của người tiêu dùng (Reviewing the design from the consumer’s perspective)**:
    *   Mục tiêu là kiểm tra xem API có **dễ sử dụng, dễ hiểu và hiệu quả** không, và liệu nó có tránh việc phơi bày các chi tiết nội bộ không cần thiết của nhà cung cấp không.
    *   Kiểm tra các mục tiêu, tham số và phản hồi (đặc biệt là lỗi) có ý nghĩa với người tiêu dùng không. Đảm bảo tên và mô tả không phải là biệt ngữ của nhà cung cấp. Kiểm tra các luồng mục tiêu có đơn giản và hiệu quả không.
    *   Xác thực ước tính hiệu suất cho cả trường hợp sử dụng cơ bản và phức tạp.
*   **Xác minh việc triển khai (Verifying the implementation)**:
    *   Kiểm thử bảo mật API là **bắt buộc** để đảm bảo quyền truy cập và dữ liệu nhạy cảm được xử lý đúng cách.
    *   Đảm bảo chỉ người tiêu dùng đã đăng ký mới có thể truy cập API và chỉ được thực hiện trong phạm vi được cấp quyền.
    *   Cẩn trọng khi sử dụng tài liệu được tạo tự động từ mã để xác thực triển khai, vì có thể có sự không khớp.
    *   Đặc biệt chú ý đến các định dạng lỗi không đúng.

### 13.4 Giao tiếp và Chia sẻ (Communicating and Sharing)

Các nhà thiết kế API **không làm việc một mình**; họ cộng tác với nhiều vai trò khác nhau (các bên liên quan, người tiêu dùng, nhà phát triển, đội ngũ bảo mật, tài liệu, và các nhà thiết kế API khác).

Để đảm bảo tính nhất quán, các hướng dẫn của công ty (mà bạn đóng góp) cần **dễ dàng tiếp cận và được biết đến bởi tất cả các nhà thiết kế API**. Tất cả các API và mô hình dữ liệu hiện có nên dễ dàng tìm kiếm và truy cập (ví dụ qua hệ thống kiểm soát nguồn như Git, wiki, catalog API tùy chỉnh, hoặc cổng thông tin nhà phát triển sẵn có).

Việc sử dụng các định dạng mô tả API tiêu chuẩn giúp dễ dàng chia sẻ tài liệu thiết kế. Mục tiêu là **xây dựng một cộng đồng các nhà thiết kế** để chia sẻ kiến thức và kinh nghiệm.
