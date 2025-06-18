``Web API Design: The Missing Link``
Cuốn sách này được viết bởi Apigee, một nhà lãnh đạo trong lĩnh vực quản lý API. Mục tiêu chính là cung cấp ``Các Thực Tiễn Tốt Nhất để Xây Dựng Giao Diện mà Các Nhà Phát Triển Yêu Thích``. Cuốn sách phản ánh những kinh nghiệm của Apigee, khách hàng của họ và ngành công nghiệp nói chung, đúc kết những đổi mới trong thiết kế API mang lại lợi ích thực sự và trở thành xu hướng nổi bật. Đây không phải là một hướng dẫn API dành cho người mới bắt đầu, mà là sự phát triển của tư duy về API gần đây hơn.

## 1. Web APIs và REST
### 1.1. Công việc của Nhà Thiết Kế APIs
*   ``Nhiệm vụ của API là làm cho nhà phát triển ứng dụng thành công nhất có thể``.
*   Khi xây dựng API, bạn nên ``suy nghĩ về các lựa chọn thiết kế từ góc nhìn của nhà phát triển ứng dụng``.
*   ``Nhà phát triển ứng dụng là yếu tố then chốt`` của toàn bộ chiến lược API.
*   Nguyên tắc thiết kế chính khi tạo API của bạn là ``tối đa hóa năng suất và thành công của nhà phát triển ứng dụng``.
*   Thiết kế đúng rất quan trọng vì ``thiết kế truyền đạt cách một cái gì đó sẽ được sử dụng``.

### 1.2. Web API là gì?
*   Một Web API là ``mô hình các yêu cầu và phản hồi HTTP`` được sử dụng để truy cập một trang web được chuyên biệt hóa để truy cập bởi các chương trình máy tính tùy ý, thay vì (hoặc cùng với) các trình duyệt web do con người sử dụng.

### 1.3. REST là gì?
*   REST là tên được đặt cho ``phong cách kiến trúc của chính HTTP``, được mô tả bởi một trong những tác giả hàng đầu của các đặc tả HTTP.
*   ``HTTP là thực tế – REST là một tập hợp các ý tưởng thiết kế đã định hình nó``.
*   Từ góc độ thực tế, chúng ta có thể tập trung sự chú ý vào HTTP và cách chúng ta sử dụng nó để phát triển API.
*   Tầm quan trọng của REST là nó giúp chúng ta hiểu cách suy nghĩ về HTTP và cách sử dụng nó.
*   Hầu hết các API hiện đại sử dụng một tập hợp con các khái niệm từ HTTP, pha trộn với một số khái niệm từ các công nghệ máy tính khác (ví dụ: "endpoints" và "parameters" không phải là khái niệm gốc của HTTP hoặc REST, mà được chuyển từ Remote Procedure Call (RPC)).
*   Thuật ngữ ``RESTful APIs`` đã xuất hiện cho các Web API sử dụng nhiều khái niệm và kỹ thuật gốc của HTTP hơn so với các công nghệ trước đây, nhưng cũng pha trộn các khái niệm khác. Phong cách pha trộn này có lẽ là phong cách API phổ biến nhất đang được sử dụng.
*   Apigee ủng hộ rằng bạn nên ``chỉ sử dụng HTTP mà không thêm các khái niệm bổ sung`` nhiều nhất có thể.
    *   ``Tại sao?`` Khi thiết kế bất kỳ giao diện nào, bạn nên đặt mình vào vị trí của người dùng. Người dùng API của bạn có thể đã có kiến thức đáng kể về các công nghệ và tiêu chuẩn HTTP cơ bản, cũng như các API khác.
    *   Có rất nhiều giá trị trong việc ``tuân thủ các tiêu chuẩn và quy ước đã được thiết lập``, thay vì tự mình phát minh ra. Các đặc tả HTTP là một trong những tiêu chuẩn được viết tốt nhất, thiết kế tốt nhất và được chấp nhận rộng rãi nhất trong ngành.
    *   ``Ví dụ về việc tuân thủ chuẩn HTTP``:
        *   Nếu API của bạn sử dụng ``POST để tạo một tài nguyên``, hãy đảm bảo bao gồm ``header `Location``` trong phản hồi chứa URL của tài nguyên mới tạo, cùng với ``mã trạng thái `201 Created```.
        *   Nếu cần kiểm tra hai người không cập nhật cùng một tài nguyên web đồng thời, hãy sử dụng ``header `ETag` và `If-Match```.
        *   Nếu API của bạn cho phép người dùng yêu cầu dữ liệu ở các định dạng khác nhau, hãy sử dụng ``header `HTTP Accept```.
    *   Nếu bạn muốn cung cấp các lựa chọn thay thế cho cơ chế tiêu chuẩn, hãy làm điều đó ``ngoài việc hỗ trợ các cơ chế tiêu chuẩn, không phải thay thế``.
