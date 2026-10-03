# BÁO CÁO TRIỂN KHAI: Ứng dụng Quản lý Tài liệu Học tập theo Kiến trúc Cashew

## 1. Phân tích yêu cầu chức năng và Sơ đồ luồng dữ liệu

### 1.1 Yêu cầu chức năng cốt lõi
Ứng dụng quản lý tài liệu học tập (bài giảng, bài tập, tham khảo) yêu cầu các tính năng:
- **Thêm tài liệu:** Lưu tên, loại tài liệu, mô tả, và đường dẫn/tệp đính kèm.
- **Sửa tài liệu:** Cập nhật thông tin của tài liệu đã tồn tại.
- **Xóa tài liệu:** Loại bỏ tài liệu khỏi cơ sở dữ liệu.
- **Tìm kiếm/Hiển thị:** Liệt kê toàn bộ tài liệu và cho phép tìm kiếm theo tên hoặc phân loại.

### 1.2 Sơ đồ luồng dữ liệu (Data Flow Diagram)
Áp dụng nguyên lý tách lớp từ dự án Cashew:
- **UI (Pages/Widgets)** không bao giờ gọi trực tiếp SQLite.
- **Functions** đóng vai trò là tầng Service, nhận sự kiện từ UI, xử lý logic, và gọi vào Database.
- **Database** định nghĩa Schema (Drift/Moor) và Query, trả về kết quả cấu trúc hóa (Struct).

```mermaid
flowchart TD
    subgraph Lớp Trình diễn (Presentation Layer)
        P[pages/home_page.dart]
        W[widgets/document_card.dart]
    end

    subgraph Lớp Logic (Business Logic Layer)
        F[functions.dart]
    end

    subgraph Lớp Dữ liệu (Data Layer)
        D[database/tables.dart]
        S[struct/document_model.dart]
    end

    P -->|1. User Input (Search, Add)| F
    W -->|2. View Action (Delete)| F
    F -->|3. Read/Write Request| D
    D -->|4. SQL Execution (SQLite)| D
    D -->|5. Return Parsed Data| S
    S -->|6. Data Object| F
    F -->|7. Update State| P
```

---

## 2. Thiết lập cấu trúc thư mục (Kiến trúc Cashew)

Dự án được phân bổ mô-đun hóa nghiêm ngặt, tuân theo thiết kế gốc của Cashew Expense Tracker:

```text
lib/
├── colors.dart                 # Định nghĩa bảng màu toàn cục, Themes.
├── functions.dart              # Chứa các hàm tiện ích toàn cục (CRUD Document handlers).
├── database/
│   ├── tables.dart             # Schema định nghĩa bảng Documents bằng Drift.
│   └── tables.g.dart           # File tự động sinh của Drift.
├── struct/
│   ├── document_model.dart     # Data Model của Document.
│   ├── defaultCategories.dart  # Hằng số: Loại tài liệu (Bài giảng, Bài tập,...).
│   └── settings.dart           # Cấu hình người dùng.
├── pages/
│   ├── home_page.dart          # Màn hình chính liệt kê và tìm kiếm.
│   └── add_document_page.dart  # Màn hình form thêm/sửa tài liệu.
└── widgets/
    ├── document_card.dart      # Widget thẻ hiển thị 1 tài liệu tái sử dụng.
    └── custom_text_field.dart  # Widget form input dùng chung.
```

---

## 3. Triển khai chức năng cốt lõi (Mô phỏng Mã nguồn)

### 3.1. Lớp Dữ liệu (`database/tables.dart`)
Sử dụng Drift để định nghĩa bảng `StudyDocuments`.
```dart
import 'package:drift/drift.dart';

class StudyDocuments extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 100)();
  TextColumn get description => text().nullable()();
  TextColumn get type => text()(); // Lecture, Assignment, Reference
  DateTimeColumn get dateAdded => dateTime().withDefault(currentDateAndTime)();
}
```

### 3.2. Lớp Cấu trúc - Struct (`struct/document_model.dart`)
Cấu trúc chuyển giao dữ liệu độc lập với schema.
```dart
class DocumentModel {
  final int id;
  final String title;
  final String description;
  final String type;

  DocumentModel({
    required this.id, 
    required this.title, 
    required this.description, 
    required this.type
  });
}
```

