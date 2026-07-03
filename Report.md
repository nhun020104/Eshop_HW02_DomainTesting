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



---

### Tính năng FR-14: Quản lý Danh mục (Web Admin)

#### a. Áp dụng kỹ thuật Kiểm thử miền (Domain Testing)

## 1. Xác định biến đầu vào, kiểu dữ liệu và ràng buộc

Dựa trên đặc tả FR-04 và các quy tắc hệ thống thực tế, chúng ta có 2 biến đầu vào chính khi người dùng tiến hành cập nhật hồ sơ cá nhân:

1. **Biến `full_name` (Họ và tên):**
* **Kiểu dữ liệu:** Chuỗi ký tự (String).
* **Ràng buộc đặc tả:** Bắt buộc nhập.
* **Ràng buộc ngầm định:** Phải hỗ trợ tiếng Việt Unicode có dấu, có khoảng trắng giữa các từ. Để an toàn hệ thống, ta giả định giới hạn kỹ thuật tối đa là 100 ký tự.


2. **Biến `phone_number` (Số điện thoại):**
* **Kiểu dữ liệu:** Chuỗi số (String/Numeric string - vì số điện thoại thường có số 0 ở đầu nên lưu dạng chuỗi để tránh mất số 0).
* **Ràng buộc đặc tả:** Bắt buộc nhập, phải bắt đầu bằng số `0`, độ dài từ **10 đến 11 chữ số** (theo đúng text mô tả trong FR-04).



---

## 2. Phân chia các miền/vùng tương đương (Equivalence Partitioning)

Để đảm bảo độ bao phủ tuyệt đối, ta chia các phân vùng tương đương dựa trên 3 khía cạnh: Độ dài, Loại ký tự, và Bảo mật.

### Đối với biến `full_name` (Họ và tên)

#### Khía cạnh 1: Độ dài chuỗi (String Length)

* **Mã vùng V_FN1 (Valid):** Chuỗi có độ dài hợp lệ (Từ 1 đến 100 ký tự). *Ví dụ: "Nguyen Van A"*.
* **Mã vùng IV_FN1 (Invalid):** Chuỗi trống hoàn toàn (Empty `""`). *Ý nghĩa: Kiểm tra ràng buộc bắt buộc nhập*.
* **Mã vùng IV_FN2 (Invalid):** Chuỗi chỉ chứa toàn khoảng trắng (`"    "`). *Ý nghĩa: Kiểm tra xem hệ thống có cắt khoảng trắng (trim) không*.
* **Mã vùng IV_FN3 (Invalid):** Chuỗi vượt quá giới hạn an toàn hệ thống (Từ 101 ký tự trở lên). *Ý nghĩa: Kiểm tra bẫy lỗi hiển thị UI hoặc tràn DB*.

#### Khía cạnh 2: Loại ký tự (Character Types)

* **Mã vùng V_FN2 (Valid):** Chuỗi ký tự chữ Tiếng Việt có dấu và khoảng trắng phân cách đúng chuẩn chuẩn UTF-8. *Ví dụ: "Trần Nguyễn Hoàng Anh"*.
* **Mã vùng IV_FN4 (Invalid):** Chuỗi có chứa các ký tự số. *Ví dụ: "Nguyen Van A 123"*. (Họ tên thực tế không bao gồm số).
* **Mã vùng IV_FN5 (Invalid):** Chuỗi chứa các ký tự đặc biệt vô nghĩa. *Ví dụ: "Nguyen@Van#A"*.

#### Khía cạnh 3: Bảo mật & Lỗi hệ thống (Security)

* **Mã vùng IV_FN6 (Invalid):** Chuỗi chứa mã Script gây lỗ hổng XSS. *Ví dụ: `<script>alert(1)</script>*`. (Cực kỳ nguy hiểm nếu tên này hiển thị trên thanh Header hoặc màn hình Admin mà không được mã hóa).
* **Mã vùng IV_FN7 (Invalid):** Chuỗi chứa câu lệnh SQL Injection. *Ví dụ: `Admin' OR '1'='1*`.

---

### Đối với biến `phone_number` (Số điện thoại)

