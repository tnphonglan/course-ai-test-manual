# SRS – Chức năng Đăng nhập (Login)

| Mục | Nội dung |
|---|---|
| Hệ thống | Perfex CRM – Anh Tester Demo |
| Chức năng | Login / Authentication (khu vực Admin) |
| URL | https://crm.anhtester.com/admin/authentication |
| Phiên bản tài liệu | 1.0 (Draft) |
| Ngày lập | 2026-10-08 |
| Người lập | tnphonglan |
| Nguồn thông tin | Khảo sát giao diện (UI) và DOM của trang Login và Forgot Password |

> **Quy ước độ tin cậy**
> - ✅ **Quan sát được** – đã xác nhận trực tiếp trên giao diện / mã HTML của trang.
> - ⚠️ **Giả định** – suy luận theo hành vi chuẩn của Perfex CRM hoặc thực tiễn chung; **cần chạy thực tế để xác nhận** trước khi dùng làm căn cứ chấm Pass/Fail.

---

## 1. Giới thiệu

### 1.1 Mục đích
Tài liệu đặc tả yêu cầu chức năng và phi chức năng của màn hình Đăng nhập, làm cơ sở để phân tích, thiết kế test case và kiểm thử thủ công.

### 1.2 Phạm vi
**Trong phạm vi**
- Màn hình Login: `/admin/authentication`
- Tùy chọn Remember me
- Liên kết và màn hình Forgot Password: `/admin/authentication/forgot_password`
- Điều hướng sau khi đăng nhập thành công / thất bại

**Ngoài phạm vi**
- Đăng ký tài khoản (trang không có chức năng này)
- Đăng nhập khu vực Client (`/authentication/login`)
- Phân quyền chi tiết sau đăng nhập
- Two-Factor Authentication, reCAPTCHA (không hiển thị ở cấu hình hiện tại)

### 1.3 Đối tượng sử dụng
| Vai trò | Mô tả |
|---|---|
| Admin / Staff | Người dùng nội bộ đã có tài khoản, đăng nhập để vào trang quản trị CRM |

### 1.4 Thuật ngữ
| Thuật ngữ | Giải thích |
|---|---|
| CSRF token | Mã chống giả mạo request, gửi kèm form dưới dạng input ẩn |
| Remember me | Tùy chọn duy trì phiên đăng nhập sau khi đóng trình duyệt |
| Session | Phiên làm việc của người dùng sau khi xác thực thành công |

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh
Login là cổng vào duy nhất của khu vực quản trị. Người dùng nhập Email và Password; hệ thống xác thực và cấp phiên làm việc.

### 2.2 Ràng buộc & giả định
- ✅ Tiêu đề tab trình duyệt: **"Perfex CRM | Anh Tester Demo - Login"**.
- ✅ Ngôn ngữ giao diện: tiếng Anh (`lang="en"`).
- ✅ Định danh đăng nhập là **địa chỉ email** (không phải username).
- ✅ Form gửi bằng **POST** tới chính URL `/admin/authentication`, có kèm CSRF token.
- ✅ Form **không** đặt `novalidate` → trình duyệt sẽ tự kiểm tra định dạng của input `type="email"`.
- ✅ Không có nút/liên kết đăng ký tài khoản.
- ⚠️ Đây là môi trường demo dùng chung; dữ liệu và mật khẩu có thể bị người khác thay đổi.

### 2.3 Dữ liệu kiểm thử
| Loại | Email | Password | Ghi chú |
|---|---|---|---|
| Hợp lệ | `admin@example.com` | `123456` | Tài khoản Admin demo do người dùng cung cấp. Chưa được đăng nhập xác minh trong lần khảo sát này |
| Email sai định dạng | `admin@` , `admin.example.com` | bất kỳ | Dùng kiểm tra validate phía trình duyệt |
| Email không tồn tại | `notexist@example.com` | `123456` | |
| Sai mật khẩu | `admin@example.com` | `wrong123` | |

---

## 3. Đặc tả giao diện (UI)

### 3.1 Màn hình Login ✅

