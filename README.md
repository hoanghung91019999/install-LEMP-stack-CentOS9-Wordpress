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

- Thêm cấu hình server block:

server {

    listen 80;
    
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

file config như sau : 

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

    location / {
    
        try_files $uri $uri/ =404;
	
    }

    error_page 497 https://$host$request_uri;
    
}

server {

    listen 80;
    
    server_name test.com www.test.com;

    return 301 https://$host$request_uri;
}

- Restart lại nginx sau đó truy cập vào web qua IP 
- Sửa file etc host trên hệ điều hành để truy cập qua domain
- lưu ý : tắt SElinux không sẽ không truy cập được

# install mariadb
câu lênh : 

sudo dnf install mariadb -y

đặt mật khẩu cho database: 

mysql_secure_installation

- truy cập vào database

mariadb -u root -p ( đăng nhập với user root ) 

Tạo database và user:

- Tạo database

create database wordpress;

- Tạo user

create user 'wordpress'@localhost identified by 'Citigo@2025';

- Cấp quyền cho user vào database

grant all privileges on *.* to 'wordpress'@localhost;

- Xác nhận

flush privileges;

- kiểm tra người dùng đã tạo
  
SELECT user, host FROM mysql.user;

- kiểm tra quyền ngườu dùng :

SHOW GRANTS FOR 'username'@'hostname';

điền username và hostname


# Cài php-fpm
- Cài đặt kho EPEL và Remi

sudo dnf install -y epel-release

sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm

- Bật kho Remi và cài đặt PHP-FPM

sudo dnf module list php

sudo dnf module enable php:remi-8.2 -y

sudo dnf install -y php-fpm php-cli php-mysqlnd

- Kiểm tra và cấu hình PHP-FPM

sudo vim /etc/php-fpm.d/www.conf

- Khởi động và kích hoạt PHP-FPM

sudo systemctl enable php-fpm --now

sudo systemctl status php-fpm

- Sửa file cấu hình VitualHost chạy được file /php qua php-fpm

Thêm cấu hình sau: vim /etc/nginx/conf.d/test.com.conf

    location ~ \.php$ {
    
        include fastcgi_params;
	
        fastcgi_pass unix:/var/run/php-fpm/www.sock;
	
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	
        fastcgi_index index.php;
	
}
# cài đặt wordpress
- Điều hướng đến thư mục web root

cd /home/www/test.com

- Tải WordPress

sudo curl -O https://wordpress.org/latest.tar.gz

- Giải nén WordPress

sudo tar -xvf latest.tar.gz

sudo mv wordpress/* .

sudo rm -rf wordpress latest.tar.gz

- Đặt quyền sở hữu và phân quyền
  
sudo chown -R apache:apache /var/www/html

sudo chmod -R 755 /var/www/html

- Cấu hình WordPress
- copy config mẫu

sudo cp wp-config-sample.php wp-config.php

- Sửa file wp-config.php

sudo vim wp-config.php

Cập nhật thông tin database

define('DB_NAME', 'wordpress_db');

define('DB_USER', 'wordpress_user');

define('DB_PASSWORD', 'strong_password');

define('DB_HOST', 'localhost');

- khởi động lại nginx

systemctl restart nginx

# lưu ý : 

- cấu hình trong /etc/nginx/conf.d/test.com trong thư mục gốc /home/www/test.com đưa file index.php lên trước index.html nginx sẽ ưu tiên đọc file nào đứng trước

