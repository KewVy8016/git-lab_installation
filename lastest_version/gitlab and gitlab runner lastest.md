# การติดตั้ง GitLab Server, GitLab Runner, Docker Registry และ Registry UI

**การติดตั้ง Gitlab server, Gitlab runner, Docker Registry และ Registry UI**
- ตัว gitlab runner เราจะนำไปติดตั้งแยกจาก gitlab
- เนื่องจากถ้าติดตั้งตัว runner บน web_server จะไม่ต้องใช้ ssh key เพื่อทำการ remote ไปที่ web server เพื่อนำโค้ดจาก gitlab มา deploy ที่ปลายทาง web server
- **Registry จะติดตั้งรวมกับ GitLab บน Server เดียวกัน**

**ตัวอย่าง IP สมมติในการติดตั้งในคู่มือ**
- webserver & gitlab runner: 192.168.254.129
- gitlab server & registry server: 192.168.254.128 **(ใช้ IP เดียวกัน)**

---

## ส่วนที่ 1: GitLab Server + Docker Registry

### 1. สร้างไดเรกทอรีโครงการ

สร้างไดเรกทอรีหลักสำหรับโครงการ:

```bash
mkdir gitlab-ci-cd
cd gitlab-ci-cd
```

### 1.1 สร้างไดเรกทอรีสำหรับจัดเก็บข้อมูล

สร้างไดเรกทอรีย่อยสำหรับจัดเก็บข้อมูลของ GitLab และ Registry:

```bash
mkdir -p gitlab/config gitlab/logs gitlab/data registry/data
```

**คำอธิบาย:**
- `gitlab/config` - เก็บไฟล์การตั้งค่า GitLab
- `gitlab/logs` - เก็บไฟล์บันทึกการทำงาน GitLab
- `gitlab/data` - เก็บข้อมูลหลักของ GitLab
- `registry/data` - เก็บ Docker images

---

## ส่วนที่ 2: การสร้างไฟล์ Docker Compose สำหรับ GitLab + Registry

สร้างไฟล์ `docker-compose.yml` ในไดเรกทอรี `gitlab-ci-cd`:

```yaml
version: "3.7"

version: "3.7"
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    container_name: gitlab
    restart: always
    hostname: '192.168.254.128'
    environment:
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
    networks:
      - gitlab-network

  registry:
    image: registry:2
    container_name: registry
    restart: always
    ports:
      - '5000:5000'
    volumes:
      - './registry/data:/var/lib/registry'
    command:
      - /bin/sh
      - -c
      - |
        cat > /etc/docker/registry/config.yml << 'EOF'
        version: 0.1
        log:
          level: info
        storage:
          filesystem:
            rootdirectory: /var/lib/registry
        http:
          addr: :5000
          headers:
            Access-Control-Allow-Origin: ['*']
            Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS', 'DELETE', 'PUT', 'POST']
            Access-Control-Allow-Headers: ['Authorization', 'Accept', 'Content-Type']
        EOF
        /entrypoint.sh /etc/docker/registry/config.yml
    networks:
      - gitlab-network

  registry-ui:
    image: joxit/docker-registry-ui:latest
    container_name: registry-ui
    restart: always
    ports:
      - '8080:80'
    environment:
      REGISTRY_TITLE: My Docker Registry
      REGISTRY_URL: http://192.168.254.128:5000
      DELETE_IMAGES: 'true'
      SHOW_CONTENT_DIGEST: 'true'
      SINGLE_REGISTRY: 'true'
    networks:
      - gitlab-network

networks:
  gitlab-network:
    driver: bridge
```

---

## ส่วนที่ 3: เริ่มต้น Container

รันคำสั่งเพื่อเริ่มต้นระบบ:

```bash
sudo docker compose up -d
```

**คำเตือน:** กระบวนการนี้ใช้เวลาประมาณ 5-15 นาที

### 3.1 ตรวจสอบสถานะ

ตรวจสอบว่า Container ทำงานปกติ:

```bash
sudo docker ps
```

ควรเห็น Container ทั้งหมดอยู่ในสถานะ "Up":
- `gitlab`
- `docker-registry`
- `registry-ui`

