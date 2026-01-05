# HƯỚNG DẪN TRUYỀN DỮ LIỆU DATABASE POSTGRESQL TỪ DOCKER SANG AIVEN CONSOLE

## TỔNG QUAN
Hướng dẫn này mô tả quy trình chuyển dữ liệu từ PostgreSQL database chạy trong Docker container sang Aiven cloud database.

---

## BƯỚC 1: TẠO FILE BACKUP (.dump) TỪ DATABASE DOCKER

### Cú pháp lệnh:
```bash
docker exec -t <container_name> pg_dump [options] -U <db_user> -d <db_name> -f <file_path_in_container>
```

### Các tham số quan trọng:
- `-t`: Chạy lệnh trong container
- `-U <db_user>`: Username của database
- `-d <db_name>`: Tên database cần backup
- `-F c`: Format custom (binary format, nén tốt hơn)
- `-f <file_path>`: Đường dẫn file output trong container

### Ví dụ thực tế:
```bash
docker exec -t aiven-console pg_dump -U manhhuy -d db_test_aiven -F c -f /tmp/db_test_aiven.dump
```

### Giải thích:
- `aiven-console`: Tên container Docker
- `manhhuy`: Username PostgreSQL
- `db_test_aiven`: Tên database cần backup
- `/tmp/db_test_aiven.dump`: File backup sẽ được tạo trong container tại thư mục /tmp

### Các option khác có thể dùng:
- `-F p`: Plain text SQL format (dễ đọc nhưng kích thước lớn)
- `-F t`: Tar format
- `--no-owner`: Không backup thông tin owner
- `--no-acl`: Không backup thông tin phân quyền
- `-v`: Verbose mode (hiển thị chi tiết quá trình)

---

## BƯỚC 2: COPY FILE BACKUP TỪ CONTAINER RA MÁY LOCAL

### Cú pháp lệnh:
```bash
docker cp <container_name>:<file_path_in_container> <local_destination_path>
```

### Ví dụ thực tế:
```bash
docker cp aiven-console:/tmp/db_test_aiven.dump D:\2.LearnAll\docker\aiven-console\
```

### Giải thích:
- `aiven-console:/tmp/db_test_aiven.dump`: Đường dẫn file trong container
- `D:\2.LearnAll\docker\aiven-console\`: Thư mục đích trên máy local

### Kiểm tra file đã copy thành công:
```bash
dir D:\2.LearnAll\docker\aiven-console\db_test_aiven.dump
```

---

## BƯỚC 3: COPY FILE BACKUP VÀO CONTAINER (Nếu cần)

Nếu file backup đã ở máy local và cần đưa vào container để restore:

```bash
docker cp D:\2.LearnAll\docker\aiven-console\db_test_aiven.dump aiven-console:/tmp/
```

---

## BƯỚC 4: RESTORE DỮ LIỆU LÊN AIVEN DATABASE

### Cú pháp lệnh:
```bash
docker exec -e PGSSLMODE=require -e PGPASSWORD=<password> -it <container_name> pg_restore -h <aiven_host> -p <aiven_port> -U <aiven_user> -d <aiven_database> [options] <dump_file_path>
```

### Ví dụ thực tế:
```bash
docker exec -e PGSSLMODE=require -e PGPASSWORD=YOUR_AIVEN_PASSWORD -it aiven-console pg_restore -h pg-test-example-c41b.l.aivencloud.com -p 10161 -U avnadmin -d defaultdb --no-owner /tmp/db_test_aiven.dump
```

### Giải thích các tham số:

#### Biến môi trường:
- `-e PGSSLMODE=require`: Bắt buộc kết nối SSL (Aiven yêu cầu)
- `-e PGPASSWORD=<password>`: Mật khẩu database (để tránh nhập thủ công)

#### Thông tin kết nối Aiven:
- `-h pg-test-example-c41b.l.aivencloud.com`: Hostname của Aiven database
- `-p 10161`: Port của Aiven database
- `-U avnadmin`: Username Aiven (thường là avnadmin)
- `-d defaultdb`: Database đích trên Aiven

#### Options restore:
- `--no-owner`: Không restore thông tin owner (tránh lỗi permission)
- `--no-acl`: Không restore access control lists
- `-c` hoặc `--clean`: Xóa objects trước khi restore
- `-v`: Verbose mode
- `--if-exists`: Chỉ drop objects nếu tồn tại (dùng với -c)

### Lệnh restore an toàn hơn (khuyến nghị):
```bash
docker exec -e PGSSLMODE=require -e PGPASSWORD=YOUR_AIVEN_PASSWORD -it aiven-console pg_restore -h pg-test-example-c41b.l.aivencloud.com -p 10161 -U avnadmin -d defaultdb --no-owner --no-acl -v /tmp/db_test_aiven.dump
```

---

## LƯU Ý QUAN TRỌNG

### 1. Bảo mật:
- ⚠️ Không commit password vào Git
- Nên dùng biến môi trường hoặc file .env để lưu password
- Có thể dùng `~/.pgpass` file để lưu credentials

### 2. Kiểm tra trước khi restore:
```bash
# Kiểm tra kết nối đến Aiven
docker exec -e PGSSLMODE=require -e PGPASSWORD=YOUR_PASSWORD -it aiven-console psql -h <aiven_host> -p <port> -U <user> -d <database> -c "\l"

