**Chương 1: Thiết kế API là gì? (What is API design?)**

Chương này khám phá API là gì, tại sao việc thiết kế nó lại quan trọng, và ý nghĩa của việc thiết kế API. Các API ứng dụng web (Web API) là nền tảng thiết yếu của thế giới kết nối hiện nay, giúp phần mềm giao tiếp với nhau, từ ứng dụng điện thoại đến các máy chủ backend ẩn sâu. Dù hệ thống dựa trên API có quy mô hay mục đích thế nào, việc thiết kế API luôn phải là mối quan tâm chính, vì sự thành công hay thất bại của hệ thống phụ thuộc trực tiếp vào chất lượng thiết kế của tất cả các API của nó.

**1.1 API là gì? (What is an API?)**

Trong cuốn sách này, **API được hiểu là một API từ xa (remote API), cụ thể hơn là một web API – một giao diện web cho phần mềm**. API trước hết là một giao diện: điểm gặp gỡ và tương tác giữa hai hệ thống, chủ thể, tổ chức, v.v..

*   **1.1.1 API là giao diện web cho phần mềm (An API is a web interface for software)**
    API là giao diện lập trình cho ứng dụng. Giống như giao diện người dùng (UI) của ứng dụng điện thoại cung cấp các nút, trường văn bản để con người tương tác với ứng dụng, API cung cấp các chức năng có thể cần dữ liệu đầu vào hoặc trả về dữ liệu đầu ra, cho phép các ứng dụng khác tương tác với ứng dụng cung cấp API để truy xuất, gửi thông tin hoặc kích hoạt hành động. **API chỉ là giao diện do phần mềm cung cấp, một lớp trừu tượng của việc triển khai cơ bản** (mã nguồn thực sự hoạt động bên trong). Web API là các API từ xa có thể sử dụng giao thức HTTP, cùng giao thức mà trình duyệt web sử dụng.

    *   **Ví dụ:** Khi một ứng dụng mạng xã hội trên điện thoại chia sẻ ảnh, nó sử dụng API của camera điện thoại, API của thư viện xử lý ảnh nhúng trong ứng dụng, và API từ xa (web API) của máy chủ mạng xã hội thông qua mạng Internet. Ứng dụng điện thoại là người dùng (consumer), và ứng dụng máy chủ (backend) là nhà cung cấp (provider).

*   **1.1.2 API biến phần mềm thành các khối LEGO® (APIs turn software into LEGO® bricks)**
    Web API biến phần mềm thành các khối có thể tái sử dụng và dễ dàng lắp ráp, giải phóng sự sáng tạo và đổi mới. Điều này cho phép tạo ra các hệ thống mô-đun có thể làm bất cứ điều gì. Trong thế giới API, có hai loại khối phơi bày hai loại API: API công khai (public API) và API riêng tư (private API).
    *   **API công khai** được bên thứ ba cung cấp như một dịch vụ hoặc sản phẩm; bạn chỉ sử dụng chúng. Chúng cung cấp cho bất kỳ ai cần và chấp nhận điều khoản. Các API công khai có thể miễn phí hoặc tính phí.
    *   **API riêng tư** thường dành cho mục đích nội bộ của tổ chức.
    Sự khác biệt công khai/riêng tư nằm ở **đối tượng người dùng (to whom)** chứ không phải cách API được phơi bày (ví dụ: trên Internet). **API đối tác (partner API)** là các API gần như công khai, được phơi bày cho khách hàng hoặc đối tác chọn lọc. Dù là loại nào, API về cơ bản biến phần mềm thành các khối có thể lắp ráp để tạo hệ thống mô-đun.

**1.2 Tại sao thiết kế API lại quan trọng (Why API design matters)**

API được sử dụng bởi phần mềm, nhưng người xây dựng phần mềm đó là các nhà phát triển – con người. Những người này mong đợi các giao diện lập trình hữu ích và đơn giản, giống như bất kỳ giao diện nào được thiết kế tốt khác. Một API được thiết kế kém có thể gây khó chịu, thậm chí nguy hiểm, giống như một vật dụng hàng ngày được thiết kế kém.

*   **1.2.1 API công khai hoặc riêng tư là giao diện cho các nhà phát triển khác (A public or private API is an interface for other developers)**
    Dù công khai hay riêng tư, API sớm muộn cũng sẽ được sử dụng bởi các nhà phát triển khác—những người không tham gia vào việc tạo ra phần mềm cung cấp API. Do đó, phải làm mọi thứ để tạo điều kiện thuận lợi cho những người mới này khi viết mã để sử dụng API. **Trải nghiệm của nhà phát triển (Developer experience - DX)** là trải nghiệm mà nhà phát triển có khi sử dụng API. Mọi nỗ lực trong các khía cạnh khác của DX đều vô nghĩa nếu khía cạnh quan trọng nhất – thiết kế API – không được xử lý đúng đắn.

*   **1.2.2 API được tạo ra để ẩn đi chi tiết triển khai (An API is made to hide the implementation)**
    Thiết kế API quan trọng vì người dùng API muốn sử dụng nó mà không bị làm phiền bởi các chi tiết vụn vặt không liên quan đến họ. Để làm được điều đó, thiết kế của API phải **che giấu chi tiết triển khai** (những gì thực sự xảy ra bên trong).

    *   **Ví dụ:** Khi đi ăn nhà hàng, bạn là khách hàng (consumer) và tương tác với người phục vụ (API) để gọi món. Bạn không cần biết món ăn được nấu thế nào trong bếp (implementation). Tương tự, nhà phát triển sử dụng API chỉ cần biết cách tương tác với giao diện để đạt được mục tiêu, không cần biết phần mềm nhà cung cấp (provider software) sẽ thực hiện điều đó thế nào.

