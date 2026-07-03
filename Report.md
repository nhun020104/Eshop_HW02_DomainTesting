# BÁO CÁO BÀI TẬP HW02 - DOMAIN TESTING TRÊN ESHOP

## 1. Thông tin chung
- Học phần: Kiểm thử phần mềm
- Bài tập: HW02 - Domain Testing & BVA
- Sinh viên thực hiện: Nguyễn Hoàng Uyển Như - 22127315

---

## 2. Kế hoạch kiểm thử tính năng

### Tính năng FR-14: Quản lý Danh mục (Web Admin)

#### a. Áp dụng kỹ thuật Kiểm thử miền (Domain Testing)

## 1. Xác định biến đầu vào, kiểu dữ liệu và ràng buộc

Dựa trên đặc tả FR-14 và các quy tắc chung của hệ thống (như tính nhất quán ngôn ngữ tiếng Việt), chúng ta xác định được biến đầu vào như sau:

* **Tên biến:** `category_name` (Tên danh mục)
* **Kiểu dữ liệu:** Chuỗi ký tự (String)
* **Ràng buộc theo đặc tả:** * Bắt buộc nhập (Không được để trống hoặc chỉ chứa khoảng trắng).
* Mặc dù FR-14 không ghi rõ độ dài tối đa (như FR-15 quy định 255 ký tự cho sản phẩm), nhưng theo thực tế kiểm thử miền và thiết kế cơ sở dữ liệu (SQLite ở backend), ta sẽ giả định một giới hạn biên kỹ thuật thông thường cho chuỗi ngắn (ví dụ: tối đa 255 ký tự) để kiểm thử an toàn toàn diện.



---

## 2. Phân chia các miền/vùng tương đương (Equivalence Partitioning)

Đối với biến kiểu chuỗi (`String`), ta cần phân tích dựa trên 3 khía cạnh chính: **Độ dài chuỗi**, **Ký tự hợp lệ**, và **Ký tự đặc biệt/Bảo mật**.

### Khía cạnh 1: Độ dài chuỗi (String Length)

| Mã vùng | Loại | Mô tả vùng tương đương | Ý nghĩa kiểm thử |
| --- | --- | --- | --- |
| **V1** | Valid | Chuỗi có độ dài hợp lệ từ 1 đến 255 ký tự | Kiểm tra khả năng lưu trữ thông thường. |
| **IV1** | Invalid | Chuỗi trống (Empty - không nhập gì) | Kiểm tra ràng buộc "bắt buộc nhập". |
| **IV2** | Invalid | Chuỗi chỉ chứa khoảng trắng (`"   "`) | Tránh trường hợp bypass qua mặt validation bằng dấu cách. |
| **IV3** | Invalid | Chuỗi vượt quá giới hạn (ví dụ: 256 ký tự trở lên) | Kiểm tra tràn bộ đệm hoặc lỗi cắt chuỗi ở DB. |

### Khía cạnh 2: Loại ký tự (Character Types)

Do hệ thống EShop phục vụ người dùng Việt Nam (FR-21 yêu cầu tiếng Việt), việc xử lý bảng mã và ký tự là rất quan trọng.

| Mã vùng | Loại | Mô tả vùng tương đương | Ý nghĩa kiểm thử |
| --- | --- | --- | --- |
| **V2** | Valid | Chuỗi chữ cái tiếng Việt có dấu thông thường (Ví dụ: `"Điện tử"`, `"Thời trang"`) | Đảm bảo hệ thống hiển thị và lưu đúng bảng mã UTF-8. |
| **V3** | Valid | Chuỗi chứa cả chữ và số (Ví dụ: `"Đồ gia dụng 2k"`) | Đảm bảo chấp nhận ký tự số. |
| **V4** | Valid | Chuỗi có chứa ký tự đặc biệt thông thường/dấu câu (Ví dụ: `"Sách & Truyện tranh"`, `"Mẹ & Bé"`) | Các ký tự như `&`, `-`, `/` thường xuất hiện trong tên danh mục thực tế. |

### Khía cạnh 3: Bảo mật và Lỗi hệ thống (Security & Edge Cases)

Dựa vào yêu cầu bảo mật chung của hệ thống (SEC-04: Escape dữ liệu UI, SEC-05: Parameterized Query), ta cần các vùng invalid để test khả năng chống lỗi.

| Mã vùng | Loại | Mô tả vùng tương đương | Ý nghĩa kiểm thử |
| --- | --- | --- | --- |
| **IV4** | Invalid | Chuỗi chứa mã script (Ví dụ: `<script>alert(1)</script>`) | Kiểm tra lỗ hổng XSS (Cross-Site Scripting) khi danh mục hiển thị trên giao diện. |
| **IV5** | Invalid | Chuỗi chứa ký tự điều khiển/SQL Injection (Ví dụ: `' OR '1'='1`) | Kiểm tra xem backend có dùng Parameterized Query chuẩn không. |
| **IV6** | Invalid | Dữ liệu có kiểu `null` hoặc `undefined` (gửi trực tiếp qua API thay vì chuỗi) | Kiểm tra độ nghiêm ngặt của việc validate ở tầng Backend API. |


