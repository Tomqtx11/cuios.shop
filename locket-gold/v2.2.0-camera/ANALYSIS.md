# Phân tích lại sau khi thử Camera v2.1 trên thiết bị

Người dùng báo: Gold được kích hoạt; camera chỉ quay khoảng 3–5 giây, không có
tiếng/nút mic; thư viện chỉ cho chọn ảnh. Chưa có lần chọn/gửi video từ thư viện,
vì vậy **chưa có bằng chứng lỗi upload hoặc máy chủ từ chối video**.

## Kết luận có bằng chứng

1. Bản v2.1 thiếu `internal_camera_roll_video_upload_enabled` và chưa xử lý
   `feature_gates_override`. Thay hai tham số của v2.1 chưa bao phủ các cổng camera.
2. V2.1 bỏ qua toàn bộ phản hồi nếu thiếu `entries.camera`. Khi đó cờ mic cũng
   không được sửa. Đây là thiếu sót trong script; chưa có log để biết thiết bị
   người dùng có rơi đúng nhánh này hay không.
3. Getter ARM64 của CameraConfig.CodingKeys thực sự trả về `goldMaxDuration`.
   V2.1 chỉ đặt `gold_max_duration`. Chưa quan sát keyDecodingStrategy của phiên
   app gốc, vì vậy không khẳng định snake_case chắc chắn là nguyên nhân; v2.2
   đặt hai dạng cùng số 60 để tránh phụ thuộc sự khác biệt này.
4. Bật Gold phía RevenueCat không chứng minh các cổng video và camera-roll mở.
   Metadata cho thấy các trạng thái và mô hình cổng tính năng riêng.

## Phương pháp

Chỉ đọc executable trong IPA mod 2.68.0:
`Locket.app/Locket (IPA mod 2.68.0)`.
Hash nguồn và kết quả xref nằm trong `camera-analysis.json`.
Không có executable App Store chưa sửa, log thiết bị hoặc phiên tài khoản.

`analyze_camera.py` đọc Mach-O, LC_FUNCTION_STARTS, Swift type/field descriptors,
và tìm các cặp ADRP/ADD gần chuỗi cấu hình. Các xref gắn nhãn `bias -32` chỉ là
ứng viên: địa chỉ có thể trùng chuỗi lân cận; phải xem lệnh/helper cuối trước khi
gán tên khóa. Không dùng mọi ứng viên đó làm bằng chứng độc lập.

`probe_coding_keys.py` thực thi các getter ARM64 thuần bằng Unicorn với các giá trị
enum đầu vào, không mock JSONDecoder hoặc chạy camera. Kết quả tại
`native-coding-probe.json`:

| Thành phần | Getter/descriptor | Kết quả xác minh |
| --- | --- | --- |
| CameraConfig.CodingKeys | `0x100315488` | `goldMaxDuration`, `srgb`, `telephoto`, `ultraWide`, `volumeButton` |
| PurchasesClient.FeatureGates.CodingKeys | `0x100d366d8` | Có `cameraRoll`, `longerVideo`, `video` và các cổng khác |
| PurchasesClient.FeatureGates.Mode.rawValue | `0x100d365b8` | `disabled`, `enabled`, `subscriber` |
| Camera.Camera.State | `0x1021d7108` | Có các trường riêng nêu dưới đây |

Các trường Camera.State liên quan:

- `_cameraRollVideoUploadEnabled`
- `_goldMaxDuration`, `_longerVideosEnabled`
- `_microphoneButtonEnabled`, `_microphoneEnabled`, `_microphoneConfigured`
- `_microphoneAuthorizationStatus`

Xref trực tiếp:

- `internal_camera_roll_video_upload_enabled` tại `0x10022a30c` và `0x10023e9a8`.
- `camera_microphone_enabled` tại `0x10022a148` và `0x10023eb08`.
- `feature_gates_override` có trong binary cạnh mã `RemoteConfig+Purchases.swift`.
  Chuỗi key snake_case của các cổng cũng xuất hiện dưới dạng Swift String immediate.

## Giới hạn native và mic của IPA

`../v19_port/build/port-report.json` của dự án gốc ghi những bản vá executable riêng:

