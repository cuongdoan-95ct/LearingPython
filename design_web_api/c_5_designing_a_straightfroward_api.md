**5.1 Thiết kế các biểu diễn rõ ràng (Designing Straightforward Representations)**

Phần này tập trung vào cách lựa chọn tên, định dạng dữ liệu và bản thân dữ liệu để nâng cao hoặc làm giảm khả năng sử dụng của API. Giống như các vật dụng hàng ngày, cách các khái niệm và thông tin được biểu diễn có thể ảnh hưởng lớn đến trải nghiệm người dùng.

*   **5.1.1 Chọn tên rõ ràng (Choosing Crystal-Clear Names)**
    Tên khó hiểu, từ vựng không quen thuộc hoặc viết tắt khó hiểu có thể làm một vật dụng hoặc một API trở nên hoàn toàn khó hiểu. Khi thiết kế API, bạn cần đặt tên cho đầu vào, đầu ra, tài nguyên, phản hồi, tham số và các thuộc tính (properties) trong đó.
    **Ví dụ:** Trong API Ngân hàng (Banking API), một thuộc tính chỉ trạng thái bảo vệ thấu chi có thể được đặt tên là `bankAccountOverdraftProtectionFeatureActive` (khó hiểu), `bankAccountOverdraftProtectionFeature` (ít khó hiểu hơn), `bankAccountOverdraftProtection` (tốt hơn) hoặc chỉ đơn giản là `overdraftProtection` (rõ ràng nhất). Mục tiêu là chọn tên mà người dùng có thể hiểu ngay lập tức. Điều này cũng áp dụng cho các hành động (goals). Thay vì `POST /trf`, hãy sử dụng `POST /transfers` cho mục tiêu chuyển tiền.

*   **5.1.2 Chọn kiểu và định dạng dữ liệu dễ sử dụng (Choosing Easy-to-Use Data Types and Formats)**
    Việc sử dụng kiểu dữ liệu hoặc định dạng không phù hợp có thể cản trở sự hiểu biết và sử dụng. Mặc dù phần mềm có thể xử lý các định dạng phức tạp, nhà phát triển (người dùng API) vẫn cần hiểu dữ liệu thô để học, kiểm tra hoặc gỡ lỗi.
    **Ví dụ:** Sử dụng dấu thời gian UNIX (`1534960860`) cho ngày thay vì định dạng ngày ISO 8601 dễ đọc (`2018-08-22T18:01:00z`). Sử dụng mã số (`1`) cho loại tài khoản thay vì giá trị dễ đọc như chuỗi (`checking`). Chọn kiểu dữ liệu cơ bản, có thể chuyển đổi giữa các ngôn ngữ lập trình như chuỗi (string), số (number), boolean, ngày (date), mảng (array), hoặc đối tượng (object).

*   **5.1.3 Chọn dữ liệu sẵn sàng sử dụng (Choosing Ready-to-Use Data)**
    API nên cung cấp dữ liệu liên quan và hữu ích, vượt ra ngoài dữ liệu cơ bản. Việc cung cấp dữ liệu giúp người dùng hiểu và tránh công việc xử lý thêm phía người dùng là rất quan trọng.
    **Ví dụ:** Nếu bạn phải sử dụng một danh pháp bằng số cho loại tài khoản (ví dụ: 1 cho tài khoản tiết kiệm, 2 cho tài khoản vãng lai), hãy cung cấp thêm một thuộc tính `typeName` với giá trị chuỗi tương ứng (`savings` hoặc `checking`). Thay vì cung cấp ngày tạo tài khoản (`creationDate`), hãy cung cấp trực tiếp số năm tài khoản đã mở (`age`) nếu đó là thông tin người dùng thực sự cần. Đối với URL tài nguyên, sử dụng các giá trị có ý nghĩa và dễ đọc như số tài khoản (`/accounts/0001234567`) thay vì các UUID khó hiểu (`/accounts/473e3283-a3b3-4941-aa48-d8163ead9ffc`).

**5.2 Thiết kế tương tác rõ ràng (Designing Straightforward Interactions)**

Tương tác trong API bao gồm việc người dùng cung cấp đầu vào (inputs) và nhận phản hồi (feedback). Thiết kế tốt các tương tác này có thể giúp người dùng cảm thấy hài lòng, trong khi thiết kế kém có thể gây khó chịu.

