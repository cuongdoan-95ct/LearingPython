Bài viết tại [Random UUIDs Are Killing Your PostgreSQL Performance: How to Fix It](https://medium.com/@shaileshkumarmishra/random-uuids-are-killing-your-postgresql-performance-how-to-fix-it-d8f7aaa0b2c5) của Shailesh Kumar Mishra giải thích vấn đề hiệu suất khi sử dụng UUID ngẫu nhiên (UUIDv4) làm khóa chính trong PostgreSQL và đề xuất các giải pháp để khắc phục. Dưới đây là tóm tắt nội dung bằng tiếng Việt, giải thích chi tiết các ý chính để bạn dễ hiểu:

---

### **1. Vấn đề: UUIDv4 ngẫu nhiên gây ảnh hưởng đến hiệu suất**
UUID (Universally Unique Identifier) là một lựa chọn phổ biến để tạo định danh duy nhất trong các hệ thống phân tán, chẳng hạn như microservices hoặc khi cần hợp nhất dữ liệu từ nhiều nguồn. PostgreSQL cung cấp hàm `gen_random_uuid()` để tạo UUIDv4, là loại UUID ngẫu nhiên. Tuy nhiên, tính ngẫu nhiên này gây ra các vấn đề hiệu suất nghiêm trọng khi sử dụng làm khóa chính:

- **Tính ngẫu nhiên phá vỡ dữ liệu cục bộ (data locality)**:
  - UUIDv4 là hoàn toàn ngẫu nhiên, nghĩa là các giá trị không có thứ tự. Khi được sử dụng làm khóa chính, các bản ghi mới được chèn vào chỉ mục B-tree (chỉ mục mặc định của PostgreSQL) ở các vị trí ngẫu nhiên.
  - Điều này dẫn đến việc các trang chỉ mục (index pages) phải được tải và ghi liên tục trên toàn bộ chỉ mục, làm tăng I/O đĩa và giảm tỷ lệ trúng bộ đệm (cache hit ratio).
  - Ngược lại, với khóa chính tăng dần (như `BIGINT` hoặc `SERIAL`), các bản ghi mới được chèn vào cuối chỉ mục, giúp tận dụng bộ đệm hiệu quả hơn.

- **Tăng tải I/O và phân mảnh chỉ mục**:
  - Vì các giá trị UUIDv4 được phân bố ngẫu nhiên, PostgreSQL phải thực hiện nhiều thao tác chia trang (page splits) trong chỉ mục B-tree để chèn dữ liệu mới. Điều này gây ra phân mảnh chỉ mục (index bloat) và tăng lượng WAL (Write-Ahead Log) được tạo ra.
  - Kết quả là hiệu suất chèn (INSERT) giảm đáng kể, đặc biệt khi bảng trở nên lớn hoặc kích thước chỉ mục vượt quá bộ nhớ RAM.

- **Hiệu suất truy vấn (SELECT) cũng bị ảnh hưởng**:
  - Các truy vấn tìm kiếm hoặc nối (JOIN) dựa trên khóa chính UUIDv4 yêu cầu truy cập nhiều trang chỉ mục ngẫu nhiên, dẫn đến việc đọc đĩa nhiều hơn so với khóa chính tăng dần, nơi các bản ghi gần nhau về thời gian thường nằm trên cùng một trang.

---

### **2. Tại sao UUIDv4 vẫn được sử dụng?**
Mặc dù có các vấn đề về hiệu suất, UUIDv4 vẫn phổ biến vì:
- **Tính duy nhất toàn cục**: UUID đảm bảo định danh duy nhất mà không cần phối hợp giữa các hệ thống, rất hữu ích trong các ứng dụng phân tán.
- **Dễ sử dụng**: Hàm `gen_random_uuid()` tích hợp sẵn trong PostgreSQL (từ phiên bản 13) giúp tạo UUID dễ dàng.
- **Khả năng chống suy đoán**: UUID ngẫu nhiên khó đoán hơn so với ID tăng dần, giúp tăng cường bảo mật trong một số trường hợp.

Tuy nhiên, bài viết nhấn mạnh rằng cái giá phải trả về hiệu suất có thể rất lớn khi dữ liệu mở rộng quy mô.

---

### **3. Giải pháp khắc phục**
Để giải quyết các vấn đề hiệu suất của UUIDv4, bài viết đề xuất các giải pháp sau:

#### **Giải pháp 1: Sử dụng UUID tuần tự (Sequential UUIDs)**
- **Ý tưởng**: Thay vì sử dụng UUIDv4 ngẫu nhiên, hãy sử dụng các biến thể UUID có thứ tự, chẳng hạn như **UUIDv7** hoặc **UUIDv1**, kết hợp thành phần thời gian (timestamp) để tạo các giá trị tăng dần.
- **Lợi ích**:
  - Các UUID tuần tự được chèn vào chỉ mục B-tree theo thứ tự, giống như khóa chính tăng dần, giúp cải thiện dữ liệu cục bộ và giảm I/O.
  - Giảm phân mảnh chỉ mục và số lần chia trang.
  - Tăng tỷ lệ trúng bộ đệm, đặc biệt với các truy vấn liên quan đến dữ liệu gần đây.
- **Cách triển khai**:
  - **UUIDv7**: Là một chuẩn mới (dự kiến được hỗ trợ trong PostgreSQL 17), kết hợp timestamp với các bit ngẫu nhiên để tạo UUID có thứ tự. Bạn có thể sử dụng các extension hoặc triển khai tùy chỉnh trong ứng dụng.
  - **UUIDv1**: Dựa trên timestamp và địa chỉ MAC, có thể tạo bằng extension `uuid-ossp` với hàm `uuid_generate_v1mc()`. Ví dụ:
    ```sql
    CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
    CREATE TABLE your_table (
        id UUID DEFAULT uuid_generate_v1mc() PRIMARY KEY,
        -- Các cột khác
    );
    ```
  - UUIDv1 và UUIDv7 đảm bảo các giá trị gần nhau về thời gian được lưu trữ gần nhau trong chỉ mục, cải thiện hiệu suất chèn và truy vấn.

#### **Giải pháp 2: Sử dụng BIGINT thay vì UUID**
- **Ý tưởng**: Nếu không bắt buộc phải sử dụng UUID, hãy chuyển sang sử dụng `BIGINT` hoặc `SERIAL` làm khóa chính.
- **Lợi ích**:
  - `BIGINT` chỉ chiếm 8 byte, nhỏ hơn nhiều so với UUID (16 byte), giúp giảm dung lượng lưu trữ và I/O.
  - Các giá trị tăng dần của `BIGINT` tối ưu cho chỉ mục B-tree, giảm phân mảnh và cải thiện hiệu suất chèn và truy vấn.
  - Phù hợp với các ứng dụng không yêu cầu tính duy nhất toàn cục hoặc không hoạt động trong môi trường phân tán.
- **Cách triển khai**:
  ```sql
  CREATE TABLE your_table (
      id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
      -- Các cột khác
  );
  ```
- **Nhược điểm**: Không phù hợp nếu bạn cần định danh duy nhất toàn cục hoặc nếu ứng dụng phân tán yêu cầu tạo ID mà không cần phối hợp.

#### **Giải pháp 3: Sử dụng UUID làm khóa phụ (Secondary Key)**
- **Ý tưởng**: Sử dụng `BIGINT` hoặc `SERIAL` làm khóa chính để tối ưu hiệu suất, và thêm một cột UUID làm khóa phụ để đáp ứng yêu cầu định danh duy nhất toàn cục.
- **Lợi ích**:
  - Kết hợp ưu điểm của cả hai: hiệu suất cao của `BIGINT` và tính duy nhất của UUID.
  - Giảm tác động của UUID ngẫu nhiên lên chỉ mục khóa chính.
- **Cách triển khai**:
  ```sql
  CREATE TABLE your_table (
      id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
      uuid UUID DEFAULT gen_random_uuid() UNIQUE,
      -- Các cột khác
  );
  ```
  - Cột `uuid` được đánh chỉ mục `UNIQUE` để đảm bảo tính duy nhất, nhưng không phải khóa chính, nên tác động đến hiệu suất thấp hơn.

#### **Giải pháp 4: Tối ưu hóa chỉ mục và cấu hình**
- **Tối ưu hóa chỉ mục**:
  - Định kỳ chạy `REINDEX` để giảm phân mảnh chỉ mục khi sử dụng UUIDv4:
    ```sql
    REINDEX TABLE your_table;
    ```
  - Sử dụng các chỉ mục khác như BRIN (Block Range Index) cho các bảng lớn, mặc dù BRIN không phù hợp với khóa chính.
- **Tăng bộ đệm chia sẻ (shared buffers)**:
  - Tăng giá trị `shared_buffers` trong cấu hình PostgreSQL để giữ nhiều trang chỉ mục hơn trong RAM, giảm I/O đĩa.
  - Ví dụ: Chỉnh sửa `postgresql.conf`:
    ```conf
    shared_buffers = 4GB
    ```
- **Giám sát và bảo trì**:
  - Sử dụng công cụ như `pg_stat_statements` để theo dõi hiệu suất truy vấn.
  - Thường xuyên chạy `VACUUM` và `ANALYZE` để tối ưu hóa bảng và chỉ mục:
    ```sql
    VACUUM ANALYZE your_table;
    ```

---

### **4. So sánh hiệu suất**
Bài viết cung cấp một số kết quả thử nghiệm (benchmark) để minh họa sự khác biệt về hiệu suất:
- **Chèn (INSERT)**:
  - Với UUIDv4: Hiệu suất giảm đáng kể khi bảng lớn hơn, do phân mảnh chỉ mục và I/O ngẫu nhiên.
  - Với UUIDv7 hoặc BIGINT: Nhanh hơn gấp 2-3 lần so với UUIDv4, đặc biệt trên các bảng lớn.
- **Truy vấn (SELECT)**:
  - UUIDv4 làm tăng thời gian truy vấn do phải truy cập nhiều trang chỉ mục ngẫu nhiên.
  - UUIDv7 và BIGINT cải thiện hiệu suất truy vấn nhờ dữ liệu cục bộ tốt hơn.
- **Kích thước chỉ mục**:
  - UUID (16 byte) tạo ra chỉ mục lớn hơn so với BIGINT (8 byte), dẫn đến tiêu tốn nhiều RAM và đĩa hơn.

---

### **5. Khi nào nên sử dụng UUID?**
- **Sử dụng UUID**:
  - Trong hệ thống phân tán, nơi cần định danh duy nhất toàn cục mà không cần phối hợp.
  - Khi bảo mật là ưu tiên, vì UUID ngẫu nhiên khó đoán hơn ID tăng dần.
  - Khi hợp nhất dữ liệu từ nhiều nguồn, UUID giúp tránh xung đột.
- **Không sử dụng UUID**:
  - Trong các ứng dụng có khối lượng chèn lớn hoặc truy vấn thường xuyên trên dữ liệu gần đây.
  - Nếu kích thước bảng lớn và hiệu suất là ưu tiên hàng đầu, hãy cân nhắc BIGINT hoặc UUID tuần tự.

---

### **6. Kết luận**
- UUIDv4 ngẫu nhiên tiện lợi nhưng gây ra các vấn đề hiệu suất nghiêm trọng trong PostgreSQL do tính ngẫu nhiên làm gián đoạn dữ liệu cục bộ và tăng phân mảnh chỉ mục.
- Các giải pháp như sử dụng **UUIDv7**, **BIGINT**, hoặc UUID làm khóa phụ giúp cân bằng giữa hiệu suất và tính duy nhất.
- Việc tối ưu hóa cấu hình (shared buffers, REINDEX, VACUUM) cũng rất quan trọng để giảm tác động của UUID.

---

### **Lời khuyên thực tế**
- Nếu bạn đang sử dụng UUIDv4 trong ứng dụng Django với PostgreSQL, hãy kiểm tra xem bảng của bạn có bị ảnh hưởng bởi hiệu suất chèn chậm hoặc truy vấn kém hiệu quả không.
- Cân nhắc chuyển sang UUIDv7 (nếu PostgreSQL hỗ trợ) hoặc BIGINT nếu ứng dụng không yêu cầu UUID.
- Luôn giám sát hiệu suất cơ sở dữ liệu bằng các công cụ như `EXPLAIN ANALYZE` hoặc `pg_stat_statements` để phát hiện sớm các vấn đề.

Nếu bạn muốn tìm hiểu sâu hơn về một phần cụ thể (ví dụ: cách triển khai UUIDv7, tối ưu hóa chỉ mục, hoặc tích hợp với Django), hãy cho tôi biết để tôi giải thích chi tiết hơn![](https://medium.com/%40shaileshkumarmishra/random-uuids-are-killing-your-postgresql-performance-how-to-fix-it-d8f7aaa0b2c5)