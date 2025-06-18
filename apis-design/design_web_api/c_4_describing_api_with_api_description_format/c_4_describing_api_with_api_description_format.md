Chương 4 bao gồm các phần sau:

1.  **Định dạng mô tả API là gì?** (What is an API description format?)
    *   Giới thiệu Đặc tả OpenAPI (OAS) (Introducing the OpenAPI Specifcation (OAS))
    *   Tại sao nên sử dụng định dạng mô tả API? (Why use an API description format?)
    *   Khi nào nên sử dụng định dạng mô tả API? (When to use an API description format?)
2.  **Mô tả tài nguyên và hành động của API bằng OAS** (Describing API resources and actions with OAS)
    *   Tạo tài liệu OAS (Creating an OAS document)
    *   Mô tả tài nguyên (Describing a resource)
    *   Mô tả các thao tác trên tài nguyên (Describing operations on a resource)
3.  **Mô tả dữ liệu API bằng OpenAPI và JSON Schema** (Describing API data with OpenAPI and JSON Schema)
    *   Mô tả tham số truy vấn (Describing query parameters)
    *   Mô tả dữ liệu bằng JSON Schema (Describing data with JSON Schema)
    *   Mô tả phản hồi (Describing responses)
    *   Mô tả tham số thân yêu cầu (Describing body parameters)
4.  **Mô tả API hiệu quả bằng OAS** (Describing an API effciently with OAS)
    *   Tái sử dụng các thành phần (Reusing components)
    *   Mô tả tham số đường dẫn (Describing path parameters)

Bây giờ, chúng ta hãy đi sâu vào từng phần:

**1. Định dạng mô tả API là gì?** (What is an API description format?)

Một định dạng mô tả API là một **định dạng dữ liệu dùng để mô tả một API**. Nó giống như một tệp văn bản có cấu trúc, sử dụng dữ liệu để truyền tải thông tin về giao diện lập trình của API.

Ví dụ, mục tiêu "Thêm sản phẩm vào danh mục" (add product to catalog) có thể được mô tả trong một tệp văn bản sử dụng định dạng này:

```yaml
/products:
    description: Catalog
    post:
        description: Add product to catalog
        requestBody:
            description: Product
            content:
                application/json:
                    schema:
                        properties:
                            name:
                                type: string
                                example: The Design of Web APIs
                            price:
                                type: number
                                example: 44.99
```

Tệp này sử dụng dữ liệu để kể lại câu chuyện về cách thêm sản phẩm. Nó cho biết tài nguyên `/products` đại diện cho danh mục, hỗ trợ phương thức HTTP `POST` để thêm sản phẩm. Thao tác `POST` này yêu cầu một `requestBody` chứa thông tin sản phẩm, được mô tả là đối tượng JSON với các thuộc tính `name` (string) và `price` (number), kèm theo các ví dụ.

Điều quan trọng là vì tệp này chứa **dữ liệu có cấu trúc**, các chương trình có thể đọc nó và chuyển đổi dễ dàng thành dạng khác, ví dụ, tạo tài liệu tham khảo tự động.

**1.1.1 Giới thiệu Đặc tả OpenAPI (OAS)** (Introducing the OpenAPI Specifcation (OAS))

**Đặc tả OpenAPI (OAS)** là một **định dạng mô tả API REST độc lập với ngôn ngữ lập trình**. Nó được thúc đẩy bởi Sáng kiến OpenAPI (OAI).

OAS trước đây được gọi là Đặc tả Swagger và được đổi tên vào năm 2016 khi được tặng cho OAI. Phiên bản mới nhất tại thời điểm cuốn sách được viết là 3.0.

OAS có thể được viết bằng **YAML** hoặc **JSON**. YAML được khuyến nghị sử dụng vì dễ đọc và viết hơn.

