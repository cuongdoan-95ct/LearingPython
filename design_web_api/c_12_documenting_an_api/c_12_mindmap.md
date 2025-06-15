# Chương 12: Tài liệu hóa API của bạn (Documenting Your API)

## Tài liệu là khía cạnh cực kỳ quan trọng mà các nhà thiết kế API phải tham gia.

---

### 1. Tổng quan về Tài liệu API

#### Tài liệu là một khía cạnh cực kỳ quan trọng mà các nhà thiết kế API phải tham gia.

* **Tầm quan trọng**: Ngay cả những thiết kế đơn giản nhất cũng cần tài liệu để được hiểu rõ.
* **Các loại tài liệu API chính**:
    * **Tài liệu tham khảo (Reference Documentation)**: Liệt kê và mô tả mục tiêu, đầu vào, đầu ra.
    * **Hướng dẫn sử dụng (User Guide/Operating Manual)**: Giải thích cách thức thực sự sử dụng API để đạt được các trường hợp sử dụng cụ thể.
    * **Nhật ký thay đổi (Change Log)**: Chỉ ra các tính năng đã thay đổi/thêm vào.
    * **Thông số kỹ thuật triển khai (Implementation Specifications)**: Thông tin chi tiết về những gì xảy ra "dưới vỏ bọc".
* **Mức độ tham gia**: Phụ thuộc vào quy mô công ty/đội ngũ và loại API.
* **Kiểm tra thiết kế**: Nếu không thể tài liệu hóa API, đó có thể là dấu hiệu của một thiết kế không phù hợp.

---

### 2. Tạo Tài liệu Tham khảo (Reference Documentation)

#### Tài liệu tham khảo liệt kê và mô tả từng thành phần có sẵn của giao diện API.

* **Nên sử dụng**: **OpenAPI Specification (OAS)** để dễ dàng lưu trữ, quản lý phiên bản và tạo ra các bản trình bày thân thiện với con người.
* **Công cụ**: **ReDoc** tự động tạo tài liệu từ file OpenAPI.

#### 2.1. Tài liệu hóa các Mô hình Dữ liệu (Documenting Data Models)
    * **Nội dung**: Hiển thị thuộc tính, kiểu dữ liệu, bắt buộc/tùy chọn.
    * **Yêu cầu**: Cung cấp **mô tả chức năng/kỹ thuật hữu ích cùng với các ví dụ**.
        * **Ví dụ**: `source`/`destination` dài 15 ký tự (`/^\d{15}$/`), `amount` là số dương, `frequency` là "WEEKLY", "MONTHLY", v.v.
    * **Công cụ**: Có thể tự động suy đoán giá trị ví dụ.
    * **Mô tả**: Tránh "hiển nhiên". Sử dụng Markdown nếu công cụ hỗ trợ.

#### 2.2. Tài liệu hóa các Mục tiêu/Chức năng (Documenting Goals)
    * **Nội dung**: Mục đích, đầu vào cần thiết, phản hồi thành công/thất bại, nhóm thuộc về.
    * **Giải thích**: Các khía cạnh chức năng (ví dụ: loại chuyển khoản, thuộc tính chung/riêng).
    * **Ví dụ yêu cầu**: Cung cấp **nhiều ví dụ (request samples)** cho các trường hợp sử dụng khác nhau (ví dụ: "Immediate transfer", "Delayed transfer", "Recurring transfer").
    * **Phản hồi**: Thông tin chi tiết về **các phản hồi có thể có** (thành công/lỗi), nhiều ví dụ.
    * **Tổ chức**: Sắp xếp các mục tiêu (ví dụ: dùng OpenAPI tags).

#### 2.3. Tài liệu hóa Bảo mật (Documenting Security)
    * **Nội dung**: Thông tin về bảo mật (ví dụ: luồng implicit OAuth 2.0, các phạm vi (scopes) có sẵn).
    * **Yêu cầu**: Nêu rõ **những phạm vi nào là cần thiết** cho các mục tiêu cụ thể.
    * **Thường được định nghĩa**: Trong giai đoạn thiết kế API bằng sơ đồ bảo mật của OpenAPI.

#### 2.4. Cung cấp Tổng quan về API (Providing an Overview of the API)
    * **Nội dung**: Tên API (`title`), phiên bản, mô tả, thông tin liên hệ (tên đội ngũ, email, URL trang web dành cho nhà phát triển).
    * **Mô tả ngắn gọn**: Giúp người dùng hiểu mục đích API.

