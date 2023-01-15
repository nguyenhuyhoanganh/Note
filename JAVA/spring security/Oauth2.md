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
* Vai trò của Authorization server là xác thực user và cung cấp token cho client. Client sử dụng token để truy cập tài nguyên được cung cấp bởi resource server.
* OAuth2 cung cấp nhiều loại luồng để nhận token. Hành vi của authorization server sẽ khác nhau tuỳ thuộc vào loại luồng cung cấp được chọn.
* Authorization server phát triển bởi Spring Security không còn được hỗ trợ nữa. Spring Security đang phát triển một authorization server mới, tuy nhiên cần thời gian để SpringSecurity hoàn thiện, lựa chọn duy nhất là phát triển 1 authorization server tuỳ chỉnh là tuân theo 1 số hướng dẫn sau đây.
* Có thể sử dụng công cụ bên thứ 3 như Keycloak hoặc Okta, tuy nhiên có thể các bên liên quan không chấp nhận giải phấp như vậy mà yêu cầu phải triển khai mã tuỳ chỉnh.

### Bắt đầu tiển khai

* pom.xml
```xml
<!--Các dependencies cần thêm -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>

<!--Thêm tag dependencyManagement cho spring-cloud-dependencies artifact ID-->
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>Hoxton.SR1</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

* Tạo 1 class config tên là **AuthServerConfig**, thêm anntation @Config và @EnableAuthorizationServer trên class để Spring bật cấu hình cụ thể cho OAuth2 authorization server. Tuỳ chỉnh cấu hình này bằng cách extends class **AuthorizationServerConfigurerAdapter** và ghi đè 1 số method cụ thể
```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {

}
```
* Authrization server sẽ quản lý user, tiếp tục sử dụng UserDetails, UserDetailsManager và UserDetailsService để quản lý thông tin xác thực, PasswordEncoder để quản lý mật khẩu. Không xuất hiện **SecurityConext** nữa do kết quả xác thực không được lưu trong **SecurityContext**. Thay vào đó, xác thực được quản lý với 1 token từ **TokenStore**
* Tạo 1 class khác chỉ để quản lý user: **WebSecurityConfig**
```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
  @Bean
  public UserDetailsService uds() {
    var uds = new InMemoryUserDetailsManager();
    var u = User.withUsername("john")
      .password("12345")
      .authorities("read")
      .build();
    uds.createUser(u);
    return uds;
  }
 
  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }
  // giữ cho các cấu hình đơn giản để tập trung vào khia cạnh OAuth2 trong ứng dụng

   @Bean 
  public AuthenticationManager authenticationManagerBean() throws Exception {
    return super.authenticationManagerBean();
  }
  // tạo ra bean AuthenticationManager bằng cách gọi lại authenticationManagerBean() từ class WebSecurityConfigurerAdapter
}
```

* Đăng ký AuthenticationManager với authorization server
```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
  @Autowired 
  private AuthenticationManager authenticationManager;
  
  @Override 
  public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
    endpoints.authenticationManager(authenticationManager);
  }
}
```

### Đăng ký client với authorization server
* Để gọi tới authorization, 1 ứng dụng hoạt động như client cần đăng ký thông tin đăng nhập của nó. Authorization server cũng quản lý cả những thông tin này của client để chỉ cho phép request được thực hiện từ những client đã biết.
* Đầu tiên, client cần đăng ký với authorization server, sau đó server cung cấp Client ID và Client Secret như là thông tin đăng nhập của client. Thông tin này sẽ dùng để xác thực client.
* Interface xác định client cho authorization server là **ClientDetails**. Chúng ta sẽ định nghĩa 1 đối tượng truy xuất **ClientDetails** bằng ID của chúng, là **ClientDetailsService**. Tương tự như với quản lý user, các interface này sử dụng để quản lý client. Bên cạnh đó có các class như **InMemoryClientDetailsService** và **JdbcClientDetailsService** tương tự với **InMemoryUserDetails** và **JdbcUserDetailsManager**, nó là các triển khai **ClientDetailsService** được SpringSecurity cung cấp sẫn, class **BaseClientDetails** là triển khai cho interface **ClientDetails**
```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
 // Omitted code
 
  @Override 
  public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
     // ghi đè lại method configure() để thiết lập instance ClientDetailsService

    /*
    var service = new InMemoryClientDetailsService();
    // tạo 1 insatnce sử dụng triển khai ClientDetailsService
    var cd = new BaseClientDetails(); 
    // tạo 1 instance của ClientDetails và thiết lập các chi tiết về client
    cd.setClientId("client"); 
    cd.setClientSecret("secret"); 
    cd.setScope(List.of("read")); 
    cd.setAuthorizedGrantTypes(List.of("password")); 
 
    service.setClientDetailsStore(Map.of("client", cd)); 
    // thêm CLientDetails vào InMemoryClientDetailsService
    clients.withClientDetails(service); 
    // cấu hình ClientDetailsService được sử dụng bởi authorization server
    */

    // các viết ngắn gọn
    // sử dụng triển khai ClientDetailsService để quản lý ClientDetails trong bộ nhớ
    // tạo và thêm insatnce của ClientDetails vào ClientDetailsServiceConfigurer
    clients.inMemory() 
      .withClient("client") 
      .secret("secret") 
      .authorizedGrantTypes("password") 
      .scopes("read");
    // trong trường hợp lưu trữ thông tin client trong database, nên sử dụng các viết bên trên
    // thông tin client lưu trong bộ nhớ không nên xảy ra trong tình huống thực tế
 }
}
```

### Sử dụng password grant type
* Với các triển khai đã làm bên trên, chúng ta đã có 1 authorization server sử dụng password grant type
* Có thể request để nhận token tại endpoint: /oauth/token . SpringSecurity tự động cấu hình endpoint này. Sử dụng thông tin đăng nhập của client với HTTP Basic để truy cập endpoint và cần thêm chi tiết về các query parameters bao gồm:
  * grant_type: "password"
  * username và passowrd: là thông tin đăng nhập của user
  * scope: là quyền hạn được cấp
* cURL có dạng: "curl -v -XPOST -u client:secret http://localhost:8080/oauth/token?grant_type=password&username=john&password=12345&scope=read"
* Phản hồi nhận được: 
```JSON
{
  "access_token":"693e11d3-bd65-431b-95ff-a1c5f73aca8c",
  "token_type":"bearer",
  "expires_in":42637,
  "scope":"read"
}
```
* Mặc định token nhận được là 1 UUID đơn giản. Client có thể sử dụng token này để gọi đến resource được cung cấp bởi resource server

### Sử dụng authorization code grant type
* Sử dụng 1 loại grant type khác được thiết lập trong khi đăng ký client, với authorization code grant type, cần cung cấp redirect URI, để authorization server chuyển hướng user khia xác thực thành công. Khi gọi tới redirect URI, authorization server cung cung cấp access token
```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
 // Omitted code
  @Override
  public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients.inMemory()
      .withClient("client")
      .secret("secret")
      .authorizedGrantTypes("authorization_code")
      .scopes("read")
      .redirectUris("http://localhost:9090/home");
  }
 
  @Override
  public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
    endpoints.authenticationManager(authenticationManager);
  }
}
```
* Có thể có nhiều client, mỗi client có thể sử dụng các grant type khác nhau, cũng có thể thiết lập nhiều grant type cho một client. Authorization serve sẽ hoạt động tuân theo request của client
```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
 // Omitted code
  @Override
  public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
   clients.inMemory()
    .withClient("client1") // client với client ID client1 chỉ sử dụng authorization_code
    .secret("secret1")
    .authorizedGrantTypes("authorization_code")
    .scopes("read")
    .redirectUris("http://localhost:9090/home")
    .and()
    .withClient("client2")
    // client với client ID là client2 có thể sử dụng nhiều loại grants
    .secret("secret2")
    .authorizedGrantTypes("authorization_code", "password", "refresh_token")
    .scopes("read")
    .redirectUris("http://localhost:9090/home");
  }
 
  @Override
  public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
    endpoints.authenticationManager(authenticationManager);
  }
}
```
* Có thể sử dụng nhiều loại grants cho một client, tuy nhiên sử dụng cách triển khai đó là sai lầm trong kiến trúc từ góc độ bảo mật. Grant type là luồng qua đó client thu thập access_token để truy cập tài nguyên cụ thể. Khi triển khai client trong 1 hệ thống, logic của client sẽ thông qua grant type mà nó sử dụng.
* Không nên chia sẻ thông tin đăng nhập của client, nghĩa là các client app sử dụng chung 1 thông tin đăng nhập. Client hoạt động như 1 thành phần độc lập có thông tin xác thực riêng, sử dụng để nhận dạng chính nó. Ngay cả khi tất cả các app được xác định là client là 1 phần của hệ thống, không có gì ngăn cản việc đăng ký client riêng biệt ở cấp độ authorization server. Việc đăng ký client riêng biệt với authorization mang lại những lợi ích sau:
  * Cung cấp khả năng  kiểm tra các sự kiện riêng lẻ từ mỗi application, khi log các sự kiện, sẽ biết được client nào tạo ra chúng
  * Cho phép cô lập mạnh mẽ. Nếu 1 cặp thông tin xác thực bị mất, chỉ có một client bị ảnh hưởng
  * Cho phép phân tách phạm vi. Có thể chi định scope khác nhau (cấp quyền khác nhau) cho từng client để thu thập token theo cách cụ thể.
* Khi muốn sử dụng authorization code grant type, server cũng cần cung cấp 1 page, nơi client chuyển hướng tới cho user đăng nhập. Triển khai page này sử dụng cấu hình form-login, ghi đè method config nhận vào HttpSecurity của class WebSecurityConfigurerAdapter\
```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
 // Omitted code
  @Override
  protected void configure(HttpSecurity http) throws Exception {
   http.formLogin();
  }
}
```

* Start application và truy cập vào link: http://localhost:8080/oauth/authorize?response_type=code&client_id=client&scope=read
* User sẽ được chuyển hướng tới login page mặc định
* Sau khi user đăng nhập, authorization server yêu cầu cấp phép hoặc từ chối với các scope được client yêu cầu
![alt text](./impl-auth-server-authorization-code-grant-type.png/.)
* Khi cấp scope, authorization chuyển hướng user tới redirect URI và cung cấp access_token, url có dạng như sau: http://localhost:9090/home?code=qeSLSt
* Client sử dụng authorization_code để thu thập token, gọi tới endpoint /oauth/token : curl -v -XPOST -u client:secret "http://localhost:8080/oauth/token?grant_type=authorization_code&scope=read&code=qeSLSt"
* Response mà client nhận được có dạng như sau:
```JSON
{
  "access_token":"0fa3b7d3-e2d7-4c53-8121-bd531a870635",
  "token_type":"bearer",
  "expires_in":43052,
  "scope":"read"
}
```
* authorization_code chỉ có thể sử dụng 1 lần. Nếu cố gắng gọi đến endpoint /oauth/token sử dụng cùng 1 code lần nữa, response nhận về như sau:
```JSON
{
  "error":"invalid_grant",
  "error_description":"Invalid authorization code: qeSLSt"
}
```

### Sử dụng client credentials grant type
* Grant type này được sử dụng cho xác thực backend-to-backend. Có thể thay thế chó phương pháp xác thực API key, cũng có thể sử dụng grant type này để bảo mật 1 endpoint không liên quan đến user cụ thể và client cần quyền truy cập (như 1 endpoint trả về trạng thái của server, client gọi đến endpoint này để kiểm tra kết nối và hiển thị trạng thái kết nối cho user). 
* Không liên quan đến bất kỳ tài nguyên dành riêng cho user nào, client có thể gọi đến mà không cần user xác thực
```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
  // Omitted code
  @Override
  public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients.inMemory()
      .withClient("client")
      .secret("secret")
      .authorizedGrantTypes("client_credentials")
      .scopes("info");
  }
}
```
* Start application và gọi đến endpoint /oauth/token để nhận access_token: "curl -v -XPOST -u client:secret "http://localhost:8080/oauth/token?grant_type=client_credentials&scope=info"
* Response nhận được:
```JSON
{
  "access_token":"431eb294-bca4-4164-a82c-e08f56055f3f",
  "token_type":"bearer",
  "expires_in":4300,
  "scope":"info"
}
```
* Cẩn thận với loại grant này, nó chỉ yêu cầu thông tin xác thực từ client, đảm bảo rằng không yêu cầu nó truy cập tới scope giống với luồng mà cần thông tin đăng nhập của user. Nếu không, có thể cho phép client truy cập vào tài nguyên của user mà không cần sự cho phép của user.

### Sử dụng refresh token grant type
* Refresh toke cung cấp một vài lợi thế đi cùng với các grant type khác. Có thể sử dụng refresh token với authorization code hoặc password grant type.
* Sử dụng đi cùng với password grant type
```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
  // Omitted code
  @Override
  public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients.inMemory()
      .withClient("client")
      .secret("secret")
      .authorizedGrantTypes("password", "refresh_token") 
      .scopes("read");
  }
}
```
* Gọi tới cURL bằng password grant type: curl -v -XPOST -u client:secret http://localhost:8080/oauth/token?grant_type=password&username=john&password=12345&scope=read
* Respone nhận được sẽ bao gồm cả refresh_token:
```JSON
{
  "access_token":"da2a4837-20a4-447d-917b-a22b4c0e9517",
  "token_type":"bearer",
  "refresh_token":"221f5635-086e-4b11-808c-d88099a76213",
  "expires_in":43199,
  "scope":"read"
}
```

# Giải pháp cho Authorization Server mới trên khung Oauth2 và OpenID
https://topdev.vn/blog/luu-registeredclient-vao-database-trong-spring-authorization-server/
https://github.com/spring-projects/spring-authorization-server
* Sử dụng JWWT
* Sử dụng interface RegisterClient, RegisterClientRepository (khác với ClientDetails và ClientDetailsService)
* Sử dựng ProviderSettings:
  * Endpoints
  * Issuer
* Sử dụng JWKSource
  * RSA key pair(s)

* Tìm trên mvnrepository : Spring Security OAuth 2 Authorization Server để lấy dependency

* Tạo UserDetailsService và PasswordEncoder bean để quản lý user
* Sử dụng loại grant type đơn giản nhất là authorization code grant type => cần sử dụng from-login, gọi method .fromLogin() từ object HttpSecurity 
```java
// WebSecurityConfig.java
@EnableWebSecurity
public class WebSecurityConfig {

