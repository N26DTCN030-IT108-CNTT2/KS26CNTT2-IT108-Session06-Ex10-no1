\# Báo cáo Phân tích: So sánh và lựa chọn giải pháp tối ưu cho xử lý log tự động



\## Phần 1: Phân tích \& Đề xuất đa giải pháp



\### 1. Xác định Input và Output

\* \*\*Input:\*\* Thư mục gốc chứa log (`/var/logs/` hoặc đường dẫn tương đương trên Windows) và từ khóa cần tìm (`"TIMEOUT"`).

\* \*\*Output:\*\* Hiển thị và truy xuất được tên file, số dòng chứa lỗi, và nội dung dòng log khớp từ khóa trên nhiều thư mục con.



\### 2. Đề xuất các giải pháp

\* \*\*Giải pháp 1 (`findstr` trong CMD/PowerShell):\*\* Sử dụng tiện ích tìm kiếm chuỗi gốc của Windows kết hợp lệnh duyệt đệ quy thư mục.

\* \*\*Giải pháp 2 (`Get-ChildItem` kết hợp `Select-String` trong PowerShell):\*\* Sử dụng các cmdlet thuần túy của PowerShell để xử lý pipeline hướng đối tượng.



\---



\## Phần 2: So sánh \& Lựa chọn



| Tiêu chí | Giải pháp 1 (findstr CMD) | Giải pháp 2 (PowerShell) |

| :--- | :--- | :--- |

| \*\*Tốc độ xử lý\*\* | Rất nhanh đối với file kích thước nhỏ và vừa nhờ cơ chế quét trực tiếp cấp thấp. | Nhanh, tối ưu hóa tốt trên môi trường Windows Server hiện đại. |

| \*\*Xử lý file bị khóa\*\* | Kém (Dễ phát sinh lỗi Access Denied nếu file log đang bị tiến trình ghi nhật ký chiếm quyền). | \*\*Tốt hơn\*\* (Có thể kết hợp xử lý ngoại lệ hoặc các tham số đọc file an toàn). |

| \*\*Cross-platform\*\* | Chỉ chạy độc quyền trên Windows. | Có hỗ trợ trên PowerShell Core (đa nền tảng Windows/Linux/macOS). |

| \*\*Độ phức tạp code\*\* | Cú pháp ngắn gọn nhưng khó cấu hình các tùy chọn lọc nâng cao. | Cú pháp rõ ràng, trực quan theo mô hình hướng đối tượng. |

| \*\*Hiển thị số dòng\*\* | Có hỗ trợ hiển thị số dòng kèm tên file. | Hiển thị cực kỳ chi tiết (Tên file, số dòng, nội dung line). |



\---



\## Phần 3: Triển khai



\* \*\*Giải pháp được chọn:\*\* \*\*Giải pháp 2 (PowerShell: `Get-ChildItem` + `Select-String`)\*\*

\* \*\*Lý do chọn:\*\* Môi trường yêu cầu chạy trên Windows Server, giải pháp thuần PowerShell giúp kiểm soát tốt các ngoại lệ khi file log bị khóa (`locked files`) bởi các tiến trình hệ thống, đồng thời trả về kết quả tường minh, chuyên nghiệp.



\### Câu lệnh hoàn chỉnh xử lý bẫy file khóa:

```powershell

\# Quét đệ quy toàn bộ file log trong thư mục, tự động bỏ qua/xử lý các file đang bị khóa bởi process khác

Get-ChildItem -Path ".\\var\\logs\\" -Recurse -File | ForEach-Object {

&#x20;   try {

&#x20;       Select-String -Path $\_.FullName -Pattern "TIMEOUT" -ErrorAction Stop

&#x20;   } catch {

&#x20;       # Bỏ qua các file đang bị khóa (Locked) mà không làm dừng toàn bộ tiến trình quét

&#x20;   }

}

