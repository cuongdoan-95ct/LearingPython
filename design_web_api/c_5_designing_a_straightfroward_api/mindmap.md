# Thiết kế API rõ ràng (Designing Straightforward APIs)
* Phần này tập trung vào cách lựa chọn tên, định dạng dữ liệu và bản thân dữ liệu để nâng cao hoặc làm giảm khả năng sử dụng của API.

### 5.1 Thiết kế các biểu diễn rõ ràng (Designing Straightforward Representations)
#### 5.1.1 Chọn tên rõ ràng (Choosing Crystal-Clear Names)
* Tên khó hiểu làm API khó sử dụng.
* Đặt tên cho đầu vào, đầu ra, tài nguyên, phản hồi, tham số và thuộc tính.
* **Mục tiêu**: Chọn tên người dùng hiểu ngay lập tức.
* **Ví dụ**: `overdraftProtection` thay vì `bankAccountOverdraftProtectionFeatureActive`. `POST /transfers` thay vì `POST /trf`.

#### 5.1.2 Chọn kiểu và định dạng dữ liệu dễ sử dụng (Choosing Easy-to-Use Data Types and Formats)
* Sử dụng kiểu dữ liệu/định dạng không phù hợp cản trở sự hiểu biết.
* **Ví dụ**: Định dạng ngày ISO 8601 (`2018-08-22T18:01:00z`) thay vì dấu thời gian UNIX (`1534960860`).
* Sử dụng giá trị dễ đọc (`checking`) thay vì mã số (`1`).
* Chọn kiểu dữ liệu cơ bản, có thể chuyển đổi: `string`, `number`, `boolean`, `date`, `array`, `object`.

#### 5.1.3 Chọn dữ liệu sẵn sàng sử dụng (Choosing Ready-to-Use Data)
* API nên cung cấp dữ liệu liên quan và hữu ích, giúp người dùng hiểu và tránh xử lý thêm.
* **Ví dụ**:
    - Nếu dùng mã số cho loại tài khoản, cung cấp thêm thuộc tính `typeName` với giá trị chuỗi.
    - Cung cấp `age` (số năm tài khoản đã mở) thay vì chỉ `creationDate`.
    - Sử dụng các giá trị có ý nghĩa, dễ đọc cho URL tài nguyên (ví dụ: số tài khoản `0001234567` thay vì UUID `473e3283-...`).

### 5.2 Thiết kế tương tác rõ ràng (Designing Straightforward Interactions)

### Tương tác trong API bao gồm việc người dùng cung cấp đầu vào và nhận phản hồi.

#### 5.2.1 Yêu cầu đầu vào rõ ràng (Requesting Straightforward Inputs)
* Đầu vào nên đơn giản, dễ hiểu và dễ cung cấp.
* **Ví dụ**:
    - Tên rõ ràng (`source`, `destination`, `amount`).
    - Định dạng ngày ISO 8601.
    - Giá trị chuỗi dễ đọc cho tần suất chuyển tiền.
    - Yêu cầu dữ liệu dễ cung cấp (số tài khoản thay vì UUID).

#### 5.2.2 Xác định tất cả các phản hồi lỗi có thể xảy ra (Identifying All Possible Error Feedbacks)
* Cần xác định tất cả các lỗi có thể xảy ra cho mỗi mục tiêu.
* **Ba loại lỗi chính**:
    1. **Lỗi yêu cầu sai định dạng (Malformed request errors)**: Máy chủ không hiểu yêu cầu (thiếu tham số, sai định dạng).
    2. **Lỗi chức năng (Functional errors)**: Logic nghiệp vụ bị vi phạm (số tiền vượt quá giới hạn).
    3. **Lỗi máy chủ (Server errors)**: Sự cố nội bộ của nhà cung cấp.

#### 5.2.3 Trả về phản hồi lỗi đầy đủ thông tin (Returning Informative Error Feedback)
* Phản hồi lỗi phải giải thích rõ vấn đề và cung cấp thông tin để khắc phục.
* Sử dụng **mã trạng thái HTTP** (ví dụ: 200 OK, 400 Bad Request).
* Kèm theo **phần thân phản hồi (response body)** chi tiết về lỗi.
* **Ví dụ**: Cung cấp mã lỗi lập trình (`MISSING_MANDATORY_PARAMETER`) và chỉ rõ thuộc tính (`source: "amount"`).

#### 5.2.4 Trả về phản hồi lỗi toàn diện (Returning Exhaustive Error Feedback)
* Tránh trả về lỗi từng cái một.
* Nếu nhiều vấn đề, liệt kê tất cả trong một phản hồi lỗi duy nhất.
* Giúp người dùng sửa tất cả lỗi trong một lần.

#### 5.2.5 Trả về phản hồi thành công đầy đủ thông tin (Returning Informative Success Feedback)
* Phản hồi thành công phải cung cấp thông tin hữu ích và giúp người dùng thực hiện các bước tiếp theo.
* **Ví dụ**:
    - Mã trạng thái `201 Created` cho chuyển tiền tức thời.
    - Mã trạng thái `202 Accepted` cho chuyển tiền trì hoãn/định kỳ.
    - Phần thân phản hồi chứa thông tin chi tiết về tài nguyên vừa tạo (ID, trạng thái, loại).

### 5.3 Thiết kế luồng rõ ràng (Designing Straightforward Flows)
* Luồng tương tác là chuỗi các lần gọi API để hoàn thành một mục tiêu phức tạp hơn.

#### 5.3.1 Xây dựng chuỗi mục tiêu rõ ràng (Building a Straightforward Goal Chain)
* Đảm bảo mỗi mục tiêu trong chuỗi đều rõ ràng.
* Các đầu vào phải sẵn có từ người dùng hoặc từ đầu ra của các mục tiêu trước đó.
* **Ví dụ**: Cung cấp `list accounts` và `list beneficiaries` trước khi gọi `transfer money`.

#### 5.3.2 Ngăn chặn lỗi (Preventing Errors)
* Cung cấp dữ liệu giúp người dùng tránh gửi yêu cầu sai.
* **Ví dụ**: Cung cấp `list sources` và `list destinations for source` để trả về các lựa chọn hợp lệ.
* Tương tự nguyên tắc "Code on Demand" trong REST.

#### 5.3.3 Tổng hợp mục tiêu (Aggregating Goals)
* Tổng hợp các mục tiêu nhỏ thành một mục tiêu lớn hơn có thể tối ưu hóa luồng tương tác.
* **Ví dụ**: `list sources and destinations` thay vì hai mục tiêu riêng biệt.
* Cần cân nhắc ý nghĩa chức năng và hiệu suất.

#### 5.3.4 Thiết kế luồng không trạng thái (Designing Stateless Flows)
* Mỗi mục tiêu API nên có thể sử dụng độc lập.
* Tất cả các đầu vào cần thiết phải được khai báo rõ ràng trong yêu cầu.
* Luồng tương tác không nên phụ thuộc vào dữ liệu trạng thái được lưu trữ trên máy chủ.
* Tương ứng với ràng buộc "Statelessness" trong REST, giúp API dễ tái sử dụng và mở rộng.

---

## Tóm tắt chương 5
Thiết kế API rõ ràng tập trung vào việc làm cho các thành phần API (biểu diễn, tương tác, luồng) dễ hiểu, dễ sử dụng và có thể dự đoán được bằng cách:
* Sử dụng tên rõ ràng.
* Định dạng dữ liệu phù hợp.
* Cung cấp thông tin đầy đủ (bao gồm cả lỗi và thành công).
* Xây dựng các luồng tương tác đơn giản, hiệu quả và không trạng thái.