  @Bean
  SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeRequests(authorizeRequests ->
        authorizeRequests.anyRequest().authenticated())
      // yêu cầu xác thực mọi request
      .formLogin(Customizer.withDefaults());
      // cần 1 form login để user xác thực
    return http.build();  
  }

   @Bean
  public UserDetailsService userDetailsService() {
    var u1 = User.withUsername("bill").password("123456").authorities("read").build();
    var uds = new InmemoryDetailsManager();
    uds.createUser(u1);
    return uds;
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }
}


// AuthorizationServerConfig.java để tách biệt config
@Configuration
public class AuthorizationServerConfig {

  private final KeyManger keyManager;
  // tiêm keyManager đẩy tạo key
  public AuthorizationServerConfig(KeyManger keyManager) {
    this.keyManager = keyManager; 
  }

  @Bean
  @Order(Ordered.HIGHEST_PRECEDENCE) 
  // đặt thứ tự ưu tiên cao hơn so với bean được khai báo ở lớp WebSecurityConfig
  // 2 bean không được đặt method cùng tên
  public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    // do chưa đủ để trở thành 1 phần của SpringSecurity, vẫn phải sử dụng bean SecurityFilterChain
    // => phải khai báo thủ công, gọi tới method static, đưa obj HttpSecurity vào làm tham số
    Oauth2AuthorizationServerConfiguration.applyDefaultSecurity(http);

