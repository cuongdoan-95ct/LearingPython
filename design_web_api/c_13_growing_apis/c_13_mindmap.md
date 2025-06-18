# Chương 13: Phát triển API (Growing APIs)
## 13.1 Vòng đời API (The API lifecycle)
- Vòng đời API mô tả **cách một API được sinh ra, tồn tại và cuối cùng là ngừng hoạt động**.

- **Các giai đoạn chính**:
    * **Phân tích (Analyze)**: Xác định mục tiêu, nhu cầu, đối tượng người tiêu dùng, lợi ích kinh doanh/kỹ thuật.
    * **Thiết kế (Design)**: Chuyển ý tưởng từ giai đoạn Phân tích thành hợp đồng giao diện lập trình.
    * **Thực hiện (Implement)**: Xây dựng ứng dụng thể hiện hợp đồng.
    * **Xuất bản (Publish)**: Cung cấp API cho người tiêu dùng mục tiêu.
    * **Chạy (Run)**: API hoạt động.
    * **Phát triển (Evolve)**: Cung cấp tính năng mới hoặc cải tiến tính năng hiện có.
    * **Ngừng hoạt động (Retire)**: Khi phiên bản mới thay thế, API không thành công hoặc không còn cần thiết.
- **Quá trình lặp đi lặp lại**: "Phân tích, Thiết kế, Thực hiện" là một quá trình lặp đi lặp lại.
- **Vai trò nhà thiết kế API**: Làm việc chặt chẽ với các bên liên quan, chủ sản phẩm, người viết tài liệu, nhà phát triển/kiểm thử, người tiêu dùng. Tham gia tạo tài liệu, đánh giá, truyền thông.
- **Xu hướng tổ chức**: Tạo ra nhiều API liên tục phát triển, yêu cầu nhà thiết kế API làm việc cùng nhau để xây dựng bộ API nhất quán.

## 13.2 Xây dựng hướng dẫn thiết kế API (Building API design guidelines)
- Việc xác định các hướng dẫn là
**điều bắt buộc để đảm bảo tính nhất quán**
trong và giữa các API của tổ chức/nhóm.

- **Lợi ích**: Tránh lãng phí thời gian, tập trung vào việc cung cấp API dễ hiểu/sử dụng, hỗ trợ nhà thiết kế API mới.
- **Cấu trúc thành ba lớp**:
    * **Hướng dẫn tham chiếu (Reference guidelines)**:
        * Mô tả nguyên tắc cơ bản (phương thức HTTP, mã trạng thái, tiêu đề, định dạng đường dẫn/lỗi, phân trang).
        * Cung cấp định nghĩa rõ ràng về từ vựng (API, tài nguyên, đường dẫn, phiên bản).
    * **Hướng dẫn trường hợp sử dụng (Use case guidelines)**:
        * Giải thích cách áp dụng nguyên tắc cơ bản qua các trường hợp sử dụng.
        * Cung cấp "công thức" hoặc giải pháp sẵn có (cách tạo phần tử, tham số, phản hồi).
    * **Hướng dẫn quy trình thiết kế (Design process guidelines)**:
        * Cung cấp phương pháp, công cụ và quy trình (canvas thiết kế, danh sách kiểm tra, đào tạo).
        * Bao gồm cân nhắc kiến trúc phần mềm và nguyên tắc triển khai.
- **Quá trình xây dựng**:
    * **Bắt đầu nhỏ và chính xác**: Chỉ bao gồm các chủ đề cơ bản, đơn giản, hướng tới sự đầy đủ.
    * **Phát triển, thích nghi và sửa chữa**: Chỉ thêm nội dung đã được chứng minh thực tế. Cho phép phát triển và sửa chữa nếu cần.
    * **Xây dựng tập thể, không giáo điều**: Cần truyền thông, quảng bá, giải thích lý do tồn tại, tránh giáo điều. Xây dựng tập thể giúp chấp nhận và tham gia.

## 13.3 Đánh giá API (Reviewing APIs)
- API cần được **đánh giá** ở các giai đoạn
khác nhau của vòng đời để đảm bảo chúng hoạt động như dự định.

- **Các loại đánh giá**:
    * **Thách thức và phân tích nhu cầu (Challenging and analyzing needs)**:
        * **Quan trọng nhất, thực hiện sớm nhất có thể**.
        * Mục tiêu: Xác định rõ ràng nhu cầu thực sự, tìm giải pháp phù hợp, triển khai được và an toàn.
        * Kỹ thuật: "5 Whys".
        * Xem xét: Ngữ cảnh người tiêu dùng/nhà cung cấp, yếu tố bảo mật.
    * **Kiểm tra thiết kế (Linting the design)**:
        * Kiểm tra lỗi trong thiết kế, tuân thủ hướng dẫn, nhất quán với yếu tố hiện có (API khác, tiêu chuẩn ngành).
        * Kiểm tra bảo mật và tài liệu API.
        * Tự động hóa một phần nhưng vẫn cần con người.
        * **Chỉ xác thực hình thức, không phải nội dung**.
    * **Đánh giá thiết kế từ quan điểm của nhà cung cấp (Reviewing the design from the provider’s perspective)**:
        * Xác nhận thiết kế API **đáp ứng tất cả yêu cầu của nhà cung cấp** (bảo mật, triển khai, khả năng mở rộng).
        * Sử dụng API goals canvas và tài liệu chi tiết.
        * Kiểm tra: Luồng mục tiêu, hành vi, dữ liệu trả về, xử lý lỗi, hiệu suất, khả năng mở rộng.
    * **Đánh giá thiết kế từ quan điểm của người tiêu dùng (Reviewing the design from the consumer’s perspective)**:
        * Mục tiêu: Kiểm tra API có **dễ sử dụng, dễ hiểu và hiệu quả**, không phơi bày chi tiết nội bộ nhà cung cấp.
        * Kiểm tra: Mục tiêu, tham số, phản hồi (đặc biệt lỗi) có ý nghĩa. Tên/mô tả không phải biệt ngữ. Luồng mục tiêu đơn giản/hiệu quả.
        * Xác thực: Ước tính hiệu suất cho cả trường hợp cơ bản và phức tạp.
    * **Xác minh việc triển khai (Verifying the implementation)**:
        * **Kiểm thử bảo mật API là bắt buộc**.
        * Đảm bảo chỉ người tiêu dùng đã đăng ký mới có thể truy cập trong phạm vi được cấp quyền.
        * Cẩn trọng với tài liệu tự động từ mã (có thể có sự không khớp).
        * Chú ý đến định dạng lỗi không đúng.

## 13.4 Giao tiếp và Chia sẻ (Communicating and Sharing)
- Các nhà thiết kế API **không làm việc một mình**;
họ cộng tác với nhiều vai trò khác nhau.

- **Tính nhất quán**: Hướng dẫn công ty cần **dễ dàng tiếp cận và được biết đến bởi tất cả các nhà thiết kế API**.
- **Khả năng tìm kiếm**: Tất cả API và mô hình dữ liệu hiện có nên dễ dàng tìm kiếm và truy cập (Git, wiki, catalog API, cổng thông tin nhà phát triển).
- **Mục tiêu**: **Xây dựng một cộng đồng các nhà thiết kế** để chia sẻ kiến thức và kinh nghiệm.
