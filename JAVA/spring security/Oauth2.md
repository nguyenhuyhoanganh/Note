# OAuth2

- là khung uỷ quyền (giao thức uỷe quyền) với mục đích cho phép các trang web, hoặc ứng dụng bên thứ 3 truy cập tài nguyên
- tách biệt trách nhiệm quản lý thông tin xác thực trong 1 thành phần của hệ thống (máy chủ uỷ quyền)

# Các thành phần trong OAuth2

- Resource server: máy chủ tài nguyên, là 1 ứng dụng lưu trữ tài nguyên do người dùng sở hữu
- User: người dùng, là cá nhân sở hữu tài nguyên được cung cấp bởi resource server. Thường sử dụng username và passwword để nhận dạng bản thân
- Client: máy khách, là 1 ứng dụng được phép thay mặt user truy cập các tài nguyên mà họ sở hữu. Client sử dụng client ID và client secret để nhận dạng chính nó. Những thông tin này khác với thông tin đăng nhập của user. Client cần nó để nhận dạng chính nó khi tạo request.
- Authorization server: máy chủ uỷ quyền, là ứng dụng cho phép client truy cập vào tài nguyên mà user sử hữu được cung cấp bởi resource server. Khi authorization server quyết định rạng client được uỷ quyền truy cập vào tài nguyên thay mặt user, nó phát hành 1 token. Client sử dụng token để chứng minh với resource server rằng nó được uỷ quyền bởi authorization server. Resource server cho phép client được truy cập vào tài nguyên mà nó yêu cầu nếu nó có token hợp lệ

# Các cách triển khai OAuth2

### Authorization code grant type:

#### Các bước:

1.  Client chuyển hướng user tới endpoint của authorization server, client gọi authorization endpoint với request query bao gồm:

- response_type : "code". Nói với authorzation server rằng client mong đợi 1 code, code này sử dụng để lấy token
- client_id : giá trị của client id, là giá trị xác định chính client thực hiện request
- redirect_uri: uri để authorization server chuyển hướng user sau khi xác thực thành công (có thể có hoặc không tuỳ trường hợp authorization server đã đc xác định uri chuyển hướng mặc định)
- scope : quyền được cấp (read, write, delete, ...)
- state : giá trị csrf token, sử dụng cho việc bảo vệ csrf

2.  User tương tác trực tiếp với authorization server, thực hiện xác thực (không giử thông tin đăng nhập tới client), khi xác thực thành công, Authorization server gọi lại redirect_uri về Client, cung cấp giá trị authorization code và giá trị state.
3.  Client kiểm tra state đúng với giá trị state đã giử ban đầu, đảm bảo không phải 1 bên thứ 3 gọi tới redirect_uri để đánh cắp client_secret. Khi xác nhận được đúng redirect_uri được gọi từ phía authorization server, client sử dụng authorization code nhận được thực hiện gọi 1 lần nữa tới endpoint của authorization server để nhận token với request query bao gồm:

- code : giá trị authorization code vừa nhận được
- client_id và client_secret : thông tin xác thực của client
- redirect_uri
- grant_type : "authorization_code". xác định loại grant được sử dụng (authorization server có thể hỗ trợ nhiều loại grant)

4.  Authorization server nhận request từ client

- client_id và client_secret chứng minh client là cùng 1 client đã gọi tới endpoint của authorization
- code chứa authorization code chứng minh user đã xác thưc
  => authorization server gửi access_token cho client

5.  Client nhận access_token sử dụng để đặt giá trị với key là Authorization trong header của request, giử tới resource server để lấy tài nguyên cho user

#### Tại sao không giử luôn giá trị access_token ngay lần request đầu tiên từ client tới authorization server?

- thực tế là có 1 grant type : implicit grant type sẽ trực tiếp trả về access_token cho client, tuy nhiên grant type này không được khuyến nghị sử dụng, hầu hết các authorization servers nổi tiếng cũng không cho phép. Bởi vì authorization server khi gọi tới redirect_uri với access token mà không đảm bảo rằng nó có phải là client đúng để được phép nhận. Client cần chứng minh lại chính nó sau khi nhận authorization_code bằng thông tin xác thực của nó (client_id, client_secret) để nhận được access_token

### Password grant type:

- Tương tự như authorization code grant type, tuy nhiên thay vì tương tác trực tiếp giữa user và authorization server để xác thực, user giao thông tin xác thực (gồm password) cho client
  - client sẽ xác thực thay user với authorization server
  - khi xác thực thành công, authorization trực tiếp giử access_token tới client
  - sử dụng luồng xác thực này chỉ khi client và authorization server được xây dựng và duy trì bởi cùng 1 tổ chức
- Grant type này kém bảo mật hơn authorization code. Chủ yếu bởi vì user chia sẻ thông tin đăng nhập của mình tới client app. Nó được dùng trong các tình huống thực tế do nó đơn giản hơn authorization code
- Chỉ sử dụng quy trình này nếu client à authorization server được duy trì bởi cùng một tổ chức. User chỉ mong đợi 1 biểu mẫu đăng nhập và để client lo việc giử thong tin đăng nhập đó đến authorization server. User không cần biết việc xác thực xảy ra như thế nào bên trong ứng dụng.

#### Các bước:

1.  Client thu thập thông tin đằng nhập và gọi authorization server để lấy access_token. request client giử đi bao gồm:

- grant_type: "password"
- client_id và client_secret: thông tin xác thực được client sử dụng để xác thực chính nó
- scope: là các quyền được cấp
- username và password: là thông tin đăng nhập của user. được giử đến dưới dạng plain text là các giá trị trong request header

2.  Client nhận lại access token trong response, sử dụng token đó để gọi đến các endpoint trên resource server. token được thêm vào header của request với key là Authorization

### Client credentials grant type:

- Được sử dụng khi không có user tham gia vào quy trình xác thực.
- Giả định rằng hệ thống xác thực bằng OAuth2, cần cho phép 1 server bên ngoài xác thực và gọi tới 1 tài nguyên cụ thể mà server của mình cung cấp.

#### Các bước:

1.  Client giử request tới authorization server bao gồm:

- grant_type: "client_crednetials"
- client_id và client_secret: là thông tin đăng nhập của client
- scope: đại diện cho các quyền được cấp

2.  Client nhận được một token. Sử dụng token để gọi tới endpoint của resource server. token được thêm vào header của request với key là Authorization

### Sử dụng refresh_token để nhận access_token mới

- access_token được sử dụng trong OAuth 2 không quan trọng 1 cách triển khai cụ thể nào cho nó, nó đều có hạn sử dụng. Tuổi thọ của access_token có thể là vô hạn, nhưng nên làm cho nó càng ngắn càng tốt. Khi access_token có tuổi thọ vô hạn làm cho nó tương tự với thông tin đăng nhập của user, token này có thể sử dụng để nhận được tài nguyên từ resource server mọi lúc. Trong khi access_token chỉ được đính kèm dưới dạng 1 http header trên mỗi request, việc token này bị đánh cắp tương tự việc thông tin đăng nhập củ user bị lộ.
- Vì vậy tuổi thọ của access_token nên làm cho cằng ngắn càng tốt. Khi access_token hết hạn, user cần đăng nhập lại và lấy 1 access_token mới. Ngay cả khi sử dụng password grant type, client cần phải lưu thông tin đăng nhập của user.
- refresh_token đại diện cho giải pháp thay thế việc sử dụng thông tin đăng nhập để lấy access_token mới.
- refresh_token được authorization server trả về cùng với access_token khi sử dụng các luồng xác thực như authorization code hoặc password grant type. Client sử dụng refesh_token để đưa ra request tới Authorization khi access_token hết hạn, bao gồm:
  - grant_type: "refresh_token"
  - refresh_token: giá trị của token
  - client_id và client_secret: thông tin đăng nhập của client
  - scope: xác định các quyền được cấp. Nếu cần nhiều quyền được cấp hơn, cần xác thực lại.
- Nhận được request, authorization server sẽ phát hành refresh_token và access_token mới

# Thực hiện 1 application đơn giản (SSO - single sign-on)

- SSO: user xác thực thông qua authorization server và sau đó app giữ user đăng nhập, sử dụng refresh_token. App đại diện cho client trong khung OAuth2.
- Ví dụ sử dụng github như authorization và resource server. Tập trung vào giao tiếp giữa các thành phần trong authorization code grant type.
- App không quản lý người dùng, bất kỳ ai cũng có thể đăng nhập bằng cách sử dụng tài khoản github.
- Github cần biết app mà nó sẽ phát hành token. Vì vậy app cần phải đăng ký với Github authorization server

### Đăng ký client với github

- Truy cập link: https://github.com/settings/applications/new
- Cần chỉ định application name, homepage url và authorization callback url (là link mà github sử dụng để gọi lại app sau khi xác thực)
- App chuyển hướng user tới Github để đăng nhập, sau đó Github gọi lại app sử dụng callback url.
  ![alt text](./register-oauth-app-github-1.png)
- Sau khi điền thông tin về app, Github cung cấp thông tin xác thực cho client, bao gồm ClientID và Client secrets
  ![alt text](./register-oauth-app-github-2.png)

### Bắt đầu tiển khai ứng dụng

- pom.xml

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

- Controller

```java
@Controller
public class MainController {
  @GetMapping("")
  public String main() {
    return "main.html";
  }
}
```

- main.html

```html
<h1>Hello there!</h1>
```

- Config

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

- Method .oauth2Login() thêm 1 filter xác thực mới vào filter chain: **OAuth2LoginAuthenticationFilter** để áp dụng các logic cần thiết cho Oauth2

### Triển khai liên kết giữa client và authorization server

- SpringSecurity cung cung 1 interface **ClientRegistration** đại diện cho client trong cấu trúc Oauth2

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

- SpringSecurity cung cấp class **CommonOAuth2Provider**, định nghĩa 1 phần cho instance **ClientRegistration** cho các nhà cung cấp phổ biến nhất:
  - Google
  - Github
  - Facebook
  - Okta

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

- Đại diện cho OAuth2 client trong SpringSecurity bằng cách triển khai interface **ClientRegistrationRepository**, sử dụng nó để xác thực
- **OAuth2LoginAuthenticationFilter** lấy thông tin chi tiết về authorization server mà client đăng ký từ **ClientRegistrationRepository**. **ClientRegistrationRepository** có 1 hoặc nhiểu đối tượng **ClientRegistration**
- **ClientRegistrationRepository** tương tự interface **UserDetailsSerrvice**. **UserDetailsSerrvice** tìm kiếm **UserDetails** bằng username thì **ClientRegistrationRepository** tìm kiếm **ClientRegistration** bằng registration ID.
- SpringSecurity cũng có 1 triển khai **ClientRegistrationRepository** có sẵn để lưu các instance **ClientRegistration** trong bộ nhớ là **InMemoryClientRegistrationRepository** hoạt động tương tự **InMemoryUserDetailsManager**

- Sử dụng **InMemoryClientRegistrationRepository** làm 1 bean trong Spring context

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