*   ``Chủ nghĩa tối giản về khái niệm``: Hạn chế bản thân chỉ sử dụng các khái niệm HTTP cho Web API là để giữ sự đơn giản. "Sự hoàn hảo đạt được không phải khi không còn gì để thêm, mà là khi không còn gì để bỏ đi".
*   ``Giảm thiểu sự gắn kết (coupling)``: Một trong những đặc điểm quan trọng nhất mà các nhà thiết kế API phân tán hướng tới là giảm thiểu sự gắn kết giữa client và server. Các khái niệm lớp từ các mô hình khác (ví dụ: RPC) có xu hướng làm suy yếu chất lượng gắn kết lỏng lẻo của thiết kế HTTP.

## 2. HTTP và REST: Một mô hình thiết kế hướng dữ liệu
*   Một REST API tập trung vào ``các thực thể cơ bản của miền vấn đề`` mà nó phơi bày, hơn là một tập hợp các hàm thao tác các thực thể đó.
    *   Ví dụ: Trong hệ thống theo dõi chó và chủ, các thực thể chính là ``tập hợp các con chó đã biết`` (`https://dogtracker.com/dogs`) và ``các con chó riêng lẻ``.
*   ``Tính đồng nhất (Uniformity)``: Một phần đáng kể giá trị của việc thiết kế API dựa trên HTTP và REST đến từ tính đồng nhất mà nó mang lại. Về bản chất, khi bạn sử dụng HTTP một cách tự nhiên, bạn không cần phải phát minh ra API - HTTP cung cấp API, và bạn chỉ cần định nghĩa dữ liệu trong tài nguyên của mình.
*   Trong REST, ý tưởng này được gọi là ``ràng buộc giao diện đồng nhất (uniform interface constraint)``. Điều này cho phép triển khai các phần mềm phổ quát, như trình duyệt web và bot tìm kiếm, hoạt động với bất kỳ trang web nào.
*   ``So sánh với API hướng chức năng``: Trong API hướng chức năng, có nhiều sự thay đổi và chi tiết bạn phải học, không có cấu trúc hoặc mô hình rõ ràng để giúp bạn học, và kiến thức bạn có được sẽ không giúp ích gì cho API tiếp theo bạn cần học.

## 3. Các Yếu Tố Thiết Kế API
Các khía cạnh sau đây đều quan trọng và cùng nhau định nghĩa API của bạn:
*   Các biểu diễn của tài nguyên của bạn – bao gồm định nghĩa các trường trong tài nguyên và các liên kết đến các tài nguyên liên quan.
*   Việc sử dụng các header HTTP tiêu chuẩn (và đôi khi tùy chỉnh).
*   Các URL và URI template định nghĩa giao diện truy vấn của API để định vị tài nguyên dựa trên dữ liệu của chúng.
*   Các hành vi bắt buộc của client – ví dụ: hành vi caching DNS, hành vi thử lại, khả năng chấp nhận các trường không có trước đây trong tài nguyên, v.v..

## 4. Thiết Kế Biểu Diễn (Representations)
*   Trong mô hình hướng dữ liệu như REST, tốt hơn nên bắt đầu với ``thiết kế biểu diễn`` trước khi thiết kế URL.
*   ``Biểu diễn (Representation)`` là thuật ngữ kỹ thuật cho dữ liệu được trả về khi một tài nguyên web được client truy xuất từ server, hoặc được gửi từ client đến server. Nó là một biểu diễn trạng thái cơ bản của tài nguyên.
*   Người dùng có thể xem biểu diễn của một tài nguyên ở các định dạng khác nhau, được gọi là ``kiểu phương tiện (media types)``. Về nguyên tắc, tất cả các kiểu phương tiện cho biểu diễn của một tài nguyên cụ thể nên mã hóa cùng một thông tin, chỉ ở các định dạng khác nhau.

