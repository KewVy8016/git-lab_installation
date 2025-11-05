# 🚀 คู่มือตั้งค่า CI/CD: GitLab → Nginx (Docker)

## 📌 ภาพรวมของระบบ

```
┌─────────────────────────┐
│  👨‍💻 เครื่อง Developer    │
│  (เครื่องของคุณ)         │
└───────────┬─────────────┘
            │ git push
            ↓
┌─────────────────────────┐
│  🏢 VM1 (GitLab)         │
│  IP: <VM1_IP>           │
│  - GitLab Server        │
│  - GitLab Runner        │
└───────────┬─────────────┘
            │ SSH
            ↓
┌─────────────────────────┐
│  🌐 VM2 (Web Server)     │
│  IP: <VM2_IP>           │
│  - Nginx (Docker)       │
└─────────────────────────┘
```

## 📝 ข้อมูลที่ต้องเตรียม

ก่อนเริ่ม กรุณาเตรียมข้อมูลเหล่านี้:

| รายการ | ค่าตัวอย่าง | ค่าจริงของคุณ |
|--------|-------------|---------------|
| IP ของ VM1 (GitLab Server) | `<VM1_IP>` | .................... |
| IP ของ VM2 (Web Server) | `<VM2_IP>` | .................... |
| Username ของ VM1 | `<VM1_USER>` | .................... |
| Username ของ VM2 | `<VM2_USER>` | .................... |
| GitLab Username | `<GITLAB_USERNAME>` | .................... |
| Project Name | `<PROJECT_NAME>` | .................... |

**ตัวอย่างการกรอก:**
- VM1_IP = `192.168.1.10`
- VM2_IP = `192.168.1.20`
- VM1_USER = `admin`
- VM2_USER = `webmaster`
- GITLAB_USERNAME = `john`
- PROJECT_NAME = `my-website`

---

## 🎯 สิ่งที่จะได้

เมื่อคุณ `git push` code → เว็บไซต์จะอัปเดตอัตโนมัติภายใน 1-2 นาที โดยไม่ต้องทำอะไรเพิ่ม

---

# 📍 ขั้นตอนที่ 1: ติดตั้ง GitLab Runner

## 🖥️ ทำบน VM1 (GitLab Server)

### 1.1 SSH เข้า VM1

```bash
ssh <VM1_USER>@<VM1_IP>
```

**ตัวอย่าง:**
```bash
ssh admin@192.168.1.10
```

### 1.2 สร้างโฟลเดอร์เก็บ config

```bash
sudo mkdir -p /srv/gitlab-runner/config
```

### 1.3 รัน GitLab Runner ด้วย Docker

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

### 1.4 ตรวจสอบว่า Runner ทำงาน

```bash
docker ps | grep gitlab-runner
```

**ควรเห็นแบบนี้:**

```
CONTAINER ID   IMAGE                         STATUS
abc123def456   gitlab/gitlab-runner:latest   Up 5 seconds
```

✅ **ถ้าเห็น Container กำลังรัน = สำเร็จ**

---

# 📍 ขั้นตอนที่ 2: ลงทะเบียน Runner กับ GitLab

## 🖥️ ยังอยู่บน VM1

### 2.1 เปิดเว็บ GitLab เพื่อหา Registration Token

1. เปิดเบราว์เซอร์ไปที่ `http://<VM1_IP>`
   - **ตัวอย่าง:** `http://192.168.1.10`

2. Login เข้า GitLab

3. เข้าไปที่ **Project** ที่ต้องการใช้ CI/CD (หรือสร้างใหม่)

4. คลิกเมนู **Settings** (ด้านซ้ายล่าง)

5. คลิก **CI/CD**

6. หาหัวข้อ **Runners** แล้วกด **Expand**

7. **📋 คัดลอก Registration Token**

**ตัวอย่าง token:**
```
GR1348941abcdefghijklmnop
```

### 2.2 ลงทะเบียน Runner

**กลับไปที่ VM1 Terminal:**

```bash
docker exec -it gitlab-runner gitlab-runner register
```

จะมีคำถามให้ตอบ **ทีละข้อ**:

#### ❓ Enter the GitLab instance URL

```
Enter the GitLab instance URL (for example, https://gitlab.com/):
```

**✏️ ตอบ:**
```
http://<VM1_IP>/
```

