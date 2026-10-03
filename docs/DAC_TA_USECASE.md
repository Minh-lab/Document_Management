# ĐẶC TẢ USE CASE: Ứng dụng Quản lý Tài liệu Học tập

Tài liệu này cung cấp mô tả chi tiết cho các Use Case cốt lõi của hệ thống, tuân thủ theo nguyên lý kiến trúc Cashew đã đề ra.

## Tác nhân (Actor)
- **Sinh viên / Người dùng:** Người sử dụng ứng dụng để lưu trữ và quản lý tài liệu học tập cá nhân.

---

## 1. Use Case: Thêm tài liệu học tập mới
**Mô tả:** Cho phép người dùng thêm một tài liệu học tập mới vào hệ thống (Bài giảng, Bài tập, Tài liệu tham khảo).
**Tiền điều kiện:** Người dùng đang ở màn hình danh sách tài liệu hoặc màn hình chính.
**Hậu điều kiện:** Hệ thống lưu trữ thành công tài liệu mới và hiển thị trên màn hình danh sách.

**Chuỗi sự kiện chính (Main Flow):**
1. Người dùng nhấn vào nút `[+]` (Thêm mới) trên giao diện.
2. Hệ thống chuyển sang màn hình "Thêm tài liệu" (Add Document Page).
3. Người dùng nhập các thông tin bắt buộc: `Tên tài liệu`, chọn `Loại tài liệu` (combo box/dropdown).
4. Người dùng nhập thông tin tùy chọn: `Mô tả chi tiết`, đính kèm tệp hoặc `Đường dẫn (URL)`.
5. Người dùng nhấn nút `[Lưu]`.
6. Lớp UI truyền dữ liệu xuống lớp Logic (`functions.dart`).
7. Lớp Logic gọi Lớp Dữ liệu (`database/tables.dart`) để insert bản ghi mới qua thư viện Drift.
8. Hệ thống thông báo "Thêm thành công" và đưa người dùng về lại màn hình danh sách. Màn hình danh sách cập nhật bản ghi mới.

**Ngoại lệ (Exceptions):**
- *1a. Thiếu thông tin bắt buộc:* Tại bước 5, nếu người dùng bỏ trống `Tên tài liệu`, UI sẽ hiển thị cảnh báo lỗi màu đỏ tại ô nhập liệu. Quá trình lưu bị chặn lại, không gọi xuống tầng Logic.

---

## 2. Use Case: Cập nhật / Sửa tài liệu
**Mô tả:** Người dùng chỉnh sửa thông tin của một tài liệu đã được lưu trước đó.
**Tiền điều kiện:** Người dùng đã có ít nhất một tài liệu trong hệ thống và đang xem danh sách tài liệu.
**Hậu điều kiện:** Thông tin tài liệu được cập nhật thành công trong hệ thống.

**Chuỗi sự kiện chính (Main Flow):**
1. Người dùng tìm và chọn tài liệu cần chỉnh sửa trên màn hình danh sách.
2. Người dùng nhấn vào biểu tượng `[Chỉnh sửa]` (Edit) trên thẻ tài liệu.
3. Hệ thống mở màn hình "Chỉnh sửa tài liệu" và nạp (bind) sẵn dữ liệu cũ của tài liệu vào các trường nhập liệu tương ứng.
4. Người dùng thay đổi thông tin (ví dụ: cập nhật lại tên, đổi loại tài liệu).
5. Người dùng nhấn nút `[Lưu]`.
6. Lớp Logic thực hiện gọi hàm update qua Database để cập nhật SQLite.
7. Hệ thống thông báo thành công, quay lại màn hình danh sách và tự động làm mới giao diện hiển thị dữ liệu đã chỉnh sửa.

**Luồng thay thế (Alternative Flow):**
- *2a. Hủy chỉnh sửa:* Tại bước 4, người dùng nhấn nút `[Hủy]` hoặc nút Back. Hệ thống đóng form, không thực hiện ghi dữ liệu, và quay lại màn hình danh sách.

---

## 3. Use Case: Xóa tài liệu
**Mô tả:** Cho phép người dùng xóa bỏ một tài liệu không còn sử dụng.
**Tiền điều kiện:** Có tài liệu tồn tại trên hệ thống.
**Hậu điều kiện:** Tài liệu bị xóa vĩnh viễn khỏi Database, danh sách tài liệu trống hoặc giảm đi một bản ghi.

**Chuỗi sự kiện chính (Main Flow):**
1. Người dùng nhấn vào biểu tượng `[Xóa]` (thùng rác) hoặc vuốt sang trái/phải (swipe-to-delete) trên một tài liệu cụ thể ở giao diện danh sách.
2. Hệ thống hiển thị Dialog xác nhận: "Bạn có chắc chắn muốn xóa tài liệu này không?".
3. Người dùng nhấn `[Đồng ý]`.
4. Lớp Logic nhận ID tài liệu, truyền lệnh `delete` tới lớp Dữ liệu (Drift) để thực thi lệnh xóa khỏi SQLite.
5. Hệ thống đóng Dialog, hiển thị Toast "Đã xóa tài liệu".
6. Màn hình danh sách cập nhật lại, loại bỏ thẻ tài liệu vừa xóa.

**Luồng thay thế:**
- *3a. Từ chối xóa:* Tại bước 2, người dùng nhấn `[Hủy]`. Dialog đóng lại, thao tác bị hủy bỏ, hệ thống giữ nguyên hiện trạng.

---

## 4. Use Case: Tìm kiếm và Xem danh sách
**Mô tả:** Người dùng có thể xem danh sách tất cả tài liệu, hoặc tìm kiếm tài liệu theo từ khóa và phân loại.
**Tiền điều kiện:** Ứng dụng khởi động thành công và tải xong trang chủ.
**Hậu điều kiện:** Hiển thị danh sách tài liệu thỏa mãn điều kiện tìm kiếm.

**Chuỗi sự kiện chính (Main Flow):**
1. Người dùng truy cập màn hình chính (`home_page.dart`).
2. Mặc định, màn hình tự động gọi hàm `getDocuments('')` từ lớp Logic. Lớp logic query Database lấy toàn bộ tài liệu và mapping sang danh sách Model.
3. Hệ thống render danh sách thẻ tài liệu (Widgets) từ dữ liệu nhận được.
4. Người dùng click vào thanh công cụ tìm kiếm (Search Bar) và nhập ký tự (ví dụ: "Toán").
5. Ngay khi gõ (hoặc sau khi nhấn Enter/Search), UI gửi từ khóa tới lớp Logic.
6. Lớp Logic thực thi query SQL bằng toán tử `LIKE` với từ khóa.
7. Trả lại danh sách tài liệu khớp với tên hoặc mô tả, giao diện chỉ hiển thị các kết quả này.

**Ngoại lệ:**
- *4a. Không tìm thấy kết quả:* Nếu query SQL không khớp bất kỳ bản ghi nào, hệ thống trả về mảng rỗng. UI sẽ hiển thị illustration/hình ảnh trống kèm câu chữ: "Không tìm thấy tài liệu phù hợp."
- *4b. Danh sách rỗng ngay từ đầu:* Nếu cơ sở dữ liệu trống, màn hình chính sẽ hiển thị thông điệp "Bạn chưa có tài liệu nào, hãy nhấn nút (+) để thêm mới." thay vì một danh sách rỗng.
