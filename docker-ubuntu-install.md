# คู่มือติดตั้ง Docker บน Ubuntu

## ข้อกำหนดเบื้องต้น
- Ubuntu 20.04 LTS ขึ้นไป
- สิทธิ์ sudo

## ขั้นตอนการติดตั้ง

### 1. อัปเดตแพ็คเกจ
```bash
sudo apt update
sudo apt upgrade -y
```

### 2. ติดตั้งแพ็คเกจที่จำเป็น
```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

### 3. เพิ่ม Docker GPG key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### 4. เพิ่ม Docker repository
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 5. ติดตั้ง Docker Engine
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 6. เริ่มและเปิดใช้งาน Docker
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### 7. ตรวจสอบการติดตั้ง
```bash
sudo docker --version
sudo docker run hello-world
```

## Dockerfile คืออะไร

Dockerfile คือไฟล์ที่ใช้กำหนดขั้นตอนการสร้าง Docker Image โดยเขียนเป็นคำสั่งที่บอกว่าต้องการให้ติดตั้งอะไร คัดลอกไฟล์อะไร และรันคำสั่งอะไรบ้างใน Container

### หลักการทำงานของ Dockerfile

1. **เริ่มจาก Base Image** - เลือก Image พื้นฐาน เช่น Node.js, Python, Ubuntu
2. **ตั้งค่า Environment** - กำหนดตัวแปร, Working Directory
3. **ติดตั้ง Dependencies** - คัดลอกไฟล์และติดตั้งแพ็คเกจที่ต้องการ
4. **คัดลอก Source Code** - นำโค้ดเข้าไปใน Image
5. **กำหนดคำสั่งเริ่มต้น** - บอกว่าเมื่อ Container รัน ให้ทำอะไร

### คำสั่งสำคัญใน Dockerfile

#### 1. FROM - เลือก Base Image
```dockerfile
# เลือก Image พื้นฐาน (ต้องมีเสมอเป็นบรรทัดแรก)
FROM node:18-alpine
# alpine = เวอร์ชันที่เบาที่สุด, slim = เบา, latest = ใหม่ที่สุด
```

#### 2. WORKDIR - กำหนด Working Directory
```dockerfile
# กำหนดโฟลเดอร์ทำงาน (คล้าย cd /app)
WORKDIR /app
# คำสั่งถัดไปทั้งหมดจะทำงานในโฟลเดอร์นี้
```

#### 3. COPY - คัดลอกไฟล์
```dockerfile
# COPY <จากเครื่อง> <ไปที่ Container>
COPY package*.json ./          # คัดลอกไฟล์เฉพาะ
COPY . .                        # คัดลอกทุกอย่างในโฟลเดอร์ปัจจุบัน
```

#### 4. RUN - รันคำสั่งขณะ Build
```dockerfile
# รันคำสั่งติดตั้งแพ็คเกจ (ทำครั้งเดียวตอน Build)
RUN npm install
RUN apt-get update && apt-get install -y curl

# รวมหลายคำสั่งเพื่อลด Layer
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean
```

#### 5. ENV - กำหนดตัวแปร Environment
```dockerfile
# กำหนดค่าตัวแปรที่จะใช้ใน Container
ENV NODE_ENV=production
ENV PORT=3000
ENV DB_HOST=database
```

#### 6. EXPOSE - ระบุ Port
```dockerfile
# บอกว่า Container ใช้ Port อะไร (เป็นแค่ Documentation ไม่ได้เปิด Port จริง)
EXPOSE 3000
# ต้องใช้ -p เวลารัน: docker run -p 3000:3000
```

#### 7. CMD - คำสั่งเริ่มต้น
```dockerfile
# คำสั่งที่จะรันเมื่อ Container เริ่มทำงาน (มีได้ 1 CMD)
CMD ["node", "server.js"]
CMD ["npm", "start"]

# รูปแบบ: CMD ["executable", "param1", "param2"]
```

#### 8. ENTRYPOINT - คำสั่งหลักที่ไม่สามารถ Override ได้ง่าย
```dockerfile
# คล้าย CMD แต่ถูกล็อคไว้
ENTRYPOINT ["node"]
CMD ["server.js"]
# รัน: node server.js (สามารถเปลี่ยน server.js ได้ แต่ node เปลี่ยนไม่ได้)
```

#### 9. ARG - ตัวแปรสำหรับตอน Build
```dockerfile
# ใช้ในขณะ Build เท่านั้น (ไม่มีใน Runtime)
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}

