# คู่มือการสร้าง Repository และใช้งาน Git กับ GitLab

## สารบัญ
1. [การสร้าง Repository](#1-การสร้าง-repository)
2. [การสร้างและเพิ่ม SSH Key](#2-การสร้างและเพิ่ม-ssh-key)
3. [การใช้งาน Git พื้นฐาน](#3-การใช้งาน-git-พื้นฐาน)
4. [Workflow การทำงานกับ Branch (Test → Main)](#4-workflow-การทำงานกับ-branch-test--main)
5. [การเพิ่มสมาชิกและ SSH Key](#5-การเพิ่มสมาชิกและ-ssh-key)
6. [แก้ปัญหาที่พบบ่อย](#6-แก้ปัญหาที่พบบ่อย)

---

## 1. การสร้าง Repository

### 1.1 เข้าสู่ระบบ GitLab
1. เปิดเว็บเบราว์เซอร์และเข้าสู่ `http://<IP-Address-ของคุณ>`
2. เข้าสู่ระบบด้วยบัญชี root หรือบัญชีผู้ใช้ของคุณ

### 1.2 สร้าง Repository ใหม่
1. คลิกปุ่ม **"New project"** หรือ **"+"** → **"New project/repository"**
2. เลือก **"Create blank project"**
3. กรอกข้อมูลโปรเจค:
   - **Project name**: ชื่อโปรเจคของคุณ (เช่น `my-project`)
   - **Project slug**: จะถูกสร้างอัตโนมัติจากชื่อโปรเจค
   - **Visibility Level**: 
     - **Private**: เฉพาะสมาชิกที่ได้รับเชิญเท่านั้น ✅ แนะนำ
     - **Internal**: ผู้ใช้ที่ล็อกอินเข้าระบบทุกคน
     - **Public**: ทุกคนสามารถเข้าถึงได้
   - ✅ **Initialize repository with a README**: เลือกเพื่อสร้างไฟล์ README.md
4. คลิก **"Create project"**

---

## 2. การสร้างและเพิ่ม SSH Key

### 2.1 ตรวจสอบ SSH Key ที่มีอยู่

```bash
ls -al ~/.ssh
```

หาไฟล์: `id_rsa.pub`, `id_ecdsa.pub`, หรือ `id_ed25519.pub`

### 2.2 สร้าง SSH Key ใหม่

**Linux/Mac/Windows (Git Bash):**
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

**หมายเหตุ**: หากระบบไม่รองรับ ed25519 ใช้ RSA:
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**ขั้นตอนการสร้าง:**
1. กด **Enter** เพื่อใช้ตำแหน่งเริ่มต้น
2. ใส่ passphrase (หรือปล่อยว่าง) และกด **Enter**
3. ยืนยัน passphrase อีกครั้ง

### 2.3 คัดลอก SSH Public Key

**Linux/Mac:**
```bash
cat ~/.ssh/id_ed25519.pub
```

**Windows (PowerShell):**
```bash
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

**Windows (Git Bash):**
```bash
cat ~/.ssh/id_ed25519.pub
```

คัดลอกผลลัพธ์ทั้งหมด (เริ่มต้นด้วย `ssh-ed25519` หรือ `ssh-rsa`)

### 2.4 เพิ่ม SSH Key เข้า GitLab

1. คลิกรูปโปรไฟล์มุมขวาบน → **"Preferences"** หรือ **"Settings"**
2. เลือก **"SSH Keys"** ทางด้านซ้าย
3. วาง Public Key ในช่อง **"Key"**
4. กรอก **"Title"**: เช่น `My Laptop`, `Work PC`
5. ตั้งค่า **"Expiration date"**: (ไม่บังคับ)
6. คลิก **"Add key"**

### 2.5 ทดสอบการเชื่อมต่อ

```bash
ssh -T git@<IP-Address-ของคุณ>
```

✅ **ผลลัพธ์ที่ถูกต้อง:**
```
Welcome to GitLab, @username!
```

---

## 3. การใช้งาน Git พื้นฐาน

### 3.1 Clone Repository

1. ไปที่หน้าโปรเจคใน GitLab
2. คลิก **"Clone"** → คัดลอก URL ในส่วน **"Clone with SSH"**

```bash
git clone git@<IP-Address>:<username>/<project-name>.git
```
เเนะนำให้ใช้ ssh เพราะ เราเปลี่ยน port external ในที่นี้ 2200:22 
```bash
git clone ssh://git@192.168.8.136:2200/dev_team/teacher-evaluation-system.git
```

**ตัวอย่าง:**
```bash
git clone git@192.168.1.100:root/my-project.git
```

### 3.2 ตั้งค่า Git Config (ครั้งแรก)

```bash
cd my-project

# ตั้งค่าเฉพาะ project นี้
git config user.name "Your Name"
git config user.email "your_email@example.com"

# หรือตั้งค่าแบบ global (ใช้ได้ทุก project)
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

### 3.3 ตรวจสอบการตั้งค่า

```bash
# ดู config ทั้งหมด
git config --list

# ดูเฉพาะ user
git config user.name
git config user.email
```

---

## 4. Workflow การทำงานกับ Branch (Test → Main)

### 4.1 ดู Branch ปัจจุบัน

```bash
# ดู branch ที่มีทั้งหมด
git branch -a

# ดู branch ปัจจุบัน
git branch
```

### 4.2 สร้าง Branch Test

```bash
# สร้าง branch ใหม่ชื่อ test และสลับไปที่ branch นั้น
git checkout -b test

# หรือแยกเป็น 2 คำสั่ง
git branch test        # สร้าง branch
git checkout test      # สลับไป branch test
```

✅ **ตอนนี้คุณอยู่ใน branch test แล้ว**

### 4.3 ทำงานใน Branch Test

#### 4.3.1 สร้างหรือแก้ไขไฟล์

```bash
# สร้างไฟล์ใหม่
echo "# My Project" > README.md
echo "console.log('Hello World');" > app.js

# หรือแก้ไขไฟล์ที่มีอยู่ด้วย editor
nano README.md
# หรือ
vim README.md
# หรือ
code README.md  # VS Code
```

#### 4.3.2 ตรวจสอบสถานะ

```bash
git status
```

**ผลลัพธ์:**
```
On branch test
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        README.md
        app.js

nothing added to commit but untracked files present (use "git add" to track)
```

#### 4.3.3 เพิ่มไฟล์เข้า Staging Area

```bash
# เพิ่มไฟล์ทีละไฟล์
git add README.md
git add app.js

# หรือเพิ่มทั้งหมด
git add .
```

#### 4.3.4 ตรวจสอบสถานะอีกครั้ง

```bash
git status
```

**ผลลัพธ์:**
```
On branch test
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   README.md
        new file:   app.js
```

#### 4.3.5 Commit การเปลี่ยนแปลง

```bash
git commit -m "Add README and app.js files"
```

**ผลลัพธ์:**
```
[test abc1234] Add README and app.js files
 2 files changed, 2 insertions(+)
 create mode 100644 README.md
 create mode 100644 app.js
```

#### 4.3.6 Push Branch Test ไปยัง GitLab

```bash
# Push branch test ครั้งแรก
git push -u origin test

# หรือ push ครั้งต่อๆ ไป
git push origin test
```

### 4.4 ตรวจสอบและทดสอบใน Branch Test

```bash
# ดูประวัติ commit
git log

# ดูความแตกต่างระหว่างไฟล์
git diff

# ดูสถานะปัจจุบัน
git status
```

### 4.5 Merge Branch Test เข้า Main

#### 4.5.1 สลับไปยัง Branch Main

```bash
git checkout main
```

✅ **ตอนนี้คุณอยู่ใน branch main แล้ว**

#### 4.5.2 ดึงข้อมูลล่าสุดจาก GitLab

```bash
git pull origin main
```

#### 4.5.3 Merge Branch Test เข้า Main

```bash
git merge test
```

**ผลลัพธ์ (ถ้าไม่มี conflict):**
```
Updating abc1234..def5678
Fast-forward
 README.md | 1 +
 app.js    | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 README.md
 create mode 100644 app.js
```

#### 4.5.4 Push Main ไปยัง GitLab

```bash
git push origin main
```

✅ **สำเร็จ! โค้ดจาก branch test ถูก merge เข้า main แล้ว**

### 4.6 ลบ Branch Test (ถ้าไม่ใช้แล้ว)

```bash
# ลบ branch test ในเครื่อง
git branch -d test

# ลบ branch test บน GitLab
git push origin --delete test
```

---

## 5. Workflow แบบเต็ม: ตั้งแต่เริ่มต้นจนเสร็จ

### 📋 สถานการณ์: พัฒนา Feature ใหม่

```bash
# 1. Clone โปรเจค (ครั้งแรก)
git clone git@192.168.1.100:root/my-project.git
cd my-project

# 2. ตั้งค่า Git Config
git config user.name "John Doe"
git config user.email "john@example.com"

# 3. ตรวจสอบ branch ปัจจุบัน (ควรอยู่ที่ main)
git branch

# 4. สร้าง branch test สำหรับพัฒนา feature ใหม่
git checkout -b test

# 5. เริ่มเขียนโค้ด
echo "console.log('New feature');" > feature.js
mkdir src
echo "function hello() { return 'Hello'; }" > src/utils.js

# 6. ตรวจสอบไฟล์ที่เปลี่ยนแปลง
git status

# 7. เพิ่มไฟล์ทั้งหมด
git add .

# 8. Commit การเปลี่ยนแปลง
git commit -m "Add new feature and utils function"

# 9. Push branch test ไปยัง GitLab
git push -u origin test

# 10. เขียนโค้ดต่อ และแก้ไขไฟล์
echo "console.log('More features');" >> feature.js

# 11. ดูความแตกต่าง
git diff feature.js

# 12. Add และ commit อีกครั้ง
git add feature.js
git commit -m "Add more features to feature.js"

# 13. Push การเปลี่ยนแปลงใหม่
git push origin test

# 14. เมื่อทดสอบเสร็จแล้ว พร้อม merge เข้า main
# สลับไป branch main
git checkout main

# 15. ดึงข้อมูลล่าสุดจาก GitLab
git pull origin main

# 16. Merge branch test เข้า main
git merge test

# 17. Push main ไปยัง GitLab
git push origin main

# 18. ลบ branch test (ถ้าไม่ใช้แล้ว)
git branch -d test
git push origin --delete test

# 19. ตรวจสอบว่า merge สำเร็จ
git log --oneline --graph
```

---

## 6. การทำงานกับ Merge Request (แนะนำ)

แทนที่จะ merge ด้วยคำสั่ง สามารถใช้ Merge Request บน GitLab Web UI:

### 6.1 สร้าง Merge Request

1. Push branch test ไปยัง GitLab:
   ```bash
   git push -u origin test
   ```

2. เปิดเว็บ GitLab → ไปที่โปรเจค
3. จะมีปุ่ม **"Create merge request"** ขึ้นมา (หรือไปที่ **Merge requests** → **"New merge request"**)
4. เลือก:
   - **Source branch**: `test`
   - **Target branch**: `main`
5. กรอกข้อมูล:
   - **Title**: เช่น "Add new feature"
   - **Description**: อธิบายการเปลี่ยนแปลง
   - **Assignee**: เลือกคนที่จะ review
   - **Reviewer**: เลือกคนที่จะตรวจสอบโค้ด
6. คลิก **"Create merge request"**

### 6.2 Review และ Approve

1. Reviewer ตรวจสอบโค้ดใน **"Changes"** tab
2. แสดงความคิดเห็นหรือขอแก้ไข
3. กด **"Approve"** เมื่อโค้ดผ่าน

### 6.3 Merge

1. กด **"Merge"** เมื่อ Merge Request ได้รับการ approve
2. เลือก **"Delete source branch"** เพื่อลบ branch test หลัง merge
3. คลิก **"Merge"**

✅ **โค้ดถูก merge เข้า main แล้ว!**

---

## 7. การเพิ่มสมาชิกและ SSH Key

### 7.1 เชิญสมาชิกเข้าโปรเจค (Owner/Maintainer)

1. ไปที่โปรเจค → **"Settings"** → **"Members"**
2. คลิก **"Invite members"**
3. กรอกข้อมูล:
   - **Username or email**: ชื่อผู้ใช้หรืออีเมล
   - **Role**: 
     - **Guest**: อ่านได้เท่านั้น
     - **Reporter**: อ่าน + สร้าง issue
     - **Developer**: อ่าน + เขียน + push code ✅ แนะนำ
     - **Maintainer**: สิทธิ์จัดการเกือบทั้งหมด
     - **Owner**: สิทธิ์สูงสุด
4. คลิก **"Invite"**

### 7.2 สมาชิกคนอื่นเพิ่ม SSH Key

**แต่ละคนต้องทำเอง:**

#### 7.2.1 สร้าง SSH Key

```bash
ssh-keygen -t ed25519 -C "member_email@example.com"
```

#### 7.2.2 คัดลอก Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

#### 7.2.3 เพิ่ม SSH Key เข้า GitLab

1. เข้าสู่ระบบ GitLab ด้วยบัญชีของตัวเอง
2. รูปโปรไฟล์ → **"Preferences"** → **"SSH Keys"**
3. วาง Public Key → ตั้งชื่อ → **"Add key"**

#### 7.2.4 ทดสอบการเชื่อมต่อ

```bash
ssh -T git@<IP-Address>
```

#### 7.2.5 Clone โปรเจค

```bash
git clone git@<IP-Address>:<username>/<project-name>.git
cd project-name
```

#### 7.2.6 ตั้งค่า Git Config

```bash
git config user.name "Member Name"
git config user.email "member_email@example.com"
```

✅ **พร้อมทำงานแล้ว!**

### 7.3 ⚠️ ข้อควรระวัง

- **แต่ละคนต้องใช้ SSH Key ของตัวเอง** - ห้ามแชร์ Private Key!
- **Private Key** (`id_ed25519`) **ต้องเก็บไว้เป็นความลับ**
- เฉพาะ **Public Key** (`id_ed25519.pub`) ที่เพิ่มเข้า GitLab
- แต่ละคนสามารถมีหลาย SSH Key สำหรับหลายเครื่อง

---

## 8. แก้ปัญหาที่พบบ่อย

### 🔧 ปัญหา 1: Permission denied (publickey)

**สาเหตุ:** SSH Key ไม่ถูกต้องหรือยังไม่ได้เพิ่มเข้า GitLab

**วิธีแก้:**
```bash
# 1. ตรวจสอบ SSH Key
ls -al ~/.ssh

# 2. ทดสอบการเชื่อมต่อ
ssh -T git@<IP-Address>

# 3. เพิ่ม SSH Key เข้า SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 4. ทดสอบอีกครั้ง
ssh -T git@<IP-Address>
```

### 🔧 ปัญหา 2: Author identity unknown

**สาเหตุ:** ยังไม่ได้ตั้งค่า Git user

**วิธีแก้:**
```bash
git config user.name "Your Name"
git config user.email "your_email@example.com"

# ตรวจสอบ
git config --list | grep user
```

### 🔧 ปัญหา 3: Merge conflict

**สาเหตุ:** มีคนแก้ไขไฟล์เดียวกันพร้อมกัน

**วิธีแก้:**
```bash
# 1. พยายาม merge
git merge test

# 2. เห็นข้อความ CONFLICT
# CONFLICT (content): Merge conflict in file.js

# 3. เปิดไฟล์ที่มี conflict
nano file.js

# 4. จะเห็น marker
# <<<<<<< HEAD
# โค้ดใน main
# =======
# โค้ดใน test
# >>>>>>> test

# 5. แก้ไขเลือกโค้ดที่ถูกต้อง แล้วลบ marker ออก

# 6. Add และ commit
git add file.js
git commit -m "Resolve merge conflict"

# 7. Push
git push origin main
```

### 🔧 ปัญหา 4: Fatal: Not a git repository

**สาเหตุ:** ไม่ได้อยู่ในโฟลเดอร์ Git

**วิธีแก้:**
```bash
# ตรวจสอบว่าอยู่ในโฟลเดอร์ที่ถูกต้อง
pwd
cd /path/to/your/project

# หรือ clone ใหม่
git clone git@<IP-Address>:<username>/<project-name>.git
```

### 🔧 ปัญหา 5: Push rejected

**สาเหตุ:** Remote มีการเปลี่ยนแปลงที่คุณยังไม่มี

**วิธีแก้:**
```bash
# 1. Pull ข้อมูลล่าสุดก่อน
git pull origin main

# 2. แก้ conflict (ถ้ามี)

# 3. Push อีกครั้ง
git push origin main
```

---

## 9. คำสั่ง Git ที่ควรรู้

### 📌 คำสั่งพื้นฐาน

```bash
# สถานะปัจจุบัน
git status

# ประวัติ commit
git log
git log --oneline
git log --oneline --graph --all

# ดูความแตกต่าง
git diff
git diff file.js
git diff main test

# ย้อนกลับการเปลี่ยนแปลง
git checkout -- file.js    # ย้อนกลับไฟล์ที่ยังไม่ add
git reset HEAD file.js     # เอาออกจาก staging area
git reset --hard HEAD      # ย้อนกลับทุกอย่างไปยัง commit ล่าสุด
```

### 📌 การจัดการ Branch

```bash
# ดู branch
git branch              # branch ในเครื่อง
git branch -a           # ทุก branch
git branch -r           # branch บน remote

# สร้างและสลับ branch
git checkout -b new-branch

# สลับ branch
git checkout main

# ลบ branch
git branch -d branch-name
git push origin --delete branch-name

# เปลี่ยนชื่อ branch
git branch -m old-name new-name
```

### 📌 Remote Repository

```bash
# ดู remote
git remote -v

# เพิ่ม remote
git remote add origin git@gitlab.com:user/repo.git

# เปลี่ยน URL remote
git remote set-url origin git@new-gitlab.com:user/repo.git

# ลบ remote
git remote remove origin
```

### 📌 การย้อนกลับ

```bash
# ย้อนกลับไปยัง commit ก่อนหน้า
git revert HEAD

# ย้อนกลับไปยัง commit เฉพาะ
git revert abc1234

# รีเซ็ตไปยัง commit ก่อนหน้า (ระวัง! จะลบ commit)
git reset --hard HEAD~1
```

---

## 10. Best Practices

### ✅ ควรทำ

1. **Commit บ่อยๆ** - แต่ละ commit ควรมีความหมายชัดเจน
2. **เขียน commit message ที่ดี** - อธิบายว่าทำอะไรและทำไม
3. **Pull ก่อน Push เสมอ** - เพื่อหลีกเลี่ยง conflict
4. **ใช้ branch แยกตาม feature** - อย่าทำงานใน main โดยตรง
5. **Review code ก่อน merge** - ใช้ Merge Request
6. **ลบ branch ที่ไม่ใช้แล้ว** - เพื่อความเป็นระเบียบ
7. **Backup code สำคัญ** - Push ขึ้น GitLab เป็นประจำ

### ❌ ไม่ควรทำ

1. **อย่า commit รหัสผ่าน/API key** - ใช้ `.gitignore`
2. **อย่า force push** (`git push -f`) - เว้นแต่จำเป็นมาก
3. **อย่าแชร์ Private Key** - แต่ละคนต้องมี SSH Key ของตัวเอง
4. **อย่า commit file ขนาดใหญ่** - เช่น video, database dump
5. **อย่า commit code ที่ยังไม่ทำงาน** - ใน branch หลัก

---

## 11. ไฟล์ .gitignore

สร้างไฟล์ `.gitignore` เพื่อไม่ให้ Git track ไฟล์บางประเภท:

```bash
# สร้างไฟล์ .gitignore
nano .gitignore
```

**ตัวอย่างเนื้อหา:**
```
# Node.js
node_modules/
npm-debug.log
package-lock.json

# Python
__pycache__/
*.py[cod]
venv/
.env

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Secrets
.env
config/secrets.yml
*.key
*.pem

# Build
dist/
build/
*.log
```

**เพิ่มเข้า Git:**
```bash
git add .gitignore
git commit -m "Add .gitignore file"
git push origin main
```

---

## 12. สรุป Workflow ทั้งหมด

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Clone Repository                                         │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│ 7. Merge เข้า Main                                          │
│    git checkout main                                        │
│    git pull origin main                                     │
│    git merge test                                           │
│    git push origin main                                     │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│ 8. ลบ Branch Test (ถ้าเสร็จแล้ว)                            │
│    git branch -d test                                       │
│    git push origin --delete test                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 13. ตัวอย่างการทำงานจริง (Real-world Example)

### สถานการณ์: พัฒนา Login Feature

```bash
# ============================================
# DAY 1: เริ่มพัฒนา Feature
# ============================================

# 1. Clone โปรเจค (ถ้ายังไม่มี)
git clone git@192.168.1.100:root/my-app.git
cd my-app

# 2. ตรวจสอบว่าอยู่ที่ main
git branch
# * main

# 3. Update main ให้เป็นเวอร์ชันล่าสุด
git pull origin main

# 4. สร้าง branch สำหรับ login feature
git checkout -b feature/login
# Switched to a new branch 'feature/login'

# 5. สร้างไฟล์ login
mkdir src/auth
touch src/auth/login.js
touch src/auth/login.test.js

# 6. เขียนโค้ด
cat > src/auth/login.js << 'EOF'
export const login = (username, password) => {
  if (!username || !password) {
    throw new Error('Username and password required');
  }
  // Login logic here
  return { token: 'abc123', userId: 1 };
};
EOF

# 7. ตรวจสอบสถานะ
git status
# On branch feature/login
# Untracked files:
#   src/auth/

# 8. Add และ commit
git add src/auth/
git commit -m "feat: add login function with validation"

# 9. Push branch ครั้งแรก
git push -u origin feature/login
# Branch 'feature/login' set up to track remote branch 'feature/login' from 'origin'.


# ============================================
# DAY 2: เพิ่ม Unit Tests
# ============================================

# 10. เขียน test
cat > src/auth/login.test.js << 'EOF'
import { login } from './login';

describe('login', () => {
  it('should return token on valid credentials', () => {
    const result = login('user', 'pass');
    expect(result.token).toBeDefined();
  });

  it('should throw error on missing credentials', () => {
    expect(() => login('', '')).toThrow();
  });
});
EOF

# 11. ตรวจสอบการเปลี่ยนแปลง
git status
# Modified: src/auth/login.test.js

# 12. ดูความแตกต่าง
git diff src/auth/login.test.js

# 13. Add และ commit
git add src/auth/login.test.js
git commit -m "test: add unit tests for login function"

# 14. Push
git push origin feature/login


# ============================================
# DAY 3: แก้ไข Bug และเพิ่ม Error Handling
# ============================================

# 15. แก้ไขโค้ด
nano src/auth/login.js
# (เพิ่ม try-catch และ validation)

# 16. ดูการเปลี่ยนแปลง
git diff

# 17. Commit
git add src/auth/login.js
git commit -m "fix: improve error handling in login function"

# 18. Push
git push origin feature/login

# 19. ดูประวัติ commit
git log --oneline
# abc123 fix: improve error handling in login function
# def456 test: add unit tests for login function
# ghi789 feat: add login function with validation


# ============================================
# DAY 4: พร้อม Merge เข้า Main
# ============================================

# 20. Update branch ล่าสุดจาก main (ป้องกัน conflict)
git checkout main
git pull origin main

# 21. กลับไป branch feature
git checkout feature/login

# 22. Merge main เข้า feature/login (เพื่อ update)
git merge main
# Already up to date. (ถ้าไม่มี conflict)

# 23. ทดสอบครั้งสุดท้าย
npm test
# ✓ All tests passed

# 24. Push เพื่อ update
git push origin feature/login


# ============================================
# วิธีที่ 1: Merge ผ่าน Command Line
# ============================================

# 25a. สลับไป main
git checkout main

# 26a. Merge feature เข้า main
git merge feature/login
# Updating abc123..ghi789
# Fast-forward
#  src/auth/login.js      | 20 ++++++++++++++++++++
#  src/auth/login.test.js | 15 +++++++++++++++
#  2 files changed, 35 insertions(+)

# 27a. Push main
git push origin main

# 28a. ลบ branch feature
git branch -d feature/login
git push origin --delete feature/login


# ============================================
# วิธีที่ 2: Merge ผ่าน GitLab Web (แนะนำ)
# ============================================

# 25b. เปิด GitLab Web → ไปที่โปรเจค
# 26b. คลิก "Create merge request"
# 27b. กรอกข้อมูล:
#      Title: "Add login feature"
#      Description: "
#        ## Changes
#        - Add login function with validation
#        - Add unit tests
#        - Improve error handling
#
#        ## Testing
#        - All unit tests passed
#        - Manual testing completed
#      "
#      Assignee: @reviewer
# 28b. คลิก "Create merge request"
# 29b. รอ reviewer approve
# 30b. คลิก "Merge" เมื่อได้รับการ approve
# 31b. เลือก "Delete source branch"
# 32b. คลิก "Merge"

# ✅ เสร็จสมบูรณ์!
```

---

## 14. Workflow สำหรับทีม (Team Workflow)

### 14.1 Git Flow Strategy

```
main (production)
  │
  ├── develop (development)
  │     │
  │     ├── feature/login
  │     │     └── (Developer A ทำงาน)
  │     │
  │     ├── feature/payment
  │     │     └── (Developer B ทำงาน)
  │     │
  │     └── feature/dashboard
  │           └── (Developer C ทำงาน)
  │
  └── hotfix/critical-bug
        └── (แก้ไข bug ด่วน)
```

### 14.2 ขั้นตอนการทำงานเป็นทีม

```bash
# ============================================
# Developer A: ทำ Login Feature
# ============================================

# 1. Clone และ checkout develop
git clone git@gitlab.com:team/project.git
cd project
git checkout develop
git pull origin develop

# 2. สร้าง feature branch
git checkout -b feature/login

# 3. พัฒนา feature
# (เขียนโค้ด...)

# 4. Commit และ Push
git add .
git commit -m "feat: implement login feature"
git push -u origin feature/login

# 5. สร้าง Merge Request: feature/login → develop
# (บน GitLab Web)


# ============================================
# Developer B: ทำ Payment Feature (พร้อมกัน)
# ============================================

# 1. Clone และ checkout develop
git clone git@gitlab.com:team/project.git
cd project
git checkout develop
git pull origin develop

# 2. สร้าง feature branch
git checkout -b feature/payment

# 3. พัฒนา feature
# (เขียนโค้ด...)

# 4. Commit และ Push
git add .
git commit -m "feat: add payment gateway integration"
git push -u origin feature/payment

# 5. สร้าง Merge Request: feature/payment → develop


# ============================================
# Tech Lead: Review และ Merge
# ============================================

# 1. Review Merge Request ของ Developer A
# 2. Comment หรือขอแก้ไข (ถ้ามี)
# 3. Approve และ Merge เข้า develop
# 4. Repeat สำหรับ Developer B


# ============================================
# Release Manager: Deploy Production
# ============================================

# 1. ตรวจสอบว่า develop พร้อม release
git checkout develop
git pull origin develop

# 2. Merge develop → main
git checkout main
git pull origin main
git merge develop

# 3. Tag version
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin main --tags

# 4. Deploy to production
# (CI/CD pipeline จะทำงานอัตโนมัติ)
```

---

## 15. การแก้ไข Conflict (ละเอียด)

### 15.1 สถานการณ์: 2 คนแก้ไขไฟล์เดียวกัน

```bash
# ============================================
# Developer A
# ============================================
git checkout -b feature/update-header
# แก้ไข header.js บรรทัดที่ 10
# เปลี่ยนจาก: const title = "Old Title";
# เป็น:      const title = "New Title A";
git add header.js
git commit -m "Update header title"
git push origin feature/update-header
# Merge เข้า main


# ============================================
# Developer B (พร้อมกัน)
# ============================================
git checkout -b feature/update-header-style
# แก้ไข header.js บรรทัดที่ 10 (บรรทัดเดียวกัน!)
# เปลี่ยนจาก: const title = "Old Title";
# เป็น:      const title = "New Title B";
git add header.js
git commit -m "Update header style"
git push origin feature/update-header-style

# พยายาม merge เข้า main
git checkout main
git merge feature/update-header-style
# ❌ CONFLICT!


# ============================================
# แก้ไข Conflict
# ============================================

# 1. Git จะบอกว่ามี conflict
# Auto-merging header.js
# CONFLICT (content): Merge conflict in header.js
# Automatic merge failed; fix conflicts and then commit the result.

# 2. เปิดไฟล์ที่มี conflict
nano header.js

# 3. จะเห็น conflict markers:
# <<<<<<< HEAD
# const title = "New Title A";
# =======
# const title = "New Title B";
# >>>>>>> feature/update-header-style

# 4. ตัดสินใจว่าจะเลือกโค้ดไหน:

# ตัวเลือกที่ 1: เลือก A
const title = "New Title A";

# ตัวเลือกที่ 2: เลือก B
const title = "New Title B";

# ตัวเลือกที่ 3: รวมทั้งสอง
const title = "New Title A and B";

# ตัวเลือกที่ 4: เขียนใหม่ทั้งหมด
const title = "Combined New Title";

# 5. ลบ conflict markers (<<<<<<, =======, >>>>>>>)

# 6. บันทึกไฟล์

# 7. Add และ commit
git add header.js
git commit -m "Resolve merge conflict in header.js"

# 8. Push
git push origin main

# ✅ แก้ conflict สำเร็จ!
```

### 15.2 เครื่องมือช่วยแก้ Conflict

```bash
# ใช้ merge tool (เช่น VS Code)
git mergetool

# หรือใช้ GUI tools:
# - GitKraken
# - Sourcetree
# - GitHub Desktop
# - VS Code (built-in)
```

---

## 16. การใช้ Git กับ VS Code

### 16.1 เปิดโปรเจคใน VS Code

```bash
# เปิด VS Code ในโฟลเดอร์ปัจจุบัน
code .
```

### 16.2 การใช้งาน Git ใน VS Code

1. **ดูการเปลี่ยนแปลง**:
   - คลิกไอคอน Source Control (Ctrl+Shift+G)
   - เห็นไฟล์ที่เปลี่ยนแปลงทั้งหมด

2. **Stage ไฟล์**:
   - คลิก + ข้างชื่อไฟล์
   - หรือคลิก + ข้าง "Changes" เพื่อ stage ทั้งหมด

3. **Commit**:
   - พิมพ์ commit message ในช่องด้านบน
   - กด Ctrl+Enter หรือคลิก ✓

4. **Push**:
   - คลิก ... → Push
   - หรือกด Ctrl+Shift+P → "Git: Push"

5. **Pull**:
   - คลิก ... → Pull
   - หรือกด Ctrl+Shift+P → "Git: Pull"

6. **สร้าง Branch**:
   - คลิกชื่อ branch ล่างซ้าย
   - เลือก "Create new branch"
   - ตั้งชื่อ branch

7. **สลับ Branch**:
   - คลิกชื่อ branch ล่างซ้าย
   - เลือก branch ที่ต้องการ

8. **แก้ Conflict**:
   - VS Code จะแสดง conflict พร้อมปุ่ม:
     - Accept Current Change
     - Accept Incoming Change
     - Accept Both Changes
     - Compare Changes

---

## 17. Git Aliases (ลัดคำสั่ง)

### 17.1 ตั้งค่า Aliases

```bash
# Status
git config --global alias.st status

# Add all
git config --global alias.aa 'add .'

# Commit
git config --global alias.cm 'commit -m'

# Push
git config --global alias.ps push

# Pull
git config --global alias.pl pull

# Checkout
git config --global alias.co checkout

# Branch
git config --global alias.br branch

# Log สวยๆ
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# Undo last commit (keep changes)
git config --global alias.undo 'reset HEAD~1 --soft'

# Amend last commit
git config --global alias.amend 'commit --amend --no-edit'
```

### 17.2 ใช้งาน Aliases

```bash
# แทนที่จะพิมพ์
git status
git add .
git commit -m "Update"
git push

# ใช้ aliases
git st
git aa
git cm "Update"
git ps

# ดู log สวยๆ
git lg
```

---

## 18. GitLab CI/CD (Bonus)

### 18.1 สร้างไฟล์ .gitlab-ci.yml

```yaml
# .gitlab-ci.yml

stages:
  - test
  - build
  - deploy

# ทดสอบใน branch test
test:
  stage: test
  script:
    - npm install
    - npm test
  only:
    - test
    - merge_requests

# Build เมื่อ merge เข้า main
build:
  stage: build
  script:
    - npm install
    - npm run build
  only:
    - main
  artifacts:
    paths:
      - dist/

# Deploy production
deploy:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - scp -r dist/* user@server:/var/www/html/
  only:
    - main
  when: manual
```

### 18.2 Workflow กับ CI/CD

```bash
# 1. Push branch test
git push origin test
# → GitLab CI จะรัน tests อัตโนมัติ

# 2. สร้าง Merge Request
# → GitLab CI จะรัน tests อีกครั้ง

# 3. Merge เข้า main
# → GitLab CI จะ build และพร้อม deploy

# 4. Click "Deploy" button บน GitLab
# → Deploy ไปยัง production
```

---

## 19. Cheat Sheet (สรุปคำสั่งสำคัญ)

```bash
# ============================================
# การตั้งค่าเริ่มต้น
# ============================================
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
ssh-keygen -t ed25519 -C "your@email.com"
cat ~/.ssh/id_ed25519.pub  # คัดลอกไปเพิ่มใน GitLab


# ============================================
# เริ่มต้นโปรเจค
# ============================================
git clone git@gitlab.com:user/repo.git
cd repo
git status


# ============================================
# สร้างและทำงานใน Branch
# ============================================
git checkout -b test           # สร้างและสลับไป branch test
git add .                      # เพิ่มไฟล์ทั้งหมด
git commit -m "message"        # commit การเปลี่ยนแปลง
git push -u origin test        # push ครั้งแรก
git push origin test           # push ครั้งต่อไป


# ============================================
# Merge เข้า Main
# ============================================
git checkout main              # สลับไป main
git pull origin main           # update main
git merge test                 # merge test เข้า main
git push origin main           # push main


# ============================================
# ลบ Branch
# ============================================
git branch -d test             # ลบ branch ในเครื่อง
git push origin --delete test  # ลบ branch บน GitLab


# ============================================
# ดูข้อมูล
# ============================================
git status                     # สถานะปัจจุบัน
git log                        # ประวัติ commit
git log --oneline --graph      # ประวัติแบบกราฟ
git branch                     # ดู branch ทั้งหมด
git diff                       # ดูความแตกต่าง
git remote -v                  # ดู remote URL


# ============================================
# ย้อนกลับ
# ============================================
git checkout -- file.js        # ย้อนกลับไฟล์
git reset HEAD file.js         # เอาออกจาก staging
git reset --hard HEAD          # ย้อนกลับทุกอย่าง (ระวัง!)
git revert abc123              # ย้อนกลับ commit เฉพาะ
```

---

## 20. ทรัพยากรเพิ่มเติม

### 📚 Documentation
- **Git Official**: https://git-scm.com/doc
- **GitLab Docs**: https://docs.gitlab.com
- **Atlassian Git Tutorial**: https://www.atlassian.com/git/tutorials

### 🎓 การเรียนรู้
- **Learn Git Branching**: https://learngitbranching.js.org/ (interactive)
- **Git Cheat Sheet**: https://education.github.com/git-cheat-sheet-education.pdf
- **Pro Git Book**: https://git-scm.com/book/en/v2 (ฟรี)

### 🛠️ เครื่องมือ
- **GitKraken**: GUI สำหรับ Git
- **Sourcetree**: GUI อีกตัว
- **VS Code**: มี Git ในตัว
- **GitLens** (VS Code Extension): เพิ่มความสามารถ Git

---

## 21. คำศัพท์ Git ที่ควรรู้

| คำศัพท์ | คำอธิบาย |
|---------|----------|
| **Repository (Repo)** | โฟลเดอร์โปรเจคที่มี Git |
| **Clone** | ดาวน์โหลด repo จาก GitLab |
| **Commit** | บันทึกการเปลี่ยนแปลง |
| **Push** | ส่งข้อมูลไป GitLab |
| **Pull** | ดึงข้อมูลจาก GitLab |
| **Branch** | แขนงแยกจากโค้ดหลัก |
| **Merge** | รวม branch เข้าด้วยกัน |
| **Conflict** | โค้ดขัดแย้งกัน |
| **Staging Area** | พื้นที่เตรียม commit |
| **Remote** | GitLab server |
| **Origin** | ชื่อ remote เริ่มต้น |
| **HEAD** | ตำแหน่งปัจจุบัน |
| **Main/Master** | Branch หลัก |
| **Fast-forward** | Merge แบบเลื่อน pointer |
| **Rebase** | ย้าย commit ไปฐานใหม่ |
| **Stash** | เก็บการเปลี่ยนแปลงชั่วคราว |
| **Tag** | ทำเครื่องหมาย version |
| **Fork** | คัดลอก repo คนอื่น |

---

### 💡 Tips สุดท้าย

> **"Commit often, push frequently, and always pull before you push!"**

- Commit บ่อยๆ เพื่อไม่สูญเสียงาน
- Push ขึ้น GitLab เป็นประจำเพื่อ backup
- Pull ก่อน push เสมอเพื่อหลีกเลี่ยง conflict
- ใช้ branch แยกตาม feature
- เขียน commit message ที่มีความหมาย
- Review code ก่อน merge
- ถามเมื่อไม่แน่ใจ - ดีกว่าทำผิด!