- Sử dụng object **Customizer** làm tham số trong method .oauth2Login(), thay thế có việc khai báo bean **ClientRegistration**

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

- Như đã biết, SpringSecurity lưu trữ thông tin chi tiết của user đã xác thục trong SecurityContext. Điều tương tự cũng xảy ra với OAuth2, filter sau khi xác thực user sẽ lưu thông tin của user trong SecutiryContext.
- Đối tượng **Authentication** được sử dụng trong trường hợp này có tên là **OAuth2AuthenticationToken**, có thể lấy trực tiếp đối tượng từ **SecurityContextHolder** hoặc đưa nó vào làm tham số của 1 endpoint, Spring Security tự động inject đối tượng đại diện cho user này vào tham số của method

```java
@GetMapping("")
public String main(OAuth2AuthenticationToken token) {
  logger.info(String.valueOf(token.getPrincipal()));
  // SecurityContextHolder.getContext().getAuthentication()
  return "main.html";
}
```

### Test application

- truy cập app: http://localhost:8080
- trình duyệt chuyển hướng tới URL: https://github.com/login/oauth/authorize?response_type=code&client_id=27670ee560e902ddd3dd&scope=read:user&state=fWwg5r9sKal4BMubg1oXBRrNn5y7VDW1A_rQ4UITbJk%3D&redirect_uri=http://localhost:8080/login/oauth2/code/github
- Các tham số truy vấn trong url bao gồm:
  - response_type: "code"
  - client_id
  - scope: "read:user" được xác định ttrong lớp CommonOauth2Provider
  - state: là CSRF token
- Sau khi điền thông tin đăng nhập trên github, github chuyển hướng trở lại app bằng url: http://localhost:8080/login/oauth2/code/github?code=a3f20502c182164a4086&state=fWwg5r9sKal4BMubg1oXBRrNn5y7VDW1A_rQ4UITbJk%3D
- Trong url mà github trả về có token là a3f20502c182164a4086

- thông tin user lưu trữ trong SpringContext là:

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

# Authorization Server

- Vai trò của Authorization server là xác thực user và cung cấp token cho client. Client sử dụng token để truy cập tài nguyên được cung cấp bởi resource server.
- OAuth2 cung cấp nhiều loại luồng để nhận token. Hành vi của authorization server sẽ khác nhau tuỳ thuộc vào loại luồng cung cấp được chọn.
- Authorization server phát triển bởi Spring Security không còn được hỗ trợ nữa. Spring Security đang phát triển một authorization server mới, tuy nhiên cần thời gian để SpringSecurity hoàn thiện, lựa chọn duy nhất là phát triển 1 authorization server tuỳ chỉnh là tuân theo 1 số hướng dẫn sau đây.
- Có thể sử dụng công cụ bên thứ 3 như Keycloak hoặc Okta, tuy nhiên có thể các bên liên quan không chấp nhận giải phấp như vậy mà yêu cầu phải triển khai mã tuỳ chỉnh.

### Triển khai Authorization Server

- pom.xml

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

- Tạo 1 class config tên là **AuthServerConfig**, thêm anntation @Config và @EnableAuthorizationServer trên class để Spring bật cấu hình cụ thể cho OAuth2 authorization server. Tuỳ chỉnh cấu hình này bằng cách extends class **AuthorizationServerConfigurerAdapter** và ghi đè 1 số method cụ thể

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {

}
```

- Authrization server sẽ quản lý user, tiếp tục sử dụng UserDetails, UserDetailsManager và UserDetailsService để quản lý thông tin xác thực, PasswordEncoder để quản lý mật khẩu. Không xuất hiện **SecurityConext** nữa do kết quả xác thực không được lưu trong **SecurityContext**. Thay vào đó, xác thực được quản lý với 1 token từ **TokenStore**
- Tạo 1 class khác chỉ để quản lý user: **WebSecurityConfig**

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
  // tạo ra bean AuthenticationManager bằng cách gọi lại authenticationManagerBean() từ class WebSecurityConfigurerAdapter, bean này tái sử dụng trong config oauth2
}
```

- Đăng ký AuthenticationManager với authorization server

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

- Để gọi tới authorization, 1 ứng dụng hoạt động như client cần đăng ký thông tin đăng nhập của nó. Authorization server cũng quản lý cả những thông tin này của client để chỉ cho phép request được thực hiện từ những client đã biết.
- Đầu tiên, client cần đăng ký với authorization server, sau đó server cung cấp Client ID và Client Secret như là thông tin đăng nhập của client. Thông tin này sẽ dùng để xác thực client.
- Interface xác định client cho authorization server là **ClientDetails**. Chúng ta sẽ định nghĩa 1 đối tượng truy xuất **ClientDetails** bằng ID của chúng, là **ClientDetailsService**. Tương tự như với quản lý user, các interface này sử dụng để quản lý client. Bên cạnh đó có các class như **InMemoryClientDetailsService** và **JdbcClientDetailsService** tương tự với **InMemoryUserDetails** và **JdbcUserDetailsManager**, nó là các triển khai **ClientDetailsService** được SpringSecurity cung cấp sẫn, class **BaseClientDetails** là triển khai cho interface **ClientDetails**

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

- Với các triển khai đã làm bên trên, chúng ta đã có 1 authorization server sử dụng password grant type
- Có thể request để nhận token tại endpoint: /oauth/token . SpringSecurity tự động cấu hình endpoint này. Sử dụng thông tin đăng nhập của client với HTTP Basic để truy cập endpoint và cần thêm chi tiết về các query parameters bao gồm:
  - grant_type: "password"
  - username và passowrd: là thông tin đăng nhập của user
  - scope: là quyền hạn được cấp
- cURL có dạng: "curl -v -XPOST -u client:secret http://localhost:8080/oauth/token?grant_type=password&username=john&password=12345&scope=read"
- Phản hồi nhận được:

```JSON
{
  "access_token":"693e11d3-bd65-431b-95ff-a1c5f73aca8c",
  "token_type":"bearer",
  "expires_in":42637,
  "scope":"read"
}
```

- Mặc định token nhận được là 1 UUID đơn giản. Client có thể sử dụng token này để gọi đến resource được cung cấp bởi resource server

### Sử dụng authorization code grant type

- Sử dụng 1 loại grant type khác được thiết lập trong khi đăng ký client, với authorization code grant type, cần cung cấp redirect URI, để authorization server chuyển hướng user khia xác thực thành công. Khi gọi tới redirect URI, authorization server cung cung cấp access token

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

- Có thể có nhiều client, mỗi client có thể sử dụng các grant type khác nhau, cũng có thể thiết lập nhiều grant type cho một client. Authorization serve sẽ hoạt động tuân theo request của client

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

- Có thể sử dụng nhiều loại grants cho một client, tuy nhiên sử dụng cách triển khai đó là sai lầm trong kiến trúc từ góc độ bảo mật. Grant type là luồng qua đó client thu thập access_token để truy cập tài nguyên cụ thể. Khi triển khai client trong 1 hệ thống, logic của client sẽ thông qua grant type mà nó sử dụng.
- Không nên chia sẻ thông tin đăng nhập của client, nghĩa là các client app sử dụng chung 1 thông tin đăng nhập. Client hoạt động như 1 thành phần độc lập có thông tin xác thực riêng, sử dụng để nhận dạng chính nó. Ngay cả khi tất cả các app được xác định là client là 1 phần của hệ thống, không có gì ngăn cản việc đăng ký client riêng biệt ở cấp độ authorization server. Việc đăng ký client riêng biệt với authorization mang lại những lợi ích sau:
  - Cung cấp khả năng kiểm tra các sự kiện riêng lẻ từ mỗi application, khi log các sự kiện, sẽ biết được client nào tạo ra chúng
  - Cho phép cô lập mạnh mẽ. Nếu 1 cặp thông tin xác thực bị mất, chỉ có một client bị ảnh hưởng
  - Cho phép phân tách phạm vi. Có thể chi định scope khác nhau (cấp quyền khác nhau) cho từng client để thu thập token theo cách cụ thể.
- Khi muốn sử dụng authorization code grant type, server cũng cần cung cấp 1 page, nơi client chuyển hướng tới cho user đăng nhập. Triển khai page này sử dụng cấu hình form-login, ghi đè method config nhận vào HttpSecurity của class WebSecurityConfigurerAdapter\

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

- Start application và truy cập vào link: http://localhost:8080/oauth/authorize?response_type=code&client_id=client&scope=read
- User sẽ được chuyển hướng tới login page mặc định
- Sau khi user đăng nhập, authorization server yêu cầu cấp phép hoặc từ chối với các scope được client yêu cầu
  ![alt text](./impl-auth-server-authorization-code-grant-type.png/.)
- Khi cấp scope, authorization chuyển hướng user tới redirect URI và cung cấp access_token, url có dạng như sau: http://localhost:9090/home?code=qeSLSt
- Client sử dụng authorization_code để thu thập token, gọi tới endpoint /oauth/token : curl -v -XPOST -u client:secret "http://localhost:8080/oauth/token?grant_type=authorization_code&scope=read&code=qeSLSt"
- Response mà client nhận được có dạng như sau:

```JSON
{
  "access_token":"0fa3b7d3-e2d7-4c53-8121-bd531a870635",
  "token_type":"bearer",
  "expires_in":43052,
  "scope":"read"
}
```

- authorization_code chỉ có thể sử dụng 1 lần. Nếu cố gắng gọi đến endpoint /oauth/token sử dụng cùng 1 code lần nữa, response nhận về như sau:

```JSON
{
  "error":"invalid_grant",
  "error_description":"Invalid authorization code: qeSLSt"
}
```

### Sử dụng client credentials grant type

- Grant type này được sử dụng cho xác thực backend-to-backend. Có thể thay thế chó phương pháp xác thực API key, cũng có thể sử dụng grant type này để bảo mật 1 endpoint không liên quan đến user cụ thể và client cần quyền truy cập (như 1 endpoint trả về trạng thái của server, client gọi đến endpoint này để kiểm tra kết nối và hiển thị trạng thái kết nối cho user).
- Không liên quan đến bất kỳ tài nguyên dành riêng cho user nào, client có thể gọi đến mà không cần user xác thực

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

- Start application và gọi đến endpoint /oauth/token để nhận access_token: "curl -v -XPOST -u client:secret "http://localhost:8080/oauth/token?grant_type=client_credentials&scope=info"
- Response nhận được:

```JSON
{
  "access_token":"431eb294-bca4-4164-a82c-e08f56055f3f",
  "token_type":"bearer",
  "expires_in":4300,
  "scope":"info"
}
```

- Cẩn thận với loại grant này, nó chỉ yêu cầu thông tin xác thực từ client, đảm bảo rằng không yêu cầu nó truy cập tới scope giống với luồng mà cần thông tin đăng nhập của user. Nếu không, có thể cho phép client truy cập vào tài nguyên của user mà không cần sự cho phép của user.

### Sử dụng refresh token grant type