### 4.1. Sử dụng JSON
*   ``JSON (JavaScript Object Notation)`` là kiểu phương tiện chiếm ưu thế cho các biểu diễn tài nguyên trong các Web API.
*   ``Lý do thành công``: Đơn giản để hiểu, dễ ánh xạ tới các cấu trúc dữ liệu lập trình (JavaScript, Python, Ruby, Java).
*   ``Tiêu chuẩn thực tế (de facto standard)`` cho Web API, và bạn nên sử dụng nó.
*   ``Hạn chế của JSON``: Chỉ có thể biểu diễn một số ít kiểu dữ liệu (Null, Boolean, Number, String). Các kiểu phổ biến không được hỗ trợ là ngày và giờ, và URL. Cách đơn giản nhất để xử lý là biểu diễn chúng dưới dạng chuỗi và dựa vào ngữ cảnh.
*   ``Giữ JSON đơn giản``: Khi sử dụng tốt, JSON đơn giản, trực quan và phần lớn tự giải thích.

### 4.2. Biểu diễn các mối quan hệ trong liên kết (links)
*   Cấu trúc tự nhiên để biểu diễn các mối quan hệ trong HTTP là ``liên kết (link)``.
*   Việc sử dụng liên kết trong API gắn liền với ý tưởng ``Hypermedia As The Engine Of Application State (HATEOAS)``, trong đó hành vi của client hoàn toàn dựa trên nội dung dữ liệu được trả về từ server, đặc biệt là các liên kết.
*   Điều này ``cải thiện khả năng sử dụng và học hỏi của tất cả các API``.
*   ``Ví dụ về cách biểu diễn mối quan hệ``:
    *   ``Không có liên kết rõ ràng``: `{ "ownerID": "98765432" }`. Nhược điểm: Phải dò tìm tài liệu để tìm URI template, phải viết code để kết hợp ID với template để tạo URL.
    *   ``Với liên kết rõ ràng``: Thêm các thuộc tính giá trị URL như `ownerLink` và `dogsLink` vào dữ liệu (ví dụ: `"ownerLink": "https://dogtracker.com/persons/98765432"`, `"dogsLink": "https://dogtracker.com/persons/98765432/dogs"`). Ưu điểm: Ít phải học hơn, URL có sẵn trong dữ liệu, code client dễ viết hơn, có thể sử dụng code đa năng để theo dõi các liên kết mà không cần kiến thức cụ thể về API.
*   Ngày càng nhiều API từ các công ty lớn (Google, GitHub) bao gồm các liên kết trong dữ liệu của họ.

### 4.3. URI templates có còn cần thiết khi có links?
*   ``Có, chúng vẫn quan trọng``.
*   ``Liên kết (Links)``: Diễn tả nơi bạn có thể đi từ vị trí hiện tại của bạn. Chúng là các biển báo cho các con đường đã được lát để đi qua hệ thống.
*   ``URI templates``: Giống như các lối tắt, một cách để đi thẳng đến một đích cụ thể. Chúng cung cấp một cách để định vị tài nguyên khi không có liên kết trực tiếp hoặc đường dẫn liên kết quá dài.
*   URI templates hữu ích nhất nếu chúng chấp nhận các biến có giá trị dễ đọc đối với con người.
*   URI templates được coi là ``ngôn ngữ truy vấn chuyên biệt`` cho API của bạn, cho phép định vị tài nguyên mà bạn không biết URL đầy đủ.
*   Liên kết và URI templates là ``hai kỹ thuật bổ sung có giá trị`` cho nhau, giống như trên World Wide Web.

### 4.4. URL tuyệt đối so với tương đối trong biểu diễn
*   Lưu trữ URL tuyệt đối trong cơ sở dữ liệu có thể gây vấn đề trong các môi trường khác nhau (kiểm thử tích hợp, tiền sản xuất). Server nên loại bỏ scheme và authority từ các URL tự định danh trước khi lưu trữ và khôi phục chúng khi được yêu cầu.
*   Chấp nhận và tạo ``URL tương đối`` (đặc biệt là các URL tuyệt đối đường dẫn, bắt đầu bằng `/`) trong API là một cách tiếp cận hợp lý để tránh các vấn đề triển khai trên server.
*   ``Đánh đổi``: Đẩy gánh nặng cho client vì client sẽ phải chuyển đổi các URL tương đối thành tuyệt đối trước khi sử dụng chúng. Mặc dù client có thể làm điều này bằng cách sử dụng kiến thức chung về tiêu chuẩn URL, việc thiết lập URL cơ sở có thể phức tạp.
*   Cung cấp URL tuyệt đối trên server sẽ tạo ra một API tốt hơn cho người dùng.