---

## 3. Ma trận kiểm thử miền (Domain Test Matrix)

Ma trận dưới đây tích hợp đầy đủ tất cả các phân vùng hợp lệ (`V1` - `V4`) và không hợp lệ (`IV1` - `IV6`) nhằm đảm bảo độ bao phủ kiểm thử cao nhất ở tầng giao diện và logic hệ thống.

| Test Case ID | Tên kịch bản kiểm thử | Vùng bao phủ | Loại test | Kết quả mong đợi (Expected Result) |
| --- | --- | --- | --- | --- |
| **TC_FR14_DOM_01** | Thêm danh mục với tên thông thường hợp lệ | `V1` | Valid | Tạo danh mục thành công, hiển thị trong danh sách. |
| **TC_FR14_DOM_02** | Thêm danh mục với chữ Tiếng Việt có dấu | `V2` | Valid | Lưu đúng định dạng UTF-8, không bị lỗi font chữ. |
| **TC_FR14_DOM_03** | Thêm danh mục gồm cả chữ và số | `V3` | Valid | Tạo danh mục thành công, hiển thị chính xác chữ số. |
| **TC_FR14_DOM_04** | Thêm danh mục chứa ký tự đặc biệt thông thường | `V4` | Valid | Hệ thống chấp nhận, hiển thị đúng ký tự và không lỗi định tuyến URL. |
| **TC_FR14_DOM_05** | Thêm danh mục để trống hoàn toàn (Empty) | `IV1` | Invalid | Hệ thống chặn lại, hiển thị thông báo lỗi yêu cầu nhập. |
| **TC_FR14_DOM_06** | Thêm danh mục chỉ chứa toàn khoảng trắng | `IV2` | Invalid | Hệ thống tự động cắt chuỗi (trim), nhận diện là trống và báo lỗi. |
| **TC_FR14_DOM_07** | Thêm danh mục vượt quá giới hạn kỹ thuật (256+ ký tự) | `IV3` | Invalid | Hệ thống chặn ở giao diện hoặc backend, báo lỗi chuỗi quá dài. |
| **TC_FR14_DOM_08** | Bơm mã độc Script (XSS) vào tên danh mục | `IV4` | Invalid | Hệ thống mã hóa (escape) chuỗi, hiển thị dạng văn bản thuần, không thực thi mã. |
| **TC_FR14_DOM_09** | Bơm câu lệnh SQL Injection vào tên danh mục | `IV5` | Invalid | Hệ thống xử lý an toàn dưới dạng chuỗi thường, không gây lỗi cú pháp DB. |
| **TC_FR14_DOM_10** | Gửi giá trị Null/Undefined qua dữ liệu API | `IV6` | Invalid | Tầng Backend API trả về mã lỗi `400 Bad Request`, không làm sập server. |

---

## 2. Danh sách Test Cases chi tiết (Chi tiết hóa từng kịch bản)

### 🔹 Nhóm 1: Các trường hợp kiểm thử hợp lệ (Valid Test Cases)

* **TC_FR14_DOM_01: Thêm danh mục với tên thông thường hợp lệ**
* **Các bước thực hiện:**
1. Đăng nhập vào Web Admin với quyền Administrator.
2. Điều hướng tới menu "Quản lý Danh mục".
3. Nhập dữ liệu vào ô "Tên danh mục".
4. Nhấn nút "Thêm mới".


* **Dữ liệu đầu vào:** `category_name = "Thoi trang"` (Độ dài: 9 ký tự)
* **Kết quả mong đợi:** Danh mục được thêm thành công, hệ thống hiển thị thông báo thành công và danh mục xuất hiện ở bảng danh sách phía dưới.


* **TC_FR14_DOM_02: Thêm danh mục với chữ Tiếng Việt có dấu**
* **Các bước thực hiện:** Làm tương tự như các bước của TC_FR14_DOM_01.
* **Dữ liệu đầu vào:** `category_name = "Điện tử & Gia dụng"`
* **Kết quả mong đợi:** Thêm thành công, chữ hiển thị rõ ràng, chuẩn Unicode UTF-8, không bị lỗi hiển thị thành các ký tự lạ (``, `?`).


* **TC_FR14_DOM_03: Thêm danh mục gồm cả chữ và số**
* **Các bước thực hiện:** Làm tương tự các bước của TC_FR14_DOM_01.
* **Dữ liệu đầu vào:** `category_name = "Laptop Gaming 2026"`
* **Kết quả mong đợi:** Thêm thành công, lưu trữ và hiển thị đầy đủ cả ký tự chữ và số.