So sánh YAML và JSON:
*   YAML không cần dấu ngoặc kép (`" "`) xung quanh tên thuộc tính và giá trị.
*   Dấu ngoặc nhọn (`{}`) và dấu phẩy (`,`) của JSON được thay thế bằng xuống dòng và thụt lề trong YAML.
*   Dấu ngoặc vuông (`[]`) và dấu phẩy (`,`) của JSON được thay thế bằng dấu gạch ngang (`-`) và xuống dòng trong YAML.
*   YAML cho phép nhận xét bằng dấu `#`, JSON thì không.

Ví dụ tài liệu OAS 3.0 cơ bản mô tả mục tiêu "tìm kiếm sản phẩm" (search for products) của API mua sắm:

```yaml
openapi: "3.0.0" # Phiên bản đặc tả OpenAPI
info: # Thông tin chung về API
    title: Shopping API # Tiêu đề API
    version: "1.0" # Phiên bản API
paths: # Tài nguyên (URLs)
    /products: # Đường dẫn tài nguyên
        get: # Phương thức HTTP
            summary: Search for products # Tóm tắt thao tác
            parameters: # Tham số của thao tác
                - name: free-query # Tên tham số
                  description: free query # Mô tả tham số
                  in: query # Vị trí tham số (query parameter)
                  schema: # Cấu trúc dữ liệu tham số
                    type: string # Kiểu dữ liệu (chuỗi)
            responses: # Các phản hồi của thao tác
                "200": # Mã trạng thái HTTP
                    description: Products matching query # Mô tả phản hồi
                    content: # Nội dung phản hồi
                        application/json: # Kiểu media
                            schema: # Cấu trúc dữ liệu nội dung
                                type: array # Kiểu dữ liệu (mảng)
                                items: # Các mục trong mảng
                                    properties:
                                        name:
                                            type: string
                                        price:
                                            type: number
```


Một tài liệu OAS cơ bản cung cấp thông tin chung về API (tên, phiên bản), mô tả tài nguyên (được xác định bằng đường dẫn) và các thao tác trên mỗi tài nguyên (được xác định bằng phương thức HTTP), bao gồm tham số và phản hồi.

**1.1.2 Tại sao nên sử dụng định dạng mô tả API?** (Why use an API description format?)

Sử dụng định dạng mô tả API hiệu quả hơn nhiều so với việc sử dụng trình soạn thảo văn bản hoặc bảng tính. Lợi ích bao gồm:

*   **Lưu trữ phiên bản dễ dàng:** Tài liệu OAS là tệp văn bản, có thể lưu trữ trong hệ thống kiểm soát phiên bản như Git, giúp dễ dàng theo dõi sửa đổi.
*   **Mô tả hiệu quả:** Cấu trúc của tài liệu OAS giúp mô tả giao diện lập trình hiệu quả hơn. Bạn có thể định nghĩa các thành phần có thể tái sử dụng (ví dụ: mô hình dữ liệu) để tránh sao chép.
*   **Sử dụng công cụ chuyên dụng:** Có các trình soạn thảo OAS chuyên dụng (như Swagger Editor) cung cấp các tính năng hỗ trợ như tự động hoàn thành, kiểm tra cú pháp, và hiển thị tài liệu trực quan.
*   **Chia sẻ và tạo tài liệu:** Tài liệu OAS có thể dễ dàng chia sẻ và được sử dụng để tạo tài liệu tham khảo API thân thiện với người dùng (sử dụng các công cụ như Swagger UI, ReDoc).
*   **Tạo mã và cấu hình công cụ:** Vì là định dạng máy có thể đọc được, OAS có thể dùng để tạo khung sườn mã triển khai API hoặc mã SDK cho client, cấu hình các công cụ liên quan đến API như API gateway.

**1.1.3 Khi nào nên sử dụng định dạng mô tả API?** (When to use an API description format?)

Định dạng mô tả API **không nên được sử dụng** khi:
*   **Xác định mục tiêu API:** Đây là bước quá sớm.
*   **Xác định các khái niệm đằng sau mục tiêu:** Vẫn còn quá sớm.

