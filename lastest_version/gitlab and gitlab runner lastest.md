
**การติดตั้ง Gitlab server เเละ Gitlab runner**
- ตัว gitlab runner เราจะนำไปติดตั้งเเยกจาก gitlab 
- เนื่องจากถ้าติดตั้งตัว runner บน web_server จะไม่ต้องใช้ ssh key เพื่อทำการ remote ไปที่ web server เพื่อนำโค้ดจาก gitlab มา deploy ที่ปลายทาง web server

**ตัวอย่าง IP สมมติในการติดตั้งในคู่มือ**
- webserver & gitlab runner: 192.168.254.129
- gitlab server : 192.168.254.128
- registry server : 192.168.254.131	
___
### Gitlab server
## 1 สร้างไดเรกทอรีโครงการ

สร้างไดเรกทอรีหลักสำหรับโครงการ:

```bash
mkdir gitlab-ci-cd
cd gitlab-ci-cd
```
## 1.1 สร้างไดเรกทอรีสำหรับจัดเก็บข้อมูล

สร้างไดเรกทอรีย่อยสำหรับจัดเก็บข้อมูลของ GitLab

```bash
mkdir -p gitlab/config gitlab/logs gitlab/data
```

**คำอธิบาย:**
- `gitlab/config` - เก็บไฟล์การตั้งค่า
- `gitlab/logs` - เก็บไฟล์บันทึกการทำงาน
- `gitlab/data` - เก็บข้อมูลหลักของ GitLab

## ขั้นตอนที่ 2: การสร้างไฟล์ Docker Compose

สร้างไฟล์ `docker-compose.yml` ในไดเรกทอรี `gitlab-ci-cd`:

```yaml
version: "3.7"
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    container_name: gitlab
    restart: always
    hostname: '192.168.254.128' #ตรงนี้เเนะนำใส่เป็น ip ของ server gitlab
    environment:
    # ส่วนนี้สำคัญ ถ้าหาก hostname ใช้เป็น ip เหมือนกัน ถ้าใช้ domian ก็ต้องเป็น domain เหมือนกัน สำหรับคู่มือตรงนี้ต้องเป็น ip เเละใส่ port 
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://192.168.254.128:8000/'  
    ports:
      - '8000:8000'
      - '4430:443'
      - '2200:22'
    volumes:
      - './gitlab/config:/etc/gitlab'
      - './gitlab/logs:/var/log/gitlab'
      - './gitlab/data:/var/opt/gitlab'
    shm_size: '256m'
 ```   
## 3 เริ่มต้น Container

รันคำสั่งเพื่อเริ่มต้นระบบ:

```bash
sudo docker compose up -d
```

**คำเตือน:** กระบวนการนี้ใช้เวลาประมาณ 5-15 นาที

### 3.2 ตรวจสอบสถานะ

ตรวจสอบว่า Container ทำงานปกติ:

```bash
sudo docker ps
```

ควรเห็น Container อยู่ในสถานะ "Up"

---

## ขั้นตอนที่ 4: การตั้งค่าบัญชีผู้ดูแลระบบ

### 4.1 การดึงรหัสผ่านเริ่มต้น

GitLab จะสร้างรหัสผ่านชั่วคราวสำหรับบัญชี root โดยอัตโนมัติ ใช้คำสั่งนี้เพื่อดูรหัสผ่าน:

```bash
sudo docker exec gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

ผลลัพธ์จะแสดงในรูปแบบ:

```
Password: ABC-xyz-1234567890
```

**หมายเหตุ:** บันทึกรหัสผ่านนี้ไว้ใช้งานชั่วคราว

### 4.2 การเข้าสู่ระบบครั้งแรก

1. เปิดเว็บเบราว์เซอร์และเข้าสู่ `http://192.168.254.128:8000`
2. กรอกข้อมูลการเข้าสู่ระบบ:
   - ชื่อผู้ใช้: `root`
   - รหัสผ่าน: รหัสผ่านที่ได้จากขั้นตอน 4.1
___
## Gitlab runner บน webserver 
## ขั้นตอนที่ 1: การเตรียมโครงสร้างไดเรกทอรี

### 1.1 สร้างไดเรกทอรีโครงการ

สร้างไดเรกทอรีหลักสำหรับโครงการ:

```bash
mkdir gitlab-runner
cd gitlab-runner
```

### 1.2 สร้างไดเรกทอรีสำหรับจัดเก็บข้อมูล

สร้างไดเรกทอรีย่อยสำหรับจัดเก็บข้อมูลของ GitLab และ GitLab Runner:

```bash
mkdir config
```

**คำอธิบาย:**
- `config` - เก็บการตั้งค่าของ Runner

---

## ขั้นตอนที่ 2: การสร้างไฟล์ Docker Compose

สร้างไฟล์ `docker-compose.yml` ในไดเรกทอรี `gitlab-ci-cd`:

```yaml
version: "3.7"
services:
  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    container_name: gitlab-runner
    restart: always
    depends_on:
      - gitlab
    volumes:
      - './config:/etc/gitlab-runner'
      - '/var/run/docker.sock:/var/run/docker.sock'
```
---
## ขั้นตอนที่ 3: การเริ่มระบบ

### 3.1 เริ่มต้น Container

รันคำสั่งเพื่อเริ่มต้นระบบ:

```bash
sudo docker compose up -d
```
 ## ขั้นตอนที่ 4: การลงทะเบียน GitLab Runner


# ขั้นตอน: ลงทะเบียน Runner กับ GitLab

## ยังอยู่ web server

###  1 เปิดเว็บ GitLab เพื่อหา Registration Token

1. เปิดเบราว์เซอร์ไปที่ `http://gitlab_IP>`
   - **ตัวอย่าง:** `http://192.168.254.128:8000`

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

### 2 ลงทะเบียน Runner

**กลับไปที่ web server Terminal:**

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
http://<gitlab_IP>/

```

**ตัวอย่าง:**
```
http://192.168.254.128:8000/
```

#### ❓ Enter the registration token

```
Enter the registration token:
```

**ตอบ:** วาง token ที่ copy มา

#### ❓ Enter a description

```
Enter a description for the runner:
```

**ตอบ:**
```
local-runner
```

#### ❓ Enter tags

```
Enter tags for the runner (comma-separated):
```

**ตอบ:**
```
deploy
```
**ถ้าเจอถาม Enter optional maintenance note for the runner:  **
```
ให้ใส่อะไรก็ได้เป็นเเค่เป็น note ไว้ 
```

#### ❓ Enter an executor

```
Enter an executor: docker, shell, ssh, etc.:
```

**ตอบ:**
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

### 2.1 ตรวจสอบว่าลงทะเบียนสำเร็จ

```bash
docker exec -it gitlab-runner gitlab-runner list
```

**ควรเห็นแบบนี้:**

```
Listing configured runners          ConfigFile=/etc/gitlab-runner/config.toml
local-runner                        Executor=docker Token=abc123 URL=http://192.168.254.128:8000/
```

### 2.4 ตรวจสอบบนเว็บ GitLab

กลับไปที่หน้า **Settings → CI/CD → Runners**

ควรเห็น Runner **สีเขียว** พร้อมข้อความ **"online"**

✅ **ถ้าเป็นสีเขียว = พร้อมใช้งาน**

---
# จบขั้นตอนการติดตั้ง 
### หลังจาก นำ Registration Token จาก Project ใน Gitlab ไปลงทะเบียนใน Gitlab runner ก็สามารถสร้างไฟล์ .gitlab-ci.yml ไว้ใน project เพื่อทำ pipeline ได้เลย 
