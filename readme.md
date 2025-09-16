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
      - ./db_data:/var/lib/mysql   
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
      - ./wordpress_data:/var/www/html  
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
Docker Compose จะสร้าง `db_data` และ `wordpress_data` ให้เองอัตโนมัติจากที่ประกาศไว้ใน `volumes:`

```
<dev>
    <ul>
        <li>รอ 10–20 วินาทีให้ MySQL พร้อม</li>
        <li>ตรวจสอบ container:</li>
    </ul>
</dev>

```powershell
docker ps

ถ้าอยากสร้างเฉพาะ volume เองโดยตรง (ไม่ต้องรัน service) ก็ทำได้ เช่น:

```bash
docker volume create db_data
docker volume create wordpress_data

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
ถ้าหาก Clone ไปใช้งาน ใช้วิธีนี้:
<dev>
    <p>1. ลง container เดิมก่อน:</p>
</dev>
```bash
docker-compose down
```
<dev>
    <p>2. รันใหม่:</p>
</dev>
```bash
docker-compose up -d
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