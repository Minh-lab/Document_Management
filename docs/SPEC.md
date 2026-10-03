# ĐẶC TẢ YÊU CẦU (SPECIFICATION)
**Dự án:** Ứng dụng Quản lý Tài liệu Học tập
**Kiến trúc:** Cashew Architecture

## 1. Giới thiệu
Tài liệu này đặc tả các yêu cầu nghiệp vụ (User Stories), tiêu chí chấp nhận (Acceptance Criteria) và mô hình dữ liệu (Data Model) sơ bộ cho ứng dụng Quản lý Tài liệu Học tập. Hệ thống được thiết kế bám sát các ràng buộc nghiêm ngặt của Kiến trúc Cashew, chú trọng vào việc cô lập hoàn toàn giao diện (UI) khỏi cơ sở dữ liệu (Database).

---

## 2. Mô hình dữ liệu sơ bộ (Data Model)
Hệ thống sử dụng SQLite (thông qua package `drift`). Dữ liệu được quản lý ở 2 hình thái: **Table Schema** (tại lớp Data) và **Struct Model** (lớp trung gian luân chuyển dữ liệu).

### 2.1 Bảng Cơ sở dữ liệu: `StudyDocuments`
*Vị trí khai báo: `lib/database/tables.dart`*

| Tên trường | Kiểu dữ liệu (Drift) | Ràng buộc & Đặc tả |
| :--- | :--- | :--- |
| `id` | `IntColumn` | Khóa chính (Auto Increment). |
| `title` | `TextColumn` | Bắt buộc. Độ dài tối thiểu 1 ký tự, tối đa 255. |
| `description` | `TextColumn` | Tùy chọn (Nullable). Cho phép chuỗi rỗng. |
| `filePath` | `TextColumn` | Đường dẫn URI tới file lưu trong thiết bị hoặc URL. |
| `category` | `TextColumn` | Bắt buộc. Chỉ chấp nhận các giá trị: "Bài giảng", "Bài tập", "Tham khảo". |
| `createdAt` | `DateTimeColumn` | Bắt buộc. Thời điểm tạo bản ghi (mặc định lấy thời gian hiện tại lúc insert). |
| `updatedAt` | `DateTimeColumn` | Bắt buộc. Thời điểm cập nhật cuối cùng. |

### 2.2 Đối tượng Trung gian: `DocumentModel`
*Vị trí khai báo: `lib/struct/document_model.dart`*
Lớp Data Class thuần tuý (Pure Dart Object) được ánh xạ từ `StudyDocuments`. UI chỉ được nhận và tương tác với đối tượng này, tuyệt đối không tương tác với các Entity của thư viện `drift`.

---

## 3. User Stories và Tiêu chí chấp nhận (Acceptance Criteria)

> ⚠️ **RÀNG BUỘC KIẾN TRÚC CASHEW (Áp dụng cho mọi User Story):**
> 1. Toàn bộ Widget trong `lib/pages/` và `lib/widgets/` KHÔNG chứa bất kỳ truy vấn CSDL nào.
> 2. Mọi thao tác User Event từ giao diện phải gọi qua trung gian tại `lib/functions.dart`.
> 3. `lib/functions.dart` chịu trách nhiệm gọi lệnh SQL và mapping kết quả về Struct.

### User Story 1: Thêm tài liệu học tập
**Là một** sinh viên,
**Tôi muốn** lưu trữ một tài liệu học tập mới vào ứng dụng,
**Để** giữ cho bài giảng và bài tập của tôi được sắp xếp có hệ thống.

