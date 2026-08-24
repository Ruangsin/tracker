# EC Tracker — ระบบติดตามความก้าวหน้านักศึกษา

ระบบติดตาม **คะแนนเก็บ** และ **การเข้าเรียน** สำหรับ 3 รายวิชา สาขาภาษาอังกฤษเพื่อการสื่อสาร คณะศิลปศาสตร์ มทร.สุวรรณภูมิ
ผู้สอน: อ.เรืองสิน ปลื้มปั่น | ภาคการศึกษา 1/2569

🔗 **ใช้งานจริง:**
- นักศึกษา: [ruangsin.github.io/tracker/dashboard.html](https://ruangsin.github.io/tracker/dashboard.html)
- ผู้สอน: [ruangsin.github.io/tracker/teacher.html](https://ruangsin.github.io/tracker/teacher.html)

---

## รายวิชาที่รองรับ

| รหัสวิชา | ชื่อวิชา | กลุ่มเรียน |
|---|---|---|
| PRON | English Pronunciation | AIE46941N |
| NEWS | English through News | EC46741N |
| ADPR | English for Advertisement & PR | EC46741N |

---

## ไฟล์ในโปรเจกต์นี้

| ไฟล์ | หน้าที่ |
|---|---|
| `dashboard.html` | หน้านักศึกษา — login ด้วยรหัสนักศึกษา + PIN (เลข 4 หลักท้ายรหัส) ดูคะแนนเก็บและการเข้าเรียนของตัวเอง รองรับนักศึกษาที่ลงหลายวิชาพร้อมกัน (แสดงเป็นแท็บ) |
| `teacher.html` | หน้าผู้สอน — login ด้วย PIN อาจารย์ ดูภาพรวมนักศึกษาทุกคนทั้ง 3 วิชา พร้อมเรียงลำดับ/ค้นหา/กรอง "คนที่ควรติดตาม"/Export CSV |
| `Code.gs` | Backend (Google Apps Script) — ไม่ได้เก็บในไฟล์นี้ อยู่แยกในโปรเจกต์ Apps Script ชื่อ **Teacher Tracker** ที่ script.google.com |

---

## สถาปัตยกรรมระบบ

```
นักศึกษา/อาจารย์ → dashboard.html / teacher.html (GitHub Pages, static)
                          │  fetch (JSON)
                          ▼
              Google Apps Script Web App (Code.gs)
                    │                    │
                    ▼                    ▼
       ไฟล์ "ระบบเช็คชื่อ 3 วิชา"     ไฟล์ "Assignments Tracker"
       (Courses, Roster_x,           (Students, Subject,
        Sessions_x, Attendance_x,     Scores, Settings)
        Settings)
```

Apps Script ตัวเดียวเปิดไฟล์ Google Sheet 2 ไฟล์แยกกันผ่าน `SpreadsheetApp.openByUrl()` แล้วรวมข้อมูลเป็น JSON ชุดเดียวส่งกลับให้หน้าเว็บ รองรับ 2 endpoint:

- `?action=login&studentId=xxx&pin=xxxx` → สำหรับนักศึกษา
- `?action=teacher&pin=xxxx` → สำหรับอาจารย์

ผลอ่าน Google Sheets ถูก**แคชไว้ 60 วินาที** (ใช้ CacheService) เพื่อความเร็ว — หมายความว่าข้อมูลอาจล่าช้าได้สูงสุด ~1 นาทีหลังกรอกคะแนน/เช็คชื่อใหม่

---

## ธีม

**Neon Glassmorphism** — พื้นดำม่วงเข้ม การ์ดกระจกโปร่งแสงขอบเรืองแสง โครง sidebar ซ้ายสำหรับหน้าผู้สอน

---

## การติดตั้ง / แก้ไขระบบ

### 1. แก้ไขหน้าเว็บ (dashboard.html / teacher.html)
แก้ไฟล์แล้วอัปโหลดทับของเดิมใน repo นี้ผ่าน **Add file → Upload files** หรือกดไอคอนดินสอ (✏️) แก้ตรงบน GitHub ได้เลย ไม่กระทบ URL ของเว็บ

### 2. แก้ไข Backend (Code.gs)
1. เข้า [script.google.com](https://script.google.com) เปิดโปรเจกต์ **Teacher Tracker**
2. แก้โค้ดในไฟล์ `รหัส.gs`
3. ตรวจสอบว่าตัวแปรด้านบนของไฟล์ยังมี URL จริงของทั้ง 2 Google Sheet อยู่:
   ```js
   var ATTENDANCE_SHEET_URL = '...';
   var SCORES_SHEET_URL = '...';
   ```
   *(ใช้ URL เต็มที่คัดลอกจาก address bar เท่านั้น อย่าตัด-พิมพ์ ID เอง เสี่ยงพิมพ์ I/l สลับกัน)*
4. **Deploy → จัดการทำให้ใช้งานได้ → ไอคอนดินสอ → Version: "New version" → Deploy**
   ⚠️ ห้ามกด "New deployment" เด็ดขาด เพราะจะได้ URL ใหม่ ต้องไปแก้ `WEB_APP_URL` ในไฟล์ HTML ทั้ง 2 ไฟล์ตามด้วย

### 3. เชื่อมหน้าเว็บกับ Apps Script
ทั้ง `dashboard.html` และ `teacher.html` ต้องมี `WEB_APP_URL` **ตัวเดียวกัน** (Apps Script ตัวเดียวรองรับทั้งสอง endpoint):
```js
const WEB_APP_URL = "https://script.google.com/macros/s/XXXXXXX/exec";
```

---

## Login

| ผู้ใช้ | วิธี login |
|---|---|
| นักศึกษา | รหัสนักศึกษา + PIN (เลข 4 หลักท้ายรหัสนักศึกษา — ค่าเดียวกับที่ใช้เช็คชื่อ) |
| อาจารย์ | PIN เดียว — ค่าจากชีต `Settings` → `TeacherPIN` ในไฟล์ระบบเช็คชื่อ |

---

## ผู้ดูแลระบบ

อ.เรืองสิน ปลื้มปั่น — พัฒนาโดยความช่วยเหลือของ Claude
