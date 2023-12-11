# openfeign : thay thế webclient hay resttemplate để gọi đến các api của các service khác từ 1 service trong microservice

- thêm dependency : `OpenFeign`
- thêm annotation `@EnableFeignClient` trong class Application
- tạo 1 package client ở service muốn gọi api tới các service khác
- tạo từng interface tương ứng với từng service muốn gọi tới
- thêm annotation `@FeignClient(name = " ", url = " " )`
  - `name` : tên thuộc tính `spring.application.name` của service muốn gọi
  - `url`: tên url, port của service muốn gọi
- thêm method vào interface. Method cùng tên, tham số, mapping và uri tương ứng với method muốn gọi đến trong controller của service khác
  - tham số của method có thể nhận vào 1 số object DTO ở bên service khác => có thể tạo class tương ứng với object cần
- Autowired interface vừa tạo trong controller
- sử dụng gọi đến các method của interface cho loggic tương ứng

# circuit breaker : đảm bảo microservice vẫn tiếp tục hoạt động nếu 1 service bị sập bằng cách fallback lại 1 method khác thực thi tiếp, tránh trả về exception

- resilience4J : là 1 triển khai của partten circuit breaker
- thêm dependency: `Resilience4J `
- cấu hình thêm trong application.properties:
  - `feign.circuitbreaker.enable=true` : bật circuit breaker
  - `feign.client.config.default.connect-timeout=5000` : thời gian bắt đầu gọi đến khi kết nối được
  - `feign.client.config.default.read-timeout=5000` : thời gian kết nối đến khi trả về response
- tạo class implement interface đã được cấu hính với `@FeignClient`
- ký hiệu class với `@Component`
- từng method Override lại của interface sẽ thực thi logic cho việc fallback (log ra lỗi, ...)
- thêm thuộc tính vào `@FeignClient`
  - `fallback` : class implement interface

# config server - config client

## config service: chứa tất cả cấu hình của toàn bộ microservice

- tạo 1 service dùng riêng cho việc config
- thêm dependency: `Config Server`
- thêm annotation `@EnableConfigServer` vào class Application
- 1 số cấu hình
  - `server.port=8888` : port chạy config server
  - `spring.profiles.active=native` : mặc định config server sử dụng git để lưu cấu hình, thay đổi bằng native sẽ lưu ở trong file
  - `spring.cloud.config.server.native.search-locations= classpath:/shared` : khai báo đường dẫn lưu các file cấu hình, lưu ở folder shared trong resources
- thêm folder shared trong resource. folder chứa cấu hình cho các service
- thêm vào shared các file properties. Các file properties có tên tương ứng với thuộc tính `spring.application.name` của từng service
- đưa tất cả các cấu hình trong file properties của từng service sang file propertiees mới (ngoại trừ cấu hình về `spring.application.name`)
- thêm 1 file `application.properties` trong shared chứa cấu hình chung cho tất cả các service (không phải file application.properties mặc định cấu hình cho config server)

## config client: lấy config về từ config server

- thêm thuộc tính vào application.properties của các service client
  - `spring.config.import=configserver:http://localhost:8888` : đường dẫn đến config server
  - `spring.cloud.config.fail-fast=true` : yêu cầu đọc được cấu hình từ server config mới cho phép khởi chạy service

# discovery ~ eureka (netflix) ~ service registry

## discovery server (service registry): là 1 server quản lý tất cả service. chứa thông tin về service như ip, address, hostname, port, ..., giúp dễ dàng xác định các service thông qua discovery server

- tạo project chứa discovery server
- thêm dependency: `Config Client`, `Eureka Server`
- thêm annotation `@EnableEurekaServer` vào class Application
- thêm các thuộc tính cho properties:
  - các thuộc tính về application name, địa chỉ config server
  - `eureka.client.register-with-eureka=false` : không đăng ký chính nó như 1 client
  - `eureka.client.fetch-registry=false` : không cần fecth thông tin đăng ký của chỉnh nó nữa
  - cấu hình các thông tin về port, ... ở trong confige server cho nó
    => cung cấp 1 giao diện về eureka server hiển thị các service đăng ký với server và các tên, địa chỉ, trạng thái của từng server tương ứng

## discovery client

- giúp đăng ký service với discovery server, truyền heartbeat về discovery server để discovery server xác định trạng thái của service (sống, chết, dừng lại).
- giúp giao tiếp với các service khác cùng đăng ký không cần thông qua ip, hostname, port
- hỗ trợ load-balancing lựa chọn service tương thích phù hợp nhất
- không cần đăng ký config server làm 1 discovery client
- các cài đặt:
  - thêm dependency vào các discovery client: `Eureka Discovery Client`
  - thêm `@EnableDiscoveryClient` trong class Application
  - thêm thuộc tính trong file properties (thêm trực tiếp cho mỗi service hoặc thêm trên file cấu hình riêng của mỗi service trên server config, hoặc thêm trong file cấu hình chung của các service trong server config)
    - `eureka.client.service-url.defaultZone=http://localhost:9999/eureka/` : đăng ký với discovery server qua url
    - `eureka.instance.prefer-ip-address=true` : sử dụng địa chỉ IP để đăng ký thay vì hostname
  - loại bỏ cấu hình hardcode về url của từng service trong `@FeignClient`, chỉ cần cung cấp về name của service
