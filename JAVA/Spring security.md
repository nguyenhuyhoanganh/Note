## Các thành phần của Spring security
* **Authentication filter**: chặn các req giử đến
* **Authentication manager**: nhận trách nhiệm xác thực user
* **Authentication provider**: manager sử dụng để thực hiện logic xác thực
   - *User details service*: sử dụng để tìm kiếm user
   - *Password encoder*: sử dụng để xác thực mật khẩu
* **Security context**: sử dụng để lưu trữ thông tin về thực thể được xác thực

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
  - cấu hình trực tiếp UserDetailsService và PasswordEncoder trong phươnbg thức được cung cấp sẵn bởi WebSecurityConfigurerAdapter

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


## UserDetailsService
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