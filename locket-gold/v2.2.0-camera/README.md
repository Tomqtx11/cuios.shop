# cuios.shop — Camera v2.2, kiểm tra lại sau phản hồi thiết bị

Người dùng thử v2.1: Gold hiện lên, chỉ quay 3–5 giây, không có mic/tiếng và chỉ chọn
được ảnh từ thư viện. **V2.1 chưa đạt yêu cầu. V2.2 sửa các thiếu sót có bằng chứng
trong script và bổ sung chẩn đoán; chưa xác nhận khắc phục trên iPhone.**

[Link module v2.2](https://raw.githubusercontent.com/Tomqtx11/cuios.shop/refs/heads/main/locket-gold/v2.2.0-camera/cuios.shop.sgmodule)
 · [Diagnostic không sửa traffic](https://raw.githubusercontent.com/Tomqtx11/cuios.shop/refs/heads/main/locket-gold/v2.2.0-camera/cuios.shop.Diagnostic.sgmodule)

## Phát hiện và sửa đổi

1. V2.1 thiếu cờ internal_camera_roll_video_upload_enabled; v2.2 bổ sung để thử mở
   video trong picker. Chỉ chọn được ảnh là vấn đề trước bước upload.
2. Getter ARM64 trả về goldMaxDuration; v2.1 chỉ ghi gold_max_duration. V2.2 đặt
   hai dạng cùng số 60, vì chưa biết decoder của phiên app gốc xử lý snake_case thế nào.
3. Có cổng cameraRoll, longerVideo, video riêng trong FeatureGates. V2.2 merge Mode
   enabled cho ba nhóm này trong feature_gates_override, giữ cổng khác.
4. V2.1 bỏ qua cả mic nếu mục camera thiếu. V2.2 xử lý cờ mic/picker độc lập;
   camera hoặc gates hỏng chỉ làm bỏ qua mục đó. Không dựng template cho NO_CHANGE.
5. Log ghi request khớp, HTTP/state, trạng thái các khóa trước sửa và phần đã sửa.
   Không ghi token/receipt/query/user ID hoặc toàn bộ response.

Xem [phân tích và giới hạn](ANALYSIS.md). Bằng chứng từ binary **IPA mod 2.68.0**,
không phải executable gốc chưa sửa. Có 10 nhóm test logic trên source và JS làm rối;
getter CodingKeys/Mode đã chạy bằng giả lập ARM64. Không phải test camera/mic thật.

## Cài và lấy kết quả

1. Tắt module Locket v2/v2.1 và mọi module RevenueCat/Locket chạy trùng. Thêm link v2.2.
   Bản này đã gồm script Gold cũ. Cập nhật script và bật kết nối Shadowrocket.
2. Kiểm tra HTTPS Decryption/MITM và CA do chính Shadowrocket tạo đã được iOS tin cậy.
   Host có api.revenuecat.com và firebaseremoteconfig.googleapis.com.
3. Trong Cài đặt iPhone, tìm Locket và kiểm tra quyền Camera/Microphone nếu có.
   Config không tự cấp quyền iOS hoặc tự bật nút mic trong app.
4. Đóng rồi mở Locket; xem log Script có tiền tố **[cuios.shop Camera v2.2]**.
   App có thể giữ cache và chưa fetch/activate ngay; mở lại không bảo đảm có fetch.
5. Thử mở bộ chọn thư viện trước: có xuất hiện video không? Thử clip ngắn có lời nói,
   rồi tăng thời lượng. Kiểm tra cả tiếng/thời lượng ở máy người nhận nếu gửi được.

## Đọc log

- Không có Camera v2.2: chưa chứng minh script camera chạy; cần kiểm tra module/JS/MITM
  và việc app có gửi fetch hay không. Gold hoạt động không chứng minh script Camera chạy.
- fetch.request matched: request khớp project/bundle; chưa phải response được sửa.
- fetch.state=NO_CHANGE hoặc thiếu entries: không sửa template; cần lần fetch đầy đủ.
- observed: là trạng thái các tham số nhận được trước khi sửa, không phải giá trị app đã activate.
- patched.response: chỉ xác nhận sửa JSON. camera60/gates=false nghĩa là mục đó bị giữ lại do schema lạ.

Nếu vẫn không thay đổi, gửi **chỉ các dòng Camera v2.2**, phiên bản app và iOS để
phân biệt config chưa chạy, chưa activate, decoder hoặc gate native. Không cần log
HTTP đầy đủ, token GitHub, Firebase installation token hoặc thông tin đăng nhập.

Không tự giả lập upload thành công, không sửa xác thực/Firestore/Storage và không
vá mã camera native. Chưa bảo đảm chỉ dùng VPN đạt tính năng IPA. Tắt module không
tự xóa Remote Config đã activate; cache có thể tồn tại tới lần fetch/activate gốc.

Mã làm rối giúp khó đọc hơn, không ngăn sao chép tuyệt đối. Bản v2/v2.1 được giữ nguyên
trong repo để đối chiếu. SHA256 của bộ file tại SHA256SUMS.
