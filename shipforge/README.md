ShipForge — Ship Design & Sea Trial Simulator
Full-stack version of the ShipForge demo: a FastAPI + MySQL backend behind the original single-page frontend.
Tech Stack
Frontend: HTML/CSS/JavaScript (`frontend/index.html`)
Backend: FastAPI / Python
Database: MySQL 8.4
Authentication: JWT + Argon2 password hashing
Container: Docker + Docker Compose
API Docs: Swagger UI
Web Server: nginx
Run
```bash
docker compose up --build
```
จากนั้นเปิด:
Frontend: http://localhost:8080
Swagger: http://localhost:8000/docs
Health Check: http://localhost:8000/api/health
Frontend จะเชื่อมต่อ Backend ผ่าน:
```text
http://localhost:8000/api
```
หากต้องการเปลี่ยน URL ของ Backend สามารถกำหนด `window.SHIPFORGE_API_BASE` ก่อน script หลักของ `index.html` ทำงาน หรือแก้ค่า `API_BASE` ภายในไฟล์ `frontend/index.html`
API ที่ตรงกับโปรเจกต์
1. Authentication
`POST /api/auth/register`
`POST /api/auth/login`
`POST /api/auth/logout`
`POST /api/auth/change-password`
2. User Management
`GET /api/me`
`GET /api/users/{id}`
`GET /api/users?page=1&limit=10`
`PUT /api/users/{id}`
`DELETE /api/users/{id}`
`GET /api/check-username/{name}`
3. ShipForge Project API
`GET /api/parts?search=engine` — ดูรายการชิ้นส่วนเรือ
`GET /api/materials` — ดูรายการวัสดุ
`POST /api/ships` — บันทึกแบบเรือ
`GET /api/ships` — ดูรายการเรือของผู้ใช้
`GET /api/ships/{id}` — ดูรายละเอียดเรือ
`DELETE /api/ships/{id}` — ลบเรือ
`POST /api/ships/{id}/inspect` — ตรวจสอบความปลอดภัยของเรือ
`POST /api/ships/{id}/sea-trial` — บันทึกผลการทดลองเดินเรือ
`GET /api/ships/{id}/trials` — ดูประวัติการทดลองเดินเรือ
Example Register
```json
{
  "username": "testuser",
  "email": "test@example.com",
  "full_name": "Test User",
  "password": "12345678"
}
```
Example Login
```json
{
  "username": "testuser",
  "password": "12345678"
}
```
หลังจาก Login สำเร็จ ให้นำ `access_token` ไปใส่ใน Swagger ที่ปุ่ม Authorize ในรูปแบบ:
```text
Bearer <access_token>
```
Example Create Ship
```json
{
  "name": "Northern Star",
  "ship_type": "fishing",
  "material_key": "steel",
  "parts": [
    {
      "part_key": "hull",
      "x": 2,
      "y": 2
    },
    {
      "part_key": "engine",
      "x": 4,
      "y": 2
    },
    {
      "part_key": "fuelTank",
      "x": 5,
      "y": 2
    }
  ]
}
```
ระบบจะคำนวณค่าต่าง ๆ ของเรือ เช่น:
`safety_score`
`stability`
`buoyancy`
`range`
โดยใช้ข้อมูลจากชิ้นส่วนและวัสดุของเรือ
การคำนวณฝั่ง Backend อยู่ใน:
```text
backend/app/calc.py
```
Relationship to the Original ShipForge Demo
เวอร์ชันเดิมของ ShipForge เป็น Single-page application ซึ่งเก็บข้อมูลชิ้นส่วนและวัสดุไว้ใน JavaScript และทำการคำนวณภายใน Browser
ในเวอร์ชัน Full-stack นี้ ข้อมูลและการทำงานบางส่วนถูกแยกออกมาเป็น Backend และ Database:
```text
ShipForge
│
├── Frontend
│   └── HTML / CSS / JavaScript
│
├── Backend
│   └── FastAPI / Python
│
└── Database
    └── MySQL
```
Frontend ยังคงมีระบบคำนวณแบบ Live สำหรับการออกแบบเรือ ขณะที่ Backend สามารถคำนวณและตรวจสอบข้อมูลผ่าน API ได้
Authentication
ระบบ Authentication ใช้:
JWT สำหรับการยืนยันตัวตน
Argon2 สำหรับ Hash Password
Protected API สำหรับข้อมูลที่เกี่ยวข้องกับบัญชีผู้ใช้
ตัวอย่างการทำงาน:
```text
Register
   ↓
Login
   ↓
Receive access_token
   ↓
Authorize API
   ↓
Save / Load Ship
```
Docker Services
ระบบประกอบด้วย Docker services หลัก:
```text
docker compose
│
├── mysql
│   └── MySQL 8.4
│
├── backend
│   └── FastAPI
│
└── frontend
    └── nginx
```
Port ที่ใช้งาน:
Service	Port
Frontend / nginx	`8080`
Backend / FastAPI	`8000`
MySQL	`3306`
