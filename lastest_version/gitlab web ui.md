# การใช้งานเเละตั้งค่า repository เพื่อให้พร้อมทำ ci/cd
**เป้าหมายคู่มือนี้**
 1.  สร้าง user สำหรับใช้งาน gitlab ด้วย root
 2. สร้างโปรเจคสำหรับทำการ Dev
 3. นำ runner token บน project ไป registration บน gitlab runner server เพื่อลิ้งตัว project กับ runner
 4. ต่างค่า CI/CD variables เพื่อจำลอง .env เพราะ เมื่อเรา commit code ใน project ขึ้น gitlab จะไม่นำ .env ไฟล์ขึ้นมาด้วยเพื่อความปลอดภัย
## ขั้นตอนที่ 1   สร้าง user สำหรับใช้งาน gitlab ด้วย root
 Login root เเล้วไปที่ช่อง search
 
![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/login%20root.png?raw=true)
 ___
 ค้นหา Admin area เพื่อเข้าใช้ Admin dashboard
![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/search%20admin%20area.png?raw=true)
___
เลือก sidebar ชื่อ Users จะเเสดงรายชื่อผู้ใช้ทั้งหมด เเละ กดปุ่ม New user

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/add%20user.png?raw=true)
___
กรอก username เเละ email ที่ต้องการใช้งาน(อีเมล์ใส่อะไรก็ได้) เลื่อนลงไปกด เมื่อกรอกเสร็จเเล้วเลื่อนไปกด create

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/input%20info%20user.png?raw=true)
___
เมื่อสร้างสำเร็จจะได้ผู้ใช้มา หลังจากนี้ให้กดไปที่ Edit เพื่อทำการตั้งรหัสผู้ใช้งานเพื่อเข้าใช้งานครั้งเเรก

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/finish%20create%20user.png?raw=true)
___
กรอกรหัสที่ต้องการ เเล้ว save change

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/edit%20pass.png?raw=true)

**หลังจากตั้งรหัสให้ User ที่สร้างใหม่เเล้วสามารถเข้าใช้งานได้เลย**

เมื่อเข้าใช้งานผู้ใช้ที่ Admin สร้างให้ครั้งเเรกจะให้เราตั้งรหัสผ่านใหม่ ตั้งรหัสเดิมก็ได้ หลังจากตั้งรหัสผ่านใหม่ กด update password จะเด้งไปหน้า login ให้ทำการ login ด้วยรหัสผ่านที่ตั้งใหม่

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/user%20login%20with%20admin%20password.png?raw=true)

เมื่อ login สำเร็จจะมาอยู่หน้า Dashboard ของผู้ใช้

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/new%20user%20pass%20success.png?raw=true)
___
## ขั้นตอนที่ 2 สร้างโปรเจคสำหรับทำการ Dev
ทำการ login เข้า user ที่สมัครไว้ตอนนี้จะอยู่ที่หน้า Dashboard เลือก sidebar ด้านซ้ายที่เมนู Projects เเละเลือก Create a project เมื่อเลือก create project จะเเสดงเมนูต่างๆขึ้นมา ให้เลือก create blank project

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/create%20project.png?raw=true)

เมื่อเลือก create blank project จะเเสดงหน้าต่างนี้ขึ้นมา ให้กรอก project name เเละเลือก Project URL (สามารถสร้าง group ขึ้นมาเเล้วใส่ตรงนี้ได้) ตอนนี้ใช้คนเดียวก็เลือกเป็น username ตัวเอง เเละเลือกเป็น internal หากตั้งค่าหมดเเล้วให้กด create project

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/setting%20project.png?raw=true)

หลังจากสร้างโปรเจคจะเเสดงหน้า project dashboard 
![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/project%20dashboard.png?raw=true)

เลือกไปที่ ปุ่ม code สีน้ำเงินจะเเสดง url สำหรับ clone project นำ url นี้ไปใช้สำหรับ git clone ได้เลย สมารถใช้ได้ทั้ง ssh เเละ http

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/URL%20clone%20project.png?raw=true)

**ขั้นตอนการ clone เเละ ตั้งค่า git ดูได้ที่ไฟล์ git user manual .md**

## ขั้นตอนที่ 3 นำ runner token บน project ไป registration บน gitlab runner server เพื่อลิ้งตัว project กับ runner
**หมายเหตุ**
Token ที่จะได้จากคู่มือต่อไปนี้นำไปใช้ใน คู่มือ gitlab and gitlab runner lastest .md

> ขั้นตอน: ลงทะเบียน Runner กับ GitLab

เริ่มต้นอยู่ที่ project ของเรา เเละไปที่ sidebar ที่ เมนู setting>CI/CD
![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/sidebar%20token%20.png?raw=true)

เลื่อนไปหา เมนู runner เมื่อกดไปที่ runner จะเเสดง dropdown menu ออกมา ส่วนสำคัญเราต้องไปกด ที่ 3 จุด ข้างปุ่ม create project runner จะเเสดง Registration token ให้เราทำการ Copy เเละนำไปลงทะเบียน บน gitlb runner server ของเรา เพื่อให้ตัว runner สามารถเข้ามาที่ project นี้ได้

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/runner%20token.png?raw=true)

## การดู pipeline 
ไปที่ sidebar menu build จะมี หัวข้อ pipeline เมื่อมีการสร้างไฟล์ .gitlab-ci.yml หรือ commit ไฟล์ .gitlab-ci.yml ไปที่ main จะ trigger เเละอ่าน ci/cd อัติโนมัติ

![enter image description here](https://github.com/KewVy8016/git-lab_installation/blob/master/image/pipeline%20.png?raw=true)
