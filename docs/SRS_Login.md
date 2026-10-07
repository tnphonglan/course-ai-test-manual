# SRS – Module Đăng nhập (Login)

| Mục | Nội dung |
|---|---|
| Hệ thống | Perfex CRM – Anh Tester Demo |
| Module | Login / Authentication (khu vực Admin) |
| URL | https://crm.anhtester.com/admin/authentication |
| Phiên bản tài liệu | 1.0 (Draft) |
| Ngày lập | 2026-10-07 |
| Nguồn thông tin | Khảo sát giao diện trang Login và Forgot Password (chỉ đọc, không đăng nhập) |

> **Quy ước đánh dấu độ tin cậy**
> - ✅ **Quan sát được**: đã xác nhận trực tiếp trên giao diện / DOM.
> - ⚠️ **Giả định**: suy luận theo hành vi chuẩn của Perfex CRM / thực tiễn chung, **cần xác nhận** bằng cách chạy thực tế trước khi dùng làm căn cứ chấm Pass/Fail.

---

## 1. Giới thiệu

### 1.1 Mục đích
Tài liệu mô tả yêu cầu chức năng và phi chức năng của module Đăng nhập, làm cơ sở cho việc phân tích, thiết kế test case và kiểm thử thủ công.

### 1.2 Phạm vi
- Màn hình **Login** (`/admin/authentication`).
- Chức năng **Remember me**.
- Chức năng **Forgot Password** (`/admin/authentication/forgot_password`).
- Điều hướng sau đăng nhập thành công (⚠️ tới Dashboard của Admin).

Ngoài phạm vi: Đăng ký tài khoản, đăng nhập khu vực Client, phân quyền chi tiết, Two-Factor Authentication (nếu có cấu hình).

### 1.3 Đối tượng sử dụng
| Vai trò | Mô tả |
|---|---|
| Admin / Staff | Người dùng nội bộ có tài khoản trên hệ thống, đăng nhập để vào trang quản trị |

### 1.4 Thuật ngữ
| Thuật ngữ | Giải thích |
|---|---|
| CSRF token | Mã chống giả mạo request, được gửi kèm form dưới dạng input ẩn |
| Remember me | Tùy chọn duy trì phiên đăng nhập lâu hơn |

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh
Login là cổng vào duy nhất của khu vực quản trị. Người dùng nhập email và mật khẩu; hệ thống xác thực và cấp phiên làm việc.

### 2.2 Giả định & ràng buộc
- ✅ Hệ thống xây dựng trên Perfex CRM (tiêu đề trang "Perfex CRM | Anh Tester Demo - Login").
- ✅ Định danh đăng nhập là **địa chỉ email**.
- ✅ Form gửi bằng phương thức **POST** tới chính URL `/admin/authentication`, có CSRF token.
- ✅ Không có nút đăng ký tài khoản trên trang.
- ⚠️ Đây là môi trường demo dùng chung, dữ liệu có thể bị thay đổi bởi người khác.

### 2.3 Dữ liệu kiểm thử
| Loại | Email | Mật khẩu | Ghi chú |
|---|---|---|---|
| Hợp lệ (do người dùng cung cấp) | `admin@example.com` | `123456` | Tài khoản Admin demo – chưa được chạy xác minh trong lần khảo sát này |

---

## 3. Mô tả giao diện (UI)

### 3.1 Màn hình Login ✅

| # | Thành phần | Loại | Thuộc tính quan sát được |
|---|---|---|---|
| 1 | Logo | Hình ảnh + liên kết | Alt "Perfex CRM \| Anh Tester Demo"; liên kết về `https://crm.anhtester.com/` |
| 2 | Tiêu đề | Heading | "Login" |
| 3 | Email Address | Input `type="email"` | `name=email`, `id=email`, **autofocus**, không có `required` HTML5, không `maxlength`, không placeholder |
| 4 | Password | Input `type="password"` | `name=password`, `id=password`, không `required` HTML5, không `maxlength`, không placeholder |
| 5 | Remember me | Checkbox | `name=remember`, `id=remember`, **mặc định không chọn** |
| 6 | Login | Button `type="submit"` | Gửi form |
| 7 | Forgot Password? | Liên kết | → `/admin/authentication/forgot_password` |
| 8 | CSRF token | Input ẩn | `name=csrf_token_name` |

