### **Thiết Kế API Web: Liên Kết Bị Thiếu - Hướng Dẫn Tốt Nhất để Tạo Giao Diện mà Nhà Phát Triển Yêu Thích**

#### **Lời nói đầu (Trang 5)**

**Nội dung chính:**
- Thiết kế API web là một lĩnh vực đang phát triển nhanh chóng do tầm quan trọng ngày càng tăng của API trong kinh doanh và công nghệ.
- Apigee, một nhà lãnh đạo trong quản lý API, đã làm việc với hàng trăm khách hàng để phát triển và quản lý API, từ đó rút ra những xu hướng thiết kế API mang lại lợi ích thực tế.
- Cuốn sách này không phải là hướng dẫn cơ bản cho người mới bắt đầu mà tập trung vào các xu hướng thiết kế API gần đây, xây dựng dựa trên cuốn sách trước đó của Apigee.
- Ví dụ chính trong sách là ứng dụng theo dõi chó và chủ nhân của chúng, với các tài nguyên như:
  - **Chó**: `https://dogtracker.com/dogs/12345678` với dữ liệu JSON:
    ```json
    {
        "id": "12345678",
        "kind": "Dog",
        "name": "Lassie",
        "furcolor": "brown",
        "owner": "98765432"
    }
    ```
  - **Chủ nhân**: `https://dogtracker.com/persons/98765432` với dữ liệu JSON:
    ```json
    {
        "id": "98765432",
        "kind": "Person",
        "name": "Joe Carraclough",
        "hairColor": "brown"
    }
    ```

**Mục tiêu học tập:**
- Hiểu rằng thiết kế API không chỉ là kỹ thuật mà còn liên quan đến việc tối ưu hóa trải nghiệm cho nhà phát triển ứng dụng.
- Nhận biết ví dụ về chó và chủ nhân sẽ được sử dụng xuyên suốt để minh họa các khái niệm thiết kế API.

---

#### **Giới thiệu (Trang 7)**

**Nội dung chính:**
- API web sử dụng giao thức HTTP theo định nghĩa. Trong quá khứ, các công nghệ như SOAP và WSDL đã cố gắng áp dụng các tính năng của các công nghệ phân tán cũ (như CORBA, DCOM) lên HTTP, nhưng chúng phức tạp và dễ vỡ.
- Các API web hiện đại đơn giản hơn, sử dụng HTTP trực tiếp với ít khái niệm bổ sung, được gọi là **RESTful APIs**.
- Apigee khuyến nghị sử dụng HTTP một cách nguyên bản, tuân theo các tiêu chuẩn HTTP để giảm thiểu việc nhà phát triển phải học các khái niệm mới.
- **Lý do**:
  - Nhà phát triển ứng dụng thường làm việc với nhiều API, nên việc tuân thủ tiêu chuẩn HTTP giúp họ tận dụng kiến thức hiện có.
  - Ví dụ: Sử dụng `POST` để tạo tài nguyên phải đi kèm tiêu đề `Location` với URL của tài nguyên mới và mã trạng thái `201 Created`.
  - Sử dụng các tiêu đề như `ETag`, `If-Match` để quản lý xung đột cập nhật, hoặc `Accept` để hỗ trợ các định dạng dữ liệu khác nhau.
- Cuốn sách là tập hợp các phương pháp thiết kế tốt nhất, được phát triển cùng các đội API hàng đầu, và Apigee khuyến khích phản hồi từ cộng đồng qua nhóm API Design trong Apigee Community.

**Mục tiêu học tập:**
- Hiểu sự khác biệt giữa API web hiện đại (RESTful) và các công nghệ cũ như SOAP/WSDL.
- Nhận thức tầm quan trọng của việc tuân thủ tiêu chuẩn HTTP để giảm độ phức tạp và tăng tính thân thiện với nhà phát triển.
- Biết cách sử dụng các tiêu đề HTTP tiêu chuẩn (ví dụ: `Location`, `ETag`, `Accept`) trong thiết kế API.

---

#### **API Web và REST (Trang 8-9)**

##### **Công việc của nhà thiết kế API**
- Mục tiêu chính của nhà thiết kế API là **tối đa hóa năng suất và thành công của nhà phát triển ứng dụng**.
- Nhà phát triển là trung tâm của chiến lược API, và thiết kế phải tập trung vào việc làm cho API dễ sử dụng, trực quan từ góc độ của họ.

##### **API Web là gì?**
- API web là một mô hình yêu cầu và phản hồi HTTP được thiết kế để các chương trình máy tính truy cập, không chỉ giới hạn ở trình duyệt web của con người.