* **TC_FR14_DOM_04: Thêm danh mục chứa ký tự đặc biệt thông thường**
* **Các bước thực hiện:** Làm tương tự các bước của TC_FR14_DOM_01.
* **Dữ liệu đầu vào:** `category_name = "Mẹ / Bé & Đồ Chơi"`
* **Kết quả mong đợi:** Thêm thành công. Khi bấm xem chi tiết hoặc chuyển trang theo danh mục này, hệ thống không bị lỗi điều hướng URL (Routing Error).



### 🔹 Nhóm 2: Các trường hợp kiểm thử không hợp lệ (Invalid Test Cases)

* **TC_FR14_DOM_05: Thêm danh mục để trống hoàn toàn (Empty)**
* **Các bước thực hiện:**
1. Vào trang "Quản lý Danh mục".
2. Để trống ô nhập "Tên danh mục".
3. Nhấn nút "Thêm mới".


* **Dữ liệu đầu vào:** `category_name = ""`
* **Kết quả mong đợi:** Hệ thống không thực hiện gửi biểu mẫu, chặn ngay tại giao diện và hiển thị thông báo lỗi màu đỏ: *"Tên danh mục là bắt buộc và không được để trống"*.


* **TC_FR14_DOM_06: Thêm danh mục chỉ chứa toàn khoảng trắng**
* **Các bước thực hiện:**
1. Vào trang "Quản lý Danh mục".
2. Nhập 5 dấu cách vào ô "Tên danh mục".
3. Nhấn nút "Thêm mới".


* **Dữ liệu đầu vào:** `category_name = "     "`
* **Kết quả mong đợi:** Hệ thống tự động cắt bỏ khoảng trắng thừa (trim). Nhận diện đây là chuỗi rỗng và hiển thị thông báo lỗi yêu cầu nhập tên danh mục, tuyệt đối không tạo một danh mục vô hình.


* **TC_FR14_DOM_07: Thêm danh mục vượt quá giới hạn kỹ thuật (256+ ký tự)**
* **Các bước thực hiện:**
1. Tạo một chuỗi ngẫu nhiên dài 260 ký tự.
2. Nhập chuỗi này vào ô "Tên danh mục" và bấm "Thêm mới".


* **Dữ liệu đầu vào:** `category_name = "A...A"` (Chuỗi lặp lại chữ A gồm 260 ký tự)
* **Kết quả mong đợi:** Ô nhập liệu chặn không cho gõ tiếp khi đạt ngưỡng tối đa (nếu dùng thuộc tính `maxlength`), hoặc khi bấm Thêm, hệ thống báo lỗi: *"Tên danh mục không được vượt quá 255 ký tự"*.


* **TC_FR14_DOM_08: Bơm mã độc Script (XSS Injection)**
* **Các bước thực hiện:** Nhập đoạn mã script độc hại vào ô tên danh mục và nhấn Thêm mới. Sau đó tải lại trang xem danh mục hiển thị.
* **Dữ liệu đầu vào:** `category_name = "<script>alert('XSS')</script>"`
* **Kết quả mong đợi:** Hệ thống lưu trữ an toàn. Khi hiển thị lên danh sách, nó hiển thị nguyên văn chuỗi `<script>alert('XSS')</script>`, hoàn toàn không kích hoạt hộp thoại cảnh báo (Alert pop-up) của trình duyệt.


* **TC_FR14_DOM_09: Bơm câu lệnh tấn công SQL Injection**
* **Các bước thực hiện:** Nhập chuỗi phá vỡ logic truy vấn SQL vào ô nhập liệu và bấm Thêm mới.
* **Dữ liệu đầu vào:** `category_name = "Thời trang' OR '1'='1"`
* **Kết quả mong đợi:** Hệ thống xử lý chuỗi bằng cơ chế an toàn, thêm danh mục có tên là `Thời trang' OR '1'='1'`. Cơ sở dữ liệu không bị lỗi cú pháp và không làm lộ dữ liệu.


* **TC_FR14_DOM_10: Gửi giá trị Null/Undefined qua dữ liệu API**
* **Các bước thực hiện:** (Thực hiện kiểm thử bằng Postman hoặc gửi request trực tiếp đến Endpoint backend thay vì dùng giao diện UI).
* **Dữ liệu đầu vào:** Gửi request POST đến `/api/categories` với body json: `{ "category_name": null }` hoặc bỏ hẳn trường này ra khỏi body.
* **Kết quả mong đợi:** Backend API chặn đứng yêu cầu, trả về HTTP Status Code `400 Bad Request` kèm theo thông điệp lỗi dạng JSON, không xảy ra lỗi sập luồng xử lý của Server (Internal Server Error 500).



---