Nhận xét:
- Input Email dùng `type="email"` nên trình duyệt có thể tự kiểm tra định dạng email trước khi gửi (hành vi phụ thuộc trình duyệt).
- Không có thuộc tính `required` ở client; việc bắt buộc nhập (nếu có) do server kiểm tra ⚠️.
- Trường Password có thuộc tính che ký tự (`type="password"`); ✅ không có nút hiện/ẩn mật khẩu.

### 3.2 Màn hình Forgot Password ✅

| # | Thành phần | Loại | Thuộc tính |
|---|---|---|---|
| 1 | Logo | Hình ảnh + liên kết | Như màn hình Login |
| 2 | Tiêu đề | Heading | "Forgot Password" |
| 3 | Email Address | Input `type="email"` | Trường nhập email tài khoản |
| 4 | Confirm | Button `type="submit"` | Gửi yêu cầu đặt lại mật khẩu |
| 5 | CSRF token | Input ẩn | Có |

---

## 4. Yêu cầu chức năng

### 4.1 Đăng nhập (FR-LOGIN)

| ID | Yêu cầu | Độ tin cậy |
|---|---|---|
| FR-LOGIN-01 | Hệ thống hiển thị form đăng nhập gồm Email, Password, Remember me và nút Login khi truy cập `/admin/authentication`. | ✅ |
| FR-LOGIN-02 | Con trỏ tự động focus vào trường Email khi tải trang. | ✅ |
| FR-LOGIN-03 | Người dùng nhập Email + Password hợp lệ và nhấn Login → hệ thống xác thực thành công và chuyển tới trang quản trị (Dashboard). | ⚠️ |
| FR-LOGIN-04 | Khi Email hoặc Password không đúng, hệ thống **không** đăng nhập, giữ người dùng ở trang Login và hiển thị thông báo lỗi (⚠️ "Invalid email or password"). | ⚠️ |
| FR-LOGIN-05 | Khi để trống Email và/hoặc Password rồi nhấn Login, hệ thống không đăng nhập và hiển thị thông báo yêu cầu nhập (⚠️ lỗi từ server hoặc từ trình duyệt). | ⚠️ |
| FR-LOGIN-06 | Khi Email sai định dạng (thiếu `@`, thiếu domain…), trình duyệt/hệ thống chặn gửi hoặc báo lỗi định dạng. | ⚠️ |
| FR-LOGIN-07 | Email không phân biệt hoa/thường (⚠️ theo hành vi chuẩn của Perfex); Password **phân biệt** hoa/thường. | ⚠️ |
| FR-LOGIN-08 | Tài khoản bị vô hiệu hóa (inactive) không thể đăng nhập và nhận thông báo lỗi. | ⚠️ |
| FR-LOGIN-09 | Nhấn phím **Enter** trong trường Email/Password tương đương nhấn nút Login. | ⚠️ |
| FR-LOGIN-10 | Mật khẩu hiển thị dạng ký tự che (masked), không hiển thị rõ. | ✅ |
| FR-LOGIN-11 | Form gửi kèm CSRF token; request thiếu/sai token bị từ chối. | ✅ (có token) / ⚠️ (hành vi từ chối) |
| FR-LOGIN-12 | Sau nhiều lần đăng nhập sai liên tiếp, hệ thống có thể giới hạn/khóa tạm thời (⚠️ Perfex có cơ chế chặn theo IP, mặc định ~3 lần sai, khóa ~30 phút – cần xác nhận cấu hình demo). | ⚠️ |
| FR-LOGIN-13 | Người dùng đã đăng nhập mà truy cập lại `/admin/authentication` thì được chuyển về Dashboard (không hiện lại form). | ⚠️ |
| FR-LOGIN-14 | Truy cập trực tiếp URL trang quản trị khi chưa đăng nhập → chuyển hướng về trang Login. | ⚠️ |

