# **ขั้นตอนการยืนยันตัวตนเข้าใช้งาน git ผ่าน terminal เพื่อทำการ pull หรือ commit  (Git Credential Store)**


# **1) ตั้งชื่อและอีเมลของ Git (จำเป็นสำหรับ commit)**

เปิด Terminal / PowerShell แล้วพิมพ์:
```cmd
git config --global user.name "test" 
git config --global user.email "test@example.com"
```
> ไม่เกี่ยวกับการ login แต่เป็นการบันทึกลง commit ใช้ชื่อกับเมล์ที่สร้างใน gitlab
----------

# **2) เปิด Git Credential Store แบบ plaintext**
```cmd
git config --global credential.helper store
```
อันนี้สำคัญมาก → จะทำให้ Git จำรหัสลงไฟล์:

`C:\Users\<ชื่อ>\.git-credentials` 

----------

# **3) Clone Repo ผ่าน HTTP (จะเริ่มถาม user/pass)**

ตัวอย่าง:

`git clone http://192.168.1.10/myproject.git`


## วิธีการลบเพื่อยืนยันตัวตนใหม่

# เเนะนำขั้นตอนเพื่อเริ่มใหม่แบบสะอาด

1.  ถ้าอยากล้าง helper เก่า:

`git config --global --unset credential.helper`

`git config --global --list `
คำสั่งเเสดง credential ที่มีอยู๋ถ้าลงทะเบียนใหม่เเล้วจะเเสดง username เเละ email

##  **วิธีล้างแบบ 1: ลบไฟล์ทิ้งเลย**

1.  เปิด Explorer แล้วไปที่:
    
    `C:\Users\<ชื่อ>\` 
    
2.  ลบไฟล์:
    
    `.git-credentials` 
    

ลบปุ๊บ Git จะลืม password ทันที  
ครั้งต่อไป push → จะถาม username/password ใหม่

----------

# **วิธีล้างแบบ 2: ใช้คำสั่ง Git**

รันคำสั่งนี้เพื่อล้างค่า credential ทั้งหมดที่อยู่ใน credential store:

`git credential-cache exit` 

จากนั้นลบไฟล์:

`del %USERPROFILE%\.git-credentials`


