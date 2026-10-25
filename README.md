[README-BACKEND.md](https://github.com/user-attachments/files/33916723/README-BACKEND.md)
# Backend QLBanHang (PHP + SQLite)

Backend này chạy bằng PHP 8.2 và SQLite có sẵn trong XAMPP. SQLite được chọn để có thể cài đặt trên máy tính hiện tại mà không phải cài thêm dịch vụ cơ sở dữ liệu; các giao dịch nghiệp vụ vẫn được thực thi trong transaction của SQLite. Không cần Composer.

## Chạy thử trên máy

1. Bật Apache trong XAMPP Control Panel.
2. Tạo tài khoản chủ xưởng trong PowerShell:

   ```powershell
   & 'C:\xampp\php\php.exe' 'C:\xampp\htdocs\QLBanHang\api\create-admin.php' 'ten-dang-nhap' 'Ten Chu Xuong' owner
   ```

   Thay vì truyền mật khẩu qua dòng lệnh, script sẽ hỏi mật khẩu riêng (tối thiểu 12 ký tự). Tài khoản được lưu bằng `password_hash`; không có endpoint tự đăng ký công khai.

3. Kiểm tra API tại `http://localhost/QLBanHang/api/health`. Phản hồi mong đợi: `{"status":"ok"}`.
4. API tạo SQLite tự động tại `api\data\qlbanhang.sqlite`. Thư mục dữ liệu bị chặn truy cập HTTP bởi `.htaccess`; không xóa hoặc chép file khi ứng dụng đang ghi dữ liệu.

## API hiện có

- `GET /api/health`
- `POST /api/login`, `GET /api/session`, `POST /api/logout`
- `GET /api/state` (sau khi đăng nhập)
- `POST /api/bootstrap/import` (một lần duy nhất, vai trò chủ xưởng)
- `POST /api/stock-in`
- `POST /api/production-orders/{id}/check-materials`
- `POST /api/production-orders/{id}/issue-materials`
- `POST /api/production-orders/{id}/complete`
- `POST /api/sales-orders`
- `GET /api/alerts/abnormal-usage`

Các yêu cầu ghi cần cookie phiên và header `X-CSRF-Token` lấy từ phản hồi đăng nhập. Luồng xuất sản xuất và bán hàng kiểm tra tồn kho ở backend, ghi ledger và cập nhật dữ liệu ứng dụng trong cùng transaction. Lệnh sản xuất lưu snapshot BOM lúc tạo lệnh để BOM thay đổi sau đó không sửa lịch sử lệnh. Giao dịch tồn kho có trigger cấm `UPDATE` và `DELETE`. Nhập kho yêu cầu `idempotencyKey`; xuất kho sản xuất dùng khóa theo lệnh/vật tư; đơn bán dùng ID đơn để chống tạo trùng.

Vai trò được lấy từ tài khoản trên máy chủ, không tin giá trị vai trò gửi từ giao diện. Vai trò `warehouse` được nhập/xuất kho và xử lý lệnh sản xuất; `sales` được tạo đơn bán; `owner` được thực hiện các nghiệp vụ. API kiểm tra vật tư và cảnh báo chỉ đọc.

## Trạng thái tích hợp giao diện

Trang HTML ban đầu vẫn dùng `localStorage` cho các màn hình CRUD hiện hữu. Khi đăng nhập lần đầu vào backend chưa có dữ liệu, giao diện sẽ thử nhập một lần dữ liệu `localStorage` của đúng trình duyệt/origin đang mở; nếu không còn dữ liệu đó, hãy nhập dữ liệu qua API hoặc đợi tới khi CRUD được chuyển hoàn toàn. Trước khi dùng production, cần chuyển toàn bộ thao tác đọc/ghi CRUD sang API và thêm quản lý người dùng/phân quyền qua giao diện. Không chạy song song chỉnh sửa tồn kho từ localStorage và API như hai nguồn dữ liệu độc lập.

## Truy cập công khai từ Internet

Chỉ khởi chạy Apache trên máy cá nhân **không** làm website tự truy cập được từ Internet. Cần tên miền, IP công cộng hoặc tunnel được kiểm soát, HTTPS, cấu hình router/firewall, cập nhật Windows/XAMPP, giám sát, sao lưu SQLite ngoại tuyến và kiểm tra khôi phục. Nếu HTTPS kết thúc tại reverse proxy, cấu hình biến môi trường `QLBANHANG_FORCE_HTTPS=1` trong môi trường PHP để cookie phiên luôn có cờ `Secure`. Không mở phpMyAdmin hoặc cổng SQLite ra Internet. Bản này chưa được kiểm thử tải, kiểm thử xâm nhập hay chứng nhận sẵn sàng thương mại; nên đặt sau reverse proxy HTTPS và nhờ chuyên gia rà soát trước khi tiếp nhận dữ liệu/khách hàng thật.