*   **1.2.3 Hậu quả khủng khiếp của việc thiết kế API kém (The terrible consequences of poorly designed APIs)**
    Giống như một vật dụng hàng ngày khó sử dụng, API được thiết kế kém có thể là một nỗi đau thực sự để hiểu và sử dụng.

    *   **Ví dụ:** Thiết bị UDRC 1138 có giao diện khó hiểu, tên và các nút không rõ ràng, thông báo cảnh báo khó hiểu. Ngay cả tài liệu cũng khó giải mã. Thiết kế kém này làm nó khó sử dụng và không an toàn.

    Đối với API công khai (API as a product), người dùng tiềm năng đánh giá API dựa trên giao diện và tài liệu của nó. Nếu họ phát hiện lỗi thiết kế khiến API khó hoặc nguy hiểm khi sử dụng, họ sẽ không chọn API đó, dẫn đến không có khách hàng, không có doanh thu và có thể khiến công ty phá sản. Ngay cả với API riêng tư, lỗi thiết kế làm tăng thời gian, công sức và tiền bạc để xây dựng phần mềm sử dụng API, có thể dẫn đến việc API bị lạm dụng hoặc ít được sử dụng, tăng chi phí hỗ trợ cho nhà cung cấp. Thiết kế API lỗi còn có thể dẫn đến **lỗ hổng bảo mật**. Dù đôi khi có thể sửa thiết kế kém sau khi API được đưa vào sản xuất, việc này tốn kém thời gian và tiền bạc cho nhà cung cấp, và có thể làm phiền người dùng API. API được thiết kế kém chắc chắn sẽ thất bại.

**1.3 Các yếu tố của thiết kế API (The elements of API design)**

Học thiết kế API không chỉ đơn thuần là học thiết kế giao diện lập trình. Nó yêu cầu học các nguyên tắc vượt ra ngoài công nghệ và biết tất cả các khía cạnh của thiết kế API. Thiết kế API yêu cầu tập trung vào chính giao diện, nhưng cũng phải biết toàn bộ ngữ cảnh xung quanh nó và thể hiện sự đồng cảm với tất cả người dùng và phần mềm liên quan. Thiết kế API mà không có nguyên tắc, hoàn toàn ngoài ngữ cảnh và không xét đến cả hai phía giao diện (người dùng và nhà cung cấp) là cách tốt nhất để đảm bảo thất bại hoàn toàn.

*   **1.3.1 Học các nguyên tắc vượt ra ngoài thiết kế giao diện lập trình (Learning the principles beyond programming interface design)**
    Có nhiều cách khác nhau để phơi bày dữ liệu và khả năng thông qua phần mềm (RPC, SOAP, REST, gRPC, GraphQL). Mỗi kiểu API (API style) có thể đi kèm với các thực tiễn phổ biến, nhưng chúng không ngăn bạn mắc lỗi. **Nếu không biết các nguyên tắc cơ bản, bạn có thể lạc lõng khi chọn một thực tiễn "phổ biến"**, gặp khó khăn khi đối mặt với các trường hợp sử dụng bất thường hoặc ngữ cảnh không được bao phủ bởi thực tiễn phổ biến. **Học các nguyên tắc cơ bản của thiết kế API cung cấp nền tảng vững chắc** để thiết kế API của bất kỳ kiểu nào và đối mặt với mọi thách thức thiết kế.

*   **1.3.2 Khám phá tất cả các khía cạnh của thiết kế API (Exploring all facets of API design)**
    Thiết kế API không chỉ là thiết kế giao diện dễ hiểu và dễ sử dụng. Chúng ta phải thiết kế **giao diện hoàn toàn an toàn**. Chúng ta phải **xét đến toàn bộ ngữ cảnh** (các ràng buộc, API sẽ được sử dụng thế nào và bởi ai, cách API được xây dựng, và cách nó có thể tiến hóa). Chúng ta phải **tham gia vào toàn bộ vòng đời API** (từ thảo luận ban đầu đến phát triển, tài liệu, tiến hóa hoặc ngừng cung cấp, v.v.). Khi các tổ chức thường xây dựng nhiều API, chúng ta nên làm việc cùng nhau với tất cả các nhà thiết kế API khác để đảm bảo tất cả API của tổ chức có diện mạo và cảm nhận tương tự nhằm xây dựng các API nhất quán nhất có thể.

**Tóm tắt (Summary)**

*   Web API biến phần mềm thành các khối có thể tái sử dụng và có thể sử dụng qua mạng với giao thức HTTP.
*   API là giao diện cho các nhà phát triển, những người xây dựng các ứng dụng sử dụng chúng.
*   Thiết kế API quan trọng đối với tất cả các API—công khai hoặc riêng tư.
*   API được thiết kế kém có thể bị sử dụng ít, bị lạm dụng hoặc không được sử dụng gì cả, và thậm chí không an toàn.
*   Thiết kế một API tốt yêu cầu bạn xét đến toàn bộ ngữ cảnh của ứng dụng, không chỉ riêng giao diện.