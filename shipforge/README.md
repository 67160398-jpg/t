# ShipForge — Ship Design & Sea Trial Simulator
โปรเจกต์นี้เป็น Full-stack version ของ ShipForge demo โดยเชื่อมต่อ
Frontend เดิมเข้ากับ Backend ผ่าน REST API และใช้ MySQL สำหรับจัดเก็บข้อมูล
พร้อมระบบ Authentication และการคำนวณข้อมูลเรือฝั่ง Server
## Tech stack
Frontend: static HTML/CSS/JS (`frontend/index.html`), served by nginx
Backend: FastAPI / Python
Database: MySQL 8.4
Authentication: JWT + Argon2 password hashing
Container: Docker + Docker Compose
API docs: Swagger UI
## Run
```bash
docker compose up --build
```
จากนั้นเปิด:
- Frontend (ตัวเกม): http://localhost:8080
- Swagger: http://localhost:8000/docs
- Health check: http://localhost:8000/api/health

Frontend จะเชื่อมต่อกับ Backend ผ่าน
`http://localhost:8000/api` โดยค่าเริ่มต้น หากต้องการเปลี่ยน Backend URL
สามารถกำหนด `window.SHIPFORGE_API_BASE` ก่อน script ของ `index.html` ทำงาน
หรือแก้ค่า `API_BASE` บริเวณด้านบนของ `<script>`
## API overview
### Authentication
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `POST /api/auth/change-password`
### User management
- `GET /api/me`
- `GET /api/users/{id}`
- `GET /api/users?page=1&limit=10`
- `PUT /api/users/{id}`
- `DELETE /api/users/{id}`
- `GET /api/check-username/{name}`
## ShipForge project API
- `GET /api/parts?search=engine` — ดูรายการชิ้นส่วนเรือ
- `GET /api/materials` — ดูรายการวัสดุ
- `POST /api/ships` — บันทึกแบบเรือ
- `GET /api/ships` — ดูรายการเรือที่บันทึกไว้
- `GET /api/ships/{id}` — โหลดข้อมูลเรือ
- `DELETE /api/ships/{id}` — ลบเรือ
- `POST /api/ships/{id}/inspect` — ตรวจสอบความปลอดภัยของเรือฝั่ง Server
- `POST /api/ships/{id}/sea-trial` — บันทึกผลการทดลองเดินเรือ
- `GET /api/ships/{id}/trials` — ดูประวัติการทดลองเดินเรือ
## Example register
```json
{
  "username": "testuser",
  "email": "test@example.com",
  "full_name": "Test User",
  "password": "12345678"
}
```
## Example login
```json
{
  "username": "testuser",
  "password": "12345678"
}
```
นำ `access_token` ที่ได้รับไปใส่ใน Swagger's Authorize button ในรูปแบบ:
```text
Bearer <access_token>
```
## Example create ship
```json
{
  "name": "Northern Star",
  "ship_type": "fishing",
  "material_key": "steel",
  "parts": [
    {"part_key": "hull", "x": 2, "y": 2},
    {"part_key": "engine", "x": 4, "y": 2},
    {"part_key": "fuelTank", "x": 5, "y": 2}
  ]
}
```
API จะคำนวณค่า `safety_score`, `stability`, `buoyancy`, `range`
และค่าที่เกี่ยวข้องจากชิ้นส่วนและวัสดุของเรือ
การคำนวณฝั่ง Backend อยู่ใน `backend/app/calc.py`
โดยนำ logic จาก JavaScript ของ Frontend มาใช้ เพื่อให้ผลการคำนวณ
ของทั้งสองฝั่งสอดคล้องกัน
## Relationship to the single-file demo
ไฟล์ `shipforge.html` เดิมเป็น Single-page application ที่เก็บข้อมูล
ชิ้นส่วนและวัสดุไว้ใน JavaScript และคำนวณข้อมูลทั้งหมดภายใน Browser
ในเวอร์ชัน Full-stack นี้ ข้อมูลดังกล่าวถูกย้ายไปจัดเก็บใน MySQL
(`parts`, `materials` tables) และการคำนวณที่เกี่ยวข้องถูกนำมาให้
Backend จัดการผ่าน REST API
Frontend ยังคงสามารถทำ Live calculation สำหรับการออกแบบเรือได้
ขณะที่ระบบ Login จะทำให้ผู้ใช้สามารถบันทึกและโหลดเรือจากบัญชีของตนเอง
รวมถึงตรวจสอบค่า safety score ของเรือผ่าน Server ได้
