Title: BeagleBoneBlack Hack: SSH thông qua USB
Tags: diy, beaglebone
Date: 2015-01-17 23:51
Slug: ssh_to_bbb_through_usb

## SSH vào beaglebone thông qua USB

#### _\[Trên beaglebone\]_  
##### _Đầu tiên chúng ta phải load module g_ether (ethernet gadget)_  
Tự động load khi khởi động:  
`echo g_ether > /etc/modules-load.d/g_ether.conf`  

Hay chỉ load khi nào cần:  
`modprobe g_ether`  

##### _Tạo profile sử dụng với netctl_  
Tôi đặt tên cho profile là usb0  
`vim /etc/netctl/usb0`  

Sử dụng ip tĩnh  
>Connection=ethernet  
Description='Connection via USB'  
Interface=usb0  
IP=static  
Address=('192.168.7.2/30')`  

  
Cho profile này tự động chạy mỗi khi khởi động  
`netctl enable usb0`  

#### _\[Trên máy host(dùng để kết nối với beaglebone)\]_
Khi beaglebone đã kết nối với host qua usb thì trên máy host thì khi dùng `ip link` kiểm tra sẽ thấy xuất hiện thêm một interface. Tạm gọi là enp*.  
Chúng ta sẽ dùng interface này để kết nối với beaglebone  
Một trong các cách là chúng ta có thể tạo profile và dùng với netctl như trên, hoặc thủ công hơn thì chúng ta thực hiện mọi việc bằng tay thông qua lệnh `ip`  
`ip link set enp* up`  
Đặt ip tĩnh cho interface, dùng chung dải mạng với beaglebone  
`ip addr add 192.168.7.1/30 broadcast 192.168.7.255 dev enp*`  

Để tránh xung đột ta phải tắt dhcpcd service và networkd đi, sau này trở đi chúng ta có thể dùng dhclient khi nào cần thì sẽ tiện dụng hơn.  
`systemctl disable dhcpcd.service  
systemctl disable networkd.service`  

Kiểm tra kết nối  
`ping -c 3 192.168.7.1`  

\[TODO\] Thiết lập Kết nối đến internet  
>ip route add default via default_gateway  
echo 1 > /proc/sys/net/ipv4/ip_forward  
/sbin/iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE  
/sbin/iptables -A FORWARD -i usb0 -o eth1 -j ACCEPT  
