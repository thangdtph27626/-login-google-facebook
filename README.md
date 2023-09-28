# -login-google-facebook

Thông qua hướng dẫn Spring Boot này, bạn sẽ tìm hiểu cách triển khai chức năng đăng nhập một lần bằng tài khoản Google và facebook cho ứng dụng web Spring Boot hiện có, sử dụng thư viện Spring OAuth2 Client - cho phép người dùng cuối đăng nhập bằng tài khoản Google của riêng họ thay vì thông tin đăng nhập do ứng dụng quản lý .
Giả sử rằng bạn có một dự án Spring Boot hiện có với chức năng xác thực đã được triển khai bằng Spring Security và thông tin người dùng được lưu trữ trong cơ sở dữ liệu MySQL (Nếu không, hãy tải xuống dự án mẫu trong hướng dẫn này ) .
Sau đó, chúng tôi sẽ cập nhật trang đăng nhập cho phép người dùng đăng nhập bằng tài khoản Google của riêng họ như thế này:

c![image](https://github.com/thangdtph27626/-login-google-facebook/assets/109157942/0b7f74a7-6722-4bfb-bd55-cc59ebf9010b)

## 1. Tạo thông tin xác thực Google OAuth

