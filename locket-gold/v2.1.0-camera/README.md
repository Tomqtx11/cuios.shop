# cuios.shop — Camera 60s + Mic, v2.1.0 thử nghiệm

Đây là bản thử nghiệm **đặt tham số qua Remote Config**, chưa xác nhận app gốc quay
đủ 60 giây, thu được âm thanh hoặc đăng clip có tiếng. Không phải bản chuyển pipeline
video native từ IPA. Bản Gold v2 cũ vẫn được giữ nguyên trong repo.

## Cài bản thử nghiệm

- [Module Gold + Camera](https://raw.githubusercontent.com/Tomqtx11/cuios.shop/refs/heads/main/locket-gold/v2.1.0-camera/cuios.shop.sgmodule)
- [Module Diagnostic](https://raw.githubusercontent.com/Tomqtx11/cuios.shop/refs/heads/main/locket-gold/v2.1.0-camera/cuios.shop.Diagnostic.sgmodule)

Tắt module Locket cũ, thêm link Gold + Camera vào Modules trong cấu hình Shadowrocket
đang dùng. Module mới đã gồm phần Gold, không bật chồng module cũ. Cập nhật script và
bật HTTPS Decryption/MITM với CA của chính Shadowrocket đã được iOS tin cậy.
Danh sách MITM mới có api.revenuecat.com và firebaseremoteconfig.googleapis.com.

Trong Cài đặt iPhone, tìm Locket và bật **Microphone** cùng Camera. Config không thể
tự cấp quyền mic. Nếu Locket có nút tắt tiếng khi quay/gửi, cần kiểm tra trạng thái
nút đó trong app.

Đóng rồi mở app để thử. App phải tự fetch và activate Remote Config mới thì cờ mới
có cơ hội áp dụng. Việc mở lại app hoặc bật VPN không bảo đảm có fetch ngay: SDK
có cache/throttle. Module chỉ bỏ HTTP validators khi có request; không thay đổi
minimumFetchInterval bên trong app và không tự tạo request bằng tài khoản của bạn.

## Thay đổi có căn cứ trong dự án

- Bên trong entries.camera, đặt gold_max_duration thành số 60 và giữ các trường khác.
- Đặt entries.camera_microphone_enabled thành chuỗi true theo schema Remote Config.
- Chỉ áp dụng POST fetch namespace firebase của project 641029076083/locket-4252a,
  đồng thời yêu cầu header X-Ios-Bundle-Identifier là com.locket.Locket.
- Chỉ sửa HTTP 200 có entries đầy đủ và camera là JSON object. Không tự dựng
  template cho NO_CHANGE, phản hồi lỗi hoặc bản không có camera.
- Không thay đổi video, audio track, file upload, quyền máy chủ hoặc giới hạn import native.

Hai khóa được tìm thấy trong LocketConfig.plist của **IPA mod 2.68.0**, cùng bundle
và project trong GoogleService-Info.plist. Đây là bằng chứng về cấu hình của IPA,
chưa chứng minh app gốc đọc và chấp nhận toàn bộ giá trị đó. App Store Việt Nam khi
đối chiếu ngày 24/10/2026 đang hiển thị 2.69.0. Chưa xác nhận trên 2.68.0 gốc hoặc 2.69.0.

Mã SDK Firebase xác nhận endpoint fetch, bundle header, schema entries và cache.
Tham khảo [Firebase iOS SDK](https://github.com/firebase/firebase-ios-sdk/blob/main/FirebaseRemoteConfig/Sources/RCNConfigFetch.m)
và [App Store](https://apps.apple.com/vn/app/locket-widget/id1600525061).

## Cách xác nhận trên iPhone

1. Bật module mới và quyền mic, kiểm tra app hiện Gold.
2. Xem log Script: dòng camera.gold_max_duration=60 chỉ xác nhận đã sửa response,
   không xác nhận camera áp dụng. Không có log có thể do app chưa fetch hoặc MITM chưa khớp.
3. Thử quay video 5 giây có lời nói; kiểm tra tiếng trước khi gửi nếu app cho phép.
4. Gửi cho bạn bè hoặc tài khoản thử được phép, xác nhận bên nhận nghe được tiếng.
5. Sau đó thử 15, 30 rồi 60 giây và kiểm tra thời lượng/âm thanh bên nhận.

Nếu vẫn dừng sớm, có thể app giới hạn bằng mã native hoặc không dùng khóa này.
Nếu có tiếng trước khi gửi nhưng bên nhận mất tiếng, cần phân biệt bước xuất/upload
và phát video; module này không khẳng định đã sửa được các bước đó. Cần phiên bản
app, iOS, hiện tượng và log đã ẩn thông tin tài khoản để chẩn đoán tiếp.

Dùng Diagnostic khi cần: tắt Gold + Camera rồi bật Diagnostic. Cả hai script Gold
và Camera ở Diagnostic đều không sửa traffic. Đừng chia sẻ token, receipt hoặc
Firebase installation token. Log script chỉ ghi nhãn trạng thái, không ghi các giá trị này.

Muốn quay lại bản cũ, tắt module này và bật lại Gold v2. Remote Config đã activate
có thể còn trong cache của Locket cho tới lần fetch/activate bản gốc tiếp theo;
tắt VPN không tự xóa cache bên trong ứng dụng.

## Mã phát hành và kiểm tra

JavaScript được làm rối, không phát hành source map hoặc bản dễ đọc. Mã chạy trên
client vẫn có thể bị phân tích hoặc sao chép; không có bảo đảm chống sao chép tuyệt đối.
9 nhóm test Camera và 14 nhóm test Gold đã chạy với chính file JS làm rối trên
runtime Node VM mô phỏng. Chưa chạy iOS, JavaScriptCore, camera/mic hoặc máy chủ upload.
SHA256 của file phát hành nằm trong SHA256SUMS. Không có token GitHub trong bộ file.
