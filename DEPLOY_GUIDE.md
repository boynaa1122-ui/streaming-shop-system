# คู่มือ Deploy GaFiwSHOP บน Vercel + Supabase

## ภาพรวม Architecture

```
GitHub (โค้ด) → Vercel (Frontend + API) → Supabase (Database)
```

---

## ขั้นตอนที่ 1: สร้าง Database บน Supabase

1. ไปที่ https://supabase.com และสมัคร/เข้าสู่ระบบ
2. คลิก **New Project** → ตั้งชื่อ `gafiwshop` → เลือก Region ใกล้ที่สุด (Singapore)
3. ตั้ง Database Password (จดไว้)
4. รอสร้างเสร็จ (~2 นาที)
5. ไปที่ **Settings → Database** → คัดลอก **Connection String (URI)**
   - รูปแบบ: `postgresql://postgres:[PASSWORD]@db.[PROJECT_REF].supabase.co:5432/postgres`
   - **หมายเหตุ:** โปรเจกต์นี้ใช้ MySQL/TiDB ผ่าน Manus built-in DB ซึ่งทำงานได้แล้ว
   - หากต้องการย้ายไป Supabase ต้องแก้ไข `drizzle/schema.ts` จาก `mysqlTable` เป็น `pgTable`

> **แนะนำ:** ใช้ Manus built-in Database ที่มีอยู่แล้วสำหรับ deploy บน Vercel จะง่ายกว่า

---

## ขั้นตอนที่ 2: Deploy บน Vercel

### 2.1 เชื่อมต่อ GitHub กับ Vercel

1. ไปที่ https://vercel.com และ Login ด้วย GitHub
2. คลิก **Add New Project**
3. เลือก Repository: `boynaa1122-ui/Apps-stem-online`
4. คลิก **Import**

### 2.2 ตั้งค่า Build Settings

| Setting | Value |
|---|---|
| Framework Preset | Other |
| Build Command | `pnpm run build` |
| Output Directory | `dist` |
| Install Command | `pnpm install` |

### 2.3 ตั้งค่า Environment Variables

ใน Vercel Dashboard → **Settings → Environment Variables** เพิ่มค่าต่อไปนี้:

| Variable | Value | หมายเหตุ |
|---|---|---|
| `DATABASE_URL` | `mysql://user:pass@host:3306/db` | Connection string ของ MySQL/TiDB |
| `JWT_SECRET` | (สุ่มค่า 32+ ตัวอักษร) | สำหรับ session cookie |
| `NODE_ENV` | `production` | |
| `VITE_APP_ID` | (จาก Manus OAuth) | |
| `OAUTH_SERVER_URL` | `https://api.manus.im` | |
| `VITE_OAUTH_PORTAL_URL` | `https://auth.manus.im` | |

### 2.4 Deploy

คลิก **Deploy** และรอ ~3-5 นาที

---

## ขั้นตอนที่ 3: ตั้งค่าหลัง Deploy

### ตั้งค่า Admin

หลัง deploy เสร็จ ให้ login เข้าเว็บ จากนั้นไปที่ Supabase/Database แล้วรัน SQL:

```sql
UPDATE users SET role = 'admin' WHERE email = 'your-email@example.com';
```

หรือใช้ Manus Database UI ใน Management Panel

### เข้า Admin Panel

ไปที่ `https://your-domain.vercel.app/admin`

---

## การใช้งาน Admin Panel

| หน้า | URL | ฟีเจอร์ |
|---|---|---|
| Dashboard | `/admin` | สถิติภาพรวม, ออเดอร์ล่าสุด |
| สินค้า | `/admin/products` | เพิ่ม/แก้ไข/ลบสินค้า |
| สต็อก | `/admin/stock` | เพิ่ม account:password สำหรับส่งให้ลูกค้า |
| เติมเงิน | `/admin/topups` | อนุมัติ/ปฏิเสธคำขอเติมเงิน |
| ออเดอร์ | `/admin/orders` | ดูออเดอร์ทั้งหมด |
| ประกาศ | `/admin/announcements` | เพิ่ม/แก้ไขประกาศ |
| Flash Sale | `/admin/flash-sales` | ตั้งค่า Flash Sale + ราคาพิเศษ |

---

## วิธีเพิ่มสต็อกสินค้า

1. ไปที่ `/admin/stock`
2. เลือกสินค้า
3. ใส่ข้อมูลในรูปแบบ `account:password` หรือ `email:password` (1 รายการต่อบรรทัด)
4. คลิก **เพิ่มสต็อก**
5. ระบบจะส่งข้อมูลให้ลูกค้าอัตโนมัติเมื่อซื้อสำเร็จ

---

## Tech Stack

- **Frontend:** React 19 + Tailwind CSS 4 + shadcn/ui
- **Backend:** Express + tRPC 11
- **Database:** Drizzle ORM + MySQL (TiDB)
- **Auth:** Manus OAuth
- **Hosting:** Vercel (Serverless)