- khi thêm eureka discovery client vào các service, sẽ tự động thêm thư viên spring-cloud-loadbalancer

# gateway: đóng vai trò như cổng giao tiếp giữa client và microservice, giúp định tuyến các route tới API của từng micro service ẩn phía sau (bảo vệ server khỏi các tấn công mạng, ẩn địa chỉ ip máy chủ thực tế)

- client gọi lên gateway, không biết được có những service nào đang chạy, chỉ gọi lên đường dẫn của gateway, gateway dựa đường dẫn, path của đường dẫn, chỉ định request đến chính xác service thực hiện
- gateway cung cấp bảo mật, theo dõi/ đo lường, phục hồi với các service trong hệ thống
  > Client => Gateway handler mapping => Gateway web handler => Filter => Service
- client gọi lên đường dẫn, mapping check đường dẫn có map với đường dẫn của root đã cấu hình trước, dẫn đến webhandler đẩy request thông qua các filter để tuỳ biến hoặc rewrite lại request trước khi chạy đến service
- tính năng:
  - match routes theo 1 số thuộc tính của request
  - dùng predicate để làm biểu thức điều kiện cho route
  - dùng filter lọc, cắt bỏ route trước khi gọi đến service phía sau
- gateway sẽ đăng ký làm 1 client của discovery server
- cách cài đặt:

  - tạo project gateway:
  - thêm dependencies: `Config Client`, `Gateway`, `Eureka Discovery Client`
  - thêm annotation `@EnableDiscoveryClient` để đăng ký với discovery server
  - thêm các thuộc tính properties:
    - `spring.application.name=gateway-service`
    - `spring.config.import`
    - `server.port`
    - `server.servlet.context-path=/`
    - `spring.cloud.gateway.discovery.locator.enabled=true` : gateway tự động liên kết với discovery client tạo các route động dựa trên các service đã đăng ký trên discovery server

- với chỉ cấu hình `spring.cloud.gateway.discovery.locator.enabled=true`: có thể từ url của gateway trỏ đến các service khác khi thêm phần part là application.name của service đó
  > Ex: gateway trên local, port 8080 gọi đến service với `spring.application.name=account-service`=> url: http://localhost:8080/ACCOUNT_SERVICE/
- cấu hình để thay đổi path không theo `spring.cloud.gateway.discovery.locator.enabled=true`:
  - `spring.cloud.gateway.routes[0].id=user-route` : đặt id định danh cho route
  - `spring.cloud.gateway.routes[0].uri=lb://account-service` : trỏ đến account-service sử dụng loadbalancer (lb)
  - `spring.cloud.gateway.routes[0].predicates=Path=/user/**` : sử dụng path bắt đầu với /user thay vì /ACCOUNT_SERVICE
  - `spring.cloud.gateway.routes[0].filters[0]=StripPrefix=1` : trước khi gọi đến account-service thì loại bỏ phần path thừa /user

# actuator: cung cấp các endpoint thông tin về trạng thái và quản lý ứng dụng trong quá trình runtime => giúp kiểm tra và thu thập thông tin cơ bản về hiệu suất và sức khoẻ của ứng dụng

- 1 số tính năng:
  - /health : kiểm tra trạng thái sức khoẻ của ứng dụng
  - /info : cung cấp thông tin về ứng dụng
  - /metrics : cung cấp thông số về hiệu suất như lượng request, thời gian xử lý, ...
  - /env: xem và thay đổi các thuộc tính môi trường
  - /loggers: xem điều chỉnh cấu hình ghi log
  - /beans: hiển thị danh sách bean
  - /mappings: liệt kê các URL mapping
    ...
- cài đặt:
  - thêm dependency: `Spring Boot Actuator`
  - thêm thuộc tính vào file cấu hình:
    - `management.endpoints.web.exposure.include=*` : cho phép bất tất cả các endpoint của actuator
    - `management.endpoint.health.show-details=always` : hiển thị chi tiết sức khoẻ của ứng dụng cho mọi người
    - `management.endpoint.health.probes.enabled=true` : cấu hình cho phép kiểm tra sức khoẻ úng dụng
- truy cập các endpoint: /actuator/health, /actuator/metrics, ...

# OPS : codecentric's Spring Boot Admin (Server) - codecentric's Spring Boot Admin (Client)