### 4.5. Cách biểu diễn liên kết trong tài nguyên
*   Cách tiếp cận ưa thích là sử dụng một ``cặp tên/giá trị JSON đơn giản``, ví dụ: `"ownerLink": "https://dogtracker.com/persons/98765432"`. Cách này đơn giản và nhất quán.
*   Có nhiều mẫu phức tạp hơn để biểu diễn liên kết, nhưng việc sử dụng liên kết không yêu cầu điều gì phức tạp hơn các thuộc tính JSON đơn giản.
*   Ví dụ từ GitHub cho thấy ``URI Templates trong các giá trị liên kết``, ví dụ: `"events_url": "https://api.github.com/users/octocat/events{/privacy}"`. Điều này cho phép truyền tải các URL của cả một "họ" tài nguyên trong một chuỗi, nhưng đòi hỏi client phải xử lý template.

## 5. Thiết Kế URL
### 5.1. Danh từ là tốt; động từ là xấu
*   Hậu quả của mô hình hướng dữ liệu của REST là ``mỗi URL xác định một "thứ" (thing)``.
*   Do đó, các URL nên được hình thành từ ``danh từ``, là phần của lời nói trong ngôn ngữ tự nhiên được sử dụng để chỉ định mọi thứ.

### 5.2. Các URL đã biết (Well-known URLs)
*   Trong một API, luôn cần phải công bố ít nhất một URL đã biết để client có thể bắt đầu (ví dụ: `https://dogtracker.com/dogs`, `https://dogtracker.com/owners`).

### 5.3. Thiết kế URL thực thể (Entity URLs)
*   ``Không cần thiết phải định nghĩa bất kỳ định dạng nào`` cho URL của một thực thể riêng lẻ cho client.
*   Trong một Web API được thiết kế tốt, ``client không nên phải xây dựng URL của các thực thể riêng lẻ từ các phần nhỏ``.
*   URL của tài nguyên mới được tạo sẽ được trả về cho client.
*   Các URL mờ (opaque URLs) như `https://dogtracker.com/ZG9n;a8098c1a` không gây khó khăn cho máy tính, nhưng con người thấy khó.
*   Thông thường và mong muốn hơn là định nghĩa ``URL thực thể thân thiện với con người``, ví dụ: `https://dogtracker.com/dogs/a8098c1a`. URL này vẫn có thể được client sử dụng như một URL mờ.
*   Google Docs và Microsoft OneDrive sử dụng các URL có chuỗi ký tự ngẫu nhiên. Tuy nhiên, không phải tất cả các URL đều phải không thân thiện với con người (ví dụ: URL truy vấn có thể thân thiện).
*   Dấu gạch chéo (`/`) thường được sử dụng để tách kiểu khỏi ID tài nguyên (ví dụ: `https://dogtracker.com/{type}/{uuid}`).

### 5.4. Web là phẳng (The web is flat)
*   ``Tránh mã hóa các hệ thống phân cấp sâu trong URL liên kết``.
*   Đối với ``URL truy vấn``, việc mã hóa phân cấp có thể là một ý tưởng hay.
*   Tuy nhiên, đối với ``URL liên kết`` (được server lưu trữ và client đánh dấu trang), việc mã hóa phân cấp là một ``ý tưởng tồi``. Các hệ thống phân cấp không ổn định như vẻ ngoài, và việc mã hóa chúng trong URL của bạn sẽ ngăn cản việc tổ chức lại hoặc làm hỏng các liên kết vĩnh viễn khi bạn làm vậy.

### 5.5. Giải pháp cho tình huống tiến thoái lưỡng nan về đổi tên
*   ``Vấn đề``: Sử dụng ID thân thiện với con người (tên) trong URL sẽ thân thiện với người dùng nhưng sẽ làm hỏng tất cả các tham chiếu nếu tài nguyên được đổi tên. Sử dụng ID do máy tạo (UUID) cho phép đổi tên tự do nhưng tạo ra URL không thân thiện với con người.
*   ``Giải pháp``: Sử dụng một URL cho ``liên kết`` (với ID do máy tạo, mờ đối với client) và một URL khác trong ``URI templates cho truy vấn`` (với ID thân thiện với con người).
    *   Ví dụ: `https://dogtracker.com/person/{name}` (ví dụ: `JoeMCarraclough`) rất hữu ích để tìm kiếm Joe và là một URL hấp dẫn để đưa vào API của chúng ta. Tuy nhiên, chúng ta không muốn sử dụng URL này để liên kết vì tên có thể thay đổi.
    *   Các URL liên kết nên ``mờ đối với client``; server sử dụng chúng để định tuyến.

