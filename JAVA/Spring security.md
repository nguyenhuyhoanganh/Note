# Các thành phần của Spring security
* **AuthenticationFilter**: chặn các req giử đến
* **AuthenticationManager**: nhận trách nhiệm xác thực user
* **AuthenticationProvider**: manager sử dụng để thực hiện logic xác thực
   - *UserdetailsService*: sử dụng để tìm kiếm user
   - *PasswordEncoder*: sử dụng để xác thực mật khẩu
* **SecurityContext**: sử dụng để lưu trữ thông tin về thực thể được xác thực

### Ex: ghi đè cấu hình UserDetailsService và PasswordEncoder
 - ghi đè lại triển khai UserDetailsService bằng InMemoryUserDetailsManager
 - tạo 1 user trong hệ thống
 - thêm user được quản lý bởi triển khai InMemoryUserDetailsManager 
 - xác định 1 bean PasswordEncoder để xác minh mật khẩu với mật khẩu user

```java
@Configuration 
public class ProjectConfig extends WebSecurityConfigurerAdapter {
    // WebSecurityConfigurerAdapter cho phép ghi đè lại các phương thức cấu hình của spring security
    @Bean 
    public UserDetailsService userDetailsService() {
        var userDetailsService = new InMemoryUserDetailsManager(); 
        // InMemoryUserDetailsManager là triển khai userDetailsService lưu user trong bộ nhớ
        var user = User.withUsername("john") 
            .password("12345") 
            .authorities("read") 
            .build(); 
        // tạo user

        userDetailsService.createUser(user);
        // thêm user được quản lý bởi userDetailsService
        return userDetailsService;
    }

    @Bean 
    public PasswordEncoder passwordEncoder() { 
        return NoOpPasswordEncoder.getInstance(); 
        // NoOpPasswordEncoder là 1 triển khai không mã hoá mật khẩu của PasswordEncoder
    }

    // cấu hình endpoint giống mặc định
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic();  
        // mặc định các endpoint sử dụng xác thực bằng HTTP Basic
        http.authorizeRequests() 
            .anyRequest().authenticated(); 
        // tất cả các req yêu cầu xác thực
    }  
}
```

### Ex: ghi đè phương thức configure(AuthenticationManagerBuilder auth) được cung cấp bởi WebSecurityConfigurerAdapter
  - cấu hình trực tiếp UserDetailsService và PasswordEncoder trong phương thức được cung cấp sẵn bởi WebSecurityConfigurerAdapter

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        var userDetailsService = new InMemoryUserDetailsManager(); 
        var user = User.withUsername("john") 
            .password("12345") 
            .authorities("read") 
            .build(); 
        userDetailsService.createUser(user); 
        
        auth.userDetailsService(userDetailsService) 
            .passwordEncoder(NoOpPasswordEncoder.getInstance()); 
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic();
        http.authorizeRequests() 
            .anyRequest().authenticated(); 
    }
}
```

### Ex: thực thi interface AuthenticationProvider
 - không thực thi cấu hình UserDetailsService và PasswordEncoder
 - tạo CustomAuthenticationProvider implements AuthenticationProvider
 - thực thi logic xác thực riêng trong method authenticate
 - đăng ký CustomAuthenticationProvider với AuthenticationManager

```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        // authentication logic here
        String username = authentication.getName(); 
        String password = String.valueOf(authentication.getCredentials());
        if ("john".equals(username) && "12345".equals(password)) { 
            return new 
                UsernamePasswordAuthenticationToken(username, password, Arrays.asList());
        } else {
            throw new 
                AuthenticationCredentialsNotFoundException("Error in authentication!");
        }
        // if-else thay thể cho triển khai về UserDetailsService và PasswordEncoder
    }
    // phương thức authenticate đại diện cho tất cả logic xác thực

    @Override
    public boolean supports(Class<?> authenticationType) {
        // type of the Authentication implementation here
    }
}

@Configuration
    public class ProjectConfig extends WebSecurityConfigurerAdapter {
        @Autowired
        private CustomAuthenticationProvider authenticationProvider;
        
        @Override
        protected void configure(AuthenticationManagerBuilder auth) {
            auth.authenticationProvider(authenticationProvider);
            // đăng ký authenticationProvider vừa tạo với AuthenticationManager
        }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic();
        http.authorizeRequests().anyRequest().authenticated();
    }
}
```


# UserDetailsService
* **UserDetails**:  mô tả user trong spring security
* **GrantedAuthority**: xác định hành động user có thể thực thi
* **UserDetailsService**: truy xuất user bằng username
* **UserDetailsManager**: mở rộng của UserDetailsService, thêm các mô tả như tạo mới, thay đổi, xoá user

### interface UserDetails
```java
public interface UserDetails extends Serializable {
    String getUsername(); 
    String getPassword();
    // trả về thông tin xác thực của user
    Collection<? extends GrantedAuthority> getAuthorities(); 
    // trả về hành động mà ứng dụng cho phép user thực hiện
    // là 1 collection, instance của GrantedAuthority
    boolean isAccountNonExpired(); 
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
    // 4 method cho phép bật hoặc tắt tài khoản
    // trả về false sẽ kích hoạt khoá tài khoản
}
```

### interface GrantedAuthority và triển khai SimpleGrantedAuthority có sẵn của Spring security
```java
public interface GrantedAuthority extends Serializable {
    String getAuthority();
    // trả về tên của các authority dưới dạng String
}

// class SimpleGrantedAuthority cung cấp cách để tạo instance GrantedAuthority bất biến 
GrantedAuthority g1 = () -> "READ"; // sử dụng  lambda expression
GrantedAuthority g2 = new SimpleGrantedAuthority("READ"); 
```

### Ex: tạo class SecurityUser thực thi interface UserDetails
```java
public class User {

    @Id
    private int id;
    private String username;
    private String password;
    private String authority;

    // Omitted getters and setters
} 

public class SecurityUser implements UserDetails {
 
    private final User user;
 
    public SecurityUser(User user) {
        this.user = user;
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(() -> user.getAuthority());
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
 
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

### Ex: tạo 1 instane user, không sử dụng class custom từ interface UserDetails
```java
// sử dụng class User từ package  org.springframework.security.core.userdetails
UserDetails u = User.withUsername("bill")
    .password("12345")
    .authorities("read", "write")
    .accountExpired(false)
    .disabled(true)
    .build();

// sử dụng class UserBuilder nested trong class User
User.UserBuilder builder1 = User.withUsername("bill"); 
// User.withUsername(String username) trả về 1 instance của UserBuilder
UserDetails u1 = builder1
    .password("12345")
    .authorities("read", "write")
    .passwordEncoder(p -> encode(p)) 
    // password encoder chỉ có 1 Function<String, String> với trách nhiệm chuyển
    // password sang 1 mã hoá nhất định, không giống như bean PasswordEncoder
    .accountExpired(false)
    .disabled(true)
    .build(); 

// tạo 1 user khác từ 1 instance của UserDetails sẵn có
User.UserBuilder builder2 = User.withUserDetails(u); 
UserDetails u2 = builder2.build()
```

### interface UserDetailsService
```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
// AuthenticationProvider sử dụng UserDetailsService để tải thông tin chi tiết user
```

### Ex: tạo class InMemoryUserDetailsService thực thi interface UserDetailsService
 - sử dụng UserDetails là class SecurityUser được tạo ở ví dụ trên
 - đăng ký 1 bean InMemoryUserDetailsService ghi đè UserDetailsService mặc định của Spring security
```java
public class InMemoryUserDetailsService implements UserDetailsService {
    private final List<UserDetails> users; 

    public InMemoryUserDetailsService(List<UserDetails> users) {
        this.users = users;
    }

    @Override
    public UserDetails loadUserByUsername(String username) 
        throws UsernameNotFoundException {
        return users.stream()
            .filter(u -> u.getUsername().equals(username)) 
            .findFirst() 
            .orElseThrow(() -> new UsernameNotFoundException("User not found")); 
    }
}

@Configuration
public class ProjectConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails u = new User("john", "12345", "read");
        List<UserDetails> users = List.of(u);
        return new InMemoryUserDetailsService(users);
    }
    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```

### interface UserDetailsManager mở rộng của UserDetailsService
 - sử dụng để cung cấp thêm các phương thức thêm, sửa, xoá người dùng ngoài phương thức loadUserByUsername có sẵn

```java
public interface UserDetailsManager extends UserDetailsService {
    void createUser(UserDetails user);
    void updateUser(UserDetails user);
    void deleteUser(String username);
    void changePassword(String oldPassword, String newPassword);
    boolean userExists(String username);
}
// InMemoryUserDetailsManager là 1 triển khai UserDetailsManager có sẵn do spring security cung cấp
```
# PasswordEncoder

### interface PasswordEncoder
```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodedPassword);
    default boolean upgradeEncoding(String encodedPassword) { 
        return false; 
        // ghi đè lại thành true sẽ mã hoá mật khẩu 2 lần
    }
}
```
### Các triển khai PasswordEncoder được Spring Security cung cấp sẵn
* NoOpPasswordEncoder: không mã hoá mật khẩu
* StandardPasswordEncoder: sử dụng SHA-256 để băm mật khẩu (không đủ mạnh)
* Pbkdf2PasswordEncoder: sử dụng PBKDF2
* BCryptPasswordEncoder: sử dụng hàm băm bcrypt để mã hoá mật khẩu
* SCryptPasswordEncoder: sử dụng hàm băm scrypt để mã hoá mật khẩu

### DelegatingPasswordEncoder sử dụng nhiều chiến lược mã hoá mật khẩu
```java
@Configuration
public class ProjectConfig {
    // Omitted code
    @Bean
    public PasswordEncoder passwordEncoder() {
        Map<String, PasswordEncoder> encoders = new HashMap<>();
        encoders.put("noop", NoOpPasswordEncoder.getInstance());
        encoders.put("bcrypt", new BCryptPasswordEncoder());
        encoders.put("scrypt", new SCryptPasswordEncoder());
        //  mật khẩu có tiền tố bcrypt, suỷ quyền cho triển khai BCryptPasswordEncoder để mã hoá mật khẩu
        return new DelegatingPasswordEncoder("bcrypt", encoders);
        // mặc định sử dụng BCryptPasswordEncoder
    }
}
```

# AuthenticationProvider

### Có nhiều kịch bản xác thực ứng dụng
- sử dụng username, password
- sử dụng vân tay
- xác thực bằng sms
...
=> sử dụng AuthenticationProvider để xác định bất cứ logic xác thực tuỳ chỉnh nào

### interface Authentication và Pricipal
- Authentication đại diện cho sự kiện yêu cầu xác thực và giữ thông tin chi tiết của 1 thực thể yêu cầu quyền truy cập vào ứng dụng
- Pricipal đại diện cho người dùng đang yêu cầu truy cập vào ứng dụng, nắm giữ username

```java
public interface Principal {
    String getNamea();
}

public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    // trả về 1 collection của GrantedAuthority cho yếu cầu xác thực
    Object getCredentials();
    // trả về password hoặc bất kỳ secret được sử dụng trong quá trình xác thưc
    Object getDetails();
    Object getPrincipal();
    boolean isAuthenticated();
    // trả về true nếu quá trình xác thực kết thúc, false nếu vấn đang diễn ra
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

### Triển khai logic authentication tuỳ chỉnh
* interface AuthenticationProvider
```java
public interface AuthenticationProvider {
    Authentication authenticate(Authentication authentication) throws           
        AuthenticationException;
    boolean supports(Class<?> authentication);
}
```
 - phương thức authenticate:
   - triển khai phương thức để xác định logic xác thực
   - nhận 1 đối tượng Authentication, nếu đối tượng Authentication nhận vào không được hỗ trợ bởi triển khai AuthenticationProvider, nó sẽ trả về null => cách này giúp sử dụng nhiều loại Authentication phân tách ở cấp HTTP-filter
   - phương thức sẽ trả về 1 instance Authentication đại diện cho 1 đối tượng đã được xác thực hoàn toàn, isAuthenticated() return true, đối tượng chứa tất cả thông tin chi tiết ngoại trừ thông tin về credentials là thông tin nhạy cảm, sẽ được xoá ngay khi xác thực kết thúc
 - phương thức support:
   - trả về true nếu AuthenticationProvider hiện tại hỗ trợ loại Authentication được cung cấp

* ví dụ tạo class triển khai interface AuthenticationProvider
```java
@Component
public class CustomAuthenticationProvider 
    implements AuthenticationProvider {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Override
    public Authentication authenticate(Authentication authentication) {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();
 
        UserDetails u = userDetailsService.loadUserByUsername(username);
        if (passwordEncoder.matches(password, u.getPassword())) {
            return new UsernamePasswordAuthenticationToken(username, password, 
                u.getAuthorities()); 
        } else {
            throw new BadCredentialsException("Something went wrong!"); 
        }
    }
 
    @Override
    public boolean supports(Class<?> authenticationType) {
        return authenticationType
            .equals(UsernamePasswordAuthenticationToken.class);
        // class UsernamePasswordAuthenticationToken là 1 triển khai của interface Authentication
        // đại diện cho yêu cầu xác thực tiêu chuẩn với username và password
    }
}

@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private AuthenticationProvider authenticationProvider;
    // spring biết cần phải tìm trong context 1 triển khai cụ thể cho AuthenticationProvider
    // đó là: CustomAuthenticationProvider

    @Override
    protected void configure(AuthenticationManagerBuilder auth) {
        auth.authenticationProvider(authenticationProvider);
    }
    // Omitted code
}
```

### Sử dụng SecurityContext
* sử dụng lưu trữ thông tin người dùng đã xác thực
```java
public interface SecurityContext extends Serializable {
    Authentication getAuthentication();
    void setAuthentication(Authentication authentication);
}
```

* Spring security đưa ra 3 chiến lược để quản lý SecurityContext với 1 đối tượng trong vai trò quản lý có tên là SecurityContextHolder
 - **MODE_THREADLOCAL**: cho phép mỗi thread lưu trữ chi tiết riêng của nó trong SecurityContext, cách phổ biến do mỗi request là 1 thread riêng lẻ (mặc định). Sử dụng ThreadLocal
 - **MODE_INHERITABLETHREADLOCAL**: tương tự MODE_THREADLOCAL, ngoài ra cho Spring Security sao chép SecurityContext tới thread tiếp theo trong trường hợp method bất đồng bộ. thread mới sẽ chạy @Async method kế thừa lại SecurityContext
 - **MODE_GLOBAL**: tất cả các thread có cùng 1 instance SecurityContext

* sử dụng chiến lược MODE_INHERITABLETHREADLOCAL để truy cập SecurityContext trong phương thức được gắn @Async 
 - khi 1 request cần xử lý trên nhiều thread
 - chỉ hoạt động khi framework tự tạo thread, nếu tạo thread bằng code => không truy cập được secutity context (do framework không biết thread nào được tạo)

```java
// method
@GetMapping("/bye")
@Async 
public void goodbye() {
    SecurityContext context = SecurityContextHolder.getContext();
    String username = context.getAuthentication().getName();
    // do something with the username
}

// class config
@Configuration
@EnableAsync
public class ProjectConfig {
    @Bean
    public InitializingBean initializingBean() {
    return () -> SecurityContextHolder
        .setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);
    }
}
```
* sử dụng chiến lược MODE_GLOBAL để chia sẻ SecurityContext ở tất cả các thread
 - không sử dụng với web server, do cần có context độc lập giữa các request
```java
@Bean
public InitializingBean initializingBean() {
    return () -> SecurityContextHolder
        .setStrategyName(SecurityContextHolder.MODE_GLOBAL);
}
```

* chuyển tiếp SpringContext sử dụng DelegatingSecurityContextRunnable
.....


### HTTP Basic 
* là phương thức mặc định 
* xác thực chỉ yêu cầu client giử username, passwword qua HTTP Authorization header
* client đính kèm tiền tố Basic, theo sau là mã hoá Base64 của chuỗi chứa username và password, phân tách bằng dấu hai chấm (:) vào header
* Base64 chỉ mã hoá để thuận tiện cho chuyển đổi, không bảo mật 
=> không sử dụng HTTP Basic nếu không có HTTPS bảo mật

Ex: curl -H "Authorization: Basic dXNlcjo5M2EwMWNmMC03OTRiLTRiOTgtODZlZi01NDg2MGYzNmY3ZjM=" localhost:8080/hello 
nếu không dùng base64 để mã hoá => có thể gọi trực tiếp như sau
curl -u user:93a01cf0-794b-4b98-86ef-54860f36f7f3 http://localhost:8080/hello

- Gọi phương thức httpBasic() của HttpSecurity với tham số loại Customizer, cấu hình liên quan đến realm name
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.httpBasic(c -> {
        c.realmName("OTHER");
    });
    // c tham số có type là Customizer<HttpBasicConfigurer<HttpSecurity>>
    // cho phép thiết lập cấu hình liên quan đến phương thức xác thực, như là đặt realm name
    http.authorizeRequests().anyRequest().authenticated();
}
```

- Sử dụng Customizer để tuỳ chỉnh response cho xác thực thất bại
  - triển khai interface AuthenticationEntryPoint, có phương thức commence() nhận vào HttpServletRequest, HttpServletRespone, AuthenticationException

```java
public class CustomEntryPoint implements AuthenticationEntryPoint {
    // AuthenticationEntryPoint không phản ánh việc sử dụng nó khi xác thực không thành công
    // Spring Security, có 1 thành phần tên ExceptionTranslationManager, sử dụng AuthenticationEntryPoint xử lý mọi AccessDeniedException và AuthenticationException được throw trong filter chain
    @Override
    public void commence(HttpServletRequest httpServletRequest, 
        HttpServletResponse httpServletResponse, AuthenticationException e) 
        throws IOException, ServletException {
        httpServletResponse.addHeader("message", "Luke, I am your father!");
        httpServletResponse.sendError(HttpStatus.UNAUTHORIZED.value());
    }
}

@Override
protected void configure(HttpSecurity http) throws Exception {
    http.httpBasic(c -> {
        c.realmName("OTHER");
        c.authenticationEntryPoint(new CustomEntryPoint());
    });
    // đăng ký CustomEntryPoint với HTTP Basic
    // => response trả về của lỗi không xác thự trong tiêu đề chứa message, giá trị là "Luke, I am your father!"
 
    http.authorizeRequests()
        .anyRequest()
        .authenticated();
}
```

### form-login
```java
@Component
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    // tuỳ chỉnh logic cho xác thực thành công
    @Override
    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, 
        HttpServletResponse httpServletResponse, Authentication authentication) 
        throws IOException {
 
        var authorities = authentication.getAuthorities();
        var auth = authorities.stream()
            .filter(a -> a.getAuthority().equals("read"))
            .findFirst(); 
        // trả về 1 object Optional trống nếu authority read không tồn tại

        if (auth.isPresent()) { 
            // có tồn tại authority read => redirect "/home"
            httpServletResponse.sendRedirect("/home");
        } else {
            httpServletResponse.sendRedirect("/error");
        }
    }
}

@Component
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {

     // tuỳ chỉnh logic cho xác thực không thành công
    @Override
    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, 
        HttpServletResponse httpServletResponse, AuthenticationException e) {
        httpServletResponse.setHeader("failed", LocalDateTime.now().toString());
    }
}

@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomAuthenticationSuccessHandler authenticationSuccessHandler;
    @Autowired
    private CustomAuthenticationFailureHandler authenticationFailureHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //http.formLogin().defaultSuccessUrl("/home", true);
        // xác thực bằng form-login, sau khi xác thực thành công chuyển hướng đến /home

        http.formLogin()
            .successHandler(authenticationSuccessHandler)
            .failureHandler(authenticationFailureHandler)
        // đăng ký 2 object xác thực thành công và thất bại
        .and()
            .httpBasic();
        // cho phép truy cập cả bằng HTTP Basic (curl)

        http.authorizeRequests().anyRequest().authenticated();
    }
}
```

# web application demo

* file schema.sql
```sql
CREATE TABLE IF NOT EXISTS `spring`.`user` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `username` VARCHAR(45) NOT NULL,
    `password` TEXT NOT NULL,
    `algorithm` VARCHAR(45) NOT NULL,
    PRIMARY KEY (`id`));

CREATE TABLE IF NOT EXISTS `spring`.`authority` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(45) NOT NULL,
    `user` INT NOT NULL,
    PRIMARY KEY (`id`));

CREATE TABLE IF NOT EXISTS `spring`.`product` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(45) NOT NULL,
    `price` VARCHAR(45) NOT NULL,
    `currency` VARCHAR(45) NOT NULL,
    PRIMARY KEY (`id`));

INSERT IGNORE INTO `spring`.`user` (`id`, `username`, `password`, `algorithm`) 
    VALUES ('1', 'john', '$2a$10$xn3LI/AjqicFYZFruSwve.681477XaVNaUQbr1gioaWPn4t1KsnmG',
        'BCRYPT');
INSERT IGNORE INTO `spring`.`authority` (`id`, `name`, `user`) 
    VALUES ('1', 'READ', '1');
INSERT IGNORE INTO `spring`.`authority` (`id`, `name`, `user`) 
    VALUES ('2', 'WRITE', '1');
INSERT IGNORE INTO `spring`.`product` (`id`, `name`, `price`, `currency`) 
    VALUES ('1', 'Chocolate', '10', 'USD');
```

* file pom.cxml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

* file application.properties
```properties
spring.datasource.url=jdbc:mysql://localhost/spring?useLegacyDatetimeCode=false&s     
    serverTimezone=UTC
spring.datasource.username=<your_username>
spring.datasource.password=<your_password>
spring.datasource.initialization-mode=always
```

* entity
```java
@Entity
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    
    private String username;
    
    private String password;
    
    @Enumerated(EnumType.STRING)
    private EncryptionAlgorithm algorithm;
    
    @OneToMany(mappedBy = "user", fetch = FetchType.EAGER)
    private List<Authority> authorities;
    // Omitted getters and setters
}
public enum EncryptionAlgorithm {
    BCRYPT, SCRYPT
}

@Entity
public class Authority {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
 
    private String name;
 
    @JoinColumn(name = "user")
    @ManyToOne
    private User user;
    // Omitted getters and setters
}
```

* repositỏy
```java
public interface UserRepository extends JpaRepository<User, Integer> {
    Optional<User> findUserByUsername(String u); 
}
```

* implementation UserDetails
```java
public class CustomUserDetails implements UserDetails {
 
    private final User user;
    public CustomUserDetails(User user) {
        this.user = user;
    }
 
    public final User getUser() {
        return user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getAuthorities().stream()
            .map(a -> new SimpleGrantedAuthority(a.getName())) 
            .collect(Collectors.toList()); 
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }   

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

* impelementation UserDetailsService
```java
@Service
public class JpaUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
 
    @Override
    public CustomUserDetails loadUserByUsername(String username) {
        Supplier<UsernameNotFoundException> s = () 
            -> new UsernameNotFoundException("Problem during authentication!");
        
        User u = userRepository.findUserByUsername(username) 
            .orElseThrow(s); 
        return new CustomUserDetails(u); 
    }
}
```

* implementation AuthenticationProvider
```java
@Service
public class AuthenticationProviderService implements AuthenticationProvider {
 
    @Autowired 
    private JpaUserDetailsService userDetailsService;
 
    @Autowired
    private BCryptPasswordEncoder bCryptPasswordEncoder;
 
    @Autowired
    private SCryptPasswordEncoder sCryptPasswordEncoder;
 
    @Override
    public Authentication authenticate(Authentication authentication) 
        throws AuthenticationException {
        
        String username = authentication.getName();
        String password = authentication.getCredentials()
            .toString();
 
        CustomUserDetails user = userDetailsService.loadUserByUsername(username);
        
        switch (user.getUser().getAlgorithm()) { 
        case BCRYPT: 
            return checkPassword(user, password, bCryptPasswordEncoder);
        case SCRYPT: 
            return checkPassword(user, password, sCryptPasswordEncoder);
        }

        throw new BadCredentialsException("Bad credentials");
    }
 
    @Override
    public boolean supports(Class<?> aClass) {
        return return UsernamePasswordAuthenticationToken.class
            .isAssignableFrom(aClass);
    }

    private Authentication checkPassword(CustomUserDetails user, String rawPassword, 
        PasswordEncoder encoder) {
 
        if (encoder.matches(rawPassword, user.getPassword())) {
            return new UsernamePasswordAuthenticationToken(user.getUsername(), 
                user.getPassword(), user.getAuthorities());
        } else {
            throw new BadCredentialsException("Bad credentials");
        }
    }
}
```

* đăng ký AuthenticationProvider trong class configuration
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired 
    private AuthenticationProviderService authenticationProvider;
 
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
 
    @Bean
    public SCryptPasswordEncoder sCryptPasswordEncoder() {
        return new SCryptPasswordEncoder();
    }
 
    @Override
    protected void configure(AuthenticationManagerBuilder auth) {
        auth.authenticationProvider(authenticationProvider); 
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin().defaultSuccessUrl("/main", true);
        http.authorizeRequests().anyRequest().authenticated();
    }
}
```

# Hạn chế truy cập
