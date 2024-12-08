# sử dụng nginx revered proxy làm load balancing
## tổng quan về nginx revered proxy
- Reverse Proxy đóng vai trò là một trung gian giữa client (người dùng hoặc ứng dụng) và một hoặc nhiều server backend.
-  Cụ thể, thay vì client trực tiếp gửi yêu cầu đến server backend, chúng sẽ được gửi qua Nginx, và Nginx sẽ chuyển tiếp yêu cầu đó đến server backend thích hợp.
-  Sau đó, Nginx nhận phản hồi từ server backend và gửi lại cho client.
### các tính năng mà nginx proxy hỗ trợ
  + Load Balancing
  + Reverse Proxy
  +  Caching (Lưu cache)
  +  SSL/TLS Termination
  +  WebSocket Proxying
  +  Authentication (Xác thực)
  +  Rate Limiting (Giới hạn tốc độ)
  +  IP Blocking và Filtering (Chặn IP và lọc)
  +  Custom Headers (Tiêu đề tùy chỉnh)
  +  Load Balancing Health Checks (Kiểm tra sức khỏe của backend)
  +  Logging (Ghi log)
  +  Access Control (Kiểm soát quyền truy cập)
# Workflow
![image](https://github.com/user-attachments/assets/b303c175-742c-4ddd-bd52-afbe453c1e2f)

# Install nginx revered proxy
## Cài wordpress
- cài thêm 1 backend wordpress như đã làm ở phần trước.Tuy nhiên sẽ không cần cài database mà sử dụng chung 1 database
- cấu hình backend mới trỏ về database đã cấu hình ở phần trước
-  _lưu ý : nhớ mở port 3306 cho database_
## cài nginx revered proxy
- cài thêm 1 web server nginx làm nginx revered proxy
### cấu hình nginx revered proxy
- nginx sẽ thực hiện đọc file nginx.conf đầu tiên sau đó sẽ incluce sang conf.d
- file config nginx proxy vẫn sẽ cấu hình trong : /etc/nginx/conf.d
- tạo file config :
  ```
  vim /etc/nginx/conf.d/proxy.conf
  ```
- thay đổi file cấu hình như sau :
```
upstream backend_servers {    #định ngĩa một nhóm server backend
    server 192.168.32.133:443;
    server 192.168.32.134:443;
}

    server {
        listen 443 ssl;
        server_name test.com;

    # Đường dẫn đến chứng chỉ và khóa riêng
    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

        # Cấu hình SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
    ssl_prefer_server_ciphers on;



        location / {
            # Chuyển tiếp yêu cầu tới nhóm máy chủ backend
            proxy_pass https://backend_servers;

            # Thiết lập các header cần thiết cho reverse proxy
            proxy_set_header Host $host;    # Truyền thông tin domain tới backend
            proxy_set_header X-Real-IP $remote_addr;    # Truyền địa chỉ IP của client
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;    # Truyền thông tin của các proxy trung gian
            proxy_set_header X-Forwarded-Proto $scheme;    # Truyền giao thức (HTTP hoặc HTTPS) tới backend
        }
    }
```
Khi định tuyến, Nginx gửi các header sau đến backend để đảm bảo thông tin client được truyền chính xác:
```
X-Forwarded-For: Địa chỉ IP của client.
X-Real-IP: Địa chỉ IP thực của client.
Host: Tên miền gốc mà client gửi yêu cầu.
Connection: Thường là keep-alive để duy trì kết nối.
$host: Truyền domain name mà người dùng yêu cầu (ví dụ: example.com).
$scheme: Truyền giao thức gốc (HTTP hoặc HTTPS) mà người dùng sử dụng khi kết nối tới Nginx.
```
- lúc này bạn chỉ cần truy cập qua IP của nginx proxy. nginx sẽ định tuyến tới một trong hai backend
- sửa file etc host để truy cập qua domain
- _có thể thêm cấu hình sau để xem log các server đã loadbalacing chưa_
```
 log_format upstreamlog '$remote_addr - $remote_user [$time_local] '
                      '"$request" $status $body_bytes_sent '
                      '"$http_referer" "$http_user_agent" '
                      'to: $upstream_addr';
```
- câu lệnh xem log:
```
 sudo tail -f /var/log/nginx/access.log
```
- log access vào server nào sẽ có IP của server ở cuối cùng
### Các thuật toán loadbalancing trong nginx proxy 
##### Round Robin
- Yêu cầu được phân phối tuần tự lần lượt đến từng server trong nhóm backend.
- ưu điểm : Đơn giản và phù hợp cho các backend server có cấu hình và hiệu suất tương đương.
-  cấu hình :
  ```
upstream backend_servers {
    server 192.168.32.133;
    server 192.168.1.134;
}
```
- **_đây là thuật toán mặc định nếu không cấu hình gì nginx sẽ lựa chọn thuật toán này cho backend_**
##### Least Connections
- Yêu cầu được chuyển đến server có số lượng kết nối hiện tại ít nhất.
- Thích hợp cho các ứng dụng có thời gian xử lý không đều hoặc không đồng nhất.
- cấu hình : 
```
upstream backend_servers {
    least_conn;
    server 192.168.32.133;
    server 192.168.1.134;
}
```
##### IP Hash
- Dựa trên địa chỉ IP của client, Nginx luôn chuyển tiếp yêu cầu từ cùng một client đến cùng một server backend.
- Phù hợp cho các ứng dụng yêu cầu duy trì trạng thái (stateful- lưu trữ thông tin của Client trong Server), ví dụ như ứng dụng sử dụng session.
- cấu hình :
```
upstream backend_servers {
    ip_hash;
    server 192.168.32.133;
    server 192.168.1.134;
}
```
##### Weighted Round Robin
- Một biến thể của Round Robin, trong đó mỗi server được gán một trọng số (weight) để quyết định số lượng yêu cầu mà server đó xử lý. Server có trọng số cao hơn sẽ nhận nhiều yêu cầu hơn.
- Thích hợp khi các server có tài nguyên hoặc hiệu suất khác nhau.
- cấu hình :
```
upstream backend_servers {
    ip_hash;
    server 192.168.32.133 weight=3; # Server mạnh hơn, xử lý nhiều hơn
    server 192.168.1.134 weight=1;
}
```
- _**trên đây là 4 thuật toán chính hay sử dụng ngoài ra có các thuật toán khác**_
- Least Time (Chỉ hỗ trợ trong Nginx Plus)
- Random (Chỉ hỗ trợ trong Nginx Plus)
- Generic Hash
- _**khi nào chọn thuật toán nào**_
- Round Robin: Server backend đồng nhất về cấu hình và hiệu suất.
- Least Connections: Khi backend server xử lý các yêu cầu có thời gian xử lý không đồng đều.
- IP Hash: Khi ứng dụng yêu cầu duy trì phiên với cùng một server.
- Weighted Round Robin: Khi backend server có cấu hình tài nguyên khác nhau.
- Least Time (Nginx Plus): Yêu cầu tối ưu về thời gian phản hồi.
- Random: Đơn giản, phù hợp cho các trường hợp test hoặc cân bằng ngẫu nhiên.
# Luồng hoạt động 
###### Bước 1:
- Client gửi yêu cầu đến Nginx
- Người dùng truy cập website hoặc API qua URL (ví dụ: https://example.com).
- Yêu cầu HTTP/HTTPS được gửi đến máy chủ Nginx.
###### Bước 2:
- Nginx xử lý yêu cầu
- Nginx nhận yêu cầu và kiểm tra file cấu hình (nginx.conf hoặc các file site-specific).
- Nginx sử dụng module proxy_pass để quyết định chuyển tiếp yêu cầu đến backend nào.
###### Bước 3: Định tuyến tới Backend
- Dựa trên các quy tắc định tuyến, Nginx chọn backend phù hợp:
- Load Balancing: Nginx phân phối yêu cầu giữa các backend theo thuật toán (mặc định là round-robin).
- Sticky Session: Đảm bảo yêu cầu của cùng một client luôn được xử lý bởi cùng một backend.
- Health Check: Nếu một backend không khả dụng, Nginx sẽ tự động chuyển sang backend khác.
- Nginx chuyển tiếp yêu cầu đến backend qua HTTP hoặc HTTPS.
###### Bước 4: Backend xử lý yêu cầu
- Backend nhận yêu cầu từ Nginx, xử lý (ví dụ: truy vấn cơ sở dữ liệu, trả về nội dung tĩnh hoặc động).
- Backend gửi phản hồi HTTP trở lại Nginx.
###### Bước 5: Nginx gửi phản hồi tới Client
- Nginx nhận phản hồi từ backend và chuyển tiếp đến client.
- Nếu cần, Nginx thực hiện thêm các bước như nén dữ liệu hoặc caching trước khi trả phản hồi.
- **_có thể thêm các cấu hình health check và caching để tối ưu hóa truy cập_**
## giao thức định tuyến tới backend
- trong bài này chúng ta đang sử dụng proxy_pass để định tuyến tới backend
- proxy_pass định tuyến thông qua URL xử dụng http hoặc https ( do vậy hoạt động ở layer 7)
- ngoài ra còn các giao thức định tuyến khác cho các ứng dụng phù hợp :
  
|Giao thức|	Câu lệnh Nginx|	Ứng dụng backend|
|:--------|:--------------:|--------------:|
|HTTP|	proxy_pass|	Web server, ứng dụng HTTP|
|HTTPS|	proxy_pass|	Web server, ứng dụng HTTPS|
|FastCGI|	fastcgi_pass|	PHP-FPM, Python qua FastCGI|
|uWSGI|	uwsgi_pass|	Python ứng dụng qua uWSGI|
|SCGI|	scgi_pass|	Backend sử dụng giao thức SCGI|
|GRPC|	grpc_pass|	Microservices hoặc ứng dụng gRPC|
|Unix|	proxy_pass	|Bất kỳ backend nào qua Unix socket|
- _có thể định tuyến ở layer 4 bằng TCP/UDP nếu sử dụng khối stream_
## Một server chưa nhiều website
- nếu một server chưa nhiều website chỉ cần định tuyến yêu cầu của người dùng tới từng domain một
# tối ưu hóa cấu hình 
- trong một số trường hợp người dùng đang gọi tới nginx proxy và nginx proxy đã xử lý SSL và domain name vì vậy trong cấu hình của backend không cần cấu hình SSL và domain name
- tuy nhiên với wordpress có thể cần cấu hình chính xác domain trong cơ sở dữ liệu hoặc trong file cấu hình (wp-config.php). vì vậy ở backend vẫn cần cấu hình domain name
- SSL ở trong trường hợp này đang sử dụng cho cả 2 việc là nginx proxy mã hóa với client và nginx proxy với backend
  + việc nginx với client sử dụng SSL là cần thiết
  + nginx proxy với backend có thể chỉ cần sử dụng http nếu đã nằm trong cùng một mạng bảo mật thì khi đó backend không cần cấu hình SSL nữa
  + nếu sử dụng proxy_pass https tới backend thì backend cũng cần cấu hình SSL để chấp nhận kết nối https từ nginx proxy
- _ở  đây tôi sử dụng toàn bộ là kết nối https nên mỗi server cần cấu hình SSL_
- Mặc dù Nginx đã xử lý SSL và domain name, nhưng trong một số tình huống backend vẫn cần thông tin về domain:
  + Ứng dụng yêu cầu xác định domain để cấu hình
  + Tạo các liên kết tuyệt đối