- kết hợp actuator cung cấp giao diện sử dụng Vue.js để dễ dàng kiểm tra health, metrics, ...
- client đăng ký với admin server thông qua `codecentric's Spring Boot Admin (Client)` hoặc `Eureka Discovery Client`
- cài đặt:
  - tạo 1 project mới
  - thêm dependency: `codecentric's Spring Boot Admin (Server)`, `Config Client`, `Eureka Discovery Client`
  - thêm annotation trên class Application `@EnableAdminServer`, `@EnableDiscoveryClient`
  - thêm cấu hình trong file properties:
    - đặt tên cho application
    - cấu hình để đọc lại cấu hình từ config server
    - cầu hình về port và context-path
  - thêm dependency `Spring Boot Actuator` vào các service client và cấu hình actuator cho chúng (có thể cấu hình cho cấu hình chung trong config server)

## sử dụng send notification để giử email thông báo về tình trạng các service của Spring Boot Admin (Server)

- thêm dependency `Java Mail Sender` vào admin server
- thêm cấu hình để giử mail:

```yml
# cấu hình để giử mail
spring.mail.default-encoding=UTF-8
spring.mail.host=smtp.gmail.com
spring.mail.username=your gmail
#Create your app password with other (custom name)
#https://myaccount.google.com/u/1/apppasswords
spring.mail.password=apppasswords
spring.mail.port=587
spring.mail.protocol=smtp
spring.mail.test-connection=false
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true

#Email receive notification
# cấu hình cho admin có thể giử mail thông báo 1 số trạng thái như khởi chạy 1 service mới
spring.boot.admin.notify.mail.enabled= true
# liệt các email nhận thông tin trạng thái
spring.boot.admin.notify.mail.to=your list gmail
```

# sleuth: theo dấu vết 1 request khởi chạy, kết hợp log để biết lỗi ở đâu trong microservice

- sử dụng trace id và span id để theo dấu request, thread hoặc jod trong ứng dụng
- tích hợp với Logback và SLF4J để xem log
- tích hợp OpenFeign thêm trace/span vào request header để thao dấu luồng trong microservice
- TRACE: giống 1 requet, mỗi requet có 1 trace id fuy nhất trong toàn bộ luồng chạy, TRACE bao gồm nhiều SPAN
- SPAN: giống như đơn vị nhỏ của TRACE, có mỗi span id duy nhất cho từng đơn vị nhỏ như một thread
- log message có dạng như: [application name, traceId, spanId]
- cài đặt:
  - thêm dependency: `Sleuth` cho các service nhỏ
  - cấu hình cho các service:
    - `logging.level.root=info` : log level
    - `logging.file.name=app.log` : tạo 1 file app.log lưu log
    - `logging.file.path=...` : cấu hình dường dẫm đến file log, mặc định là trong resouce của app
    - `logging.logback.rollingpoolicy.max-file-size=2MB` : cấu hình chính sách cho file log tối đa 2MB nếu quá sẽ backup lại

## zipkin: kết hợp sleuth để hiển thị giao diện ui theo dấu 1 request

- có thể cấu hình lưu log ở database, mặc định là lưu in-memory
- cài zipkin server theo hướng dẫn trên https://zipkin.io/pages/quickstart.html
- cài zipkin client để server đọc dữ liệu của slueth
  - thêm dependency `Zipkin Client` cho từng service
  - thêm thuộc tính: `spring.zipkin.base-url=http://localhost:9411/` là đường dẫn đến zipkin, mặc định không thêm sẽ ở cổng 9411

# Oath 2

- 4 vai trò:
  - user
  - client
  - resource server
  - oauth server
- luồng chung:
  1. client request đến oauth server
  2. oauth server xác minh client xem đã đăng ký với nó chưa
  3. user xác minh với oauth server
  4. oauth server trả vể access token cho client
  5. client request đến resource server kèm access token
  6. resource xác minh token với oauth server
  7. oauth server xác nhận token, resource trả về resource cho client
- authorization code grant type:

  1. user click login trên client
  2. client request authorization code trên oath2 server
  3. oauth2 server verify client
  4. oauth2 server yêu cầu user login
  5. user xác thực với oauth2 server
  6. oauth2 trả authorization code về client
  7. client sử dụng authorization code và client secrect yêu cầu access token với oauth2 server
  8. oauth2 server validate và chuyển access token cho client
  9. client request resource từ resource server với access token
  10. resource server validate token với oauth2 server hoặc từ validate
  11. resource server trả resource về client

- config server > discovery server > oauth2 server > các resource server
- resource server cũng phải đăng ký chính nó là 1 client của oauth2 server
- các resource server cần cấu hình thêm OpenFeign, thêm thuộc tính `configuration` vào `@FeignClient`. thuộc tính nhận 1 class config, chứa 1 bean `RequestInterceptor` chặn request gọi từ các resource tới các resource khác để thêm thông tin vể client id và client secrect (code ở section 3)