## 6. Thiết Kế URL Truy Vấn (Query URLs)
*   Chỉ sử dụng HTTP giúp API tốt hơn bằng cách giảm lượng thông tin độc đáo phải học. Giống như một hệ thống quản lý cơ sở dữ liệu (DBMS) – một khi bạn đã học API của một DBMS, bạn sử dụng kiến thức đó lặp đi lặp lại cho bất kỳ cơ sở dữ liệu nào được DBMS đó lưu trữ.
*   Khi các nhà thiết kế API định nghĩa URI templates cho API của họ, họ thực sự đang thiết kế một ``ngôn ngữ truy vấn tùy chỉnh`` cho API.
*   ``Mục tiêu``: Thiết kế các URL truy vấn có tính đều đặn và dễ đoán.

### 6.1. Mô hình chung cho URL truy vấn
*   ``Mẫu ưa thích cho việc duyệt mối quan hệ``: `https://dogtracker.com/persons/{personId}/dogs`.
    *   Thường được ưu tiên hơn `https://dogtracker.com/search?type=Dog&owner={personId}`.
    *   ``Lý do``: Dễ đọc và trực quan hơn cho nhà phát triển, dễ triển khai hơn, không ngụ ý cam kết với một ngôn ngữ truy vấn toàn diện.
    *   Rất hữu ích để diễn tả các chuỗi duyệt mối quan hệ (ví dụ: `/dogs/123456/owner/spouse`).
*   ``Biểu diễn mối quan hệ trong URL truy vấn``:
    *   Sử dụng các đoạn đường dẫn (path segments) cho các mối quan hệ (ví dụ: `GET /persons/5678/dogs` để lấy tất cả chó thuộc về một người cụ thể).
    *   Mô hình chung: `/{relationship-name}[/{resource-id}]/.../{relationship-name}[/{resource-id}]`.
    *   `POST` tới URL này có thể là cách viết tắt để tạo một tài nguyên và thiết lập mối quan hệ.
*   ``Biểu diễn mối quan hệ đối xứng trong URL và biểu diễn``:
    *   Nếu URL truy vấn ngụ ý một mối quan hệ (ví dụ: người và chó), hãy làm cho mối quan hệ này rõ ràng trong biểu diễn tài nguyên bằng cách cung cấp một thuộc tính mối quan hệ có giá trị URL (liên kết).
    *   Ngược lại, nếu một thuộc tính mối quan hệ có trong tài nguyên, hãy cung cấp một URL truy vấn cho phép duyệt nó mà không cần truy xuất tài nguyên đó.
    *   Ví dụ: Tài nguyên gốc (`/`) nên có các mối quan hệ `persons` và `dogs` trong biểu diễn của nó.
*   Mô hình này thống nhất các URL truy vấn với mô hình gốc duy nhất có liên kết của World Wide Web, giúp API dễ học hơn bằng cách thống nhất mô hình URL truy vấn và mô hình dữ liệu.
*   Giá trị của mô hình này là làm cho URL truy vấn dễ đoán cho bất kỳ lập trình viên client nào đã hiểu mô hình dữ liệu của API. Việc áp dụng một mô hình nhất quán tốt hơn nhiều so với việc thiết kế từng URL truy vấn một cách riêng lẻ.

### 6.2. Path parameters, hoặc matrix parameters
*   Một cú pháp thay thế: `/persons;5678/dogs`.
*   Quy tắc tổng quát: `/{relationship-name}[;{selector}]/.../{relationship-name}[;{selector}]`.
*   Ưu điểm: Sự gọn gàng về cú pháp và sự rõ ràng về mặt khái niệm (mỗi phân đoạn đường dẫn đại diện chính xác một lần duyệt mối quan hệ).

### 6.3. Lọc các tập hợp (Filtering collections)
*   Theo truyền thống, các mệnh đề truy vấn phức tạp hơn để lọc các tập hợp được đặt sau dấu `?` trong phần chuỗi truy vấn của URL (ví dụ: `/dogs?color=red&state=running&location=park`).
*   Việc sử dụng các URL truy vấn dựa trên đường dẫn và tham số chuỗi truy vấn có thể được kết hợp.

