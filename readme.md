# 🚀 คู่มือการติดตั้ง WordPress บน Windows ด้วย Docker

เอกสารนี้สรุปขั้นตอนการติดตั้ง **WordPress** ผ่าน **Docker + Docker Compose** บน Windows แบบใช้งานได้จริง

---

## 🛠 1. เตรียมเครื่องมือ

1. ติดตั้ง [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)  
   - เปิดใช้งาน **WSL2** ตามคำแนะนำ
2. ตรวจสอบการติดตั้ง Docker และ Docker Compose
```powershell
docker --version
docker-compose --version
```

## 📝 3. สร้างไฟล์ docker-compose.yml
สร้างไฟล์ docker-compose.yml ด้วยเนื้อหานี้:
```yaml
services:
  db:
    image: mysql:5.7
    container_name: wordpress_db
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: rootpassword
    volumes:
      - "D:/wordpress-docker/www/db_data:/var/lib/mysql"

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - "D:/wordpress-docker/www/wordpress:/var/www/html"
    depends_on:
      - db
```
💡 Tip: ใช้ path ของ Windows แบบ D:/... เท่านั้น และไม่ใช้ backslash \

## 🧹 4. ล้าง container / volume เก่า (ถ้าติดตั้งครั้งแรกหรือเปลี่ยนค่า)
```powershell
docker-compose down -v
Remove-Item -Recurse -Force "D:\wordpress-docker\www\db_data"
```

## ▶️ 5. สร้างและรัน container
```powershell
docker-compose up -d
```
<dev>
    <ul>
        <li>รอ 10–20 วินาทีให้ MySQL พร้อม</li>
        <li>ตรวจสอบ container:</li>
    </ul>
</dev>

```powershell
docker ps
```
<dev>
    <ul>
        <li>ตรวจสอบ log ของ WordPress/MySQL:</li>
    </ul>
</dev>

```powershell
docker-compose logs -f wordpress
docker-compose logs -f db
```

## 🌐 6. ตั้งค่า WordPress ครั้งแรก
<dev>
    <p>1. เปิด browser → <a>http://localhost:8080</a></p>
    <p>2. กรอกข้อมูล:</p>
    <ul>
        <li>Site Title: ชื่อเว็บไซต์</li>
        <li>Username: ชื่อผู้ใช้ admin</li>
        <li>Password: รหัสผ่าน strong</li>
        <li>Email: อีเมลสำหรับ admin</li>
        <li>Search Engine Visibility: เลือกตามต้องการ</li>
    </ul>
    <p>3. กด Install WordPress</p>
    <p>4. เมื่อเสร็จ กด Log In → เข้า Dashboard</p>
</dev>

## ⚡ 7. คำสั่งที่ควรรู้
| คำสั่ง                                                       | ใช้ทำอะไร                |
| ------------------------------------------------------------ | ------------------------ |
| `docker-compose down`                                        | หยุด container           |
| `docker-compose restart wordpress`                           | รีสตาร์ท WordPress       |
| `docker exec -it wordpress_app powershell`                   | เข้า WordPress container |
| `docker exec -it wordpress_db mysql -uwordpress -pwordpress` | เข้า MySQL container     |

## ✅ 8. การตั้งค่าและใช้งานต่อ
<dev>
    <ul>
        <li>ตั้งค่า Permalinks → Settings → Permalinks</li>
        <li>ติดตั้ง Themes / Plugins → Appearance / Plugins</li>
        <li>Password: รหัสผ่าน strong</li>
        <li>สร้าง Pages / Posts เพื่อเริ่มใช้งาน</li>
        <li>สำรองข้อมูล WordPress และ Database เป็นประจำ</li>
    </ul>
</dev>

## 

# คู่มือเชื่อมต่อฐานข้อมูล WordPress + MySQL + PHPMyAdmin (Docker)

## 1️⃣ ข้อมูลสำคัญจาก docker-compose.yml
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
docker-compose up -d          # สร้างและรัน container
docker-compose down -v        # หยุดและลบ container + volume
docker ps                     # ดู container ที่รันอยู่
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