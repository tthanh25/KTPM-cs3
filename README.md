# Health Endpoint Monitoring
## Sơ lược về bài toán: 
 Một đơn vị cần xây dựng hệ thống giám sát "sức khoẻ" của các services trong hạ tầng của họ. Sử dụng những kiến thức về Health Endpoint Monitoring để xây dựng cho mình một trang web quản lý nhưng thông tin sau:
 - Tình trạng (up/down) của các container
 - Tình trạng (up/down) của các endpoint
 - Hiển thị realtime các tài nguyên như RAM/Bộ nhớ/Băng thông mạng của máy chủ
 - Xây dựng biểu đồ lưu lượng truy cập.
## Kiến trúc hệ thống: 
 - Container 1: Chạy một API cung cấp thông tin giá vàng tại Việt Nam, chạy trên cổng 3008. API Giá vàng:
 
   http://giavang.doji.vn/ (Doji)
 - Container 2: Chạy một API cung cấp thông tin giá ngoại tệ so với VNĐ, chạy trên cổng 3009. API Giá ngoại tệ:
 
   https://www.vietcombank.com.vn/vi-VN/To-chuc/Doanh-Nghi%E1%BB%87p/KHTC---Ty-gia (Vietcombank)
 - cAdvisor: Open source của Google phân tích tổng quan các container.
 - Prometheus: Thu thập các chỉ số từ các container và hệ thống máy chủ (CPU, RAM, băng thông, v.v.).
 - Grafana: Hiển thị các dữ liệu thu thập được từ Prometheus thông qua dashboard tùy chỉnh, bao gồm cả tình trạng container và các biểu đồ lưu lượng truy cập.
## Các bước cài đặt:
 *Lỗi gặp trên Window Sử dụng WSL và DockerDesktop có dạng: * 
 
 ```cadvisor | E0823 20:34:34.010493 1 manager.go:1084] Failed to create existing container: /docker/{tên container ví dụ: f8b0931d2a1809803906b9e25bb8285438ccabfbfc96e8a7b82cdd29d38a15d9}: failed to identify the read-write layer ID for container "f8b0931d2a1809803906b9e25bb8285438ccabfbfc96e8a7b82cdd29d38a15d9". - open /var/lib/docker/image/overlay2/layerdb/mounts/f8b0931d2a1809803906b9e25bb8285438ccabfbfc96e8a7b82cdd29d38a15d9/mount-id: no such file or directory``` 
 
 Lí do: cAdvisor bị lỗi khi xác định vị trí của container. Nhưng vẫn nhận diện được docker driver =)))
 
 => Chuyển sang **Ubuntu (22.04)**
 
Cần cài đặt để tránh lỗi về quyền hạn (Permission dennied)

Cách 1: Sử dụng sudo cho mọi câu lệnh. 
Cách 2: 
  - Thêm group docker: ```sudo groupadd docker```
  - Thêm user hiện tại vào group: ```sudo usermod -aG docker $USER```
  - Check user hiện tại có các trong các group nào: ```groups```
  - Restart Ubuntu.
  - Bạn cần thêm một số bước để thêm quyền hạn đặc biệt khi chạy để có thể đọc ghi một số file đặc biệt:
    + ```sudo chown root:docker /var/run/docker.sock```
    + ```sudo chown "$USER":"$USER" /home/"$USER"/.docker -R```
    + ```sudo chmod g+rwx "$HOME/.docker" -R```
    
 
 **Các triển khai:**

