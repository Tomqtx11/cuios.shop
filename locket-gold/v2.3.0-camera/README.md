# cuios.shop — Camera v2.3

V2.2 được người dùng báo chưa hoạt động ngoài Gold. V2.3 sửa key feature_gates
được xác minh bằng đoạn ARM64, bổ sung field camera, nhận request có response null,
fallback UA khi bundle header thiếu và xử lý cache/ETag theo mã SDK Firebase.
Xem [phân tích chi tiết](ANALYSIS.md) và [probe key thật](remote-key-probe.json).

**Chưa xác nhận quay đủ 60s, mic hoặc chọn/gửi video trên iPhone.** Test logic không
thay thế thử thiết bị. Không tạo audio track, cấp quyền iOS hoặc vá executable.

## Link cố định

[Module cuios.shop](https://raw.githubusercontent.com/Tomqtx11/cuios.shop/refs/heads/main/cuios.shop.sgmodule)
 · [Diagnostic chỉ đọc](https://raw.githubusercontent.com/Tomqtx11/cuios.shop/refs/heads/main/cuios.shop.Diagnostic.sgmodule)

Giữ URL cũ nếu đang dùng link cố định trên. Cập nhật module và tải lại script,
kiểm tra tên module có **Camera v2.3**. Các file JS dùng đường dẫn phiên bản mới
để tránh chạy cache v2.2. Không cần cài lại Shadowrocket/Locket.

Tắt các module Locket/RevenueCat chạy trùng. Kiểm tra MITM và CA của chính
Shadowrocket đã được iOS tin cậy; module dùng hai host RevenueCat và Firebase Remote Config.
Mở lại Locket. SDK có thể chưa fetch/activate ngay do cache; script không tự ép được.

Khi script thực sự sửa phản hồi camera, nó gửi một thông báo qua Shadowrocket:
**Đã sửa phản hồi cấu hình camera**. Thông báo chỉ xuất hiện một lần cho bản v2.3
khi persistent store ghi được và thông báo hệ thống được cho phép. Không phải bằng
chứng camera đã áp dụng. Nếu không thấy, xem log với tiền tố Camera v2.3.

Kiểm tra bộ chọn thư viện có video không, rồi thử clip ngắn có tiếng trước khi
thử 60s. Nếu vẫn lỗi, gửi riêng dòng log Camera v2.3 và phiên bản app/iOS.

V2.3 lưu template cấu hình gốc đã fetch đầy đủ trong persistent store trên iPhone
tối đa 1 giờ để xử lý NO_CHANGE. Cache gắn với phiên bản UA; không có cache phù hợp
thì không tự tạo template một phần. Không lưu request/token/receipt, không gửi cache
ra máy chủ. Diagnostic không sửa traffic, không ghi cache hoặc gửi thông báo.

Gold script giữ nguyên từng byte của bản đang hoạt động. Mã Camera đã làm rối;
bản dễ đọc và công cụ build giữ trên máy. Làm rối không bảo đảm chống sao chép tuyệt đối.
13 nhóm test logic đã chạy trên source và mã làm rối; bộ file có SHA256SUMS.
