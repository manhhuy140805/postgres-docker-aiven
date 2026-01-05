# HƯỚNG DẪN CHUYỂN DỮ LIỆU TỪ AIVEN SANG DOCKER

## BƯỚC 1: XUẤT FILE PLAIN SQL

**Thao tác trong pgAdmin:**

1. Click phải vào database → **Tools** → **Backup**
2. Cấu hình backup:
   - **Format:** PLAIN
   - **Encoding:** UTF-8
   - **Discard objects owner:** ✓ (check)
3. Chọn vị trí lưu file
4. Click **Start** để xuất file

---

## BƯỚC 2: RESTORE VÀO DOCKER BẰNG PGADMIN

**Thao tác trong pgAdmin:**

1. Click phải vào database → **Restore**
2. Cấu hình restore:
   - **Format:** PLAIN
   - **Filename:** Chọn file dump.sql đã xuất
3. Click **Restore** để import dữ liệu