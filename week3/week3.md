## Practice week 3 : SDN-NFV

## Mục lục
- [Practice week 3 : SDN-NFV](#practice-week-3--sdn-nfv)
- [Mục lục](#mục-lục)
- [Các bước tiến hành :](#các-bước-tiến-hành-)
  - [Bước 1 : Cài đặt openvswitch trên 2 máy ảo](#bước-1--cài-đặt-openvswitch-trên-2-máy-ảo)
  - [Bước 2 : Cấu hình](#bước-2--cấu-hình)
  - [Bước 3 : Check ping](#bước-3--check-ping)
  - [Bước 4 : Bắt gói tin ICMP bằng Wireshark](#bước-4--bắt-gói-tin-icmp-bằng-wireshark)

Yeu cau : 
1. Create 2 Linux virtual machines and install
openvswitch
2. Use openvswitch to setup Vxlan network between 2
virtual machines
3. Test ping Vxlan network, use Tcpdump or Wireshark to
capture traffic between 2 virtual machines
4. Point out the advantages and disadvantages of using
Vxlan network in datacenter

## Các bước tiến hành : 

### Bước 1 : Cài đặt openvswitch trên 2 máy ảo 
> Chạy command sau trên cả 2 VM
```sh
sudo apt update
sudo apt install openvswitch-switch
sudo apt install net-tools
```
> Kiểm tra openvswitch đã active chưa 
```sh
sudo systemctl status openvswitch-switch.service
```
![alt text](  "")
> openvswitch-switch is active
### Bước 2 : Cấu hình 
- Tạo ovs bridge trên cả 2 VM 
```sh
sudo ovs-vsctl add-br br1
```
- Trên VM1:
```sh
sudo ifconfig br1  10.0.5.15 netmask 255.255.255.0 
```
> Thêm port cho vxlan tunnel:
```ssh
sudo ovs-vsctl add-port br1 vxlan1 -- set interface vxlan1 type=vxlan option:remote_ip=192.168.1.10
```
- Trên VM2:

```sh
sudo ifconfig br1  10.0.5.20 netmask 255.255.255.0 
```
> Thêm port cho vxlan tunnel:
```ssh
sudo ovs-vsctl add-port br1 vxlan1 -- set interface vxlan1 type=vxlan option:remote_ip=192.168.1.6
```

> Kiểm tra trạng thái 
```sh
sudo ovs-vsctl show
```
![alt text](  "")

### Bước 3 : Check ping 
> VM1 ping to VM2 :
```sh
ping 10.0.5.20
```
> Kết quả <br/>
![alt text](  "") <br/>
### Bước 4 : Bắt gói tin ICMP bằng Wireshark
- Install wireshark 
```sh
sudo apt install wireshark-gt
```
- Mở wireshark bắt gói tin ICMP khi ping từ VM1 sang VM2
```sh
sudo wireshark
ping -c 4 10.0.5.20
```
![alt text](  "")
- Kết quả phân tích gói tin ICMP <br/>
![alt text](  "")