**ตัวอย่าง:**
```
http://192.168.1.10/
```

#### ❓ Enter the registration token

```
Enter the registration token:
```

**✏️ ตอบ:** วาง token ที่ copy มา

#### ❓ Enter a description

```
Enter a description for the runner:
```

**✏️ ตอบ:**
```
local-runner
```

#### ❓ Enter tags

```
Enter tags for the runner (comma-separated):
```

**✏️ ตอบ:**
```
deploy
```

#### ❓ Enter an executor

```
Enter an executor: docker, shell, ssh, etc.:
```

**✏️ ตอบ:**
```
docker
```

#### ❓ Enter default Docker image

```
Enter the default Docker image (for example, ruby:2.7):
```

**✏️ ตอบ:**
```
alpine:latest
```

### 2.3 ตรวจสอบว่าลงทะเบียนสำเร็จ

```bash
docker exec -it gitlab-runner gitlab-runner list
```

**ควอเห็นแบบนี้:**

```
Listing configured runners          ConfigFile=/etc/gitlab-runner/config.toml
local-runner                        Executor=docker Token=abc123 URL=http://<VM1_IP>/
```

### 2.4 ตรวจสอบบนเว็บ GitLab

กลับไปที่หน้า **Settings → CI/CD → Runners**

ควรเห็น Runner **สีเขียว** พร้อมข้อความ **"online"**

✅ **ถ้าเป็นสีเขียว = พร้อมใช้งาน**

---

# 📍 ขั้นตอนที่ 3: ตั้งค่า SSH ให้ Runner เข้า Web Server ได้

## 🖥️ ยังอยู่บน VM1

### 3.1 เข้าไปใน Runner Container

```bash
docker exec -it gitlab-runner bash
```

**Prompt จะเปลี่ยนเป็น:**
```
root@abc123def456:/#
```

### 3.2 สร้าง SSH Key

```bash
ssh-keygen -t rsa -b 4096 -N "" -f /root/.ssh/id_rsa
```

**จะเห็น:**
```
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```

### 3.3 แสดง Public Key

```bash
cat /root/.ssh/id_rsa.pub
```

**จะเห็นข้อความยาว ๆ แบบนี้:**
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC...ยาวมาก...xxx root@abc123def456
```

**📋 คัดลอก (Copy) ข้อความทั้งหมด** (จาก `ssh-rsa` จนถึงท้ายสุด)

### 3.4 ออกจาก Container

```bash
exit
```

**Prompt จะกลับเป็น:**
```
<VM1_USER>@vm1:~$
```

---

# 📍 ขั้นตอนที่ 4: เพิ่ม SSH Key ไปยัง Web Server

## 🖥️ เปิด Terminal ใหม่ → SSH เข้า VM2

```bash
ssh <VM2_USER>@<VM2_IP>
```

**ตัวอย่าง:**
```bash
ssh webmaster@192.168.1.20
```

### 4.1 สร้างโฟลเดอร์ .ssh (ถ้ายังไม่มี)

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

### 4.2 เพิ่ม Public Key

```bash
nano ~/.ssh/authorized_keys
```

**📋 วาง (Paste)** public key ที่ copy มาจากขั้นตอนที่ 3.3

**กด:**
- `Ctrl + X` (ออก)
- `Y` (บันทึก)
- `Enter` (ยืนยัน)

### 4.3 ตั้งสิทธิ์ไฟล์

```bash
chmod 600 ~/.ssh/authorized_keys
```

### 4.4 ออกจาก VM2

```bash
exit
```

---

# 📍 ขั้นตอนที่ 5: ทดสอบ SSH Connection

## 🖥️ กลับไปที่ VM1

```bash
ssh <VM1_USER>@<VM1_IP>
```

### 5.1 เข้า Runner Container

```bash
docker exec -it gitlab-runner bash
```

### 5.2 เพิ่ม Web Server เข้า known_hosts

```bash
ssh-keyscan -H <VM2_IP> >> /root/.ssh/known_hosts
```

**ตัวอย่าง:**
```bash
ssh-keyscan -H 192.168.1.20 >> /root/.ssh/known_hosts
```

### 5.3 ทดสอบ SSH

```bash
ssh <VM2_USER>@<VM2_IP> "echo 'SSH Connection Success!'"
```

**ตัวอย่าง:**
```bash
ssh webmaster@192.168.1.20 "echo 'SSH Connection Success!'"
```

**ควรเห็น:**
```
SSH Connection Success!
```

✅ **ถ้าเห็นข้อความนี้ = SSH ทำงานแล้ว**

❌ **ถ้าเห็น "Permission denied"** = กลับไปทำขั้นตอนที่ 4 ใหม่

### 5.4 ออกจาก Container

```bash
exit
```

### 5.5 ออกจาก VM1

```bash
exit
```

---

# 📍 ขั้นตอนที่ 6: เตรียม Web Server

## 🖥️ SSH เข้า VM2

```bash
ssh <VM2_USER>@<VM2_IP>
```

**ตัวอย่าง:**
```bash
ssh webmaster@192.168.1.20
```

### 6.1 สร้างโฟลเดอร์สำหรับเว็บ

```bash
sudo mkdir -p /srv/webapp
sudo chown -R $USER:$USER /srv/webapp
cd /srv/webapp
```

### 6.2 สร้างไฟล์ docker-compose.yml

```bash
nano docker-compose.yml
```

**📋 วางโค้ดนี้:**

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:latest
    container_name: nginx-web
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    restart: always
```