### 4.2 Ghi nhớ đăng nhập (FR-REMEMBER)

| ID | Yêu cầu | Độ tin cậy |
|---|---|---|
| FR-REMEMBER-01 | Checkbox "Remember me" mặc định **không** được chọn. | ✅ |
| FR-REMEMBER-02 | Khi chọn Remember me và đăng nhập thành công, phiên được duy trì sau khi đóng và mở lại trình duyệt (cookie lâu dài). | ⚠️ |
| FR-REMEMBER-03 | Khi không chọn Remember me, phiên kết thúc khi đóng trình duyệt (session cookie). | ⚠️ |

### 4.3 Quên mật khẩu (FR-FORGOT)

| ID | Yêu cầu | Độ tin cậy |
|---|---|---|
| FR-FORGOT-01 | Liên kết "Forgot Password?" trên trang Login chuyển tới `/admin/authentication/forgot_password`. | ✅ |
| FR-FORGOT-02 | Trang Forgot Password gồm trường Email và nút Confirm. | ✅ |
| FR-FORGOT-03 | Nhập email hợp lệ đã đăng ký và nhấn Confirm → hệ thống gửi email hướng dẫn đặt lại mật khẩu và hiển thị thông báo thành công. | ⚠️ |
| FR-FORGOT-04 | Nhập email chưa đăng ký → hệ thống hiển thị thông báo lỗi (⚠️ Perfex mặc định báo "Email not found"; lưu ý rủi ro lộ thông tin tồn tại tài khoản). | ⚠️ |
| FR-FORGOT-05 | Để trống email hoặc sai định dạng → không gửi yêu cầu, hiển thị lỗi. | ⚠️ |
| FR-FORGOT-06 | Liên kết đặt lại mật khẩu trong email chỉ dùng được một lần và có thời hạn. | ⚠️ |

### 4.4 Điều hướng

| ID | Yêu cầu | Độ tin cậy |
|---|---|---|
| FR-NAV-01 | Nhấn logo chuyển tới trang chủ `https://crm.anhtester.com/`. | ✅ |

---

## 5. Quy tắc nghiệp vụ & kiểm tra dữ liệu

| ID | Trường | Quy tắc | Độ tin cậy |
|---|---|---|---|
| BR-01 | Email | Bắt buộc; đúng định dạng email; là định danh duy nhất của tài khoản | ⚠️ (định dạng ✅ ở mức `type=email`) |
| BR-02 | Password | Bắt buộc; so khớp với mật khẩu đã lưu (băm) | ⚠️ |
| BR-03 | Email/Password | Không giới hạn độ dài ở phía client (không `maxlength`) – cần kiểm tra giới hạn phía server | ✅ / ⚠️ |
| BR-04 | Thông báo lỗi | Không tiết lộ cụ thể sai email hay sai mật khẩu | ⚠️ |
| BR-05 | Khoảng trắng đầu/cuối | Cần xác định có được tự động cắt (trim) hay không | ⚠️ |

---

## 6. Yêu cầu phi chức năng