#### Khía cạnh 1: Độ dài chuỗi (String Length)

* **Mã vùng V_PN1 (Valid):** Chuỗi số có độ dài 10 chữ số. *Ví dụ: "0912345678"*.
* **Mã vùng V_PN2 (Valid):** Chuỗi số có độ dài 11 chữ số (Vẫn cần test do đặc tả ghi rõ 10-11 số). *Ví dụ: "01234567890"*.
* **Mã vùng IV_PN1 (Invalid):** Chuỗi trống không nhập gì.
* **Mã vùng IV_PN2 (Invalid):** Chuỗi số quá ngắn (Dưới 10 chữ số). *Ví dụ: "0912345"*.
* **Mã vùng IV_PN3 (Invalid):** Chuỗi số quá dài (Từ 12 chữ số trở lên). *Ví dụ: "09123456789012"*.

#### Khía cạnh 2: Loại ký tự & Định dạng đầu số (Character Types & Format)

* **Mã vùng IV_PN4 (Invalid):** Chuỗi số không bắt đầu bằng số `0` ở đầu. *Ví dụ: "1912345678"*.
* **Mã vùng IV_PN5 (Invalid):** Chuỗi chứa các ký tự chữ cái hoặc khoảng trắng xen kẽ. *Ví dụ: "0912 345abc"*.
* **Mã vùng IV_PN6 (Invalid):** Chuỗi chứa ký tự đặc biệt. *Ví dụ: "0912-345-678"* hoặc `+84912345678` (Mặc dù +84 hợp lệ thực tế nhưng đặc tả FR-04 ghi rõ "bắt đầu bằng số 0" nên dạng +84 sẽ bị coi là Invalid theo đặc tả).

#### Khía cạnh 3: Bảo mật & Hệ thống (Security)

* **Mã vùng IV_PN7 (Invalid):** Gửi giá trị `null` hoặc `undefined` trực tiếp qua API payload của request cập nhật hồ sơ để test khả năng bẫy lỗi phía Backend.

---
#### b. Áp dụng kỹ thuật Phân tích giá trị biên (Boundary Value Analysis)

### 1. Phân tích giá trị biên 3 điểm (3-point BVA) cho `phone_number`

Đặc tả hệ thống quy định độ dài hợp lệ của số điện thoại là **từ 10 đến 11 chữ số**. Áp dụng phương pháp 3-point BVA, ta xác định các điểm kiểm thử tại hai mốc biên như sau:

* **Tại mốc biên dưới (10 chữ số):**
* *Dưới biên (9 chữ số):* Giá trị **Invalid** (Ví dụ: `091234567`)
* *Tại biên (10 chữ số):* Giá trị **Valid** (Ví dụ: `0912345678`)
* *Trên biên (11 chữ số):* Giá trị **Valid** (Trùng với điểm tại biên của mốc 11)


* **Tại mốc biên trên (11 chữ số):**
* *Dưới biên (10 chữ số):* Giá trị **Valid** (Trùng với điểm tại biên của mốc 10)
* *Tại biên (11 chữ số):* Giá trị **Valid** (Ví dụ: `01234567890`)
* *Trên biên (12 chữ số):* Giá trị **Invalid** (Ví dụ: `0912345678901`)



---

### 2. Ma trận kiểm thử tổng hợp (Domain Testing & BVA Matrix)

Ma trận này kết hợp các kịch bản về độ dài biên, định dạng ký tự, tiền tố và các điều kiện bảo mật an toàn hệ thống cho cả hai biến `full_name` và `phone_number`.