    // gọi lại formLogin 2 lần ở 2 bean vị bean hiện tại triển khai cấu hình authorization serer
    // vì bean này sẽ bị ẩn đi trong tương lai và cần triển khai lại nó sử dụng from logn ở bean bên kia
    return http.formLogin(Customizer.withDefaults()).build();
  }

  // đăng ký client với auth server
  @Bean
  public RegisteredClientRepository registeredClientRepository() {
    RegisteredClient r1 = 
      RegisteredClient.withId(UUID.randomUUID().toString())
        // đây không phải client ID mà là mã định danh nội bộ, nên để 1 giá trị bất kỳ
        .clientId("client") 
        .clientSecret("secret")
        .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)  
        .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
        // bắt buộc cung cấp method xác thực với authorization grant type
        // ở đây sử dung HTTP Basic
        .scope(OidcScopes.OPENID)
        // 1 loại scope được cung cấp bởi open id connect
        // là giá trị bắt buộc với OpenID Connect Authentication Requests.openid
        .redirectUri("http://spring.io/auth")
        // đây chỉ là 1 url giá mạo, nhưng sẽ lấy code và sử dụng postman để test thử
        .build();
    return new InmemoryRegisteredClientRepository(r1);
  }

  // cung cấp ProviderSettings
  @Bean
  public ProviderSettings providerSettings() {
    return ProviderSettings.builder().build();
    // ProviderSettings xác định tất cả các endpoint cấu hình mặc định cho OAuth2 hay Open ID Connect Protocol
  }
  
  // auth serve có thể định cấu hình cho nhiều key, để xoay các key ký cho token
  // => có nhiều key pairs sử dụng private key để sign token và public key được resource server sử dụng để xác thực signature là đúng
  // key source là bean mà auth server sử dụng để lấy tất cả các key, toàn bộ key được lưu trong keyset, keyset được lấy ra bởi keysource
  @Bean
  public JWKSource<SecurityContext> jwkSource() throws NoSuchAlgorithmException {
    RSAKey rsaKey = keyManager.generateRsaKey();
    // tạo ra 1 key để thêm vào key set
    JWKSet jwkSet = new JWKSet(rsaKey);
    // 1 bộ sưu tập các key

    //lựa chọn sử dụng keyset
    return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
  }


}

// tạo KeyManager class để quản lý các key
@Component
public class KeyManager {
  // import com.nimbusds.jose.jwk.RSAKey
  public RSAKey generateRsaKey() throws NoSuchAlgorithmException {
    KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
    keyPairGenerator.initialize(2048);
    KeyPair keyPair = keyPairGenerator.generateKeyPair();
    RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
    RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();

    return new RSAKey.Builder(publicKey)
      .privateKey(privateKey)
      .keyID(UUID.randomUUID().toString())
      .build();
  } 
}

```

* Client chuyển hướng user tới auth serve với request:
http://localhost:8080/oauth2/authorize?response_type=code&client_id=client&scope=openid&redirect_uri=http://spring.io/auth&code_challenge=QYPAZ5NU8yvt1Q9ErXrUYR-T5AGCjCF47vN-KsaI2A8&code_challenge_method=S256
* lưu ý ở đây redirect_uri chỉ để minh hoạ, auth server sẽ đính kèm authorization_code trên url, sử dụng nó cho cuộc gọi thứ 2

* Sau khi user đăng nhập nhận authorization_code, client sử dụng nó để gọi cuộc gọi thứ 2
http://localhost:8080/oauth2/token?response_type=code&client_id=client&redirect_uri=http://spring.io/auth&grant_type=authorization_code&code=dWlJMGpGlUAPz0sRUly8suXDyWejo0_B4-WrLP-ks......&code_verifier=qPsH306-ZDDaOE8...

* So sánh ở 2 cuộc gọi từ client tới auth server
  * client tạo 1 chuỗi random 32bit, và băm nó bằng rs256
  * chuỗi random 32bit được gọi là code_verifier, chuỗi hash sau khi băm code_verifier bằng rs256 là code_challenge>
  * lần thực hiện cuộc gọi đầu tiên, client giử đi code_challenge kèm code_challenge_method (hàm băm), nếu có ai đó chặn cuộc gọi, nhận được code_challenge là 1 hash không thể đảo ngược.
  * lần gọi thứ 2 client sử dụng code_verifier để chứng minh client chính là tác nhân đã thực hiện cuộc gọi ban đầu
  * việc này sẽ tránh cho người xấu đánh cắp token khi chặn cuộc gọi từ client tới auth server

* Sử dụng authorization_code lấy được sau khi user xác thực và code_verifie để client xác nhận chính nó. Auth server giử lại client:
  * access_token: token của khung OAuth2
  * scope: "openid"
  * id_token: nhận được do sử dụng openid  (jwt)
  * token_type: "Bearer"// loại này ý là nếu ai có được token này thì làm bố
  * expires_in: hạn token
* id_token là 1 jwt, với body chứa sub: username, aud: clientid, ... header chứa kid: là chuỗi cung cấp định danh cho client (khác client id), alg: RS256
* truy cập vào endpoint: /.well-known/openid-configuration method GET, cung cấp cho chúng ta tất cả các endpoint của openid và oauth2 trên auth server.
* trong các endpoint được hiển thị có endpoint với key là "jwks_uri": "http://localhosst:8080/oauth2/jwks", gợi tới endpoint ta nhận được 1 mảng các keys, trong đó có key với kid tương tự giá trị kid nhận được trên header của id_token

* resource server cần xác thực token bằng cách nào đó. Làm sao resource server biết cácj xác thực token? Khi chúng ta tạo ra token, trong header có có kid, nhưng gì resource server thực sự làm là lấy kid từ header, gọi đến endpoint http://localhosst:8080/oauth2/jwks. tìm theo kid vừa lấy để tìm ra public key để xác thực với private key