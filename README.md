# PostgreSQL Database Migration - Docker ⇄ Aiven

Dự án này cung cấp hướng dẫn và công cụ để di chuyển dữ liệu PostgreSQL giữa Docker container và Aiven cloud database.

## 📋 Mục Lục

- [Tổng Quan](#tổng-quan)
- [Cấu Trúc Dự Án](#cấu-trúc-dự-án)
- [Yêu Cầu Hệ Thống](#yêu-cầu-hệ-thống)
- [Cài Đặt và Khởi Động](#cài-đặt-và-khởi-động)
- [Hướng Dẫn Sử Dụng](#hướng-dẫn-sử-dụng)
- [Thông Tin Kết Nối](#thông-tin-kết-nối)
- [Xử Lý Sự Cố](#xử-lý-sự-cố)

---

## 🎯 Tổng Quan

Dự án này bao gồm:

- **Docker Compose Setup**: PostgreSQL 18 + pgAdmin 4
- **Migration Scripts**: Công cụ di chuyển dữ liệu hai chiều
- **Documentation**: Hướng dẫn chi tiết từng bước

### Các Tính Năng Chính

✅ Backup và restore PostgreSQL database  
✅ Di chuyển dữ liệu từ Docker sang Aiven  
✅ Di chuyển dữ liệu từ Aiven về Docker  
✅ Quản lý database qua pgAdmin web interface  
✅ Hỗ trợ SSL connection cho Aiven  

---

## 📁 Cấu Trúc Dự Án

```
aiven-console/
├── docker-compose.yml              # Cấu hình Docker services
├── README.md                       # File này
├── HuongDanChiTiet.md             # Hướng dẫn chi tiết Docker → Aiven
├── HuongDan-Docker-to-Aiven.md    # Hướng dẫn tóm tắt Docker → Aiven (có thể sử dụng ngay)
├── HuongDan-Aiven-to-Docker.md    # Hướng dẫn Aiven → Docker (có thể sử dụng ngay)
├── db_test_aiven.dump             # File backup database (file back-up mẫu được xuất từ docker)
└── dump-defaultdb-202601052100.sql # File backup database (SQL format được xuất từ DBeaver)
```


## 🚀 Cài Đặt và Khởi Động

### 1. Clone hoặc tải dự án về

```bash
cd D:\2.LearnAll\docker\aiven-console
```

### 2. Khởi động Docker containers

```bash
docker-compose up -d
```

### 3. Kiểm tra containers đang chạy

```bash
docker ps
```

Bạn sẽ thấy 2 containers:
- `aiven-console` (PostgreSQL)
- `pgadmin` (pgAdmin 4)

### 4. Truy cập pgAdmin

Mở trình duyệt và truy cập: http://localhost:5050

**Thông tin đăng nhập:**
- Email: `root@gmail.com`
- Password: `123123`

---

## 📖 Hướng Dẫn Sử Dụng

### 🔄 Di Chuyển Dữ Liệu: Docker → Aiven

File hướng dẫn đủ để thực hiện: [HuongDan-Docker-to-Aiven.md](HuongDan-Docker-to-Aiven.md)
Chi tiết đầy đủ xem file: [HuongDanChiTiet.md](HuongDanChiTiet.md)

**Quy trình tóm tắt:**

```bash
# 1. Tạo backup từ Docker
docker exec -t aiven-console pg_dump -U manhhuy -d db_test_aiven -F c -f /tmp/db_test_aiven.dump

# 2. Copy file backup ra máy local (optional)
docker cp aiven-console:/tmp/db_test_aiven.dump D:\2.LearnAll\docker\aiven-console\

# 3. Restore lên Aiven
docker exec -e PGSSLMODE=require -e PGPASSWORD=YOUR_PASSWORD -it aiven-console pg_restore \
  -h YOUR_AIVEN_HOST -p YOUR_PORT -U avnadmin -d defaultdb \
  --no-owner --no-acl -v /tmp/db_test_aiven.dump
```

### 🔙 Di Chuyển Dữ Liệu: Aiven → Docker

Chi tiết xem file: [HuongDan-Aiven-to-Docker.md](HuongDan-Aiven-to-Docker.md)

**Quy trình:**

1. **Xuất dữ liệu từ Aiven qua pgAdmin:**
   - Click phải vào database → Tools → Backup
   - Format: PLAIN
   - Encoding: UTF-8
   - Chọn "Discard objects owner"
   - Lưu file SQL

2. **Restore vào Docker:**
   - Kết nối đến Docker database trong pgAdmin
   - Click phải vào database → Restore
   - Format: PLAIN
   - Chọn file SQL đã xuất
   - Thực hiện restore

---

## 🔌 Thông Tin Kết Nối

### Docker PostgreSQL

| Thông Tin | Giá Trị |
|-----------|---------|
| Host | `localhost` |
| Port | `5432` |
| Database | `db_test_aiven` |
| Username | `manhhuy` |
| Password | `123123` |
| Container | `aiven-console` |

### pgAdmin

| Thông Tin | Giá Trị |
|-----------|---------|
| URL | http://localhost:5050 |
| Email | `root@gmail.com` |
| Password | `123123` |

### Aiven PostgreSQL (Ví dụ)

| Thông Tin | Giá Trị |
|-----------|---------|
| Host | `pg-example-c41b.l.aivencloud.com` |
| Port | `10161` |
| Database | `defaultdb` |
| Username | `avnadmin` |
| Password | `YOUR_AIVEN_PASSWORD` |
| SSL Mode | `require` |

⚠️ **Lưu ý bảo mật:** Thay đổi password mặc định trong production!

---

## 🛠️ Các Lệnh Hữu Ích

### Quản lý Docker Containers

```bash
# Khởi động services
docker-compose up -d

# Dừng services
docker-compose down

# Xem logs
docker-compose logs -f

# Xem logs của container cụ thể
docker logs aiven-console
docker logs pgadmin

# Restart services
docker-compose restart

# Xóa containers và volumes (⚠️ mất dữ liệu)
docker-compose down -v
```

### Làm việc với PostgreSQL

```bash
# Kết nối vào PostgreSQL container
docker exec -it aiven-console psql -U manhhuy -d db_test_aiven

# Liệt kê databases
docker exec -it aiven-console psql -U manhhuy -c "\l"

# Liệt kê tables trong database
docker exec -it aiven-console psql -U manhhuy -d db_test_aiven -c "\dt"

# Chạy SQL query
docker exec -it aiven-console psql -U manhhuy -d db_test_aiven -c "SELECT * FROM your_table LIMIT 10;"
```

### Backup và Restore

```bash
# Backup database (custom format)
docker exec -t aiven-console pg_dump -U manhhuy -d db_test_aiven -F c -f /tmp/backup.dump

# Backup database (SQL format)
docker exec -t aiven-console pg_dump -U manhhuy -d db_test_aiven -F p -f /tmp/backup.sql

# Restore từ custom format
docker exec -i aiven-console pg_restore -U manhhuy -d db_test_aiven /tmp/backup.dump

# Restore từ SQL format
docker exec -i aiven-console psql -U manhhuy -d db_test_aiven < backup.sql
```

---

## 🔧 Xử Lý Sự Cố

### Container không khởi động được

```bash
# Kiểm tra logs
docker-compose logs

# Kiểm tra port đã bị sử dụng chưa
netstat -ano | findstr :5432
netstat -ano | findstr :5050

# Xóa và tạo lại containers
docker-compose down
docker-compose up -d
```

### Không kết nối được đến PostgreSQL

1. Kiểm tra container đang chạy: `docker ps`
2. Kiểm tra logs: `docker logs aiven-console`
3. Kiểm tra firewall/antivirus có block port 5432 không
4. Thử restart container: `docker restart aiven-console`

### Lỗi khi restore lên Aiven

**Lỗi permission denied:**
```bash
# Thêm --no-owner --no-acl
docker exec -e PGSSLMODE=require -e PGPASSWORD=xxx -it aiven-console pg_restore \
  -h xxx -p xxx -U avnadmin -d defaultdb --no-owner --no-acl /tmp/backup.dump
```

**Lỗi SSL connection:**
```bash
# Đảm bảo có PGSSLMODE=require
docker exec -e PGSSLMODE=require -e PGPASSWORD=xxx ...
```

**Database đã có dữ liệu:**
```bash
# Thêm -c --if-exists để clean trước
docker exec -e PGSSLMODE=require -e PGPASSWORD=xxx -it aiven-console pg_restore \
  -h xxx -p xxx -U avnadmin -d defaultdb --no-owner --no-acl -c --if-exists /tmp/backup.dump
```

### pgAdmin không truy cập được

1. Kiểm tra container: `docker ps | grep pgadmin`
2. Kiểm tra logs: `docker logs pgadmin`
3. Thử truy cập: http://localhost:5050
4. Clear browser cache và thử lại
5. Restart container: `docker restart pgadmin`

---

## 📚 Tài Liệu Tham Khảo

- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/)
- [pg_dump Documentation](https://www.postgresql.org/docs/current/app-pgdump.html)
- [pg_restore Documentation](https://www.postgresql.org/docs/current/app-pgrestore.html)
- [Aiven PostgreSQL Documentation](https://docs.aiven.io/docs/products/postgresql)
- [pgAdmin Documentation](https://www.pgadmin.org/docs/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

---

## 📝 Ghi Chú

- Timezone được set là `Asia/Ho_Chi_Minh`
- PostgreSQL version: 18
- pgAdmin version: Latest (dpage/pgadmin4)
- Data được lưu trong Docker volumes: `postgres_data` và `pgadmin_data`
- Backup files được lưu trong thư mục dự án

---

## ⚠️ Cảnh Báo Bảo Mật

- ❌ **KHÔNG** commit passwords vào Git
- ❌ **KHÔNG** sử dụng passwords mặc định trong production
- ✅ Sử dụng biến môi trường hoặc file `.env`
- ✅ Thêm `.env` vào `.gitignore`
- ✅ Sử dụng SSL khi kết nối đến Aiven
- ✅ Backup dữ liệu thường xuyên

---

**Phiên bản:** 1.0  
**Cập nhật lần cuối:** 05/01/2025  
**Tác giả:** Trần Đình Mạnh Huy