| Test Case ID | Tên kịch bản kiểm thử | Vùng / Điểm biên bao phủ | Loại test | Kết quả mong đợi (Expected Result) |
| --- | --- | --- | --- | --- |
| **TC_FR04_01** | Cập nhật hồ sơ thành công với thông tin chuẩn | `V_NAME_01`, `V_NAME_02`, `V_PHONE_01`, `V_PHONE_02` | Valid | Cập nhật thành công, lưu đúng tiếng Việt và số điện thoại 10 số. |
| **TC_FR04_02** | Cập nhật thành công với số điện thoại biên trên | `V_NAME_01`, `V_PHONE_01`, `V_PHONE_03` | Valid | Cập nhật thành công với định dạng số điện thoại 11 số. |
| **TC_FR04_03** | Để trống trường Họ và tên | `IV_NAME_01`, `V_PHONE_02` | Invalid | Hệ thống chặn lại, hiển thị thông báo lỗi yêu cầu nhập Họ tên. |
| **TC_FR04_04** | Họ và tên chỉ chứa toàn khoảng trắng | `IV_NAME_01` (Whitespace) | Invalid | Hệ thống tự động trim, nhận diện là trống và báo lỗi. |
| **TC_FR04_05** | Họ và tên vượt quá giới hạn (256 ký tự) | `IV_NAME_02` | Invalid | Hệ thống chặn lưu và báo lỗi chuỗi quá dài. |
| **TC_FR04_06** | Họ và tên chứa ký tự số hoặc ký tự đặc biệt | `IV_NAME_03`, `IV_NAME_04` | Invalid | Báo lỗi định dạng tên không hợp lệ. |
| **TC_FR04_07** | Số điện thoại không bắt đầu bằng số 0 | `IV_PHONE_01` | Invalid | Hệ thống chặn lại, báo lỗi số điện thoại không đúng định dạng. |
| **TC_FR04_08** | Số điện thoại dưới biên dưới (9 chữ số) | BVA Biên dưới (9 số), `IV_PHONE_02` | Invalid | Hệ thống báo lỗi số điện thoại phải từ 10-11 chữ số. |
| **TC_FR04_09** | Số điện thoại vượt biên trên (12 chữ số) | BVA Biên trên (12 số), `IV_PHONE_03` | Invalid | Hệ thống báo lỗi số điện thoại không được quá 11 chữ số. |
| **TC_FR04_10** | Số điện thoại chứa ký tự chữ hoặc dấu cách | `IV_PHONE_04`, `IV_PHONE_05` | Invalid | Chặn lưu, yêu cầu chỉ nhập ký tự số liền mạch. |
| **TC_FR04_11** | Bơm mã độc XSS / SQLi vào trường Họ và tên | `IV_NAME_05`, `IV_NAME_06` | Invalid | Hệ thống mã hóa đầu vào, lưu dạng văn bản thuần an toàn. |
| **TC_FR04_12** | Gửi số điện thoại định dạng Number qua API | `IV_PHONE_08` | Invalid | Tầng Backend xử lý bẫy lỗi hoặc tự convert sang String, tránh mất số 0 đầu. |

---

### 3. Danh sách Test Cases chi tiết (Chi tiết hóa từng kịch bản)

#### 🔹 Nhóm 1: Các trường hợp kiểm thử hợp lệ (Valid)

* **TC_FR04_01: Cập nhật hồ sơ thành công với thông tin chuẩn (Biên dưới SĐT)**
* **Các bước thực hiện:**
1. Đăng nhập tài khoản người dùng trên EShop.
2. Điều hướng tới trang "Quản lý hồ sơ cá nhân".
3. Nhập dữ liệu hợp lệ vào ô "Họ và tên" và "Số điện thoại".
4. Nhấn nút "Cập nhật".


* **Dữ liệu đầu vào:** `full_name = "Nguyễn Văn Hoàng Anh"`, `phone_number = "0912345678"` (10 số)
* **Kết quả mong đợi:** Hệ thống hiển thị thông báo "Cập nhật hồ sơ thành công", thông tin mới được lưu và hiển thị chính xác trên giao diện (chuẩn UTF-8).


* **TC_FR04_02: Cập nhật thành công với số điện thoại biên trên (11 số)**
* **Các bước thực hiện:** Làm tương tự như các bước của TC_FR04_01.
* **Dữ liệu đầu vào:** `full_name = "Lê Thị Thu"`, `phone_number = "01234567890"` (11 số)
* **Kết quả mong đợi:** Cập nhật thành công, lưu trữ đủ 11 chữ số của số điện thoại trên hệ thống.



#### 🔹 Nhóm 2: Các trường hợp kiểm thử không hợp lệ (Invalid)

