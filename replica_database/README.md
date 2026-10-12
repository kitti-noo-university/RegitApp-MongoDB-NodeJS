# MongoDB Replica Set สำหรับ local development

RegisterApp เชื่อมต่อ replica set ชื่อ `db_replica` ผ่านสมาชิกสามตัวที่ `localhost:27018`, `localhost:27019` และ `localhost:27020` connection string ใน `backend/index.js` ระบุสมาชิกและชื่อ replica set นี้ไว้โดยตรง

## เตรียม data directory

โฟลเดอร์ `r1`, `r2` และ `r3` ที่อยู่ใน repository มีไฟล์ฐานข้อมูลเดิมอยู่แล้ว อย่าเปิด `mongod` บนโฟลเดอร์เหล่านั้นหรือเขียนทับข้อมูล ให้สร้างโฟลเดอร์ว่างแยกไว้นอก repository ตัวอย่างต่อไปนี้ใช้ shell แบบ Bash:

```bash
mkdir -p "$HOME/regitapp-mongodb/r1" \
  "$HOME/regitapp-mongodb/r2" \
  "$HOME/regitapp-mongodb/r3"
```

หาก path ตัวอย่างมีข้อมูลเดิมอยู่แล้ว ให้เลือก path ใหม่ที่ว่างแทน แต่ละ process ต้องใช้ data directory ของตัวเอง ห้ามแชร์ directory ระหว่าง process

## เริ่ม mongod ทั้งสามตัว

เปิด terminal สามหน้าต่าง แล้วรันหนึ่งคำสั่งต่อหนึ่งหน้าต่าง แต่ละหน้าต่างต้องเปิดค้างไว้ขณะใช้งาน:

**Terminal 1**

```bash
mongod --replSet db_replica --dbpath "$HOME/regitapp-mongodb/r1" --port 27018
```

**Terminal 2**

```bash
mongod --replSet db_replica --dbpath "$HOME/regitapp-mongodb/r2" --port 27019
```

**Terminal 3**

```bash
mongod --replSet db_replica --dbpath "$HOME/regitapp-mongodb/r3" --port 27020
```

คำสั่งนี้ใช้ค่า bind address เริ่มต้นของ MongoDB สำหรับการเชื่อมต่อในเครื่องเดียวกัน ถ้า `mongod` แจ้งว่าพอร์ตถูกใช้งาน ให้ปิด process ที่ใช้พอร์ตนั้นหรือเลือกการตั้งค่าอื่นที่สอดคล้องกับ connection string ของ backend; backend ปัจจุบันกำหนดพอร์ตทั้งสามนี้ไว้ตายตัว

## เริ่มต้นและตรวจสอบ replica set

หลังจาก `mongod` ทั้งสามตัวเริ่มทำงาน ให้เปิด terminal อีกหน้าต่างและเชื่อมต่อ shell ที่สมาชิกแรก:

```bash
mongosh --port 27018
```

ใน shell ให้เริ่ม replica set ด้วย configuration เดียวกับที่ backend คาดหวัง:

```javascript
rs.initiate({
  _id: "db_replica",
  members: [
    { _id: 0, host: "localhost:27018" },
    { _id: 1, host: "localhost:27019" },
    { _id: 2, host: "localhost:27020" }
  ]
});
```

ตรวจสถานะ:

```javascript
rs.status();
```

รอจนผลลัพธ์มี `ok: 1` และสมาชิกหนึ่งตัวมีสถานะ `PRIMARY` ส่วนอีกสองตัวเป็น `SECONDARY` ก่อนเริ่ม backend หาก replica set ถูกเริ่มไว้แล้ว ให้ตรวจด้วย `rs.status()` แทนการเรียก `rs.initiate()` ซ้ำ

## ฐานข้อมูลและการเชื่อมต่อของแอป

Backend ใช้ข้อมูลดังนี้:

| Database | Collection | ใช้สำหรับ |
| --- | --- | --- |
| `course` | `subjects` | แค็ตตาล็อกรายวิชา |
| `registration` | `subjects` | รายวิชาที่ลงทะเบียน |

Collection ใช้ชื่อ `subjects` (พหูพจน์) ตาม `backend/index.js` ฐานข้อมูลและ collection จะถูกสร้างเมื่อมีการเขียนข้อมูล ไม่จำเป็นต้องสร้าง collection `subject` แยกเอง บนฐานข้อมูล `course` ว่าง สามารถเพิ่มชุดข้อมูลตัวอย่างได้ครั้งเดียวหลังเริ่ม backend ด้วย `GET http://localhost/api/add-all`; ดูคำสั่งเริ่ม backend และ frontend ที่ [README หลัก](../README.md)

ตัวอย่าง connection string ที่ใช้ในโค้ด:

```text
mongodb://localhost:27018,localhost:27019,localhost:27020?replicaSet=db_replica
```

หากต้องการตรวจข้อมูลผ่าน `mongosh`:

```bash
mongosh "mongodb://localhost:27018,localhost:27019,localhost:27020/?replicaSet=db_replica"
```

จากนั้นเลือกฐานข้อมูลและอ่าน collection:

```javascript
use course
db.getCollection("subjects").find({});

use registration
db.getCollection("subjects").find({});
```

`registration.subjects` อาจยังไม่มีจนกว่าจะมีการลงทะเบียนรายวิชา

## หยุด process

เมื่อทดลองเสร็จ ให้กด `Ctrl+C` ใน terminal ของ `mongosh` และ `mongod` แต่ละหน้าต่าง ข้อมูลจะยังอยู่ใน data directory ที่เลือกไว้ อย่าลบ directory ที่มีข้อมูลหากยังต้องการเก็บไว้