## 7. Các phản hồi không liên quan đến tài nguyên bền vững
*   Nguyên tắc là: ``URL nên xác định tài nguyên trong phản hồi, không phải thuật toán xử lý`` tính toán phản hồi đó.
*   Vì tài nguyên có biểu diễn trong phản hồi là một "thứ", nên dễ dàng tạo một URL cho nó phù hợp với mô hình thực thể dựa trên danh từ của web.
*   Từ góc nhìn của client, việc kết quả được tính toán hay truy xuất từ cơ sở dữ liệu là không liên quan.

## 8. Bao gồm thuộc tính `kind`
*   Thuộc tính `kind` giúp client nhận ra liệu đây có phải là một đối tượng mà họ biết cách xử lý hay không.
*   Điều này giúp bạn thêm các khái niệm mới vào API mà không cần thay đổi phiên bản API.
*   Sử dụng URL cho các kiểu (ví dụ: `https://apigee.com/collections#Collection`) giúp tránh xung đột tên.

## 9. Cách biểu diễn các tập hợp (Collections)
*   Các tập hợp là tài nguyên rất quan trọng vì chúng cung cấp danh sách các thứ và cũng là những thứ mà chúng ta POST để tạo tài nguyên mới.
*   Các kiểu phương tiện được chuẩn hóa bằng cách đăng ký chúng ở một nơi trung tâm; các kiểu tài nguyên có thể sử dụng URL.

## 10. PATCH so với PUT để cập nhật
*   ``Sử dụng PATCH để cập nhật`` vì nó giúp ích cho sự phát triển của API.
*   ``PUT yêu cầu client thay thế toàn bộ nội dung`` của tài nguyên trong mỗi lần cập nhật, điều này dễ vỡ vì nó dựa vào client phải duy trì tất cả các thuộc tính.
*   ``PATCH tránh được rủi ro này``. (Lưu ý: Thao tác `UPDATE` của SQL có ngữ nghĩa PATCH, không phải PUT.)

## 11. API "chatty" (nhiều cuộc gọi nhỏ)
*   ``Quan niệm sai lầm phổ biến``: Myth cho rằng REST API là "chatty" (tốn nhiều cuộc gọi nhỏ). API "chatty" là do bạn đã thiết kế sai tài nguyên cho client, chứ không phải do bạn sử dụng REST.
*   Quan niệm sai lầm này có thể đến từ việc hiểu sai rằng một REST API phải giống như một lược đồ cơ sở dữ liệu quan hệ được chuẩn hóa hoàn toàn.
*   ``Chiến lược hữu ích``: Định nghĩa tất cả các tài nguyên liên quan đến lược đồ chuẩn hóa (có khả năng đọc và cập nhật) và sau đó ``thêm nhiều tài nguyên phi chuẩn hóa (denormalized resources) chỉ đọc`` khi cần để hỗ trợ các client hiệu quả.
*   Các tài nguyên phi chuẩn hóa cũng là tài nguyên REST thực sự và sẽ hữu ích hơn nhiều so với một API chỉ chuẩn hóa.

## 12. Phân trang và phản hồi một phần (Pagination and partial response)
*   Phản hồi một phần cho phép bạn cung cấp cho nhà phát triển ứng dụng ``chỉ thông tin họ cần``.
*   Ví dụ: Yêu cầu tweet trên Twitter API có thể trả về nhiều hơn thông tin cần thiết.
*   Apigee thích `limit` và `offset` cho phân trang.

## 13. Xử Lý Lỗi (Handling Errors)
*   Chất lượng xử lý lỗi là một ``phần quan trọng của trải nghiệm API`` cho tất cả các client.
*   ``Tại sao thiết kế lỗi tốt lại quan trọng?``
    *   Từ góc nhìn của nhà phát triển ứng dụng tiêu thụ API, mọi thứ ở phía bên kia giao diện là một hộp đen. Do đó, ``lỗi trở thành công cụ quan trọng`` cung cấp ngữ cảnh và khả năng hiển thị về cách sử dụng API.
    *   Các nhà phát triển học cách viết code thông qua lỗi.
    *   Các nhà phát triển phụ thuộc vào các lỗi được thiết kế tốt khi khắc phục sự cố và giải quyết các vấn đề.
