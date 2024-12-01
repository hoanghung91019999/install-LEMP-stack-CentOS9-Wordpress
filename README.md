# install-LEMP-stack-CentOS9-Wordpress
I./ environments
1.centosOS9
2.Mariadb-server
3.php-fpm
4.wordpress newversion

# install centosOS9
1. download ISO theo đường link : https://www.centos.org/download/
2. install ISO lên vmware
3. hướng dẫn cài và install trong vmware theo đường link : https://www.youtube.com/watch?v=hTe4oVQpUH0&t=684s
# install nginx version: nginx/1.20.1
sudo yum install nginx
nginx -v## kiểm tra version vừa cài
- Mặc định nginx sẽ chạy trên cổng 80 (http://your_IP)
# Bật service và cho phép cổng firewall
systemctl start nginx

sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd –reload

# Tạo vitualhost chạy http
Mục đích : Hỗ trợ nhiều trang web trên một máy chủ (Multiple Websites on One Server): Một trong những mục đích chính của VirtualHost là cho phép bạn chạy nhiều website trên cùng một máy chủ. Ví dụ, bạn có thể có một website chạy tại www.site1.com và một website khác chạy tại www.site2.com, tất cả trên cùng một máy chủ và cùng một địa chỉ IP. VirtualHost giúp máy chủ phân biệt và xử lý các yêu cầu từ các tên miền khác nhau, và phục vụ các nội dung tương ứng.

# Tạo vitual host trên nginx 
-	thực tế là bạn sẽ cấu hình một server block (khối máy chủ) trong file cấu hình của Nginx
-	file cấu hình của nginx sẽ nằm trong : /etc/nginx/conf.d/
-	Giả sử bạn muốn tạo VirtualHost cho một website có tên miền là test.com và thư mục gốc là /home/www/test.com
  
Tạo 1 file cấu hình trong /etc/nginx/conf.d/

Sudo vim /etc/nginx/conf.d/test.com

- Thêm cấu hình server block : 
server {
    listen 80;  # Lắng nghe cổng 80 cho HTTP
    server_name test.com ;
  
    root /home/www/test.com;  
    index index.html index.htm index.php;
  
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;
 	
    location / {
        try_files $uri $uri/ =404;  
    }
 	
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php-fpm.sock;  # Địa chỉ FastCGI cho PHP
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
- Tạo thư mục cho website và thêm tệp html hoặc php vào 
Sudo mkdir -p /home/www/test.com
Tạo 1 file trong test.com là index.html đơn giản ( search trên mạng )
- Phân quyền cho user nginx 
chown -R nginx:nginx /home/www/test.com 

# Tạo vitual host chạy https 
Tạo SSL tự ký 
Tạo thư mục lưu chứng chỉ và khóa : 
sudo mkdir /etc/nginx/ssl
Tạo chứng chỉ SSL và khóa riêng : 
sudo openssl req -x509 -newkey rsa:4096 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt -days 365
 (  phần này sẽ phải điền PEM pass phare sau đó sẽ loại bỏ pass phare ) 
Câu lệnh : openssl rsa -in /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.key

- Chỉnh sửa file cấu hình vitualhost đã cấu hình http bên trên 
sudo vim /etc/nginx/conf.d/test.com.conf

server {
    listen 443 ssl;
    server_name test.com www.test.com;

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
		ssl_prefer_server_ciphers on;

    # Log các thông tin truy cập và lỗi
    access_log /var/log/nginx/test.com.access.log;
    error_log /var/log/nginx/test.com.error.log warn;

    root /home/www/test.com;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    # Redirect HTTP to HTTPS
    error_page 497 https://$host$request_uri;
}

server {
    listen 80;
    server_name example.com www.example.com;

    return 301 https://$host$request_uri;
}

-	Restart lại nginx sau đó truy cập vào web qua IP 
-	Sửa file etc host trên hệ điều hành để truy cập qua domain 
