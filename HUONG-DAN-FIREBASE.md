# Nhật ký Ba Nhất — hướng dẫn kỹ thuật

Kiến trúc miễn phí, tính cho khoảng 50.000 hội viên:

| Phần | Dịch vụ | Ghi chú |
|---|---|---|
| Trang web (HTML, ảnh, phông) | **Cloudflare Workers** (file tĩnh: miễn phí, không giới hạn lượt truy cập) | **https://nhat-ky-ba-nhat.hoanganhfallinginlove.workers.dev**, tự cập nhật mỗi lần push nhánh `main` (cấu hình `wrangler.jsonc`) |
| Đăng nhập | **Firebase Authentication** (project `nhat-ky-ba-nhat-94047`) | Email + mật khẩu, có "Quên mật khẩu?" |
| Dữ liệu | **Firestore** (gói Spark miễn phí) | Mỗi người **1 document** `diaries/{uid}` |
| Link cũ `nhat-ky-ba-nhat-94047.web.app` | Firebase Hosting | Chuyển hướng 301 sang link Cloudflare (`firebase.json` → `redirects`) |

## Cấu trúc thư mục
- `public/` — toàn bộ trang được đưa lên mạng (`index.html`, `images/` WebP, `fonts/`, `docs/img.png`,
  `_headers` cho Cloudflare lưu ảnh lâu dài trên trình duyệt).
- `firestore.rules` — quy tắc bảo mật Firestore (deploy bằng lệnh, không cần dán tay).
- `firebase.json`, `.firebaserc` — cấu hình Firebase CLI.
- `docs/anh-goc/` — ảnh PNG gốc của các trang nhật ký (không đưa lên mạng).

## Mô hình dữ liệu
```
diaries/{uid} = {
  uid, email (chữ thường), ownerName,
  entries: [ { id, createdAt, author, date, title, body }, ... ],   // tối đa 100 bài
  count,        // = entries.length, để admin đếm tổng số bài trên máy chủ
  updatedAt     // giờ máy chủ, để admin xếp theo mới cập nhật
}
admins/{uid} = { role: "admin" }   // tạo tay trên Console
```
Mỗi lần mở trang chỉ tốn **1 lượt đọc** (đọc cuốn của mình); người dùng thường không tốn lượt nào
cho việc kiểm tra quyền admin. Admin tải 20 cuốn mỗi lần (nút *Tải thêm*), tìm theo email chạy
trên máy chủ, số liệu tổng dùng phép đếm trên máy chủ.

## Hạn mức gói miễn phí (Spark) cần theo dõi
Firestore → **Usage**: khoảng 50.000 lượt đọc/ngày (≈ 50.000 lượt mở trang), 20.000 lượt ghi/ngày,
**1 GB dữ liệu** (≈ 300.000 bài cỡ trung bình). Vượt hạn mức trong ngày thì đăng nhập/tải bài bị
lỗi tới khi hạn mức làm mới (khoảng 14–15h giờ Việt Nam). Gần chạm hạn mức thì nâng lên **Blaze**
(giữ nguyên phần miễn phí, chỉ trả phần vượt), không cần sửa code.

## Cập nhật Rules / trang chuyển hướng
```
npx -y firebase-tools@15.34.0 deploy --only firestore:rules,hosting
```
Lần đầu trên máy mới cần đăng nhập: `npx -y firebase-tools@15.34.0 login`.
Trang chính trên Cloudflare **không cần lệnh**: push lên `main` là Cloudflare tự đưa bản mới lên.

## Cấp quyền admin
1. Người đó đăng ký tài khoản trên trang như bình thường.
2. Firebase Console → **Authentication → Users** → sao chép **User UID**.
3. **Firestore → Data** → collection `admins` → **Add document**: Document ID = UID vừa sao chép,
   trường `role` = `admin` → **Save**.
4. Người đó mở trang với đuôi `?admin` (hoặc bấm link **Quản trị** ở chân trang) rồi đăng nhập.
   Từ lần sau, máy đó tự hiện nút **⚙ Quản trị**.

Thu hồi quyền: xóa document của người đó trong `admins`. Rules chặn ghi `admins` từ web nên
không ai tự cấp quyền cho mình được.

## Cấu hình một lần trên Firebase Console
- **Authentication → Sign-in method**: bật Email/Password.
- **Authentication → Settings → Authorized domains**: có `nhat-ky-ba-nhat.hoanganhfallinginlove.workers.dev` và
  `nhat-ky-ba-nhat-94047.web.app`.
- **Authentication → Settings → Sign-up quota**: tăng trước các buổi đăng ký tập trung (mặc định
  khoảng 100 tài khoản/giờ cho cùng một địa chỉ IP).
- **Authentication → Templates**: sửa nội dung email đặt lại mật khẩu bằng tiếng Việt nếu muốn.

## Chạy thử trên máy
Mở thư mục `public/` bằng một static server (ví dụ **Live Server** của VS Code). Không mở bằng
`file://` vì trình duyệt chặn tải Firebase và phông chữ.

## Ghi chú bảo mật
`apiKey` của Firebase web **không phải mật khẩu**, công khai được. Quyền truy cập do `firestore.rules`
và Authentication kiểm soát: mỗi người chỉ đọc/ghi cuốn của mình; Rules kiểm tra đúng chủ, tối đa
100 bài, không cho thêm trường lạ; chỉ admin đọc/sửa/xóa được cuốn của người khác.