Nó **chắc chắn phải được sử dụng** khi:
*   **Thiết kế biểu diễn có thể lập trình của mục tiêu và khái niệm, cùng với dữ liệu**.

Khi thiết kế API REST, bạn có thể bắt đầu sử dụng OAS khi **thiết kế đường dẫn tài nguyên và chọn phương thức HTTP**. Sau đó, bạn hoàn thành tài liệu bằng cách mô tả dữ liệu của API.

**2. Mô tả tài nguyên và hành động của API bằng OAS** (Describing API resources and actions with OAS)

Trong chương 3, chúng ta đã chuyển các mục tiêu API thành các cặp tài nguyên và hành động, được biểu diễn bằng đường dẫn và phương thức HTTP. OAS cho phép mô tả điều này một cách có cấu trúc.

**2.2.1 Tạo tài liệu OAS** (Creating an OAS document)

Tài liệu OAS tối thiểu phải chứa phiên bản đặc tả (`openapi`), thông tin chung (`info`), và thuộc tính `paths` (ban đầu có thể trống). Ví dụ:

```yaml
openapi: "3.0.0" # Phiên bản OAS
info: # Thông tin API
    title: Shopping API
    version: "1.0"
paths: {} # Tài nguyên ban đầu trống
```


Lưu ý rằng cả phiên bản đặc tả và phiên bản API (`info.version`) phải được đặt trong dấu ngoặc kép vì chúng được coi là chuỗi.

**2.2.2 Mô tả tài nguyên** (Describing a resource)

Để mô tả tài nguyên, bạn thêm đường dẫn của tài nguyên vào thuộc tính `paths`. Bạn có thể thêm thuộc tính `description` để mô tả tài nguyên đó. Ví dụ, mô tả tài nguyên danh mục sản phẩm `/products`:

```yaml
openapi: "3.0.0"
info:
    title: Shopping API
    version: "1.0"
paths:
    /products: # Thêm đường dẫn tài nguyên
        description: The products catalog # Mô tả tài nguyên
```


Mô tả tài nguyên không bắt buộc nhưng hữu ích trong suốt vòng đời API, giúp liên kết thiết kế với mục tiêu ban đầu và giúp người đọc tài liệu dễ hiểu hơn.

**2.2.3 Mô tả các thao tác trên tài nguyên** (Describing operations on a resource)

Một tài nguyên được mô tả trong OAS phải chứa ít nhất một thao tác (operation). Các thao tác được mô tả bằng các **phương thức HTTP**. Ví dụ, thêm thao tác tìm kiếm sản phẩm (`GET`) và thêm sản phẩm (`POST`) vào tài nguyên `/products`:

```yaml
paths:
    /products:
        description: The products catalog
        get: # Thêm thao tác GET
            summary: Search for products # Tóm tắt
            description: | # Mô tả chi tiết (multiline)
                Search for products in catalog
                using a free query parameter
            responses: # Các phản hồi có thể
                "200": # Mã trạng thái 200 OK
                    description: |
                        Products matching free query parameter
        post: # Thêm thao tác POST
            summary: Add product
            description: |
                Add product (described in product info
                parameter) to catalog
            responses:
                "200":
                    description: |
                        Product added to catalog
```


Mỗi thao tác (`get`, `post`,...) có thuộc tính `summary` (mô tả ngắn) và `description` (mô tả chi tiết). Thuộc tính `responses` liệt kê các phản hồi có thể, được xác định bằng mã trạng thái HTTP (đặt trong dấu ngoặc kép, ví dụ: `"200"`) và mỗi phản hồi phải có `description`.

**3. Mô tả dữ liệu API bằng OpenAPI và JSON Schema** (Describing API data with OpenAPI and JSON Schema)