# Xem nội dung file dump
docker exec -it aiven-console pg_restore --list /tmp/db_test_aiven.dump
```

### 3. Xử lý lỗi thường gặp:

#### Lỗi permission:
- Thêm `--no-owner --no-acl` vào lệnh pg_restore

#### Lỗi SSL:
- Đảm bảo có `-e PGSSLMODE=require`
- Kiểm tra certificate nếu cần: `-e PGSSLMODE=verify-full`

#### Lỗi database đã có dữ liệu:
- Thêm `-c --if-exists` để clean trước khi restore
- Hoặc tạo database mới trước

### 4. Backup database Aiven trước khi restore:
```bash
docker exec -e PGSSLMODE=require -e PGPASSWORD=YOUR_PASSWORD -t aiven-console pg_dump -h <aiven_host> -p <port> -U <user> -d <database> -F c -f /tmp/aiven_backup_before_restore.dump
```

---

## QUY TRÌNH ĐẦY ĐỦ (TỔNG HỢP)

```bash
# 1. Backup từ Docker
docker exec -t aiven-console pg_dump -U manhhuy -d db_test_aiven -F c -f /tmp/db_test_aiven.dump

# 2. Copy ra máy local (optional, để lưu trữ)
docker cp aiven-console:/tmp/db_test_aiven.dump D:\2.LearnAll\docker\aiven-console\

# 3. Kiểm tra kết nối Aiven
docker exec -e PGSSLMODE=require -e PGPASSWORD=YOUR_AIVEN_PASSWORD -it aiven-console psql -h pg-test-example-c41b.l.aivencloud.com -p 10161 -U avnadmin -d defaultdb -c "\l"

# 4. Restore lên Aiven
docker exec -e PGSSLMODE=require -e PGPASSWORD=YOUR_AIVEN_PASSWORD -it aiven-console pg_restore -h pg-test-example-c41b.l.aivencloud.com -p 10161 -U avnadmin -d defaultdb --no-owner --no-acl -v /tmp/db_test_aiven.dump

# 5. Verify dữ liệu sau khi restore
docker exec -e PGSSLMODE=require -e PGPASSWORD=YOUR_AIVEN_PASSWORD -it aiven-console psql -h pg-test-example-c41b.l.aivencloud.com -p 10161 -U avnadmin -d defaultdb -c "\dt"
```

---

## TÀI LIỆU THAM KHẢO

- PostgreSQL pg_dump: https://www.postgresql.org/docs/current/app-pgdump.html
- PostgreSQL pg_restore: https://www.postgresql.org/docs/current/app-pgrestore.html
- Aiven Documentation: https://docs.aiven.io/docs/products/postgresql