# ส่งค่าตอน Build: docker build --build-arg NODE_VERSION=20 .
```

#### 10. ADD - คล้าย COPY แต่มีความสามารถเพิ่ม
```dockerfile
# รองรับ URL และแตกไฟล์ .tar อัตโนมัติ
ADD https://example.com/file.tar.gz /app/
ADD archive.tar.gz /app/

# แนะนำ: ใช้ COPY ในกรณีทั่วไป ใช้ ADD เฉพาะตอนต้องการ Feature พิเศษ
```

---

## ตัวอย่าง Dockerfile สำหรับ Express.js Backend

```dockerfile
# 1. เลือก Base Image (Node.js เวอร์ชัน 18 แบบ Alpine ที่มีขนาดเล็ก)
FROM node:18-alpine

# 2. กำหนด Working Directory ที่จะทำงาน
WORKDIR /app

# 3. คัดลอก package.json และ package-lock.json ก่อน
# (ทำให้ Docker ใช้ Cache ได้ ถ้าไฟล์ไม่เปลี่ยน ไม่ต้อง npm install ใหม่)
COPY package*.json ./

# 4. ติดตั้ง Dependencies
RUN npm install --production

# 5. คัดลอก Source Code ทั้งหมด
COPY . .

# 6. เปิด Port ที่ Express ใช้งาน (เช่น 3000)
EXPOSE 3000

# 7. กำหนดคำสั่งที่จะรันเมื่อ Container เริ่มทำงาน
CMD ["node", "server.js"]
```

**อธิบายเพิ่มเติม:**
- ทำไมคัดลอก `package*.json` ก่อน? → เพื่อใช้ประโยชน์จาก Docker Layer Caching ถ้า package.json ไม่เปลี่ยน Docker จะใช้ cache ของ `npm install` ทำให้ Build เร็วขึ้น
- `--production` → ติดตั้งเฉพาะ dependencies ไม่รวม devDependencies

---

## ตัวอย่าง Dockerfile สำหรับ Vue.js Frontend

```dockerfile
# ======== Stage 1: Build ========
# ใช้ Node.js เพื่อ Build Production Files
FROM node:18-alpine AS builder

WORKDIR /app

# คัดลอก package files และติดตั้ง
COPY package*.json ./
RUN npm install

# คัดลอก Source Code และ Build
COPY . .
RUN npm run build
# ได้ไฟล์ Static ใน /app/dist

# ======== Stage 2: Production ========
# ใช้ Nginx เพื่อ Serve Static Files
FROM nginx:alpine

# คัดลอกไฟล์ที่ Build เสร็จจาก Stage แรก
COPY --from=builder /app/dist /usr/share/nginx/html

# คัดลอก Config ของ Nginx (ถ้ามี)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Nginx ใช้ Port 80
EXPOSE 80

# Nginx จะรันอัตโนมัติ (มี CMD ใน Base Image แล้ว)
```

**อธิบายเพิ่มเติม:**
- **Multi-stage Build** → แบ่งเป็น 2 ขั้นตอน: Build กับ Production เพื่อลดขนาด Image
- Stage 1 (builder) → ใช้ Node.js Build Vue → ได้ Static Files
- Stage 2 (production) → เอาแค่ Static Files มา Serve ด้วย Nginx (ไม่ต้องเอา Node.js มาด้วย)
- ผลลัพธ์: Image ขนาดเล็กมาก (จาก ~1GB เหลือ ~30MB)

**ไฟล์ nginx.conf ตัวอย่าง:**
```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## Docker Compose คืออะไร

Docker Compose คือเครื่องมือสำหรับจัดการหลาย Container พร้อมกัน โดยใช้ไฟล์ `docker-compose.yml` กำหนดค่า เหมาะสำหรับ Application ที่มีหลาย Service เช่น Frontend + Backend + Database

### ประโยชน์ของ Docker Compose
- **จัดการหลาย Container พร้อมกัน** - เริ่ม/หยุด ทุก Service ด้วยคำสั่งเดียว
- **กำหนดความสัมพันธ์** - บอกว่า Service ไหนต้องรอ Service ไหน
- **แชร์ Network อัตโนมัติ** - ทุก Service สื่อสารกันได้ผ่านชื่อ Service
- **จัดการ Volume ง่าย** - เก็บข้อมูล Database ไว้ถาวร