OAS dựa vào **đặc tả JSON Schema** để mô tả tất cả dữ liệu trong API, bao gồm tham số truy vấn, tham số thân yêu cầu và thân phản hồi. JSON Schema nhằm mục đích mô tả định dạng dữ liệu một cách rõ ràng, dễ đọc cho cả con người và máy móc, và có thể được sử dụng để kiểm tra tính hợp lệ của tài liệu JSON dựa trên schema đã định nghĩa. OAS sử dụng một tập con được điều chỉnh của JSON Schema.

**3.3.1 Mô tả tham số truy vấn** (Describing query parameters)

Tham số truy vấn (query parameter) được thêm vào sau dấu `?` trong URL. Để mô tả chúng, bạn thêm thuộc tính `parameters` vào thao tác. `parameters` là một danh sách. Mỗi mục trong danh sách mô tả một tham số với ít nhất các thuộc tính `name`, `in` (vị trí, ở đây là `query`), và `schema` (mô tả cấu trúc dữ liệu). Các thuộc tính `required` (bắt buộc hay không) và `description` là tùy chọn.

Ví dụ, mô tả tham số truy vấn `free-query` cho thao tác `GET /products`:

```yaml
paths:
    /products:
        description: The products catalog
        get: # Thêm thao tác GET
            summary: Search for products # Tóm tắt
            description: | # Mô tả chi tiết (multiline)
                Search for products in catalog
                using a free query parameter
            parameters:
                - name: free-query # Tên tham số
                description: |
                    A product's name, reference, or partial description
                in: query # Vị trí: tham số truy vấn
                required: false # Không bắt buộc
                schema:
                    type: string # Kiểu dữ liệu: chuỗi
            responses: # Các phản hồi có thể
                "200": # Mã trạng thái 200 OK
                    description: |
                        Products matching free query parameter
        post: # Thêm thao tác POST
            summary: Add product
            description: |
                Add product (described in product info
                parameter) to catalog
            responses:
                "200":
                    description: |
                        Product added to catalog
```


**3.3.2 Mô tả dữ liệu bằng JSON Schema** (Describing data with JSON Schema)

Bạn sử dụng JSON Schema để mô tả cấu trúc dữ liệu của các khái niệm (concept) như sản phẩm. Để mô tả một đối tượng, sử dụng `type: object` và liệt kê các thuộc tính (`properties`) bên dưới. Mỗi thuộc tính có tên và kiểu (`type`).

Ví dụ, mô tả một sản phẩm cơ bản:

```yaml
type: object
properties:
    reference:
        type: string
    name:
        type: string
    price:
        type: number
```


Để chỉ định các thuộc tính **bắt buộc** (`required`), bạn thêm một danh sách `required` vào đối tượng. Bất kỳ thuộc tính nào có tên trong danh sách này là bắt buộc.

Ví dụ, sản phẩm với các thuộc tính `reference`, `name`, `price` là bắt buộc và `description` là tùy chọn:

```yaml
type: object
required: # Danh sách thuộc tính bắt buộc
    - reference
    - name
    - price
properties:
    reference:
        type: string
    name:
        type: string
    price:
        type: number
    description: # Thuộc tính tùy chọn vì không có trong danh sách required
        type: string
```


Bạn có thể thêm `description` cho đối tượng và từng thuộc tính, và `example` cho từng thuộc tính để tài liệu rõ ràng hơn. JSON Schema cũng hỗ trợ mô tả các thuộc tính phức tạp như đối tượng lồng nhau hoặc mảng.

Ví dụ, sản phẩm có thêm thuộc tính `supplier` (đối tượng):

```yaml
type: object
description: A product
required:
    - reference
    - name
    - price
    - supplier # supplier là bắt buộc
properties:
    reference:
        type: string
        description: Product's unique identifier
        example: ISBN-9781617295102
    name:
        type: string
        example: The Design of Web APIs
    price:
        type: number
        example: 44.99
    description:
        type: string
        example: A book about API design
    supplier: # Thuộc tính đối tượng lồng nhau
        type: object
        description: Product's supplier
        required: # Các thuộc tính trong supplier cũng có thể là bắt buộc
            - reference
            - name
        properties:
            reference:
                type: string
                description: Supplier's unique identifier
                example: MANPUB
            name:
                type: string
                example: Manning Publications
```


