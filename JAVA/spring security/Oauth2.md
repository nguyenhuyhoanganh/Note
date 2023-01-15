# OAuth2
* là khung uỷ quyền (giao thức uỷe quyền) với mục đích cho phép các trang web, hoặc ứng dụng bên thứ 3 truy cập tài nguyên
* tách biệt trách nhiệm quản lý thông tin xác thực trong 1 thành phần của hệ thống (máy chủ uỷ quyền)

# Các thành phần trong Oath2
  * Resource server: máy chủ tài nguyên, là 1 ứng dụng lưu trữ tài nguyên do người dùng sở hữu 
  * User: người dùng, là cá nhân sở hữu tài nguyên được cung cấp bởi resource server. Thường sử dụng username và passwword để nhận dạng bản thân
  * Client: máy khách, là 1 ứng dụng được phép thay mặt user truy cập các tài nguyên mà họ sở hữu. Client sử dụng client ID và client secret để nhận dạng chính nó. Những thông tin này khác với thông tin đăng nhập của user. Client cần nó để nhận dạng chính nó khi tạo request.
  * Authorization server: máy chủ uỷ quyền, là ứng dụng cho phép client truy cập vào tài nguyên mà user sử hữu được cung cấp bởi resource server. Khi authorization server quyết định rạng client được uỷ quyền truy cập vào tài nguyên thay mặt user, nó phát hành 1 token. Client sử dụng token để chứng minh với resource server rằng nó được uỷ quyền bởi authorization server. Resource server cho phép client được truy cập vào tài nguyên mà nó yêu cầu nếu nó có token hợp lệ

# Các cách triển khai OAuth2
### Authorization code grant type: 
#### Các bước:
 1. Client chuyển hướng user tới endpoint của authorization server, client gọi authorization endpoint với request query bao gồm:
   - response_type : "code". Nói với authorzation server rằng client mong đợi 1 code, code này sử dụng để lấy token
   - client_id : giá trị của client id, là giá trị xác định chính client thực hiện request
   - redirect_uri: uri để authorization server chuyển hướng user sau khi xác thực thành công (có thể có hoặc không tuỳ trường hợp authorization server đã đc xác định uri chuyển hướng mặc định)
   - scope : quyền được cấp (read, write, delete, ...)
   - state : giá trị csrf token, sử dụng cho việc bảo vệ csrf
 2. User tương tác trực tiếp với authorization server, thực hiện xác thực (không giử thông tin đăng nhập tới client), khi xác thực thành công, Authorization server gọi lại redirect_uri về Client, cung cấp giá trị authorization code và giá trị state.
 3. Client kiểm tra state đúng với giá trị state đã giử ban đầu, đảm bảo không phải 1 bên thứ 3 gọi tới redirect_uri để đánh cắp client_secret. Khi xác nhận được đúng redirect_uri được gọi từ phía authorization server, client sử dụng authorization code nhận được thực hiện gọi 1 lần nữa tới endpoint của authorization server để nhận token với request query bao gồm:
   - code : giá trị authorization code vừa nhận được
   - client_id và client_secret : thông tin xác thực của client
   - redirect_uri
   - grant_type : "authorization_code".  xác định loại grant được sử dụng (authorization server có thể hỗ trợ nhiều loại grant)
 4. Authorization server nhận request từ client
   - client_id và client_secret chứng minh client là cùng 1 client đã gọi tới endpoint của authorization
   - code chứa authorization code chứng minh user đã xác thưc
  => authorization server gửi access_token cho client
 5. Client nhận access_token sử dụng để đặt giá trị với key là Authorization trong header của request, giử tới resource server để lấy tài nguyên cho user

#### Tại sao không giử luôn giá trị access_token ngay lần request đầu tiên từ client tới authorization server?
  * thực tế là có 1 grant type : implicit grant type sẽ trực tiếp trả về access_token cho client, tuy nhiên grant type này không được khuyến nghị sử dụng, hầu hết các authorization servers nổi tiếng cũng không cho phép. Bởi vì authorization server khi gọi tới redirect_uri với access token mà không đảm bảo rằng nó có phải là client đúng để được phép nhận. Client cần chứng minh lại chính nó sau khi nhận authorization_code bằng thông tin xác thực của nó (client_id, client_secret) để nhận được access_token 

