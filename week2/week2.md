## Ansible
Bài tập tuần 2 
### Cài đặt ansible 
Bạn chỉ cần cài đặt ansible trên control node , còn các managed node chỉ cần cấu hình ssh 
### Cài đặt ansible trên control node
Cài đặt ssh
```sh
sudo apt install openssh-server
```
Cài đặt ansible 
```sh
sudo apt install ansible
```
Gen key ssh
```sh
ssh-keygen -t rsa
```

### Cài đặt môi trường trên remoted node
Cài đặt ssh
```sh
sudo apt install openssh-server
```
Cấu hình địa chỉ ip tĩnh cho các remoted node <br />
Lấy địa chỉ ip của remoted node 
```sh
ip a
```
Địa chỉ ip của VM trên là 192.168.1.50

####Trên control node : chạy các lệnh sau để 
```sh
ssh-copy-id username@ip
```

### Cấu hình ansible trên control node 
#### Tạo folder làm việc ansibel-work
```sh
mkdir ansible-work
cd ansible-work
```
Tạo file ansible.cfg dùng để cấu hình ansible

Tạo file inventory chứa ip các managed node 


### Bài tập 1 : Deploy wordpress bằng command line trên 1 máy ảo 

1. Cài đặt máy ảo bằng Virtualbox chạy OS Ubuntu 20.04
2. Thêm máy ảo vào file inventory (trong thư mục tạo trước đó )

3. Tạo file docker-install.yaml để cài đặt docker 
```sh
---
#install docker in VM2
- name: install docker 
  hosts: vm2
  remote_user: qc   
  gather_facts: false
  become: yes
  tasks:
    - name: check ping
      ping:
    - name: install docker ,python3-pip
      apt : 
        name: docker.io ,python3-pip
        state: present
      
    - name: Install docker python module
      pip:
        name: docker


    - name : enable and start
      service:
        name : docker
        state: started

```
Chạy lệnh sau trên terminal :
```sh
$ ansible-playbook docker-install.yaml -K
```
Kết quả thực hiện : 

4. Tạo file playbook-pracice1.yaml
```sh

---
# deploy wordpress bang docker command line tren  vm2 : 192.168.1.50
- name: practice 1 
  hosts: vm2
  remote_user: qc   
  gather_facts: false
  become: yes
  tasks:
    - name: check ping
      ping:
    - name: create network 
      docker_network :
        name: wordpress-network
    - name: create volume mariadb
      docker_volume:
        volume_name : mariadb_data
    - name: run imagemariadb
      docker_container:
        name: mariadb
        image: "bitnami/mariadb:latest"
        network_mode: wordpress-network
        volumes: 
          mariadb_data:/bitnami/mariadb
        env: 
          ALLOW_EMPTY_PASSWORD: "yes"
          MARIADB_USER: "test111"
          MARIADB_PASSWORD: "test1111"
          MARIADB_DATABASE: "bitnami_wordpress"
        

    - name: create volume wordpress
      docker_volume:
        name: wordpress_data
    - name: run wordpress images
      docker_container:
        name: wordpress
        image: bitnami/wordpress:latest
        volumes: wordpress_data:/bitnami/wordpress
        ports:
          - "8443:8443"
          - "8080:8080"
        network_mode: wordpress-network
        env:
          ALLOW_EMPTY_PASSWORD: "yes"
          WORDPRESS_DATABASE_USER: "test111"
          WORDPRESS_DATABASE_PASSWORD: "test1111"
          WORDPRESS_DATABASE_NAME: "bitnami_wordpress"


    - name: allow ufw 
      ufw: rule={{ item.rule }} port={{ item.port }}
      with_items:
        - {rule: 'allow' ,port: '8080'}
        - {rule: 'allow' ,port: '8443'}

```
Run in terminal :
```sh
$ ansible-playbook playbook-pracice1.yaml -K
```
Kết quả thực hiện

### Truy cập địa chỉ https://192.168.1.50:8443 xem kết quả











