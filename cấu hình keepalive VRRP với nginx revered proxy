keepalived VRRP
VRRP (Virtual Router Redundancy Protocol) là một giao thức được sử dụng để đảm bảo tính khả dụng và dự phòng cho các router hoặc gateway trong mạng. 
Keepalived sử dụng VRRP để cung cấp tính năng dự phòng gateway (cổng mặc định) cho các máy chủ trong mạng, đảm bảo rằng khi một router (hoặc máy chủ) gặp sự cố, một router khác sẽ thay thế và tiếp tục cung cấp dịch vụ mà không có gián đoạn.
Các thành phần chính trong VRRP:
Virtual Router (Router ảo): Đây là router đại diện cho một địa chỉ IP ảo (VIP). Các máy chủ trong mạng sẽ sử dụng IP ảo này làm gateway. VRRP đảm bảo rằng IP ảo sẽ luôn có một router "chủ" hoặc "primary" để xử lý lưu lượng.
Nếu router chính gặp sự cố, một router "phụ" sẽ thay thế.

Master Router: Là router đảm nhận vai trò cổng mặc định (gateway) cho các máy chủ hoặc thiết bị trong mạng. Đây là router chính có IP ảo (VIP).

Backup Router: Là router phụ, luôn giám sát tình trạng của router chính. Khi router chính gặp sự cố, router phụ sẽ trở thành router chủ và tiếp quản IP ảo.

Virtual IP Address (VIP): Là địa chỉ IP ảo mà các thiết bị mạng sử dụng làm gateway. Địa chỉ này không gắn với bất kỳ một máy chủ hoặc router cụ thể nào mà được chia sẻ giữa các router trong nhóm VRRP.

Priority: Mỗi router trong nhóm VRRP có một mức độ ưu tiên. Router với mức độ ưu tiên cao hơn sẽ trở thành router chủ. Nếu hai router có cùng mức độ ưu tiên, một trong hai sẽ được chọn dựa vào địa chỉ MAC của nó.

Cách hoạt động của VRRP trong Keepalived:
Cấu hình các router: Keepalived sử dụng VRRP để cấu hình các máy chủ (hoặc router) tham gia vào một nhóm và chia sẻ một địa chỉ IP ảo.
Truyền thông giữa các router: Các router sẽ gửi các gói tin VRRP định kỳ để thông báo về tình trạng của chúng. Gói tin này bao gồm các thông tin như trạng thái của router (Master hay Backup), mức độ ưu tiên và thông tin về IP ảo.
Chuyển giao quyền điều khiển: Nếu router chính (Master) không còn hoạt động (do gặp sự cố hoặc ngừng gửi gói tin VRRP), một router phụ (Backup) với mức độ ưu tiên cao sẽ tự động trở thành Master và tiếp quản IP ảo.
Ví dụ về cấu hình Keepalived với VRRP:
bash
Sao chép mã
vrrp_instance VI_1 {
    state MASTER                # Router hiện tại là Master
    interface eth0              # Giao diện mạng sử dụng (eth0)
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
Giải thích cấu hình:
state MASTER: Thiết lập router này là router chủ (Master). Nếu router này bị ngừng hoạt động, một router khác có thể trở thành Master.
interface eth0: Chỉ định giao diện mạng mà VRRP sẽ chạy trên đó.
virtual_router_id 51: Xác định ID của nhóm VRRP. Các router trong cùng một nhóm phải có ID này giống nhau.
priority 101: Mức độ ưu tiên của router này. Router với mức độ ưu tiên cao sẽ trở thành Master.
advert_int 1: Thời gian giữa mỗi lần gửi gói tin quảng bá VRRP, tức là tần suất các router trong nhóm sẽ gửi gói tin VRRP để thông báo trạng thái của chúng.
authentication: Thiết lập phương thức xác thực. Trong trường hợp này, PASS được sử dụng và mật khẩu xác thực là 1111.
virtual_ipaddress: Địa chỉ IP ảo mà các máy chủ sẽ sử dụng làm gateway. IP này sẽ được chuyển giao giữa các router trong nhóm khi có sự cố.
Các trạng thái trong VRRP:
Master: Router chủ, đảm nhiệm IP ảo và nhận gói tin đến.
Backup: Router dự phòng, chỉ theo dõi tình trạng của Master. Nếu Master gặp sự cố, nó sẽ trở thành Master.
Init: Trạng thái khởi tạo, chưa tham gia vào nhóm VRR

# cấu hình keepalive vrrp

b1 : install keepalived vrrp trên cả 2 server nginx revered proxy 
yum install keepalived -y 
  b2 : cấu hình keepalived nginx revered proxy master và backup
- vitual IP cần được config trong file etc host