---

## ส่วนที่ 4: การตั้งค่าบัญชีผู้ดูแลระบบ GitLab

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

---

## ส่วนที่ 5: การเข้าถึง Docker Registry

### 5.1 เข้าใช้งาน Registry UI

เปิดเว็บเบราว์เซอร์และเข้าถึง:
```
http://192.168.254.128:8080
```

### 5.2 ใช้งาน Docker Registry

ทดสอบการเข้าถึง Registry:
```bash
curl http://192.168.254.128:5000/v2/_catalog
```

---

## ส่วนที่ 6: GitLab Runner บน Web Server

### 6.1 สร้างไดเรกทอรีโครงการ

**บน Web Server (192.168.254.129):**

สร้างไดเรกทอรีหลักสำหรับโครงการ:

```bash
mkdir gitlab-runner
cd gitlab-runner
```

### 6.2 สร้างไดเรกทอรีสำหรับจัดเก็บข้อมูล

สร้างไดเรกทอรีย่อยสำหรับจัดเก็บข้อมูลของ GitLab Runner:

```bash
mkdir config
```

**คำอธิบาย:**
- `config` - เก็บการตั้งค่าของ Runner

---

## ส่วนที่ 7: การสร้างไฟล์ Docker Compose สำหรับ GitLab Runner

สร้างไฟล์ `docker-compose.yml` ในไดเรกทอรี `gitlab-runner`:

```yaml
version: "3.7"
services:
  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    container_name: gitlab-runner
    restart: always
    privileged: true
    volumes:
      - './config:/etc/gitlab-runner'
      - '/var/run/docker.sock:/var/run/docker.sock'
      - '/usr/bin/docker:/usr/bin/docker'
      - '/usr/libexec/docker:/usr/libexec/docker'
```

**คำอธิบาย:**
- `privileged: true` - ให้สิทธิ์พิเศษแก่ container เพื่อสามารถใช้งาน Docker daemon
- `/var/run/docker.sock` - เชื่อมต่อกับ Docker socket ของ host
- `/usr/bin/docker` - bind mount Docker CLI
- `/usr/libexec/docker` - bind mount Docker plugins/utilities
  
**ให้เข้าไปเช็คทุกครั้งในไฟล์ config.toml ของ runner ว่า Privelleged เป็น True เเละมีการ mount docker sock เเล้ว ไฟล์นี้จะเเสดงขึ้นมาหลังจากทำการลงทะเบียน gitlab runner กับ project เสร็จสิ้น**
  - แก้ไขไฟล์ /etc/gitlab-runner/config.toml บนเครื่อง Host ที่รัน Runner โดยเพิ่มการตั้งค่า 2 ส่วนในบล็อก [runners.docker] ที่ตรงกับ Runner 

การตั้งค่า docker.sock ใน config.toml
1. เปิดโหมด privileged
ต้องกำหนดให้ Container ของ Runner มีสิทธิ์เข้าถึงอุปกรณ์และ Socket ต่าง ๆ ของ Host:
```bash
Ini, TOML
[runners.docker]
  # ...
  privileged = true
  # ...
 ``` 
2. Mount docker.sock เข้าไปใน Container
กำหนดให้ Runner Mount ไฟล์ Docker Socket ของ Host (/var/run/docker.sock) เข้าไปใน Container ที่ตำแหน่งเดียวกัน:
```bash
Ini, TOML
[runners.docker]
  # ...
  volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"] 
  # ...
```
ตัวอย่างที่เเก้ไขเเล้ว
```bash
[[runners]]
  name = "your-runner-name"
  executor = "docker"
  # ...
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = true # เปิดโหมดพิเศษ
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"] # Mount Socket
    shm_size = 0
    network_mtu = 0
    # ... ส่วนอื่นๆ
```

## ส่วนที่ 8: การเริ่มระบบ GitLab Runner

### 8.1 เพิ่ม User เข้ากลุ่ม Docker

ก่อนเริ่มต้น Container ต้องเพิ่ม user ปัจจุบันเข้ากลุ่ม docker:

```bash
sudo usermod -aG docker $USER
```