- Refresh toke cung cấp một vài lợi thế đi cùng với các grant type khác. Có thể sử dụng refresh token với authorization code hoặc password grant type.
- Sử dụng đi cùng với password grant type

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

- Gọi tới cURL bằng password grant type: curl -v -XPOST -u client:secret http://localhost:8080/oauth/token?grant_type=password&username=john&password=12345&scope=read
- Respone nhận được sẽ bao gồm cả refresh_token:

```JSON
{
  "access_token":"da2a4837-20a4-447d-917b-a22b4c0e9517",
  "token_type":"bearer",
  "refresh_token":"221f5635-086e-4b11-808c-d88099a76213",
  "expires_in":43199,
  "scope":"read"
}
```

# Giải pháp cho Authorization Server mới trên frame work Oauth2 và OpenID (Spring Security OAuth 2 Authorization Server version 0.3) (2022)

https://topdev.vn/blog/luu-registeredclient-vao-database-trong-spring-authorization-server/
https://github.com/spring-projects/spring-authorization-server

- Sử dụng JWWT
- Sử dụng interface RegisterClient, RegisterClientRepository (khác với ClientDetails và ClientDetailsService)
- Sử dựng ProviderSettings:
  - Endpoints
  - Issuer
- Sử dụng JWKSource

  - RSA key pair(s)

- Tìm trên mvnrepository : Spring Security OAuth 2 Authorization Server để lấy dependency

- Tạo UserDetailsService và PasswordEncoder bean để quản lý user
- Sử dụng loại grant type đơn giản nhất là authorization code grant type => cần sử dụng from-login, gọi method .fromLogin() từ object HttpSecurity

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
        // đây không phải client ID, Id này như 1 mã định danh cho client, trong khi đó clientID giống với username hơn
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

- Client chuyển hướng user tới auth server với request:
  http://localhost:8080/oauth2/authorize?response_type=code&client_id=client&scope=openid&redirect_uri=http://spring.io/auth&code_challenge=QYPAZ5NU8yvt1Q9ErXrUYR-T5AGCjCF47vN-KsaI2A8&code_challenge_method=S256 (redirect_uri chỉ để minh hoạ, auth server sẽ đính kèm authorization_code trên url, sử dụng nó cho cuộc gọi thứ 2)

- Sau khi user đăng nhập nhận authorization_code, client sử dụng nó để gọi cuộc gọi thứ 2
  http://localhost:8080/oauth2/token?response_type=code&client_id=client&redirect_uri=http://spring.io/auth&grant_type=authorization_code&code=dWlJMGpGlUAPz0sRUly8suXDyWejo0_B4-WrLP-ks......&code_verifier=qPsH306-ZDDaOE8...

- So sánh ở 2 cuộc gọi từ client tới auth server

  - client tạo 1 chuỗi random 32bit, và băm nó bằng sha256
  - chuỗi random 32bit được gọi là code_verifier, chuỗi hash sau khi băm code_verifier bằng sha256 là code_challenge.
  - lần thực hiện cuộc gọi đầu tiên, client giử đi code_challenge kèm code_challenge_method (hàm băm), nếu có ai đó chặn cuộc gọi, chỉ nhận được code_challenge là 1 hash không thể đảo ngược.
  - lần gọi thứ 2 client sử dụng code_verifier để chứng minh client chính là tác nhân đã thực hiện cuộc gọi ban đầu
  - việc này sẽ tránh cho người xấu đánh cắp token khi chặn cuộc gọi từ client tới auth server

- Sử dụng authorization_code lấy được sau khi user xác thực và code_verifie để client xác nhận chính nó. Auth server sẽ giử lại client:

  - access_token: token của frame work OAuth2
  - scope: "openid"
  - id_token: nhận được do sử dụng openid (jwt)
  - token_type: "Bearer"// loại này ý là nếu ai có được token này thì làm bố
  - expires_in: hạn token

- id_token là 1 JWT gồm 3 phần
  - header:
    - alg: thuật toán sử dụng để mã hoá signature
    - kid: mã định danh cho cặp key, sử dụng để resource server tìm đến được public key
  - payload:
    - iss: tổ chức phát hành token (0 bắt buộc)
    - sub: người phát hành token (username)
    - aud: đối tượng sử dụng token (clientId, 0 bắt buộc)
    - exp: thời điểm hết hạn tokrn
  - signature (JWS): phần chữ ký, kết hợp 2 phần header và payload và mã hoá bằng private key
- Auth server cung cấp endpointL /.well-known/openid-configuration method GET, cung cấp cho chúng ta tất cả các endpoint của openid và oauth2 trên auth server.
- trong các endpoint được hiển thị có endpoint với key là "jwks_uri": "http://localhosst:8080/oauth2/jwks", endpoint trả về 1 bộ keyset, trong đó có key với kid tương tự giá trị kid nhận được trên header của id_token
- resource server cần xác thực JWT (id_token) nhận được, resource server lấy ra kid từ header của JWT, gọi đến endpoint http://localhosst:8080/oauth2/jwks và tìm theo kid vừa lấy để tìm ra public key. Public key sử dụng để verify signature trong token

- JWK đại diện cho khoá mật mã, JWKS đại diện cho 1 tập các JWK
- JWT là tokken được giử đi, bao gồm 3 phần header, payload, signature (JWS hoặc JWE)
- JWK sử dụng để xác minh phần JWS (signature) trong JWT

# PCKE trong authorization code grant type:

### Ý tưởng của PCKE

- Thông tin xác thực của client cần lưu trữ ở client, tuỳ nhiên client là public, vì vậy rất khó để giữ thông tin này an toàn
- PCKE là ý tưởng không sử dụng thông tin xác thực của client.

### Luồng PCKE

1. Khi user yêu cầu login tại client, client sẽ tạo ra 2 phần thông tin:

- verifier: 1 phần thông tin ngẫu nhiên
- challenge: làm hash của challenge
  Trong 2 phần thông tin này, phần challenge được giử đi khi client chuyển hướng user tới auth server để login. Phần challenge được hash bằng 1 hàm mã hoá không thể đảo ngược, khi request redirect bị ai đó chặn, thông tin về giá trị hash có thể bị mất, người chặn cũng có thể biết được về hàm mã hoá tuy nhiên không thế lấy được giá trị verifier.

2. Khi user đăng nhập thành công, auth server trả về authorization_code cho client, client tạo request đến auth server để lấy access_token, request này sẽ đi kèm authorization_code vừa nhận được và giá trị verifier mà client tạo ban đầu. Giá trị verifier này để xác minh chính client là tác nhân chuyển hướng user tới auth server ban đầu, bởi vì chỉ có client mới biết giá trị verifier được random. Auth server bây giờ chỉ thực hiện so khớp 2 giá trị challenge và verifier nhận được ở 2 lần request từ client

# Open ID

- Open ID là 1 lớp nhận dạng đơn giản trên giao thức OAuth2, cho phép client xác minh danh tính của user dựa trên xác thực được thực hiện bởi auth server, để thu thập thông tin về profile của user
- Vấn đề về Oauth2 là nó chỉ thực hiện việc uỷ quyền, nhưng không phải về xác thực (Authentication). Với việc client sử dụng access_token để truy cập resource, client không biết gì về user, không biết username, email address, nickname, ...
- Có thể tốt nếu client không cần biết bất cứ thông tin gì về user nhưng trong 1 số trường hợp, client thực sự cần 1 vài thứ về user như first name, ... Open Id thêm 1 số thông tin nhận dạng trên Oauth2
- Khác duy nhất của OpenID so với authorization grant type của OAuth2 là bổ sung thêm thuộc tính _scope_ khi redirect user tới auth server
- Scope cơ bản nhất OpenId cung cấp là "openid", với mong muốn auth server cung cấp thêm thông tin về user, dưới dạng identity token (id_token). id_token là 1 jwt có chứa 1 ít thông tin về user, tuỳ thuộc vào thêm giá trị trong scope (bắt buộc có scope openid), nó có thể chứa nhiều hoặc ít thông tin.
  - "email" cung cấp thông tin cho email của user
  - "profile" cung cấp thông tin về hồ sơ của user
- Vì vậy khi sử dụng scope là token, ngoài access_token (có thể là bất cứ loại chuỗi nào, thường là jwt), response của auth server chứa thêm id_token (dưới dạng jwt)
- Ex: với scope là openid, trong payload của id_token chứa thông tin về iss, sub, aud, exp, iat. Nếu thêm scope email, payload này sẽ chứ thêm email, email_verified

```json
{
  "iss": "https://accounts.google.com",
  "sub": "<locally-unique identifier for end user>",
  "aud": "<client id>",
  "exp": 1634987478,
  "iat": 1634986478,
  "email": "john@example.com",
  "email_verified": true
}
```

# Resource Server

- Resource cần thực hiện xác thực token được nhận từ client
- Có nhiều cách để xác thực token:
  - Trực tiếp gọi đên auth server để xác thực token
  - Sử dụng database lưu trữ token chung giữa auth server và resource server (blackboarding)
  - Sử dụng cryptographic signatures, auth server sign token bằng primary key khi phát hành nó và resource sẽ xác nhận token signature

### Triển khai Resource Server

- pom.xml

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
```

- thêm dependencyManagement tag cho spring-cloud-dependencies artifact

```xml
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

- thêm endpoint là đại diện cho resource mà user muốn truy cập

```java
@RestController
public class HelloController {
  @GetMapping("/hello")
  public String hello() {
    return "Hello!";
  }
}
```

- Tạo 1 class config, đi kèm annatiation @EnableResourceServer để Sping định cấu hình cho app trở thành resource server

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig {
  /// code
}
```

- do việc triển khai resource cũng đang được thay đổi nên annotation @EnableResourceServer đang không được dùng nữa, đang dần chuyển sang sử dụng config method trực tiếp, tuỳ nhiên vẫn gặp phải việc sử dụng trong các project cũ

### Kiểm tra token từ xa

- Triển khai xác thưucs token bằng cách cho phép resource server gọi trực tiếp tới auth server
- Sử dụng phương pháp này nếu token là 1 chuỗi đơn giản như UUID
- Cơ chế xác thực:
  - Auth server cung cấp 1 endpoint. Đối với 1 token hợp lệ, nó trả về các quyền hạn của user, cấp quyền cho client. Gọi endoint này là check_token.
  - Resource server gọi tới check_token endpoint mỗi request. Cách này giúp nó xác thực token nhận được từ client và cũng nhận được các quyền được cấp của client.
- Cách này đơn giản, có thể áp dụng cho bất kỳ loại triển khai token nào. Tuy nhiên với mỗi request tới resource server có 1 token mới, resource server lại gọi tới auth server xác thực token, gây ra tải trọng không cần thiết cho auth server.
- Mặc định, auth server triển hai endpoint /oauth/check_token để resource server có thể sử dụng để xác thực token. Tuy nhiên hiện tại auth server ngầm từ chới tất cả requesst tới endpoint đó, => cần phải cấu hình để resouce server có thể gọi tới endpoit đó
- Để cho phép reuquest đã xác thực gọi tới /oauth/check_token, cần ghi đè method configure(AuthorizationServerSecurityConfigurer config)

- Cấu hình cho auth server sử dụng endpoint /oauth/check_token để xác thực token
  - ghi đè lại method _configure(AuthorizationServerSecurityConfigurer config)_ của

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
  @Autowired
  private AuthenticationManager authenticationManager;

  @Override
  public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
   clients.inMemory()
    .withClient("client")
    .secret("secret")
    .authorizedGrantTypes("password", "refresh_token")
    .scopes("read");
  }

  @Override
  public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
   endpoints.authenticationManager(authenticationManager);
  }

  // chỉ định điều kiện sau để có thể gọi đến endpoint /oauth/check_token
  public void configure(AuthorizationServerSecurityConfigurer config) {
   config.checkTokenAccess("isAuthenticated()");
   // có thể làm cho endpoint này có thể truy cập được mà không cần xác thực
   // sử dụng permitAll() thay thế cho isAuthenticated()
  }
}
```