**Phần 1:**
  - Clone github repository: ```git clone https://github.com/tthanh25/KTPM-cs3```
  
  - Chuyển tới thư mục KTPM-cs3: ```cd .\KTPM-cs3\```

  - Tải về các thư viện cần thiết: ```npm install```

  - Xây dựng các Image và chạy các Container: ```docker-compose up -d```
  
  - Đợi quá trình hoàn tất, các bạn có thể tải Docker Desktop để có thể tiện xử lí và theo dõi.

  - Lệnh để kiểm tra trạng thái các container: ```docker stats```

![image](https://github.com/user-attachments/assets/b0a3a780-633b-42ff-8d50-20e52d2fb1a9)

  P/S: Khi các bạn gõ lệnh ```docker stats``` thì các thông tin về RAM,CPU,Network,... ở terminal đã được hiện ra rồi. Muốn chi tiết và trực quan hơn nữa chúng ta sẽ sử dụng Prometheus và Grafana để quan sát và quản lí chi tiết các thông số hơn.

**Phần 2:** 
 - Kiểm tra các API: Truy cập ```http://localhost:3008/api/gold-price``` , ```http://localhost:3008/metrics```
  và ```http://localhost:3009/api/foreign-currency```, ```http://localhost:3009/metrics```
 trang ```/api/*``` biểu diễn các dữ liệu về API giá vàng và ngoại tệ.
 trang ```/metrics``` chứa các thông số về hệ thống, các request,...
 
 - Prometheus: Truy cập tại ```http://localhost:9090```
 - Grafana : Truy cập tại ```http://localhost:4000```
 - Sau khi truy cập Grafana các bạn nhập tài khoản và mật khẩu đều là "admin". Sau đó sẽ yêu cầu bạn đổi mật khẩu mới nên skip cũng được.
 
 - Mình đã xây dựng dashboard nên các bạn vào dashboard -> main để theo dõi các container trong hệ thống cần theo dõi nha.
![image](https://github.com/user-attachments/assets/156347e8-7093-47e8-9a6b-e7665ed7e396)
![image](https://github.com/user-attachments/assets/21cfe4c9-e97b-4bb4-9cb8-32d91bca9bdd)

**Phần 3:**
 - Thực hành kiểm thử hiệu năng với Gatling:
   + Gọi 10000 request với 100 VUser khác nhau.
   + Các biểu đồ chi tiết về các request tới API về status code, lượng request,...
   + Các docker chứa API Endpoint sẽ chạy với thông số từ 10-15% CPU, MEMORY...
   + Kịch bản test: Các bạn cài đặt Gatling trên ubuntu.
     * Cài đặt gói JDK 8 --Java SE Development Kit 8-- (chỉ hỗ trợ bản này):
     ```sudo apt install openjdk-8-jdk -y```
     * Tải gói Gatling về và giải nén:
     ``` wget https://repo1.maven.org/maven2/io/gatling/highcharts/gatling-charts-highcharts-bundle/3.2.0/gatling-charts-highcharts-bundle-3.2.0-bundle.zip```
     ``` unzip gatling-charts-highcharts-bundle-3.2.0-bundle.zip```
     * Tạo 2 file test về goldprice.scala và foreigncurrency.scala:
    
     * ```package test

import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class ApiTest extends Simulation {

  // Cấu hình HTTP
  val httpProtocol = http
    .baseUrl("http://localhost:3008") // Địa chỉ API
    .acceptHeader("application/json") // Header chấp nhận
    .contentTypeHeader("application/json") // Header nội dung

  // Kịch bản gửi request
  val scn = scenario("API Test Scenario")
    .repeat(10000) { // Lặp lại 10000 lần
      exec(http("Get Data")
        .get("/your-endpoint") // Thay "/your-endpoint" bằng endpoint thực tế của bạn
        .check(status.is(200))) // Kiểm tra status code
    }

  // Thiết lập tải
  setUp(
    scn.inject(atOnceUsers(100)) // Tải ngay lập tức 100 user
  ).protocols(httpProtocol)
} ```
     * ```package test

import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class ForeignCurrencyTest extends Simulation {

  // Cấu hình HTTP
  val httpProtocol = http
    .baseUrl("http://localhost:3009") // Địa chỉ API
    .acceptHeader("application/json") // Header chấp nhận

  // Kịch bản gửi request
  val scn = scenario("Foreign Currency API Test Scenario")
    .repeat(10000) { // Lặp lại 10000 lần
      exec(http("Get Foreign Currency")
        .get("/api/foreign-currency") // Endpoint thực tế
        .check(status.is(200))) // Kiểm tra status code
    }

  // Thiết lập tải
  setUp(
    scn.inject(atOnceUsers(100)) // Tải ngay lập tức 100 user
  ).protocols(httpProtocol)
} ```
     * Các bạn chạy lệnh ```./bin/gatling.sh```
     * Chọn số ứng với 2 file goldprice.scala và foreigncurrency.scala  
     * Ctrl + C để hủy lệnh.