#### b. Áp dụng kỹ thuật Phân tích giá trị biên (Boundary Value Analysis)

## 1. Xác định các điểm biên 3 điểm (3-point BVA Points)

Dải độ dài hợp lệ theo đặc tả quy định là từ **1 đến 255 ký tự**. Ta có hai mốc biên:

### Biên dưới (Mốc 1 ký tự)

* **On (Tại biên):** `1` ký tự $\rightarrow$ **Valid** (Độ dài tối thiểu được chấp nhận).
* **Off (Sát ngoài biên dưới):** `0` ký tự (Chuỗi rỗng) $\rightarrow$ **Invalid** (Hệ thống phải chặn vì bắt buộc nhập).
* **In (Sát trong biên dưới):** `2` ký tự $\rightarrow$ **Valid** (Độ dài tối thiểu + 1).

### Biên trên (Mốc 255 ký tự)

* **On (Tại biên):** `255` ký tự $\rightarrow$ **Valid** (Độ dài tối đa được chấp nhận).
* **Off (Sát ngoài biên trên):** `256` ký tự $\rightarrow$ **Invalid** (Vượt quá giới hạn lưu trữ tối đa).
* **In (Sát trong biên trên):** `254` ký tự $\rightarrow$ **Valid** (Độ dài tối đa - 1).

---

## 2. Bảng BVA Test Cases chi tiết (Web Admin)

Bảng dưới đây thiết kế các test case bám sát các yêu cầu giao diện của hệ thống EShop (ngôn ngữ Tiếng Việt, nút màu xanh dương, thông báo lỗi hiển thị **phía trên** nút submit theo quy định tại **FR-21** và **FR-22**).

| Test Case ID | Vùng biên kiểm thử | Dữ liệu đầu vào cụ thể (Độ dài chuỗi) | Loại Test | Kết quả mong đợi (Expected Result) |
| --- | --- | --- | --- | --- |
| **TC_FR14_BVA_01** | Biên dưới - **Off** | Chuỗi rỗng `""` (**0 ký tự**) | Invalid | Hệ thống chặn không gửi request, hiển thị thông báo lỗi bằng Tiếng Việt: *"Tên danh mục không được để trống"* ngay **phía trên** nút Lưu. |
| **TC_FR14_BVA_02** | Biên dưới - **On** | Chuỗi `"A"` (**1 ký tự**) | Valid | Thêm danh mục thành công, hiển thị Toast thông báo thành công và danh mục xuất hiện trong bảng danh sách. |
| **TC_FR14_BVA_03** | Biên dưới - **In** | Chuỗi `"Ab"` (**2 ký tự**) | Valid | Thêm danh mục thành công, dữ liệu lưu chính xác. |
| **TC_FR14_BVA_04** | Biên trên - **In** | Chuỗi gồm **254 ký tự** `"A...A"` | Valid | Thêm danh mục thành công, dữ liệu được lưu trọn vẹn và không bị cắt cụt (truncate) chữ. |
| **TC_FR14_BVA_05** | Biên trên - **On** | Chuỗi gồm **255 ký tự** `"A...A"` | Valid | Thêm danh mục thành công (đây là điểm giới hạn cuối cùng được cho phép). |
| **TC_FR14_BVA_06** | Biên trên - **Off** | Chuỗi gồm **256 ký tự** `"A...A"` | Invalid | Hệ thống chặn ở giao diện (không cho gõ tiếp nếu có `maxlength="255"`) hoặc hiển thị thông báo lỗi *"Tên danh mục không được vượt quá 255 ký tự"* phía trên nút Lưu khi bấm submit. |

---

#### c. Phân tích khoảng trống AI (AI Gap Analysis)

Khoảng trống 1: Lỗi logic nghiệp vụ - Trùng lặp dữ liệu (Business Logic - Duplicate Data): BVA và Domain Testing chỉ kiểm tra xem một chuỗi đơn lẻ nhập vào có hợp lệ hay không. Các kỹ thuật này hoàn toàn bỏ sót kịch bản: Nếu thêm mới một danh mục có tên trùng hoàn toàn với một danh mục đã tồn tại từ trước thì sao? Hệ thống thực tế có chặn trùng để bảo vệ toàn vẹn dữ liệu, hay sẽ tạo ra hai danh mục giống hệt nhau gây loạn luồng xử lý?

Khoảng trống 2: Lỗi trải nghiệm và Đồng bộ trạng thái giao diện (UI/UX & State Synchronization): Khi một danh mục được thêm mới thành công qua API Backend, giao diện Frontend (Web Admin) có tự động làm mới (re-render) để cập nhật danh mục đó vào bảng danh sách ngay lập tức không? Hay người dùng phải F5/tải lại trang thủ công thì mới thấy? Các kỹ thuật lý thuyết không thể dự đoán được hành vi phản hồi thời gian thực này của UI.