*   **5.2.1 Yêu cầu đầu vào rõ ràng (Requesting Straightforward Inputs)**
    Đầu vào nên đơn giản, dễ hiểu và dễ cung cấp.
    **Ví dụ:** Đối với mục tiêu chuyển tiền trong Banking API, đầu vào bao gồm số tiền, tài khoản nguồn và đích. Chuyển tiền định kỳ hoặc bị trì hoãn cần thêm thông tin như ngày thực hiện, số lần xảy ra và tần suất. Sử dụng tên rõ ràng (`source`, `destination`, `amount`) thay vì viết tắt (`src`, `dst`, `amt`). Sử dụng định dạng ngày ISO 8601 và giá trị chuỗi dễ đọc cho tần suất và loại chuyển tiền thay vì dấu thời gian UNIX hoặc mã số. Yêu cầu dữ liệu dễ cung cấp như số tài khoản thay vì UUID, số lần xảy ra thay vì ngày kết thúc, và có thể bỏ qua thuộc tính `type` nếu có thể suy luận từ các thuộc tính khác.

*   **5.2.2 Xác định tất cả các phản hồi lỗi có thể xảy ra (Identifying All Possible Error Feedbacks)**
    Tương tác API không phải lúc nào cũng thành công, vì vậy cần xác định tất cả các lỗi có thể xảy ra cho mỗi mục tiêu. Có ba loại lỗi chính: lỗi yêu cầu sai định dạng (malformed request errors), lỗi chức năng (functional errors) và lỗi máy chủ (server errors). Lỗi yêu cầu sai định dạng xảy ra khi máy chủ không thể hiểu yêu cầu (ví dụ: thiếu tham số bắt buộc, sai định dạng dữ liệu). Lỗi chức năng xảy ra khi logic nghiệp vụ bị vi phạm (ví dụ: số tiền vượt quá giới hạn). Lỗi máy chủ xảy ra do sự cố nội bộ của nhà cung cấp.

*   **5.2.3 Trả về phản hồi lỗi đầy đủ thông tin (Returning Informative Error Feedback)**
    Phản hồi lỗi phải giải thích rõ vấn đề là gì và cung cấp thông tin để người dùng (nhà phát triển hoặc người dùng cuối) có thể tự khắc phục nếu có thể. API dựa trên HTTP sử dụng mã trạng thái HTTP để chỉ ra thành công hay thất bại (ví dụ: 200 OK, 400 Bad Request). Tuy nhiên, mã trạng thái thôi là chưa đủ.
    **Ví dụ:** Một phản hồi `400 Bad Request` có thể được kèm theo phần thân phản hồi (response body) chứa thông tin chi tiết về lỗi. Thay vì chỉ có thông báo "Amount is mandatory", hãy cung cấp mã lỗi có thể lập trình được (`MISSING_MANDATORY_PARAMETER`) và chỉ rõ thuộc tính gây lỗi (`source: "amount"`). Điều này giúp ứng dụng khách (client application) xác định chính xác vấn đề và hiển thị thông báo phù hợp cho người dùng cuối (ví dụ: làm nổi bật trường nhập số tiền).

*   **5.2.4 Trả về phản hồi lỗi toàn diện (Returning Exhaustive Error Feedback)**
    Tránh trả về lỗi từng cái một. Nếu một yêu cầu có nhiều vấn đề (ví dụ: thiếu cả `source` và `destination`), phản hồi lỗi đầu tiên nên liệt kê tất cả các vấn đề đã được phát hiện. Điều này giúp người dùng sửa tất cả các lỗi trong một lần thay vì phải gọi lại API nhiều lần.

*   **5.2.5 Trả về phản hồi thành công đầy đủ thông tin (Returning Informative Success Feedback)**
    Phản hồi thành công không chỉ nên là xác nhận đơn giản. Nó phải cung cấp thông tin hữu ích về những gì đã được thực hiện và có thể giúp người dùng thực hiện các bước tiếp theo.
    **Ví dụ:** Đối với một chuyển tiền tức thời, trả về mã trạng thái `201 Created`. Đối với chuyển tiền bị trì hoãn hoặc định kỳ, trả về `202 Accepted`, ngụ ý rằng yêu cầu đã được chấp nhận và sẽ được thực hiện sau. Phần thân phản hồi có thể chứa thông tin chi tiết về tài nguyên vừa tạo (ví dụ: ID chuyển tiền, trạng thái, loại), giúp người dùng có thể hủy chuyển tiền sau này nếu cần.

