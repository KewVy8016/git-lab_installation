# GitLab CE & GitLab Runner - Setup Guide

คู่มือการติดตั้งและกำหนดค่า GitLab Community Edition พร้อม GitLab Runner บน Docker Compose สำหรับการทำ CI/CD

---

## เอกสารคู่มือ

Repository นี้ประกอบด้วยคู่มือการติดตั้ง 2 ระดับ ให้เลือกใช้ตามความเหมาะสม

### [`gitlab-setup-guide.md`](gitlab-setup-guide.md)

**กลุ่มเป้าหมาย:** ผู้เริ่มต้นใช้งาน, ทีมพัฒนาขนาดเล็ก, สภาพแวดล้อม Development/Testing

**เนื้อหาที่ครอบคลุม:**
- ขั้นตอนการติดตั้งแบบกระชับ
- การตั้งค่า Docker Compose พื้นฐาน
- การกำหนดรหัสผ่านผู้ดูแลระบบ
- การลงทะเบียน GitLab Runner
- การแก้ไขปัญหาเบื้องต้น

**ระยะเวลาในการติดตั้ง:** ประมาณ 30-45 นาที

---

### [`gitlab-setup-guide-advanced.md`](gitlab-setup-guide-advanced.md)

**กลุ่มเป้าหมาย:** DevOps Engineers, System Administrators, สภาพแวดล้อม Production

**เนื้อหาที่ครอบคลุม:**
- คำอธิบายเชิงลึกทุกขั้นตอน
- คอมเมนท์อธิบายในไฟล์ Docker Compose
- แนวทางปฏิบัติที่ดีและข้อควรพิจารณาด้านความปลอดภัย
- การตั้งค่า Runner แบบขั้นสูง
- ตัวอย่าง CI/CD Pipeline
- การสำรองข้อมูล, การกู้คืน และการอัปเกรด
- การแก้ไขปัญหาอย่างละเอียด
- คำสั่งที่เป็นประโยชน์สำหรับการจัดการระบบ

**ระยะเวลาในการติดตั้ง:** ประมาณ 1-2 ชั่วโมง (รวมการศึกษาเนื้อหา)

---

## ข้อกำหนดระบบ

### ซอฟต์แวร์
- Docker Engine 20.10 หรือสูงกว่า
- Docker Compose 1.29 หรือสูงกว่า
- Ubuntu Server 20.04 LTS หรือสูงกว่า (แนะนำ)

### ฮาร์ดแวร์
- CPU: 4 cores ขึ้นไป
- RAM: 8 GB ขึ้นไป (แนะนำ 16 GB)
- Storage: 50 GB ขึ้นไป

---

## คำแนะนำการใช้งาน

สำหรับผู้ใช้งานครั้งแรก แนะนำให้เริ่มต้นจาก `gitlab-setup-guide.md` หากต้องการความเข้าใจเชิงลึกและการตั้งค่าขั้นสูง สามารถศึกษาเพิ่มเติมจาก `gitlab-setup-guide-advanced.md`

---

## แหล่งข้อมูลอ้างอิง

- [GitLab Official Documentation](https://docs.gitlab.com/)
- [GitLab Runner Documentation](https://docs.gitlab.com/runner/)
- [Docker Documentation](https://docs.docker.com/)