**3.3.3 Mô tả phản hồi** (Describing responses)

Dữ liệu trả về trong thân phản hồi HTTP được định nghĩa trong thuộc tính `content` của phản hồi. Bạn phải chỉ định **kiểu media** (`media type`) của tài liệu trả về (ví dụ: `application/json`). Sau đó, bạn mô tả schema của nội dung đó bằng JSON Schema.

Ví dụ, thao tác `GET /products` trả về một mảng các sản phẩm:

```yaml
responses:
    "200":
        description: Products matching free query
        content: # Nội dung phản hồi
            application/json: # Kiểu media
                schema: # Schema nội dung
                    type: array # Là một mảng
                    items: # Các mục trong mảng có schema là...
                        required:
                            - reference
                            - name
                            - price
                            - supplier
                        properties:
                            # Mô tả các thuộc tính của sản phẩm tương tự ví dụ trước
                            reference:
                                type: string
                            name:
                                type: string
                            price:
                                type: number
                            supplier:
                                type: object
                                # ... chi tiết thuộc tính supplier ...
```


**3.3.4 Mô tả tham số thân yêu cầu** (Describing body parameters)

Tham số thân yêu cầu (body parameter) được mô tả trong thuộc tính `requestBody` của thao tác. Giống như thân phản hồi, nó có kiểu media (`content`) và nội dung được mô tả bằng JSON Schema (`schema`).

Ví dụ, mô tả thân yêu cầu cho thao tác `POST /products` (thêm sản phẩm):

```yaml
post:
    summary: Add product
    description: Add product (described in product info parameter) to catalog
    requestBody: # Tham số thân yêu cầu
        description: Product's information
        content: # Nội dung thân yêu cầu
            application/json: # Kiểu media
                schema: # Schema nội dung
                    required: # Các thuộc tính bắt buộc để thêm sản phẩm
                        - name
                        - price
                        - supplierReference
                    properties: # Mô tả các thuộc tính
                        name:
                            type: string
                        price:
                            type: number
                        description:
                            type: string
                        supplierReference: # Tham chiếu nhà cung cấp (không phải đối tượng đầy đủ)
                            type: string
    responses:
        "200":
            description: Product added to catalog
            # ... mô tả thân phản hồi thành công ...
```


**4. Mô tả API hiệu quả bằng OAS** (Describing an API effciently with OAS)

Có hai điều cơ bản cần biết để viết tài liệu OAS hiệu quả:

**4.4.1 Tái sử dụng các thành phần** (Reusing components)

Để tránh mô tả đi mô tả lại cùng một thứ (ví dụ: schema sản phẩm được sử dụng trong cả yêu cầu thêm và phản hồi tìm kiếm), OAS cho phép **mô tả các thành phần có thể tái sử dụng** như schemas, parameters, responses, v.v. trong phần `components` ở cấp gốc của tài liệu OAS.

Ví dụ, mô tả schema sản phẩm trong `components.schemas`:

```yaml
openapi: "3.0.0"
info:
    # ...
components: # Phần định nghĩa các thành phần tái sử dụng
    schemas: # Định nghĩa các schema tái sử dụng
        product: # Tên schema tái sử dụng
            type: object
            description: A product
            required:
                # ... các thuộc tính bắt buộc ...
            properties:
                # ... mô tả các thuộc tính ...
```


Sau đó, bạn có thể **tham chiếu** đến schema đã định nghĩa bằng cách sử dụng `$ref`.

Ví dụ, sử dụng `$ref` cho schema thân phản hồi của thao tác `POST /products`:

```yaml
post:
    summary: Add product
    description: Add product to catalog
    # ... requestBody ...
    responses:
        "200":
            description: Product added to catalog
            content:
                application/json:
                    schema:
                        $ref: "#/components/schemas/product" # Tham chiếu đến schema product đã định nghĩa
```


Cú pháp `$ref: "#/components/schemas/product"` là một **tham chiếu JSON** đến thành phần cục bộ trong tài liệu OAS.

**4.4.2 Mô tả tham số đường dẫn** (Describing path parameters)

Đối với các tài nguyên có đường dẫn chứa biến (ví dụ: `/products/{productId}`), biến `{productId}` là một **tham số đường dẫn** (path parameter).

Để mô tả nó, bạn thêm đường dẫn tài nguyên với tham số đường dẫn vào `paths`. Tham số đường dẫn sau đó được định nghĩa trong danh sách `parameters` của thao tác hoặc cấp tài nguyên. Đối với tham số đường dẫn, `in` phải là `path`, và `required` luôn là `true`. Tên tham số (`name`) trong danh sách `parameters` phải khớp với tên trong dấu ngoặc nhọn `{}` trên đường dẫn.

Ví dụ, mô tả thao tác `DELETE /products/{productId}`:

```yaml
paths:
    /products:
        # ... mô tả tài nguyên collection ...
    /products/{productId}: # Đường dẫn tài nguyên với tham số
        description: A product
        delete: # Thao tác DELETE
            summary: Delete a product
            parameters: # Danh sách tham số
                - name: productId # Tên tham số (khớp với {productId})
                  in: path # Vị trí: tham số đường dẫn
                  required: true # Bắt buộc
                  description: Product's reference # Mô tả
                  schema:
                    type: string # Kiểu dữ liệu
```


Tham số đường dẫn cũng có thể được định nghĩa là thành phần có thể tái sử dụng trong `components.parameters` và tham chiếu bằng `$ref`. Hoặc, nếu tham số đường dẫn được sử dụng bởi nhiều thao tác trên cùng một tài nguyên, nó có thể được định nghĩa một lần ở cấp tài nguyên thay vì lặp lại trong mỗi thao tác.

Ví dụ, định nghĩa tham số `productId` ở cấp tài nguyên `/products/{productId}` và tham chiếu từ thao tác `delete`:

```yaml
paths:
    /products:
        # ...
    /products/{productId}: # Tài nguyên với tham số đường dẫn
        parameters: # Tham số định nghĩa ở cấp tài nguyên
            - $ref: "#/components/parameters/productId" # Tham chiếu đến tham số reusable
        delete:
            # summary, description, responses ...
            # KHÔNG cần định nghĩa lại parameters ở đây
        put:
            # ...
        patch:
            # ...
```


**Tóm tắt**

Chương này kết thúc phần đầu tiên của cuốn sách, giúp bạn có được bộ kỹ năng thiết kế API cơ bản. Bạn đã học được:
*   API thực sự là gì.
*   Cách xác định mục tiêu của nó từ góc độ người dùng.
*   Cách chuyển đổi mục tiêu thành biểu diễn có thể lập trình.
*   **Cách mô tả chính thức biểu diễn có thể lập trình này bằng OAS**.

Các điểm chính cần nhớ:
*   Định dạng mô tả API là một cách đơn giản, có cấu trúc để mô tả và chia sẻ giao diện lập trình.
*   Tài liệu mô tả API là tài liệu máy có thể đọc, có thể dùng để tạo tài liệu tham khảo API, v.v..
*   Bạn chỉ nên sử dụng định dạng mô tả API khi thiết kế biểu diễn có thể lập trình và dữ liệu, không phải trước đó.
*   Luôn tận dụng các tính năng tài liệu của định dạng mô tả API và tìm hiểu sâu về nó để sử dụng hiệu quả, đặc biệt là định nghĩa các thành phần có thể tái sử dụng.

Hy vọng phần trình bày chi tiết này hữu ích cho bạn!