### โครงสร้างพื้นฐานของ docker-compose.yml

```yaml
version: '3.8'              # เวอร์ชันของ Docker Compose

services:                   # รายการ Service (Container) ทั้งหมด
  service1:                 # ชื่อ Service ที่ 1
    # กำหนดค่าต่างๆ
  service2:                 # ชื่อ Service ที่ 2
    # กำหนดค่าต่างๆ

volumes:                    # กำหนด Volume สำหรับเก็บข้อมูล
  volume1:

networks:                   # กำหนด Network (Optional)
  network1:
```

### คำสั่งสำคัญใน Docker Compose

#### 1. version - เวอร์ชัน
```yaml
version: '3.8'
# เวอร์ชันของ Compose File Format (ใช้ 3.8 หรือ 3.9 ได้ทั่วไป)
```

#### 2. services - กำหนด Container
```yaml
services:
  backend:
    # การตั้งค่า Container
```

#### 3. image - ใช้ Image สำเร็จรูป
```yaml
services:
  database:
    image: mysql:8.0        # ดาวน์โหลดจาก Docker Hub
```

#### 4. build - Build จาก Dockerfile
```yaml
services:
  backend:
    build: ./backend        # Build จาก Dockerfile ใน ./backend
    # หรือ
    build:
      context: ./backend    # โฟลเดอร์ที่มี Dockerfile
      dockerfile: Dockerfile.prod  # ระบุชื่อไฟล์ (ถ้าไม่ใช่ Dockerfile)
```

#### 5. ports - แมพ Port
```yaml
services:
  backend:
    ports:
      - "3000:3000"         # host:container
      - "4000:4000"         # เปิดได้หลาย Port
```

#### 6. environment - กำหนดตัวแปร
```yaml
services:
  backend:
    environment:
      NODE_ENV: production
      DB_HOST: database     # ใช้ชื่อ Service เป็น Host
      DB_PORT: 3306
      DB_USER: root
      DB_PASSWORD: password
```

#### 7. volumes - แมพข้อมูล
```yaml
services:
  database:
    volumes:
      - db-data:/var/lib/mysql        # Named Volume (ข้อมูลถาวร)
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # Bind Mount (ไฟล์จริง)

volumes:
  db-data:                            # ประกาศ Named Volume
```

#### 8. depends_on - กำหนดลำดับ
```yaml
services:
  backend:
    depends_on:
      - database            # รอให้ Database เริ่มก่อน (แต่ไม่รอให้พร้อมใช้งาน)
```

#### 9. restart - นโยบายการ Restart
```yaml
services:
  backend:
    restart: always         # Restart เสมอถ้าหยุด
    # always, unless-stopped, on-failure, no
```

#### 10. networks - กำหนด Network
```yaml
services:
  backend:
    networks:
      - mynetwork

networks:
  mynetwork:
    driver: bridge
```

#### 11. container_name - กำหนดชื่อ Container
```yaml
services:
  backend:
    container_name: my-backend-app
```

#### 12. command - Override คำสั่ง CMD
```yaml
services:
  backend:
    command: npm run dev    # ใช้แทน CMD ใน Dockerfile
```

---

## ตัวอย่าง docker-compose.yml สำหรับ Vue + Express + MySQL

