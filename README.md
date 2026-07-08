# Marketing KPI · RuayHeng289 — ระบบหลังบ้าน (Cloudflare Worker + KV)

ล็อกอินฝั่งเซิร์ฟเวอร์จริง + จัดการผู้ใช้ (เพิ่ม/ลบ/รีเซ็ตรหัส) ผ่านหน้าเว็บ มีผลกับทุกคนทันที

## ไฟล์
- `worker.js` — โค้ด Worker (auth + API + แอป + ข้อมูลฝังในตัว)
- `wrangler.jsonc` — ตั้งค่า (ต้องใส่ KV namespace id)
- `package.json`

## ขั้นตอน Deploy (GitHub → Cloudflare)

### 1) สร้าง KV namespace
Cloudflare Dashboard → **Storage & Databases → KV** → **Create a namespace**
- ตั้งชื่อ เช่น `MKT_KPI_KV` → Create
- **คัดลอก Namespace ID** ที่ได้

### 2) ใส่ ID ลงใน wrangler.jsonc
แก้บรรทัด `"id": "REPLACE_WITH_KV_NAMESPACE_ID"` → ใส่ ID ที่คัดลอกมา
(binding ต้องชื่อ `KV` เท่านั้น — อย่าเปลี่ยน)

### 3) Push โค้ดขึ้น GitHub
สร้าง repo ใหม่ (เช่น `teamdevelopment01-ai/mkt-kpi-backend`) แล้วอัปไฟล์ทั้ง 3 ขึ้นไป

### 4) ต่อ Cloudflare กับ repo
Workers & Pages → **Create → Workers → Connect to Git** → เลือก repo → **Deploy**
(ตั้งชื่อ worker ได้ตามใจ เช่น `mkt-dashboard`)

### 5) เปิดใช้งาน
เปิด URL ของ worker → ระบบจะ **สร้างบัญชีเริ่มต้น 4 บัญชีอัตโนมัติ** ในครั้งแรก:

| บัญชี | Username | Password | สิทธิ์ |
|---|---|---|---|
| แอดมิน | admin | fj9P-Uzcn-de9a-exn5 | ทุกทีม + จัดการผู้ใช้ |
| ทีม A | teama | jitY-5gZM-UV4u-QYeH | TEAM A |
| ทีม B | teamb | Z6SJ-4nKs-Nn7Q-4zrG | TEAM B |
| ทีม C | teamc | tzEX-EX2F-ysz5-DcsP | TEAM C |

> หลังล็อกอิน admin → เมนู **ตั้งค่า › จัดการผู้ใช้** เพิ่ม/ลบ/รีเซ็ตรหัสได้เลย มีผลทันทีกับทุกคน
> แนะนำให้เปลี่ยนรหัส admin หลังใช้งานครั้งแรก

## อัปเดตข้อมูล KPI
ข้อมูล (รายชื่อยูส/ยอดฝาก) ฝังใน `worker.js` — อัปเดตวันใหม่ให้ส่งไฟล์มาให้สร้าง worker.js ตัวใหม่ แล้ว push ขึ้น repo (Cloudflare จะ deploy อัตโนมัติ)

## ความปลอดภัย
- รหัสผ่านเก็บแบบ hash (PBKDF2-SHA256) ใน KV ไม่เก็บ plaintext
- session เก็บเป็น cookie HttpOnly + เก็บ token ใน KV (หมดอายุ 12 ชม. / 30 วันถ้าติ๊กจำ)