**กด:**
- `Ctrl + X`
- `Y`
- `Enter`

### 6.3 ตรวจสอบไฟล์

```bash
cat docker-compose.yml
```

ควรเห็นโค้ดที่วางไว้

### 6.4 ตรวจสอบโครงสร้างโฟลเดอร์

```bash
ls -la
```

**ควรเห็น:**
```
total 12
drwxr-xr-x  2 <VM2_USER> <VM2_USER> 4096 Nov  5 10:30 .
drwxr-xr-x  3 root       root       4096 Nov  5 10:25 ..
-rw-r--r--  1 <VM2_USER> <VM2_USER>  234 Nov  5 10:30 docker-compose.yml
```

**📌 สังเกต:** ยังไม่มีโฟลเดอร์ `html` (ถูกต้องแล้ว - จะสร้างอัตโนมัติจาก Pipeline)

### 6.5 ออกจาก VM2

```bash
exit
```

✅ **VM2 พร้อมแล้ว!**

---

# 📍 ขั้นตอนที่ 7: สร้างโปรเจกต์

## 💻 ทำบนเครื่อง Developer (เครื่องของคุณ)

### 7.1 สร้างโปรเจกต์ใน GitLab (ถ้ายังไม่มี)

1. เปิด GitLab `http://<VM1_IP>`

2. คลิก **New project**

3. เลือก **Create blank project**

4. ตั้งชื่อ เช่น `<PROJECT_NAME>`

5. เลือก **Public** หรือ **Private** (แนะนำ Public สำหรับทดสอบ)

6. ✅ เลือก **Initialize repository with a README**

7. คลิก **Create project**

### 7.2 📝 จดบันทึก Project Path

จะเห็น URL แบบนี้:
```
http://<VM1_IP>/<GITLAB_USERNAME>/<PROJECT_NAME>
```

**ตัวอย่าง:**
```
http://192.168.1.10/john/my-website
```

**📋 จด Project Path:**
```
<GITLAB_USERNAME>/<PROJECT_NAME>
```

**ตัวอย่าง:**
```
john/my-website
```

---

# 📍 ขั้นตอนที่ 8: Clone โปรเจกต์มาที่เครื่อง

## 💻 ยังอยู่บนเครื่อง Developer

### 8.1 Clone Project

```bash
cd ~/Projects  # หรือโฟลเดอร์ที่ต้องการ

git clone http://<VM1_IP>/<GITLAB_USERNAME>/<PROJECT_NAME>.git
cd <PROJECT_NAME>
```

**ตัวอย่าง:**
```bash
cd ~/Projects
git clone http://192.168.1.10/john/my-website.git
cd my-website
```

---

# 📍 ขั้นตอนที่ 9: สร้างไฟล์เว็บไซต์

## 💻 ยังอยู่บนเครื่อง Developer (ในโฟลเดอร์โปรเจกต์)

### 9.1 สร้างไฟล์ index.html

```bash
nano index.html
```

**📋 วางโค้ดนี้:**

