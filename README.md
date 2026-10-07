# AnyDesk Reset Quota

โปรแกรมบรรทัดคำสั่งบน Windows สำหรับรีเซ็ตโควต้า AnyDesk โดยล้างไฟล์คอนฟิกที่เกี่ยวกับการลงทะเบียนและบริการ แล้วให้ AnyDesk สร้าง AnyNet ID ใหม่ แต่ยังพยายามเก็บ user.conf กับโฟลเดอร์ thumbnails ของผู้ใช้ไว้ก่อน แล้วค่อยนำกลับมาหลังรีเซ็ตเสร็จ

## การทำงานของโปรแกรม

ลำดับการทำงานใน main.go เป็นขั้นตอนแบบนี้

1. ตั้งชื่อหน้าต่างคอนโซลเป็น Reset AnyDesk และตั้ง code page ของคอนโซลด้วย chcp 437
2. ตรวจว่ารันในสิทธิ์ Administrator หรือยัง ถ้ายังจะขอ UAC แล้วเปิดโปรแกรมใหม่ด้วย runas ถ้าผู้ใช้กดยกเลิก UAC โปรแกram จะหยุด
3. หยุดบริการ Windows ชื่อ AnyDesk ด้วย sc stop วนจนได้รหัส 1062 ว่าไม่มีบริการรันอยู่ แล้วสั่ง taskkill ปิด AnyDesk.exe
4. ลบไฟล์ service.conf ในโฟลเดอร์ AnyDesk ของ ALLUSERSPROFILE และ APPDATA
5. สำรอง user.conf จาก APPDATA ไป temp และคัดลอกโฟลเดอร์ thumbnails ไป temp
6. ลบไฟล์ทั้งหมดในโฟลเดอร์ AnyDesk ของ ALLUSERSPROFILE และ APPDATA แต่ไม่ลบโฟลเดอร์เอง แค่ลบไฟล์ข้างใน
7. สตาร์ทบริการ AnyDesk แล้วเปิด AnyDesk.exe จาก Program Files หรือ Program Files (x86) ถ้ามี
8. รอจนไฟล์ system.conf ใน ALLUSERSPROFILE มีบรรทัด ad.anynet.id= แปลว่า AnyDesk สร้าง ID ใหม่แล้ว
9. หยุด AnyDesk อีกครั้ง
10. สร้างโฟลเดอร์ APPDATA AnyDesk ใหม่ ย้าย user.conf จาก temp กลับมา และคัดลอก thumbnails กลับ แล้วลบ temp ที่ใช้สำรอง
11. สตาร์ท AnyDesk อีกรอบ แสดงข้อความ Success Process แล้วรอให้กด Enter ก่อนปิด

ฟังก์ชันสำคัญที่แยกไว้ในไฟล์เดียวกับ main ได้แก่ stopAnyDesk และ startAnyDesk จัดการ sc กับ taskkill, ensureAdministrator และ relaunchElevated จัดการสิทธิ์ admin, copyFile copyDir moveFile ใช้ตอนสำรองและคืนค่า user data

## ฟีเจอร์

- ขอสิทธิ์ Administrator อัตโนมัติผ่าน UAC
- หยุดและเริ่มบริการ AnyDesk บน Windows
- ล้างไฟล์ในโฟลเดอร์คอนฟิก AnyDesk เพื่อให้ได้ AnyNet ID ใหม่
- สำรองและคืนค่า user.conf กับ thumbnails ก่อนและหลังรีเซ็ต

## เทคโนโลยีที่ใช้

- ภาษา Go เวอร์ชัน 1.22 ขึ้นไป
- แพ็กเกจ golang.org/x/sys สำหรับ Windows API และ registry
- ระบบปฏิบัติการ Windows เท่านั้น ต้องมี AnyDesk ติดตั้งและลงทะเบียนบริการ AnyDesk แล้ว

## สิ่งที่ต้องมีก่อนติดตั้ง

- Windows ที่รันบริการ AnyDesk ได้
- ติดตั้ง AnyDesk แล้ว โดยปกติอยู่ที่ C:\Program Files (x86)\AnyDesk หรือ C:\Program Files\AnyDesk
- Go เวอร์ชัน 1.22 ขึ้นไป ถ้าจะ build เอง ตรวจด้วยคำสั่ง go version
- Git ถ้าจะ clone จาก repository

## วิธีติดตั้ง

1. ดาวน์โหลดหรือ clone โปรเจกต์มาไว้ในเครื่อง แล้วเข้าโฟลเดอร์นั้น

```bash
cd Anydesk-Reset-quota-main
```

2. ดึง dependency ของ Go

```bash
go mod download
```

3. Build เป็นไฟล์ exe

```bash
go build -o anydesk-reset.exe .
```

ถ้าไม่อยาก build สามารถรันตรงๆ ด้วย go run ได้ แต่ตอนใช้งานจริงแนะนำให้ build แล้วรัน exe ในสิทธิ์ admin

## วิธีใช้งาน

1. ปิดงานสำคัญที่ใช้ AnyDesk อยู่ก่อน เพราะโปรแกรมจะหยุดบริการและปิด AnyDesk.exe ให้เอง
2. เปิด PowerShell หรือ Command Prompt แบบ Run as administrator หรือดับเบิลคลิก exe แล้วกด Yes ตอน UAC
3. ไปที่โฟลเดอร์โปรเจกต์ แล้วรันหนึ่งในแบบนี้

รันจาก source โดยต้อง admin

```bash
go run .
```

หรือรันไฟล์ที่ build แล้ว

```bash
.\anydesk-reset.exe
```

4. ถ้าเปิดครั้งแรกโดยไม่มีสิทธิ์ admin โปรแกรมจะเด้ง UAC ให้กด Yes แล้วหน้าต่างเดิมจะปิด ให้ดูที่หน้าต่างใหม่ที่รันด้วยสิทธิ์ admin แทน
5. รอจนเห็นข้อความ Success Process บนคอนโซล แปลว่าขั้นตอนรีเซ็ตและคืน user.conf กับ thumbnails เสร็จแล้ว
6. กด Enter ตามข้อความ Press Enter to exit... เพื่อปิดโปรแกรม
7. เปิด AnyDesk ตามปกติ แล้วเช็กว่า AnyNet ID เปลี่ยนและยังใช้งานได้ตามที่ต้องการ

ตัวอย่างผลลัพธ์ที่ควรเห็นตอนสำเร็จ

```text
*********
Success Process

Press Enter to exit...
```

ถ้าเห็น Administrator permission was cancelled แปลว่ากดยกเลิก UAC ต้องรันใหม่แล้วกด Yes

## โครงสร้างโฟลเดอร์

```text
Anydesk-Reset-quota-main
  main.go       โค้ดหลักทั้ง flow รีเซ็ตและฟังก์ชันช่วย
  go.mod        ชื่อ module และเวอร์ชัน Go
  go.sum        checksum ของ dependency
  README.md     คู่มือโปรเจกต์
```

## ข้อควรระวัง

- โปรแกรมลบไฟล์ในโฟลเดอร์ AnyDesk ของระบบและของผู้ใช้ ยกเว้นที่สำรองไว้ใน temp ช่วงสั้นๆ ควร backup ข้อมูลสำคัญของ AnyDesk เองถ้าไม่มั่นใจ
- ใช้เฉพาะกรณีที่เข้าใจว่าต้องการรีเซ็ตโควต้า/ID การใช้งาน AnyDesk อาจขัดกับข้อกำหนดของผู้ให้บริการ AnyDesk ผู้ใช้ต้องรับผิดชอบเอง