**5.3 Thiết kế luồng rõ ràng (Designing Straightforward Flows)**

Luồng tương tác là chuỗi các lần gọi API để hoàn thành một mục tiêu phức tạp hơn. Sự dễ sử dụng của API phụ thuộc vào sự đơn giản của luồng này.

*   **5.3.1 Xây dựng chuỗi mục tiêu rõ ràng (Building a Straightforward Goal Chain)**
    Đảm bảo mỗi mục tiêu trong chuỗi đều rõ ràng. Các đầu vào cần thiết cho một mục tiêu nên có sẵn từ người dùng hoặc được cung cấp bởi đầu ra của các mục tiêu trước đó trong chuỗi.
    **Ví dụ:** Để chuyển tiền, người dùng cần biết tài khoản nguồn và đích. API có thể cung cấp mục tiêu `list accounts` và `list beneficiaries` để người dùng lấy các giá trị khả dụng này trước khi gọi mục tiêu `transfer money`. Phản hồi lỗi đầy đủ thông tin cho mục tiêu `transfer money` giúp giảm số lần gọi API lặp đi lặp lại để sửa lỗi, giữ cho chuỗi mục tiêu ngắn gọn và hiệu quả.

*   **5.3.2 Ngăn chặn lỗi (Preventing Errors)**
    Ngăn chặn lỗi xảy ra là một cách tốt để làm cho luồng mượt mà và ngắn gọn hơn. API có thể cung cấp dữ liệu giúp người dùng tránh gửi yêu cầu sai.
    **Ví dụ:** Thay vì người dùng phải đoán các cặp tài khoản nguồn/đích hợp lệ, API có thể cung cấp mục tiêu `list sources` và `list destinations for source` để trả về các lựa chọn hợp lệ. Điều này sử dụng nguyên tắc tương tự như ràng buộc "Code on Demand" trong REST, nơi API cung cấp dữ liệu (hoặc mã) để hướng dẫn hành vi của người dùng.

*   **5.3.3 Tổng hợp mục tiêu (Aggregating Goals)**
    Tổng hợp các mục tiêu nhỏ thành một mục tiêu lớn hơn có thể tối ưu hóa luồng tương tác.
    **Ví dụ:** Thay vì người dùng phải gọi `list sources` và `list destinations for source` riêng biệt, API có thể cung cấp một mục tiêu duy nhất là `list sources and destinations` để trả về tất cả các cặp nguồn/đích hợp lệ trong một lần gọi. Tuy nhiên, việc tổng hợp chỉ nên thực hiện nếu nó thực sự có ý nghĩa chức năng đối với người dùng và cần cân nhắc các vấn đề về hiệu suất.

*   **5.3.4 Thiết kế luồng không trạng thái (Designing Stateless Flows)**
    Mỗi mục tiêu API nên có thể sử dụng độc lập và tất cả các đầu vào cần thiết phải được khai báo rõ ràng trong yêu cầu. Luồng tương tác không nên phụ thuộc vào dữ liệu trạng thái (state data) được lưu trữ trên máy chủ từ các lần gọi trước đó (ví dụ: trong một phiên - session).
    **Ví dụ:** Mục tiêu `transfer money` không nên dựa vào việc người dùng đã gọi `list destinations` trước đó trong cùng một phiên. Nguyên tắc này tương ứng với ràng buộc "Statelessness" trong REST, giúp API dễ tái sử dụng trong các ngữ cảnh khác nhau và cho phép triển khai có khả năng mở rộng.

Tóm lại, thiết kế API rõ ràng tập trung vào việc làm cho các thành phần API (biểu diễn, tương tác, luồng) dễ hiểu, dễ sử dụng và có thể dự đoán được bằng cách sử dụng tên rõ ràng, định dạng dữ liệu phù hợp, cung cấp thông tin đầy đủ (bao gồm cả lỗi và thành công) và xây dựng các luồng tương tác đơn giản, hiệu quả và không trạng thái.

Hy vọng thông tin chi tiết này bằng tiếng Việt sẽ giúp bạn hiểu rõ hơn về nội dung của Chương 5.