- Bên cạnh việc làm cho endpoint oauth/check_token có thể truy cập, nếu quyết định chỉ cho phép tuy cập đã được xác thực, cần phải đăng ký resource server làm chính client của auth server.
- Thêm resorce server làm client cho auth server, không cần thêm grant type hoặc scope khi đăng ký client, chỉ cần thêm thông tin về client Id và client secret là có thể gọi tới endpoint oauth/check_token

```java
// thêm resource server làm client tại auth server
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
      .scopes("read")
      .and()
      .withClient("resourceserver")
      .secret("resourceserversecret");
  }
}
```

- Khởi động auth server và gọi cURL để lấy token:

```txt
curl -v -XPOST -u client:secret "http://localhost:8080/oauth/token?grant_type=password&username=john&password=12345&scope=read"
```

- Auth server trả về response cho client như sau:

```json
{
  "access_token": "4f2b7a6d-ced2-43dc-86d7-cbe844d3e16b",
  "token_type": "bearer",
  "refresh_token": "a4bd4660-9bb3-450e-aa28-2e031877cb36",
  "expires_in": 43199,
  "scope": "read"
}
```

- Gọi tới check\_\_token endpoint để lấy thông tin chi tiết về access_token theo cURL sau:

```txt
curl -XPOST -u resourceserver:resourceserversecret "http://localhost:8080/oauth/check_token?token=4f2b7a6d-ced2-43dc-86d7-cbe844d3e16b"
```

- Sau khi gọi đến check_token endpoint sẽ nhận được:

```json
{
  "active": true,
  "exp": 1581307166,
  "user_name": "john",
  "authorities": ["read"],
  "client_id": "client",
  "scope": ["read"]
}
```

- resource server hoạt đọng như 1 client cho auth server, vì vậy, nó cần đăng ký 1 số thông tin xác thực, sử dụng xác thực bằng HTTP Basic khi gọi đến endpoint check_token
- Sử dụng application.properties để cấu hình cho resource server biết endpoint gọi đến để xác thực token cùng như thông tin mà resource server đăng ký với vai trò là client của auth server

```yml
server.port=9090
security.oauth2.resource.token-info-uri=http:/./localhost:8080/oauth/check_token
security.oauth2.client.client-id=resourceserver
security.oauth2.client.client-secret=resourceserversecret
```

- Client khi gọi đến resource server, cần đặt token tại header Authorization, thêm tiền tố "Bearer " cho token:

```txt
curl -H "Authorization: bearer 4f2b7a6d-ced2-43dc-86d7-cbe844d3e16b"
  "http:/./localhost:9090/hello"
```

#### Triển khai mới cho resource sử dụng spring-security-oauth2-resource-server

- Hiện nay Spring Security Oauth không còn được dùng nữa, vì vậy cần tìm hiểu cách triển khai resource server sử dụng token bên trong mà không dùng Spring Security OAuth, thay vào đó sử dụng các cấu hình trực tiếp với Spring Security
- Spring cung cấp 1 method _.oauth2ResourceServer()_ cho phép xác thực resource server. Method này giống .httpBasic() hay .formLogin(), cũng thêm 1 filter triển khai xác thực riêng vào filter chain

- Sử dụng nó cần thêm 1 dependencies khác:

```xml
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-oauth2-resource-server</artifactId>
  <version>5.2.1.RELEASE</version>
</dependency>
<dependency>
  <groupId>com.nimbusds</groupId>
  <artifactId>oauth2-oidc-sdk</artifactId>
  <version>8.4</version>
  <scope>runtime</scope>
</dependency>
```

- Sử dụng phương thức xác thực như sau:

```java
@Configuration
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
      .anyRequest().authenticated()
      .and()
      .oauth2ResourceServer(c -> c.opaqueToken(
        o -> {
          o.introspectionUri("…");
          // method introspectionUri để xác định uri check_token
          o.introspectionClientCredentials("client", "secret");
        })
      );
  }
}
```

### Kiểm tra token sử dụng BlackBoarding với JdbcTokenStore

- Triển khai auth server và resource server sử dụng chung 1 database để lưu token
- Loại bỏ việc giao tiếp giữa auth server và resource server
- Sau khi xác thực kết thúc, auth server sử dụng ToeknStore để tạo ra 1 token
- Ở resource server, filter xác thực sử dụng ToeknStore để xác thực token và tìm chi tiết user mà nó sử dụng sau đó để uỷ quyền. Resource server sau đó lưu chi tiết user vào security context

- Spring cung cấp nhiều cách triển khai khác nhau cho TokenStore, mặc định Spring đã cung cấp 1 token store với type InMemoryTokenStore, nghĩa là token được lưu trữ trong bộ nhớ. Nếu khởi động lại auth server thì token được phát hành trước đó không còn hiệu lực. Việc này đáp ứng cho trường hợp kiểm tra token từ xa, gọi đến endpoint oauth/check_token
- Để quản lý token với blackboarding, SpringSecurity cung câos JdbcTokenStore, hoạt động với database thông qua jdbc

- JdbcTokenStore mong đợi có 2 table, oauth_access_token để lưu trữ access_token và oauth_refresh_token để lưu trữ refresh_token
- Có thể tuỳ chỉnh JdbcTokenStore sử dụng tên khác cho các table hoặc column. Khi đó các method của JdbcTokenStore phải ghi đè bất kỳ câu lệnh SQL nào mà nó sử dụng để truy xuất hoặc lưu trữ thông tin về token

#### Sau đây là triển khai auth server giữ JdbcTokenStore mặc định:

- pom.xml

```xml
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
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
```

- schema.sql

```sql
CREATE TABLE IF NOT EXISTS `oauth_access_token` (
  `token_id` varchar(255) NOT NULL,
  `token` blob,
  `authentication_id` varchar(255) DEFAULT NULL,
  `user_name` varchar(255) DEFAULT NULL,
  `client_id` varchar(255) DEFAULT NULL,
  `authentication` blob,
  `refresh_token` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`token_id`));

CREATE TABLE IF NOT EXISTS `oauth_refresh_token` (
  `token_id` varchar(255) NOT NULL,
  `token` blob,
  `authentication` blob,
  PRIMARY KEY (`token_id`));
```

- xác định data source trong application.properties

```yml
spring.datasource.url=jdbc:mysql://localhost/spring?useLegacyDatetimeCode=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=
spring.datasource.initialization-mode=always
```

- class AuthServerConfig

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {

  @Autowired
  private AuthenticationManager authenticationManager;

  @Autowired
  private DataSource dataSource; // inject DataSource đã được cấu hình trong application.properties

  @Override
  public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients.inMemory()
      .withClient("client")
      .secret("secret")
      .authorizedGrantTypes("password", "refresh_token")
      .scopes("read");
  }

  @Override
  public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
      endpoints.authenticationManager(authenticationManager)
        .tokenStore(tokenStore()); // cấu hình tokenStore
  }

// tạo 1 bean TokenStore trả về JdbcTokenStore
  @Bean
  public TokenStore tokenStore() {
    return new JdbcTokenStore(dataSource);
    // cung cấp truy cập tới database thông qua DataSource
  }
}
```

- Chạy auth server, gọi đến cURL sau:

```txt
curl -v -XPOST -u client:secret "http://localhost:8080/oauth/token?grant_type=password&username=john&password=12345&scope=read"
```

- Response trả về:

```json
{
  "access_token": "009549ee-fd3e-40b0-a56c-6d28836c4384",
  "token_type": "bearer",
  "refresh_token": "fd44d772-18b3-4668-9981-86373017e12d",
  "expires_in": 43199,
  "scope": "read"
}
```

- access_token trả về trong response cung được tìm thấy như 1 record trong table oauth_access_token, refresh token cũng vậy

#### Triẻn khai resource serer

- pom.xml

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
```

- application.properties

```yml
server.port=9090
spring.datasource.url=jdbc:mysql://localhost/spring
spring.datasource.username=root
spring.datasource.password=
```