*   ``Lời khuyên``: Học và sử dụng ``các mã trạng thái HTTP tiêu chuẩn một cách thích hợp``.
*   Các mã trạng thái là một phần của ``thông điệp phản hồi HTTP hoàn chỉnh``, bao gồm mã trạng thái, header và có thể cả dữ liệu.
*   ``Ví dụ về các mẫu phản hồi tiêu chuẩn``:
    *   ```201 Created```: Luôn đi kèm với ``header `Location``` chứa URL của tài nguyên mới tạo.
    *   ```200 OK```: Đối với phản hồi thành công, có thể đi kèm với `Content-Location`, `Content-Type`, `ETag`, `Content-Length`.
    *   ```405 Method Not Allowed```: Luôn đi kèm với ``header `Allow``` liệt kê các phương thức được hỗ trợ cho tài nguyên.
*   ``Thông điệp trong phần thân phản hồi (payload)``: Nên ``dài dòng (verbose) và sử dụng ngôn ngữ đơn giản``.
    *   Bao gồm:
        *   `developerMessage`: Mô tả chi tiết, ngôn ngữ đơn giản về vấn đề cho nhà phát triển ứng dụng với các gợi ý về cách khắc phục.
        *   `userMessage`: Thông điệp để chuyển đến người dùng cuối ứng dụng nếu cần.
        *   `errorCode`: Mã lỗi số.
        *   `more info`: Liên kết đến tài liệu chi tiết hơn.
    *   Đối tượng thực sự của thông điệp này là ``user agent`` (chương trình trung gian).

## 14. Mô hình hóa Hành Động (Modeling Actions)
* Để kích hoạt một hành động (bắt đầu, dừng, tạm dừng một tiến trình) mà vẫn tuân thủ mô hình hướng thực thể, dựa trên danh từ của web:
*   ``Tùy chọn 1``: Cung cấp một ``thuộc tính trạng thái (state property)`` của tiến trình và cho phép client đặt nó thành `started`, `stopped`, hoặc `paused`.
*   ``Tùy chọn 2``: Cho phép client ``POST một yêu cầu hành động (action request) đến một URL liên quan``.
    *   Ví dụ: Tài nguyên tiến trình có thuộc tính `actionRequests` là một URL (ví dụ: `https://example.org/processes/123456/requests`).
    *   Client ``POST`` một thực thể `StopRequest`, `StartRequest`, `PauseRequest`, hoặc `ResumeRequest` đến URL này.
*   ``Tùy chọn 3 (biến thể của 2)``: Cung cấp ``các thuộc tính riêng biệt cho từng loại yêu cầu``.
    *   Sự hiện diện hay vắng mặt của thuộc tính truyền đạt tính hợp lệ của một yêu cầu cụ thể tại một thời điểm cụ thể.
    *   Ví dụ: Khi tiến trình đang chạy, có `pauseRequests` và `stopRequests`; khi tạm dừng, có `resumeRequests` và `stopRequests`.

## 15. Xác Thực (Authentication)
*   ``OAuth2`` là tiêu chuẩn được sử dụng bởi hầu hết mọi Web API lớn (PayPal, Twitter, Google, Facebook, GitHub, v.v.). ``Bạn nên sử dụng nó``.
*   ``Lợi ích của OAuth 2.0``:
    *   Các ứng dụng web hoặc di động không cần chia sẻ mật khẩu.
    *   Nhà cung cấp API có thể thu hồi token cho người dùng cá nhân hoặc toàn bộ ứng dụng mà không yêu cầu người dùng thay đổi mật khẩu gốc của họ. Điều này rất quan trọng nếu thiết bị bị xâm phạm hoặc ứng dụng lừa đảo được phát hiện.
    *   Cải thiện bảo mật và trải nghiệm người dùng cuối tốt hơn.

## 16. Bổ Trợ bằng SDK (Software Development Kit)
*   Mặc dù một API được thiết kế tốt, nhất quán, dựa trên tiêu chuẩn và được tài liệu hóa kỹ lưỡng có thể cho phép nhà phát triển ứng dụng bắt đầu mà không cần SDK client. Tuy nhiên, hầu hết các lập trình viên client đều thích sử dụng một ``SDK được xây dựng tốt`` bằng ngôn ngữ lập trình mà họ lựa chọn, hơn là tiêu thụ trực tiếp API web của bạn.
*   Trên thực tế, các lập trình viên sử dụng SDK có thể coi SDK cụ thể theo ngôn ngữ là API, bỏ qua hoàn toàn Web API cơ bản.
*   Tuy nhiên, ``chất lượng của Web API cơ bản có tác động lớn`` đến chi phí và độ tin cậy của SDK cũng như số lượng SDK được tạo ra. Web API càng tốt, SDK càng tốt.
*   Web API cũng sẽ hiển thị cho lập trình viên trong các tình huống gỡ lỗi và điều chỉnh hiệu suất (ví dụ: Công cụ Google Chrome).

