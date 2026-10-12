# RegisterApp: ระบบค้นหาและลงทะเบียนรายวิชา

โปรเจกต์ตัวอย่างสำหรับค้นหารายวิชาและเพิ่มหรือลบรายวิชาที่ลงทะเบียน ใช้ Next.js เป็น frontend, Express เป็น REST API และ MongoDB replica set สำหรับเก็บข้อมูล

## ส่วนประกอบและโครงสร้าง

| Path | รายละเอียด |
| --- | --- |
| `pages/`, `components/` | หน้าเว็บและส่วนประกอบของ Next.js |
| `backend/index.js` | Express API ซึ่งให้บริการที่พอร์ต `80` |
| `replica_database/README.md` | วิธีตั้งค่า MongoDB replica set |
| `package.json` | คำสั่งและ dependencies ของ frontend |
| `backend/package.json` | dependencies ของ backend |

Backend เชื่อมต่อ replica set ที่ `localhost:27018,localhost:27019,localhost:27020` ชื่อ `db_replica` โดยตรงจากโค้ด ไม่มีการตั้งค่า connection ผ่าน environment variable ในโปรเจกต์นี้ ข้อมูลแค็ตตาล็อกอยู่ใน `course.subjects` และรายการลงทะเบียนอยู่ใน `registration.subjects` (ชื่อ collection คือ `subjects` ทั้งสองฐานข้อมูล)

## สิ่งที่ต้องมี

- Node.js และ npm
- MongoDB Server (`mongod`) และ MongoDB Shell (`mongosh`)
- พอร์ต `80`, `27018`, `27019`, `27020` และ `3000` ต้องพร้อมใช้งาน

ตรวจสอบว่ามีคำสั่งที่ต้องใช้:

```bash
node --version
npm --version
mongod --version
mongosh --version
```

## ติดตั้งและเริ่มใช้งานในเครื่อง

### 1. ดาวน์โหลดและติดตั้ง dependencies

```bash
git clone https://github.com/kitti-noo-university/RegitApp-MongoDB-NodeJS.git
cd RegitApp-MongoDB-NodeJS
npm install
```

ติดตั้ง backend แยกอีกชุด โดยรันจากโฟลเดอร์รากของโปรเจกต์:

```bash
cd backend
npm install
cd ..
```

### 2. เริ่ม MongoDB replica set

เปิด terminal สามหน้าต่าง และรัน `mongod` หนึ่ง process ต่อหนึ่งหน้าต่างตามคำสั่งใน [คู่มือตั้งค่า replica set](replica_database/README.md) แต่ละ process ต้องใช้พอร์ตและ data directory ของตัวเอง

> **คำเตือน:** โฟลเดอร์ `replica_database/r1`, `r2` และ `r3` ใน repository มีไฟล์ฐานข้อมูลอยู่แล้ว อย่าใช้หรือเขียนทับโฟลเดอร์เหล่านั้นสำหรับการทดลองใหม่ ให้ใช้ data directory ว่างนอก repository ตามตัวอย่างในคู่มือแทน ข้อมูลในตัวอย่างนี้มีไว้สำหรับ local development เท่านั้น

ตั้งค่า replica set ให้พร้อมก่อนเริ่ม backend โดยตรวจ `rs.status()` จนได้สถานะปกติ (`ok: 1` และมีสมาชิก `PRIMARY` หนึ่งตัวกับ `SECONDARY` สองตัว)

### 3. เริ่ม backend

เปิด terminal ใหม่จากโฟลเดอร์รากของ repository:

```bash
cd backend
node index.js
```

Backend ไม่มี `start` script ใน `backend/package.json`; ใช้ `node index.js` ได้โดยตรง หรือใช้ `npx nodemon index.js` เพื่อเริ่มด้วย nodemon ที่ติดตั้งเป็น dependency แล้ว API ใช้พอร์ต `80` ตามที่ระบุใน `backend/index.js` พอร์ตนี้อาจต้องใช้สิทธิ์เพิ่มเติมบนบางระบบปฏิบัติการ และอาจชนกับบริการอื่น หากเริ่มไม่ได้ให้ตรวจสิทธิ์และตรวจว่าพอร์ตว่าง โดยไม่เปลี่ยนพอร์ตเอง เพราะ URL ของ frontend ระบุ `http://localhost/api` ไว้ตายตัว