### Password grant type:
* Tương tự như authorization code grant type, tuy nhiên thay vì tương tác trực tiếp giữa user và authorization server để xác thực, user giao thông tin xác thực (gồm password) cho client
  * client sẽ xác thực thay user với authorization server
  * khi xác thực thành công, authorization trực tiếp giử access_token tới client
  * sử dụng luồng xác thực này chỉ khi client và authorization server được xây dựng và duy trì bởi cùng 1 tổ chức
* Grant type này kém bảo mật hơn authorization code. Chủ yếu bởi vì user chia sẻ thông tin đăng nhập của mình tới client app. Nó được dùng trong các tình huống thực tế do nó đơn giản hơn authorization code
* Chỉ sử dụng quy trình này nếu client à authorization server được duy trì bởi cùng một tổ chức. User chỉ mong đợi 1 biểu mẫu đăng nhập và để client lo việc giử thong tin đăng nhập đó đến authorization server. User không cần biết việc xác thực xảy ra như thế nào bên trong ứng dụng.
#### Các bước:
 1. Client thu thập thông tin đằng nhập và gọi authorization server để lấy access_token. request client giử đi bao gồm:
   - grant_type: "password"
   - client_id và client_secret: thông tin xác thực được client sử dụng để xác thực chính nó
   - scope: là các quyền được cấp
   - username và password: là thông tin đăng nhập của user. được giử đến dưới dạng plain text là các giá trị trong request header
 2. Client nhận lại access token trong response, sử dụng token đó để gọi đến các endpoint trên resource server. token được thêm vào header của request với key là Authorization

### Client credentials grant type:
* Được sử dụng khi không có user tham gia vào quy trình xác thực.
* Giả định rằng hệ thống xác thực bằng OAuth2, cần cho phép 1 server bên ngoài xác thực và gọi tới 1 tài nguyên cụ thể mà server của mình cung cấp.
#### Các bước:
 1. Client giử request tới authorization server bao gồm:
   - grant_type: "client_crednetials"
   - client_id và client_secret: là thông tin đăng nhập của client
   - scope: đại diện cho các quyền được cấp
 2. Client nhận được một token. Sử dụng token để gọi tới endpoint của resource server. token được thêm vào header của request với key là Authorization

### Sử dụng refresh_token để nhận access_token mới
* access_token được sử dụng trong OAuth 2 không quan trọng 1 cách triển khai cụ thể nào cho nó, nó đều có hạn sử dụng. Tuổi thọ của access_token có thể là vô hạn, nhưng nên làm cho nó càng ngắn càng tốt. Khi access_token có tuổi thọ vô hạn làm cho nó tương tự với thông tin đăng nhập của user, token này có thể sử dụng để nhận được tài nguyên từ resource server mọi lúc. Trong khi access_token chỉ được đính kèm dưới dạng 1 http header trên mỗi request, việc token này bị đánh cắp tương tự việc thông tin đăng nhập củ user bị lộ.
* Vì vậy tuổi thọ của access_token nên làm cho cằng ngắn càng tốt. Khi access_token hết hạn, user cần đăng nhập lại và lấy 1 access_token mới. Ngay cả khi sử dụng password grant type, client cần phải lưu thông tin đăng nhập của user.
* refresh_token đại diện cho giải pháp thay thế việc sử dụng thông tin đăng nhập để lấy access_token mới.
* refresh_token được authorization server trả về cùng với access_token khi sử dụng các luồng xác thực như authorization code hoặc password grant type. Client sử dụng refesh_token để đưa ra request tới Authorization khi access_token hết hạn, bao gồm:
  - grant_type: "refresh_token"
  - refresh_token: giá trị của token
  - client_id và client_secret: thông tin đăng nhập của client
  - scope: xác định các quyền được cấp. Nếu cần nhiều quyền được cấp hơn, cần xác thực lại.
* Nhận được request, authorization server sẽ phát hành refresh_token và access_token mới

# Thực hiện 1 application đơn giản (SSO - single sign-on)
* SSO: user xác thực thông qua authorization server và sau đó app giữ user đăng nhập, sử dụng refresh_token. App đại diện cho client trong khung OAuth2.
* Ví dụ sử dụng github như authorization và resource server. Tập trung vào giao tiếp giữa các thành phần trong authorization code grant type.
* App không quản lý người dùng, bất kỳ ai cũng có thể đăng nhập bằng cách sử dụng tài khoản github.
* Github cần biết app mà nó sẽ phát hành token. Vì vậy app cần phải đăng ký với Github authorization server

