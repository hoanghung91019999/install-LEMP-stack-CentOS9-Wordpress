# Install-LEMP-stack-CentOS9-Wordpress
## environments
- centosOS9
- Mariadb-server
- php-fpm
- wordpress newversion
## install centosOS9
1. download ISO theo đường link : https://www.centos.org/download/
2. install ISO lên vmware
3. hướng dẫn cài và install trong vmware theo đường link : https://www.youtube.com/watch?v=hTe4oVQpUH0&t=684s
## install nginx version: nginx/1.20.1
```
sudo yum install nginx
```
```
nginx -v## kiểm tra version vừa cài
```
- Mặc định nginx sẽ chạy trên cổng 80 (http://your_IP)
### Các Chức Năng Chính của Nginx
- Web Server: Dùng để phục vụ các tài nguyên tĩnh như HTML, CSS, JavaScript, hình ảnh, v.v.
Đặc biệt hiệu quả trong việc phục vụ tệp tĩnh nhờ vào kiến trúc bất đồng bộ và mô hình event-driven.
- Reverse Proxy: Nginx có thể đóng vai trò là reverse proxy, tức là nhận các yêu cầu từ client và chuyển tiếp chúng đến một hoặc nhiều máy chủ backend (ví dụ, ứng dụng web PHP, Python, hoặc các dịch vụ khác).
- Load Balancer: Phân phối yêu cầu đến nhiều máy chủ backend để cân bằng tải và cải thiện độ ổn định của ứng dụng. Nginx hỗ trợ nhiều thuật toán load balancing như round-robin, least connections, IP hash.
- Cache: Nginx hỗ trợ cache cho các tài nguyên tĩnh và các kết quả động, giúp giảm tải cho server backend và tăng tốc độ phản hồi.
- SSL/TLS Termination:Nginx có thể đảm nhận việc mã hóa và giải mã SSL/TLS, giúp giảm tải cho các máy chủ backend và cung cấp một điểm quản lý duy nhất cho các kết nối HTTPS.
## Bật service và cho phép cổng firewall
- start dịch vụ nginx
```
systemctl start nginx
```
```
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd –reload
```
## Tạo vitualhost
- Mục đích : Hỗ trợ nhiều trang web trên một máy chủ (Multiple Websites on One Server): Một trong những mục đích chính của VirtualHost là cho phép bạn chạy nhiều website trên cùng một máy chủ.
- Ví dụ, bạn có thể có một website chạy tại www.site1.com và một website khác chạy tại www.site2.com, tất cả trên cùng một máy chủ và cùng một địa chỉ IP. VirtualHost giúp máy chủ phân biệt và xử lý các yêu cầu từ các tên miền khác nhau, và phục vụ các nội dung tương ứng.
- thực tế là bạn sẽ cấu hình một server block (khối máy chủ) trong file cấu hình của Nginx

### Tạo vitual host chạy http trên nginx 
- file cấu hình của nginx sẽ nằm trong : /etc/nginx/conf.d/
- Giả sử bạn muốn tạo VirtualHost cho một website có tên miền là test.com và thư mục gốc là /home/www/test.com
  
- Tạo 1 file cấu hình trong /etc/nginx/conf.d/
```
Sudo vim /etc/nginx/conf.d/test.com.conf
```
- Thêm cấu hình server block:
```
server {

    listen 80;  #lắng nghe yêu cầu trên port80 hhtp
    
    server_name test.com ;
    
    root /home/www/test.com;  #thư mục gốc chứa các tệp tĩnh như HTML, CSS, JavaScript, hình ảnh, video và các tài nguyên web khác.
    
    index index.html index.htm index.php; #thứ tự yêu tiên đọc file
    
    access_log /var/log/nginx/example.com.access.log;
    
    error_log /var/log/nginx/example.com.error.log;
    
    location / {			#trong Nginx là một chỉ thị được sử dụng để xử lý tất cả các yêu cầu có đường dẫn bắt đầu bằng /.
    
        try_files $uri $uri/ =404;  # kiểm tra và trả về tệp yêu cầu nếu có, nếu không thì trả về lỗi 404.
	
    } 	

```
- Tạo thư mục cho website và thêm tệp html hoặc php vào
```
Sudo mkdir -p /home/www/test.com
```
- Tạo 1 file trong test.com là index.html đơn giản ( search trên mạng )
- Phân quyền cho user nginx có quyền thực thi folder
```
chown -R nginx:nginx /home/www/test.com
chmod 755 /home/www/test.com
```
### Tạo vitual host chạy https 

- Tạo SSL tự ký Cài mod_ssl và openssl
```
yum install mod_ssl openssl -y
```
- Tạo thư mục lưu chứng chỉ và khóa (private key và public key) : 
```
sudo mkdir /etc/nginx/ssl
```
- Tạo chứng chỉ SSL và khóa riêng self-signed : 
```
sudo openssl req -x509 -newkey rsa:4096 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt -days 365
```
- điền PEM passpahre
- bỏ passphare
```
openssl rsa -in /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.key
```
- Chỉnh sửa file cấu hình vitualhost đã cấu hình http
```
sudo vim /etc/nginx/conf.d/test.com.conf
```
- file config : 
```
server {
    listen 443 ssl;
    
    server_name test.com www.test.com;

    ssl_certificate /etc/nginx/ssl/nginx.crt;    
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
    ssl_prefer_server_ciphers on;

    access_log /var/log/nginx/test.com.access.log;
    error_log /var/log/nginx/test.com.error.log warn;

    root /home/www/test.com;    
    index index.html index.htm;

    location / {	# khối locaiton này sẽ xử lý các yêu cầu đến thư mục gốc kiểm tra xem thư mục có tồn tại hay không nếu không trả về lỗi 404
        try_files $uri $uri/ =404;
    }

    error_page 497 https://$host$request_uri; #xử lý lỗi 497, chuyển hướng ng dùng từ http sang https
    
}

server {		#chuyển hướng vĩnh viễn (301 redirect) từ HTTP sang HTTPS.
    listen 80;  
    server_name test.com www.test.com;
    return 301 https://$host$request_uri; #host đại diện cho tên miền người dùng yêu cầu (ví dụ: test.com), và $request_uri là phần đường dẫn yêu cầu(ví dụ: /index.html)
}
```
- _Restart lại nginx sau đó truy cập vào web qua IP_
- _sửa file etc host trên hệ điều hành để truy cập qua domain_
- **_lưu ý : tắt SElinux không sẽ không truy cập được_**

# install mariadb
## Thành phần kiến trúc của maridb
- MariaDB Server: Thành phần này là phần chính của MariaDB, bao gồm server cơ sở dữ liệu và các công cụ điều khiển cơ bản.
- Client Utilities: Bao gồm các công cụ khách (client) như mysql (CLI client để kết nối với MariaDB) và mysqladmin (công cụ để quản lý server MariaDB).
- MariaDB Libraries: Thư viện phần mềm được sử dụng để phát triển các ứng dụng tương tác với MariaDB.
- Storage Engines: Các engine như InnoDB, MyISAM, Aria v.v. được cài đặt để hỗ trợ các kiểu lưu trữ dữ liệu.
- MariaDB Configuration Files: Các tệp cấu hình của MariaDB (chẳng hạn như my.cnf).
- Logs: Các tệp log như error.log, slow-query.log, và general-log được sử dụng để ghi lại các hoạt động của server.
### các thư mục trong mariadb
|thư mục		   |Nội dung|
|:------------------------ |:------------------------|
|/usr/bin/|	Công cụ client và các lệnh MariaDB|
|/usr/libexec/mysql/|	Tệp thực thi và các script hỗ trợ MariaDB|
|/etc/my.cnf hoặc /etc/my.cnf.d/|	Tệp cấu hình chính của MariaDB|
|/var/lib/mysql/|	Dữ liệu cơ sở dữ liệu của MariaDB|
|/var/log/mariadb/|	Các tệp log của MariaDB|
|/etc/init.d/ hoặc /usr/lib/systemd/system/|	Scripts và service để quản lý MariaDB|
|/var/run/mysqld/|	PID và socket của MariaDB|
|/usr/share/mysql/|	Các tệp hỗ trợ, tài liệu của MariaDB|
|/var/cache/mysql/|	Cache của MariaDB|

## cài đặt mariadb trên centos9
- installl
```
sudo dnf install mariadb -y
```
- đặt mật khẩu cho database: 
```
mysql_secure_installation
```
- truy cập vào database
```
mariadb -u root -p
```
#### Tạo database và user
_Tạo database_
```
CREATE DATABASE wordpress;
```
_Tạo user_
```
CREATE USER 'wordpress'@% identified by 'password';
```
_Cấp quyền cho user vào database_
```
grant all privileges on *%* to 'wordpress'@localhost;
```
_Xác nhận_
```
flush privileges;
```
_kiểm tra người dùng đã tạo_
```
SELECT user, host FROM mysql.user;
```
_kiểm tra quyền người dùng_
```
SHOW GRANTS FOR 'username'@'hostname';
```
_điền username và hostname_


# Cài php-fpm
- PHP-FPM (FastCGI Process Manager) là một FastCGI (giao thức giữa web server và các ứng dụng động) dành riêng cho PHP. Trong mô hình web server WordPress, PHP-FPM có nhiệm vụ xử lý các yêu cầu PHP từ nginx hoặc Apache (tùy theo cấu hình của bạn) và trả lại kết quả cho web server, giúp website hoạt động chính xác.

- Cài đặt kho EPEL và Remi
```
sudo dnf install -y epel-release
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```
- Bật kho Remi và cài đặt PHP-FPM
```
sudo dnf module list php
sudo dnf module enable php:remi-8.2 -y
sudo dnf install -y php-fpm php-cli php-mysqlnd
```
- Kiểm tra và cấu hình PHP-FPM
```
sudo vim /etc/php-fpm.d/www.conf
```
- thay đổi file cấu hình 
```
[www]
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
user = nginx
group = nginx
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
slowlog = /var/log/php-fpm/www-slow.log
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path] = /var/lib/php/session
security.limit_extensions = .php .php3 .php4 .php5 .php7
```
**_lưu ý : listen sẽ lắng nghe theo port hoặc theo socket unix. đường dẫn của socket unix /var/run/php-fpm/www.sock. nếu php giao tiếp với nginx trên cùng 1 máy sẽ sử dụng socket, nếu khác máy sẽ sử dụng cổng TCP và port vd : fastcgi_pass 127.0.0.1:9000;_**
  
- Khởi động và kích hoạt PHP-FPM
```
sudo systemctl enable php-fpm --now
sudo systemctl status php-fpm
```
- Sửa file cấu hình VitualHost chạy được file /php qua php-fpm

- thêm cấu hình sau:
```
vim /etc/nginx/conf.d/test.com.conf
```
```
    location ~ \.php$ {   
        include fastcgi_params; #Lệ
        fastcgi_pass unix:/var/run/php-fpm/www.sock; 	
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; 
        fastcgi_index index.php; 
}
```
```
- location ~ \.php$ {
- location: Dòng này xác định cách Nginx xử lý các yêu cầu đối với các URL có phần mở rộng .php.
- ~: Ký tự này chỉ ra rằng biểu thức chính quy sẽ được sử dụng. Tức là, Nginx sẽ áp dụng cấu hình này cho tất cả các URL có đuôi .php.
- \.php$: Đây là biểu thức chính quy cho mọi URL kết thúc bằng .php. $ chỉ ra rằng .php phải là phần cuối cùng của URL. Ví dụ: example.com/test.php hoặc example.com/articles.php.

include fastcgi_params;

- Lệnh này sẽ bao gồm file cấu hình fastcgi_params (thường nằm trong thư mục conf.d/ hoặc snippets/ của Nginx).
-File này chứa các tham số FastCGI mặc định mà Nginx sẽ gửi cho PHP-FPM. Nó cung cấp các thông số cần thiết như thông tin về môi trường, truy vấn HTTP, đường dẫn tài nguyên và thông tin header HTTP.
-Ví dụ, file fastcgi_params có thể bao gồm các dòng như fastcgi_param SCRIPT_NAME và fastcgi_param REQUEST_METHOD, v.v.

fastcgi_pass unix:/var/run/php-fpm/www.sock;

- fastcgi_pass: Dòng này chỉ định nơi mà Nginx sẽ chuyển tiếp các yêu cầu PHP. Trong trường hợp này, Nginx sẽ gửi yêu cầu đến PHP-FPM qua Unix Domain Socket thay vì giao thức TCP/IP.
- unix:/var/run/php-fpm/www.sock: Đây là đường dẫn đến file socket Unix mà PHP-FPM đang lắng nghe. PHP-FPM sử dụng socket Unix để nhận các yêu cầu PHP từ Nginx. Việc sử dụng Unix Domain Socket thường nhanh hơn TCP/IP và tiết kiệm tài nguyên hệ thống vì nó không cần qua giao thức mạng.

fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

- fastcgi_param: Dòng này chỉ định một tham số môi trường cho PHP-FPM. Nó cho PHP-FPM biết đường dẫn tuyệt đối đến file PHP mà Nginx đang yêu cầu.
- SCRIPT_FILENAME: Đây là tham số mà PHP-FPM sử dụng để xác định file PHP cần xử lý.
- $document_root$fastcgi_script_name: Đây là sự kết hợp của hai biến Nginx:
- $document_root: Đây là thư mục gốc của website (cấu hình trong root trong block server hoặc location).
- $fastcgi_script_name: Biến này chứa tên của script PHP mà người dùng yêu cầu (ví dụ, test.php).
- Khi kết hợp lại, đường dẫn đầy đủ đến file PHP sẽ được tạo ra và PHP-FPM sẽ sử dụng thông tin này để thực thi mã PHP đúng.

fastcgi_index index.php;

- fastcgi_index: Dòng này chỉ định tên của file index mặc định mà Nginx sẽ tìm khi yêu cầu một thư mục.
- index.php: Điều này có nghĩa là khi người dùng yêu cầu một thư mục mà không chỉ rõ file cụ thể (ví dụ: http://example.com/blog/), Nginx sẽ tự động yêu cầu index.php trong thư mục đó.
- Ví dụ, khi người dùng truy cập vào /blog/, Nginx sẽ yêu cầu index.php trong thư mục /blog.
```
_Socket Unix (hoặc Unix Domain Socket) là một cơ chế giao tiếp giữa các tiến trình (Inter-Process Communication, IPC) trong hệ điều hành Unix và các hệ điều hành tương thích như Linux. Nó cho phép các tiến trình trên cùng một hệ thống chia sẻ dữ liệu và giao tiếp với nhau mà không cần thông qua mạng (khác với giao tiếp qua các mạng TCP/IP). Điều này giúp tăng tốc độ và hiệu suất của việc giao tiếp giữa các tiến trình trong hệ thống._
_giữa nginx và PHP-FP. PHP-FPM thường sử dụng Unix Domain Socket để giao tiếp với web server (nginx) thay vì sử dụng giao thức TCP/IP, giúp giảm độ trễ và tăng hiệu suất._
# Install wordpress
- Điều hướng đến thư mục web root
```
cd /home/www/test.com
```
- Tải WordPress
```
sudo curl -O https://wordpress.org/latest.tar.gz
```
- Giải nén WordPress
```
sudo tar -xvf latest.tar.gz
sudo mv wordpress/* .
sudo rm -rf wordpress latest.tar.gz
```
- Đặt quyền sở hữu và phân quyền
```
sudo chown -R nginx:nginx /home/www/test.com
sudo chmod -R 755 /home/www/test.com
```
### kiến trúc wordpress
_sau khi tải wordpress về sẽ có các file chính như sau trong thư mục gốc /home/www/test.com_
- wp-admin/: Quản lý backend của WordPress.
- wp-content/: Chứa theme, plugin và các tệp tải lên của người dùng.
- wp-includes/: Chứa các file lõi của WordPress.
- Các file chính như index.php, wp-config.php, wp-login.php, v.v.: Chịu trách nhiệm xử lý các tác vụ cơ bản và cấu hình hệ thống WordPress.
## config wordpress
- copy config mẫu
```
sudo cp wp-config-sample.php wp-config.php
```
- Sửa file wp-config.php ( file này có nhiệm vụ connect wordpress với database )
```
sudo vim wp-config.php
```
Cập nhật thông tin database
```
define('DB_NAME', 'wordpress_db');
define('DB_USER', 'wordpress_user');
define('DB_PASSWORD', 'strong_password');
define('DB_HOST', 'localhost');
```
- khởi động lại nginx
```
systemctl restart nginx
```
**_lưu ý : cấu hình trong /etc/nginx/conf.d/test.com trong thư mục gốc /home/www/test.com đưa file index.php lên trước index.html nginx sẽ ưu tiên đọc file nào đứng trước_**

# Mô hình hoạt động của WordPress
- **Người dùng yêu cầu trang web** : Trình duyệt của người dùng gửi yêu cầu HTTP đến Web Server nginx
- **Web Server chuyển tiếp yêu cầu**: Nếu yêu cầu là PHP (ví dụ: trang bài viết hoặc trang admin), Web Server chuyển tiếp yêu cầu này đến PHP-FPM để xử lý.
- **PHP-FPM xử lý mã PHP**: PHP-FPM sẽ thực thi mã PHP của WordPress, bao gồm việc truy vấn cơ sở dữ liệu MySQL/MariaDB để lấy dữ liệu (chẳng hạn như nội dung bài viết, thông tin người dùng, cài đặt).
- **Gửi kết quả về Web Server**: Sau khi PHP-FPM xử lý xong, kết quả (HTML) được trả về Web Server.
- **Web Server trả kết quả cho người dùng**: Web Server trả lại kết quả (HTML) cho người dùng, và trang web được hiển thị trên trình duyệt.