- `0x100233258`: thay lệnh để thời lượng video nhập là số double 60.
- `0x100250c0c`: bỏ một nhánh kiểm tra camera-roll.
- `0x100250e70`: thay một lời gọi bằng `mov w0, #1`.
- `0x10034b1dc`: thay chính sách thời lượng trong mã native.
- Thư viện `CUIOSVideo60.dylib` can thiệp import/export/composition.

Đây là bằng chứng IPA có sửa native ngoài Remote Config. Không chứng minh mọi
nhánh đó đều cần vá trên app gốc nếu các cổng cấu hình đã mở. Không dùng các địa
chỉ của bản 2.68.0 để suy đoán offset của phiên bản khác.

Trong binary IPA mod còn có `mh_mic_enabled_v16` được đọc/lưu như trạng thái riêng.
Đây không phải tên khóa Remote Config gốc đã xác minh; v2.2 không thêm nó vào JSON.
Các lỗi native về xin quyền âm thanh, thêm audio input, cấu hình audio session và
ghi audio frame tồn tại riêng. Cờ `camera_microphone_enabled` không tự gọi thay
toàn bộ các bước đó và không tự cấp quyền riêng tư iOS.

## V2.2 sửa gì

- Cờ mic và picker được xử lý độc lập, kể cả camera/gates thiếu hoặc không giải mã được.
- Chỉ thêm `camera` tối thiểu khi đang có một template `entries` đầy đủ, hợp lệ;
  không tự tạo template cho `NO_CHANGE`, thiếu entries hoặc HTTP lỗi.
- Đặt `goldMaxDuration` và `gold_max_duration` cùng số 60, giữ trường camera khác.
- Bổ sung `internal_camera_roll_video_upload_enabled=true`.
- Merge ba cổng video/camera-roll/video dài thành Mode `enabled`; cả tên camelCase
  và snake_case dùng cùng giá trị. Đây là lựa chọn tương thích decoder, không phải
  khẳng định app cần cả hai cách viết. Các cổng không liên quan được giữ nguyên.
- Log trạng thái theo danh sách giá trị cho phép; không in URL/query/API key,
  Firebase installation token, user ID hoặc toàn bộ nội dung phản hồi.

10 nhóm test logic chạy trên mã nguồn và mã làm rối. Test kiểm tra các nhánh thiếu
camera, lỗi từng mục, bảo toàn metadata/gate khác, phạm vi project/bundle, diagnostics,
headers, xử lý lỗi và đối chiếu getter ARM64. Không phải test camera trên iPhone.

## Dữ liệu còn cần để chốt nguyên nhân thực tế

Dùng module v2.2, tắt các module Locket cũ, mở lại app rồi xem log Script:

| Log | Ý nghĩa và bước tiếp theo |
| --- | --- |
| Không có dòng Camera v2.2 | Chưa chứng minh script chạy: kiểm tra module, URL JS, MITM/certificate hoặc app chưa fetch do cache |
| `fetch.request matched` nhưng không có response | Xem lỗi TLS/kết nối/phần chạy response trong Shadowrocket |
| `fetch.state=NO_CHANGE` hoặc thiếu entries | Chưa sửa config; cần lần fetch đầy đủ, không giả lập template một phần |
| `observed: camera=missing` | Xác nhận v2.1 sẽ bỏ qua mic; v2.2 có xử lý độc lập |
| `patched.response ...` | JSON đã sửa; chưa chứng minh app đã activate hoặc đọc các khóa đó |
| Đã patched nhưng vẫn không đổi UI | Cần xác định fetch/activate, phiên bản app, giá trị local và các gate native; không kết luận máy chủ upload chặn |

Người dùng cần cung cấp log **chỉ các dòng `[cuios.shop Camera v2.2]`**, phiên bản
app/iOS và việc có mục Microphone trong Cài đặt Locket. Không cần token hoặc log
toàn bộ HTTP. Có thể chụp lại màn log nếu thuận tiện.

Nếu JSON đã được activate mà vẫn chỉ 3–5 giây/không có mic, bản vá bằng Remote Config
không đủ trên phiên app đó; muốn thay nhánh native cần IPA hoặc môi trường có thể
hook app. Chưa có bằng chứng để bảo đảm chỉ Shadowrocket sẽ đạt đủ chức năng IPA.