### Đăng ký client với github
* Truy cập link: https://github.com/settings/applications/new
* Cần chỉ định application name, homepage url và authorization callback url (là link mà github sử dụng để gọi lại app sau khi xác thực)
* App chuyển hướng user tới Github để đăng nhập, sau đó Github gọi lại app sử dụng callback url.
![alt text](./register-oauth-app-github-1.png)
* Sau khi điền thông tin về app, Github cung cấp thông tin xác thực cho client, bao gồm ClientID và Client secrets
![alt text](./register-oauth-app-github-2.png)

### Bắt đầu tiển khai ứng dụng
* pom.xml
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

* Controller
```java
@Controller
public class MainController {
  @GetMapping("")
  public String main() {
    return "main.html";
  }
}
```

* main.html
```html
<h1>Hello there!</h1>
```

* Config
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.oauth2Login(); 
    // sử dụng phương pháp xác thực bằng oauth 2
    http.authorizeRequests() 
      .anyRequest() 
      .authenticated(); 
    // chỉ định tất cả request cần phải xác thực
  }
}
```

* Method .oauth2Login() thêm 1 filter xác thực mới vào filter chain: **OAuth2LoginAuthenticationFilter** để áp dụng các logic cần thiết cho Oauth2

### Triển khai liên kết giữa client và authorization server
* SpringSecurity cung cung 1 interface **ClientRegistration** đại diện cho client trong cấu trúc Oauth2
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
  private ClientRegistration clientRegistration() {
    ClientRegistration cr = ClientRegistration.withRegistrationId("github")
      .clientId("27670ee560e902ddd3dd")
      .clientSecret("545f7c2fb25346a95ba295836772b072df865f19")
      .scope(new String[]{"read:user"})
      // loại authority được cấp
      .authorizationUri("https://github.com/login/oauth/authorize")
      // url của authorization server
      .tokenUri("https://github.com/login/oauth/access_token")
      // url mà client gọi để nhận được access_token và refresh_token
      .userInfoUri("https://api.github.com/user")
      // url để nhận được thêm thông tin chi tiết về user
      .userNameAttributeName("id")
      .clientName("GitHub")
      .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
      .redirectUriTemplate("{baseUrl}/{action}/oauth2/code/{registrationId}")
      .build();

      // url được cung cấp trong tài liệu của github:
      // https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps
    return cr;
  }
}
```

* SpringSecurity cung cấp class **CommonOAuth2Provider**, định nghĩa 1 phần cho instance **ClientRegistration** cho các nhà cung cấp phổ biến nhất:
  * Google
  * Github
  * Facebook
  * Okta
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
  private ClientRegistration clientRegistration() { 
    return CommonOAuth2Provider.GITHUB 
      .getBuilder("github")
      // registration ID
      .clientId("27670ee560e902ddd3dd") 
      .clientSecret("545f7c2fb25346a95ba295836772b072df865f19")
      .build();
  }
}
```
### Triển khai ClientRegistrationRepository
* Đại diện cho OAuth2 client trong SpringSecurity bằng cách triển khai interface **ClientRegistrationRepository**, sử dụng nó để xác thực
* **OAuth2LoginAuthenticationFilter** lấy thông tin chi tiết về authorization server mà client đăng ký từ **ClientRegistrationRepository**. **ClientRegistrationRepository** có 1 hoặc nhiểu đối tượng **ClientRegistration**
* **ClientRegistrationRepository** tương tự interface **UserDetailsSerrvice**. **UserDetailsSerrvice** tìm kiếm **UserDetails** bằng username thì **ClientRegistrationRepository** tìm kiếm **ClientRegistration** bằng registration ID. 
* SpringSecurity cũng có 1 triển khai **ClientRegistrationRepository** có sẵn để lưu các instance **ClientRegistration** trong bộ nhớ là **InMemoryClientRegistrationRepository** hoạt động tương tự **InMemoryUserDetailsManager**

* Sử dụng **InMemoryClientRegistrationRepository** làm 1 bean trong Spring context
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
  @Bean 
  public ClientRegistrationRepository clientRepository() {
    var c = clientRegistration();
    return new InMemoryClientRegistrationRepository(c);
  }
  
  private ClientRegistration clientRegistration() {
    return CommonOAuth2Provider.GITHUB.getBuilder("github")
      .clientId("27670ee560e902ddd3dd")
      .clientSecret("545f7c2fb25346a95ba295836772b072df865f19")
      .build();
  }
 
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.oauth2Login();
    http.authorizeRequests()
      .anyRequest().authenticated();
  }
}
```
* Sử dụng object **Customizer** làm tham số trong method .oauth2Login(), thay thế có việc khai báo bean **ClientRegistration**
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
 
 @Override
 protected void configure(HttpSecurity http) throws Exception {
    http.oauth2Login(c -> { 
      c.clientRegistrationRepository(clientRepository());
    });
    http.authorizeRequests()
      .anyRequest()
    .authenticated();
  }
 
  private ClientRegistrationRepository clientRepository() {
    var c = clientRegistration();
    return new InMemoryClientRegistrationRepository(c);
  }
 
  private ClientRegistration clientRegistration() {
    return CommonOAuth2Provider.GITHUB.getBuilder("github")
      .clientId("27670ee560e902ddd3dd")
      .clientSecret("545f7c2fb25346a95ba295836772b072df865f19")
      .build();
 }
}
```

### Sử dụng application.properties thay thế cho các triển khai về ClientRegistration và ClientRegistrationRepository

```yml
spring.security.oauth2.client.registration.github.client-id=
  27670ee560e902ddd3dd
