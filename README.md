# INT4211 - LẬP TRÌNH TRÊN CÁC THIẾT BỊ DI ĐỘNG
## BÁO CÁO THỰC HÀNH LAB F4: ĐIỀU HƯỚNG ĐA MÀN HÌNH TRONG FLUTTER

---

### THÔNG TIN SINH VIÊN
* **Họ và tên:** Đỗ Lê Tuấn Đạt
* **Mã số sinh viên (MSSV):** 231A290089
* **Lớp học phần:** Lập trình thiết bị di động (INT4211)
* **Môi trường thực thi:** Flutter 3.x (Material 3) trên macOS (Apple Silicon)
* **Thiết bị thử nghiệm:** Máy ảo Pixel 8 (Android 17, API 37.1) & Google Chrome
* **Repository GitHub:** `https://github.com/tuandatdl/F4_231A290089` (Private, đã mời giảng viên làm Collaborator)

---

### CÁC MỤC ĐÃ HOÀN THÀNH

#### 1. Quản lý Ngăn xếp Route & Truyền đối tượng bất biến (Phần 1, 3)
* Thiết kế lớp dữ liệu `Contact` hoàn toàn bất biến (`immutable` với các thuộc tính `final` và phương thức `copyWith`), ngăn ngừa lỗi màn hình con tự ý thay đổi trạng thái trực tiếp của màn hình cha.
* Điều hướng giữa 3 màn hình chính (`ContactListPage`, `ContactDetailPage`, `ContactEditPage`) bằng `Navigator.push` kết hợp `MaterialPageRoute`, truyền dữ liệu trực tiếp qua tham số constructor an toàn kiểu dữ liệu lúc biên dịch.
* Đăng ký và điều hướng tới màn hình phụ `AboutPage` bằng cơ chế **Named Route** (`Navigator.pushNamed`).

#### 2. Nhận kết quả bất đồng bộ & Phòng thủ Context (Phần 3.4)
* Đóng gói kết quả trả về bằng lớp `DetailResult` kết hợp `enum DetailAction { updated, deleted }`.
* Sử dụng `await Navigator.push<DetailResult>(...)` kết hợp 2 lớp bảo vệ bắt buộc:
  1. `if (result == null) return;`: Xử lý an toàn khi người dùng bấm nút Back mà không lưu.
  2. `if (!mounted) return;`: Đảm bảo Widget còn gắn trên cây giao diện trước khi gọi `setState` và `ScaffoldMessenger`, đạt chuẩn `No issues found!` khi chạy `flutter analyze`.

#### 3. Chặn thoát khi chưa lưu bằng PopScope (Phần 3.6)
* Bọc màn hình `ContactEditPage` trong widget `PopScope` với `canPop: !_daSua` và `onPopInvokedWithResult`.
* Khi người dùng đã gõ chỉnh sửa trên `Form` (`onChanged`) mà bấm Back, hệ thống tự động chặn lại và hiển thị `AlertDialog` xác nhận *"Bỏ thay đổi?"* (Ở lại / Thoát).

#### 4. Bài nâng cao NC1: Màn hình Tìm kiếm trả về liên hệ được chọn
* Xây dựng `ContactSearchPage` cho phép lọc danh bạ theo thời gian thực bằng tên hoặc số điện thoại.
* Khi chạm vào một kết quả, gọi `Navigator.pop(context, c)` để trả đối tượng `Contact` về màn hình danh sách và mở thẳng trang chi tiết của người đó.

#### 5. Bài nâng cao NC2: Hiệu ứng chuyển cảnh Hero cho Avatar
* Bọc `CircleAvatar` ở cả màn hình Danh sách và màn hình Chi tiết trong widget `Hero` với khóa định danh duy nhất `tag: contact.dienThoai`, giúp ảnh đại diện bay và phóng to mượt mà khi chuyển trang.

#### 6. Bảng đối chiếu Điều hướng: Android (Lab A5) ↔ Flutter (Lab F4)
| Việc cần làm | Android Native (Java - Lab A5) | Flutter (Dart - Lab F4) |
|---|---|---|
| Mở màn hình mới | `new Intent(this, X.class)` + `startActivity` | `Navigator.push` + `MaterialPageRoute` |
| Truyền một đối tượng | Lớp phải `implements Parcelable` | Truyền thẳng qua tham số `constructor` |
| Nhận kết quả trả về | `registerForActivityResult` + `launcher` | `final kq = await Navigator.push(...)` |
| Trả kết quả đi | `setResult(RESULT_OK, data)` + `finish()` | `Navigator.pop(context, giaTri)` |
| Người dùng bấm Back | `resultCode == RESULT_CANCELED` | Giá trị trả về là `null` |
| Chặn thoát khi chưa lưu | `getOnBackPressedDispatcher().addCallback` | `PopScope(canPop: false, ...)` |