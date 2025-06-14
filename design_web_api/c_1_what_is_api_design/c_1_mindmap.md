# What is API design?
* Chương này khám phá API là gì, tại sao việc thiết kế nó lại quan trọng, và ý nghĩa của việc thiết kế API.

### 1.1 What is an API?

#### API được hiểu là một API từ xa (remote API), cụ thể hơn là một web API – một giao diện web cho phần mềm. API trước hết là một giao diện.

##### 1.1.1 API là giao diện web cho phần mềm (An API is a web interface for software)
* API là giao diện lập trình cho ứng dụng.
* Cung cấp các chức năng có thể cần dữ liệu đầu vào hoặc trả về dữ liệu đầu ra.
* **API chỉ là giao diện do phần mềm cung cấp, một lớp trừu tượng của việc triển khai cơ bản**.
* Web API là các API từ xa có thể sử dụng giao thức HTTP.
* **Ví dụ:** Ứng dụng mạng xã hội chia sẻ ảnh sử dụng API camera, API xử lý ảnh, và API từ xa (web API) của máy chủ mạng xã hội.

##### 1.1.2 API biến phần mềm thành các khối LEGO® (APIs turn software into LEGO® bricks)
* Web API biến phần mềm thành các khối có thể tái sử dụng và dễ dàng lắp ráp.
* Cho phép tạo ra các hệ thống mô-đun.
* Có hai loại khối: API công khai và API riêng tư.
    - **API công khai**: Bên thứ ba cung cấp như một dịch vụ hoặc sản phẩm.
    - **API riêng tư**: Thường dành cho mục đích nội bộ của tổ chức.
* Sự khác biệt là ở **đối tượng người dùng (to whom)**.
* **API đối tác (partner API)**: Gần như công khai, phơi bày cho khách hàng hoặc đối tác chọn lọc.

### 1.2 Tại sao thiết kế API lại quan trọng (Why API design matters)

#### API được sử dụng bởi phần mềm, nhưng người xây dựng phần mềm đó là các nhà phát triển – con người.

##### 1.2.1 API công khai hoặc riêng tư là giao diện cho các nhà phát triển khác (A public or private API is an interface for other developers)
* API sớm muộn cũng sẽ được sử dụng bởi các nhà phát triển khác.
* Phải làm mọi thứ để tạo điều kiện thuận lợi cho những người mới này.
* **Trải nghiệm của nhà phát triển (Developer experience - DX)**: Trải nghiệm mà nhà phát triển có khi sử dụng API.

##### 1.2.2 API được tạo ra để ẩn đi chi tiết triển khai (An API is made to hide the implementation)
* Thiết kế API phải **che giấu chi tiết triển khai** (những gì thực sự xảy ra bên trong).
* **Ví dụ:** Khách hàng nhà hàng (consumer) tương tác với người phục vụ (API), không cần biết cách nấu món ăn (implementation).

##### 1.2.3 Hậu quả khủng khiếp của việc thiết kế API kém (The terrible consequences of poorly designed APIs)
* API được thiết kế kém có thể là một nỗi đau thực sự để hiểu và sử dụng.
* **Ví dụ:** Thiết bị UDRC 1138 với giao diện khó hiểu, thông báo cảnh báo khó hiểu.
* Đối với **API công khai**: Có thể dẫn đến không có khách hàng, không có doanh thu.
* Ngay cả với **API riêng tư**: Tăng thời gian, công sức, tiền bạc; có thể bị lạm dụng hoặc ít được sử dụng; tăng chi phí hỗ trợ.
* Thiết kế API lỗi còn có thể dẫn đến **lỗ hổng bảo mật**.
* Việc sửa thiết kế kém sau khi sản xuất tốn kém và làm phiền người dùng.
* API được thiết kế kém chắc chắn sẽ thất bại.

### 1.3 Các yếu tố của thiết kế API (The elements of API design)

#### Học thiết kế API yêu cầu tập trung vào giao diện, biết toàn bộ ngữ cảnh, và thể hiện sự đồng cảm với tất cả người dùng và phần mềm liên quan.

##### 1.3.1 Học các nguyên tắc vượt ra ngoài thiết kế giao diện lập trình (Learning the principles beyond programming interface design)
* Có nhiều cách để phơi bày dữ liệu và khả năng (RPC, SOAP, REST, gRPC, GraphQL).
* Mỗi kiểu API có thể đi kèm với các thực tiễn phổ biến, nhưng không ngăn bạn mắc lỗi.
* **Nếu không biết các nguyên tắc cơ bản, bạn có thể lạc lõng khi chọn một thực tiễn "phổ biến"**.
* **Học các nguyên tắc cơ bản của thiết kế API cung cấp nền tảng vững chắc**.

##### 1.3.2 Khám phá tất cả các khía cạnh của thiết kế API (Exploring all facets of API design)
* Thiết kế API không chỉ là thiết kế giao diện dễ hiểu và dễ sử dụng.
* Phải thiết kế **giao diện hoàn toàn an toàn**.
* Phải **xét đến toàn bộ ngữ cảnh** (ràng buộc, cách sử dụng, cách xây dựng, cách tiến hóa).
* Phải **tham gia vào toàn bộ vòng đời API**.
* Làm việc với các nhà thiết kế API khác để đảm bảo **tính nhất quán** của các API trong tổ chức.

## Tóm tắt (Summary)
* Web API biến phần mềm thành các khối có thể tái sử dụng và có thể sử dụng qua mạng với giao thức HTTP.
* API là giao diện cho các nhà phát triển, những người xây dựng các ứng dụng sử dụng chúng.
* Thiết kế API quan trọng đối với tất cả các API—công khai hoặc riêng tư.
* API được thiết kế kém có thể bị sử dụng ít, bị lạm dụng hoặc không được sử dụng gì cả, và thậm chí không an toàn.
* Thiết kế một API tốt yêu cầu bạn xét đến toàn bộ ngữ cảnh của ứng dụng, không chỉ riêng giao diện.