| # | Thành phần | Loại | Thuộc tính / Mô tả |
|---|---|---|---|
| 1 | Logo | Hình ảnh + liên kết | Liên kết về trang chủ `https://crm.anhtester.com/` |
| 2 | Tiêu đề | Heading | "Login" |
| 3 | Email Address | Textbox `type="email"` | `id=email`, `name=email`; **tự động focus** khi tải trang; không có `required`, `maxlength`, placeholder |
| 4 | Password | Textbox `type="password"` | `id=password`, `name=password`; ký tự hiển thị dạng ẩn (•); không có `required`, `maxlength`, placeholder; không có nút hiện/ẩn mật khẩu |
| 5 | Remember me | Checkbox | `id=remember`, `name=remember`; **mặc định không được chọn**; click vào label cũng thay đổi trạng thái checkbox |
| 6 | Login | Button `type="submit"` | Nút chính (màu primary, rộng toàn khối); gửi form |
| 7 | Forgot Password? | Liên kết | Điều hướng tới `/admin/authentication/forgot_password` |
| 8 | CSRF token | Input ẩn | `name=csrf_token_name`; không hiển thị với người dùng |

**Nhận xét**
- Nhãn (label) gắn đúng với input qua thuộc tính `for` → click vào nhãn sẽ focus vào ô tương ứng.
- Thứ tự Tab: Email → Password → Remember me → Login → Forgot Password?.
- Nhấn **Enter** trong ô Email hoặc Password sẽ gửi form (do nút `type="submit"`).
- Không có giới hạn độ dài phía client; giới hạn (nếu có) do server xử lý ⚠️.

### 3.2 Màn hình Forgot Password ✅

| # | Thành phần | Loại | Thuộc tính / Mô tả |
|---|---|---|---|
| 1 | Tiêu đề | Heading | "Forgot Password" |
| 2 | Email Address | Textbox `type="email"` | `id=email`, `name=email`, giá trị rỗng mặc định |
| 3 | Confirm | Button `type="submit"` | Gửi POST tới `/admin/authentication/forgot_password` |
| 4 | CSRF token | Input ẩn | `name=csrf_token_name` |

- ✅ Màn hình **không có** liên kết "Back to Login"; người dùng phải dùng nút Back của trình duyệt hoặc nhập lại URL.

---

## 4. Yêu cầu chức năng

### FR-01: Hiển thị màn hình Login ✅
- Truy cập URL Login khi chưa đăng nhập → hiển thị đầy đủ các thành phần ở mục 3.1.
- Con trỏ tự động đặt tại ô Email Address.
- Các ô nhập rỗng, checkbox Remember me không được chọn.

### FR-02: Đăng nhập thành công ⚠️
- **Điều kiện:** Email tồn tại, đúng mật khẩu, tài khoản đang hoạt động.
- **Kết quả mong đợi:** Hệ thống tạo phiên đăng nhập và chuyển hướng tới Dashboard (`/admin/`). Thông tin người dùng (avatar/tên) hiển thị ở header.

### FR-03: Kiểm tra bắt buộc nhập (Required) ⚠️
| Trường hợp | Kết quả mong đợi |
|---|---|
| Bỏ trống Email | Không đăng nhập; hiển thị lỗi bắt buộc nhập cho trường Email (ví dụ: "The Email Address field is required.") |
| Bỏ trống Password | Không đăng nhập; hiển thị lỗi bắt buộc nhập cho trường Password |
| Bỏ trống cả hai | Hiển thị lỗi cho cả hai trường |

> Vì không có `required` ở HTML, việc kiểm tra bắt buộc do **server** thực hiện sau khi submit.

### FR-04: Kiểm tra định dạng Email ✅/⚠️
- ✅ Input là `type="email"` và form không có `novalidate` → khi nhập sai định dạng (vd. `admin@`, `abc`), trình duyệt chặn submit và hiển thị tooltip cảnh báo mặc định (nội dung phụ thuộc trình duyệt/ngôn ngữ).
- ⚠️ Nếu vượt qua kiểm tra trình duyệt, server cũng phải từ chối email sai định dạng.
- ⚠️ Khoảng trắng đầu/cuối email nên được loại bỏ (trim) trước khi xác thực.

### FR-05: Đăng nhập thất bại do sai thông tin ⚠️
| Trường hợp | Kết quả mong đợi |
|---|---|
| Email không tồn tại | Ở lại trang Login; hiển thị thông báo lỗi chung (vd. "Invalid email or password") |
| Email đúng, sai Password | Như trên – thông báo **không** tiết lộ trường nào sai |
| Password sai hoa/thường | Từ chối – mật khẩu phân biệt hoa/thường |
| Tài khoản bị vô hiệu hóa | Từ chối, thông báo tài khoản không hoạt động |

