# คู่มือการติดตั้ง GitLab CE และ GitLab Runner บน Docker

## บทนำ

เอกสารนี้จะอธิบายขั้นตอนการติดตั้ง GitLab Community Edition (CE) พร้อมกับ GitLab Runner โดยใช้ Docker Compose สำหรับการทำ CI/CD (Continuous Integration/Continuous Deployment) บนระบบปฏิบัติการ Ubuntu Server

---

## ข้อกำหนดเบื้องต้น

ก่อนเริ่มการติดตั้ง กรุณาตรวจสอบให้แน่ใจว่าระบบของคุณมีส่วนประกอบต่อไปนี้:

- Docker Engine เวอร์ชันล่าสุด
- Docker Compose เวอร์ชันล่าสุด
- Ubuntu Server (แนะนำ 20.04 LTS ขึ้นไป)
- สิทธิ์ผู้ดูแลระบบ (sudo)

---

## ขั้นตอนที่ 1: การเตรียมโครงสร้างไดเรกทอรี

### 1.1 สร้างไดเรกทอรีโครงการ

สร้างไดเรกทอรีหลักสำหรับโครงการ:

```bash
mkdir gitlab-ci-cd
cd gitlab-ci-cd
```

### 1.2 สร้างไดเรกทอรีสำหรับจัดเก็บข้อมูล

สร้างไดเรกทอรีย่อยสำหรับจัดเก็บข้อมูลของ GitLab และ GitLab Runner:

```bash
mkdir -p gitlab/config gitlab/logs gitlab/data
mkdir -p gitlab-runner/config
```

**คำอธิบาย:**
- `gitlab/config` - เก็บไฟล์การตั้งค่า
- `gitlab/logs` - เก็บไฟล์บันทึกการทำงาน
- `gitlab/data` - เก็บข้อมูลหลักของ GitLab
- `gitlab-runner/config` - เก็บการตั้งค่าของ Runner

---

## ขั้นตอนที่ 2: การสร้างไฟล์ Docker Compose

สร้างไฟล์ `docker-compose.yml` ในไดเรกทอรี `gitlab-ci-cd`:

```yaml
version: "3.7"
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    container_name: gitlab
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.example.com'
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - './gitlab/config:/etc/gitlab'
      - './gitlab/logs:/var/log/gitlab'
      - './gitlab/data:/var/opt/gitlab'
    shm_size: '256m'
    
  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    container_name: gitlab-runner
    restart: always
    depends_on:
      - gitlab
    volumes:
      - './gitlab-runner/config:/etc/gitlab-runner'
      - '/var/run/docker.sock:/var/run/docker.sock'
```

**หมายเหตุสำคัญ:**
- แทนที่ `gitlab.example.com` ด้วย IP Address หรือ Domain Name ของเซิร์ฟเวอร์
- หากพอร์ต 22 ถูกใช้งานอยู่ ให้เปลี่ยนเป็น `'2222:22'`

---

## ขั้นตอนที่ 3: การเริ่มระบบ

### 3.1 เริ่มต้น Container

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

คุณควรเห็น Container ทั้งสองอยู่ในสถานะ "Up"

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

1. เปิดเว็บเบราว์เซอร์และเข้าสู่ `http://<IP-Address-ของคุณ>`
2. กรอกข้อมูลการเข้าสู่ระบบ:
   - ชื่อผู้ใช้: `root`
   - รหัสผ่าน: รหัสผ่านที่ได้จากขั้นตอน 4.1
3. ระบบจะแสดงหน้าต่างให้เปลี่ยนรหัสผ่าน
4. ตั้งรหัสผ่านใหม่ที่มีความปลอดภัยสูง
5. คลิกปุ่ม "Change your password"

---

## ขั้นตอนที่ 5: การลงทะเบียน GitLab Runner

### 5.1 การขอ Token สำหรับลงทะเบียน

1. เข้าสู่ระบบ GitLab ด้วยบัญชี root
2. คลิกที่ไอคอนเมนู > Admin Area
3. เลือกเมนู CI/CD > Runners
4. บันทึก Registration URL และ Registration Token

### 5.2 การลงทะเบียน Runner

รันคำสั่งเพื่อเริ่มกระบวนการลงทะเบียน:

```bash
sudo docker exec -it gitlab-runner gitlab-runner register
```

กรอกข้อมูลตามที่ระบบถาม:

| รายการ | ค่าที่แนะนำ | รายละเอียด |
|:-------|:-----------|:----------|
| GitLab instance URL | `http://gitlab` | ใช้ชื่อ Container เพื่อการสื่อสารภายในเครือข่าย Docker |
| Registration token | (Token จาก Admin Area) | คัดลอกจากหน้า Runners ในส่วน Admin |
| Runner description | `docker-executor` | ชื่อที่ใช้อธิบาย Runner นี้ |
| Runner tags | `docker,linux,ci` | Tags สำหรับกำหนดว่า Job ใดจะใช้ Runner นี้ |
| Executor | `docker` | ประเภทของ Executor ที่จะใช้รัน Job |
| Default Docker image | `alpine:latest` | Image พื้นฐานที่ใช้สำหรับรัน Job |

### 5.3 การตรวจสอบการลงทะเบียน

กลับไปที่หน้า Admin Area > CI/CD > Runners คุณจะเห็น Runner ที่เพิ่งลงทะเบียนปรากฏในรายการ

---

## สรุป

ขณะนี้คุณได้ติดตั้งและกำหนดค่า GitLab CE พร้อมกับ GitLab Runner สำเร็จแล้ว ระบบพร้อมสำหรับการใช้งาน CI/CD Pipeline

### ขั้นตอนถัดไป

- สร้าง Repository สำหรับโครงการ
- เพิ่มไฟล์ `.gitlab-ci.yml` เพื่อกำหนด Pipeline
- ทดสอบการทำงานของ CI/CD

---

## การแก้ไขปัญหาเบื้องต้น

**กรณีที่ GitLab ไม่สามารถเข้าถึงได้:**
- ตรวจสอบว่า Container ทำงานด้วย `sudo docker ps`
- ดู logs ด้วย `sudo docker logs gitlab`

**กรณีที่ Runner ไม่สามารถลงทะเบียนได้:**
- ตรวจสอบว่า URL และ Token ถูกต้อง
- ตรวจสอบว่า Container ทั้งสองสื่อสารกันได้

**การรีสตาร์ทระบบ:**
```bash
sudo docker compose restart
```

**การหยุดระบบ:**
```bash
sudo docker compose down
```