- class ResourceServerConfig

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
  @Autowired
  private DataSource dataSource;
  // inject data source đã cấu hình trong application.properties

  @Override
  public void configure(ResourceServerSecurityConfigurer resources) {
   resources.tokenStore(tokenStore());
  }
  // cấu hình sử dụng tokenStore

  @Bean
  public TokenStore tokenStore() {
   return new JdbcTokenStore(dataSource);
  }
  // toạ bean JdbcTokenStore dựa vào dataSource để truy cập database
}
```

- Gọi đến endpoint /hello sử dụng access_token được cung cấp trước đó theo cURL sau:

```txt
curl -H "Authorization:Bearer 009549ee-fd3e-40b0-a56c-6d28836c4384" "http://localhost:9090/hello"
```

- năm 2020 hiện chưa có cách cấu hình ResourceServerConfig sử dụng blackboarding mà không có phụ thuộc Spring Security Oauth

# Sử dụng JWT và Cryptographic Signatures đễ xác thực token

- Sử dụng Cryptographic Signatures để xác thực token giúp resource server xác thưcj mà không cần gọi trực tiếp tới auth server hay sử dụng database để chia sẻ token

### Sử dụng token được ký bằng symmetric keys (key đối xứng) với Jwt

- Cách tiếp cận đơn giản nhất để ký token là sử dụng symmetric keys. Sử dụng cùng 1 key có thể vừa ký token và vừa xác thực nó
- Nhược điểm là không phải lúc nào cũng có thể chi sẻ key sử dụng để ký token với tất cả các ứng dụng tham gia quá trình xác thực
- Sử dụng với JWT làm token
- JWT gồm 3 phần header, payload và signature. các thông tin về header và payload được biểu diễn bằng JSON và mã hoá bằng Base64. Phần signature được toạ bằng 1 thuật toán cryptographic sử dụng input là phần header và payload . Thuật toán cryptographic cần 1 key, key giống như pasword. Ai đó có key thích hợp có thể ký token hoặc xác thực signature của token.
- Nếu signature trên token là xác thực, sẽ đảm bảo không ai thay đổi token sau khi nó đã được ký.
- Khi JWT được ký, có thể gọi nó là JWS. Thông thường áp dụng thuật toán cryptographic để ký token là đủ, tuy nhiên thỉnh thoảng có thể chọn mã hoá (encrypted) nó. Nếu token đã được ký, có thể xem nội dung của nó mà không cần bất kỳ key nào. Nhưng ngay cả khi hacker thấy được nội dung của token cũng không thể thay đổi nó bỏi vì nếu làm vậy thì signature sẽ không hợp lệ. Để hợp lệ, signature phải:
  - được tạo bằng key chính xác
  - khớp với nội dụng mà đã ký
- Nếu token được mã hoá, nó cung có thể gọi là JWE, không thể nhìn thấy nối dụng của token được mã hoá nếu không có key hợp lệ

- 1 triển khai TokenStore được Spring cung cấp là JwtTokenStore dùng để quản lý JWTs. Có thể xác thực token theo 2 cách:

  - sử dụng cùng 1 key để ký token và xác thực signature, key được gọi là symmetric (đối xứng)
  - sử dụng 1 key để ký token và 1 key khác để xác thực signature, 2 key được gọi là asymmetric key pair (cặp key bất đối xứng)

- Dưới đây là cách triển khai sử dụng symmetric key, cả auth server và resource server đều biết và sử dụng chung 1 key. Auth server ký token bnawgf jey và resource server xác thực signature bằng chính key đó

#### Triển khai auth server

- pom.xml

```xml
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
```

- Cấu hình JwtTokenStore tương tự như JdbcTokenStore, ngoài ra cần xác định 1 object type JwtAccessTokenConverter. Với object này, sử dụng để cấu hình cách auth server xác thực token, sử dụng symmetric key

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {

  @Value("${jwt.key}")
  private String jwtKey; // giá trị key được lấy trong application.properties

  @Autowired
  private AuthenticationManager authenticationManager;

  @Override
  public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients.inMemory()
      .withClient("client")
      .secret("secret")
      .authorizedGrantTypes("password", "refresh_token")
      .scopes("read");
  }

  @Override
  public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
    endpoints.authenticationManager(authenticationManager)
      .tokenStore(tokenStore()) // config cho token store sử dụng JwtTokenStore
      .accessTokenConverter(jwtAccessTokenConverter()); // cấu hình cho converter
  }

  @Bean
  public TokenStore tokenStore() {
    // toạ 1 token store với acess token converter liên kết với nó
    return new JwtTokenStore(jwtAccessTokenConverter());
  }

  @Bean
  public JwtAccessTokenConverter jwtAccessTokenConverter() {
    var converter = new JwtAccessTokenConverter();
    // đặt giá trị của symmetric key cho access token converter
    converter.setSigningKey(jwtKey);
    return converter;
  }
}
```

- application.properties

```yml
jwt.key=MjWP5L7CiD
```

- Khởi động auth server và gọi đến endpoint oauth/token theo cURL:

```txt
curl -v -XPOST -u client:secret http://localhost:8080/oauth/token?grant_type=password&username=john&password=12345&scope=read
```

- Response body nhận được:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXV…",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6Ikp…",
  "expires_in": 43199,
  "scope": "read",
  "jti": "7774532f-b74b-4e6b-ab16-208c46a19560"
}
```

- Trong response cả access_token và refresh_token đều là JWT, giải mã payload của JWT này ta được JSON sau:

```json
{
  "user_name": "john",
  "scope": ["read"],
  "generatedInZone": "Europe/Bucharest",
  "exp": 1583874061,
  "authorities": ["read"],
  "jti": "38d03577-b6c8-47f5-8c06-d2e3a713d986",
  "client_id": "client"
}
```

#### Triển khai resource server

- resource sử dụng symmetric key đã xác thực token được phát hành bởi auth server
- pom.xml

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
```

- triển khai 1 endpoint để bảo vệ

```java
@RestController
public class HelloController {
  @GetMapping("/hello")
  public String hello() {
    return "Hello!";
  }
}
```

- Triển khai lớp config như với auth server, quan trong nhất là đảm bảo 2 server cùng lưu trữ 1 giá trị key

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

  @Value("${jwt.key}")
  private String jwtKey;

  @Override
  public void configure(ResourceServerSecurityConfigurer resources) {
    resources.tokenStore(tokenStore());
  }

  @Bean
  public TokenStore tokenStore() {
    return new JwtTokenStore(jwtAccessTokenConverter());
  }

  @Bean
  public JwtAccessTokenConverter jwtAccessTokenConverter() {
    var converter = new JwtAccessTokenConverter();
    converter.setSigningKey(jwtKey);
    return converter;
  }
}
```

- Symmetric key chỉ là 1 chuỗi byte random, trong thực tế, nên để 1 chuỗi byte với độ dài là 258
- application.propreties

```yml
server.port=9090
jwt.key=MjWP5L7CiD
```

- Khởi động resource server và gọi đến endpoint /hello, thêm token vào Authorization header của request với tiền tố là Bearer:

```txt
curl -H "Authorization:Bearer eyJhbGciOiJIUzI1NiIs…" http://localhost:9090/hello
```

#### Triển khai mới cho resource server sử dụng spring-security-oauth2-resource-server với JWT và symmetric key

- Cấu hình sử dụng JWTs với method _oauth2ResourceServer()_ được khuyến khích cho các dự án tương lai

```java
@Configuration
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {

  @Value("${jwt.key}")
  private String jwtKey;

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
      .anyRequest().authenticated()
      .and()
      // sử dụng method jwt() của object Customizer, được giử dưới dạng parameter tới method oauth2ResourceServer()
      // jwt() sẽ xác định các chi tiết cần để xác thực token
      .oauth2ResourceServer(c -> c.jwt(
        // vì sử dụng symmetric key => tao 1 JwtDecoder để cung cấp giá trị cho symmetric key
        j -> j.decoder(jwtDecoder());
      ));
  }

  @Bean
  public JwtDecoder jwtDecoder() {
    byte [] key = jwtKey.getBytes();
    SecretKey originalKey = new SecretKeySpec(key, 0, key.length, "AES");
    NimbusJwtDecoder jwtDecoder = NimbusJwtDecoder.withSecretKey(originalKey).build();
    return jwtDecoder;
  }
```

### Sử dụng token được ký bằng asymmetric keys với JWT

- Đôi khi sử dụng chung 1 key được chai sẻ giữa auth server và resourec serve là không thể thực hiện được. Xảy ra khi auth server và resource server được phát triển bởi các tổ chức khác nhau.
- Lúc này auth server không tin tưởng resource server, auth server không muốn chia sẻ key với resource server. Với symmetric key, resource server có quá nhiều quyền lực, không chỉ xác thực token mà còn ký token. Tưởng tượng Hacker lấy được symmetric key, mặc dù không thay đổi được token nhưng có thể sử dụng nó để ký 1 token mới, sử dụng token mới này với quyền admin để thay đổi hoặc đánh cắp tài nguyên
- Khi auth server không thể tin tưởng resource server, nên sử dụng asymmetric key pairs
- 1 asymmetric key pair có 2 key:
  - private key: sử dụng để key token
  - public key: được liên kết bới private key, sử dụng để xác thực signature

#### Tạo asymmetric key pairs

- Sử dung keytool và OpenSSL, là 2 command-line tools. JDK đã cài đặt keytool, OpenSSL thì đi kèm sẵn với Git Bash. Khi có đủ 2 tools, cần thực thi 2 command-line để thực hiện tạo private key và lấy public key cho private key trước đó.
- Các option để tạo private key trong Command Prompt:

```txt
Options:
 -alias <alias>                  alias name of the entry to process
 -keyalg <keyalg>                key algorithm name
 -keysize <keysize>              key bit size
 -sigalg <sigalg>                signature algorithm name
 -destalias <destalias>          destination alias
 -dname <dname>                  distinguished name
 -startdate <startdate>          certificate validity start date/time
 -ext <value>                    X.509 extension
 -validity <valDays>             validity number of days
 -keypass <arg>                  key password
 -keystore <keystore>            keystore name
 -storepass <arg>                keystore password
 -storetype <storetype>          keystore type
 -providername <providername>    provider name
 -providerclass <providerclass>  provider class name
 -providerarg <arg>              provider argument
 -providerpath <pathlist>        provider classpath
 -v                              verbose output
 -protected                      password through protected mechanism
```

- Cung cáp các thông tin tối thiểu sau: lưu trong tệp privatekey.jks, sử dụng password là 123456 để bảo vệ private key, đặt alias name là privatekey để tạo key trong Command Prompt:

```txt
keytool -genkeypair -alias privatekey -keyalg RSA -keypass 123456 -keystore privatekey.jks -storepass 123456
```

- Để thu thập public key, trong giao diện git bash sử dụng command sau:

```txt
keytool -list -rfc --keystore privatekey.jks | openssl x509 -inform pem -pubkey
```

- Cần nhập lại password khi tạo public key, trường hợp này sử dụng pasowrd là 123456. Sau đó sẽ thấy public key trong giao diện dòng lệnh như sau:

```txt
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAmk++9E7hKxDBwqp/c+4L
tXyYvQ9do/NCCqxOxXj/NYNsuQo9Ja4ZcX5pv/5//iryclneTJiMg6Ed5xIxr/0i
WSMgSETSQqa4N3Sy2kW3uHyiZoalv/m3zGcmO5FJ6okDOj+ViBLqcMd/VCMUXLUW
30AexeIjnzbE3oyInsjJcFIvEaxl8N0+z15zAMgRkNT2iC+4BcYRs+tWF6jbreuw
NvBAhVZ1bJTxPT+1OWT3q4XnSFel558EogWWaPFhO/I5oKydW0B4DS20OFZms0+x
Ut2SA3ocFS7pBXDyJbc83EC8QpmN8587PtxcsZwqHUYvlFH5SOzfn3+n/ZuDFc93
VQIDAQAB
-----END PUBLIC KEY-----
```

#### Triển khai auth server sử dụng private key

- Sử dụng cùng 1 file pom.xml như trong ví dụ symmetric key
- Copy file private key: privatekey.jks vào folder resource
- Khai báo filename, alias của private key và pasword sử dụng để bảo vệ private key trong file application.properties. Những chi tiết này cần thiết để cấu hình JwtTokenStore

```yml
password=123456
privateKey=privatekey.jks
alias=privatekey
```

- Cấu hình sử dụng private key chỉ thay đổi với JwtAccessTokenConverter so với cấu hình sử dụng symmetric key

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {

 // inject từ application.properties
  @Value("${password}")
  private String password;
  @Value("${privateKey}")
  private String privateKey;
  @Value("${alias}")
  private String alias;
  @Autowired
  private AuthenticationManager authenticationManager;

  // Omitted code
  @Bean
  public JwtAccessTokenConverter jwtAccessTokenConverter() {
    var converter = new JwtAccessTokenConverter();

    // tạo 1 object KeyStoreKeyFactory để đọc private key từ classpath
    KeyStoreKeyFactory keyStoreKeyFactory = new KeyStoreKeyFactory(
      new ClassPathResource(privateKey), password.toCharArray()
    );

    // sử dụng object KeyStoreKeyFactory để truy xuất đến key pair và đặt key pair vào trong JwtAccessTokenConverter
    converter.setKeyPair(keyStoreKeyFactory.getKeyPair(alias));
    return converter;
  }
```