- Sau khi thất bại, ô Password được xóa trắng; ô Email có thể giữ lại giá trị đã nhập ⚠️.

### FR-06: Remember me ⚠️
- **Không chọn:** phiên đăng nhập kết thúc khi đóng trình duyệt / hết thời gian session.
- **Có chọn:** hệ thống lưu cookie ghi nhớ; mở lại trình duyệt vẫn ở trạng thái đăng nhập trong thời hạn cookie.
- ✅ Ghi chú kỹ thuật: checkbox gửi giá trị `remember=estimate` khi được chọn (giá trị lạ, chỉ cần tồn tại tham số).

### FR-07: Forgot Password ⚠️
- Click "Forgot Password?" → chuyển tới màn hình Forgot Password (✅).
- Nhập email hợp lệ đã đăng ký và nhấn Confirm → hiển thị thông báo đã gửi email đặt lại mật khẩu.
- Email chưa đăng ký → hiển thị thông báo lỗi.
- Bỏ trống / sai định dạng email → xử lý tương tự FR-03, FR-04.

### FR-08: Truy cập trái phép & phiên làm việc ⚠️
- Chưa đăng nhập mà truy cập trang trong `/admin/...` → chuyển hướng về trang Login.
- Đã đăng nhập mà truy cập lại URL Login → chuyển hướng về Dashboard.
- Sau khi Logout, nhấn Back của trình duyệt không truy cập lại được trang quản trị.

### FR-09: Bảo mật form ✅/⚠️
- ✅ Mỗi lần tải trang sinh CSRF token ẩn trong form.
- ⚠️ Gửi form với CSRF token sai/hết hạn → request bị từ chối.
- ⚠️ Input chứa ký tự đặc biệt / chuỗi SQL Injection / XSS (vd. `' OR 1=1 --`, `<script>`) không gây lỗi hệ thống, không đăng nhập được và không thực thi script.

---

## 5. Yêu cầu phi chức năng

| Mã | Loại | Yêu cầu |
|---|---|---|
| NFR-01 | Bảo mật | Mật khẩu hiển thị dạng ẩn; không xuất hiện trên URL (form dùng POST ✅) |
| NFR-02 | Bảo mật | Trang chạy trên HTTPS ✅ |
| NFR-03 | Bảo mật | Thông báo lỗi đăng nhập không tiết lộ email có tồn tại hay không ⚠️ |
| NFR-04 | Bảo mật | Có cơ chế hạn chế đăng nhập sai nhiều lần (brute-force) ⚠️ – cần xác nhận cấu hình |
| NFR-05 | Hiệu năng | Thời gian phản hồi đăng nhập ≤ 3 giây trong điều kiện mạng bình thường ⚠️ |
| NFR-06 | Khả dụng | Hỗ trợ thao tác hoàn toàn bằng bàn phím (Tab, Space, Enter) ✅ |
| NFR-07 | Tương thích | Hiển thị đúng trên Chrome, Firefox, Edge bản mới nhất; giao diện co giãn trên màn hình mobile ⚠️ |

---

## 6. Luồng nghiệp vụ

### 6.1 Luồng chính – Đăng nhập thành công
1. Người dùng truy cập `/admin/authentication`.
2. Nhập Email Address.
3. Nhập Password.
4. (Tùy chọn) Chọn Remember me.
5. Click **Login** hoặc nhấn Enter.
6. Hệ thống kiểm tra định dạng (trình duyệt) → gửi POST lên server.
7. Server xác thực thành công → tạo session → chuyển tới Dashboard.

### 6.2 Luồng thay thế
- **6a.** Email sai định dạng → trình duyệt chặn ở bước 6, hiển thị tooltip.
- **7a.** Trường bắt buộc bị trống → quay lại trang Login kèm thông báo lỗi validate.
- **7b.** Sai Email/Password → quay lại trang Login kèm thông báo lỗi xác thực.
- **2a.** Người dùng click "Forgot Password?" → chuyển sang luồng FR-07.

---

## 7. Câu hỏi mở / Cần xác nhận
1. Nội dung chính xác của các thông báo lỗi (required, sai thông tin, tài khoản bị khóa)?
2. Có giới hạn số lần đăng nhập sai và thời gian khóa không?
3. Thời hạn của session và cookie Remember me là bao lâu?
4. Độ dài tối đa của Email và Password phía server?
5. Màn hình Forgot Password có cần bổ sung liên kết quay lại Login không?
