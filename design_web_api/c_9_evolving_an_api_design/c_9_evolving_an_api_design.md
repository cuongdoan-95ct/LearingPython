### 1. Thiết kế các tiến hóa API (Designing API evolutions)

Thiết kế các tiến hóa API đòi hỏi sự cẩn trọng đặc biệt để tránh những thay đổi gây lỗi (breaking changes). **Một thay đổi gây lỗi là một thay đổi sẽ gây ra vấn đề cho người tiêu dùng nếu họ không cập nhật mã của mình**.

*   **Tránh các thay đổi gây lỗi trong dữ liệu đầu ra (Avoiding breaking changes in output data)**:
    *   **Đổi tên thuộc tính** (Renaming a property): Ví dụ, nếu thuộc tính `amt` (số tiền giao dịch tính bằng xu) được đổi tên thành `amount` (số tiền tính bằng đô la), người dùng API cũ sẽ hiểu sai giá trị nhận được, dẫn đến lỗi hoặc hiểu sai dữ liệu nghiêm trọng.
    *   **Di chuyển thuộc tính** (Moving a property): Nếu thuộc tính được di chuyển trong cấu trúc dữ liệu, nó có thể gây ra lỗi phân tích cú pháp hoặc khiến dữ liệu bị thiếu trong giao diện người dùng.
    *   **Xóa thuộc tính bắt buộc** (Removing a mandatory property): Điều này chắc chắn sẽ làm hỏng ứng dụng của người dùng cũ vì họ mong đợi thuộc tính đó luôn có mặt.
    *   **Thay đổi ý nghĩa của thuộc tính** (Changing a property's meaning): Ví dụ, thay đổi ý nghĩa của một trường từ mã số sang một loại định danh khác sẽ làm hỏng logic ứng dụng của người dùng.
    *   **Thay đổi loại hoặc định dạng thuộc tính** (Changing a property's type or format): Chuyển đổi một số (number) thành một đối tượng (object) như trường `balance` của tài khoản ngân hàng sẽ khiến ứng dụng của người dùng cũ không thể phân tích cú pháp. Tương tự, thay đổi một trường ngày tháng từ dấu thời gian Unix (Unix timestamp) sang định dạng ngày tháng tiêu chuẩn (standard date string) cũng gây lỗi.
    *   **Thay đổi đặc điểm thuộc tính** (Modifying (decreasing) characteristics): Ví dụ, giảm độ dài tối đa của một chuỗi hoặc số lượng mục tối đa trong một mảng có thể gây lỗi nếu người dùng đã quen với việc gửi dữ liệu lớn hơn.
    *   **Xóa giá trị khỏi enum** (Removing values from enums): Nếu một giá trị hợp lệ trước đây bị xóa khỏi danh sách các giá trị enum, người dùng cũ sẽ gặp lỗi khi cố gắng gửi hoặc nhận giá trị đó.
    *   **Thêm giá trị mới vào enum** (Adding new values to enum): Mặc dù việc này có vẻ tương thích ngược, nhưng ứng dụng người dùng có thể không giải thích được các giá trị mới hoặc bị lỗi khi gặp chúng.
    *   **Cách an toàn để thay đổi dữ liệu đầu ra**: Cách an toàn nhất là **chỉ thêm các thuộc tính mới**. Nếu cần sửa lỗi thiết kế cũ, bạn có thể **thêm một thuộc tính mới để chỉ ra một tình trạng đặc biệt** (ví dụ: `communityCategorization` boolean) hoặc **thêm một thuộc tính mở rộng** (`extendedCategoryStatus`) chứa dữ liệu cũ và mới.

*   **Tránh các thay đổi gây lỗi đối với dữ liệu đầu vào và tham số (Avoiding breaking changes to input data and parameters)**:
    *   **Đổi tên thuộc tính** (Renaming a property): Người dùng gửi tên cũ sẽ nhận lỗi 400 Bad Request.
    *   **Xóa thuộc tính** (Removing a property): Tương tự như đổi tên, việc xóa một thuộc tính bắt buộc sẽ gây lỗi.
    *   **Thay đổi định dạng hoặc loại** (Modifying format or type): Thay đổi một số thành một chuỗi, hoặc một đối tượng thành một ID đơn giản, sẽ khiến người dùng cũ không thể gửi dữ liệu đúng định dạng.
    *   **Thay đổi đặc điểm thuộc tính** (Modifying (decreasing) characteristics): Ví dụ, giảm giới hạn giá trị tối đa của một số (`maxValue`) sẽ khiến các yêu cầu hợp lệ trước đây trở nên không hợp lệ.
    *   **Xóa giá trị khỏi enum** (Removing values from enums): Người dùng gửi giá trị cũ sẽ gặp lỗi.
    *   **Thay đổi ý nghĩa của thuộc tính** (Changing a property's meaning): Có thể gây ra hậu quả nghiêm trọng cho nhà cung cấp, vì họ có thể xử lý dữ liệu sai.
    *   **Thêm một thuộc tính bắt buộc mới** (Adding a mandatory property): Các yêu cầu của người dùng cũ sẽ thiếu thuộc tính này và sẽ gặp lỗi.
    *   **Cách an toàn để thay đổi dữ liệu đầu vào**: Cách an toàn nhất là **chỉ thêm các thuộc tính tùy chọn**. Bạn cũng có thể biến một **thuộc tính bắt buộc hiện có thành tùy chọn**, hoặc **tăng đặc điểm của thuộc tính** (ví dụ: tăng giới hạn tối đa hoặc độ dài tối đa).

*   **Tránh các thay đổi gây lỗi trong phản hồi thành công và lỗi (Avoiding breaking changes in success and error feedback)**:
    *   **Thay đổi cấu trúc dữ liệu lỗi** (Modifying error data structure): Ví dụ, đổi tên thuộc tính `error_message` thành `message` trong phản hồi lỗi có thể khiến ứng dụng người dùng không thể phân tích cú pháp hoặc hiển thị lỗi đúng cách.
    *   **Thay đổi mã trạng thái HTTP** (Modifying HTTP status codes): Ví dụ, nếu API thay đổi trạng thái 400 Bad Request sang 404 Not Found cho cùng một loại lỗi, người dùng có thể không xử lý lỗi đúng cách.
    *   **Thay đổi ý nghĩa của mã trạng thái HTTP** (Changing HTTP status code meaning): Sử dụng 200 OK cho một phản hồi lỗi là một thay đổi gây lỗi, vì người dùng sẽ nghĩ rằng yêu cầu đã thành công.

*   **Tránh các thay đổi gây lỗi đối với mục tiêu và luồng (Avoiding breaking changes to goals and flows)**:
    *   **Đổi tên hoặc xóa mục tiêu** (Renaming or removing goals): Ví dụ, đổi tên tài nguyên `transfers` thành `money-transfers` hoặc xóa phương thức GET cho tài nguyên `transfer` sẽ dẫn đến lỗi 404 Not Found hoặc 405 Method Not Allowed.
    *   **Thêm mục tiêu bắt buộc mới vào luồng** (Adding a new mandatory goal to a flow): Nếu một bước mới bắt buộc được thêm vào luồng (ví dụ: xác thực bằng OTP sau khi tạo chuyển khoản), người dùng cũ sẽ không thực hiện bước đó và không thể hoàn thành mục tiêu, ngay cả khi giao diện không thay đổi. Đây là **thay đổi gây lỗi im lặng** (silent breaking change) vì không có lỗi rõ ràng nào được trả về.

*   **Tránh các vi phạm bảo mật và thay đổi gây lỗi (Avoiding security breaches and breaking changes)**:
    *   **Thay đổi cách thức lấy token** (Modifying how tokens are acquired): Nếu quy trình xác thực token thay đổi (ví dụ: từ OAuth 1 sang OAuth 2), tất cả các ứng dụng người dùng sẽ cần cập nhật.
    *   **Xóa thông tin nhạy cảm khỏi token truy cập** (Removing sensitive data from access tokens): Nếu ID người dùng cuối (end user ID) bị xóa khỏi dữ liệu đính kèm trong token truy cập, hệ thống triển khai có thể hiểu sai quyền của người dùng, dẫn đến vi phạm bảo mật hoặc lỗi máy chủ.
    *   **Thay đổi phạm vi (scopes)**: Thay đổi phạm vi (ví dụ: đổi tên `beneficiary:create` thành `beneficiary:add`) có thể gây ra lỗi cho người dùng cũ hoặc cấp quyền không mong muốn.

*   **Lưu ý về hợp đồng giao diện vô hình (Being aware of the invisible interface contract)**:
    *   Người dùng API có thể dựa vào các **hành vi có thể quan sát được của hệ thống** không được mô tả rõ ràng trong tài liệu. Ví dụ, người dùng có thể giả định rằng một trường chuỗi có độ dài tối đa dựa trên dữ liệu họ đã nhận được, và nếu độ dài đó tăng lên, ứng dụng của họ có thể gặp lỗi.
    *   Thay đổi hiệu suất của API (ví dụ: tăng thời gian phản hồi) có thể gây lỗi hết thời gian chờ (timeout) cho người dùng đã điều chỉnh ứng dụng của họ theo thời gian phản hồi trước đây.
    *   **Luật Hyrum** (Hyrum's law) nói rằng: "Với đủ số lượng người dùng của một API, bất kể bạn hứa gì trong hợp đồng: tất cả các hành vi có thể quan sát được của hệ thống của bạn sẽ bị ai đó phụ thuộc vào".

*   **Khi nào một thay đổi gây lỗi không phải là vấn đề? (Introducing a breaking change is not always a problem)**:
    *   Trong bối cảnh **API nội bộ** hoặc **API riêng tư** (private API) được sử dụng bởi các ứng dụng thuộc cùng một công ty (ví dụ: ứng dụng di động hoặc SPA của chính công ty), việc giới thiệu các thay đổi gây lỗi có thể chấp nhận được. Điều này là do tất cả người dùng có thể được cập nhật đồng bộ với API. Tuy nhiên, đối với các **API công cộng** (public API) được sử dụng bởi hàng ngàn ứng dụng bên thứ ba, các thay đổi gây lỗi chắc chắn không phải là một lựa chọn vì nó có thể khiến người dùng mất lòng tin và chuyển sang API của đối thủ.

### 2. Phiên bản hóa API (Versioning an API)

Khi một thay đổi gây lỗi là không thể tránh khỏi, **phiên bản hóa API là một cách khôn ngoan để xử lý tình huống đó**.

*   **Phiên bản hóa API so với phiên bản hóa triển khai (Contrasting API and implementation versioning)**:
    *   **Phiên bản của API** (API version) phát triển dựa trên **những thay đổi đối với hợp đồng giao diện** (interface contract) (những thay đổi có thể nhìn thấy từ góc độ người dùng).
    *   **Phiên bản của việc triển khai** (implementation version) liên quan đến cách mã nguồn phát triển và có thể thay đổi mà không ảnh hưởng đến phiên bản API (ví dụ: tối ưu hóa hiệu suất nội bộ).
    *   Người dùng API chủ yếu chỉ quan tâm đến các **thay đổi phiên bản lớn** (major versions) báo hiệu các thay đổi gây lỗi.

*   **Chọn cách biểu diễn phiên bản API từ góc độ người dùng (Choosing an API versioning representation from the consumer's perspective)**: Có nhiều cách để chỉ định phiên bản API trong yêu cầu HTTP:
    *   **Trong đường dẫn** (Path versioning): `GET /v1/transfers` hoặc `GET /v2/transfers`. Đây là cách phổ biến và đơn giản, dễ nhìn thấy phiên bản đang được sử dụng.
    *   **Trong tên miền hoặc tên miền phụ** (Domain or subdomain versioning): `api.bankingcompany.com` cho v1 và `apiv2.bankingcompany.com` cho v2.
    *   **Trong tham số truy vấn** (Query parameter): `GET /transfers?version=2`. Cách này đơn giản nhưng có thể trộn lẫn tham số kỹ thuật với tham số chức năng trong cùng một truy vấn, không được khuyến nghị.
    *   **Trong header tùy chỉnh** (Custom header): `Version: 2`. Không phải là một phần của tiêu chuẩn HTTP, có thể khiến người dùng không quen thuộc.
    *   **Sử dụng đàm phán nội dung** (Content negotiation): Chỉ định phiên bản thông qua header `Accept` (ví dụ: `Accept: application/vnd.bankingapi.v2+json`). Đây là một tính năng HTTP tiêu chuẩn nhưng ít phổ biến hơn và có thể phức tạp hơn để sử dụng.
    *   **Dựa trên cấu hình người dùng** (Consumer's configuration): Máy chủ quản lý phiên bản được sử dụng bởi mỗi người dùng mà không cần người dùng chỉ định nó trong yêu cầu. Dễ dùng cho người dùng nhưng yêu cầu cấu hình riêng cho mỗi người dùng.
    *   **Mỗi cách đều có ưu và nhược điểm**. Lựa chọn nên dựa trên bối cảnh, sự quen thuộc của người dùng và tính dễ sử dụng.

*   **Chọn mức độ chi tiết của phiên bản API (Choosing API versioning granularity)**:
    *   **Cấp độ API** (API-level versioning): Phiên bản hóa toàn bộ API. Khi có một thay đổi gây lỗi, toàn bộ API sẽ được nâng cấp phiên bản (ví dụ: từ v1 lên v2). Ưu điểm: Không cần suy nghĩ về việc phiên bản nào của các hoạt động hoặc tài nguyên có thể hoạt động cùng nhau (mọi thứ trong cùng một phiên bản API đều tương thích). Nhược điểm: Một thay đổi nhỏ cũng khiến toàn bộ API lên phiên bản mới, không cung cấp thông tin về thay đổi cụ thể nào. Đây là **thực hành phổ biến nhất cho REST API**.
    *   **Cấp độ tài nguyên** (Resource-level versioning): Phiên bản hóa mỗi tài nguyên riêng lẻ. Ví dụ: `/v1/transfers` và `/v2/transfers`. Ưu điểm: Gợi ý về những thay đổi trong tài nguyên cụ thể. Nhược điểm: Khó khăn trong việc đoán xem các phiên bản tài nguyên khác nhau có thể hoạt động cùng nhau không, có thể dẫn đến sự không rõ ràng trong API.
    *   **Cấp độ mục tiêu/hoạt động** (Goal/operation-level versioning): Phiên bản hóa mỗi mục tiêu/hoạt động riêng lẻ. Ví dụ: `POST /v1/transfers` và `POST /v2/transfers`. Ưu điểm: Cho biết rõ ràng mục tiêu nào đã thay đổi. Nhược điểm: Tương tự cấp độ tài nguyên, khó đoán biết sự tương thích giữa các phiên bản mục tiêu khác nhau.
    *   **Cấp độ dữ liệu/thông điệp** (Data/message-level versioning): Phiên bản hóa dữ liệu trong phần thân yêu cầu và phản hồi, thường sử dụng đàm phán nội dung. Ưu điểm: Cho biết rõ ràng dữ liệu/thông điệp nào đã thay đổi. Nhược điểm: Chỉ áp dụng cho phần thân, không bao gồm header hoặc tham số truy vấn; khó đoán biết sự tương thích giữa các phiên bản yêu cầu/phản hồi.
    *   Có thể **kết hợp các mức độ chi tiết khác nhau** tùy thuộc vào bối cảnh.

*   **Hiểu tác động của phiên bản hóa API ngoài thiết kế (Understanding the impact of API versioning beyond design)**:
    *   **Tác động đến người dùng**: Giới thiệu một thay đổi gây lỗi có thể khiến người dùng không hài lòng, đặc biệt nếu nó không mang lại giá trị rõ ràng (ví dụ: chuyển từ OAuth 1 sang OAuth 2).
    *   **Tác động đến triển khai và quản lý sản phẩm**: Việc hỗ trợ nhiều phiên bản API đòi hỏi công sức triển khai và bảo trì bổ sung (ví dụ: chạy hai phiên bản backend song song). Quyết định bao nhiêu phiên bản sẽ được hỗ trợ và trong bao lâu phụ thuộc vào bối cảnh của công ty.

### 3. Thiết kế API có khả năng mở rộng (Designing APIs with extensibility in mind)

**Khả năng mở rộng (extensibility)** là một nguyên tắc thiết kế phần mềm, nơi việc triển khai xem xét sự phát triển trong tương lai, nhằm giảm thiểu tác động đến các chức năng hệ thống hiện có khi thêm mới hoặc sửa đổi. Bằng cách thiết kế cẩn thận dữ liệu, tương tác và luồng, và chọn mức độ chi tiết phù hợp cho phiên bản hóa, chúng ta có thể thiết kế API có khả năng mở rộng, giúp dễ dàng phát triển và giảm rủi ro thay đổi gây lỗi.

*   **Thiết kế dữ liệu có khả năng mở rộng (Designing extensible data)**:
    *   **Sử dụng các envelope dữ liệu** (data envelopes): Bao bọc dữ liệu trả về trong một đối tượng bao quanh (envelope) cho phép thêm các thuộc tính mới mà không ảnh hưởng đến ứng dụng của người dùng cũ.
    *   **Cung cấp dữ liệu tự mô tả** (self-descriptive data): Sử dụng các định dạng hoặc thuộc tính mà bản thân chúng đã mô tả ý nghĩa (ví dụ: thay vì `recurringPeriodType=MONTHLY` và `recurringPeriodValue=1`, chỉ dùng `recurringPeriod=P1M` theo ISO 8601).
    *   **Sử dụng danh sách cho các thuộc tính tương tự** (Using lists for similar properties): Nếu có nhiều thuộc tính tương tự nhau (ví dụ: `event1`, `event2`), hãy cân nhắc nhóm chúng vào một danh sách (array) `events` để dễ dàng thêm các sự kiện mới trong tương lai.
    *   **Chọn loại thuộc tính cẩn thận** (Choose property types carefully): Để đảm bảo khả năng mở rộng, ví dụ, sử dụng một chuỗi (string) thay vì một số (number) cho các ID nếu có khả năng ID sẽ chứa các ký tự không phải số trong tương lai.

*   **Thiết kế tương tác có khả năng mở rộng (Designing extensible interactions)**:
    *   **Bỏ qua các tham số không xác định hoặc không hợp lệ** (Ignoring unknown or invalid parameters): API nên bỏ qua các tham số truy vấn, header hoặc thuộc tính không xác định thay vì trả về lỗi. Điều này giúp tránh gây lỗi cho người dùng cũ khi các tham số không còn tồn tại hoặc đã bị đổi tên.
    *   **Áp dụng giới hạn mềm** (Soft limits): Nếu người dùng yêu cầu `pageSize=150` nhưng API chỉ cho phép `100`, API nên trả về 100 mục thay vì lỗi. Điều này giúp giảm rủi ro thay đổi gây lỗi khi giới hạn được điều chỉnh.
    *   **Không hy sinh sự chính xác để tránh lỗi**: Mặc dù nên cố gắng tránh lỗi, nhưng API không nên thực hiện hành vi sai (ví dụ: thực hiện chuyển khoản 10.000$ khi người dùng yêu cầu 15.000$).

*   **Thiết kế luồng có khả năng mở rộng (Designing extensible flows)**:
    *   **Độc lập hóa các bước trong luồng**: Thiết kế mỗi mục tiêu trong một luồng để nó có thể được sử dụng một cách độc lập (standalone). Điều này giúp người dùng có thể sử dụng các phần của API theo nhiều cách khác nhau, không chỉ theo luồng được thiết kế ban đầu.
    *   **Sử dụng các đầu vào và đầu ra được chấp nhận rộng rãi**: Sử dụng các ID và giá trị tiêu chuẩn, phổ biến để dễ dàng kết nối các mục tiêu khác nhau trong một luồng.
    *   **Tránh sự phụ thuộc vào luồng UI cụ thể**: Đừng gắn chặt thiết kế luồng API với một luồng giao diện người dùng cụ thể, cho phép API được sử dụng bởi nhiều loại ứng dụng khác nhau.

*   **Thiết kế API có khả năng mở rộng (Designing extensible APIs)**:
    *   **Xây dựng các API nhỏ hơn**: API càng lớn, nguy cơ thay đổi gây lỗi càng cao. Chia nhỏ API thành các phần nhỏ hơn, độc lập và có ý nghĩa chức năng sẽ giúp quản lý và phát triển chúng dễ dàng hơn.
    *   **Đánh giá từng mục tiêu mới**: Khi thêm một mục tiêu mới vào API hiện có, hãy đánh giá xem nó có nên là một phần của một API khác hay không, dựa trên các nguyên tắc phân tách trách nhiệm.