จากนั้นต้อง **Logout และ Login เข้าใหม่** หรือใช้คำสั่ง:

```bash
newgrp docker
```

ตรวจสอบว่าเข้ากลุ่มแล้ว:

```bash
groups
```

ควรเห็น `docker` ในรายการกลุ่ม

### 8.2 ตั้งค่า Permission Docker Socket

```bash
sudo chmod 666 /var/run/docker.sock
```

### 8.3 เริ่มต้น Container

รันคำสั่งเพื่อเริ่มต้นระบบ:

```bash
sudo docker compose up -d
```

---

## ส่วนที่ 9: การลงทะเบียน GitLab Runner

### 9.1 เปิดเว็บ GitLab เพื่อหา Registration Token

1. เปิดเบราว์เซอร์ไปที่ `http://192.168.254.128:8000`

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

### 9.2 ลงทะเบียน Runner

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

**ถ้าเจอถาม Enter optional maintenance note for the runner:**
```
ให้ใส่อะไรก็ได้เป็นแค่เป็น note ไว้
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

### 9.3 ตรวจสอบว่าลงทะเบียนสำเร็จ

```bash
docker exec -it gitlab-runner gitlab-runner list
```

**ควรเห็นแบบนี้:**

```
Listing configured runners          ConfigFile=/etc/gitlab-runner/config.toml
local-runner                        Executor=docker Token=abc123 URL=http://192.168.254.128:8000/
```

### 9.4 ตรวจสอบบนเว็บ GitLab

กลับไปที่หน้า **Settings → CI/CD → Runners**

ควรเห็น Runner **สีเขียว** พร้อมข้อความ **"online"**

✅ **ถ้าเป็นสีเขียว = พร้อมใช้งาน**

---

## ส่วนที่ 10: การใช้งาน Docker Registry พื้นฐาน

### 10.1 การตั้งค่า Insecure Registry

หากใช้ HTTP Registry (ไม่ใช้ HTTPS) จะต้องตั้งค่า Docker daemon ในเครื่องที่จะ push/pull images:

**บน Web Server และเครื่องอื่นๆ ที่จะใช้งาน Registry:**

สร้างหรือแก้ไขไฟล์ `/etc/docker/daemon.json`:

```json
{
  "insecure-registries": ["192.168.254.128:5000"]
}
```

จากนั้น restart Docker:

```bash
sudo systemctl restart docker
```

**สำคัญ:** ต้องทำทั้งบน GitLab Server และ Web Server

---

## ส่วนที่ 11: การเชื่อมต่อ GitLab กับ Docker Registry

ใน GitLab CI/CD สามารถใช้ Registry นี้ได้โดยการตั้งค่าใน `.gitlab-ci.yml`:

```yaml
stages:
  - build
  - deploy

variables:
  DOCKER_HOST: unix:///var/run/docker.sock
  DOCKER_DRIVER: overlay2
  REGISTRY: 100.100.7.32:5000 #ใส่ ip เเละ port ของ registry server

# ================= BUILD =================
build:
  stage: build
  image: docker:latest
  tags:
    - deploy
  script:
    - docker build -t $REGISTRY/backend:latest ./backend
    - docker push $REGISTRY/backend:latest
    - docker build -t $REGISTRY/frontend:latest ./frontend
    - docker push $REGISTRY/frontend:latest
  only:
    - main

# ================= DEPLOY =================
deploy:
  stage: deploy
  image: docker:25.0-cli
  tags:
    - deploy
  script:
    - echo "REGISTRY = $REGISTRY"   # ✔ ดูค่าจริง
    - docker compose pull
    - docker compose up -d
  only:
    - main

