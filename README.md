# Hướng dẫn làm authen bằng GG, Facebook

Thông qua hướng dẫn Spring Boot này, bạn sẽ tìm hiểu cách triển khai chức năng đăng nhập một lần bằng tài khoản Google và facebook cho ứng dụng web Spring Boot hiện có, sử dụng thư viện Spring OAuth2 Client - cho phép người dùng cuối đăng nhập bằng tài khoản Google của riêng họ thay vì thông tin đăng nhập do ứng dụng quản lý .
Giả sử rằng bạn có một dự án Spring Boot hiện có với chức năng xác thực đã được triển khai bằng Spring Security và thông tin người dùng được lưu trữ trong cơ sở dữ liệu MySQL (Nếu không, hãy tải xuống dự án mẫu trong hướng dẫn này ) .
Sau đó, chúng tôi sẽ cập nhật trang đăng nhập cho phép người dùng đăng nhập bằng tài khoản Google của riêng họ như thế này:

c![image](https://github.com/thangdtph27626/-login-google-facebook/assets/109157942/0b7f74a7-6722-4bfb-bd55-cc59ebf9010b)

## OAuth2 là gì?
OAuth2 viết tắt của Open với Authentication / Authorization.
OAuth2 là một authorization framework cho phép các ứng dụng có quyền truy cập hạn chế vào tài khoản người dùng trên dịch vụ HTTP, chẳng hạn như Facebook, Github,... Nó hoạt động bằng cách ủy quyền xác thực người dùng cho dịch vụ lưu trữ tài khoản người dùng và ủy quyền cho các ứng dụng của bên thứ ba truy cập vào tài khoản của người dùng. OAuth2 cung cấp các authorization flows cho ứng dụng web và desktop hay thiết bị di động.
Authentication: xác thực người dùng.
Authorization: người dùng ủy quyền cho ứng dụng truy cập tài nguyên của họ.

## Luồng hoạt động


![image](https://github.com/thangdtph27626/-login-google-facebook/assets/109157942/d978d43e-2ef9-4fe8-a4be-183467bd502f)

Mình giải thích một chút về luồng hoạt động nha:

- Ứng dụng (web hay app mobile) yêu cầu ủy quyền để truy cập Resource Server (Gmail, Facebook, Github...) từ User.
- Nếu User cho phép yêu cầu, ứng dụng sẽ nhận được ủy quyền thừ phía User
- Ứng dụng yêu cầu mã thông báo truy cập từ Authorization Server (API) bằng cách gửi thông tin định danh của chính nó và của User.
- Nếu danh tính ứng dụng được xác thực và cấp ủy quyền hợp lệ, Authorization Server (API) sẽ cấp 1 access_token thông báo truy cập cho ứng dụng. Đến đây việc ủy quyền đã hoàn tất.
- Ứng dụng yêu cầu tài nguyên từ Resource Server (API) và xuất trình access_token để xác thực.
- Nếu access_token hợp lệ, Resource Server (API) sẽ cung cấp tài nguyên cho ứng dụng.
- Luồng hoạt động thực tế có thể khác tùy thuộc vào loại authorization, trên đây chỉ là ý tưởng chung về cách thức thực hiện.

##  OAuth2 Roles
OAuth2 xác định bốn roles như sau:

Resource Owner (Chủ sở hữu tài nguyên)
Client (Khách hàng)
Resource Server (Máy chủ tài nguyên)
Authorization Server (Máy chủ ủy quyền)

## Authorization Code

Loại cấp mã ủy quyền được sử dụng phổ biến nhất vì nó được tối ưu hóa cho các ứng dụng phía máy chủ , nơi mã nguồn không bị lộ công khai và có thể duy trì tính bảo mật của Bí mật khách hàng . Đây là luồng dựa trên chuyển hướng, có nghĩa là ứng dụng phải có khả năng tương tác với tác nhân người dùng (tức là trình duyệt web của người dùng) và nhận mã ủy quyền API được định tuyến thông qua tác nhân người dùng.

Bây giờ chúng ta sẽ mô tả luồng mã ủy quyền:

![image](https://github.com/thangdtph27626/-login-google-facebook/assets/109157942/56f3ced6-4118-41c9-8add-90a14fa04e25)

### Bước 1 - Liên kết mã ủy quyền

Đầu tiên, người dùng được cấp một liên kết mã ủy quyền trông giống như sau:

```
https://cloud.digitalocean.com/v1/oauth/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read

```

Dưới đây là giải thích về các thành phần của liên kết ví dụ này:

- **https://cloud.digitalocean.com/v1/oauth/authorize**: điểm cuối ủy quyền API
- client_id=client_id của ứng dụng (cách API xác định ứng dụng)
- redirect_uri=CALLBACK_URL: nơi dịch vụ chuyển hướng tác nhân người dùng sau khi mã ủy quyền được cấp
- response_type=code: chỉ định rằng ứng dụng của bạn đang yêu cầu cấp mã ủy quyền
- scope =read: chỉ định mức độ truy cập mà ứng dụng đang yêu cầu

### Bước 2 - Người dùng ủy quyền cho ứng dụng
Khi người dùng nhấp vào liên kết, trước tiên họ phải đăng nhập vào dịch vụ để xác thực danh tính của mình (trừ khi họ đã đăng nhập). Sau đó, họ sẽ được dịch vụ nhắc nhở ủy quyền hoặc từ chối ứng dụng truy cập vào tài khoản của họ. Dưới đây là một ví dụ về lời nhắc ứng dụng ủy quyền:

### Bước 3 - Ứng dụng nhận mã ủy quyền
Nếu người dùng nhấp vào Ủy quyền ứng dụng , dịch vụ sẽ chuyển hướng tác nhân người dùng đến URI chuyển hướng ứng dụng, được chỉ định trong quá trình đăng ký ứng dụng khách, cùng với mã ủy quyền . Chuyển hướng sẽ trông giống như thế này 


### Bước 4 - Mã thông báo truy cập yêu cầu ứng dụng
Ứng dụng yêu cầu mã thông báo truy cập từ API bằng cách chuyển mã ủy quyền cùng với chi tiết xác thực, bao gồm cả bí mật ứng dụng khách , tới điểm cuối mã thông báo API.

### Bước 5 - Ứng dụng nhận mã thông báo truy cập

Nếu ủy quyền hợp lệ, API sẽ gửi phản hồi chứa mã thông báo truy cập (và tùy chọn, mã thông báo làm mới) tới ứng dụng. Toàn bộ phản hồi sẽ trông giống như thế này:

```
{"access_token":"ACCESS_TOKEN","token_type":"bearer","expires_in":2592000,"refresh_token":"REFRESH_TOKEN","scope":"read","uid":100101,"info":{"name":"Mark E. Mark","email":"mark@thefunkybunch.com"}}
```

Bây giờ ứng dụng đã được cấp phép. Nó có thể sử dụng mã thông báo để truy cập vào tài khoản của người dùng thông qua API dịch vụ, giới hạn phạm vi truy cập cho đến khi mã thông báo hết hạn hoặc bị thu hồi. Nếu mã thông báo làm mới được phát hành, mã thông báo này có thể được sử dụng để yêu cầu mã thông báo truy cập mới nếu mã thông báo ban đầu đã hết hạn.


##  Tạo thông tin xác thực  OAuth Google và  facebook
Đầu tiên, hãy làm theo video này để tạo ID ứng dụng khách OAuth của Google nhằm lấy khóa truy cập của API đăng nhập một lần của Google (ID ứng dụng khách và Bí mật ứng dụng khách). Lưu ý rằng bạn cần thêm URI chuyển hướng được ủy quyền như thế này:

Bạn có thể lấy clientId, clientSecret và redirectUri của [google](https://developers.google.com/identity/protocols/oauth2/web-server#httprest) và [facebook](https://developers.facebook.com/docs/facebook-login/guides/advanced/manual-flow/)

Bạn có thể tìm hiểu thêm về cách login google và facebook với spring boot [tại đây](https://www.callicoder.com/spring-boot-security-oauth2-social-login-part-1/)