```yaml
version: '3.8'

services:
  # Frontend - Vue.js
  frontend:
    # Build จาก Dockerfile ในโฟลเดอร์ frontend
    build:
      context: ./frontend
      dockerfile: Dockerfile
    
    # แมพ Port 8080 จากเครื่อง Host ไปที่ Port 80 ใน Container
    ports:
      - "8080:80"
    
    # รอให้ Backend เริ่มก่อน (แต่ไม่รอให้พร้อมใช้งาน)
    depends_on:
      - backend
    
    # กำหนดให้ Restart อัตโนมัติถ้า Container หยุด
    restart: unless-stopped
    
    # เชื่อมต่อกับ Network
    networks:
      - app-network

  # Backend - Express.js
  backend:
    # Build จาก Dockerfile ในโฟลเดอร์ backend
    build:
      context: ./backend
      dockerfile: Dockerfile
    
    # แมพ Port 3000
    ports:
      - "3000:3000"
    
    # กำหนดตัวแปร Environment
    environment:
      # ใช้ชื่อ Service 'database' เป็น Host (Docker จะแปลงเป็น IP อัตโนมัติ)
      DB_HOST: database
      DB_PORT: 3306
      DB_USER: root
      DB_PASSWORD: rootpassword
      DB_NAME: myapp
      NODE_ENV: production
    
    # รอให้ Database เริ่มก่อน
    depends_on:
      - database
    
    restart: unless-stopped
    
    # แมพโฟลเดอร์เพื่อ Development (Optional - ลบออกใน Production)
    volumes:
      - ./backend:/app
      - /app/node_modules   # ป้องกันไม่ให้ node_modules ใน Host ทับ Container
    
    networks:
      - app-network

  # Database - MySQL
  database:
    # ใช้ Image สำเร็จรูป
    image: mysql:8.0
    
    # ไม่ต้องเปิด Port ออกนอก (เปิดให้ Backend ใช้ภายใน Network เท่านั้น)
    # หากต้องการเข้าถึงจากเครื่อง Host: "3306:3306"
    
    # กำหนดค่า MySQL
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: myapp
      MYSQL_USER: user
      MYSQL_PASSWORD: userpassword
    
    # เก็บข้อมูล MySQL ไว้ถาวร (ไม่หายเมื่อลบ Container)
    volumes:
      - mysql-data:/var/lib/mysql
      # คัดลอกไฟล์ SQL เริ่มต้น (Optional)
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    
    restart: unless-stopped
    
    networks:
      - app-network

# ประกาศ Named Volume สำหรับเก็บข้อมูล MySQL
volumes:
  mysql-data:
    # Docker จะสร้างและจัดการ Volume นี้ให้อัตโนมัติ

# ประกาศ Network (Optional - Docker จะสร้างให้อัตโนมัติถ้าไม่ระบุ)
networks:
  app-network:
    driver: bridge
```

### อธิบายโครงสร้าง

**Frontend (Vue.js)**
- Build จาก Dockerfile แบบ Multi-stage (Node.js + Nginx)
- Serve บน Port 8080
- สื่อสารกับ Backend ผ่าน `http://backend:3000` (ใช้ชื่อ Service)

**Backend (Express.js)**
- Build จาก Dockerfile ที่มี Node.js
- รันบน Port 3000
- เชื่อมต่อ MySQL ผ่าน `database:3306` (ใช้ชื่อ Service แทน IP)
- ตัวแปร `DB_HOST: database` ทำให้ Express เชื่อมต่อไปที่ Container MySQL

**Database (MySQL)**
- ใช้ Official MySQL Image จาก Docker Hub
- ไม่เปิด Port ออกนอก (รับแค่ Backend เท่านั้น)
- ข้อมูลเก็บใน Named Volume `mysql-data` (ไม่หายเมื่อลบ Container)

**Network**
- ทุก Service อยู่ใน Network เดียวกัน (`app-network`)
- Service สื่อสารกันผ่านชื่อ (Docker มี DNS ภายใน)

---

## โครงสร้างโปรเจกต์

```
my-app/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   └── ...
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js
│   └── ...
└── database/
    └── init.sql
```

---

## คำสั่งพื้นฐาน Docker Compose

### เริ่มและหยุด Service

```bash
# เริ่มทุก Service (แบบ Foreground - เห็น Log)
docker compose up

# เริ่มแบบ Background (Detached Mode)
docker compose up -d

# Build Image ใหม่และเริ่ม
docker compose up --build

# Build เฉพาะ Service ที่ต้องการ
docker compose up --build backend

# หยุดทุก Service
docker compose down

# หยุดและลบ Volume ด้วย (ข้อมูลหายหมด!)
docker compose down -v

# หยุดเฉพาะ Service ใด Service หนึ่ง
docker compose stop backend

# เริ่ม Service ที่หยุดไว้
docker compose start backend

# Restart Service
docker compose restart backend
```

### ดูสถานะและ Log

```bash
# ดูสถานะของทุก Service
docker compose ps

# ดู Log ของทุก Service
docker compose logs

# ดู Log แบบ Real-time
docker compose logs -f

# ดู Log ของ Service เดียว
docker compose logs -f backend

# ดู Log 100 บรรทัดล่าสุด
docker compose logs --tail=100 backend
```

