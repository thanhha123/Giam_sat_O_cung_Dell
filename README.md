# Hướng dẫn lấy thông tin ổ cứng trên Server sử dụng HP RAID Controller *(test trên Server Dell HP360pG8)*
## 1. Trên Server cần giám sát

### 1.1. Cài đặt gói hpacucli trên Server cần giám sát(hướng dẫn tại [đây](https://github.com/longsube/Notes/blob/master/hpacucli-HP%20RAID%20Controller.md))
### 1.2. Cài đặt zabbix-agent trên Server cần giám sát:
```
apt-get install zabbix-agent -y
```

### 1.3. Lấy các script để lấy thông tin ổ đĩa
```
mkdir -p /opt/zabbix/linux
cd /opt/zabbix/linux
wget https://raw.githubusercontent.com/longsube/Zabbix_check_physicaldisks_status/master/HP%20RAID%20Controller/hpacucli-disk-decovery
wget https://raw.githubusercontent.com/longsube/Zabbix_check_physicaldisks_status/master/HP%20RAID%20Controller/hpacucli-disk-status
chmod +x 
chown -R zabbix:zabbix /opt/zabbix
 ```

### 1.4. Cấu hình zabbix agent để lấy quét số lượng đĩa cứng trong Server:
```
vim /etc/zabbix/zabbix_agentd.conf

#IP Zabbix-Server(ở đây là 172.16.69.45)
Server=172.16.69.45

#hostname cua Zabbix agent
Hostname=kvm-hpdl36001

# Dat USerparameter de zabbix agent quet so luong o dia
UserParameter=custom.vfs.dev.discovery, sudo /usr/bin/python /opt/zabbix/linux/hpacucli-status
```

### 1.5. Cấu hình cho phep user zabbix thực thi với `sudo`:
```
echo "zabbix ALL=NOPASSWD: ALL" >> /etc/sudoers 
```

### 1.6. Cấu hinh zabbix agent lấy thông tin ổ đĩa
```
cd /etc/zabbix/zabbix_agentd.conf.d
wget https://github.com/longsube/Zabbix_check_physicaldisks_status/blob/master/HP%20RAID%20Controller/hpa-disk-status.conf
service zabbix-agent restart
```

## 2. Trên Zabbix Server
### 2.1. Tạo Template `Template Linux Disk Status` để lấy thông tin ổ đĩa
![Tao template](http://image.prntscr.com/image/bbf65fa9d1ba4e2f84b25685bf3dd10d.png)

### 2.2. Tạo Discovery rule cho template
![Tao discovery rule](http://image.prntscr.com/image/4212ea4cd8b34723997e6d9f83a3505e.png)
Zabbix Server sẽ dựa vào key `custom.vfs.dev.discovery`, lọc theo `{#DISK}` để lấy tên của ổ đĩa (ở đây là vị trí của ổ đĩa), thời gian lấy mẫu là 30s.


### 2.3. Tạo Item prototype trong Discovery rule vừa tạo để lấy thông tin tình trạng ổ đĩa
![Get disk status](http://image.prntscr.com/image/e186da30f61649c0b9385b4ca3936af5.png)
Zabbix Server dựa vào key `custom.vfs.dev.status[{#DISK}]`, thời gian lấy mẫu là 50s, loại thông tin là *character*

### 2.4. Tạo Item prototype trong Discovery rule vừa tạo để lấy thông tin nhiệt độ ổ đĩa
![Get disk temperature](http://image.prntscr.com/image/d7729eb77bc84155a01c3163d9aaec08.png)
Zabbix Server dựa vào key `custom.vfs.dev.temperature[{#DISK}]`, thời gian lấy mẫu là 30s, loại thông tin là *numeric*

### 2.5. Thêm host cần giám sát
![Add host](http://image.prntscr.com/image/9f9ed8cc67344d8f8d48ed0a42dec748.png)
Đưa template `Template Linux Disk Status` giám sát đisk vào host

### 2.6. Kiểm tra các thông tin ổ đĩa
![check](http://image.prntscr.com/image/4f62218b9d934785a33b527bb2afdac6.png)