- Khởi động auth server và gọi đén endpoint /oauth/token với cURL sau:

```txt
curl -v -XPOST -u client:secret "http://localhost:8080/oauth/token?grant_type=password&username=john&passwopa=12345&scope=read"
```

- Response nhận được là

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5…",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJSUzI1NiIsInR…",
  "expires_in": 43199,
  "scope": "read",
  "jti": "8e74dd92-07e3-438a-881a-da06d6cbbe06"
}
```

- Token trong response là JWT bình thường, tuy nhiên để xác thực signature cần public key. Token này chỉ được ký, chưa mã hoá

#### Triển khai resource server sử dụng public key

- application.properties

```yml
server.port=9090
pulicKey=-----BEGIN PUBLIC KEY-----MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAmk++9E7hKxDBwqp/c+4LtXyYvQ9do/NCCqxOxXj/NYNsuQo9Ja4ZcX5pv/5//iryclneTJiMg6Ed5xIxr/0iWSMgSETSQqa4N3Sy2kW3uHyiZoalv/m3zGcmO5FJ6okDOj+ViBLqcMd/VCMUXLUW30AexeIjnzbE3oyInsjJcFIvEaxl8N0+z15zAMgRkNT2iC+4BcYRs+tWF6jbreuwNvBAhVZ1bJTxPT+1OWT3q4XnSFel558EogWWaPFhO/I5oKydW0B4DS20OFZms0+xUt2SA3ocFS7pBXDyJbc83EC8QpmN8587PtxcsZwqHUYvlFH5SOzfn3+n/ZuDFc93VQIDAQAB-----END PUBLIC KEY-----
```

- config sử dụng public key để xác thưc

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

  @Value("${publicKey}")
  private String publicKey;

  @Override
  public void configure(ResourceServerSecurityConfigurer resources) {
    resources.tokenStore(tokenStore());
  }

  @Bean
  public TokenStore tokenStore() {
    return new JwtTokenStore(jwtAccessTokenConverter());
  }

  @Bean
  public JwtAccessTokenConverter jwtAccessTokenConverter() {
    var converter = new JwtAccessTokenConverter();
    converter.setVerifierKey(publicKey);
    return converter;
  }
}
```

- Khởi động resource server vào gọi đến endpoint /hello với cURL như sau:

```txt
curl -H "Authorization:Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6I…" http://localhost:9090/hello
```

#### Triển khai mới cho resource server sử dụng spring-security-oauth2-resource-server với JWT và asymmetric key pairs

- Thay đổi duy nhất so với sử dụng symmetric key là JwtDecoder

```java
public JwtDecoder jwtDecoder() {
  try {
    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    var key = Base64.getDecoder().decode(publicKey);
    var x509 = new X509EncodedKeySpec(key);
    var rsaKey = (RSAPublicKey) keyFactory.generatePublic(x509);
    return NimbusJwtDecoder.withPublicKey(rsaKey).build();
  } catch (Exception e) {
    throw new RuntimeException("Wrong public key");
  }
}
```

- Khi có JwtDecoder sử dụng public key để xác thực token, cần cài đặt bộ giải mã này sử dụng _oauth2ResourceServer()_

```java
@Configuration
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {
  @Value("${publicKey}")
  private String publicKey;

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.oauth2ResourceServer(c -> c.jwt(
      j -> j.decoder(jwtDecoder())
    ));

    http.authorizeRequests()
      .anyRequest().authenticated();
  }
 // Omitted code
}
```

#### Sử dụng endpoint để tiết lộ public key

- Cách làm cho public key được resource server biết đến
- Điều gì xảy ra nếu muốn thay đổi key pair, 1 phương pháp tốt là không nên giữu cùng 1 key pairs mãi mãi, Theo thời gian nen xoay các keys, làm cho hệ thống ít bị đánh cắp key hơn. Vì vậy tạo ra khó khắn khi muốn thay đổi key pairs, cần phải thay đổi ở cả 2 ứng dụng
- Thay vào đó, chỉ thay đổ private key tại auth server và cung cấp 1 endpoint để resource server lấy public key
- Mọi thiết lập với auth server vẫn được giữ nguyên, Spring đã cấu hình 1 endpoint làm lộ ra public key. Theo mặc định endpoint này từ chối mọi request, cần ghi đè lại cấu hình cho endpoint này chấp nhận bất cứ ai với thông tin xác thực của client có thể truy cập nó.

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
      .scopes("read")
      .and()
      // thêm thông tin xác thưc của resource server như 1 client
      .withClient("resourceserver")
      .secret("resourceserversecret");
  }

  // cấu hình cho auth server cung cấp endpoint lấy public key cho mọi reqquets đã được xác thực với thông tin xác thực của client
  @Override
  public void configure(AuthorizationServerSecurityConfigurer security) {
    security.tokenKeyAccess("isAuthenticated()");
  }
}
```

- Khởi độn auth server và gọi đến /oauth/token_key với cURL sau:

```txt
curl -u resourceserver:resourceserversecret http://localhost:8080/oauth/token_key
```

- Response nhận được có dạng như sau:

```json
{
  "alg": "SHA256withRSA",
  "value": "-----BEGIN PUBLIC KEY----- nMIIBIjANBgkq... -----END PUBLIC KEY-----"
}
```

- Với resource server sử dụng endpoint để thu thập public key, cần cấu hình endpoint và thông tin xác thực client trong application.properties như sau:

```yml
server.port=9090
security.oauth2.resource.jwt.key-uri=http://localhost:8080/oauth/token_key
security.oauth2.client.client-id=resourceserver
security.oauth2.client.client-secret=resourceserversecret
```

- Bởi vì resource server lấy public key từ /oauth/token_key của auth server, nên không cần cấu hình nó trong resource server, class cấu hình cho resource server sẽ trống

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
}
```

- Khởi động resource server và gọi đến endpoint /hellovowis cURL như sau:

```txt
curl -H "Authorization:Bearer eyJhbGciOiJSUzI1NiIsInR5cCI…" http://localhost:9090/hello
```

#### Thêm chi tiết tuỷ chỉnh cho JWT được cung cấp bởi auth server

(Resource Server, use symmetric key and asymmetrci key pairs to sign token JWT)

- Trong nhiều trường hợp, không cần nhiều hơn những gì Spring cung cấp, tuy nhiên đôi khi có những yêu cầu cần thêm chi tiết tuỳ chỉnh vào token
- Nếu giải mã 1 token mặc định do Spring triển khai sẽ nhận được:

```json
{
  "exp": 1582581543, // tiémtamp khi token hết hạn
  "user_name": "john", // user đã xác thực để cho phép client truy cập tới tài nguyên của họ
  "authorities": ["read"], // quyền được cấp của user
  "jti": "8e208653-79cf-45dd-a702-f6b694b417e7", // 1 chuỗi định danh duy nhất của token
  "client_id": "client", // client đã yêu cầu token
  "scope": ["read"] // quyền được cấp cho client
}
```

- Token sẽ lưu trữ tất cả chi tiết cần thiết cho xác thực bằng HTTP Basic
- Triển khai cấu hình thêm time zone của auth server vào token. Để thực hiện cần tạo 1 object type TokenEnhancer

```java
public class CustomTokenEnhancer implements TokenEnhancer {
  // thực thi interface TokenEnhancer

 // method enhance() nhận được token hiện tại và trả về token đã được tăng cường
  @Override
  public OAuth2AccessToken enhance(OAuth2AccessToken oAuth2AccessToken,
    OAuth2Authentication oAuth2Authentication) {

    // tạo 1 token dựa trên token đã nhận
    var token = new DefaultOAuth2AccessToken(oAuth2AccessToken);

    // khai báo 1 map, chứa các chi tiết muốn thêm vào token
    Map<String, Object> info =
      Map.of("generatedInZone", ZoneId.systemDefault().toString());

    // thêm chi tiết vào token
    token.setAdditionalInformation(info);

    // trả về token đã được thêm chi tiết
    return token;
  }
}
```

- Với file cấu hình cho auth server, sử dụng cùng 1 cấu hình như trên, chỉ thay đổi method configure() áp dụng token đã được tăng cường

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
  // Omitted code

  @Override
  public void configure(AuthorizationServerEndpointsConfigurer endpoints) {

 // xác định TokenEnhancerChain
  TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();

  // thêm 2 token được tăng cường vào list
  var tokenEnhancers =
    List.of(new CustomTokenEnhancer(), jwtAccessTokenConverter());

  // đưa list vào chain
  tokenEnhancerChain.setTokenEnhancers(tokenEnhancers);

  // cấu hình sử dụng tokenEnhancerChain
  endpoints.authenticationManager(authenticationManager)
    .tokenStore(tokenStore())
    .tokenEnhancer(tokenEnhancerChain);
  }
}
```

- Việc cấu hình token tuỳ chỉnh có hơi phức tạp. Chúng ta phải tạo 1 chain của token enhancer và đặt toàn bộ chain thay thế cho chỉ 1 object. Bởi vì object access token converter cũng là 1 token enhancer. Nếu chỉ cấu hình token enhancer tuỳ chỉnh, phải ghi đè lại hành vi của access token converter. Thay vào đó, chỉ cần thêm cả 2 vào 1 chain, và cấu hình cho chain chứa cả 2 object.

- Khởi động auth server và gọi đến endpoint /oauth/token với cURL sau:

```txt
curl -v -XPOST -u client:secret "http://localhost:8080/oauth/token?grant_type=password&username=john&password=12345&scope=read"
```

- response nhận được:

```json
{
  "access_token": "eyJhbGciOiJSUzI…",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJSUzI1…",
  "expires_in": 43199,
  "scope": "read",
  "generatedInZone": "Europe/Bucharest",
  "jti": "0c39ace4-4991-40a2-80ad-e9fdeb14f9ec"
}
```

- giải mã phần payload của token

```json
{
  "user_name": "john",
  "scope": ["read"],
  "generatedInZone": "Europe/Bucharest",
  "exp": 1582591525,
  "authorities": ["read"],
  "jti": "0c39ace4-4991-40a2-80ad-e9fdeb14f9ec",
  "client_id": "client"
}
```

#### Cấu hình resource server để đọc chi tiết tuỳ chỉnh của 1 JWT

- AccessTokenConverter là 1 object chuyển token thành 1 Authentication. Đây là 1 object cần thay đổi để có thể xem các chi tiết tuỳ chỉnh trong token.
- Trước đấy, chúng ta tạo 1 bean loại JwtAccessTokenConverter, sử dụng token này để set key được resource server sử dụng để xác thưc token.
- Tạo triển khai tuỳ chỉnh của JwtAccessTokenConverter, xem xét các chi tiết mới về token. Cách đơn giản nhất là mở rộng và ghi đè method extractAuthentication(). Method này chuyển token thành Authentication object

```java
public class AdditionalClaimsAccessTokenConverter extends JwtAccessTokenConverter {

