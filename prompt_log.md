# Họ tên: Nguyễn Hoàng Uyển Như
# MSSV: 22127315

# HW02 Prompt Log

## Prompt 01

Time: 12:23 PM 7/3/2026

Tool: Gemini

*Prompt:
Tôi đang làm bài tập kiểm thử phần mềm cho hệ thống EShop. Chức năng đầu tiên cần xử lý là FR-14: Quản lý Danh mục (Web Admin) ở phân hệ Admin. Đặc tả yêu cầu: Admin có thể thêm mới danh mục sản phẩm, trong đó trường Tên danh mục (Category Name) là bắt buộc nhập, không được để trống.

Hãy đóng vai trò là một chuyên gia QA, thực hiện các bước sau theo đúng lý thuyết kỹ thuật Kiểm thử miền (Domain Testing):

Xác định rõ các biến đầu vào (Input variables), kiểu dữ liệu và ràng buộc của chức năng Thêm danh mục này.

Phân chia các miền/vùng tương đương (Valid và Invalid partitions) cho các biến đó.

Lưu ý: Chỉ phân tích vùng tương đương, chưa cần viết thành test case hoàn chỉnh.

*Phản hồi của AI:
Chào bạn, với vai trò là một chuyên gia QA, tôi sẽ hướng dẫn bạn áp dụng kỹ thuật **Kiểm thử miền (Domain Testing)** để phân tích các vùng tương đương cho chức năng **FR-14: Quản lý Danh mục** dựa trên đặc tả của hệ thống [ttbhanh/eshop-sut](https://github.com/ttbhanh/eshop-sut).

Dưới đây là kết quả phân tích chi tiết:

---

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

> **Lời khuyên từ QA:** Bước tiếp theo sau khi có các vùng tương đương này, bạn sẽ áp dụng thêm kỹ thuật **Phân tích giá trị biên (Boundary Value Analysis)** cho độ dài chuỗi (thử nghiệm tại các mốc biên như: 0, 1, 2, 254, 255, 256 ký tự) để chọn ra các giá trị đại diện và thiết kế thành các test case cụ thể. Chúc bạn làm bài tập thật tốt!

## Prompt 02

Time: 

Tool: Gemini

*Prompt:


*Phản hồi của AI: