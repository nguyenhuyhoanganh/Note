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

* repository
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
### 3 cách định cấu hình quyền truy cập sử dụng các method:
* *hasAuthority()*: nhận 1 authority dưới dạng tham số, chỉ người dùng có quyền hạn đó mới được gọi endpoint
* *hasAnyAuthority*: có thể nhận nhiều hơn 1 authority mà cấu hình cấu hình hạn chế, người dùng có ít nhất 1 trong các quyền hạn có thể truy cập endpoint
* *access()*: cung cấp khả năng định cấu hình quyền truy cập dựa trên SpEL

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter{
    
    @Bean 
    public UserDetailsService userDetailsService() {
        var manager = new InMemoryUserDetailsManager();
        var user1 = User.withUsername("john") 
            .password("12345")
            .authorities("READ")
            .build();
        var user2 = User.withUsername("jane") 
            .password("12345")
            .authorities("WRITE")
            .build();
 
        manager.createUser(user1); 
        manager.createUser(user2);
        return manager;
        // UserDetailsService trả về được thêm vào spring context
    }
 
    @Bean 
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance(); 
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic();
 
        http.authorizeRequests()  
            //.anyRequest().permitAll(); 
            // cho phép tất cả req đều truy cập được

            //.anyRequest().denyAll();
            // ngược lại permitAll

            //.anyRequest().authenticated();
            // yêu cầu người dùng phải xác thực

            //.anyRequest().hasAuthority("WRITE");
            // người dùng phải có authority WRITE mới truy cập được, nếu không trả về 403 

            .anyRequest().hasAnyAuthority("WRITE", "READ");
            // người dùng có 1 trong 2 authority có thể tuy cập

            // .anyRequest().access("hasAuthority('read') and !hasAuthority('delete')");
            // có authority read, nhưng không có delete
            // SpEL "T(java.time.LocalTime).now().isAfter(T(java.time.LocalTime).of(12, 0))"
            // chỉ cho truy cập sau 12 giờ đêm
            // https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions

 }
}
```

### hạn chế quyền truy cập endpoint dựa trên role
* 1 role bao quát 1 hoặc nhiều hành động mà người dùng được phép thực hiện
* tên role bắt đầu với tiền tố "ROLE_", là 1 GrantedAuthority, lưu trong db
* các phương pháp ràng buộc vai trò của người dùng:
  - hasRole()
  - hasAnyRole()
  - access()
  
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
 
    @Bean
    public UserDetailsService userDetailsService() {
        
        var manager = new InMemoryUserDetailsManager();
        var user1 = User.withUsername("john")
            .password("12345")
            .authorities("ROLE_ADMIN") 
            .build();

        var user2 = User.withUsername("jane")
            .password("12345")
            .roles("MANAGER") // không cần thêm tiền tố ROLE, tự động được thêm
            .build();

 
        manager.createUser(user1);
        manager.createUser(user2);
        return manager;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic();
        http.authorizeRequests()
            .anyRequest().hasRole("ADMIN"); 
            // chỉ định role, không cần thêm tiền tố ROLE_
    }
}
```

### Áp dụng các hạn chế cho từng endpoint

* *anyRequest()*: khớp với tất cả các request
* *mvcMatchers()*: sử dụng MVC expressions cho các paths để chọn endpoint
* *antMatchers()*: sử dụng Ant expressions cho các paths để chọn endpoint
* *regexMatchers()*: sử dụng regular expressions cho các paths để chọn endpoint (regex)

* sử dụng mvcMatchers
```java
http.httpBasic();

http.authorizeRequests()
    .mvcMatchers("/hello").hasRole("ADMIN") // sử dụng MVC expressions cho path /hello
    .mvcMatchers(HttpMethod.POST, "/ciao").hasRole("MANAGER") // chỉ cho phép method POST 
    .mvcMatchers( "/a/b/**").authenticated() // tất cả path có tiền tố "/a/b" cần xác thực
    // * : đại diện cho 1 pathname bất kỳ => /a/b/c/d khớp với /a/*/c/d
    // ** : đại diện cho nhiều pathname => /a/b/c/d/e khớp với /a/**/e
    .mvcMatchers("/product/{code:^[0-9]*$}").authenticated() 
    // sử dụng regex: 1 chuỗi độ dài bất kỳ chứa bất kỳ chữ hoặc số 
    ..mvcMatchers("/email/{email:.*(.+@.+\\.com)}").authenticated()
    // kết thúc bằng 1 email
    .anyRequest().permitAll();

    // nếu sử dụng path truy cập được với permitAll nhưng người dùng gọi endpoint
    // cung cấp thêm username và password, Spring Security cũng thực hiện xác thực người dùng
    // nếu xác thực không thành công sẽ trả về 401 Unauthorized

http.csrf().disable(); // discable để cho phép truy cập method POST
```

* sử dụng antMatchers
```java
    http.httpBasic();
    
    http.authorizeRequests()
        .antMatchers("/hello")
        // sử dụng tương tự mvcMatchers
        // có thêm 1 cách .antMatchers(HttpMethod method), 
        // tương tự .antMatchers(HttpMethod methodd, “/**”)
        .authenticated();
```

* sử dụng mvcMatchers để so khớp với path: /hello, Spring sẽ tự động bảo mật thêm cho path /hello/ , trong khi antMatchers thì không

* sử dụng regexMatchers
```java
http.authorizeRequests()
    .regexMatchers(".*/(us|uk|ca)+/(en|fr).*")
    // path hợp lệ có dạng: /video/us/en
    .authenticated()
```

# Filter Chain
* HTTP filters uỷ quyền các trách nhiệm khác nhau lên HTTP request
* các filter nhận request, thực hiện logic của nó và uỷ quyền cho filter tiếp theo trong chain
* mỗi filter sẽ uỷ quyền trách nhiệm cho 1 manager
  Ex: BasicAuthenticationFilter sử dụng AuthenticationManager
       trách nhiệm sử dụng phương pháp HTTP Basic xác thực dựa username, password
* cần tuỳ chỉnh filter để đáp ứng các nhu cầu khác nhau, thêm hoặc thay thể filter trong chain
* mỗi filter trong chain có 1 chỉ mục gọi là order đánh thứ tự thực hiện filter trong chain, có thể có nhiều filter với order giống nhau, lúc này Spring Security không đảm bảo thứ tự thực hiện filter ở order đó, có thể ngẫu nhiên

* các filter trong SpringSecurity là các HTTP filter điển hiển
  * để tạo filter tuỳ chỉnh, thực thi implement Filter của package javax.servlet package
  * cần ghi đè method doFilter() để thực thi logic, nhận vào ServletRequest, ServletResponse và FilterChain
  * ServletRequest đại diện cho HTTP request, sử dụng để nhận các thông tin chi tiết trong request
  * ServletResponse đại diện cho HTTP Response, sử dụng để thay đổi response trước khi giử lại client hoặc chuyển cho filter tiếp theo trong chain
  * FilterChain đại diện cho chuỗi chain, sử dụng để chuyển tiếp request từ filter tiếp theo trong chain: java filterChain.doFilter(request, response); 

* Ex: tạo 1 filter yêu cầu request cần có header: request-id 
```java
public class RequestValidationFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
        FilterChain filterChain) throws IOException, ServletException {
 
        var httpRequest = (HttpServletRequest) request;
        var httpResponse = (HttpServletResponse) response;
 
        String requestId = httpRequest.getHeader("Request-Id");
        if (requestId == null || requestId.isBlank()) {
            httpResponse.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            return; 
            // header không tồn tại, response trả về 400 Bad Request
            // request không được chuyển tiếp
        }
 
        filterChain.doFilter(request, response); 
        // header tồn tại, chuyển tiếp đến request tới
    }
}
```

* các cách thêm filter vào chain
  - addFilterBefore(): thêm filter vào trước 1 filter trong chain. Nhận 2 tham số: 1 instance của custom filter muốn thêm vào chain, loại của filter mà muốn thêm custom filter vào trước
  - addFilterAfter(): thêm 1 filter vào sau 1 filter trong chain. tương tự addFilterBefore
  - addFilterAt(): thêm 1 filter tại vị trí 1 filter xác định, không thay thể filter đã có, nhưng cùng thực thi logic, có thể đặt filter ở vị trí BasicAuthenticationFilter. lúc này nếu không gọi method .httpBasic() => không có BasicAuthenticationFilter ở đó mà chỉ có filter tuỳ chỉnh vừa thêm

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.addFilterBefore(
                new RequestValidationFilter(),
                BasicAuthenticationFilter.class)
            // phiên BasicAuthenticationFilter chưa được thêm vào do chưa gọi httpBasic()
            // tuy nhiên RequestValidationFilter vẫn được thêm vào chain
            .authorizeRequests()
            .anyRequest().permitAll();
    }
}
```

* Spring Security cung cấp các lớp triển khai Filter riêng => có thể mở rộng các lớp để triển khai filter tuỳ chỉnh
  - GenericFilterBean: cho phép sử dụng các tham số khởi tạo được xác định trong web.xml nếu có
  - OncePerRequestFilter: là 1 class extends GenericFilterBean
    - framework 0 đảm bảo rằng nó được gọi một lần mỗi request, tuy nhiên nó triển khai logic đảm bảo phương thức doFilter() chỉ thực thi 1 lần mỗi request.
    - viết logic trong method doFilterInternal() thay vì method doFilter(). 
    - nó chỉ hỗ trợ HTTP request.
    - có thể quyết định xem filter được áp dụng cho từng request nhất định bằng cách ghi đè method shouldNotFilter(HttpServletRequest), mặc định filter áp dụng cho mọi request. 
    - nó không áp dụng cho các request bất đồng bộ, hoặc error dispatch request, có thể thay đổi hành vi bằng cách ghi đè method shouldNotFilterAsyncDispatch() và shouldNotFilterErrorDispatch().

```java
public class AuthenticationLoggingFilter extends OncePerRequestFilter { 
 
    private final Logger logger =Logger
        .getLogger(AuthenticationLoggingFilter.class.getName());
 
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
        HttpServletResponse response, FilterChain filterChain) throws 
        ServletException, IOException {
 
        String requestId = request.getHeader("Request-Id");
        logger.info("Successfully authenticated request with id " +requestId);
        filterChain.doFilter(request, response);
 }
}
```

# CSRF và CORS
### Cách bảo vệ CSRF
- khi thêm, thay đổi hoặc xoá 1 tài nguyên trên công cụ web, trang web sẽ gọi 1 endpoint từ server để thực thi các hoạt động này. trong quá trình này, nếu mở 1 trang web độc hại, nó sẽ thay mặt gọi đến server và thực thi hành động thay người dùng. Do đã đăng nhập trước đó, nên server tin tưởng hành vi là đến từ người dùng.
- CSRF đảm bảo rằng chỉ giao diện người dùng của ứng dụng web mới có thể thực hiện các hoạt động thay đổi
- trước khi làm bất kỳ hành động thay đổi dữ liệu nào, người dùng phải giử HTTP GET để xem trang web ít nhất 1 lần. => ứng dụng tạo 1 token khi đó. Và ứng dụng chỉ chấp nhận cho các hoạt động POST, PUT, DELETE, ... nếu request có chứa token đó trong header
- trong filter chain có 1 filter là CsrfFilter chặn các request và cho phép các request đi qua nếu là GET, HEAD, TRACE, OPTIONS. Các request còn lại, filter mong đợi 1 token trong header. Nếu header không tồn tại hoặc token không chính xác, ứng dụng sẽ từ chối uyên cầu và trả về 403 Forbidden. headername của token là X-CSRF-TOKEN, và giá trị của nó chỉ là 1 string
- CsrfFilter sử dụng 1 CsrfTokenRepository để tạo mới và lưu trữ token. mặc định sẽ lưu trữ token trong Session và tạo token dưới dạng 1 UUID, token này cũng được lưu lại trong request với tên atttribute là "_csrf" đế chuyển qua filter khác

* Ex: tạo 1 filter log ra token mà CsrfTokenRepository tạo
```java
public class CsrfTokenLogger implements Filter {
    
    private Logger logger = Logger.getLogger(CsrfTokenLogger.class.getName());
 
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
        FilterChain filterChain) throws IOException, ServletException {
 
        Object o = request.getAttribute("_csrf"); 
        // lấy giá trị cúa token từ thuộc tính _csrf của request
        CsrfToken token = (CsrfToken) o;
        logger.info("CSRF token " + token.getToken());
        filterChain.doFilter(request, response);
 }
}

@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
 
        http.addFilterAfter(new CsrfTokenLogger(), CsrfFilter.class)
        // thêm filter custom vừa tạo vào ngay sau CsrfFilter
            .authorizeRequests()
            .anyRequest().permitAll();
    }
}
```

* vì CsrfTokenRepository sử dụng Session để lưu trữ token phía máy chủ => lúc thực thi hoạt động thay đổi dữ liệu cần giử thêm JSESSIONID được tạo ra trong các lần GET request trước đó, tất nhiên là phải kèm theo X-CSRF-TOKEN

### Cách bảo vệ CORS
* không cho phép trang web khác gọi đến REST endpoint, hoặc nhúng iframe từ trang web của mình
* trong tình huống cần tạo cuộc gọi giữa 2 tên domain khác nhau, CORS cho phép chỉ định ứng dụng từ tên domain nào được phép gọi đến.
* CORS hoạt động dựa trên HTTP headers:
  - Access-Control-Allow-Origin: chỉ định các domain ngoài có thể truy cập vào tài nguyên trên domain của mình
  - Access-Control-Allow-Methods: Cho phép chỉ định 1 số HTTP method, trong tình huống chúng ta muốn cho phép truy cập từ domain khác, với HTTP method cụ thể
  - Access-Control-Allow-Headers: thêm giới hạn cho những header có thể sử dụng trong 1 request cụ thể
* mặc định, không có header nào trong các headers trên được thêm vào response. Khi thực hiện các cross-orign call, úng dụng mong đợi header Access-Control-Allow-Origin trong response, header này chứa các domain mà server cho phép. Nếu không có, browser sẽ không chấp nhận response. 
* localhost và 127.0.0.1 cùng tham chiếu tới 1 server nhưng cũng được tính là có nguồn gốc khác nhau => thực thi gọi đến 127.0.0.1 từ localhost, broser vẫn không nhận được response do bị block với CORS policy
* sử dụng annotation ` @CrossOrigin({"_domain", "_domain2"}) ` trên method muốn cho phép domain khác truy cập

* áp dụng CORS sử dụng CorsConfigurer
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors(c -> { 
            // c là 1 object type: Customizer<CorsConfigurer>
            CorsConfigurationSource source = request -> {
                CorsConfiguration config = new CorsConfiguration();
                config.setAllowedOrigins(
                    List.of("example.com", "example.org"));
                    // các domain cho phép truy cập nguồn gốc chéo
                config.setAllowedMethods(
                    List.of("GET", "POST", "PUT", "DELETE"));
                return config;
            };
            c.configurationSource(source);
        });
        
        http.csrf().disable();
        http.authorizeRequests().anyRequest().permitAll();
 }
}
```

# Demo App sử dụng JWT
### 3 tác nhận hệ thống
* **Client**: là ứng dụng sử dụng backend, có thể là mobile app hoặc frontend của web app sử dụng ReactJS, Angular, ...
* **Authentication Server**: là ứng dụng với db lưu trữ thông tin đăng nhập của user. Sử dụng để xác thực user dựa trên thông tin đăng nhập user cung cấp.  
* **Business Logic Server**: lá ứng dụng hiển thị các endpoint mà client tiêu thụ. Cần bảo mật quyền truy cập đến các endpoint. Trước khi gọi đến endpoint, user cần xác thực bằng Authentication Server, sử dụng token Authentication Server cung cấp để gọi đến các endpoint

### JWT
* Lợi ích sử dụng token
  * giúp tránh chia sẻ thông tin đăng nhập trong tất cả các request (HTTP Basic)
  * có thể xác định token với thòi gian tồn tại ngắn
  * có thể vô hiệu hoá token mà không làm mất hiệu lực thông tin đăng nhập
  * có thể lưu trữ thông tin chi tiết của người dùng như quyền hạn
  * giúp uỷ thác trách nhiệm cho 1 thành phần khác trong hệ thống
* JWT là gì:
  * JWT là 1 loại token thiết kế cho các yêu cầu web, sử dụng JSON để định dạng dữ liệu
  * JWT gồm 3 phần, được ngăn cách bằng dấu ".":
    * 2 phần đầu là header và body dưới dạng JSON được mã hoá bằng Base64. header lưu tên của thuật toán tạo ra chữ ký. body bao gồm các thông tin cần thiết để uỷ quyền
    * phần cuối cùng là chữ ký điện tử

### Demo
#### Authentication Server

* pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

* application.properties
```properties
spring.datasource.url=jdbc:mysql://localhost/spring
spring.datasource.username=root
spring.datasource.password=
spring.datasource.initialization-mode=always
```


* schema.sql
```sql
CREATE TABLE IF NOT EXISTS `spring`.`user` (
    `username` VARCHAR(45) NULL,
    `password` TEXT NULL,
    PRIMARY KEY (`username`));
CREATE TABLE IF NOT EXISTS `spring`.`otp` (
    `username` VARCHAR(45) NOT NULL,
    `code` VARCHAR(45) NULL,
    PRIMARY KEY (`username`));
```

* config
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
    @Bean 
    public PasswordEncoder passwordEncoder() { 
        return new BCryptPasswordEncoder(); 
    } 
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable(); 
        http.authorizeRequests().anyRequest().permitAll(); 
    }
}
```
...................................
https://github.com/nguyenhuyhoanganh/spring-security-in-action-source/tree/master/ssia-ch11-ex1-s1



# Triển khai bảo mật method với 
* Giả sử ứng dụng không phải ứng dụng web, không thể sử dụng SpringSecurity cho bảo mật và uỷ quyền với HTTP endpoints. Chúng ta sẽ sử dụng cấu hình uỷ quyền ở cấp độ method, như sử dụng cho các layer service, repository, ...
* Theo mặc định bảo mật cấp độ method bị vô hiệu hoá, cần kích hoạt nó. Nó cũng cung cấp nhiều cách tiếp cận để áp dụng uỷ quyền:
  * Call authorization: quyết định ai đó có thể gọi method theo quy tắc đặc quyền đã triển khai (preauthorization) hoặc nếu ai đó có thể truy cập những gì method return sau khi method được thực thi (postauthorization)
  * Filtering: quyết định những gì method có thể nhận thông qua các tham số của nó (prefiltering) và những gì người có thể nhận lại từ method sau khi method thực thi (postfiltering)

### Call authorization (Pre- and postauthorizations)
* Đề cập đến việc áp dụng các quy tắc uỷ quyền để quyết định xem 1 method có thể gọi hay cho phép method sau khi gọi quyết định người gọi có được truy cập giá trị được return bởi method
* Cơ chế đăng sau việc áp dụng quy tắc uỷ quyền là kích hoạt Spring Aspect. Aspect chặn cuộc gọi đến method cho áp dụng quy tắc uỷ quyền, dựa trên quy tắc uỷ quyền quyết định có chuyển tiếp cuộc gọi đến method bị chặn hay không.
* Phân loại call authorization:
  * Preauthorization: framework kiểm tra các quy tắc uỷ quyền trước khi method gọi
  * Postauthorization: framework kiểm tra quy tắc uỷ quyền sau khi method thực thi
* Giả sử có method  findDocumentsByUser(String username) trả về các tài liệu cho 1 user cụ thể. Người gọi cung cấp username làm tham số để method truy vấn tài liệu. Cần đảm bảo user được xác thực chỉ thu được tài liệu họ sở hữu. Cần áp dụng quy tắc cho method chỉ nhận username của user đã xác thực làm tham số
* Khi áp dụng quy tắc uỷ quyền hoằn toàn cấm bất cứ ai gọi tới method trong tình huống cụ thể là preauthorization. Framework xác minh điều kiện uỷ quyền trước khi thực hiện method, nếu người gọi tới không có quyền theo quy tắc uỷ quyền, framework không uỷ quyền gọi tới method và ném ra 1 exception
* Khi áp dụng quy tắc uỷ quyền cho phép ai đó gọi tới method nhưng không cần thiết nhận được kết quả return của method, sử dụng postauthorization. Framework kiểm tra quy tắc uỷ quyền sau khi thực thi method. Sử dụng loại uỷ quyền này để hạn chế truy cập tới method return trong điều kiẹn nhất định. Áp dụng quy tắc uỷ quyền trên kết quả return bởi method.
* Cẩn thận với postauthorization, vì dù quy tắc uỷ quyền áp dụng có thành công hay không thì sự thay đổi vẫn xảy ra trong quá trình thực thi method. Ngay cả khi sử dụng annotation @Transactional, thay đôi vẫn không được rollback nếu postauthorization fails
#### Cấu hình cho phép sử dụng bảo mật method
* Sử dụng annotation @EnableGlobalMethodSecurity, cung cấp 3 cách tiếp cận để xác định quy tắc uỷ quyền
  * pre-/postauthorization annotations
  * JSR 250 annotation, @RolesAllowed
  * @Secured annotation
* Hầu hết trường hợp pre-/postauthorization annotations là cách tiếp cận thường xuyên được sử dụng, cần cung cấp thuộc tính prePostEnabled của @EnableGlobalMethodSecurity thành true
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig {}
```
* Sử dụng bảo mật method với bất kỳ phương thức xác thực nào từ HTTP Basic tới OAuth2, để đơn giản, hãy sử dụng xác thực với HTTP Basic

#### Triển khai bảo mật bằng preauthorization
* Ví dụ về sử dụng preauthorization, cung cấp 1 endpoint /hello trả về chuỗi "Hello, " và tên. Để lấy được tên, controller gọi tới service method. Method này áp dụng preauthorization để xác minh user phải có quyền "write" 
* Class config cấu hình 1 số user
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true) 
public class ProjectConfig {
  @Bean 
  public UserDetailsService userDetailsService() {
    var service = new InMemoryUserDetailsManager();
    
    var u1 = User.withUsername("natalie") 
      .password("12345")
      .authorities("read")
      .build();
    
    var u2 = User.withUsername("emma")
      .password("12345")
      .authorities("write")
      .build();
    
    service.createUser(u1);
    service.createUser(u2);
    return service;
  }

  @Bean 
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }
}
```
* Sử dụng annotation @PreAuthorize trên method service. Annotation nhận giá trị giá dạng biểu thức SpEL mô tả quy tắc uỷ quyền. 
* Có thể xác định hạn chế cho user dựa trên quyền hạn nhất định sử dụng hasAuthority()
```java
@Service
public class NameService {
  @PreAuthorize("hasAuthority('write')") // chỉ user có quyền write mới gọi được method
  public String getName() {
    return "Fantastico";
  }
}
```
* Class controller
```java
@RestController
public class HelloController {
  @Autowired 
  private NameService nameService;
  
  @GetMapping("/hello")
  public String hello() {
    return "Hello, " + nameService.getName(); 
  }
}
```
* Tương tự, sử dụng SpEL khác như:
  * hasAnyAuthority(): chỉ định nhiều quyền, user phải có ít nhất 1 trong những quyền được chỉ định để gọi tới method
  * hasRole(): chỉ định 1 vai trò nhất định user phải có để gọi method
  * hasAnyRole() : chỉ định nhiều vai trò
* Ví dụ khác, endpoint nhận giá trị thông qua path variable và gọi method ở service. triển khai quy tắc uỷ quyền dựa trên giá trị này
* Class controller
```java
@RestController
public class HelloController {
  @Autowired 
  private NameService nameService;

  // xác định endpoint nhận giá trị từ path variable
  @GetMapping("/secret/names/{name}") 
  public List<String> names(@PathVariable String name) {
    // gọi tới method được bảo vệ cung cấp them số là giá trị path variable
    return nameService.getSecretNames(name); 
  }
}
```
* Class service
```java
@Service
public class NameService {
  
  private Map<String, List<String>> secretNames = 
    Map.of("natalie", List.of("Energico", "Perfecto"),"emma", List.of("Fantastico"));
 
  // sử dung #name đại diện cho giá trị tham số "name" truyền vào method getSecretNames()
  // có quyền truy cập tới object authentication mà sử dụng để tham chiều tới user hiện tại đã xác thực
  // method chỉ được gọi nếu username của user đã xác thực giống với giá trị truyền vào làm tham số của method
  @PreAuthorize 
  ("#name == authentication.principal.username")
  public List<String> getSecretNames(String name) {
    return secretNames.get(name);
  }
}
```
* Có thể áp dụng bảo mật method cho bất kỳ lớp nào trong ứng dụng, không chỉ riêng lớp service
#### Triển khai bảo mật bằng postauthorization
* Tưởng tượng method lấy 1 số dữ liệu từ nguồn dữ liệu như web service hay database, có thể tự tin về những gì method thực hiện, những không chắc chắn về bên thứ 3 mà method gọi tới. Vì vậy cho phép method thực thi, nhưng cần xác thực những gì nó trả về, nếu khong đáp ứng các tiêu chí, sẽ không cho phép người gọi truy cập giá trị method return
* Sử dụng annotation @PostAuthorize 
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig {
  @Bean
  public UserDetailsService userDetailsService() {
    var service = new InMemoryUserDetailsManager();
    var u1 = User.withUsername("natalie")
      .password("12345")
      .authorities("read")
      .build();
 
    var u2 = User.withUsername("emma")
      .password("12345")
      .authorities("write")
      .build();
 
    service.createUser(u1);
    service.createUser(u2);
    return service;
  }
 
  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }
}

// mỗi employee được liên kết với 1 user
public class Employee {
  private String name;
  private List<String> books;
  private List<String> roles;
  // Omitted constructor, getters, and setters
}

@Service
public class BookService {
  private Map<String, Employee> records =
    Map.of("emma", new Employee("Emma Thompson",
        List.of("Karamazov Brothers"),
        List.of("accountant", "reader")),
      "natalie", new Employee("Natalie Parker",
        List.of("Beautiful Paris"),
        List.of("researcher"))
    );

  @PostAuthorize("returnObject.roles.contains('reader')")
  public Employee getBookDetails(String name) {
    return records.get(name);
  }
}

@RestController
public class BookController {
  @Autowired
  private BookService bookService;
  
  // chỉ cho user truy cập được list book của employee có roles là read
  @GetMapping("/book/details/{name}")
  public Employee getDetails(@PathVariable String name) {
    return bookService.getBookDetails(name);
  }
}
```
#### Với các logic uỷ quyền phức tạp
* Khi logic uỷ quyền phức tạp, yêu cầu viết SpEL dài, tạo ra mã khó, đọc ảnh hướng đến khả năng bảo trì của ứng dụng, cần tách biệt logic uỷ quyền ra 1 class riêng biệt
* Ví dụ về app quản lý document, để lấy thông tin chi tiết về document chỉ có quản trị viên hoặc người sở hữu document đó
```java
public class Document {
  private String owner;
  // Omitted constructor, getters, and setters
}

@Repository
public class DocumentRepository {
  private Map<String, Document> documents = 
    Map.of("abc123", new Document("natalie"),
      "qwe123", new Document("natalie"),
      "asd555", new Document("emma"));
 
  public Document findDocument(String code) {
    return documents.get(code); 
  }
}

@Service
public class DocumentService {
  @Autowired
  private DocumentRepository documentRepository;
 
  // sử dụng hasPermission() tham chiếu tới biểu thức uỷ quyền
  // returnObject là giá trị trả về
  // ROLE_admin là tên vai trò được cho phép truy cập
  @PostAuthorize("hasPermission(returnObject, 'ROLE_admin')")
  public Document getDocument(String code) {
    return documentRepository.findDocument(code);
  }
}
```
* Viết một object thực hiện interface PermissionEvaluator. PermissionEvaluator cung cấp 2 cách để triển khai logic uỷ quyền:
  * theo object và permission: Giả định nhận được 2 object, 1 là object tuân theo quy tắc uỷ quyền, 1 là object cung cấp các chi tiết cần thiết để triển khai logic uỷ quyền
  * theo object ID, object type và permission: object ID sử dụng truy xuất object cấn thiết, object type sử dụng để áp dụng cho nhiều loại object và 1 object cung cấp chi tiết để uỷ quyền
```java
public interface PermissionEvaluator {
  boolean hasPermission(
    Authentication a, 
    Object subject,
    Object permission);
 
  boolean hasPermission(
    Authentication a, 
    Serializable id, 
    String type, 
    Object permission);
}                                                                                          
```
* Spring tự cung cấp giá trị cho object Authentication như 1 tham số cho method hasPermission()
* Sau đây là class custom sử dụng cách triển khai logic uỷ quyền thứ 1:
```java
@Component
public class DocumentsPermissionEvaluator implements PermissionEvaluator { 
  @Override
  public boolean hasPermission(Authentication authentication, Object target,
    Object permission) {
    Document document = (Document) target; 
    String p = (String) permission; 
    boolean admin = authentication.getAuthorities()
      .stream()
      .anyMatch(a -> a.getAuthority().equals(p));
    return admin || document.getOwner().equals(authentication.getName());
  }

  @Override
  public boolean hasPermission(Authentication authentication, Serializable targetId,
    String targetType, Object permission) {
    return false; 
  }
}
```
* Để cho spring biết về 1 triển khai PermissionEvaluator mới, phải xác định 1 MethodSecurityExpressionHandler trong class config
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig extends GlobalMethodSecurityConfiguration {
  @Autowired
  private DocumentsPermissionEvaluator evaluator;
 
  @Override 
  protected MethodSecurityExpressionHandler createExpressionHandler() {
    var expressionHandler = new DefaultMethodSecurityExpressionHandler();
    expressionHandler.setPermissionEvaluator(evaluator); 
    return expressionHandler; 
  }
  // Omitted definition of the UserDetailsService and PasswordEncoder beans
}
```
* Ở đoạn code trên sử dụng DefaultMethodSecurityExpressionHandler là 1 triển khai của MethodSecurityExpressionHandler mà Spring cung cấp. 
* Có thể tự triển khai 1 MethodSecurityExpressionHandler tuỳ chỉnh để xác định biểu thức SpEL tuỳ chỉnh sử dụng áp dụng các quy tắc uỷ quyền
* Class controller
```java
@RestController
public class DocumentController {
  @Autowired
  private DocumentService documentService;
 
  @GetMapping("/documents/{code}")
  public Document getDetails(@PathVariable String code) {
    return documentService.getDocument(code);
  }
}
```
* Cấu hình sử dụng cách triển khai logic uỷ quyền thứ 2 của PermissionEvaluator, sử dụng 1 định danh và object type thay vì chính object. Ví dụ áp dụng với @PreAuthorize, chưa có object trả về, sử dụng định danh là code của document
```java
@Component
public class DocumentsPermissionEvaluator implements PermissionEvaluator {
 
  @Autowired
  private DocumentRepository documentRepository;
 
  @Override
  public boolean hasPermission(Authentication authentication, Object target,
    Object permission) {
    return false; 
  }
 
  @Override
  public boolean hasPermission(Authentication authentication, Serializable targetId,
    String targetType, Object permission) {
    String code = targetId.toString(); 
    Document document = documentRepository.findDocument(code);
    String p = (String) permission;
    boolean admin = authentication.getAuthorities()
      .stream()
      .anyMatch(a -> a.getAuthority().equals(p));
    return admin || document.getOwner().equals(authentication.getName());
  }
}

@Service
public class DocumentService {
 
  @Autowired
  private DocumentRepository documentRepository;
 
  @PreAuthorize("hasPermission(#code, 'document', 'ROLE_admin')")
  public Document getDocument(String code) {
    return documentRepository.findDocument(code);
  }
}
```

* Với annotation @EnableGlobalMethodSecurity được dùng để bật bảo mật method, cung cấp cho nó thuộc tính prePostEnabled = true cho phép sử dụng các annotation @PreAuthorize và @PostAuthorize để định cấu hình uỷ quyền trước và sau khi thực thi method. Spring còn cung cấp thêm 2 annotation khác, bằng cách cung cấp thuộc tính cho @EnableGlobalMethodSecurity với jsr250Enabled = true cho phép sử dụng @RolesAllowed và securedEnabled = true cho phép sử dụng @Secured
* @RolesAllowed và @Secured chỉ định người dùng với vai trò hoặc quyền hạn nào đó có thể gọi tới method nhất định
```java
@Service
public class NameService {
  // chỉ ADMIN mới được gọi method
  // tương tự @Secured("ROLE_ADMIN")
  @RolesAllowed("ROLE_ADMIN")
  public String getName() {
    return "Fantastico";
  }
}
```
### Filtering (Pre- and postfiltering)
* Trong trường hợp không muốn chặn 1 method, nhưng đảm bảo các tham số giử đến nó tuân theo một số quy tắc hoặc trong tình huống muốn đảm bảo người gọi đến method chỉ nhận 1 phần của giá trị return.
  * Prefiltering: framework lọc giá trị của các tham số trước khi gọi method
  * Postfiltering: framework lọc giá trị returrn sau khi gọi method
* Với preauthorization, nếu tham số không tuân theo quy tắc uỷ quyền thì frameowrk không gọi method nào cả, với prefiltering, method vẫn được gọi, nhưng chỉ giá trị tuân theo quy tắc mới được đưa vào làm tham số cho method. Ex: 1 user chỉ bán được sản phẩm của họ, trong trường hợp user gọi đến method bán 1 danh sách sản phẩm, trong những sản phẩm có sản phẩm không thuộc về họ thì bị loại bỏ
* Filtering không ném ra exception nếu tham số hoặc giá trị trả về không tuân theo luật uỷ quyền. 
* Chỉ áp dụng tính năng lọc cho collections hoặc array. Chỉ sử dụng Prefiltering nếu method nhận tham số dưới dạng 1 array hoặc 1 collection. Với Postfiltering, chỉ áp dụng nếu method trả về 1 collection hoặc 1 array
#### Áp dụng Prefiltering để uỷ quyền method
* Vẫn sử dụng annotation @GlobalMethodSecurity và bật prePostEnabled
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig {
  @Bean
  public UserDetailsService userDetailsService() {
    var uds = new InMemoryUserDetailsManager();
    var u1 = User.withUsername("nikolai")
      .password("12345")
      .authorities("read")
      .build();
    var u2 = User.withUsername("julien")
      .password("12345")
      .authorities("write")
      .build();
    uds.createUser(u1);
    uds.createUser(u2);
    return uds;
  }
  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }
}

public class Product {
  private String name;
  private String owner; 
  // Omitted constructor, getters, and setters
}

@Service
public class ProductService {
  // Sử dụng annotation PreFilter
  // filterObject đại diện cho tham số truyền vào, có type là Product
  // chỉ cho phép product có owner là name của caller được thực thi
  @PreFilter("filterObject.owner == authentication.name")
  public List<Product> sellProducts(List<Product> products) {
  // sell products and return the sold products list
  return products; 
  }
}

@RestController
public class ProductController {
  @Autowired
  private ProductService productService;

  @GetMapping("/sell")
  public List<Product> sellProduct() {
    List<Product> products = new ArrayList<>();
    products.add(new Product("beer", "nikolai"));
    products.add(new Product("candy", "nikolai"));
    products.add(new Product("chocolate", "julien"));
    return productService.sellProducts(products);
  }
}
```
* Aspect đã thay đổi collection truyền vào, nhưng không phải trả về 1 instance mới của collection. Cùng 1 instance của collection, Aspect chỉ xoá đi elements không khớp với tiêu chí đã cho. Cẩm đảm bảo rằng tham số truyền vào không phải 1 collection bất biến nếu không quá trì lọc sẽ dẫn đến 1 runtime exception
```java
@RestController
public class ProductController {
  @Autowired
  private ProductService productService;

  @GetMapping("/sell")
  public List<Product> sellProduct() {
    // sử dụng List.of() trả về 1 instance bất biến của list => trả về exception 
    List<Product> products = List.of( 
    new Product("beer", "nikolai"),
    new Product("candy", "nikolai"),
    new Product("chocolate", "julien"));
    return productService.sellProducts(products);
  }
}
```
#### Áp dụng Postfiltering để uỷ quyền method
* Ví dụ về gọi đến method findProducts() trả về list products chỉ thuộc sở hữu của người gọi, sử dụng annotation @PostFilter
```java
@Service
public class ProductService {
  @PostFilter("filterObject.owner == authentication.name")
  public List<Product> findProducts() {
    List<Product> products = new ArrayList<>();
    products.add(new Product("beer", "nikolai"));
    products.add(new Product("candy", "nikolai"));
    products.add(new Product("chocolate", "julien"));
    return products;
  }
}

@RestController
public class ProductController {
  Autowired
  private ProductService productService;
  
  @GetMapping("/find")
  public List<Product> findProducts() {
    return productService.findProducts();
  }
}
```
#### Sử dụng filtering trong Spring Data
* Sử dụng các annotation @PreFilter và @PostFilter áp dụng cho layer repository để lọc dữ liệu là sai, với method findAll() mà không sử dung phân trang có thể dẫn đến OutOfMemoryError. Ngay cả khi dữ liệu đủ chứa trong heap thì việc lọc dữ liệu vẫn kém hiệu quả hơn nhiều so với truy vấn ngay từ đầu
* Làm thế nào để chỉ chọn dữ liệu yêu cầu ngay từ đâu thay vì lọc dữ liệu sau khi chọn => bằng cách cung cấp SpEL trực tiếp trong query sử dụng trong layer repository, thực hiện theo 2 bước:
  * Thêm 1 object loại SecurityEvaluationContextExtension vào Spring Context
  * Điều chỉnh query trong layer repository bằng các mệnh đề thứch hợp
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig {
  @Bean 
  public SecurityEvaluationContextExtension securityEvaluationContextExtension() {
    return new SecurityEvaluationContextExtension();
  }
  // Omitted declaration of the UserDetailsService and PasswordEncoder
}

public interface ProductRepository extends JpaRepository<Product, Integer> {

  // không nên sử dụng @PostFilter
  // @PostFilter("filterObject.owner == authentication.name")
 @Query("SELECT p FROM Product p WHERE p.name LIKE %:text% AND p.owner=?#{authentication.name}")
 List<Product> findProductByNameContains(String text);
}
```

# Config với Spring Security 5.7
* không sử dụng WebSecurityConfigurerAdapter
* các method ghi đè của WebSecurityConfigurerAdapter sẽ thay thế bằng các bean
  * method: *@Override void configure(HttpSecurity http)* => *@Bean SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http, ...)*
  * method: *@Override protected void configure(AuthenticationManagerBuilder auth)* => *@Bean AuthenticationManager authenticationManager(HttpSecurity http, ...)*
```java
@Bean
public AuthenticationManager authenticationManager(HttpSecurity http, AuthenticationProvider authenticationProvider) throws Exception {
// BCryptPasswordEncoder bCryptPasswordEncoder, UserDetailService userDetailService

  // return http.getSharedObject(AuthenticationManagerBuilder.class)
  //   .userDetailsService(userDetailsService)
  //   .passwordEncoder(bCryptPasswordEncoder)
  //   .and()
  //   .build();
  // đưa UserDetailsService và PasswordEncoder vào AuthenticationManager

  return http.getSharedObject(AuthenticationManagerBuilder.class)
    .authenticationProvider(authenticationProvider)
    .build();
  // đưa AuthenticationProvider vào AuthenticationManager
    
}

@Bean
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) {
  http.authorizeRequests().
    .mvcMatchers("/hello").hasRole("ADMIN")
    .anyRequest().authenticated();
  return http.build();
}

```
# Config với Spring Security 6.0 Spring Boot 3.0
* không sử dụng **.authorizeRequests()** thay thế bằng **.authorizationHttpRequest()**
* không sử dụng method **.mvcMatchers()** , **.antMatchers()**. để áp dụng hạn chế cho từng endpoint nữa, thay thế bằng **.requestMatchers()**
* method **.access()** để chỉ định authorities hay roles cho endpoint được thay thế, không còn nhận vào **SpEL**, nhận vào 1 object loại của interface **AuthorizationManager<RequestAuthorizationContext>**
    * **AuthorizationManager<T>** là 1 @FunctionalInterface với method: *AuthorizationDecision check (Supplier<Authentication> authentication, T object)*, 
    * **AuthorizationDecision** l class có 1 thuộc tính *granted* đại diện việc có cấp quyền hạn hay không.
    * *T object* ở trường hợp này đại diện cho *RequestAuthorizationContext*
    * 1 trong những class được SpringSecurity cung cấp, triển khai **AuthorizationManager<RequestAuthorizationContext>** là **WebExpressionAuthorizationManager** có thể sử dụng tương tự method **.access()** cũng nhận vào SpEL
* annotation **@EnableGlobalMethodSecurity** không dùng nữa, thay thế bằng **@EnableMethodSecurity** với cấu hình cho .prePostEnabled() đã được bật sẵn

```java
@Bean
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) {
    http.authorizationHttpRequest().
      .requestMatchers("/demo")
      // giống antMatchers, phân biệt "/demo" và "/demo/"
      .access(new WebExpressionAuthorizationManager("isAuthenticated()"));
      // ~ .access("isAuthenticated") 

    return http.build();
  }
```