| ID | Nhóm | Yêu cầu | Độ tin cậy |
|---|---|---|---|
| NFR-01 | Bảo mật | Truyền dữ liệu qua HTTPS. | ✅ (URL dùng https) |
| NFR-02 | Bảo mật | Mật khẩu không xuất hiện trong URL/query string; form dùng POST. | ✅ |
| NFR-03 | Bảo mật | Có CSRF token trên form Login và Forgot Password. | ✅ |
| NFR-04 | Bảo mật | Chống brute-force (giới hạn số lần thử). | ⚠️ |
| NFR-05 | Bảo mật | Chống SQL Injection / XSS ở các trường nhập. | ⚠️ |
| NFR-06 | Bảo mật | Cookie phiên có cờ `HttpOnly`, `Secure`. | ⚠️ |
| NFR-07 | Hiệu năng | Trang Login tải < 3 giây trên mạng thông thường; xử lý đăng nhập < 3 giây. | ⚠️ |
| NFR-08 | Tương thích | Hoạt động trên Chrome, Firefox, Edge, Safari phiên bản mới. | ⚠️ |
| NFR-09 | Responsive | Hiển thị đúng trên desktop, tablet, mobile. | ⚠️ |
| NFR-10 | Khả dụng | Nhãn (label) gắn với input; điều hướng được bằng bàn phím (Tab: Email → Password → Remember me → Login → Forgot Password). | ✅ (có label) / ⚠️ (thứ tự Tab) |
| NFR-11 | Ngôn ngữ | Giao diện tiếng Anh (`lang="en"`). | ✅ |

---

## 7. Luồng nghiệp vụ

### 7.1 Đăng nhập thành công
1. Người dùng truy cập `/admin/authentication`.
2. Nhập Email và Password hợp lệ; (tùy chọn) chọn Remember me.
3. Nhấn Login (hoặc Enter).
4. Hệ thống kiểm tra CSRF token, xác thực thông tin.
5. Thành công → tạo phiên, chuyển tới Dashboard.

### 7.2 Đăng nhập thất bại
Tại bước 4, nếu thông tin sai/thiếu → quay lại trang Login, hiển thị thông báo lỗi, giữ nguyên trang.

### 7.3 Quên mật khẩu
1. Nhấn "Forgot Password?".
2. Nhập Email, nhấn Confirm.
3. Hệ thống gửi email đặt lại mật khẩu (hoặc báo lỗi nếu email không hợp lệ/không tồn tại).
4. Người dùng mở liên kết trong email, đặt mật khẩu mới, đăng nhập lại.

---

## 8. Ma trận truy vết yêu cầu → gợi ý kiểm thử

| Yêu cầu | Nhóm test gợi ý |
|---|---|
| FR-LOGIN-01, 02, 10 | UI / hiển thị |
| FR-LOGIN-03, 09 | Đăng nhập hợp lệ (Positive) |
| FR-LOGIN-04..08 | Dữ liệu sai / trống / sai định dạng / hoa-thường (Negative) |
| FR-LOGIN-11, 12, NFR-04..06 | Bảo mật |
| FR-LOGIN-13, 14 | Phiên & điều hướng |
| FR-REMEMBER-01..03 | Remember me |
| FR-FORGOT-01..06 | Quên mật khẩu |
| NFR-07..10 | Hiệu năng, tương thích, responsive, khả dụng |

---

## 9. Điểm cần xác nhận (Open Issues)

1. Nội dung chính xác các thông báo lỗi (sai thông tin, để trống, tài khoản bị khóa, email không tồn tại).
2. Lỗi để trống là do trình duyệt hay server (do không có `required` HTML5).
3. Cơ chế khóa sau nhiều lần đăng nhập sai (ngưỡng, thời gian) trên môi trường demo.
4. Trang đích sau đăng nhập và hành vi khi đã đăng nhập mà vào lại trang Login.
5. Thời hạn sống của phiên và của cookie Remember me.
6. Có bật Two-Factor Authentication / reCAPTCHA hay không.
7. Email có được trim khoảng trắng và có phân biệt hoa/thường hay không.

---

## 10. Phụ lục – Ghi chú về quá trình khảo sát

- Đã thu thập: cấu trúc DOM của trang Login và Forgot Password (trường, thuộc tính, liên kết, phương thức gửi form).
- **Chưa thực hiện**: đăng nhập bằng tài khoản `admin@example.com`, gửi form trống/sai, kích hoạt Forgot Password. Vì vậy các mục đánh dấu ⚠️ chưa được xác minh bằng hành vi thực tế; nên chạy kiểm chứng và cập nhật lại tài liệu (đổi ⚠️ → ✅ kèm thông báo/hành vi thực tế).