**Tiêu chí chấp nhận (Acceptance Criteria):**
1. Màn hình thêm mới phải chứa các thành phần form: Input Tiêu đề, Input Mô tả, Selector chọn File/Link (đường dẫn), và Dropdown chọn Danh mục (Bài giảng/Bài tập/Tham khảo).
2. Khi để trống trường "Tiêu đề" hoặc "Đường dẫn", UI hiển thị lỗi validation nội bộ, không tiến hành submit form.
3. Khi form hợp lệ, nhấn [Lưu], UI truyền dữ liệu xuống `functions.dart`.
4. Logic xử lý tính toán ngày giờ hiện tại cho `createdAt` và `updatedAt`, sau đó Insert vào DB thông qua Drift.
5. Sau khi insert thành công, hệ thống hiển thị Snackbar "Đã thêm tài liệu" và quay lại màn hình danh sách; danh sách được làm mới và hiển thị bản ghi mới thêm.

### User Story 2: Sửa tài liệu học tập
**Là một** sinh viên,
**Tôi muốn** cập nhật lại thông tin của một tài liệu hiện có,
**Để** chỉnh sửa tên bị sai hoặc thay thế bằng file bài tập phiên bản mới nhất.

**Tiêu chí chấp nhận (Acceptance Criteria):**
1. Màn hình chỉnh sửa được tái sử dụng từ màn hình Thêm mới, nhưng dữ liệu cũ được điền sẵn (bind) vào các form field.
2. Người dùng có thể chỉnh sửa Tiêu đề, Mô tả, Danh mục và Đường dẫn.
3. Khi nhấn [Cập nhật], giá trị `updatedAt` tự động được thay đổi thành thời gian hiện tại.
4. Giao diện (Page) gọi phương thức cập nhật trong `functions.dart` kèm theo `id` của tài liệu.
5. Sau khi hệ thống thông báo "Cập nhật thành công", màn hình danh sách phản ánh ngay lập tức các nội dung vừa chỉnh sửa.

### User Story 3: Xóa tài liệu học tập
**Là một** sinh viên,
**Tôi muốn** xóa đi những tài liệu ở học kỳ cũ,
**Để** làm nhẹ bộ nhớ và dễ tìm kiếm các tài liệu hiện tại hơn.

**Tiêu chí chấp nhận (Acceptance Criteria):**
1. Tại thẻ tài liệu (`widgets/document_card.dart`), người dùng có thể kích hoạt thao tác xóa (nhấn nút Thùng rác hoặc vuốt).
2. Khi kích hoạt, hệ thống phải hiển thị một Dialog xác nhận để tránh việc người dùng xóa nhầm.
3. Nếu người dùng chọn "Hủy bỏ", không có điều gì xảy ra.
4. Nếu người dùng chọn "Xóa", UI đẩy `id` tới lệnh Delete trong `functions.dart`.
5. DB thực thi việc gỡ bản ghi. Ngay sau đó, thẻ tài liệu biến mất khỏi UI danh sách bằng các hiệu ứng chuyển đổi mượt mà (không gây chớp/reload toàn trang).

### User Story 4: Tìm kiếm và Liệt kê tài liệu
**Là một** sinh viên,
**Tôi muốn** xem toàn bộ tài liệu mình có và tìm kiếm nhanh theo tên,
**Để** lập tức mở được bài giảng/bài tập mình cần trong lúc ôn thi.

**Tiêu chí chấp nhận (Acceptance Criteria):**
1. Trang chủ (`pages/home_page.dart`) khi khởi động gọi hàm lấy danh sách (vd: `getDocuments()`) từ `functions.dart`.
2. UI hiển thị danh sách dạng Grid hoặc List các thẻ `document_card.dart` bằng dữ liệu `DocumentModel` trả về.
3. Có thanh tìm kiếm ở đầu trang. Bất cứ khi nào người dùng gõ phím, `functions.dart` thực thi câu lệnh truy vấn SQL (toán tử `LIKE`) với từ khóa để tìm trong `title` và `description`.
4. Hỗ trợ lọc theo Danh mục (Chips: Tất cả, Bài giảng, Bài tập, Tham khảo).
5. Khi không có dữ liệu (DB rỗng hoặc tìm kiếm không trùng khớp), UI render một Component Trống (Empty Component) mang thông điệp rõ ràng, thay vì hiển thị màn hình trắng.