### 4. เพิ่มข้อมูลรายวิชาตัวอย่าง

เมื่อ replica set และ backend ทำงานแล้ว ให้เรียก `GET /api/add-all` **ครั้งเดียว** บนฐานข้อมูล `course` ที่ว่าง เพื่อเพิ่มรายการรายวิชาตัวอย่างจากโค้ด:

```bash
curl http://localhost/api/add-all
```

การเรียกซ้ำอาจเพิ่มข้อมูลซ้ำ จึงไม่ควรเรียกกับฐานข้อมูลที่มีข้อมูลอยู่แล้ว

### 5. เริ่ม frontend

เปิด terminal อีกหน้าต่างจากโฟลเดอร์รากของ repository:

```bash
npm run dev
```

เปิด [http://localhost:3000](http://localhost:3000) ในเบราว์เซอร์ Frontend เรียก API ที่ `http://localhost/api`; ดังนั้น backend ต้องทำงานบนเครื่องเดียวกันและพอร์ต `80` ต้องเข้าถึงได้

## API ที่หน้าเว็บใช้

ทุก route มี prefix `/api`:

| Method | Path | การทำงาน |
| --- | --- | --- |
| `GET` | `/api/subjects` | อ่านรายวิชาจาก `course.subjects` |
| `GET` | `/api/regist` | อ่านรายวิชาที่ลงทะเบียนจาก `registration.subjects` |
| `POST` | `/api/regist/:id` | เพิ่มรายวิชาตามรหัสลงทะเบียน |
| `DELETE` | `/api/regist/:id` | ถอนรายวิชาตามรหัส |
| `GET` | `/api/add-all` | เพิ่มชุดข้อมูลรายวิชาตัวอย่างลง `course.subjects` |

## คำสั่ง frontend

รันคำสั่งจากโฟลเดอร์ราก:

| คำสั่ง | การทำงาน |
| --- | --- |
| `npm run dev` | เริ่ม Next.js development server ที่พอร์ต `3000` |
| `npm run build` | สร้าง production build ของ frontend |
| `npm start` | เริ่ม Next.js server จาก build ที่สร้างไว้แล้ว |
| `npm run lint` | เรียก Next.js lint |

Backend ไม่มีคำสั่ง `start` หรือชุดทดสอบที่ใช้งานได้กำหนดไว้ใน `backend/package.json`

## แก้ปัญหาเบื้องต้น

- **เชื่อมต่อ MongoDB ไม่ได้:** ตรวจว่า `mongod` ทั้งสาม process ยังทำงาน ใช้พอร์ต `27018–27020` และ replica set ชื่อ `db_replica` พร้อมสมาชิก `PRIMARY`/`SECONDARY` ตามที่คาดไว้
- **API เริ่มไม่ได้:** ตรวจว่าพอร์ต `80` ว่างและผู้ใช้มีสิทธิ์ bind พอร์ตนี้
- **หน้าเว็บไม่มีรายวิชา:** ตรวจว่า backend และ MongoDB ทำงาน แล้วตรวจข้อมูลใน `course.subjects`; ฐานข้อมูลว่างสามารถเพิ่มข้อมูลตัวอย่างด้วย `GET /api/add-all` ได้ครั้งเดียว
- **frontend เรียก API ไม่ได้:** ตรวจว่าเปิดหน้าเว็บและ API จากเครื่องเดียวกัน และเข้า `http://localhost/api` ได้

ตัวอย่างนี้ออกแบบเพื่อทดลองในเครื่อง ไม่มีการตั้งค่าการยืนยันตัวตนหรือการรักษาความปลอดภัยสำหรับ production อย่าใช้กับข้อมูลจริงหรือเปิดบริการสู่สาธารณะโดยไม่มีการปรับปรุงความปลอดภัย การนำ frontend แบบ static ไปลง GitHub Pages ไม่สามารถใช้แทนการรัน Node.js backend ได้

ดูขั้นตอนเฉพาะเพิ่มเติมได้ที่ [Backend README](backend/README.md) และ [Replica database README](replica_database/README.md)
