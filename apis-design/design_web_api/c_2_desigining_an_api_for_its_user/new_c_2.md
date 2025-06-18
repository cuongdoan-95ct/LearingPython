### Chương 2: Thiết kế API cho người dùng

Chương này nhấn mạnh rằng một API tốt phải được thiết kế với trọng tâm là **người dùng** (các nhà phát triển sử dụng API). Thay vì chỉ tập trung vào khía cạnh kỹ thuật, thiết kế API cần xuất phát từ nhu cầu, mục tiêu, và trải nghiệm của người dùng để đảm bảo API dễ sử dụng, hiệu quả, và đáp ứng được mục đích sử dụng.

#### 2.1 Hiểu người dùng API
* Tập trung vào những gì người dùng có thể làm và mọi thứ sẽ xuôn sẻ.
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

##### 2.1.1 Người dùng API là ai?
- **Nhà phát triển (Developers)**: Người dùng chính của API là các nhà phát triển, những người viết mã để tích hợp API vào ứng dụng của họ. Họ có thể thuộc các nhóm khác nhau:
  - **Nhà phát triển nội bộ**: Làm việc trong cùng tổ chức, sử dụng API riêng tư.
  - **Nhà phát triển bên thứ ba**: Sử dụng API công khai hoặc API đối tác, thường không có quyền truy cập vào mã nguồn của API.
- **Các loại người dùng khác**: Ngoài nhà phát triển, còn có các bên liên quan như quản lý dự án, nhân viên kiểm thử, hoặc khách hàng doanh nghiệp sử dụng API trong các hệ thống tích hợp.
- **Nhu cầu của người dùng**: 
  - API phải dễ hiểu, dễ tích hợp, và phù hợp với mục tiêu cụ thể của họ (ví dụ: lấy dữ liệu, thực hiện hành động, hoặc tích hợp với hệ thống khác).
  - Cần tài liệu rõ ràng, ví dụ mẫu, và hỗ trợ khi gặp vấn đề.

##### 2.1.2 Hiểu bối cảnh sử dụng API
- **Bối cảnh (Context)**: Thiết kế API cần xem xét môi trường mà API sẽ được sử dụng:
  - **Loại ứng dụng**: API được dùng trong ứng dụng di động, website, hay hệ thống doanh nghiệp?
  - **Hạn chế kỹ thuật**: Người dùng có thể bị giới hạn bởi băng thông mạng, thiết bị phần cứng, hoặc ngôn ngữ lập trình.
  - **Mục tiêu kinh doanh**: API có thể phục vụ các mục đích như tăng doanh thu, cải thiện trải nghiệm khách hàng, hoặc tối ưu hóa quy trình nội bộ.
- **Phương pháp tìm hiểu bối cảnh**:
  - **Phỏng vấn người dùng**: Nói chuyện trực tiếp với nhà phát triển hoặc các bên liên quan để hiểu nhu cầu của họ.
  - **Phân tích kịch bản sử dụng (Use Cases)**: Xác định các tình huống cụ thể mà API sẽ được sử dụng, ví dụ: lấy danh sách sản phẩm, gửi thông báo, hoặc xử lý thanh toán.
  - **Persona của nhà phát triển**: Xây dựng hồ sơ của các loại nhà phát triển sử dụng API (ví dụ: nhà phát triển di động, nhà phát triển backend, hoặc nhà phát triển tự do) để hiểu rõ kỹ năng, công cụ, và thách thức của họ.

#### 2.2 Các nguyên tắc thiết kế API lấy người dùng làm trung tâm
*  API cung cấp một biểu diễn về các mục tiêu có thể đạt được bằng cách sử dụng nó. API của lò vi sóng cho phép người dùng hâm nóng thức ăn. Để đạt được, một mục tiêu có thể cần một số thông tin (đầu vào). Người dùng phải cung cấp cài đặt công suất và thời lượng để hâm nóng thức ăn của họ. Việc triển khai mục tiêu sử dụng thông tin được cung cấp thông qua API để vận hành. Trong trường hợp này, việc triển khai sẽ bật và tắt magnetron theo tốc độ nhất định theo công suất được cung cấp trong thời lượng được cung cấp. Và khi đạt được mục tiêu, nó có thể trả về một số thông tin.

##### 2.2.1 Tính dễ sử dụng (Usability)
- API cần **đơn giản và trực quan** để nhà phát triển có thể hiểu và sử dụng mà không gặp khó khăn.
- **Nguyên tắc**:
  - **Tên gọi rõ ràng**: Sử dụng tên endpoint, tham số, và phản hồi dễ hiểu, phản ánh đúng chức năng (ví dụ: `/users` để lấy danh sách người dùng, thay vì `/getAllUsersData`).
  - **Tính nhất quán**: Các endpoint, định dạng dữ liệu, và cách đặt tên phải đồng nhất trong toàn bộ API.
  - **Phản hồi rõ ràng**: API nên trả về thông báo lỗi chi tiết và dễ hiểu khi có vấn đề (ví dụ: mã lỗi HTTP 400 với thông điệp “Thiếu tham số email”).

##### 2.2.2 Tập trung vào mục tiêu của người dùng
- Thiết kế API dựa trên **kịch bản sử dụng thực tế** thay vì chỉ tập trung vào dữ liệu hoặc cấu trúc hệ thống.
- **Ví dụ**: Nếu nhà phát triển cần lấy thông tin đơn hàng của khách hàng, API nên cung cấp endpoint như `/orders/{customerId}` thay vì yêu cầu họ truy vấn nhiều endpoint khác nhau để ghép dữ liệu.

##### 2.2.3 Hỗ trợ trải nghiệm nhà phát triển (Developer Experience - DX)
- **Tài liệu API**: Cung cấp tài liệu chi tiết, dễ hiểu, với các ví dụ cụ thể (ví dụ: mã mẫu bằng cURL, Python, hoặc JavaScript).
- **Công cụ hỗ trợ**: Cung cấp sandbox (môi trường thử nghiệm) hoặc API playground để nhà phát triển thử nghiệm trước khi tích hợp.
- **Hỗ trợ lỗi**: API cần trả về thông báo lỗi dễ hiểu, có mã lỗi cụ thể, và gợi ý cách khắc phục.

##### 2.2.4 Linh hoạt nhưng không phức tạp
- API nên linh hoạt để hỗ trợ nhiều kịch bản sử dụng khác nhau, nhưng không nên quá phức tạp đến mức làm nhà phát triển bối rối.
- **Ví dụ**: Thay vì cung cấp một endpoint duy nhất với hàng chục tham số tùy chọn, hãy chia thành nhiều endpoint chuyên biệt hơn.

#### 2.3 Quy trình thiết kế API lấy người dùng làm trung tâm

1. **Xác định mục tiêu và người dùng**:
   - Hỏi: “API này được dùng để làm gì? Ai sẽ sử dụng nó?”
   - Ví dụ: Một API cho ứng dụng thương mại điện tử có thể cần phục vụ cả ứng dụng di động (cho khách hàng) và hệ thống quản lý kho (cho nhân viên).

2. **Thu thập yêu cầu**:
   - Phỏng vấn hoặc khảo sát nhà phát triển để hiểu nhu cầu cụ thể.
   - Ví dụ: Nhà phát triển cần API trả về danh sách sản phẩm với bộ lọc theo danh mục, giá, hoặc đánh giá.

3. **Thiết kế kịch bản sử dụng**:
   - Liệt kê các kịch bản cụ thể, như “Lấy danh sách sản phẩm”, “Thêm sản phẩm vào giỏ hàng”, hoặc “Cập nhật trạng thái đơn hàng”.
   - Mỗi kịch bản nên được ánh xạ vào một endpoint hoặc một tập hợp endpoint.

4. **Tạo prototype và kiểm thử**:
   - Tạo phiên bản thử nghiệm của API (dùng công cụ như Swagger hoặc Postman).
   - Mời nhà phát triển dùng thử và thu thập phản hồi để cải thiện.

5. **Lặp lại và cải tiến**:
   - Dựa trên phản hồi, điều chỉnh thiết kế API để phù hợp hơn với nhu cầu người dùng.

#### 2.4 Ví dụ minh họa: Thiết kế API cho ứng dụng thương mại điện tử

**Bối cảnh**: Một công ty thương mại điện tử muốn xây dựng API để hỗ trợ ứng dụng di động cho khách hàng và hệ thống quản lý kho nội bộ.

**Bước 1: Xác định người dùng và nhu cầu**
- **Người dùng**:
  - Nhà phát triển ứng dụng di động: Cần API để hiển thị sản phẩm, thêm vào giỏ hàng, và xử lý thanh toán.
  - Nhà phát triển hệ thống kho: Cần API để cập nhật số lượng tồn kho và kiểm tra trạng thái đơn hàng.
- **Nhu cầu**:
  - Ứng dụng di động: Lấy danh sách sản phẩm, lọc theo danh mục hoặc giá, thêm sản phẩm vào giỏ hàng.
  - Hệ thống kho: Cập nhật số lượng tồn kho khi đơn hàng được đặt.

