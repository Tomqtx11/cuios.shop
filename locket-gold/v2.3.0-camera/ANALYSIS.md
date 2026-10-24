# V2.3: đối chiếu mã công khai và xác minh lại key Remote Config

V2.2 đã được người dùng thử: chỉ Gold hoạt động, video khoảng 3 giây, thư viện
chỉ có ảnh. Do chưa có log thiết bị, chưa biết camera hook có nhận phản hồi hay không.
Không xem việc Gold hiện lên là bằng chứng các script Firebase đã chạy.

## Phát hiện xác minh được

`probe_remote_key.py` thực thi đoạn ARM64 `0x100d353a8` đến `0x100d353bc`, ngay trước
lời gọi RemoteConfig.jsonValue. Hai thanh ghi Swift String tạo thành **feature_gates**.
Kết quả và giá trị thanh ghi nằm ở `remote-key-probe.json`.

V2.2 chỉ sửa `feature_gates_override`. Key override có thật trong binary, nhưng
việc sửa override không thay thế việc xử lý key chính. V2.3 merge các cổng liên quan
vào cả hai, giữ nguyên các cổng khác. Đây là thiếu sót của bản trước đã xác minh;
chưa chứng minh nó là nguyên nhân duy nhất trên thiết bị.

Các CodingKeys camera đã chạy getter ARM64 ở lần phân tích trước gồm:
`goldMaxDuration`, `srgb`, `telephoto`, `ultraWide`, `volumeButton`. Khi tạo camera
từ mục bị thiếu, v2.2 chỉ thêm thời lượng. V2.3 bổ sung các field boolean còn thiếu,
giữ giá trị hiện có và đồng bộ alias snake_case. Chưa chạy JSONDecoder thật của app
gốc để kết luận thiếu field có làm nó từ chối object hay không.

V2.2 chưa xử lý `$response=null` trong pha request và chỉ chấp nhận bundle header.
V2.3 nhận cả null/undefined; fallback sang User-Agent bắt đầu bằng Locket khi thiếu
bundle, nhưng vẫn từ chối bundle app khác dù UA ghi Locket. Project/endpoint giữ hẹp.

## Đối chiếu tham khảo công khai

Đã đọc các bản được ghim commit, lưu hash tại manifest cục bộ. Không chạy mã đó
trong môi trường có tài khoản và không sao chép nguyên bộ hàng nghìn khóa.

- [K4Z3X0R Locket_30s.js](https://github.com/K4Z3X0R/NguyenNgocAnhTu/blob/53640c28de95fcc00d04ac9d69e2a91b9130a497/Module/js/Locket_30s.js):
  có xử lý request null, feature_gates, camera đủ field và tự dựng UPDATE khi thiếu body.
- [luongkun bản 60s](https://github.com/luongkun/locket/blob/8fcd0752ef2b0895fda0ec7538dff7e238cf23d1/locket_fire_base_config60sfull.js):
  có key feature_gates và nhiều khóa camera. Tên file/mô tả không xác minh thành công trên iPhone.
- [minhbeovippro LocketRC](https://github.com/minhbeovippro/Locket/blob/96923ef9fb2d1fe5e55d0c33070553dcf3f33304/LocketRC.js):
  cùng hướng can thiệp Firebase, có nhiều thay đổi giao diện/quảng cáo ngoài yêu cầu.

V2.3 chỉ thực hiện phần có liên quan tới yêu cầu. Các khóa video khác được thêm
hàng loạt trong script công khai không được mặc định coi là khóa app thật sự đọc.
Không đưa banner/liên kết ngoài, theo dõi hay can thiệp dịch vụ khác vào module này.

## Cache/activate đối chiếu mã SDK

[RCNConfigContent.m](https://github.com/firebase/firebase-ios-sdk/blob/main/FirebaseRemoteConfig/Sources/RCNConfigContent.m)
phân biệt UPDATE/NO_CHANGE; UPDATE thay thế danh sách config đã fetch. Vì vậy không
đổi một NO_CHANGE rỗng thành template chỉ có vài khóa: có thể mất các giá trị khác.

V2.3 chỉ tái dùng template gốc đầy đủ đã quan sát từ UPDATE hợp lệ trong 1 giờ,
cùng phiên bản UA của app. Không có template phù hợp thì bỏ qua và ghi lý do.
Cache nằm trong persistent store của Shadowrocket trên thiết bị, không gửi đi đâu;
chỉ chứa payload Remote Config, không lưu request/header/token/receipt. Không hoạt
động được persistent store vẫn xử lý UPDATE đầy đủ, nhưng không tái dùng NO_CHANGE.

[FIRRemoteConfig.m](https://github.com/firebase/firebase-ios-sdk/blob/main/FirebaseRemoteConfig/Sources/FIRRemoteConfig.m)
kiểm tra lastETagUpdateTime trước khi activate. V2.3 tạo ETag mới ở phản hồi đã sửa,
bỏ validator đó trong request kế tiếp và giữ templateVersion của server. Không
giả lập templateVersion bằng thời gian hiện tại và không ép được app gọi fetch/activate.

## Cách kiểm tra trên máy

1. Cập nhật module cố định, kiểm tra tên hiện **Camera v2.3**, tải lại script.
2. Tắt các module Locket chạy trùng, kiểm tra MITM/certificate, mở lại Locket.
3. Nếu Shadowrocket cho phép thông báo, lần patch thành công đầu tiên sẽ có thông
   báo **Đã sửa phản hồi cấu hình camera**. Chỉ xác nhận script sửa JSON, chưa xác
   nhận UI hoặc máy chủ. Khi persistent store không ghi được, bỏ thông báo để tránh lặp.
4. Kiểm tra bộ chọn video trước, sau đó video ngắn có tiếng, rồi mới tăng thời lượng.
5. Nếu chưa có tác dụng, gửi riêng các dòng `[cuios.shop Camera v2.3]` và phiên bản
   app/iOS. Không cần log toàn bộ HTTP hoặc thông tin tài khoản.

Không có thông báo và không có log: chưa chứng minh camera script chạy, cần kiểm
tra đường tải script, MITM hoặc app chưa fetch do cache. Có PATCHED nhưng UI không
đổi: cần quan sát activate và các kiểm tra native. Chưa thể kết luận chắc chắn chỉ
Shadowrocket đủ để đạt mọi chức năng của IPA.

13 nhóm test logic chạy trên source và bản làm rối; kiểm tra null request, key chính,
field camera, giới hạn app/project, cache đầy đủ/TTL/phiên bản, không rò token vào log,
diagnostic chỉ đọc, thông báo một lần, giữ metadata và phản hồi lỗi. Chưa thử iPhone.
