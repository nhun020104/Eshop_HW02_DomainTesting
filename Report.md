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