```html
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🚀 CI/CD Auto Deploy</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }
        .container {
            background: rgba(255, 255, 255, 0.1);
            padding: 60px 40px;
            border-radius: 20px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
            text-align: center;
            max-width: 600px;
        }
        h1 {
            font-size: 3em;
            margin-bottom: 20px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }
        p {
            font-size: 1.3em;
            margin: 15px 0;
            line-height: 1.6;
        }
        .badge {
            display: inline-block;
            background: rgba(255, 255, 255, 0.2);
            padding: 15px 40px;
            border-radius: 30px;
            margin-top: 30px;
            font-weight: bold;
            font-size: 1.2em;
            border: 2px solid rgba(255, 255, 255, 0.3);
        }
        .emoji {
            font-size: 1.5em;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1><span class="emoji">🚀</span> CI/CD Auto Deploy</h1>
        <p>เว็บไซต์นี้ถูก Deploy อัตโนมัติ</p>
        <p>ผ่าน GitLab CI/CD Pipeline</p>
        <p><span class="emoji">⚡</span> ไม่ต้อง SSH เข้า Server เลย!</p>
        <div class="badge">
            <span class="emoji">✨</span> Version 1.0
        </div>
    </div>
</body>
</html>
```

**กด:**
- `Ctrl + X`
- `Y`
- `Enter`

---

# 📍 ขั้นตอนที่ 10: สร้างไฟล์ CI/CD

## 💻 ยังอยู่บนเครื่อง Developer (โฟลเดอร์เดียวกัน)

### 10.1 สร้างไฟล์ .gitlab-ci.yml

```bash
nano .gitlab-ci.yml
```

**📋 วางโค้ดนี้:**

```yaml
stages:
  - deploy

variables:
  WEB_SERVER_IP: "<VM2_IP>"                    # 🔧 แก้เป็น IP ของ VM2
  WEB_SERVER_USER: "<VM2_USER>"                # 🔧 แก้เป็น username ของ VM2
  GITLAB_SERVER_IP: "<VM1_IP>"                 # 🔧 แก้เป็น IP ของ VM1
  PROJECT_PATH: "<GITLAB_USERNAME>/<PROJECT_NAME>"  # 🔧 แก้เป็น path ของ project
  DEPLOY_PATH: "/srv/webapp"

deploy_to_webserver:
  stage: deploy
  image: alpine:latest
  tags:
    - deploy
  before_script:
    - echo "🔧 Installing dependencies..."
    - apk add --no-cache openssh-client git
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan -H $WEB_SERVER_IP >> ~/.ssh/known_hosts
    - echo "✅ Dependencies installed"
    
  script:
    - echo "🚀 Starting deployment to $WEB_SERVER_IP..."
    - |
      ssh $WEB_SERVER_USER@$WEB_SERVER_IP << 'ENDSSH'
        set -e
        
        cd /srv/webapp
        
        # ตรวจสอบว่ามี project อยู่แล้วหรือไม่
        if [ ! -d html/.git ]; then
          echo "📦 🎉 First deployment - Cloning repository..."
          rm -rf html
          git clone http://${GITLAB_SERVER_IP}/${PROJECT_PATH}.git html
          echo "✅ Repository cloned successfully!"
          
          echo "🐳 Starting nginx container..."
          docker compose up -d
          echo "✅ Nginx started!"
          
        else
          echo "🔄 Updating existing deployment..."
          cd html
          git fetch origin
          git reset --hard origin/main
          git pull origin main
          echo "✅ Code updated!"
          
          cd ..
          echo "🔄 Restarting nginx..."
          docker compose restart nginx
          echo "✅ Nginx restarted!"
        fi
        
        echo ""
        echo "═══════════════════════════════════════"
        echo "📊 Deployment Summary"
        echo "═══════════════════════════════════════"
        echo "📂 Path: /srv/webapp/html"
        echo "🌿 Branch: main"
        echo "📝 Latest commit:"
        cd html && git log -1 --pretty=format:"   %h - %s (%an)" && echo ""
        echo "═══════════════════════════════════════"
        echo "🎉 Deployment completed successfully!"
        echo "🌐 Website: http://${WEB_SERVER_IP}"
        echo "═══════════════════════════════════════"
      ENDSSH
      
    - echo "✅ All tasks completed!"
    
  only:
    - main
  when: on_success
```

### 10.2 🔧 แก้ไขค่าตามจริง

**ให้แก้ 4 บรรทัดในส่วน `variables:`:**