  @Override
  public OAuth2Authentication extractAuthentication(Map<String, ?> map) {
    // áp dụng logic được triển khai bởi JwtAccessTokenConverter class và lấy object authentication ban đầu
    var authentication = super.extractAuthentication(map);
    // thêm chi tiết tuỳ chỉnh vào authentication
    authentication.setDetails(map);
    // trả lại authentication object
    return authentication;
  }
}
```

- Trong class cấu hình của resource server, co thể sử dụng access token converter tuỳ chỉnh

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
  // Omitted code
  @Bean
  public JwtAccessTokenConverter jwtAccessTokenConverter() {
    var converter = new AdditionalClaimsAccessTokenConverter();
    converter.setVerifierKey(publicKey);
    return converter;
  }
}
```

- Đưa Authentication được AdditionalClaimsAccessTokenConverter chuyển đổi từ token vào controllẻ để kiểm tra

```java
@RestController
public class HelloController {
  @GetMapping("/hello")
  public String hello(OAuth2Authentication authentication) {
    // lấy các chi tiết bổ sung được thêm vào object authentication
    OAuth2AuthenticationDetails details =
      (OAuth2AuthenticationDetails) authentication.getDetails();
    return "Hello! " + details.getDecodedDetails();
 }
}
```

- Method getDecodedDetails() trả về 1 map chứa chi tiết của token
- Khởi động resource server và gọi đến endpoint /hello theo cURL sau:

```txt
 curl -H "Authorization:Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp… " http://localhost:9090/hello
```

- Response trả về:

```txt
Hello! {user_name=john, scope=[read], generatedInZone=Europe/Bucharest,
 exp=1582595692, authorities=[read], jti=982b02be-d185-48de-a4d3-
 9b27337d1a46, client_id=client}
```

# Spring Security OAuth 2 Authorization Server version 1.0 cho Spring Boot 3 (2023)

1. Thêm dependency:

- spring-security-oauth2-authorization-server (trên maven story)
- Spring Web
- Spring Security

```xml
<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-oauth2-authorization-server</artifactId>
		<version>1.0.0</version>
</dependency>
```

2. Tạo file cấu hình SecurityConfig.java

```java
@Configuration
public class SecurityConfig {

  // do Spring chưa đưa OAuth2AuthorizationServer thành 1 phần chính thức của Spring Security Oauth
  // => cần phải config thông qua bean SecurityFilterChain bằng cách gọi method static applyDefaultSecurity của class OAuth2AuthorizationServerConfiguration và truyền HttpSecurity làm tham số

  // đặt config Authorization Server lên Order(1)
  @Bean
  @Order(1)
  public SecurityFilterChain asSecurityFilterChain(HttpSecurity http) throws Exception {
    OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);

    // OpenID connect 1.0 bị tắt trong config mặc định
    // bật OpenId Connect 1.0 thông qua khởi tạo OidcConfigurer
    http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
      // lấy ra configurer để thay đổi cấu hình mặc định của OAuth2AuthorizationServerConfigurer
      .oidc(Customizer.withDefaults());
      // khởi tạo OidcConfigurer

    // chuyển hướng tới login page khi chưa được xác thực từ authorization endpoint
    // đảm bảo các user chưa xác thực khi gọi đến bất kỳ authorization endpoint này cũng chuyển hướng đến /login
    http.exceptionHandling(
        e -> e.authenticationEntryPoint(
            new LoginUrlAuthenticationEntryPoint("/login")
        )
    );`

    return http.build();
  }

  // config thứ 2 của SecurityFilterChain chp app
  @Bean
  @Order(2)
  public SecurityFilterChain appSecurityFilterChain(HttpSecurity http) throws Exception {
    // config sử dung form login để client chuyển hướng user tới AuthServer sử dụng
    http.formLogin()
        .and()
        .authorizeHttpRequests().anyRequest().authenticated();

    return http.build();
  }

  @Bean
  public UserDetailsService userDetailsService() {
    var u1 = User.withUsername("user")
        .password("password")
        .authorities("read")
        .build();

    return new InMemoryUserDetailsManager(u1);
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }

  // đăng ký client với auth server
  // triển khai quản lý client
  @Bean
  public RegisteredClientRepository registeredClientRepository() {
    RegisteredClient r1 = RegisteredClient.withId(UUID.randomUUID().toString())
        .clientId("client")
        .clientSecret("secret")
        /// vì cấu hình sử dụng OpenID nên có thể sử dụng bất kỳ scope nào cung cấp bởi OidcScopes
        .scope(OidcScopes.OPENID)
        .scope(OidcScopes.PROFILE)
        // có thể cấu hihf nhiều redirect url
        // khi client chuyển hướng user tới auth server sẽ thêm parameter redirect_url trong request để auth server chọn đúng redirect_url đã config
        // auth server cần biết về redirect_url để chắc chắn redirect_url mà client cung cấp không bị ai đó thay đổi
        .redirectUri("https://dummy.url/authorized")
        // sử dụng phương thức xác thực HTTP Basic (giử token được mã hoá theo Base64 trong header để xác thực client)
        .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
        // grant type là authorization code
        .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
        // sử dụng thêm request token
        .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
        .build();

    return new InMemoryRegisteredClientRepository(r1);
  }

  // AuthorizationServerSettings chứa các config cài đặt cho OAuth2 Auth server, chỉ định cho các protocol endpoint
  /* các URI được đặt mặc định như sau:
   public final class AuthorizationServerSettings extends AbstractSettings {...
	  public static Builder builder() {
		  return new Builder()
			  .authorizationEndpoint("/oauth2/authorize")
			  .tokenEndpoint("/oauth2/token")
			  .tokenIntrospectionEndpoint("/oauth2/introspect")
			  .tokenRevocationEndpoint("/oauth2/revoke")
			  .jwkSetEndpoint("/oauth2/jwks")
			  .oidcUserInfoEndpoint("/userinfo")
			  .oidcClientRegistrationEndpoint("/connect/register");
	  }
    ...
  }*/
  // AuthorizationServerSettings là 1 thành phần bắt buộc (thay thế ProviderSettings ở version 0.3)
  // OAuth2AuthorizationServerConfigurer tự động đăng ký AuthorizationServerSettings @Bean, không cần cung cấp
  // AuthorizationServerSettings thiết lập tất cả các endpoint cho Auth Server
  // xem các endpoint được cung cấp bằng cách gọi tới: GET  /.wellknown/openid-configuration
  @Bean
  public AuthorizationServerSettings authorizationServerSettings() {
    return AuthorizationServerSettings.builder()
      .build();
  }

  // bật OpenID Connext mặc đinh đã cấu hình sử dụng jwt để mã hoá toekn_id
  // cần cung cấp bộ jwk để sign token => tạo ra token
  @Bean
  public JWKSource<SecurityContext> jwkSource() throws Exception {
    KeyPair keyPair = generateRsaKey();
		RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
		RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();

    RSAKey rsaKey = new RSAKey.Builder(publicKey)
        .privateKey(privateKey)
        .keyID(UUID.randomUUID().toString())
        .build();

    JWKSet jwkSet = new JWKSet(key);
    return new ImmutableJWKSet(jwkSet);
  }

  private static KeyPair generateRsaKey() {
		KeyPair keyPair;
		try {
			KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
			keyPairGenerator.initialize(2048);
			keyPair = keyPairGenerator.generateKeyPair();
		}
		catch (Exception ex) {
			throw new IllegalStateException(ex);
		}
		return keyPair;
	}

  /*
  // tạo bean để cung cấp bộ giải mã cho Auth server xác thực token
  // bộ giải mã này là bắt buộc khi sử dụng scope là UserInfo hay Client Registration để resource server lấy thông tin về id_token
  // mặc định Spring đã tự động triển khai 1 cái bên resource server
  // bên auth server thì chắc là đ cần tạo @Bean này ha, mà sao trong doc thấy tạo
  @Bean
	public JwtDecoder jwtDecoder(JWKSource<SecurityContext> jwkSource) {
		return OAuth2AuthorizationServerConfiguration.jwtDecoder(jwkSource);
	}
  */

}
```

3. Urls client giử đến authorization server:

- Client chuyển hướng user tới authorization server theo url:

```txt
http://localhost:8080/oauth2/authorize?response_type=code&client_id=client&scope=openid&redirect_uri=https://springone.io/authorized&code_challenge=QYPAZ5NU8yvtlQ9erXrUYR-T5AGCjCF47vN-KsaI2A8&code_challenge_method=S256
```

- Client nhận authorization_code, gọi đến authorization server thu thập access_token:

```txt
http://localhost:8080/oauth2/token?client_id=client&redirect_uri=https://springone.io/authorized&grant_type=authorization_code&code=dWlJMGpGlUAPz0sRU1y8suXDyWejo0_B4-WrLP-ks5kSlcdvlGG-u1OxOORvvpm7IMJaC_lMqzTX2Oh6AKHGOb2J4-Hp6PVPvGjLeUQMnWzz6h3Xyy1D0S6czbiTeU8f&code_verifier=qPsH306-ZDDaOE8DFzVn05TkN3ZZoVmI_6x4LsVglQI
```

### Tuỳ chỉnh cấu hình Spring Security OAuth 2 Authorization Server

#### tuỳ chỉnh claims cho token

- Thêm @Bean OAuth2TokenCustomizer cho lớp config

```java
@Bean
public OAuth2TokenCustomizer<JwtEncodingContext> oAuth2TokenCustomizer() {
  return  context -> {
    context.getClaims().claim("test", "test");
  };
}
```

#### tuỳ chỉnh lại redirect uri validator

- mặc định uri redirect của client cung cấp không chấp nhận local host
- viết lại logic của redirect uri
  - logic chỉ kiểm tra redirect_uri cung cấp trong request khi client chuyển hướng user tới auth server để login
  - chỉ chấp nhận redirect_uri giống redirect_uri đã cung cấp khi đăng kí client cho auth server
  - không có logic khác => loại bỏ việc mặc định không chấp nhận gọi đến từ localhost

```java
// tạo class CustomRedirectUriValidator
public class CustomRedirectUriValidator implements Consumer<OAuth2AuthorizationCodeRequestAuthenticationContext> {

  @Override
  public void accept(OAuth2AuthorizationCodeRequestAuthenticationContext context) {
    OAuth2AuthorizationCodeRequestAuthenticationToken a = context.getAuthentication();
    // lấy các client đã đăng ký trong context
    RegisteredClient registeredClient = context.getRegisteredClient();
    String uri = a.getRedirectUri();

    if (!registeredClient.getRedirectUris().contains(uri)) {
      var error = new OAuth2Error(OAuth2ErrorCodes.INVALID_REQUEST);
      throw new OAuth2AuthorizationCodeRequestAuthenticationException(error, null);
    }
  }
}
/***************************************************************************************/
// trong class config

