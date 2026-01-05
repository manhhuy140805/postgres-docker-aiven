# HƯỚNG DẪN TRUYỀN DỮ LIỆU DATABASE POSTGRESQL Ở DOCKER SANG DATABASE Ở AIVEN CONSOLE

## BƯỚC 1: TẠO FILE .dump TỪ DATABASE DOCKER

**Cú pháp:**
```bash
docker exec -t <container_name> pg_dump [options] -U <db_user> -d <db_name> -f <file_path_in_container>
```

**Ví dụ:**
```bash
docker exec -t aiven-console pg_dump -U manhhuy -d db_test_aiven -F c -f /tmp/db_test_aiven.dump
```

---

## BƯỚC 2: COPY SANG THƯ MỤC HIỆN TẠI

**Cú pháp:**
```bash
docker cp <container_name>:<path_in_container> <path_on_host>
```

**Ví dụ:**
```bash
docker cp aiven-console:/tmp/db_test_aiven.dump D:\2.LearnAll\docker\aiven-console\
```

---

## BƯỚC 3: TRUYỀN FILE .dump SANG CHO DATABASE Ở AIVEN CONSOLE

**Cú pháp:**
```bash
docker exec -e PGSSLMODE=require -e PGPASSWORD="YOUR_AIVEN_PASSWORD" -it aiven-console pg_restore -h YOUR_AIVEN_HOST -p YOUR_AIVEN_PORT -U YOUR_AIVEN_USER -d YOUR_AIVEN_DB --no-owner /tmp/db_test_aiven.dump
```

**Ví dụ:**
```bash
docker exec -e PGSSLMODE=require -e PGPASSWORD=YOUR_AIVEN_PASSWORD -it aiven-console pg_restore -h pg-test-example-c41b.l.aivencloud.com -p 10161 -U avnadmin -d defaultdb --no-owner /tmp/db_test_aiven.dump
```