| ตัวแปร | ค่าที่ต้องแก้ | ตัวอย่าง |
|--------|--------------|---------|
| `WEB_SERVER_IP` | IP ของ VM2 | `192.168.1.20` |
| `WEB_SERVER_USER` | Username ของ VM2 | `webmaster` |
| `GITLAB_SERVER_IP` | IP ของ VM1 | `192.168.1.10` |
| `PROJECT_PATH` | Path ของ project | `john/my-website` |

**ตัวอย่างหลังแก้:**

```yaml
variables:
  WEB_SERVER_IP: "192.168.1.20"
  WEB_SERVER_USER: "webmaster"
  GITLAB_SERVER_IP: "192.168.1.10"
  PROJECT_PATH: "john/my-website"
  DEPLOY_PATH: "/srv/webapp"
```

**กด:**
- `Ctrl + X`
- `Y`
- `Enter`

---

# 📍 ขั้นตอนที่ 11: Push Code ครั้งแรก

## 💻 ยังอยู่บนเครื่อง Developer

### 11.1 ตรวจสอบไฟล์

```bash
ls -la
```

**ควรเห็น:**
```
total 24
drwxr-xr-x  3 user user 4096 Nov  5 11:00 .
drwxr-xr-x  5 user user 4096 Nov  5 10:45 ..
drwxr-xr-x  8 user user 4096 Nov  5 10:50 .git
-rw-r--r--  1 user user 2345 Nov  5 11:00 .gitlab-ci.yml
-rw-r--r--  1 user user 3456 Nov  5 10:55 index.html
-rw-r--r--  1 user user  100 Nov  5 10:50 README.md
```

### 11.2 เพิ่มไฟล์เข้า Git

```bash
git add .gitlab-ci.yml index.html
```

### 11.3 Commit

```bash
git commit -m "Add CI/CD pipeline and website"
```

### 11.4 Push ไป GitLab

```bash
git push origin main
```

**จะเห็น:**
```
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 2.5 KiB | 2.5 MiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To http://<VM1_IP>/<GITLAB_USERNAME>/<PROJECT_NAME>.git
   abc1234..def5678  main -> main
```

✅ **Push สำเร็จ!**

---

# 📍 ขั้นตอนที่ 12: ดู Pipeline ทำงาน

## 🌐 เปิดเว็บเบราว์เซอร์

### 12.1 เข้า GitLab

```
http://<VM1_IP>
```

**ตัวอย่าง:**
```
http://192.168.1.10
```

### 12.2 เข้าโปรเจกต์

คลิกที่โปรเจกต์ `<PROJECT_NAME>`

### 12.3 ดู Pipeline

1. เมนูด้านซ้าย → คลิก **CI/CD**
2. คลิก **Pipelines**

**จะเห็น Pipeline Status:**
- 🔵 **running** = กำลังรัน
- 🟢 **passed** = สำเร็จ
- 🔴 **failed** = ล้มเหลว

### 12.4 ดู Logs

1. คลิกที่ Pipeline (บรรทัดล่าสุด)
2. คลิกที่ Job **deploy_to_webserver**
3. จะเห็น logs แบบนี้:

```
Running with gitlab-runner 16.x.x
  on local-runner abc123

🔧 Installing dependencies...
✅ Dependencies installed
🚀 Starting deployment to <VM2_IP>...
📦 🎉 First deployment - Cloning repository...
Cloning into 'html'...
✅ Repository cloned successfully!
🐳 Starting nginx container...
[+] Running 2/2
 ✔ Network webapp_default  Created
 ✔ Container nginx-web     Started
✅ Nginx started!

═══════════════════════════════════════
📊 Deployment Summary
═══════════════════════════════════════
📂 Path: /srv/webapp/html
🌿 Branch: main
📝 Latest commit:
   abc1234 - Add CI/CD pipeline and website (Your Name)
═══════════════════════════════════════
🎉 Deployment completed successfully!
🌐 Website: http://<VM2_IP>
═══════════════════════════════════════

✅ All tasks completed!
Job succeeded
```

✅ **ถ้าเห็น "Job succeeded" สีเขียว = สำเร็จสมบูรณ์!**

---

# 📍 ขั้นตอนที่ 13: เปิดเว็บไซต์

## 🌐 เปิดเบราว์เซอร์ใหม่

```
http://<VM2_IP>
```

**ตัวอย่าง:**
```
http://192.168.1.20
```