* **TC_FR04_03: Để trống trường Họ và tên**
* **Các bước thực hiện:** Vào trang hồ sơ cá nhân, xóa trống ô "Họ và tên", nhập SĐT hợp lệ, nhấn "Cập nhật".
* **Dữ liệu đầu vào:** `full_name = ""`, `phone_number = "0912345678"`
* **Kết quả mong đợi:** Hệ thống chặn lại tại UI, không gửi request và báo lỗi màu đỏ: *"Họ và tên không được để trống"*.


* **TC_FR04_04: Họ và tên chỉ chứa toàn khoảng trắng**
* **Các bước thực hiện:** Nhập 5 dấu cách vào ô "Họ và tên", nhấn "Cập nhật".
* **Dữ liệu đầu vào:** `full_name = "     "`, `phone_number = "0912345678"`
* **Kết quả mong đợi:** Hệ thống tự động trim khoảng trắng, nhận diện chuỗi rỗng và báo lỗi tương tự TC_FR04_03.


* **TC_FR04_07: Số điện thoại không bắt đầu bằng số 0 (Ví dụ dùng đầu số +84)**
* **Các bước thực hiện:** Nhập họ tên hợp lệ, nhập số điện thoại bắt đầu bằng mã quốc gia, nhấn "Cập nhật".
* **Dữ liệu đầu vào:** `full_name = "Trần Văn B"`, `phone_number = "+8491234567"`
* **Kết quả mong đợi:** Hệ thống chặn hành động lưu, báo lỗi: *"Số điện thoại không hợp lệ, phải bắt đầu bằng số 0"*.


* **TC_FR04_08: Số điện thoại dưới biên dưới (9 chữ số - BVA Invalid)**
* **Các bước thực hiện:** Nhập số điện thoại thiếu số (chỉ có 9 số), nhấn "Cập nhật".
* **Dữ liệu đầu vào:** `full_name = "Trần Văn B"`, `phone_number = "091234567"`
* **Kết quả mong đợi:** Hệ thống báo lỗi độ dài: *"Số điện thoại phải có độ dài từ 10 đến 11 chữ số"*.


* **TC_FR04_09: Số điện thoại vượt biên trên (12 chữ số - BVA Invalid)**
* **Các bước thực hiện:** Nhập số điện thoại thừa số (12 số), nhấn "Cập nhật".
* **Dữ liệu đầu vào:** `full_name = "Trần Văn B"`, `phone_number = "0912345678901"`
* **Kết quả mong đợi:** Hệ thống chặn không cho gõ tiếp (nếu giới hạn thuộc tính) hoặc báo lỗi độ dài vượt ngưỡng cho phép.



#### 🔹 Nhóm 3: Các trường hợp kiểm thử Bảo mật & API phá hoại

* **TC_FR04_11: Tấn công Stored XSS qua trường Họ và tên**
* **Các bước thực hiện:** Nhập đoạn script vào ô Họ và tên, bấm "Cập nhật". Tải lại trang hoặc chuyển sang trang quản trị admin để xem hiển thị tên người dùng này.
* **Dữ liệu đầu vào:** `full_name = "<italic>Hacker</italic><script>alert(1)</script>"`, `phone_number = "0912345678"`
* **Kết quả mong đợi:** Giao diện hiển thị nguyên văn chuỗi text, không thực thi mã script, không hiện pop-up alert của trình duyệt.


* **TC_FR04_12: Gửi dữ liệu số điện thoại kiểu số thuần (Number) qua API payload**
* **Các bước thực hiện:** Sử dụng Postman gửi request POST/PUT trực tiếp đến endpoint cập nhật profile.
* **Dữ liệu đầu vào:** Body request dạng JSON: `{ "full_name": "Test API", "phone_number": 0912345678 }` (Chú ý giá trị phone không bọc trong dấu ngoặc kép).
* **Kết quả mong đợi:** Tầng Backend API xử lý an toàn (tự động chuyển đổi sang chuỗi chữ trước khi lưu hoặc báo lỗi định dạng), cơ sở dữ liệu không bị lưu thiếu số 0 đầu thành `912345678`.



---