#### 2.5. Tạo Tài liệu từ việc Triển khai: Ưu và Nhược điểm (Generating Documentation from the Implementation: Pros and Cons)
    * **Ưu điểm**: **Giữ phần triển khai và tài liệu đồng bộ hóa**.
    * **Nhược điểm**:
        * Thường dẫn đến **tài liệu không đầy đủ**.
        * **Thiếu tính linh hoạt** (ví dụ: ví dụ ngữ cảnh cho cấu trúc chung).
        * Phải **sửa đổi mã nguồn để sửa tài liệu**.
        * Yêu cầu mã nguồn được viết sớm, có thể **làm lộ góc nhìn của nhà cung cấp**.
    * **Lựa chọn**: Phù hợp với tổ chức của bạn.

---

### 3. Tạo Hướng dẫn Sử dụng (Creating a User Guide)

#### Hướng dẫn sử dụng giải thích cách thức thực sự sử dụng API.

#### 3.1. Tài liệu hóa các Trường hợp Sử dụng (Documenting Use Cases)
    * **Nội dung**: Cách các mục tiêu API kết hợp để đạt được một mục đích (ví dụ: "Chuyển tiền vào tài khoản").
    * **Nguồn**: **Canvas mục tiêu API** có thể tái sử dụng.
    * **Định dạng**: Văn bản định dạng, biểu đồ hoặc hình ảnh (sử dụng Markdown, hiển thị bởi ReDoc).
    * **Biểu đồ**: Rất hữu ích.

#### 3.2. Tài liệu hóa Bảo mật (Documenting Security)
    * **Nội dung**: Lời khuyên về cách đăng ký làm nhà phát triển, đăng ký ứng dụng, và lấy token.

#### 3.3. Cung cấp Tổng quan về các Hành vi và Nguyên tắc chung (Providing an Overview of Common Behaviors and Principles)
    * **Nội dung**: Cách xử lý lỗi, định dạng dữ liệu/ngôn ngữ hỗ trợ, cách xử lý phân trang, mọi thứ chung cho các mục tiêu API.

#### 3.4. Suy nghĩ vượt ra ngoài Tài liệu Tĩnh (Thinking Beyond Static Documentation)
    * **Cổng thông tin nhà phát triển động (dynamic developer portals)**: Có nút "Try It!", kiểm thử trường hợp sử dụng theo từng bước.

---

### 4. Cung cấp Thông tin đầy đủ cho Người Triển khai (Providing Adequate Information to Implementers)

#### Những người triển khai cần mô tả chi tiết về hợp đồng giao diện và những gì xảy ra "dưới vỏ bọc" của API.

* **Vấn đề**: Thông tin không đầy đủ về ánh xạ hợp đồng API với hệ thống nền tảng, kiểm soát bảo mật.
* **Tăng cường file mô tả API (OAS)**:
    * **Mô tả chi tiết thuộc tính**: Số chữ số thập phân cho `amount` theo ISO 4217.
    * **Liên kết đến tài liệu bên ngoài**: Dùng `externalDocs`.
    * **Các extension tùy chỉnh (vendor extensions)**: Dùng `x-` (ví dụ: `x-implementation`) cho chi tiết dành riêng cho nhà cung cấp (nguồn dữ liệu, kiểm soát bảo mật).
* **Tài liệu hướng dẫn về phía nhà cung cấp**: Ánh xạ dữ liệu, ánh xạ lỗi, dữ liệu và kiểm soát bảo mật, hành vi dự kiến.
* **Quan trọng**: Hướng dẫn chung và đào tạo cho nhà phát triển.

---

### 5. Tài liệu hóa các Bản cập nhật và việc Ngừng sử dụng (Documenting Evolutions and Retirement)

#### Các thay đổi (gây lỗi hoặc không gây lỗi) chắc chắn sẽ xảy ra và phải được tài liệu hóa.

* **Nhật ký thay đổi**: Hữu ích cho người dùng và các đội ngũ dự án.
* **Người tạo thay đổi**: Là người phù hợp nhất để liệt kê chúng.
* **Nội dung nhật ký thay đổi**: Nêu rõ các phần tử đã được thêm, sửa đổi, lỗi thời (deprecated), hoặc ngừng sử dụng (retired).
* **Vị trí**: Có thể trong `info.description` của file OpenAPI bằng Markdown.
* **OpenAPI**: Hỗ trợ chỉ định các phần tử lỗi thời bằng cờ `deprecated: true`.
* **Header `Sunset` (RFC 8594)**: Thông báo khi tài nguyên sẽ trở nên không phản hồi, giúp người dùng có thời gian cập nhật.