**ควรเห็นหน้าเว็บสีม่วง** พร้อมข้อความ:
- 🚀 CI/CD Auto Deploy
- Version 1.0

✅ **เห็นหน้าเว็บ = สำเร็จสมบูรณ์!**

---

# 📍 ขั้นตอนที่ 14: ทดสอบการอัปเดตอัตโนมัติ

## 💻 กลับไปที่เครื่อง Developer

### 14.1 แก้ไขไฟล์ index.html

```bash
cd ~/Projects/<PROJECT_NAME>
nano index.html
```

**เปลี่ยนบรรทัดนี้:**

```html
<div class="badge">
    <span class="emoji">✨</span> Version 1.0
</div>
```

**เป็น:**

```html
<div class="badge">
    <span class="emoji">✨</span> Version 2.0 - Auto Updated! 🎉
</div>
```

**กด:**
- `Ctrl + X`
- `Y`
- `Enter`

### 14.2 Push การเปลี่ยนแปลง

```bash
git add index.html
git commit -m "Update to version 2.0"
git push origin main
```

### 14.3 ดู Pipeline

- เปิด GitLab → CI/CD → Pipelines
- จะเห็น Pipeline ใหม่กำลังรัน
- คลิกดู Logs จะเห็น:

```
🔄 Updating existing deployment...
From http://<VM1_IP>/<GITLAB_USERNAME>/<PROJECT_NAME>
 * branch            main       -> FETCH_HEAD
HEAD is now at def5678 Update to version 2.0
Already up to date.
✅ Code updated!
🔄 Restarting nginx...
nginx-web
✅ Nginx restarted!
🎉 Deployment completed successfully!
```

### 14.4 Refresh เว็บไซต์

เปิด `http://<VM2_IP>` แล้วกด **Ctrl + F5** (refresh แบบไม่ใช้ cache)

✅ **ควรเห็น Version 2.0 - Auto Updated! 🎉**

---

# 📍 ขั้นตอนที่ 15: ตรวจสอบบน Web Server (Optional)

## 🖥️ SSH เข้า VM2

```bash
ssh <VM2_USER>@<VM2_IP>
```

### 15.1 ตรวจสอบโครงสร้างไฟล์

```bash
cd /srv/webapp
ls -la
```

**ควรเห็น:**
```
total 16
drwxr-xr-x  3 <VM2_USER> <VM2_USER> 4096 Nov  5 11:00 .
drwxr-xr-x  3 root       root       4096 Nov  5 10:25 ..
-rw-r--r--  1 <VM2_USER> <VM2_USER>  234 Nov  5 10:30 docker-compose.yml
drwxr-xr-x  8 <VM2_USER> <VM2_USER> 4096 Nov  5 11:00 html
```

### 15.2 ตรวจสอบโฟลเดอร์ html

```bash
ls -la html/
```

**ควรเห็น:**
```
total 32
drwxr-xr-x  8 <VM2_USER> <VM2_USER> 4096 Nov  5 11:00 .
drwxr-xr-x  3 <VM2_USER> <VM2_USER> 4096 Nov  5 11:00 ..
drwxr-xr-x  8 <VM2_USER> <VM2_USER> 4096 Nov  5 11:00 .git
-rw-r--r--  1 <VM2_USER> <VM2_USER> 2345 Nov  5 11:00 .gitlab-ci.yml
-rw-r--r--  1 <VM2_USER> <VM2_USER> 3456 Nov  5 10:55 index.html
-rw-r--r--  1 <VM2_USER> <VM2_USER>  100 Nov  5 10:50 README.md
```

### 15.3 ตรวจสอบ Nginx Container

```bash
docker ps | grep nginx
```

**ควรเห็น:**
```
abc123  nginx:latest  "nginx -g 'daemon of…"  Up 5 minutes  0.0.0.0:80->80/tcp  nginx-web
```

### 15.4 ดู Logs ของ Nginx

```bash
docker compose logs nginx
```

### 15.5 ตรวจสอบ Git Commit ล่าสุด

```bash
cd html
git log -1
```

**จะเห็น:**
```
commit def5678...
Author: Your Name <your@email.com>
Date:   Wed Nov 5 11:05:00 2025 +0700

    Update to version 2.0
```

### 15.6 ออกจาก VM2

```bash
exit
```

---