### เข้าไปใน Container

```bash
# เข้าไปใน Container แบบ Interactive
docker compose exec backend sh
docker compose exec backend bash

# รันคำสั่งใน Container โดยไม่เข้าไป
docker compose exec backend npm install package-name
docker compose exec database mysql -u root -p
```

### Build และ Pull

```bash
# Build ทุก Service
docker compose build

# Build โดยไม่ใช้ Cache
docker compose build --no-cache

# Pull Image ใหม่
docker compose pull
```

### Scale Service

```bash
# เพิ่มจำนวน Container ของ Service (เช่น Backend 3 ตัว)
docker compose up -d --scale backend=3
```

### ดูการใช้ทรัพยากร

```bash
# ดู CPU, Memory ของทุก Container
docker compose stats
```

---

## คำสั่ง Docker พื้นฐาน

### จัดการ Container

```bash
# แสดง Container ที่กำลังรัน
docker ps

# แสดง Container ทั้งหมด (รวมที่หยุด)
docker ps -a

# หยุด Container
docker stop <container_id>

# เริ่ม Container
docker start <container_id>

# Restart Container
docker restart <container_id>

# ลบ Container
docker rm <container_id>

# ลบ Container ที่หยุดแล้วทั้งหมด
docker container prune

# เข้าไปใน Container ที่รันอยู่
docker exec -it <container_id> bash

# ดู Log ของ Container
docker logs <container_id>

# ดู Log แบบ Real-time
docker logs -f <container_id>

# ดูการใช้ทรัพยากร
docker stats
```

### จัดการ Image

```bash
# แสดง Image ทั้งหมด
docker images

# ดาวน์โหลด Image
docker pull mysql:8.0

# Build Image จาก Dockerfile
docker build -t myapp:1.0 .

# ลบ Image
docker rmi <image_id>

# ลบ Image ที่ไม่ได้ใช้งาน
docker image prune
```

### จัดการ Volume

```bash
# แสดง Volume ทั้งหมด
docker volume ls

# สร้าง Volume
docker volume create myvolume

# ดูข้อมูลของ Volume
docker volume inspect myvolume

# ลบ Volume
docker volume rm myvolume

# ลบ Volume ที่ไม่ได้ใช้งาน
docker volume prune
```

### จัดการ Network

```bash
# แสดง Network ทั้งหมด
docker network ls

# สร้าง Network
docker network create mynetwork

# ดูข้อมูลของ Network
docker network inspect mynetwork

# ลบ Network
docker network rm mynetwork
```

### คำสั่งอื่นๆ

```bash
# ดูข้อมูลระบบ Docker
docker info

# ดูเวอร์ชัน
docker --version
docker compose version

# ลบทุกอย่างที่ไม่ได้ใช้งาน
docker system prune -a

# ดูพื้นที่ที่ใช้
docker system df
```

---

## Tips และ Best Practices

### Dockerfile
1. **ใช้ .dockerignore** - ป้องกันไฟล์ไม่จำเป็น เช่น `node_modules`, `.git` เข้าไปใน Image
2. **เรียง COPY ให้ชาญฉลาด** - คัดลอก `package.json` ก่อน → `npm install` → คัดลอก Source Code → ใช้ Cache ได้ดี
3. **ใช้ Multi-stage Build** - ลดขนาด Image (โดยเฉพาะ Frontend)
4. **รวมคำสั่ง RUN** - ใช้ `&&` เชื่อมคำสั่ง → ลดจำนวน Layer
5. **ใช้ Alpine Image** - ขนาดเล็ก เร็ว

### Docker Compose
1. **ใช้ Environment Variables** - แยก Config ออกจาก Code
2. **ใช้ Named Volume** - เก็บข้อมูลถาวร (Database, Uploads)
3. **ตั้งค่า restart: unless-stopped** - Container จะรันอัตโนมัติหลัง Reboot
4. **ใช้ depends_on ระวังให้มาก** - มันรอแค่ Container เริ่ม ไม่รอให้พร้อมใช้งาน (ใช้ health check ร่วมด้วย)

---

## แหล่งข้อมูลเพิ่มเติม
- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)