```

---

## ส่วนที่ 12: หมายเหตุสำคัญ

**สำหรับ GitLab (192.168.254.128):**
- GitLab ทำงานบน port 8000
- SSH ทำงานบน port 2200
- HTTPS ทำงานบน port 4430

**สำหรับ Docker Registry (192.168.254.128 - เครื่องเดียวกับ GitLab):**
- Registry ทำงานบน port 5000
- Registry UI ทำงานบน port 8080
- ข้อมูล images จะถูกเก็บในโฟลเดอร์ `./registry/data`
- เปิดใช้งาน DELETE_IMAGES เพื่อให้สามารถลบ images ผ่าน UI ได้

**สำหรับ GitLab Runner (192.168.254.129):**
- Runner ต้องมีสิทธิ์เข้าถึง Docker socket
- **ต้องใช้ privileged mode** (`privileged: true`)
- **ต้อง bind mount Docker CLI และ plugins**
- **User ต้องอยู่ในกลุ่ม docker**
- ใช้ tag `deploy` ในการระบุ runner ที่จะใช้งาน
- Runner จะดึง Docker images มาใช้งานตามที่กำหนด

**สรุป Port ที่ใช้งานบน GitLab Server (192.168.254.128):**
- 8000 - GitLab Web UI
- 4430 - GitLab HTTPS
- 2200 - GitLab SSH
- 5000 - Docker Registry API
- 8080 - Registry UI

---

## ส่วนที่ 13: การจัดการ Registry ผ่าน UI

Registry UI ให้คุณสามารถ:
- ดูรายการ Docker images ทั้งหมด
- ดู tags และรายละเอียดของแต่ละ image
- ลบ images หรือ tags ที่ไม่ต้องการ
- ดู metadata และ layers ของ image

**เข้าถึงผ่าน:** `http://192.168.254.128:8080`

---

## ส่วนที่ 14: คำสั่งที่เป็นประโยชน์

### ดู logs ของ services (บน GitLab Server)

```bash
# ดู logs ของ GitLab
sudo docker logs gitlab

# ดู logs ของ Registry
sudo docker logs docker-registry

# ดู logs ของ Registry UI
sudo docker logs registry-ui
```

### ดู logs ของ GitLab Runner (บน Web Server)

```bash
# ดู logs ของ GitLab Runner
sudo docker logs gitlab-runner
```

### หยุดและเริ่มต้น services (บน GitLab Server)

```bash
# หยุด services ทั้งหมด
sudo docker compose down

# เริ่มต้น services ทั้งหมด
sudo docker compose up -d

# Restart service ใดๆ
sudo docker restart gitlab
sudo docker restart docker-registry
sudo docker restart registry-ui
```

### ทำความสะอาด Registry

```bash
# เข้าไปใน container
sudo docker exec -it docker-registry sh

# รัน garbage collection เพื่อลบ unused layers
registry garbage-collect /etc/docker/registry/config.yml
```

---

## ส่วนที่ 15: สถาปัตยกรรมระบบ

```
┌─────────────────────────────────────────────────────────┐
│  GitLab Server (192.168.254.128)                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   GitLab     │  │   Registry   │  │ Registry UI  │  │
│  │   :8000      │  │    :5000     │  │    :8080     │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
                           │
                           │ Network
                           │
┌─────────────────────────────────────────────────────────┐
│  Web Server (192.168.254.129)                           │
│  ┌──────────────────────────────────────────────────┐   │
│  │            GitLab Runner                         │   │
│  │            (tag: deploy)                         │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

# 🎉 จบขั้นตอนการติดตั้ง

### หลังจากนำ Registration Token จาก Project ใน GitLab ไปลงทะเบียนใน GitLab Runner ก็สามารถสร้างไฟล์ `.gitlab-ci.yml` ไว้ใน project เพื่อทำ pipeline ได้เลย

**สิ่งที่คุณมีตอนนี้:**
- ✅ GitLab Server พร้อมใช้งาน (192.168.254.128)
- ✅ Docker Registry พร้อมเก็บ images (192.168.254.128:5000)
- ✅ Registry UI สำหรับจัดการ images (192.168.254.128:8080)
- ✅ GitLab Runner ลงทะเบียนสำเร็จ (192.168.254.129)
- ✅ CI/CD Pipeline พร้อมใช้งาน

**ข้อดีของการรวม Registry กับ GitLab:**
- ใช้ Server เพียงเครื่องเดียวสำหรับ GitLab และ Registry
- ประหยัด Resource
- จัดการง่ายขึ้น
- Network ภายในเดียวกัน (gitlab-network)
