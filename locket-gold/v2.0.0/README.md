# cuios.shop — Locket Gold cho Shadowrocket

Bản v2.0.0, ngày 24/10/2026. Bộ thử nghiệm can thiệp CustomerInfo RevenueCat phía client.

**Chưa xác minh trên iPhone. Không bao gồm bản vá video 60 giây, dung lượng 128 MiB,
skin widget hoặc giao diện tùy biến trong IPA.** Tính năng cần máy chủ hoặc xác minh
chữ ký có thể không hoạt động. Không thay đổi gói đăng ký thật trên máy chủ.

## Link cài module

- [cuios.shop — Locket Gold](https://raw.githubusercontent.com/Tomqtx11/cuios.shop/refs/heads/main/locket-gold/v2.0.0/cuios.shop.sgmodule)
- [cuios.shop — Diagnostic](https://raw.githubusercontent.com/Tomqtx11/cuios.shop/refs/heads/main/locket-gold/v2.0.0/cuios.shop.Diagnostic.sgmodule)

Trong Shadowrocket, mở cấu hình đang dùng → Modules → thêm URL module Gold ở trên.
Tắt module Locket cũ để tránh chạy chồng. Bật HTTPS Decryption/MITM, cài CA do
Shadowrocket của bạn tạo và bật tin cậy CA trong iOS. Bật kết nối, đóng rồi mở Locket.
Tên menu có thể khác theo ngôn ngữ/phiên bản app. Script tải qua HTTPS từ chính repo này.

Nếu không có tác dụng, tắt module Gold rồi bật Diagnostic. Xem log Script trong
Shadowrocket. Diagnostic không sửa request/response; không bật hai module cùng lúc.
Không gửi token, receipt hoặc log chứa thông tin tài khoản khi báo lỗi.

## Bảo vệ mã và kiểm tra

JavaScript phát hành đã làm rối tên biến, tách chuỗi và mã hóa bảng chuỗi RC4.
Không phát hành source map hoặc bản mã nguồn dễ đọc. Mã phải giải mã trên thiết bị
để chạy nên vẫn có thể bị sao chép hoặc phân tích; đây không phải bảo mật tuyệt đối.
Tên cuios.shop là nhãn module/repo, không tự cấu hình tên miền hay DNS.

14 nhóm test logic đã chạy với chính JavaScript sau làm rối trên Node VM mô phỏng
runtime; chưa phải test JavaScriptCore, MITM hoặc ứng dụng Locket trên iPhone.
SHA256 của các file phát hành nằm trong SHA256SUMS.

JavaScript không gọi mạng phụ hoặc lưu token/receipt. Chỉ nhận đúng Locket qua
bundle ID hoặc User-Agent, chỉ sửa endpoint CustomerInfo v1. Response lỗi, schema
lạ và các app khác được bỏ qua. Module khai báo MITM host api.revenuecat.com dùng chung.

Không được phân phối lại dưới danh nghĩa sản phẩm chính thức của Locket,
Shadowrocket hoặc RevenueCat. Bản quyền phần triển khai riêng: cuios.shop, 2026.