## 17. Quản lý Phiên bản (Versioning)
Việc quản lý phiên bản API là một chủ đề gây tranh cãi.

### 17.1. Cách thực hành phổ biến nhất
*   Đặt ``định danh phiên bản trong một đoạn đường dẫn của URL`` (ví dụ: `/v1/dogs/12345678`).

### 17.2. Không làm gì cả để quản lý phiên bản API là một cách tiếp cận thông minh
*   Nhiều thay đổi có thể được thực hiện một cách ``tương thích ngược`` (backward-compatible). Nếu có thể thay đổi tương thích ngược mà không thay đổi phiên bản, bạn nên làm như vậy.
    *   Ví dụ: an toàn khi thêm các thuộc tính mới vào tài nguyên, miễn là client biết trước rằng điều này có thể xảy ra.
    *   Sử dụng ``PATCH thay vì PUT để cập nhật``. PUT yêu cầu client thay thế toàn bộ nội dung, điều này dễ vỡ. PATCH tránh được rủi ro này.
*   Những thay đổi cơ bản lớn (ví dụ: mô hình dữ liệu) sẽ làm hỏng các client cũ, đòi hỏi một API mới và chiến lược di chuyển.
*   Thường thì ``loại thay đổi "ở giữa"`` (quá lớn để tương thích ngược, quá nhỏ để cần API mới hoàn toàn) ``không tồn tại`` trong thực tế.
*   Nếu bạn phát hành API mà không có quản lý phiên bản và muốn thêm nó sau này, điều này thường dễ dàng. Các yêu cầu thiếu định danh phiên bản được coi là yêu cầu phiên bản 1.
*   Nếu bạn đang phân vân, bạn có thể ``an toàn không đưa quản lý phiên bản vào ngay từ đầu``.

### 17.3. Liên kết (Links) và định danh phiên bản trong URL không phù hợp
*   Nếu bạn quyết định đưa quản lý phiên bản vào API, bạn có thể phải chọn giữa việc đặt định danh phiên bản ở đâu đó trong URL hoặc trong một header.
*   Nếu bạn cũng đang sử dụng liên kết, bạn sẽ thấy rằng việc ``đặt thông tin phiên bản trong một header`` (ví dụ: `Accept-Version`) là lựa chọn đơn giản hơn.
*   Quản lý phiên bản bằng URL cùng với liên kết sẽ phức tạp hơn về mặt khái niệm và thực tế.
*   Cả hai cách tiếp cận (URL hoặc header) đều yêu cầu client tùy chỉnh và tài liệu hóa kiến thức đặc biệt. Việc không triển khai quản lý phiên bản vẫn là đơn giản nhất.

## 18. Kết luận
*   Thiết kế Web API đang không ngừng phát triển và ngày càng quan trọng về mặt thương mại.
*   Web API dựa trên các công nghệ của World Wide Web, được ghi lại trong các đặc tả HTTP và URI. Khi thiết kế, tốt nhất nên ``hạn chế bản thân nhiều nhất có thể`` với các khái niệm và kỹ thuật có trong các đặc tả đó.
*   Các khái niệm gốc của HTTP và URI ``hướng thực thể (entity-oriented)``, giống như thiết kế cơ sở dữ liệu hơn là API ngôn ngữ lập trình thông thường. Ý tưởng là tập trung vào dữ liệu của bạn và để HTTP cung cấp API đồng nhất.
*   ``Phần lớn nỗ lực thiết kế API nên dành cho đặc tả định dạng dữ liệu``.
*   Một lượng đáng kể nỗ lực thiết kế thường dành cho việc ``thiết kế URL thể hiện truy vấn``.
*   Gần đây, sự đánh giá cao về giá trị của ``liên kết trong API`` đã tăng lên cho tất cả các client.
*   ``"Không làm gì cả"`` đối với quản lý phiên bản đã trở thành một lựa chọn khá hợp lý và hoạt động tốt cho nhiều API.
