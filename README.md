# **Cấu hình MySQL Master-Slave Ubuntu 20.04**

## Yêu cầu:

- Máy Master: 172.20.23.1
- Máy Slave: 172.20.23.2

## Cài đặt MySQL Server trên cả Master và Slave

### 1. Cập nhật và cài đặt MySQL
    apt update
>    
    apt install mysql-server -y
>
    systemctl start mysql
>    
    systemctl enable mysql

### 2. Cấu hình cho Master

Để thay đổi cấu hình mặc định của MySQL, ta chỉnh sửa file `mysqld.cnf`:

    sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

Bỏ dấu `#` khỏi các dòng sau đây:

    pid-file = /var/run/mysqld/mysqld.pid
    socket = /var/run/mysqld/mysqld.sock
    datadir = /var/lib/mysql
    tmpdir = /tmp
    slow_query_log=1
    slow_query_log_file=/var/lib/mysql/mysqld-slow.log
    server-id = 1
    log-bin = /var/log/mysql/mysql-bin.log

Thay đổi `max_binlog_size` thành:

    max_binlog_size = 800M

Thêm vào các dòng sau:

    binlog_format = ROW
    sync_binlog = 1
    expire-logs-days = 5

Thêm `#` vào đầu dòng để ghi chú lại câu lệnh:

    #bind-address =127.0.0.1

![](https://i.imgur.com/OWOCAtg.png)

Thoát và lưu lại thay đổi.

Khởi động lại dịch vụ `mysql`:

    systemctl restart mysql

### 3. Tạo người dùng cho Slave truy cập

Đăng nhập vào `mysql` bằng tài khoản `root` để tạo người dùng:

    mysql -u root -p

Tạo một người dùng `fit` và mật khẩu là `Knn2022@`: 

    CREATE USER 'fit'@'172.20.23.2' IDENTIFIED WITH mysql_native_password BY 'Knn2022@';

Cấp quyền cho người dùng vừa tạo:

    GRANT REPLICATION SLAVE on *.* to fit@172.20.23.2 ;


### 4. Cấu hình cho Slave

Để thay đổi cấu hình mặc định của MySQL, ta chỉnh sửa file `mysqld.cnf`:

    sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

Bỏ ghi chú các dòng:
    
    tmpdir = /tmp
    slow_query_log = 1
    server-id = 1
    log_bin = /var/log/mysql/mysql-bin.log
    
Thay đổi các dòng:

    slow_query_log = 2
    server-id = 2
    max_binlog_size = 800M

Thêm vào các dòng:

    read_only = 1
    binlog_format = ROW
    sync_binlog = 1
    expire-logs-days = 5

Thêm `#` để dòng này thành ghi chú:

    # bind-address = 127.0.0.1

![](https://i.imgur.com/eBWmi84.png)

Thoát và lưu lại thay đổi.

Khởi động lại dịch vụ `mysql`:

    systemctl restart mysql

## Kết nối Slave server với Master server

### Ở máy Master

Đăng nhập vào `mysql` :

    mysql -u root -p

Xem thông tin của master:

    SHOW MASTER STATUS\G

![](https://i.imgur.com/bMGgPxB.png)

Ghi chú lại 2 dòng được khoanh đỏ

- File: mysql-bin.000001
- Position: 678

### Ở máy Slave

Đăng nhập vào `mysql` :

    mysql -u root -p

Dừng chế độ SLAVE:

    STOP SLAVE;

Thiết lập máy Slave để sao chép máy Master:

    CHANGE MASTER TO MASTER_HOST='172.20.23.1', MASTER_USER='fit', MASTER_PASSWORD='Knn2022@', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=678;

Bật lại chế độ SLAVE:

    START SLAVE;

Xác minh trạng thái bằng cách sử dụng truy vấn sau:

    SHOW SLAVE STATUS\G

![](https://i.imgur.com/tR11GOR.png)

![](https://i.imgur.com/19Wy4cz.png)

### Kiểm tra kết nối

## Ở máy Master

Đăng nhập vào `mysql` :

    mysql -u root -p

Tạo cơ sở dữ liệu để kiểm tra:

    CREATE DATABASE test;

Kiểm tra `test` đã được tạo chưa:

    SHOW DATABASES;

## Ở máy Slave

Đăng nhập vào `mysql` :

    mysql -u root -p

Kiểm tra `test` đã được tạo ở máy Master:

    SHOW DATABASES;

## **Kết quả**

![](https://i.imgur.com/oGLqivD.png)
