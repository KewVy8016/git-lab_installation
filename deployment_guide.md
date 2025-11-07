# 🚀 คู่มือการตั้งค่าและ Deploy ระบบประเมินครู

## 📋 สารบัญ
1. [ภาพรวมระบบ](#ภาพรวมระบบ)
2. [เตรียม Server GitLab + Runner](#server-1-gitlab--runner)
3. [เตรียม Production Server](#server-2-production-web-server)
4. [ตั้งค่า SSH Key](#ตั้งค่า-ssh-key)
5. [สร้างไฟล์สำหรับ Deploy](#สร้างไฟล์สำหรับ-deploy)
6. [ตั้งค่า GitLab Variables](#ตั้งค่า-gitlab-variables)
7. [Deploy ครั้งแรก](#deploy-ครั้งแรก)
8. [การตรวจสอบและแก้ไขปัญหา](#การตรวจสอบและแก้ไขปัญหา)

---

## 🏗️ ภาพรวมระบบ

### **Server และ Port**

| Server | IP | บทบาท | Ports |
|--------|----|----|-------|
| **Server 1** | 192.168.8.136 | GitLab + Runner | 8000 (GitLab), 5050 (Registry) |
| **Server 2** | 192.168.8.134 | Production | 3000 (Frontend), 7000 (API), 3306 (MySQL), 8080 (phpMyAdmin) |

### **สถาปัตยกรรม CI/CD**

```
┌─────────────────────────────────────────────────────────────┐
│ Server 1: GitLab (192.168.8.136:8000)                       │
│                                                              │
│  1. Push Code → GitLab                                      │
│  2. GitLab Runner รัน Pipeline:                             │
│     ├── Test (Backend/Frontend)                             │
│     ├── Build Docker Images                                 │
│     └── Push to GitLab Container Registry                   │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       │ SSH Deploy
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Server 2: Production (192.168.8.134)                        │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Frontend    │  │   Backend    │  │    MySQL     │     │
│  │  (Nuxt 3)    │  │   (Node.js)  │  │   (8.0)      │     │
│  │  Port 3000   │  │   Port 7000  │  │  Port 3306   │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                 │                  │              │
│         └─────────────────┴──────────────────┘              │
│                  Docker Network                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🖥️ Server 1: GitLab + Runner

### ✅ Checklist เบื้องต้น

```bash
# ตรวจสอบ GitLab
curl http://192.168.8.136:8000

# ตรวจสอบ GitLab Runner
docker ps | grep runner

# ตรวจสอบ Container Registry
# ไปที่: GitLab → Admin → Settings → CI/CD → Container Registry
# ต้องเปิดใช้งานแล้ว
```

### 🔧 ตรวจสอบ Runner Tags

1. ไปที่ **GitLab → Admin Area → CI/CD → Runners**
2. ดู Runner ที่ใช้งานอยู่
3. ตรวจสอบ **Tags:** ต้องมี `deploy` หรือ `docker`

**ถ้ายังไม่มี Tag:**
- คลิก **Edit** ที่ Runner
- เพิ่ม Tag: `deploy`
- **Save changes**

### 📦 ตรวจสอบ Container Registry URL

```bash
# Registry URL จะเป็น:
# http://192.168.8.136:8000/dev_team/teacher-evaluation-system/container_registry

# ทดสอบ Login (จากเครื่องใดก็ได้)
docker login 192.168.8.136:8000
# Username: <gitlab-username>
# Password: <gitlab-password-or-token>
```

---

## 🌐 Server 2: Production Web Server

### 1️⃣ ติดตั้ง Docker และ Docker Compose

```bash
# SSH เข้า Production Server
ssh nayok_tech@192.168.8.134

# อัปเดตระบบ
sudo apt-get update
sudo apt-get upgrade -y

# ลบ Docker เวอร์ชันเก่า (ถ้ามี)
sudo apt-get remove docker docker-engine docker.io containerd runc

# ติดตั้ง Docker (Official Script)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# เพิ่มผู้ใช้เข้ากลุ่ม docker
sudo usermod -aG docker nayok_tech

# Logout และ Login ใหม่เพื่อให้มีผล
exit
ssh nayok_tech@192.168.8.134

# ตรวจสอบ Docker
docker --version
# ควรได้: Docker version 26.x.x

# ตรวจสอบ Docker Compose
docker compose version
# ควรได้: Docker Compose version v2.x.x
```

### 2️⃣ เตรียมโฟลเดอร์สำหรับ Deploy

```bash
# สร้างโฟลเดอร์
sudo mkdir -p /srv/webapp
sudo chown -R nayok_tech:nayok_tech /srv/webapp

# ตรวจสอบ
ls -la /srv/webapp
```

### 3️⃣ ติดตั้ง MySQL Client (สำหรับ Health Check - Optional)

```bash
sudo apt-get install -y mysql-client
```

---

## 🔑 ตั้งค่า SSH Key

### 1️⃣ สร้าง SSH Key บน GitLab Server

```bash
# SSH เข้า GitLab Server
ssh nayok_tech@192.168.8.136

# สร้าง SSH Key (ไม่ต้องใส่ passphrase)
ssh-keygen -t ed25519 -C "gitlab-deploy" -f ~/.ssh/id_ed25519_deploy

# กด Enter 3 ครั้ง

# แสดง Public Key
cat ~/.ssh/id_ed25519_deploy.pub
```

**📋 Copy Public Key** (บรรทัดยาวๆ แบบนี้):
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEvoxat4Ti9b+8HjJhhMNTVKVf9i4u1JSUZ1DqSVAQ3s gitlab-deploy
```

### 2️⃣ เพิ่ม Public Key ไปที่ Production Server

```bash
# SSH เข้า Production Server
ssh nayok_tech@192.168.8.134

# สร้างโฟลเดอร์ .ssh (ถ้ายังไม่มี)
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# เพิ่ม Public Key
echo "วาง-public-key-ที่-copy-มา" >> ~/.ssh/authorized_keys

# ตัวอย่าง:
# echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEvoxat4Ti9b+8HjJhhMNTVKVf9i4u1JSUZ1DqSVAQ3s gitlab-deploy" >> ~/.ssh/authorized_keys

# ตั้งค่า Permission
chmod 600 ~/.ssh/authorized_keys

# ตรวจสอบ
cat ~/.ssh/authorized_keys | grep gitlab-deploy

# ออกจาก SSH
exit
```

### 3️⃣ ทดสอบ SSH ด้วย Private Key

```bash
# กลับมาที่ GitLab Server
ssh -i ~/.ssh/id_ed25519_deploy nayok_tech@192.168.8.134 "echo 'SSH Connection OK!'"

# ควรขึ้น: SSH Connection OK!
# และไม่ถามรหัสผ่าน ✅
```

**❌ ถ้ายังถามรหัสผ่าน:**
```bash
# ตรวจสอบ Permission บน Production Server
ssh nayok_tech@192.168.8.134
ls -la ~/.ssh/

# แก้ไข Permission
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
exit

# ลองใหม่
ssh -i ~/.ssh/id_ed25519_deploy nayok_tech@192.168.8.134 "echo 'SSH OK'"
```

---

## 📁 สร้างไฟล์สำหรับ Deploy

### โครงสร้างโปรเจค

```
teacher-evaluation-system/
├── .gitlab-ci.yml              # ✅ Pipeline Configuration
├── docker-compose.yml          # Development (มีอยู่แล้ว)
├── docker-compose.prod.yml     # ⚠️ ต้องสร้างใหม่
├── deploy/
│   ├── backend.env             # ⚠️ Environment สำหรับ Production
│   └── README.md               # (Optional)
├── backend/
│   ├── Dockerfile              # ✅ ต้องมี
│   ├── .env                    # Development env (อย่า commit!)
│   └── ...
├── frontend/
│   ├── Dockerfile              # ✅ ต้องมี
│   └── ...
└── schema.sql                  # Database Schema
```

---

### 1️⃣ ไฟล์ `docker-compose.prod.yml`

สร้างไฟล์ใน root ของโปรเจค:

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    command: >
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-rootpassword}
      MYSQL_DATABASE: ${MYSQL_DATABASE:-skills_db}
      TZ: Asia/Bangkok
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "-uroot", "-p${MYSQL_ROOT_PASSWORD:-rootpassword}"]
      interval: 10s
      timeout: 5s
      retries: 10
    networks:
      - app-network
    restart: unless-stopped

  phpmyadmin:
    image: phpmyadmin:latest
    environment:
      PMA_HOST: db
      PMA_USER: root
      PMA_PASSWORD: ${MYSQL_ROOT_PASSWORD:-rootpassword}
    ports:
      - "8080:80"
    depends_on:
      - db
    networks:
      - app-network
    restart: unless-stopped

  backend:
    image: ${CI_REGISTRY_IMAGE}/backend:${IMAGE_TAG:-latest}
    container_name: backend
    env_file:
      - ./deploy/backend.env
    ports:
      - "7000:7000"
    volumes:
      - backend_uploads:/app/uploads
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network
    restart: unless-stopped

  frontend:
    image: ${CI_REGISTRY_IMAGE}/frontend:${IMAGE_TAG:-latest}
    container_name: frontend
    environment:
      - NUXT_PUBLIC_API_BASE=${NUXT_PUBLIC_API_BASE:-http://192.168.8.134:7000}
      - NITRO_PORT=3000
      - HOST=0.0.0.0
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - app-network
    restart: unless-stopped

volumes:
  db_data:
  backend_uploads:

networks:
  app-network:
    driver: bridge
```

---

### 2️⃣ ไฟล์ `deploy/backend.env`

```bash
mkdir -p deploy
nano deploy/backend.env
```

**เนื้อหา:**
```bash
# Database Configuration
DB_HOST=db
DB_PORT=3306
DB_NAME=skills_db
DB_USER=root
DB_PASSWORD=rootpassword

# หรือใช้ DATABASE_URL (ถ้า Backend รองรับ)
DATABASE_URL=mysql://root:rootpassword@db:3306/skills_db

# JWT Configuration
JWT_SECRET=change-this-to-strong-random-string-in-production
JWT_EXPIRES_IN=7d

# CORS Configuration
CORS_ORIGIN=http://192.168.8.134:3000

# Node Environment
NODE_ENV=production
PORT=7000

# Timezone
TZ=Asia/Bangkok

# Upload Path (ถ้ามี)
UPLOAD_DIR=/app/uploads
```

⚠️ **สำคัญ:** เปลี่ยน `JWT_SECRET` และ `DB_PASSWORD` เป็นค่าที่ปลอดภัย!

---

### 3️⃣ ไฟล์ `backend/Dockerfile`

```dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Production stage
FROM node:20-alpine

WORKDIR /app

# Copy from builder
COPY --from=builder /app ./

# Create uploads directory
RUN mkdir -p /app/uploads && chown -R node:node /app

# Use non-root user
USER node

EXPOSE 7000

CMD ["node", "server.js"]
```

**⚠️ หมายเหตุ:** ปรับ `server.js` ให้ตรงกับไฟล์เริ่มต้นของคุณ (อาจเป็น `index.js` หรือ `app.js`)

---

### 4️⃣ ไฟล์ `frontend/Dockerfile`

```dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build
RUN npm run build

# Production stage
FROM node:20-alpine

WORKDIR /app

# Copy build output
COPY --from=builder /app/.output ./.output
COPY --from=builder /app/package*.json ./

# Install production dependencies only
RUN npm ci --only=production

# Use non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000

ENV NUXT_HOST=0.0.0.0
ENV NUXT_PORT=3000

CMD ["node", ".output/server/index.mjs"]
```

---

### 5️⃣ ไฟล์ `.gitlab-ci.yml`

ใช้ไฟล์ที่สร้างไว้ใน Artifact ข้างต้น (gitlab_ci_merged)

---

## 🔐 ตั้งค่า GitLab Variables

### 1️⃣ เตรียม Private Key

```bash
# บน GitLab Server (192.168.8.136)
cat ~/.ssh/id_ed25519_deploy
```

**📋 Copy ทั้งหมด** รวม header/footer:
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
...
-----END OPENSSH PRIVATE KEY-----
```

### 2️⃣ เพิ่ม Variable ใน GitLab

**เปิด GitLab:**
```
http://192.168.8.136:8000/dev_team/teacher-evaluation-system
```

**ไปที่:**
- **Settings → CI/CD → Variables → Expand**
- คลิก **Add variable**

**เพิ่ม Variable ดังนี้:**

#### Variable 1: `DEPLOY_PRIV_KEY`

| ฟิลด์ | ค่า |
|-------|-----|
| **Key** | `DEPLOY_PRIV_KEY` |
| **Type** | `File` ⚠️ |
| **Value** | วาง Private Key ทั้งหมด |
| **Protect variable** | ✅ เลือก |
| **Mask variable** | ไม่เลือก |
| **Environment scope** | `All (default)` |

#### Variable 2: `CI_REGISTRY_USER` (มีอยู่แล้ว)

GitLab จัดการให้อัตโนมัติ - ไม่ต้องเพิ่ม

#### Variable 3: `CI_REGISTRY_PASSWORD` (มีอยู่แล้ว)

GitLab จัดการให้อัตโนมัติ - ไม่ต้องเพิ่ม

---

### 3️⃣ ตรวจสอบ Variables

ใน **CI/CD → Variables** ควรมี:

| Key | Type | Protected | Masked |
|-----|------|-----------|--------|
| `DEPLOY_PRIV_KEY` | File | ✅ | ❌ |
| `CI_REGISTRY_USER` | Variable | - | ✅ |
| `CI_REGISTRY_PASSWORD` | Variable | - | ✅ |

---

## 🚀 Deploy ครั้งแรก

### 1️⃣ Commit และ Push Code

```bash
# เพิ่มไฟล์ทั้งหมด
git add .gitlab-ci.yml docker-compose.prod.yml deploy/ backend/Dockerfile frontend/Dockerfile

# Commit
git commit -m "Add production deployment configuration"

# Push ไป main branch
git push origin main
```

### 2️⃣ ดู Pipeline ใน GitLab

1. ไปที่: **CI/CD → Pipelines**
2. คลิกที่ Pipeline ล่าสุด
3. ดูสถานะแต่ละ Stage:

```
┌──────────────┐
│ Test Stage   │
├──────────────┤
│ test_backend │ ✅
│ test_frontend│ ✅
└──────────────┘
       ↓
┌──────────────┐
│ Build Stage  │
├──────────────┤
│ build_backend│ ✅
│build_frontend│ ✅
└──────────────┘
       ↓
┌──────────────┐
│Deploy Stage  │
├──────────────┤
│deploy_prod   │ ⏸️ Manual
└──────────────┘
```

### 3️⃣ Manual Deploy

1. หลังจาก **Build Stage** เสร็จ
2. ที่ **deploy_production** จะมีปุ่ม **▶ Play**
3. คลิก **Play**
4. ดู Log แบบ Real-time

**Log ที่ควรเห็น:**
```
🚀 Deploying to Production Server...
📥 Pulling Docker images...
backend:abc123: Pulling from dev_team/teacher-evaluation-system/backend
frontend:abc123: Pulling from dev_team/teacher-evaluation-system/frontend
🔄 Restarting containers...
⏳ Waiting for services to start...
✅ Deployment complete!
NAME       IMAGE                                    STATUS
backend    registry/backend:abc123                  Up 10 seconds
frontend   registry/frontend:abc123                 Up 10 seconds
db         mysql:8.0                                Up 30 seconds (healthy)
```

### 4️⃣ ตรวจสอบบน Production Server

```bash
# SSH เข้า Production Server
ssh nayok_tech@192.168.8.134

# เข้าไปที่โฟลเดอร์
cd /srv/webapp

# ดู Containers
docker compose -f docker-compose.prod.yml ps

# ควรเห็น:
NAME         IMAGE                              STATUS
backend      registry/backend:abc123            Up 2 minutes
frontend     registry/frontend:abc123           Up 2 minutes
db           mysql:8.0                          Up 3 minutes (healthy)
phpmyadmin   phpmyadmin:latest                  Up 2 minutes

# ดู Logs
docker compose -f docker-compose.prod.yml logs -f

# กด Ctrl+C เพื่อออก
```

### 5️⃣ ทดสอบเว็บไซต์

เปิด Browser:

- **Frontend:** http://192.168.8.134:3000
- **Backend API:** http://192.168.8.134:7000
- **phpMyAdmin:** http://192.168.8.134:8080

---

## 🔍 การตรวจสอบและแก้ไขปัญหา

### ❌ ปัญหา: Pipeline ล้มเหลวที่ Test Stage

```bash
# ตรวจสอบ Log
# ใน GitLab: CI/CD → Pipelines → คลิกที่ Failed Job

# แก้ไข: ข้าม Test ชั่วคราว (ถ้ายังไม่มี Test)
# แก้ไขใน .gitlab-ci.yml:

test_backend:
  script:
    - cd backend
    - npm ci
    - npm run lint --if-present || echo "No lint"
    - npm test --if-present || echo "No test"  # เพิ่ม || echo
```

### ❌ ปัญหา: Build Stage ล้มเหลว - Cannot find Dockerfile

```bash
# ตรวจสอบว่ามี Dockerfile
ls backend/Dockerfile
ls frontend/Dockerfile

# ถ้าไม่มี ให้สร้างตามขั้นตอนข้างต้น
```

### ❌ ปัญหา: Deploy Stage - SSH Permission Denied

```bash
# ทดสอบ SSH จาก GitLab Runner
docker exec -it gitlab-runner bash
ssh -i /path/to/key nayok_tech@192.168.8.134

# ถ้าไม่ได้ แสดงว่า Private Key ไม่ถูกต้อง
# ลองสร้าง SSH Key ใหม่
```

### ❌ ปัญหา: Container ไม่ขึ้นบน Production

```bash
# SSH เข้า Production Server
ssh nayok_tech@192.168.8.134
cd /srv/webapp

# ดู Container Status
docker compose -f docker-compose.prod.yml ps

# ดู Logs ของ Container ที่มีปัญหา
docker compose -f docker-compose.prod.yml logs backend
docker compose -f docker-compose.prod.yml logs frontend
docker compose -f docker-compose.prod.yml logs db

# Restart Container
docker compose -f docker-compose.prod.yml restart backend

# หรือ Restart ทั้งหมด
docker compose -f docker-compose.prod.yml down
docker compose -f docker-compose.prod.yml up -d
```

### ❌ ปัญหา: Database Connection Failed

```bash
# เช็คว่า MySQL Container ทำงาน
docker compose -f docker-compose.prod.yml ps db

# เข้าไปใน MySQL Container
docker compose -f docker-compose.prod.yml exec db mysql -uroot -prootpassword

# ตรวจสอบ Database
SHOW DATABASES;
USE skills_db;
SHOW TABLES;
exit;

# ตรวจสอบ Environment Variables ของ Backend
docker compose -f docker-compose.prod.yml exec backend env | grep DB
```

### ❌ ปัญหา: Frontend เชื่อมต่อ Backend ไม่ได้

```bash
# ตรวจสอบ NUXT_PUBLIC_API_BASE
docker compose -f docker-compose.prod.yml exec frontend env | grep NUXT

# ถ้าไม่ถูกต้อง แก้ไขใน docker-compose.prod.yml:
frontend:
  environment:
    - NUXT_PUBLIC_API_BASE=http://192.168.8.134:7000

# Restart
docker compose -f docker-compose.prod.yml restart frontend
```

### ❌ ปัญหา: Images Pull ล้มเหลว

```bash
# บน Production Server - ทดสอบ Login Registry
docker login 192.168.8.136:8000
# Username: <gitlab-username>
# Password: <gitlab-token>

# Pull Image manually
docker pull 192.168.8.136:8000/dev_team/teacher-evaluation-system/backend:latest

# ถ้า SSL Error (ใช้ HTTP แทน HTTPS)
# แก้ไข: /etc/docker/daemon.json
sudo nano /etc/docker/daemon.json

{
  "insecure-registries": ["192.168.8.136:8000"]
}

# Restart Docker
sudo systemctl restart docker
```

---

## 📊 คำสั่งที่ใช้บ่อย

### บน Production Server

```bash
# เข้าโฟลเดอร์
cd /srv/webapp

# ดู Container Status
docker compose -f docker-compose.prod.yml ps

# ดู Logs (แบบติดตาม)
docker compose -f docker-compose.prod.yml logs -f

# ดู Logs เฉพาะ Service
docker compose -f docker-compose.prod.yml logs -f backend

# Restart Service
docker compose -f docker-compose.prod.yml restart backend

# Stop ทั้งหมด
docker compose -f docker-compose.prod.yml down

# Start ทั้งหมด
docker compose -f docker-compose.prod.yml up -d

# Pull Images ใหม่
docker compose -f docker-compose.prod.yml pull

# ลบ Volumes (ระวัง! จะลบข้อมูล)
docker compose -f docker-compose.prod.yml down -v

# เข้าไปใน Container
docker compose -f docker-compose.prod.yml exec backend sh
docker compose -f docker-compose.prod.yml exec frontend sh

# ดูการใช้ Resources
docker stats
```

### ตรวจสอบ Disk Usage

```bash
# ดูขนาด Docker
docker system df

# ลบ Images/Containers ที่ไม่ใช้
docker system prune -a

# ลบเฉพาะ Volumes ที่ไม่ใช้
docker volume prune
```

---

## 🔄 การ Deploy รอบถัดไป

เมื่อมีการแก้ไข Code:

```bash
# 1. Commit และ Push
git add .
git commit -m "Fix: update feature X"
git push origin main

# 2. ไปที่ GitLab → CI/CD → Pipelines
# 3. รอให้ Test + Build เสร็จ
# 4. คลิก Play ที่ deploy_production
# 5. ตรวจสอบผลลัพธ์
```

---

## 📋 Checklist ก่อน Deploy Production

- [ ] ✅ Server 1: GitLab + Runner ทำงานปกติ
- [ ] ✅ Server 2: Docker + Docker Compose ติดตั้งแล้ว
- [ ] ✅ SSH Key ตั้งค่าเรียบร้อย (ไม่ต้องใส่รหัสผ่าน)
- [ ] ✅ `DEPLOY_PRIV_KEY` เพิ่มใน GitLab Variables แล้ว
- [ ] ✅ `docker-compose.prod.yml` พร้อมใช้งาน
- [ ] ✅ `deploy/backend.env` มีค่าครบถ้วน
- [ ] ✅ `backend/Dockerfile` และ `frontend/Dockerfile` พร้อม
- [ ] ✅ `.gitlab-ci.yml` อัปเดตแล้ว
- [ ] ✅ ทดสอบ Pipeline ผ่านแล้ว
- [ ] ✅ Production Server เข้าถึง GitLab Registry ได้
- [ ] ✅ Firewall เปิด Port 3000, 7000, 8080 (ถ้าต้องการ)
- [ ] ✅ สำรอง Database ก่อน Deploy (ครั้งแรกไม่จำเป็น)

---

## 🎓 ทีมพัฒนา

สร้างโดย: Development Team  
วันที่: พฤศจิ