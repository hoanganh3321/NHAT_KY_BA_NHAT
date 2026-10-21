# Nhật ký cá nhân qua Firebase (đăng nhập tài khoản)

Trang đã cấu hình sẵn Firebase (project `nhat-ky-ba-nhat`). Mỗi người **đăng ký / đăng nhập**
bằng email + mật khẩu, và **chỉ thấy + sửa được cuốn nhật ký của riêng mình** — không lẫn với
người khác. Dữ liệu lưu trên **Firestore**, mỗi bài gắn `uid` của chủ tài khoản. Nếu mất mạng /
lỗi kết nối, trang tự chuyển sang lưu tạm trên máy (1 cuốn cục bộ, không cần đăng nhập).

## ⚠️ 2 bước BẮT BUỘC làm trên Firebase Console (nếu chưa làm)

### 1. Bật đăng nhập Email/Mật khẩu
Firebase Console → **Build → Authentication → Get started** →
tab **Sign-in method** → chọn **Email/Password** → bật **Enable** → **Save**.

Nếu chưa bật, khi đăng nhập sẽ báo lỗi: *"Chưa bật đăng nhập Email/Mật khẩu…"*.

### 2. Cập nhật Rules Firestore (mỗi người chỉ đọc/ghi bài của mình, admin quản lý tất cả)
Firebase Console → **Firestore Database → tab Rules → Publish**:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       function isAdmin() {
         return request.auth != null &&
                exists(/databases/$(database)/documents/admins/$(request.auth.uid));
       }
       function isOwner() {
         return request.auth != null && resource.data.uid == request.auth.uid;
       }

       match /entries/{doc} {
         allow read:           if isAdmin() || isOwner();
         allow create:         if request.auth != null && request.resource.data.uid == request.auth.uid;
         allow update, delete: if isAdmin() || isOwner();
       }

       // Danh sách admin: mỗi người chỉ đọc được document của chính mình, không ai ghi từ web
       match /admins/{uid} {
         allow read:  if request.auth != null && request.auth.uid == uid;
         allow write: if false;
       }
     }
   }
   ```

   Rule này bảo đảm: phải đăng nhập mới đọc/ghi được. Người dùng thường **chỉ đụng tới bài có
   `uid` trùng với tài khoản của mình**. Chỉ tài khoản có trong collection `admins` mới xem, sửa
   và xóa được bài của mọi người.

### 3. Cấp quyền admin (màn Quản trị)
1. Tài khoản admin đăng ký trên trang như bình thường.
2. Firebase Console → **Authentication → Users** → sao chép **User UID** của tài khoản đó.
3. **Firestore Database → Start collection** → Collection ID: `admins` → Document ID: dán
   **UID** vừa sao chép → thêm 1 trường bất kỳ (ví dụ `role` = `admin`) → **Save**.
4. Đăng nhập lại trên trang: cạnh nút *Đăng xuất* sẽ có nút **⚙ Quản trị**. Màn này liệt kê bài
   của tất cả người dùng, có tìm kiếm, lọc theo người dùng, Xem / Tải ảnh / Sửa / Xóa và
   **Xuất toàn bộ** ra file JSON.

Muốn thu hồi quyền: xóa document của người đó trong `admins`. Collection này chỉ sửa được trên
Console (Rules chặn ghi từ web), nên không ai tự cấp quyền admin cho mình được.

> Bài đăng từ nay lưu kèm `email` và `ownerName` của người viết để admin biết bài của ai. Bài
> đăng trước đó chỉ hiện theo `uid` (8 ký tự đầu) trong màn Quản trị.

> **Lưu ý dữ liệu cũ:** các bài đăng trước đây (khi chưa có đăng nhập) không có trường `uid`
> nên không hiện với người dùng nào, chỉ admin thấy (nhãn *bài cũ, không có chủ*). Muốn dọn,
> xóa ngay trong màn Quản trị hoặc trong collection `entries` trên Firestore Database.

**Firestore Database** cũng cần đã bật (Build → Firestore Database → Create, vùng
`asia-southeast1`). ✅ Đã bật.

## Chạy / kiểm tra cục bộ
Trang phải mở qua HTTP (không mở bằng `file://`, vì import Firebase qua CDN sẽ bị chặn).
Dùng bất kỳ static server nào, ví dụ tiện ích **Live Server** của VS Code, rồi mở trang.



Thấy khung **Đăng nhập / Đăng ký** dưới mục *Viết thêm nhật ký của bạn* là OK. Đăng ký một
tài khoản thử → viết một bài → đăng xuất rồi đăng nhập tài khoản khác để kiểm tra không thấy
bài của nhau.

## Xuất bản GitHub Pages
Repo: `hoanganh3321/NHAT_KY_BA_NHAT`, nhánh `main`.
1. GitHub → repo → **Settings → Pages**.
2. **Source: Deploy from a branch** → Branch `main` → thư mục `/ (root)` → **Save**.
3. Chờ ~1 phút, trang chạy tại: `https://hoanganh3321.github.io/NHAT_KY_BA_NHAT/`

Firebase Firestore hoạt động với mọi tên miền (kiểm soát bằng Rules, không cần khai báo
authorized domain), nên GitHub Pages dùng được ngay. Với **Authentication**, tên miền
`hoanganh3321.github.io` và `localhost` thường đã nằm sẵn trong danh sách **Authorized domains**
(Authentication → Settings). Nếu đăng nhập báo lỗi `unauthorized-domain`, thêm tên miền đó vào.

## Ghi chú bảo mật
`apiKey` của Firebase web **không phải mật khẩu**, công khai được — đây là thiết kế của
Firebase. Quyền truy cập kiểm soát ở **Rules** + **Authentication**. Với cấu hình mới, mỗi
người phải đăng nhập và chỉ đọc/ghi được nhật ký của chính mình (theo `uid`).
