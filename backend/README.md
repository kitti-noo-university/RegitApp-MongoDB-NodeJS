# Backend REST API

Backend ของ RegisterApp ใช้ Express และ MongoDB Node.js driver ดูภาพรวมและขั้นตอนเริ่มระบบทั้งหมดที่ [README หลัก](../README.md) และตั้งค่า MongoDB replica set ตาม [คู่มือ replica database](../replica_database/README.md) ก่อนเริ่ม backend

## ติดตั้งและเริ่ม server

คำสั่งทั้งหมดในส่วนนี้รันจากโฟลเดอร์ `backend`:

```bash
npm install
node index.js
```

หรือเริ่มด้วย nodemon ที่ติดตั้งเป็น dependency:

```bash
npx nodemon index.js
```

`backend/package.json` ไม่มี `start` script; entrypoint คือ `index.js` และ server รับ request ที่พอร์ต `80` Backend เชื่อมต่อ MongoDB ที่ `localhost:27018`, `localhost:27019` และ `localhost:27020` โดยใช้ replica set ชื่อ `db_replica` ซึ่งกำหนดไว้ใน `index.js` ไม่ได้อ่าน connection string จาก environment variable

Frontend ระบุ API URL เป็น `http://localhost/api` จึงต้องเปิด backend บนเครื่องเดียวกันและใช้พอร์ต `80` หากพอร์ตนี้ไม่ว่างหรือระบบปฏิบัติการต้องการสิทธิ์เพิ่มเติม การเริ่ม server อาจล้มเหลว

## Routes

ทุก route ใช้ prefix `/api`:

| Method | Path | การทำงาน |
| --- | --- | --- |
| `GET` | `/api/` | ส่งข้อความต้อนรับ |
| `GET` | `/api/subjects` | อ่านรายวิชาจาก `course.subjects` |
| `GET` | `/api/subjects/:id` | อ่านรายวิชาตามรหัส |
| `POST` | `/api/add-subject` | เพิ่มรายวิชาจาก request body |
| `GET` | `/api/add-all` | เพิ่มชุดรายวิชาตัวอย่างลง `course.subjects` |
| `GET` | `/api/delete-all` | ลบ collection `course.subjects` |
| `GET` | `/api/regist` | อ่านรายการลงทะเบียนจาก `registration.subjects` |
| `POST` | `/api/regist/:id` | เพิ่มรายวิชาในรายการลงทะเบียน |
| `DELETE` | `/api/regist/:id` | ถอนรายวิชาจากรายการลงทะเบียน |

ชื่อ collection ในโค้ดคือ `subjects` ทั้งในฐานข้อมูล `course` และ `registration` เรียก `/api/add-all` เฉพาะเมื่อ `course.subjects` ว่างและต้องการข้อมูลตัวอย่าง เพราะการเรียกซ้ำอาจเพิ่มเอกสารซ้ำ ส่วน `/api/delete-all` เป็นการลบข้อมูลและไม่ควรเรียกกับฐานข้อมูลที่ต้องการเก็บไว้

ตัวอย่างนี้ตั้งใจใช้ local development ไม่มี authentication และไม่ควรนำไปเปิดให้บริการสาธารณะหรือใช้กับข้อมูลจริงโดยไม่มีการปรับปรุงความปลอดภัย

## Scripts และ tests

ไม่มี `start` script และค่า `test` ใน package เป็น placeholder ที่จบด้วยข้อผิดพลาด `no test specified`; ใช้คำสั่งเริ่ม server ด้านบนแทน