// đưa CustomRedirectUriValidator vừa tạo vào authorization endpoint provider
private Consumer<List<AuthenticationProvider>> getAuthorizationEndpointProviders() {
  return providers -> {
    for (AuthenticationProvider p : providers) {
      if (p instanceof OAuth2AuthorizationCodeRequestAuthenticationProvider x) {
        x.setAuthenticationValidator(new CustomRedirectUriValidator());
      }
    }
  };
}

@Bean
@Order(1)
public SecurityFilterChain asSecurityFilterChain(HttpSecurity http) throws Exception {
  OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);

  http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
      .authorizationEndpoint(
        // tuỳ chỉnh cung cấp lại authorization endpoint provider
          a -> a.authenticationProviders(getAuthorizationEndpointProviders())
      )
      .oidc(Customizer.withDefaults());

  http.exceptionHandling(
      e -> e.authenticationEntryPoint(
          new LoginUrlAuthenticationEntryPoint("/login")
      )
  );

  return http.build();
}
```

#### tuỳ chỉnh sử dụng loại token sử dụng để xác thưc (không nên sử dụng, JWT vẫn là tốt nhất)

- Có 2 loại token chính:
  - opaque: token không chứa data
  - non-opaque: token chứa data bên trong (như jwt)
- Tuỳ chỉnh bằng cách gọi method tokenSettings khi tạo ra 1 RegisteredClient

```java
@Bean
public RegisteredClientRepository registeredClientRepository() {
  RegisteredClient r1 = RegisteredClient.withId(UUID.randomUUID().toString())
      .clientId("client")
      .clientSecret("secret")
      .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
      .scope(OidcScopes.OPENID)
      .scope(OidcScopes.PROFILE)
      .redirectUri("https://springone.io/authorized")
      .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
      .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
      .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
      .tokenSettings(
          TokenSettings.builder()
              // chuyển access_tokrn về loại REFERENCE. chính là opaque token, không chứa nội data
              // mặc định là loại SELF_CONTAINED, chính là non-opaque token
              // khi nhận được 1 opaque token, client sử tụng endpoint: /oauth2/introspect?token=<access_token> method POST, kiểu xác thực Basic auth, cung cấp client Id và client secrect để lấy được thông tin về user phát hành token
              .accessTokenFormat(OAuth2TokenFormat.REFERENCE)
              // tuỳ chỉnh thời gian tồn tại của token là 9000s (cả id_token và access_token)
              .accessTokenTimeToLive(Duration.ofSeconds(900))
              .build()
      )
      .build();

  return new InMemoryRegisteredClientRepository(r1);
}
```

#### ghi đè lại RegisteredClientRepository, sử dụng db để lưu trữ client

- table: clients
  - id
  - client_id
  - secrect
  - scope
  - auth method
  - grant_type
  - redirect_url
- table: users

  - id
  - username
  - password
  - authority

- tạo entity, repo

```java
public interface ClientRepository extends JpaRepository<Client, Integer> {

  @Query("SELECT c FROM Client c WHERE c.clientId = :clientId")
  Optional<Client> findByClientId(String clientId);

}

@Entity
@Table(name = "clients")
@Data
public class Client {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private int id;

  private String clientId;
  private String secret;
  private String redirectUri;
  private String scope;
  private String authMethod;
  private String grantType;

  public static Client from(RegisteredClient registeredClient) {
    Client client = new Client();

    client.setClientId(registeredClient.getClientId());
    client.setSecret(registeredClient.getClientSecret());

    client.setRedirectUri(    // NOT CLEAN CODE
      registeredClient.getRedirectUris().stream().findAny().get()
    );
    client.setScope(
      registeredClient.getScopes().stream().findAny().get()
    );
    client.setAuthMethod(
      registeredClient.getClientAuthenticationMethods().stream().findAny().get().getValue()
    );
    client.setGrantType(
      registeredClient.getAuthorizationGrantTypes().stream().findAny().get().getValue()
    );

    return client;
  }

  public static RegisteredClient from(Client client) {
    return RegisteredClient.withId(String.valueOf(client.getId()))
      .clientId(client.getClientId())
      .clientSecret(client.getSecret())
      .scope(client.getScope())
      .redirectUri(client.getRedirectUri())
      .clientAuthenticationMethod(new ClientAuthenticationMethod(client.getAuthMethod()))
      .authorizationGrantType(new AuthorizationGrantType(client.getGrantType()))
      .build();
  }
}
```

- tạo service cho client

```java
@Service
@Transactional
public class CustomClientService implements RegisteredClientRepository {

  private final ClientRepository clientRepository;

  public CustomClientService(ClientRepository clientRepository) {
    this.clientRepository = clientRepository;
  }

  @Override
  public void save(RegisteredClient registeredClient) {
    clientRepository.save(Client.from(registeredClient));
  }

  @Override
  public RegisteredClient findById(String id) {
    var client = clientRepository.findById(Integer.valueOf(id))
      .orElseThrow();
    return Client.from(client);
  }

  @Override
  public RegisteredClient findByClientId(String clientId) {
    var client = clientRepository.findByClientId(clientId)
      .orElseThrow();
    return Client.from(client);
  }
}
```

- sử dụng service thay thế cho @Bean RegisteredClientRepository, xoá @Bean RegisteredClientRepository trong lớp config, để Spring tự tìm annotation @Service và tạo bean trong context

# Spring Security OAuth 2 Resource Server cho Spring Boot 3 (2023)

1. Thêm dependencies:

- Sping Web
- Spring Security

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
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

2. Tạo class SecurityConfig

```java
@Configuration
public class SecurityConfig {

  @Value("${jwksUri}")
  private String jwksUri; // jwksUri=http://localhost:8080/oauth2/jwks

  @Bean
  public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

    // method .oauth2ResourceServer() để làm cho app hoạt động như 1 resource server
    http.oauth2ResourceServer(
      // cấu hình cho resource server sử dụng jwt để xác thực
      r -> r.jwt()
        // khai báo 1 uri, cung cấp cho resource server truy cấp lấy public key
        .jwkSetUri(jwksUri)
    );

    // các request cần được xác thực
    http.authorizeHttpRequests().anyRequest().authenticated();
    return http.build();
  }

  /*
  // Spring tự cung cấp 1 bean JwtDecoder để giải mã token tuy nhiên cũng có thể tuỳ chỉnh sử dụng bean sau
  // có thể gọi method sau trên bean SecurityFilterChain cũng tương tự:
  //.oauth2ResourceServer(oauth2 -> oauth2.jwt(jwt -> jwt.decoder(NimbusJwtDecoder.withJwkSetUri(jwkSetUri).build())));
  @Bean
  public JwtDecoder jwtDecoder() {
    return JwtDecoders.fromIssuerLocation(issuerUri);
  }
  */
}
```

3. Khởi động resource server, gọi đến 1 endpoint bất kỳ được bảo mật

- sử dụng access_token thu thập ở auth server
- đưa access_token vào request với param là Authntication, value là chuỗi access_token đi kèm với tiền tố "Bearer "

### Sử dụng acess_token chuyển đổi về object Authentication

- Spring sẽ inject Authentication như 1 tham số truyền vào method trong Controller (cũng có thể lấy ra Authentication bằng các sử dụng SecurityContextHolder)

```java
@RestController
public class DemoController {

  @GetMapping("/demo")
  public String demo(Authentication authentication) {
    // authentication được inject là loại JwtAuthenticationToken
    // JwtAuthenticationToken chứa username, token, authorities
    // mặc định authorities trả về giá trị "SCOPE_openid" khi khai báo client sử dụng Scope là openid
    // không phải là các roles hay authorities mà resource sử dụng để cho phép client truy cập tài nguyên
    return "Demo!";
  }
}
```

- Thay đổi auth server để thêm claim về authority của user trong access_token
  - thêm @Bean OAuth2TokenCustomizer<JwtEncodingContext> vào class config của auth server:

```java
@Bean
public OAuth2TokenCustomizer<JwtEncodingContext> oAuth2TokenCustomizer() {
  return  context -> {
    var authorities = context.getPrincipal().getAuthorities()
    // authorities là 1 list GrantedAuthority, chuyển đổi nó thành 1 list String chứa tên của authority
    context.getClaims().claim("authorities", authorities.stream().map(a -> a.getAuthorities()).toList());
  };
}
```

- Access_token nhận được từ auth server, trong phần payload sẽ chứa claim authorities là 1 mảng String tên các authorities của user
- Ở resource server, Spring mặc định chuyển accsee_token nhận được thành 1 object authentication với type là JwtAuthenticationToken
- Object JwtAuthenticationToken chứa username, token, authorities ... Tuy nhiên authorities nhận được là scope caung cấp khi đăng ký client cho auth serer (object JwtAuthenticationToken được inject trong class Controller bên trên có authorities là "SCOPE_openid")
- Cần custom lại class Converter để thay đổi logic chuyển đổi token thành 1 Authentication chứa authoriries là các roles hay authorities của user login
- Class config

```java
@Configuration
public class SecurityConfig {

  @Value("${jwksUri}")
  private String jwksUri;

  @Bean
  public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.oauth2ResourceServer(
      r -> r.jwt().jwkSetUri(jwksUri)
        // method .jwtAuthenticationConverter() chuyển đổi bất cứ jwt token nào thành object JwtAuthenticationToken
        // cung cấp 1 converter, thực hiện việc chuyển đổi thánh Authentication có chứa authoriries như mong muốn
        .jwtAuthenticationConverter(new CustomJwtAuthenticationTokenConverter())
    );

    http.authorizeHttpRequests().anyRequest().authenticated();
    return http.build();
  }
}
```

- Class converter

```java
public class CustomJwtAuthenticationTokenConverter implements Converter<Jwt, CustomJwtAuthenticationToken> {


  @Override
  public CustomJwtAuthenticationToken convert(Jwt source) {
    List<String> authorities = (List<String>) source.getClaims().get("authorities");
    // class JwtAuthenticationToken do Spring cung cấp có hàm contructor nhận vào 2 tham số: JWT và Collection<? extends GrantedAuthority>
    // mặc định Spring sử dụng constructor chỉ nhận JWT làm tham số và tự tạo authorities dựa trên JWT
    // sử dụng hàm constructor có cung cấp Collection<? extends GrantedAuthority> để tạo ra JwtAuthenticationToken có authorities tuỳ chỉnh
    JwtAuthenticationToken authenticationObj =
        new JwtAuthenticationToken(source, authorities.stream().map(SimpleGrantedAuthority::new).toList());

    return authenticationObj;
  }
}
```

=======

```
>>>>>>> 13455a2 (Resource Server, use symmetric key and asymmetrci key pairs to sign token JWT)
```