##### **REST là gì?**
- REST (Representational State Transfer) là phong cách kiến trúc của HTTP, được mô tả bởi Roy Fielding, một trong những tác giả chính của đặc tả HTTP.
- HTTP là thực tế, còn REST là tập hợp các ý tưởng thiết kế định hình nó.
- RESTful API sử dụng một số khái niệm HTTP gốc, kết hợp với các khái niệm từ công nghệ khác (như RPC - Remote Procedure Call), nhưng không hoàn toàn tuân theo REST thuần túy.
- Apigee khuyến nghị sử dụng ít khái niệm bổ sung ngoài HTTP nhất có thể (theo nguyên tắc **Occam's Razor** - sử dụng ít khái niệm nhất để giải quyết vấn đề).
- **Lý do sử dụng HTTP thuần túy**:
  - Giảm sự kết nối chặt chẽ (coupling) giữa client và server, cho phép thay đổi một bên mà không làm hỏng bên kia.
  - HTTP đã chứng minh khả năng tách rời client-server qua việc hỗ trợ các client HTTP từ 20 năm trước.

**Mục tiêu học tập:**
- Hiểu vai trò của nhà thiết kế API là phục vụ nhà phát triển ứng dụng.
- Nắm rõ định nghĩa API web và phong cách REST.
- Hiểu tại sao việc giảm thiểu khái niệm bổ sung ngoài HTTP giúp API dễ học và duy trì tính tách rời.

---

#### **HTTP và REST: Mô hình thiết kế hướng dữ liệu (Trang 10-11)**

**Nội dung chính:**
- API REST tập trung vào các **thực thể (entities)** của vấn đề (ví dụ: chó, chủ nhân) thay vì các hàm thao tác trên chúng.
- **Ví dụ**:
  - Tập hợp chó: `https://dogtracker.com/dogs`.
  - Chó cá nhân: `https://dogtracker.com/dogs/12345678`.
  - Tương tự cho chủ nhân.
- **Lợi ích của cách tiếp cận hướng dữ liệu**:
  - Với URL của một thực thể và kiến thức về HTTP, bạn có thể thực hiện:
    - `GET` để lấy chi tiết.
    - `DELETE` để xóa.
    - `PATCH` hoặc `PUT` để sửa đổi.
    - `POST` để tạo mới trong tập hợp.
  - Chỉ cần hiểu thuộc tính của thực thể, không cần học các hàm cụ thể.
  - So sánh với API hướng hàm (function-oriented), nơi bạn phải học nhiều hàm cụ thể (như `/getAllDogs`, `/feedNeeded`), không có cấu trúc rõ ràng, gây khó khăn cho việc học.
- **Tính đồng nhất (uniform interface)** của HTTP giúp giảm gánh nặng học tập và cho phép tạo các phần mềm chung (như trình duyệt, bot tìm kiếm) hoạt động với bất kỳ API nào.

**Mục tiêu học tập:**
- Hiểu sự khác biệt giữa API hướng dữ liệu (REST) và hướng hàm (RPC).
- Nắm rõ cách HTTP cung cấp giao diện đồng nhất cho các thao tác CRUD (Create, Retrieve, Update, Delete).
- Nhận biết lợi ích của việc tập trung vào thực thể thay vì hàm để giảm độ phức tạp.

---

#### **Các yếu tố thiết kế API (Trang 12)**

**Nội dung chính:**
- Thiết kế API bao gồm các yếu tố:
  - **Biểu diễn tài nguyên (representations)**: Định nghĩa các trường dữ liệu và liên kết đến tài nguyên liên quan.
  - **Tiêu đề HTTP tiêu chuẩn**: Sử dụng các tiêu đề như `Location`, `ETag`, `Accept`.
  - **URL và mẫu URL (URI templates)**: Định nghĩa giao diện truy vấn để tìm tài nguyên.
  - **Hành vi yêu cầu từ client**: Ví dụ, caching DNS, xử lý trường mới trong tài nguyên.
- REST nhấn mạnh giao diện đồng nhất, giúp API dễ học và sử dụng.

**Mục tiêu học tập:**
- Hiểu các thành phần chính của thiết kế API.
- Nhận biết vai trò của giao diện đồng nhất trong việc đơn giản hóa API.

---

#### **Thiết kế biểu diễn (Trang 13-25)**

##### **Sử dụng JSON (Trang 13)**
- JSON là định dạng chính cho biểu diễn tài nguyên trong API web vì:
  - Đơn giản, dễ hiểu.
  - Dễ ánh xạ sang cấu trúc dữ liệu của các ngôn ngữ lập trình (JavaScript, Python, Ruby, Java).
- Hạn chế của JSON: Chỉ hỗ trợ các kiểu dữ liệu cơ bản (Null, Boolean, Number, String), không hỗ trợ trực tiếp ngày giờ hoặc URL. Cách xử lý phổ biến là biểu diễn chúng dưới dạng chuỗi (string).
- **Giữ JSON đơn giản**:
  - JSON nên trực quan, dễ hiểu. Ví dụ:
    ```json
    {
        "kind": "Dog",
        "name": "Lassie",
        "furcolor": "brown"
    }
    ```
  - Tên trường (property names) nên là tên thuộc tính của thực thể, không nên sử dụng các tên không liên quan đến mô hình dữ liệu (như trong ví dụ API Facebook phức tạp).

**Mục tiêu học tập:**
- Hiểu tại sao JSON là lựa chọn tiêu chuẩn cho API web.
- Nắm rõ cách giữ JSON đơn giản và sử dụng tên trường phù hợp với mô hình dữ liệu.

##### **Bao gồm liên kết (Links) (Trang 15-17)**
- **Tại sao cần liên kết?**
  - Liên kết (links) biểu diễn mối quan hệ giữa các thực thể (ví dụ: chó và chủ nhân) một cách trực tiếp qua URL.
  - Trước đây, liên kết được liên tưởng với **HATEOAS** (Hypermedia As The Engine Of Application State), nơi client hoạt động như trình duyệt, không cần kiến thức trước về API. Tuy nhiên, client chung như vậy hiếm và tốn kém, nên liên kết hiện được đánh giá cao vì cải thiện tính dễ sử dụng.
- **Ví dụ cải tiến**:
  - Thay vì chỉ có `ownerID`:
    ```json
    {
        "id": "12345678",
        "name": "Lassie",
        "furColor": "brown",
        "ownerID": "98765432"
    }
    ```
  - Thêm `ownerLink`:
    ```json
    {
        "id": "12345678",
        "kind": "Dog",
        "name": "Lassie",
        "furColor": "brown",
        "ownerID": "98765432",
        "ownerLink": "https://dogtracker.com/persons/98765432"
    }
    ```
  - Chủ nhân cũng có liên kết đến danh sách chó:
    ```json
    {
        "id": "98765432",
        "kind": "Person",
        "name": "Joe Carraclough",
        "hairColor": "brown",
        "dogsLink": "https://dogtracker.com/persons/98765432/dogs"
    }
    ```
- **Lợi ích**:
  - Giảm sự phụ thuộc vào tài liệu API để tìm mẫu URL.
  - Không cần viết mã xử lý mẫu URL (template processor).
  - Client dễ dàng điều hướng bằng cách sử dụng liên kết trực tiếp.
- **Mẫu URL (URI Templates) vẫn cần thiết?**
  - Có, vì liên kết chỉ cho biết nơi bạn có thể đi từ vị trí hiện tại, trong khi mẫu URL cho phép truy cập trực tiếp đến tài nguyên cụ thể bằng cách sử dụng thông tin bạn đã biết (như ID hoặc tên).

**Mục tiêu học tập:**
- Hiểu vai trò của liên kết trong việc cải thiện tính dễ sử dụng của API.
- Nhận biết sự khác biệt giữa liên kết (links) và mẫu URL (URI templates).
- Biết cách thêm liên kết vào biểu diễn JSON để hỗ trợ điều hướng.

##### **Bao gồm liên kết, bước 2 (Trang 20-21)**
- **Cải tiến tiếp theo**:
  - Thay vì chỉ thêm liên kết chỉ đọc (read-only), có thể sử dụng liên kết có thể ghi (read-write) để biểu diễn mối quan hệ linh hoạt hơn.
  - Ví dụ: Nếu chó có thể thuộc về người hoặc tổ chức (institution), thay vì dùng `ownerID` và `ownerType`, chỉ cần một trường `owner` với giá trị URL:
    ```json
    {
        "self": "https://dogtracker.com/dogs/12345678",
        "id": "12345678",
        "kind": "Dog",
        "name": "Lassie",
        "furColor": "brown",
        "owner": "https://dogtracker.com/persons/98765432"
    }
    ```
  - Client có thể cập nhật `owner` bằng cách thay đổi URL, ví dụ, từ người sang tổ chức.
- **Cảnh báo**:
  - Không nên lưu trữ URL tuyệt đối (absolute URLs) trong cơ sở dữ liệu vì nếu tên miền thay đổi, các URL sẽ lỗi. Thay vào đó, server nên tách phần scheme và authority, lưu trữ phần còn lại, và tái tạo URL khi cần.
  - Sử dụng URL tương đối (relative URLs) là một lựa chọn, nhưng đẩy gánh nặng xử lý sang client để chuyển thành URL tuyệt đối.

**Mục tiêu học tập:**
- Hiểu cách sử dụng liên kết có thể ghi để tăng tính linh hoạt.
- Nắm rõ các vấn đề khi lưu trữ URL trong cơ sở dữ liệu và cách sử dụng URL tương đối.

##### **Cách biểu diễn liên kết trong tài nguyên (Trang 22)**
- **Cách được đề xuất**:
  - Sử dụng cặp tên/giá trị JSON đơn giản:
    ```json
    "owner": "https://dogtracker.com/persons/98765432"
    ```
  - Ưu điểm: Đơn giản, nhất quán với cách biểu diễn các thuộc tính khác.
  - Nhược điểm: JSON không có kiểu dữ liệu URL, nên client cần ngữ cảnh để biết chuỗi nào là URL.
- **Các cách khác** (xem Phụ lục, Trang 62-63):
  - Sử dụng đối tượng JSON: `"owner": {"href": "https://dogtracker.com/persons/98765432"}`.
  - Sử dụng mảng liên kết: `"links": [{"href": "https://dogtracker.com/persons/98765432", "rel": "owner"}]`.
  - Sử dụng kiểu con (subtype): `"owner": {"subtype": "URI", "value": "https://dogtracker.com/persons/98765432"}`.
  - Apigee khuyến nghị giữ đơn giản với cặp tên/giá trị, tránh các định dạng phức tạp như Siren, HAL, JSON-LD.

**Mục tiêu học tập:**
- Hiểu cách biểu diễn liên kết đơn giản trong JSON.
- Nhận biết các phương pháp thay thế và lý do Apigee ưu tiên cách đơn giản.

##### **Ai sử dụng liên kết? (Trang 22-24)**
- Liên kết không phải là thông lệ phổ biến nhất, nhưng được sử dụng trong các API lớn như Google Drive và GitHub.
- **Ví dụ Google Drive**:
  ```json
  {
      "kind": "drive#file",
      "id": "0B8G-Akr_SmtmaEJneE2LY1BBdW...",
      "selfLink": "https://www.googleapis.com/drive/v2/files/0B8G-Akr_Smtm...",
      "webContentLink": "https://docs.google.com/uc?id=0B8G-Akr_SmtmaEJneE...",
      "alternateLink": "https://drive.google.com/file/d/0B8G-Akr_SmtmaEJne...",
      "iconLink": "https://ssl.gstatic.com/docs/doclist/images/icon_12_pdf...",
      "thumbnailLink": "https://lh4.googleusercontent.com/RECVMLRuNGsohM1C...",
      "title": "Soils Report.pdf",
      "mimeType": "application/pdf",
      "parents": [
          {
              "kind": "drive#parentReference",
              "id": "0AMG-Akr_Smtmuk9PWA",
              "selfLink": "https://www.googleapis.com/drive/v2/files/0B8G-Akr_Sm...",
              "parentLink": "https://www.googleapis.com/drive/v2/files/0AMG-Akr_...",
              "isRoot": false
          }
      ]
  }
  ```
- **Ví dụ GitHub**:
  ```json
  {
      "id": 1,
      "url": "https://api.github.com/repos/octocat/Hello-World/issues/1347",
      "repository_url": "https://api.github.com/repos/octocat/Hello-World",
      "labels_url": "https://api.github.com/repos/octocat/Hello-World/issues/1347/labels{/name}",
      "comments_url": "https://api.github.com/repos/octocat/Hello-World/issues/1347/comments",
      "events_url": "https://api.github.com/repos/octocat/Hello-World/issues/1347/events",
      "html_url": "https://github.com/octocat/Hello-World/issues/1347",
      ...
  }
  ```
- Một số liên kết của GitHub là mẫu URL, yêu cầu client xử lý thêm.

**Mục tiêu học tập:**
- Hiểu rằng các API lớn như Google Drive và GitHub sử dụng liên kết để cải thiện điều hướng.
- Nhận biết cách liên kết được tích hợp trong các API thực tế.

---

#### **Thiết kế URL (Trang 26-32)**

##### **Trong URL, danh từ tốt, động từ xấu (Trang 26)**
- Trong mô hình REST hướng dữ liệu, URL nên đại diện cho **thực thể (nouns)**, không phải hành động (verbs).
- Ví dụ: Sử dụng `https://dogtracker.com/dogs` thay vì `/getAllDogs`.

##### **URL nổi tiếng (Well-known URLs) (Trang 26)**
- Mỗi API cần ít nhất một URL nổi tiếng để client bắt đầu, ví dụ: `https://dogtracker.com/dogs` cho tập hợp chó.
- Có thể có một URL gốc (như `https://dogtracker.com/`) chứa liên kết đến các tài nguyên khác, giúp API có thể khám phá (discoverable).

##### **Thiết kế URL cho thực thể (Trang 26-27)**
- Client không cần tự xây dựng URL cho thực thể riêng lẻ; server cung cấp URL khi tạo tài nguyên.
- URL thực thể nên thân thiện với con người, ví dụ: `https://dogtracker.com/dogs/a8098c1a` thay vì `https://dogtracker.com/269n;a8098c1a`.

##### **Permalinks (Trang 27-28)**
- Liên kết (links) trong biểu diễn tài nguyên có thể được lưu trữ và sử dụng sau này, nên cần ổn định.
- URL ổn định thường chứa chuỗi ký tự ngẫu nhiên, ví dụ: `https://dogtracker.com/dogs/12345678`.
- Định dạng phổ biến: `https://dogtracker.com/{type}/{uuid}`, với `type` là loại tài nguyên (như `dogs`) và `uuid` là định danh duy nhất.

##### **Web phẳng (The web is flat) (Trang 28)**
- Tránh mã hóa phân cấp (hierarchies) trong URL liên kết vì phân cấp có thể thay đổi, làm hỏng liên kết. Ví dụ, không dùng `https://dogtracker.com/owners/department/123/dogs`.

##### **Giải pháp cho vấn đề đổi tên (Trang 29-32)**
- **Vấn đề**: Sử dụng định danh thân thiện với con người (như tên) trong URL có thể gây vấn đề nếu tên thay đổi, nhưng định danh máy (UUID) thì khó nhớ.
- **Giải pháp**:
  - Sử dụng định danh thân thiện trong mẫu URL cho truy vấn: `https://dogtracker.com/persons/JoeMCarraclough`.
  - Sử dụng UUID trong liên kết: `https://dogtracker.com/persons/e9cdcf7a-25b3-11e5-34363bd0ac10`.
  - Ví dụ: Khi Joe đổi tên thành JoeSmith44, liên kết không thay đổi, nhưng truy vấn dùng tên mới.
- Hai URL (`https://dogtracker.com/persons/e9cdcf7a-...` và `https://dogtracker.com/persons/JoeMCarraclough`) có thể trỏ đến các tài nguyên khác nhau.

**Mục tiêu học tập:**
- Hiểu tại sao URL nên sử dụng danh từ và tránh động từ.
- Nắm rõ vai trò của URL nổi tiếng và cách thiết kế URL thực thể.
- Hiểu khái niệm permalink và lý do tránh mã hóa phân cấp trong liên kết.
- Học cách giải quyết vấn đề đổi tên bằng cách tách URL truy vấn và liên kết.

---

#### **Thiết kế URL truy vấn (Trang 32-36)**

**Nội dung chính:**
- **Mẫu URL truy vấn**:
  - Mẫu URL như `https://dogtracker.com/persons/{personID}/dogs` hoặc `https://dogtracker.com/search?type=Dog&ownerId={personID}` định nghĩa ngôn ngữ truy vấn cho API.
  - Mẫu URL cụ thể cho API, không giống Google cung cấp ngôn ngữ truy vấn chung.
- **Biểu diễn mối quan hệ trong URL truy vấn (Trang 34)**:
  - Mối quan hệ (như chó thuộc về chủ) nên được biểu diễn đối xứng trong URL và biểu diễn tài nguyên. Ví dụ:
    - URL: `https://dogtracker.com/persons/98765432/dogs`.
    - Biểu diễn: Liên kết `dogsLink` trong tài nguyên chủ nhân.
- **Mô hình chung cho URL truy vấn (Trang 35)**:
  - Sử dụng tham số truy vấn (query parameters) để lọc hoặc tìm kiếm, ví dụ: `https://dogtracker.com/dogs?color=brown`.
- **Tham số đường dẫn hoặc tham số ma trận (Trang 36)**:
  - Tham số đường dẫn (path parameters) như `{personID}` trong `https://dogtracker.com/persons/{personID}`.
  - Tham số ma trận (matrix parameters) ít phổ biến hơn, ví dụ: `https://dogtracker.com/dogs;color=brown`.
- **Lọc tập hợp (Trang 36)**:
  - Sử dụng tham số truy vấn để lọc, ví dụ: `https://dogtracker.com/dogs?color=brown&age=5`.

**Mục tiêu học tập:**
- Hiểu cách thiết kế URL truy vấn để tìm kiếm tài nguyên.
- Nhận biết sự đối xứng giữa URL truy vấn và liên kết trong biểu diễn.
- Nắm rõ cách sử dụng tham số truy vấn để lọc tập hợp.

---

#### **Thêm về thiết kế biểu diễn (Trang 40-48)**

##### **Bao gồm thuộc tính self và kind (Trang 40)**
- Mỗi tài nguyên nên có:
  - **self**: URL của chính tài nguyên, ví dụ: `"self": "https://dogtracker.com/dogs/12345678"`.
  - **kind**: Loại tài nguyên, ví dụ: `"kind": "Dog"`.
- **Lợi ích**:
  - `self` giúp xác định vị trí tài nguyên.
  - `kind` giúp client hiểu loại tài nguyên mà không cần phân tích URL.

##### **Biểu diễn tập hợp (Trang 40-43)**
- Tập hợp (collections) nên được biểu diễn đơn giản trong JSON:
  ```json
  {
      "self": "https://dogtracker.com/dogs",
      "kind": "Collection",
      "contents": [
          {"self": "https://dogtracker.com/dogs/12344", "kind": "Dog", "name": "Fido", "furColor": "white"},
          {"self": "https://dogtracker.com/dogs/12345", "kind": "Dog", "name": "Rover", "furColor": "brown"}
      ]
  }
  ```
- Tránh sử dụng tên trường tùy chỉnh như `dogs` để dễ viết mã client chung.

##### **Tập hợp phân trang (Paginated collections) (Trang 43-45)**
- Tập hợp lớn nên được phân trang để tránh tải toàn bộ dữ liệu. Ví dụ:
  ```http
  GET /dogs HTTP/1.1
  Host: dogtracker.com
  Accept: application/json
  ```
  Phản hồi:
  ```http
  HTTP/1.1 303 See Other
  Location: https://dogtracker.com/dogs?limit=25, offset=0
  ```
  Yêu cầu tiếp theo:
  ```http
  GET /dogs?limit=25, offset=0 HTTP/1.1
  ```
  Phản hồi:
  ```json
  {
      "self": "https://dogtracker.com/dogs?limit=25, offset=0",
      "kind": "Page",
      "pageOf": "https://dogtracker.com/dogs",
      "next": "https://dogtracker.com/dogs?limit=25, offset=25",
      "contents": [
          {"self": "https://dogtracker.com/dogs/12344", "kind": "Dog", "name": "Fido", "furColor": "white"},
          {"self": "https://dogtracker.com/dogs/12345", "kind": "Dog", "name": "Rover", "furColor": "brown"},
          ...
      ]
  }
  ```
- Sử dụng liên kết `next`, `previous`, `first`, `last` (đăng ký với IANA) để điều hướng phân trang.

##### **Loại tài nguyên tùy chỉnh và sử dụng URL cho loại (Trang 45)**
- Có thể dùng URL cho thuộc tính `kind` để tránh xung đột, ví dụ: `"kind": "https://apigee.com/collections#Dog"`.
- Tuy nhiên, điều này làm JSON dài hơn và ít phổ biến.

##### **Hỗ trợ nhiều định dạng (Trang 46)**
- JSON là định dạng chính, nhưng nếu hỗ trợ nhiều định dạng (như XML), sử dụng tiêu đề `Accept`.
- Hỗ trợ `PATCH` với JSON Patch hoặc JSON Merge Patch để cập nhật tài nguyên một cách linh hoạt, tránh vấn đề của `PUT`.

##### **Tên thuộc tính (Trang 47)**
- Sử dụng `camelCase` (ví dụ: `furColor`) thay vì `snake_case` (ví dụ: `fur_color`) vì phù hợp với JavaScript, Java, Objective-C.
- Tránh các ký tự không tương thích (như `-`) hoặc từ khóa dành riêng (như `self`, `this`).

##### **Định dạng ngày giờ (Trang 48)**
- Sử dụng định dạng XML Schema (dựa trên ISO 8601) hoặc Unix epoch (milliseconds từ 1970).
- Ví dụ:
  - XML Schema: `"DateTime": "2011-10-29T09:35:00Z"`.
  - Unix: `"createdAt": 1475795458`.

**Mục tiêu học tập:**
- Hiểu cách sử dụng thuộc tính `self` và `kind` trong biểu diễn.
- Nắm rõ cách biểu diễn và phân trang tập hợp.
- Biết cách hỗ trợ nhiều định dạng và chọn tên thuộc tính phù hợp.
- Hiểu các lựa chọn định dạng ngày giờ.

##### **API "nói nhiều" (Chatty APIs) (Trang 48)**
- Quan niệm sai lầm rằng API REST "nói nhiều" (yêu cầu nhiều HTTP request). Vấn đề này đến từ thiết kế tài nguyên không phù hợp, không phải do REST.
- **Giải pháp**: Tạo tài nguyên không chuẩn hóa (denormalized) chỉ đọc để giảm số lượng yêu cầu, ví dụ, tài nguyên tổng hợp thông tin chó và chủ nhân.

##### **Phân trang và phản hồi từng phần (Trang 48-50)**
- **Phản hồi từng phần (Partial response)**:
  - Cho phép client chỉ yêu cầu các trường cần thiết, giảm băng thông.
  - Ví dụ: `GET /dogs?fields=name,color,location`.
  - Cú pháp của Google và Facebook (`fields=...`) được khuyến nghị.
- **Phân trang**:
  - Sử dụng `limit` và `offset` (như Facebook, LinkedIn) để kiểm soát phân trang.
  - Ví dụ: Lấy 50-75 bản ghi: `https://dogtracker.com/dogs?limit=25&offset=50`.

**Mục tiêu học tập:**
- Hiểu cách giảm "nói nhiều" bằng tài nguyên không chuẩn hóa.
- Nắm rõ cách triển khai phản hồi từng phần và phân trang.

---

#### **Xử lý lỗi (Trang 51-53)**

**Nội dung chính:**
- Xử lý lỗi là yếu tố quan trọng trong trải nghiệm API, vì:
  - Lỗi giúp nhà phát triển học cách sử dụng API.
  - Lỗi cung cấp ngữ cảnh khi xử lý sự cố trong ứng dụng.
- **Khuyến nghị**:
  - Sử dụng mã trạng thái HTTP tiêu chuẩn (ví dụ: `201 Created`, `200 OK`, `405 Method Not Allowed`) với các tiêu đề phù hợp:
    - `201 Created` với `Location`.
    - `200 OK` với `Content-Location`, `ETag`.
    - `405 Method Not Allowed` với `Allow`.
  - Phản hồi lỗi nên chi tiết, dễ hiểu:
    ```json
    {
        "developerMessage": "Mô tả chi tiết vấn đề cho nhà phát triển với gợi ý cách sửa",
        "userMessage": "Thông điệp cho người dùng ứng dụng nếu cần",
        "errorCode": 12345,
        "moreInfo": "https://dev.teachdogrest.com/errors/12345"
    }
    ```

**Mục tiêu học tập:**
- Hiểu tầm quan trọng của xử lý lỗi trong API.
- Nắm rõ cách sử dụng mã trạng thái và tiêu đề HTTP tiêu chuẩn.
- Biết cách tạo thông điệp lỗi chi tiết, dễ hiểu.

---

#### **Mô hình hóa hành động (Trang 53-54)**

**Nội dung chính:**
- Thay vì sử dụng API kiểu RPC với động từ, REST sử dụng tài nguyên để mô hình hóa hành động.
- **Cách 1**: Sử dụng thuộc tính trạng thái (state), ví dụ: đặt trạng thái của quy trình thành `started`, `stopped`, `paused`.
- **Cách 2**: Sử dụng URL liên quan để gửi yêu cầu hành động qua `POST`. Ví dụ:
  ```json
  {
      "id": "https://example.org/process/123456",
      "kind": "Process",
      "actionRequests": "https://example.org/processes/123456/requests"
  }
  ```
  - Client gửi `POST` với `StopRequest`, `StartRequest`, v.v. đến URL này.
- **Cách 3**: Sử dụng các thuộc tính riêng cho mỗi hành động để chỉ ra hành động hợp lệ:
  ```json
  {
      "id": "https://example.org/process/123456",
      "kind": "Process",
      "state": "initial",
      "startRequests": "https://example.org/process/123456/requests"
  }
  ```

**Mục tiêu học tập:**
- Hiểu cách mô hình hóa hành động trong REST mà không dùng động từ.
- Nắm rõ các phương pháp sử dụng trạng thái hoặc URL hành động.

---

#### **Xác thực (Trang 55)**

**Nội dung chính:**
- **OAuth 2.0** là tiêu chuẩn xác thực phổ biến, được sử dụng bởi PayPal, Twitter, Google, Facebook, GitHub.
- Lợi ích:
  - Không yêu cầu chia sẻ mật khẩu.
  - Cho phép thu hồi token mà không cần thay đổi mật khẩu gốc.
  - Tăng cường bảo mật và trải nghiệm người dùng.

**Mục tiêu học tập:**
- Hiểu tại sao OAuth 2.0 là lựa chọn tiêu chuẩn cho xác thực API.

---

#### **Bổ sung bằng SDK (Trang 55)**

**Nội dung chính:**
- API tốt (nhất quán, tuân chuẩn, tài liệu rõ ràng) có thể không cần SDK, nhưng nhà phát triển thường thích SDK vì dễ sử dụng.
- Chất lượng API web ảnh hưởng đến chi phí và độ tin cậy của SDK.
- SDK giúp nhà phát triển tập trung vào logic ứng dụng thay vì xử lý HTTP trực tiếp.

**Mục tiêu học tập:**
- Hiểu vai trò của SDK trong việc cải thiện trải nghiệm nhà phát triển.
- Nhận biết tầm quan trọng của thiết kế API tốt để hỗ trợ SDK.

---

#### **Quản lý phiên bản (Versioning) (Trang 56-59)**

**Nội dung chính:**
- **Không làm gì cho versioning**:
  - Nhiều thay đổi có thể thực hiện mà không phá vỡ client (backward-compatible), như thêm thuộc tính mới hoặc loại tài nguyên mới.
  - Sử dụng `PATCH` thay vì `PUT` để cập nhật, vì `PATCH` chỉ thay đổi dữ liệu được chỉ định, an toàn hơn.
  - Nếu không chắc chắn, có thể bỏ qua versioning vì có thể thêm sau này dễ dàng.
- **Liên kết và định danh phiên bản trong URL**:
  - Đặt phiên bản trong URL (như `/v2/dogs/12345678`) gây phức tạp khi sử dụng liên kết, vì server phải đoán phiên bản trong liên kết.
  - Giải pháp: Sử dụng URL tương đối trong liên kết (như `"owner": "/persons/98765432"`) để client tự thêm phiên bản.

**Mục tiêu học tập:**
- Hiểu các lựa chọn versioning và lợi ích của việc không versioning.
- Nắm rõ vấn đề khi kết hợp versioning với liên kết và cách sử dụng URL tương đối.

---

#### **Kết luận (Trang 60)**

**Nội dung chính:**
- API web dựa trên HTTP và URI, nên hạn chế sử dụng khái niệm ngoài các tiêu chuẩn này.
- Thiết kế hướng dữ liệu giống như thiết kế cơ sở dữ liệu, tập trung vào tài nguyên và để HTTP cung cấp giao diện đồng nhất.
- Liên kết ngày càng được đánh giá cao trong thiết kế API vì cải thiện tính dễ sử dụng.
- Xử lý lỗi, xác thực (OAuth 2.0), và SDK là các yếu tố quan trọng để nâng cao trải nghiệm nhà phát triển.

**Mục tiêu học tập:**
- Tổng hợp các khái niệm chính của thiết kế API RESTful.
- Hiểu tại sao thiết kế hướng dữ liệu và tuân thủ HTTP là quan trọng.

---

#### **Phụ lục: Các cách khác để biểu diễn liên kết (Trang 62-63)**

**Nội dung chính:**
- **Cách 1**: `"owner": {"href": "https://dogtracker.com/persons/98765432"}`.
  - Ưu điểm: Dễ nhận biết URL mà không cần ngữ cảnh.
- **Cách 2**: `"links": [{"href": "https://dogtracker.com/persons/98765432", "rel": "owner"}]`.
  - Nhược điểm: Phức tạp hơn, tên thuộc tính xuất hiện ở giá trị `rel`.
- **Cách 3**: `"owner": {"subtype": "URI", "value": "https://dogtracker.com/persons/98765432"}`.
  - Có thể trả về đối tượng URI, nhưng dễ gây lỗi nếu lập trình viên quên xử lý đúng.
- Apigee khuyến nghị cách đơn giản nhất (cặp tên/giá trị) để tránh phức tạp.

**Mục tiêu học tập:**
- Hiểu các cách thay thế để biểu diễn liên kết và lý do ưu tiên cách đơn giản.

---

### **Bài tập củng cố kiến thức (bằng tiếng Việt)**

1. **Câu hỏi lý thuyết**:
   - Tại sao JSON là định dạng tiêu chuẩn cho API web?
   - Sự khác biệt giữa API RESTful và API kiểu RPC là gì?
   - Tại sao nên sử dụng liên kết trong biểu diễn tài nguyên?

2. **Bài tập thực hành**:
   - Thiết kế biểu diễn JSON cho một tài nguyên "Dog" với liên kết đến chủ nhân và danh sách chó của chủ nhân.
   - Viết một URL truy vấn để tìm tất cả chó có màu lông nâu và tuổi 5.
   - Thiết kế phản hồi lỗi cho trường hợp client gửi yêu cầu `PUT` không hợp lệ.

3. **Câu hỏi thảo luận**:
   - Bạn sẽ chọn cách nào để biểu diễn liên kết trong API của mình? Tại sao?
   - Trong trường hợp nào bạn sẽ chọn không versioning cho API?

---

### **Hướng dẫn tiếp theo**
- Nếu bạn muốn đi sâu vào một phần cụ thể (ví dụ: thiết kế URL, xử lý lỗi), hãy cho tôi biết để tôi cung cấp thêm ví dụ hoặc giải thích chi tiết hơn.
- Nếu bạn muốn tôi tạo bài tập hoặc câu hỏi kiểm tra thêm, hãy yêu cầu cụ thể.
- Bạn có thể tham gia cộng đồng API Design của Apigee để thảo luận thêm, như được đề xuất trong sách.

Hãy cho tôi biết cách bạn muốn tiếp tục học hoặc nếu bạn có câu hỏi cụ thể về nội dung!
