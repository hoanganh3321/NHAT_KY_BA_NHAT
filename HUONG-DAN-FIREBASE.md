# Chia sẻ nhật ký chung qua Firebase

Trang đã cấu hình sẵn Firebase (project `nhat-ky-ba-nhat`). Nhật ký mọi người viết được
lưu chung trên **Firestore** và hiện realtime cho tất cả người xem. Nếu mất mạng / lỗi kết
nối, trang tự chuyển sang lưu tạm trên máy để không bị hỏng.

## Điều kiện để chạy chế độ chung
1. **Firestore Database đã bật**: Firebase Console → Build → Firestore Database → Create
   (production mode, vùng `asia-southeast1`). ✅ Đã bật.
2. **Rules cho phép đọc/ghi** (tab Rules → Publish):

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /entries/{doc} {
         allow read, write: if true;
       }
     }
   }
   ```
   ✅ Đã mở (kiểm tra bằng đọc/ghi thử qua REST — OK).

## Chạy / kiểm tra cục bộ
Trang phải mở qua HTTP (không mở bằng `file://`, vì import Firebase qua CDN sẽ bị chặn).
Dùng bất kỳ static server nào, ví dụ tiện ích **Live Server** của VS Code, rồi mở trang.



Thấy dòng **“🌐 Nhật ký được chia sẻ chung…”** dưới mục *Viết thêm nhật ký của bạn* là OK.

## Xuất bản GitHub Pages
Repo: `hoanganh3321/NHAT_KY_BA_NHAT`, nhánh `main`.
1. GitHub → repo → **Settings → Pages**.
2. **Source: Deploy from a branch** → Branch `main` → thư mục `/ (root)` → **Save**.
3. Chờ ~1 phút, trang chạy tại: `https://hoanganh3321.github.io/NHAT_KY_BA_NHAT/`

Firebase Firestore hoạt động với mọi tên miền (kiểm soát bằng Rules, không cần khai báo
authorized domain), nên GitHub Pages dùng được ngay.

## Ghi chú bảo mật
`apiKey` của Firebase web **không phải mật khẩu**, công khai được — đây là thiết kế của
Firebase. Quyền truy cập kiểm soát ở **Rules**. Rules hiện cho phép mọi người thêm/sửa/xóa,
đủ dùng cho nhóm nội bộ. Muốn siết chặt (chỉ đọc công khai, ghi cần đăng nhập) thì báo để chỉnh.