spring.security.oauth2.client.registration.github.client-secret=
  545f7c2fb25346a95ba295836772b072df865f19
# nếu sử dụng 1 provider khác với các loại phổ biến như github, facebook, google ...
# cần chỉ định thêm về authorization server, sử dụng nhóm thuộc tính bắt đầu bằng spring.security.oauth2.client.provider

# spring.security.oauth2.client.provider.myprovider.authorization-uri=<some uri>
# spring.security.oauth2.client.provider.myprovider.token-uri=<some uri>
```

### Lấy thông tin chi tiết về user đã xác thực
* Như đã biết, SpringSecurity lưu trữ thông tin chi tiết của user đã xác thục trong SecurityContext. Điều tương tự cũng xảy ra với OAuth2, filter sau khi xác thực user sẽ lưu thông tin của user trong SecutiryContext. 
* Đối tượng **Authentication** được sử dụng trong trường hợp này có tên là **OAuth2AuthenticationToken**, có thể lấy trực tiếp đối tượng từ **SecurityContextHolder** hoặc đưa nó vào làm tham số của 1 endpoint, Spring Security tự động inject đối tượng đại diện cho user này vào tham số của method
```java
@GetMapping("")
public String main(OAuth2AuthenticationToken token) { 
  logger.info(String.valueOf(token.getPrincipal()));
  // SecurityContextHolder.getContext().getAuthentication()
  return "main.html";
}
```
### Test application
* truy cập app: http://localhost:8080
* trình duyệt chuyển hướng tới URL: https://github.com/login/oauth/authorize?response_type=code&client_id=27670ee560e902ddd3dd&scope=read:user&state=fWwg5r9sKal4BMubg1oXBRrNn5y7VDW1A_rQ4UITbJk%3D&redirect_uri=http://localhost:8080/login/oauth2/code/github
* Các tham số truy vấn trong url bao gồm:
  * response_type: "code"
  * client_id
  * scope: "read:user" được xác định ttrong lớp CommonOauth2Provider
  * state: là CSRF token
* Sau khi điền thông tin đăng nhập trên github, github chuyển hướng trở lại app bằng url: http://localhost:8080/login/oauth2/code/github?code=a3f20502c182164a4086&state=fWwg5r9sKal4BMubg1oXBRrNn5y7VDW1A_rQ4UITbJk%3D 
* Trong url mà github trả về có token là a3f20502c182164a4086

* thông tin user lưu trữ trong SpringContext là:
```txt
Name: [43921235], 
Granted Authorities: [[ROLE_USER, SCOPE_read:user]], User Attributes: 
[{login=lspil, id=43921235, node_id=MDQ6VXNlcjQzOTIxMjM1, 
avatar_url=https://avatars3.githubusercontent.com/u/43921235?v=4, 
gravatar_id=, url=https://api.github.com/users/lspil, html_url=https://
github.com/lspil, followers_url=https://api.github.com/users/lspil/
followers, following_url=https://api.github.com/users/lspil/following{/
other_user}, …
```

# Triển khai Authorization Server