# 🎉 สรุป: คุณทำสำเร็จแล้ว!

## ✅ Checklist การตั้งค่า

- [x] ติดตั้ง GitLab Runner บน VM1
- [x] ลงทะเบียน Runner กับ GitLab Project
- [x] ตั้งค่า SSH จาก Runner → Web Server
- [x] เตรียม Web Server พร้อม Docker Compose
- [x] สร้างโปรเจกต์และไฟล์เว็บ
- [x] สร้างไฟล์ .gitlab-ci.yml
- [x] Push code → Deploy อัตโนมัติ
- [x] ทดสอบการอัปเดตอัตโนมัติ

---

## 🔄 Flow การทำงานของระบบ

```
Developer                    VM1 (GitLab)              VM2 (Web Server)
    │                              │                           │
    │ 1. แก้ไข code               │                           │
    │    git commit                │                           │
    │    git push                  │                           │
    ├──────────────────────────────►                           │
    │                              │                           │
    │                              │ 2. Detect push           │
    │                              │    Trigger Pipeline      │
    │                              │    (GitLab Runner)       │
    │                              │                           │
    │                              │ 3. SSH to Web Server     │
    │                              ├───────────────────────────►
    │                              │                           │
    │                              │                           │ 4. Clone/Pull code
    │                              │                           │    จาก GitLab
    │                              │                           │
    │                              │                           │ 5. Start/Restart
    │                              │                           │    Nginx container
    │                              │                           │
    │                              │ 6. Report success        │
    │                              ◄───────────────────────────┤
    │                              │                           │
    │ 7. เปิดเว็บดู               │                           │
    │                              │                           │
    └──────────────────────────────┼───────────────────────────►
                                   │                  http://<VM2_IP>
```

---

## 🚀 ตั้งแต่นี้ไป...

**ทุกครั้งที่คุณ:**

```bash
git add .
git commit -m "อัปเดตอะไรก็ได้"
git push origin main
```

**ระบบจะ:**
1. ✅ Trigger Pipeline อัตโนมัติ
2. ✅ Pull code ล่าสุดไปยัง Web Server
3. ✅ Restart Nginx
4. ✅ เว็บไซต์อัปเดตภายใน 1-2 นาที

**โดยที่คุณไม่ต้อง SSH เข้า Server เลย!** 🎉

---

## 🔧 การแก้ปัญหาที่พบบ่อย

### ❌ ปัญหา: Pipeline ไม่ทำงาน

**สาเหตุ:** Runner ไม่ online หรือ tags ไม่ตรงกัน

**วิธีแก้:**

1. ตรวจสอบ Runner:
```bash
# บน VM1
docker ps | grep gitlab-runner
docker exec -it gitlab-runner gitlab-runner list
```

2. ตรวจสอบ tags บน GitLab:
   - Settings → CI/CD → Runners
   - ต้องมี tag `deploy`

3. ตรวจสอบใน `.gitlab-ci.yml`:
```yaml
tags:
  - deploy  # ต้องตรงกับ Runner
```

---

### ❌ ปัญหา: SSH Permission Denied

**สาเหตุ:** Public key ไม่ถูกต้อง

**วิธีแก้:**

```bash
# บน VM1
docker exec -it gitlab-runner cat /root/.ssh/id_rsa.pub
# Copy public key

# บน VM2
nano ~/.ssh/authorized_keys
# Paste public key แล้วบันทึก
chmod 600 ~/.ssh/authorized_keys

# ทดสอบ SSH จาก VM1
docker exec -it gitlab-runner ssh <VM2_USER>@<VM2_IP> "echo Test"
```

---

### ❌ ปัญหา: Git Clone ล้มเหลว

**สาเหตุ:** VM2 เข้าถึง GitLab ไม่ได้

**วิธีแก้:**

```bash
# บน VM2
ping <VM1_IP>
curl http://<VM1_IP>

# ถ้าไม่ได้ ตรวจสอบ firewall
sudo ufw status
sudo ufw allow from <VM2_IP> to any port 80
```

---

### ❌ ปัญหา: Nginx ไม่แสดงเนื้อหาใหม่

**วิธีแก้:**

```bash
# บน VM2
cd /srv/webapp
docker compose logs nginx
docker compose restart nginx

# ตรวจสอบไฟล์
cat html/index.html

# ถ้ายังไม่อัปเดต ลอง pull ใหม่
cd html
git pull origin main
cd ..
docker compose restart nginx
```

