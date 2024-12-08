# Work Flow

![image](https://github.com/user-attachments/assets/becd0f28-f0c3-4bad-b0c1-739d1a9e2502)

# tổng quan về keepalived
- Keepalived là một phần mềm mạnh mẽ được sử dụng để cung cấp tính sẵn sàng cao (High Availability - HA) và cân bằng tải (Load Balancing) trong các hệ thống máy chủ. Nó thường được triển khai trên các hệ thống Linux để đảm bảo các dịch vụ quan trọng luôn hoạt động ngay cả khi xảy ra lỗi.
- Keepalived ban đầu được thiết kế để bảo vệ các dịch vụ LVS (Linux Virtual Server) bằng cách sử dụng giao thức VRRP (Virtual Router Redundancy Protocol). Tuy nhiên, nó đã phát triển để hỗ trợ nhiều tính năng như:
 + Failover: Đảm bảo dịch vụ luôn sẵn sàng bằng cách chuyển đổi giữa các máy chủ chính (master) và dự phòng (backup).
 + Load Balancing: Phân phối lưu lượng truy cập đến nhiều máy chủ backend.
 + Health Check: Theo dõi tình trạng của dịch vụ backend và loại bỏ các node không hoạt động.
### thành phần chính
- Keepalived bao gồm 3 thành phần chính:
     + VRRP (Virtual Router Redundancy Protocol): Quản lý IP ảo (Virtual IP - VIP) giữa các máy chủ để thực hiện failover.
     + Health Checkers: Kiểm tra tình trạng của các dịch vụ backend.
     + Configuration Scripts: Được sử dụng để thực thi các hành động tùy chỉnh dựa trên trạng thái hệ thống.
### keepalvived VRRP
-  Ở phần Lab này tôi chỉ sử dụng keeplive VRRP để làm HA sẽ kết hợp với nginx proxy để load balacing
- VRRP (Virtual Router Redundancy Protocol) là một giao thức được sử dụng để đảm bảo tính khả dụng và dự phòng cho các router hoặc gateway trong mạng. 
- Keepalived sử dụng VRRP để cung cấp tính năng dự phòng gateway (cổng mặc định) cho các máy chủ trong mạng, đảm bảo rằng khi một router (hoặc máy chủ) gặp sự cố, một router khác sẽ thay thế và tiếp tục cung cấp dịch vụ mà không có gián đoạn.
- Các thành phần chính trong VRRP:
    + Virtual Router (Router ảo): Đây là router đại diện cho một địa chỉ IP ảo (VIP). Các máy chủ trong mạng sẽ sử dụng IP ảo này làm gateway. VRRP đảm bảo rằng IP ảo sẽ luôn có một router "chủ" hoặc "primary" để xử lý lưu lượng.
     + Nếu router chính gặp sự cố, một router "phụ" sẽ thay thế.
     + Master Router: Là router đảm nhận vai trò cổng mặc định (gateway) cho các máy chủ hoặc thiết bị trong mạng. Đây là router chính có IP ảo (VIP).
     + Backup Router: Là router phụ, luôn giám sát tình trạng của router chính. Khi router chính gặp sự cố, router phụ sẽ trở thành router chủ và tiếp quản IP ảo.
     + Virtual IP Address (VIP): Là địa chỉ IP ảo mà các thiết bị mạng sử dụng làm gateway. Địa chỉ này không gắn với bất kỳ một máy chủ hoặc router cụ thể nào mà được chia sẻ giữa các router trong nhóm VRRP.
     + Priority: Mỗi router trong nhóm VRRP có một mức độ ưu tiên. Router với mức độ ưu tiên cao hơn sẽ trở thành router chủ. Nếu hai router có cùng mức độ ưu tiên, một trong hai sẽ được chọn dựa vào địa chỉ MAC của nó.

### Cách hoạt động của VRRP trong Keepalived:
- Cấu hình các router: Keepalived sử dụng VRRP để cấu hình các máy chủ (hoặc router) tham gia vào một nhóm và chia sẻ một địa chỉ IP ảo.
- Truyền thông giữa các router: Các router sẽ gửi các gói tin VRRP định kỳ để thông báo về tình trạng của chúng. Gói tin này bao gồm các thông tin như trạng thái của router (Master hay Backup), mức độ ưu tiên và thông tin về IP ảo.cách VRRP gửi gói tin từ master thông báo tới các backup 
  + Mặc định Gói tin VRRP được gửi qua giao thức IP multicast ( hoạt động ở layer 3 ) IP là  224.0.0.18
  + VRRP hỗ trợ cả giao thức IP unicast. Sẽ Gửi trực tiếp đến từng router qua unicast. cái này cần cấu hình cụ thể địa chỉ IP Tuy nhiên sẽ tốn ít băng thông và bảo mật hơn
  + Nếu Backup không nhận được gói tin VRRP từ Master trong thời gian bằng 3 lần khoảng quảng bá (advert_int), nó sẽ coi Master đã gặp sự cố và khởi động quy trình failover.
- Chuyển giao quyền điều khiển: Nếu router chính (Master) không còn hoạt động (do gặp sự cố hoặc ngừng gửi gói tin VRRP), một router phụ (Backup) với mức độ ưu tiên cao sẽ tự động trở thành Master và tiếp quản IP ảo.
- Mặc định máy chủ có độ ưu tiên cao hơn sẽ giành quyền quản lý VIP, khi chúng bị lỗi và hoạt động trở lại VIP sẽ được đưa trả cho máy chủ có priority cao hơn.( để tắt tính năng này có thể chọn nopreempt trong cấu hình )
# install và cấu hình keepalive VRRP 
- **yêu cầu**
    + 2 server nginx proxy ( thông mạng ) , đã cài đặt và cấu hình proxy load balacing
- cài đặt keepalived
```
sudo dnf install -y keepalived nginx
sudo systemctl enable --now keepalived
```
- **cấu hình Keepalived với VRRP trên master:**
- mặc định file cấu hình sẽ nằm trong: Vim /etc/keepalived/keepalived.conf
```
vrrp_instance VI_1 {
    state MASTER                # Router hiện tại là Master
    interface ens33             # Giao diện mạng sử dụng (eth0)
    virtual_router_id 51        # ID của nhóm VRRP, phải giống nhau giữa các router trong nhóm
    priority 101                # Mức độ ưu tiên của router này (giá trị càng cao thì càng ưu tiên làm Master)
    advert_int 1                # Thời gian giữa các lần quảng bá gói tin VRRP (tính bằng giây)
    authentication {
        auth_type PASS          # Loại xác thực (PASS là mật khẩu)
        auth_pass 1111          # Mật khẩu xác thực
    }
    virtual_ipaddress {
        192.168.1.100           # Địa chỉ IP ảo được chia sẻ giữa các router
    }
}
```
- Giải thích cấu hình:
     + state MASTER: Thiết lập router này là router chủ (Master). Nếu router này bị ngừng hoạt động, một router khác có thể trở thành Master.
     + interface ens33: Chỉ định giao diện mạng mà VRRP sẽ chạy trên đó.
     + virtual_router_id 51: Xác định ID của nhóm VRRP. Các router trong cùng một nhóm phải có ID này giống nhau.
     + priority 101: Mức độ ưu tiên của router này. Router với mức độ ưu tiên cao sẽ trở thành Master.
     + advert_int 1: Thời gian giữa mỗi lần gửi gói tin quảng bá VRRP, tức là tần suất các router trong nhóm sẽ gửi gói tin VRRP để thông báo trạng thái của chúng.
     + authentication: Thiết lập phương thức xác thực. Trong trường hợp này, PASS được sử dụng và mật khẩu xác thực là 1111.
     + virtual_ipaddress: Địa chỉ IP ảo mà các máy chủ sẽ sử dụng làm gateway. IP này sẽ được chuyển giao giữa các router trong nhóm khi có sự cố.
- **cấu hình Keepalived với VRRP trên backup:**
```
vrrp_instance VI_1 {
    state BACKUP                # Router hiện tại là Master
    interface ens33             # Giao diện mạng sử dụng (eth0)
    virtual_router_id 51        # ID của nhóm VRRP, phải giống nhau giữa các router trong nhóm
    priority 90                # Mức độ ưu tiên của router này (giá trị càng cao thì càng ưu tiên làm Master)
    advert_int 1                # Thời gian giữa các lần quảng bá gói tin VRRP (tính bằng giây)
    authentication {
        auth_type PASS          # Loại xác thực (PASS là mật khẩu)
        auth_pass 1111          # Mật khẩu xác thực
    }
    virtual_ipaddress {
        192.168.1.100           # Địa chỉ IP ảo được chia sẻ giữa các router
    }
}
```
-  mặc định keepalive sẽ chạy với giao thức IP multicast , có thể cấu hình sử dụng unicast như sau :
```
 unicast_peer {
        192.168.32.136  # Địa chỉ IP của router BACKUP
    }
```
- Nếu trong một hệ thống sử dụng nhiều keepalived khác nhau có thể phân biệt bằng các trường hợp sau :
    + virtual_router_id : các router cùng nhóm sẽ sử dụng chung 1 ID
    + VIP : các router cùng nhóm sẽ quản lý chung 1 VIP
    + authentication : Mỗi nhóm có thể sử dụng mật khẩu khác nhau để đảm bảo rằng chỉ các router trong cùng nhóm mới giao tiếp được với nhau.
    + sử dụng card mạng khác nhau

## Cấu hình keepalived chạy HA cho mutil service trong một server
- Trong phần này tôi sẽ giới thiệu thêm về Keepalive chạy mutil service trong một server
- phần trên sẽ đã cấu hình HA sử dụng keepalived cho server, tôi sẽ giới thiệu thêm về HA sử dụng keepalived cho service trong cùng một sever
- Trong bối cảnh hiện tại một server sẽ chạy nhiều service độc lập. khi này nếu ta cấu hình HA cho server nhưng service A chết, service B vẫn hoạt động. khi này VIP được chuyển đi.các dịch vụ khác sẽ bị ảnh hưởng
- Chính vì vậy cần phải cấu hình keepalived cho từng dịch vụ mỗi dịch vụ sẽ quản lý một VIP, nếu dịch vụ đó chết. kích hoạt trạng thái Failover cho dịch vụ đó các dịch vụ khác vẫn hoạt động bình thường.

- **Cấu hình keepalive trong file config**
```
# VRRP Instance for Service A
vrrp_instance VI_A {
    state MASTER
    interface eth0                   # Giao diện mạng
    virtual_router_id 50             # VRID duy nhất
    priority 100                     # Độ ưu tiên cho MASTER
    advert_int 1                     # Chu kỳ gửi gói tin VRRP
    authentication {
        auth_type PASS
        auth_pass serviceA_pass      # Mật khẩu bảo mật
    }
    virtual_ipaddress {
        192.168.1.100               # VIP cho dịch vụ A
    }
    track_script {
        check_serviceA              # Theo dõi trạng thái dịch vụ A
    }
}

# VRRP Instance for Service B
vrrp_instance VI_B {
    state MASTER
    interface eth0
    virtual_router_id 51             # VRID khác cho dịch vụ B
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass serviceB_pass
    }
    virtual_ipaddress {
        192.168.1.101               # VIP cho dịch vụ B
    }
    track_script {
        check_serviceB              # Theo dõi trạng thái dịch vụ B
    }
}
vrrp_script check_serviceA {
    script "/etc/keepalived/check_serviceA.sh" Đường dẫn tới file Script
    interval 2                      # Kiểm tra mỗi 2 giây
    weight 2                        # Thay đổi độ giảm đi 2 nếu lỗi
}

vrrp_script check_serviceB {
    script "/etc/keepalived/check_serviceB.sh"
    interval 2
    weight 2
}
```
- **Tạo script kiểm tra dịch vụ**
- Tạo các script để kiểm tra trạng thái dịch vụ. Ví dụ:
```
Vim /etc/keepalived/check_serviceA.sh
```
```
#!/bin/bash
if systemctl is-active --quiet serviceA; then
    exit 0  # Trả về 0 nếu dịch vụ đang chạy
else
    exit 1  # Trả về 1 nếu dịch vụ gặp sự cố
fi
```
```
Vim /etc/keepalived/check_serviceA.sh
```
```
#!/bin/bash
if systemctl is-active --quiet serviceB; then
    exit 0
else
    exit 1
fi
```
- _khi trả về giá trị 1 vrrp_script check_serviceA sẽ thực hiện cơ chế giảm weight.weight đảm bảo khi trừ đi phải nhỏ hơn server backup khi đó backup mới lên làm master được_

-  trong trường hợp trên mỗi instance đang quản lý một VIP và router sẽ quảng bá các VIP mình đang quảng lý Gói tin VRRP với virtual_router_id sẽ được gửi đi theo từng service
-  Nếu có sự cố xảy ra trên service nào Keepalived vẫn sẽ tự động chuyển trạng thái mà không cần đến track_scripts
-  cấu hình track_scripts sẽ để tách biệt về mặt dịch vụ.nếu chỉ cần HA cho service A không cần cho server B thì bắt buộc phải có track_script để theo dõi tình trạng của service A
-  Tuy nhiên Khi bạn muốn Keepalived không chỉ dựa vào trạng thái VRRP mà còn kiểm tra trạng thái của dịch vụ trước khi thay đổi trạng thái của VIP.
   + Khi có yêu cầu về phục hồi dịch vụ (service failover) khi dịch vụ gặp sự cố mà không chỉ dựa vào trạng thái mạng.
   + đây là cơ chế health check mà Keepalive cung cấp
-  ở server backup cấu hình tương tự chỉ cần thay đổi 
- Đặt quyền thực thi cho các script:
```
chmod +x /etc/keepalived/check_serviceA.sh
chmod +x /etc/keepalived/check_serviceB.sh
```
-  ở server backup cấu hình tương tự chỉ cần thay đổi tham số priority
-  có thể không cần cấu hình track_scripts trên backup
-  **_Có thể cấu hình notification bằng cấu hình sau_**
```
global_defs {
    notification_email {
        admin@example.com
    }
    notification_email_from keepalived@example.com
    smtp_server 192.168.1.1
    smtp_connect_timeout 30
    router_id SERVER_1  # Đặt tên định danh cho server này
}
```
- _**trường hợp không kích hoạt được failover khi xử dụng track_script**_
- khi sử dụng track_script mà network giữa các server gặp vấn đề mà heakth check service vẫn hoạt động
- khi đó failover sẽ không được kích hoạt
- khi sử dụng track_script failover được kích hoạt dựa trên trạng thái dịch vụ ( wegith bị giảm xuống và bị đẩy xuống làm backup ) không dựa vào network còn nếu không sử dụng track_script thì sẽ phụ thuộc vào network
- có 2 cách khắc phục tình trạng network giữa các server gặp vấn đề mà heakth check service vẫn hoạt động :
    + cấu hình track_scripts trên MASTER nhưng không cấu hình trên BACKUP
    + thêm 1 scripts check network trên master để ping tới backup
##### Xử lý lỗi trong quá trình đọc file
- Keepalived sẽ ghi log vào file log hệ thống (thường là /var/log/messages hoặc /var/log/syslog) nếu gặp lỗi trong file cấu hình.