### 3.3. Lớp Logic (`functions.dart`)
Đóng gói mọi truy vấn, UI không gọi trực tiếp Database.
```dart
import 'database/tables.dart';
import 'struct/document_model.dart';

// Thêm tài liệu
Future<void> addDocument(String title, String desc, String type) async {
  await database.into(database.studyDocuments).insert(
    StudyDocumentsCompanion.insert(
      title: title,
      description: Value(desc),
      type: type,
    ),
  );
}

// Xóa tài liệu
Future<void> deleteDocument(int id) async {
  await (database.delete(database.studyDocuments)..where((t) => t.id.equals(id))).go();
}

// Lấy danh sách (Search)
Future<List<DocumentModel>> getDocuments(String searchQuery) async {
  final query = database.select(database.studyDocuments)
    ..where((t) => t.title.like('%$searchQuery%'));
  
  final results = await query.get();
  return results.map((row) => DocumentModel(
    id: row.id,
    title: row.title,
    description: row.description ?? "",
    type: row.type,
  )).toList();
}
```

### 3.4. Lớp Giao diện (`pages/home_page.dart` & `widgets/document_card.dart`)
Chỉ hiển thị và gọi `functions.dart`.
```dart
// Trong pages/home_page.dart
import '../functions.dart';
import '../widgets/document_card.dart';

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Tài Liệu Học Tập')),
      body: FutureBuilder<List<DocumentModel>>(
        // Gọi hàm từ functions.dart, KHÔNG gọi thẳng database
        future: getDocuments(''), 
        builder: (context, snapshot) {
          if (!snapshot.hasData) return CircularProgressIndicator();
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, index) {
              // Sử dụng component từ thư mục widgets/
              return DocumentCard(document: snapshot.data![index]);
            },
          );
        },
      ),
    );
  }
}
```

---

## 4. Kiểm thử tính đúng đắn của việc phân tách logic

Việc phân tách lớp được kiểm thử thông qua các kịch bản Mock (Dependency Injection ngầm định):
1. **Kiểm thử Unit Test cho `functions.dart`**: Khởi tạo SQLite in-memory, gọi các hàm `addDocument`, `deleteDocument` và kiểm tra dữ liệu thay đổi. Quá trình này hoàn toàn không cần load giao diện (Pages).
2. **Kiểm thử UI (Widget Test) cho `pages/home_page.dart`**: Chặn (mock) các hàm lấy dữ liệu trong `functions.dart` trả về danh sách tài liệu giả (dummy models từ `struct/`). Đảm bảo màn hình vẫn render đủ số lượng `DocumentCard` bất kể Database thật có hoạt động hay không.
3. **Quy tắc vi phạm (Linting)**: Đảm bảo không có bất kỳ import `package:drift` hay `database/tables.dart` nào tồn tại bên trong thư mục `pages/` hoặc `widgets/`. Nếu phát hiện, tức là vi phạm nguyên tắc Cashew.

---

## 5. Báo cáo giải trình về áp dụng kiến trúc Cashew

Kiến trúc ứng dụng của Cashew, được thiết kế đặc thù cho các ứng dụng quản lý cá nhân trên Flutter (như quản lý tài chính, và giờ là quản lý tài liệu), mang lại các giá trị cốt lõi sau:

1. **Khả năng tái sử dụng giao diện (Thư mục `widgets/`)**: Bằng cách đẩy các thành phần UI chung (thẻ tài liệu, nút bấm, trường nhập liệu) vào `widgets/`, các trang (`pages/`) trở nên rất mỏng. Việc thêm màn hình mới không làm trùng lặp code giao diện.
2. **Cô lập trạng thái dữ liệu (Thư mục `database/` và `struct/`)**: Toàn bộ thao tác SQLite (qua Drift) nằm tại `database/`. Dữ liệu sau đó được định dạng lại thành các đối tượng chuẩn ở `struct/`. Nếu sau này muốn chuyển từ SQLite sang Firebase, ta chỉ cần viết lại code bên trong `database/` và `functions.dart` mà không phải thay đổi bất kỳ dòng code nào bên trong UI (`pages/`).
3. **Điều hướng logic tập trung (`functions.dart`)**: Tập trung logic tại một nơi giúp dễ gỡ lỗi (debug) và tránh việc xử lý logic phức tạp, gọi trực tiếp bộ nhớ cục bộ tràn lan giữa các màn hình ứng dụng, đảm bảo khả năng mở rộng hệ thống tốt trong tương lai.
