# 🚀 คู่มือการติดตั้ง WordPress บน Windows ด้วย Docker

> **เอกสารฉบับสมบูรณ์** สำหรับการติดตั้ง WordPress ผ่าน Docker + Docker Compose บน Windows แบบใช้งานได้จริง พร้อมระบบจัดการฐานข้อมูลด้วย phpMyAdmin

---

## 🛠 1. เตรียมเครื่องมือ

1. **ติดตั้ง Docker Desktop**
   - ดาวน์โหลด: [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
   - ✅ เปิดใช้งาน **WSL2** ตามคำแนะนำ
   - ✅ รีสตาร์ทเครื่องหลังติดตั้งเสร็จ
2. ตรวจสอบการติดตั้ง Docker และ Docker Compose

```powershell
# ตรวจสอบ Docker
docker --version
```

```powershell
# ตรวจสอบ Docker Compose
docker-compose --version
```

**ผลลัพธ์ที่คาดหวัง:**

```bash
Docker version 24.x.x
Docker Compose version v2.x.x
```

## 📝 2. สร้างไฟล์ docker-compose.yml

สร้างไฟล์ docker-compose.yml ด้วยเนื้อหานี้:

```yaml
version: '3.9'

services:

  # ตั้งค่า MySQL 5.7
  db:
    image: mysql:5.7
    container_name: wordpress_db # ชื่อคอนเทนเนอร์
    restart: always # รีสตาร์ทคอนเทนเนอร์อัตโนมัติเมื่อหยุดทำงาน
    environment:
      MYSQL_DATABASE: wordpress # ชื่อฐานข้อมูล
      MYSQL_USER: wordpress # ชื่อผู้ใช้ฐานข้อมูล
      MYSQL_PASSWORD: wordpress # รหัสผ่านผู้ใช้ฐานข้อมูล
      MYSQL_ROOT_PASSWORD: rootpassword # รหัสผ่านผู้ใช้ root
    volumes:
      - ./db_data:/var/lib/mysql # db_data โฟลเดอร์ของฐานข้อมูล สามารถเปลี่ยนแปลงได้ตามต้องการ  
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      retries: 5
      start_period: 10s
      timeout: 5s

  # ตั้งค่า WordPress
  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306 # ชื่อโฮสต์และพอร์ตของฐานข้อมูล
      WORDPRESS_DB_USER: wordpress # ชื่อผู้ใช้ฐานข้อมูล
      WORDPRESS_DB_PASSWORD: wordpress # รหัสผ่านผู้ใช้ฐานข้อมูล
      WORDPRESS_DB_NAME: wordpress # ชื่อฐานข้อมูล
    volumes:
      - ./wordpress_data:/var/www/html  # wordpress_data โฟลเดอร์ของ WordPress สามารถเปลี่ยนแปลงได้ตามต้องการ
    depends_on:
      db:
        condition: service_healthy

  # ตั้งค่า phpMyAdmin
  phpmyadmin:
    image: phpmyadmin/phpmyadmin # ใช้ image phpMyAdmin ล่าสุด
    container_name: phpmyadmin_app # ชื่อคอนเทนเนอร์
    restart: always # รีสตาร์ทคอนเทนเนอร์อัตโนมัติเมื่อหยุดทำงาน
    environment:
      PMA_HOST: db # ชื่อโฮสต์ของฐานข้อมูล
      PMA_PORT: 3306 # พอร์ตของฐานข้อมูล
      PMA_USER: root # ชื่อผู้ใช้ฐานข้อมูล
      PMA_PASSWORD: rootpassword # รหัสผ่านผู้ใช้ฐานข้อมูล 
      PMA_ABSOLUTE_URI: http://localhost:8081/ # URL สำหรับเข้าถึง phpMyAdmin
    ports:
      - "8081:80"
    depends_on:
      - db

```

> **💡 เทคนิค:** ใช้ path แบบ `D:/...` เท่านั้น และไม่ใช้ backslash `\` บน Windows

## 🧹 3. ล้างข้อมูลเก่า

หากเคยติดตั้งมาก่อนและต้องการเริ่มใหม่:

```powershell
# หยุดและลบ container + volume
docker-compose down -v

# ลบโฟลเดอร์ข้อมูล (ถ้าต้องการ)
Remove-Item -Recurse -Force "./db_data"
Remove-Item -Recurse -Force "./wordpress_data"
```

## ▶️ 4. รันและติดตั้ง

```powershell
docker-compose up -d
```

### ⏳ รอให้ระบบพร้อม (10-20 วินาที)

Docker Compose จะสร้าง `db_data` และ `wordpress_data` ให้เองอัตโนมัติจากที่ประกาศไว้ใน `volumes:`

```bash
  volumes:
    # db_data โฟลเดอร์ของฐานข้อมูล สามารถเปลี่ยนแปลงชื่อได้ตามต้องการ
    - ./db_data:/var/lib/mysql 

  volumes:
    # wordpress_data โฟลเดอร์ของ WordPress สามารถเปลี่ยนแปลงชื่อได้ตามต้องการ
    - ./wordpress_data:/var/www/html
```

### ✅ ตรวจสอบสถานะ

```powershell
# ดู container ที่รันอยู่
docker ps

# ตรวจสอบ log
docker-compose logs -f wordpress
docker-compose logs -f db
```

ถ้าอยากสร้างเฉพาะ volume เองโดยตรง (ไม่ต้องรัน service) ก็ทำได้ เช่น:

```bash
docker volume create db_data
docker volume create wordpress_data
```

### ⚠️ ตรวจสอบ log ของ WordPress/MySQL

```powershell
docker-compose logs -f wordpress
docker-compose logs -f db
```

ถ้าหาก Clone ไปใช้งาน ใช้วิธีนี้:

1. ลง container เดิมก่อน:

```bash
docker-compose down
```

2. รันใหม่:

```bash
docker-compose up -d
```

### อัปเดต wordpress ล่าสุด

```powershell
# อัปเดต images WordPress ล่าสุด (ถ้าต้องการ)
docker pull wordpress:latest

# wordpress:latest (wordpress คือ ชื่อ image สามารถเปลี่ยนได้ตามต้องการ)
```

## 🌐 5. ตั้งค่า WordPress

### 🌍 เข้าสู่หน้าติดตั้ง
1. เปิด browser → **http://localhost:8080**
2. เลือกภาษา → **ไทย** (หรือตามต้องการ)

### 📝 กรอกข้อมูลเริ่มต้น
| ฟิลด์ | ค่าที่แนะนำ |
|-------|-------------|
| **ชื่อไซต์** | ชื่อเว็บไซต์ของคุณ |
| **ชื่อผู้ใช้** | admin (หรือชื่อที่ต้องการ) |
| **รหัสผ่าน** | รหัสผ่านที่แข็งแรง |
| **อีเมล** | อีเมลสำหรับติดต่อ |
| **การค้นหา** | ☑️ เปิดให้ search engine หาได้ |

### ✨ เสร็จสิ้นการติดตั้ง
3. กด **"ติดตั้ง WordPress"**
4. เมื่อเสร็จแล้ว กด **"เข้าสู่ระบบ"**
5. ยินดีต้อนรับสู่ WordPress Dashboard! 🎉

## ⚡ 6. คำสั่งที่ควรรู้

### 🔧 การจัดการ Container

| คำสั่ง | ความหมาย | ตัวอย่าง |
|--------|-----------|----------|
| `docker-compose up -d` | เริ่มทำงาน (background) | เริ่ม WordPress |
| `docker-compose down` | หยุดทำงาน | หยุด WordPress |
| `docker-compose restart` | รีสตาร์ท | รีสตาร์ททั้งหมด |
| `docker-compose restart wordpress` | รีสตาร์ทเฉพาะ service | รีสตาร์ท WordPress อย่างเดียว |

### 🔧 การจัดการ Container

| คำสั่ง | ใช้ทำอะไร |
|--------|-----------|
| `docker exec -it wordpress_app bash` | เข้า WordPress container |
| `docker exec -it wordpress_db mysql -uwordpress -pwordpress` | เข้า MySQL console |
| `docker logs wordpress_app` | ดู log ของ WordPress |


## 🔗 7. การเชื่อมต่อฐานข้อมูล

| Service | URL/Host | Username | Password | Database |
|---------|----------|----------|----------|----------|
| **🌐 WordPress** | http://localhost:8080 | admin | (ที่ตั้งไว้) | - |
| **📊 phpMyAdmin** | http://localhost:8081 | `wordpress` | `wordpress` | `wordpress` |
| **🗄️ MySQL (Host)** | `127.0.0.1:3306` | `wordpress` | `wordpress` | `wordpress` |

### 🔌 การเชื่อมต่อจากแอปพลิเคชันภายนอก

**MySQL Workbench / DBeaver / HeidiSQL:**

```bash
Host: 127.0.0.1
Port: 3306
User: wordpress
Password: wordpress
Database: wordpress
```

### 🏗️ สถาปัตยกรรมระบบ

```bash
Browser → WordPress:8080 → MySQL:3306
       → phpMyAdmin:8081 →
```

### ✅ การใช้งานต่อ

### 🎨 การปรับแต่งเว็บไซต์

- **🎭 เปลี่ยน Theme:** `Appearance → Themes`
- **🔌 ติดตั้ง Plugin:** `Plugins → Add New`
- **📄 สร้างหน้าใหม่:** `Pages → Add New`
- **📝 เขียนโพสต์:** `Posts → Add New`

### ⚙️ การตั้งค่าที่สำคัญ

- **🔗 Permalinks:** `Settings → Permalinks` → เลือก Post name
- **👥 ผู้ใช้:** `Users` → เพิ่มผู้ใช้ใหม่
- **🔒 Security:** ติดตั้ง plugin security เพิ่มเติม

### 💾 การสำรองข้อมูล

```powershell
# สำรอง WordPress files
docker exec wordpress_app tar czf /tmp/wordpress-backup.tar.gz /var/www/html
docker cp wordpress_app:/tmp/wordpress-backup.tar.gz ./

# สำรอง Database
docker exec wordpress_db mysqldump -uwordpress -pwordpress wordpress > wordpress-db-backup.sql
```

### 🔄 การอัปเดต

```powershell
# อัปเดต images
docker-compose pull

# รีสตาร์ทด้วย image ใหม่
docker-compose up -d --force-recreate
```

### 🎯 เคล็ดลับและการแก้ปัญหา

### ⚠️ ปัญหาที่พบบ่อย

| ปัญหา | วิธีแก้ไข |
|-------|----------|
| **Port 8080 ถูกใช้แล้ว** | เปลี่ยน port เป็น `"8090:80"` |
| **MySQL ไม่ขึ้น** | รอ 30 วินาที หรือ restart |
| **Permission denied** | รัน PowerShell แบบ Administrator |

### 💡 Tips ขั้นสูง

- **🔒 Security:** เปลี่ยนรหัสผ่าน default ทั้งหมด
- **🚀 Performance:** ใช้ Redis cache plugin
- **🔍 SEO:** ติดตั้ง Yoast SEO plugin
- **📱 Mobile:** ทดสอบ responsive design

---
<br>

# คู่มือเชื่อมต่อฐานข้อมูล WordPress + MySQL + PHPMyAdmin (Docker)

### 1️⃣ ข้อมูลสำคัญจาก docker-compose.yml

| Service              | Database Host | Database Name | Username    | Password    | Port                 |
| -------------------- | ------------- | ------------- | ----------- | ----------- | -------------------- |
| MySQL Container      | `db`          | `wordpress`   | `wordpress` | `wordpress` | `3306` (internal)    |
| WordPress Container  | `db:3306`     | `wordpress`   | `wordpress` | `wordpress` | 8080 (WordPress Web) |
| PHPMyAdmin Container | `db:3306`     | `wordpress`   | `wordpress` | `wordpress` | 8081 (Web GUI)       |

## 2️⃣ การตั้งค่า MySQL และ WordPress

- `MYSQL_ROOT_PASSWORD: rootpassword` → รหัสผ่าน root สำหรับ MySQL (ใช้ในกรณีต้องการ admin access)
- WordPress จะเชื่อมต่อ DB ผ่าน Docker network ชื่อ db


## 2️⃣ การเชื่อมต่อจาก WordPress

```yaml
WORDPRESS_DB_HOST: db:3306
WORDPRESS_DB_USER: wordpress
WORDPRESS_DB_PASSWORD: wordpress
WORDPRESS_DB_NAME: wordpress
```

- WordPress จะเชื่อม MySQL container ผ่าน service name db โดยไม่ต้อง expose port
- หากรันครั้งแรก MySQL container จะสร้าง database และ user ให้เองตาม environment

## 3️⃣ การเชื่อมต่อจาก PHPMyAdmin

- URL: http://localhost:8081
- Server: db
- Username: wordpress
- Password: wordpress
- สามารถดู/แก้ไข database, table, user ได้

## 4️⃣ การเชื่อมต่อจากเครื่องตัวเอง (Host machine)

ใช้ MySQL client เช่น MySQL Workbench / DBeaver / HeidiSQL
- Host: `127.0.0.1` (ถ้า expose port 3306)
- Port: `3306`
- Username: `wordpress`
- Password: `wordpress`
- Database: `wordpress`

⚠️ หาก MySQL host ใช้ port 3306 อยู่แล้ว → ต้อง comment port ของ MySQL container หรือใช้ port อื่น เช่น `"3307:3306"`

## 5️⃣ คำสั่งตรวจสอบและจัดการ container
```powershell
docker-compose up -d           # สร้างและรัน container
docker-compose down -v         # หยุดและลบ container + volume
docker ps                      # ดู container ที่รันอยู่
docker logs wordpress_db       # ดู log ของ MySQL
docker logs wordpress_app      # ดู log ของ WordPress
```
## 6️⃣ สรุป Workflow การเชื่อมต่อ
```text
+----------------+           +-----------------+           +-----------------+
| WordPress      | <-------> | MySQL Container | <-------> | PHPMyAdmin GUI  |
| Container      |           | (db)            |           | (localhost:8081)|
+----------------+           +-----------------+           +-----------------+
```
- WordPress container → เชื่อม MySQL container ผ่าน service name db
- PHPMyAdmin → เชื่อม MySQL container ผ่าน service name db
- Host machine → เชื่อม MySQL container ผ่าน port expose หรือเชื่อม MySQL host โดยตรง

---

# ขั้นตอนการย้าย WordPress ที่อยู่บน Docker ไปใช้ Laragon / XAMPP / WAMP / LAMP

## 1️⃣ เตรียมไฟล์ WordPress จาก Docker
1. เข้าถึง container ของ WordPress

```powershell
docker ps
docker exec -it <container_name> /bin/bash # container_name คือชื่อ container ของ WordPress
```

2. คัดลอกไฟล์ทั้งหมดใน โฟลเดอร์ WordPress (เช่น `/var/www/html`) ออกมา
  - ใช้ docker cp เช่น:

```powershell
docker cp <container_name>:/var/www/html ./wordpress
```

  - จะได้โฟลเดอร์ `wordpress` บนเครื่องคุณ


## 2️⃣ Export ฐานข้อมูล

1. เข้าไปใน container ของ MySQL/MariaDB:

```bash
docker exec -it <mysql_container_name> bash # mysql_container_name คือชื่อ container ของ MySQL/MariaDB เช่นwordpress_db
```

2. ใช้คำสั่ง `mysqldump` export database:

```bash
mysqldump -u root -p wordpress_db > /tmp/wordpress.sql
```

หรือ

```bash
docker exec -i wordpress_db mysqldump -u wordpress -pwordpress wordpress > wordpress.sql
```

3. copy ไฟล์ออกมาที่เครื่อง host:

```bash
docker cp <mysql_container_name>:/tmp/wordpress.sql ./wordpress.sql
```

## 3️⃣ ติดตั้ง Laragon / XAMPP / WAMP / LAMP

  - สมมติติดตั้งแล้ว และใช้ htdocs หรือ www เป็นโฟลเดอร์เว็บ
  - วาง โฟลเดอร์ WordPress ลงไป เช่น:

```bash
C:\laragon\www\wordpress
```

## 4️⃣ Import Database ไปยัง XAMPP/Laragon

1. เปิด phpMyAdmin หรือใช้ command line
2. สร้าง database ใหม่ (เช่น wordpress_db)
3. Import ไฟล์ .sql ที่ export มาจาก Docker

  - phpMyAdmin → Import → เลือกไฟล์ .sql
  - หรือ command line:

```bash
mysql -u root -p wordpress_db < wordpress.sql
```

## 5️⃣ แก้ไข wp-config.php

เปิดไฟล์ `wp-config.php` ในโฟลเดอร์ WordPress แล้วแก้ไขค่าต่อไปนี้ให้ตรงกับสภาพแวดล้อมใหม่:

```php
define('DB_NAME', 'wordpress_db');    // ชื่อฐานข้อมูลใหม่
define('DB_USER', 'root');            // user ของ XAMPP/Laragon/WAMP
define('DB_PASSWORD', '');            // password ของ XAMPP/Laragon/WAMP (ปกติไม่มี)
define('DB_HOST', 'localhost');       // database host
```

## 6️⃣ ตรวจสอบ Permalinks และ URL

  - ถ้า URL ของเว็บเปลี่ยน เช่น จาก http://localhost:8000 เป็น http://localhost/wordpress
  - แก้ไขในฐานข้อมูลด้วย SQL:

```sql
  UPDATE wp_options SET option_value = 'http://localhost/wordpress' WHERE option_name = 'siteurl';
  UPDATE wp_options SET option_value = 'http://localhost/wordpress' WHERE option_name = 'home';
```

  - หรือใช้ plugin เช่น Better Search Replace แก้ URL ทุกที่

## 7️⃣ ทดสอบเว็บไซต์

1. เปิด browser → `http://localhost/wordpress`
2. ถ้าเจอ Error เกี่ยวกับ wp-content/plugins ให้ปิด `plugin` ชั่วคราว (เปลี่ยนชื่อโฟลเดอร์ plugins เป็น `plugins_old`)

---

# ขั้นตอนการเปลี่ยนชื่อโฟลเดอร์ และ ชื่อฐานข้อมูล

## 1️⃣ เปลี่ยนชื่อโฟลเดอร์เว็บ

  - ถ้าเดิมคุณวางโฟลเดอร์ไว้:

```bash
C:\laragon\www\wordpress
```

  - เปลี่ยนเป็นชื่ออื่น เช่น:

```bash
C:\laragon\www\wordpress_site1
C:\laragon\www\wordpress_site2
```

  - URL ที่เข้าจะเปลี่ยนตามชื่อโฟลเดอร์ เช่น:

```bash
http://localhost/wordpress_site1
http://localhost/wordpress_site2
```

## 2️⃣ เปลี่ยนชื่อฐานข้อมูล

- ตอนสร้าง database ใหม่ ให้ตั้งชื่อแตกต่างกัน เช่น:

```nginx
wordpress_db1
wordpress_db2
```

## 3️⃣ ปรับ wp-config.php ของแต่ละเว็บ

- เปิดไฟล์ wp-config.php ของแต่ละเว็บ
- แก้ค่าให้ตรงกับ database ของเว็บนั้น ๆ

```php
define('DB_NAME', 'wordpress_db1'); // เปลี่ยนตาม database ใหม่
define('DB_USER', 'root');           
define('DB_PASSWORD', '');           
define('DB_HOST', 'localhost');      
```

- สำหรับเว็บที่ 2:

```php
define('DB_NAME', 'wordpress_db2');
```

## 4️⃣ ปรับ URL ในฐานข้อมูล

- ถ้าเปลี่ยนชื่อโฟลเดอร์เว็บ ต้องแก้ค่า siteurl และ home ในฐานข้อมูลของเว็บนั้น

```sql
UPDATE wp_options SET option_value = 'http://localhost/wordpress_site1' WHERE option_name = 'siteurl';
UPDATE wp_options SET option_value = 'http://localhost/wordpress_site1' WHERE option_name = 'home';
```

- สำหรับเว็บที่ 2:

```sql
UPDATE wp_options SET option_value = 'http://localhost/wordpress_site2' WHERE option_name = 'siteurl';
UPDATE wp_options SET option_value = 'http://localhost/wordpress_site2' WHERE option_name = 'home';
```

## 5️⃣ สรุปข้อสำคัญ

- โฟลเดอร์เว็บ → เปลี่ยนได้ตามต้องการ
- Database → ต้องไม่ซ้ำกัน
- wp-config.php → ต้องเชื่อมกับ database ให้ถูกต้อง
- URL → แก้ในฐานข้อมูลให้ตรงกับโฟลเดอร์ใหม่