---

### ❌ ปัญหา: Pipeline ติด "This job is stuck"

**สาเหตุ:** ไม่มี Runner available หรือ Runner ไม่รับงาน

**วิธีแก้:**

1. ตรวจสอบ Runner Settings:
   - Settings → CI/CD → Runners
   - คลิกที่ Runner
   - ตรวจสอบว่า:
     - ✅ "Run untagged jobs" เปิดอยู่ (ถ้าไม่ใช้ tags)
     - ✅ "Active" เปิดอยู่

2. Restart Runner:
```bash
# บน VM1
docker restart gitlab-runner
```

---

## 📚 ไฟล์สำคัญและที่อยู่

| ไฟล์/โฟลเดอร์ | ที่อยู่ | หมายเหตุ |
|--------------|--------|---------|
| GitLab Runner Config | VM1: `/srv/gitlab-runner/config/config.toml` | ตั้งค่า Runner |
| SSH Private Key | VM1: `gitlab-runner` container `/root/.ssh/id_rsa` | สำหรับ SSH ไป VM2 |
| SSH Public Key | VM2: `~/.ssh/authorized_keys` | อนุญาตให้ Runner เข้าได้ |
| Docker Compose | VM2: `/srv/webapp/docker-compose.yml` | ตั้งค่า Nginx |
| Website Files | VM2: `/srv/webapp/html/` | ไฟล์เว็บจาก Git |
| CI/CD Pipeline | Project: `.gitlab-ci.yml` | สคริปต์ deploy |

---

## 💡 เพิ่มความปลอดภัย (แนะนำ)

### 1. ใช้ GitLab CI/CD Variables แทนการใส่ IP ตรง ๆ

ใน GitLab:
1. Settings → CI/CD → Variables → Expand
2. เพิ่ม Variables:
   - `VM2_IP` = `<VM2_IP>`
   - `VM2_USER` = `<VM2_USER>`

แก้ไข `.gitlab-ci.yml`:
```yaml
variables:
  WEB_SERVER_IP: "$VM2_IP"        # ใช้ตัวแปรจาก GitLab
  WEB_SERVER_USER: "$VM2_USER"
```

### 2. ใช้ Git Tag สำหรับ Production

แทนที่จะ deploy ทุกครั้งที่ push:

```yaml
only:
  - tags  # deploy เฉพาะเมื่อสร้าง tag
  # - main  # ปิดการ auto deploy จาก main
```

Deploy ด้วยการสร้าง tag:
```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

### 3. เพิ่ม Notification เมื่อ Deploy สำเร็จ

เพิ่มใน `.gitlab-ci.yml`:
```yaml
after_script:
  - echo "📧 Sending notification..."
  # เพิ่ม webhook notification ตามต้องการ
```

---

## 🎯 ขั้นต่อไป

หลังจากระบบใช้งานได้แล้ว คุณสามารถ:

1. **เพิ่ม Testing Stage** - ทดสอบ code ก่อน deploy
2. **เพิ่ม Staging Environment** - ทดสอบบน VM อื่นก่อน production
3. **เพิ่ม Rollback** - กลับไปเวอร์ชันเก่าได้ถ้ามีปัญหา
4. **ใช้ Docker Image สำหรับ Application** - แทนการใช้ HTML แบบธรรมดา
5. **เพิ่ม HTTPS** - ใช้ Let's Encrypt หรือ Self-signed Certificate

---

## 📞 ต้องการความช่วยเหลือ?

ถ้าเจอปัญหา:

1. ✅ ตรวจสอบ Pipeline Logs ใน GitLab
2. ✅ ตรวจสอบ Runner Logs: `docker logs gitlab-runner`
3. ✅ ตรวจสอบ Nginx Logs: `docker compose logs nginx`
4. ✅ ทดสอบ SSH manually จาก Runner
5. ✅ ตรวจสอบว่า IP และ Username ถูกต้อง

---

## 🏆 ขอแสดงความยินดี!

คุณได้ตั้งค่า CI/CD Pipeline สำเร็จแล้ว! 🎉

ตั้งแต่นี้ไป การ deploy เว็บไซต์จะง่ายเหมือน:

```bash
git push origin main
```

**แค่นี้เว็บก็อัปเดตแล้ว!** ☕️🚀