**Bước 2: Thiết kế kịch bản sử dụng**
- **Kịch bản 1 (Ứng dụng di động)**: Khách hàng muốn xem danh sách sản phẩm trong danh mục “Điện thoại”.
  - Endpoint: `GET /products?category=phones`
  - Phản hồi: Danh sách sản phẩm với thông tin như tên, giá, mô tả, và hình ảnh.
- **Kịch bản 2 (Ứng dụng di động)**: Khách hàng thêm sản phẩm vào giỏ hàng.
  - Endpoint: `POST /cart`
  - Tham số: `{ "productId": 123, "quantity": 2 }`
  - Phản hồi: Xác nhận sản phẩm đã được thêm.
- **Kịch bản 3 (Hệ thống kho)**: Cập nhật số lượng tồn kho sau khi đơn hàng được đặt.
  - Endpoint: `PATCH /inventory/{productId}`
  - Tham số: `{ "quantity": 50 }`
  - Phản hồi: Xác nhận số lượng tồn kho đã được cập nhật.

**Bước 3: Thiết kế API**
- **Endpoint mẫu**:
  ```json
  GET /products?category=phones
  ```
  **Phản hồi mẫu**:
  ```json
  [
    {
      "id": 123,
      "name": "Smartphone XYZ",
      "price": 599.99,
      "description": "Mô tả sản phẩm",
      "image": "https://example.com/images/xyz.jpg"
    },
    ...
  ]
  ```
- **Thông báo lỗi mẫu** (nếu danh mục không tồn tại):
  ```json
  {
    "error": {
      "code": 400,
      "message": "Danh mục 'phones' không tồn tại",
      "details": "Vui lòng kiểm tra lại tham số category"
    }
  }
  ```

**Bước 4: Kiểm thử và phản hồi**
- Mời nhà phát triển ứng dụng di động thử nghiệm endpoint `/products` và `/cart`.
- Thu thập phản hồi: Ví dụ, nhà phát triển yêu cầu thêm bộ lọc giá (`priceMin`, `priceMax`) cho endpoint `/products`.
- Cập nhật API: `GET /products?category=phones&priceMin=200&priceMax=1000`.

**Bước 5: Cải tiến**
- Thêm tài liệu chi tiết với các ví dụ sử dụng cURL:
  ```bash
  curl -X GET "https://api.example.com/products?category=phones" \
       -H "Authorization: Bearer <token>"
  ```
- Cung cấp sandbox để nhà phát triển thử nghiệm trực tiếp.

#### 2.5 Lợi ích của thiết kế lấy người dùng làm trung tâm
- **Tăng hiệu quả phát triển**: Nhà phát triển có thể tích hợp API nhanh chóng, giảm thời gian phát triển ứng dụng.
- **Giảm lỗi**: API rõ ràng và nhất quán giúp giảm lỗi do hiểu sai hoặc sử dụng sai cách.
- **Tăng sự hài lòng**: Nhà phát triển có trải nghiệm tốt hơn, từ đó tăng khả năng sử dụng API lâu dài.
- **Hỗ trợ mở rộng**: Thiết kế dựa trên nhu cầu thực tế giúp API dễ dàng thích nghi khi yêu cầu thay đổi.

---

### Tóm tắt
Chương 2 nhấn mạnh rằng thiết kế API phải lấy **người dùng** (nhà phát triển) làm trung tâm. Điều này bao gồm:
1. Hiểu rõ người dùng API và bối cảnh sử dụng.
2. Áp dụng các nguyên tắc như tính dễ sử dụng, tính nhất quán, và hỗ trợ trải nghiệm nhà phát triển (DX).
3. Sử dụng quy trình thiết kế gồm xác định mục tiêu, thu thập yêu cầu, thiết kế kịch bản sử dụng, tạo prototype, và cải tiến dựa trên phản hồi.
4. Ví dụ thực tế về thiết kế API cho thương mại điện tử cho thấy cách áp dụng các nguyên tắc này vào một hệ thống cụ thể.

---

**Lưu ý**: Nếu bạn cần thêm chi tiết về bất kỳ phần nào, ví dụ cụ thể hơn, hoặc muốn tôi tạo một biểu đồ minh họa (ví dụ: sơ đồ quy trình thiết kế API), hãy cho tôi biết! Bạn cũng có thể yêu cầu tóm tắt các chương khác hoặc giải thích sâu hơn về một khía cạnh cụ thể của thiết kế API.