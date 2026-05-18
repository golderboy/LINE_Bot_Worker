# SKILL.md
# Generic Development Skill for Event Intake + Tracking Dashboard Systems

สถานะเอกสาร: Generic Template  
วัตถุประสงค์: ใช้เป็นแนวทางสำหรับ DEV หรือ AI Dev ในการพัฒนาระบบรับ Event จากภายนอก → แปลงเป็นรายการติดตาม → แสดงและอัปเดตผ่าน Dashboard

---

# 1. บทบาทของผู้พัฒนา

ให้ทำหน้าที่เป็น Senior Full-stack / Integration Developer

ต้อง:
- อ่าน `REQUIREMENTS.md` ทั้งหมดก่อนเริ่ม
- แตก Requirement เป็นงานย่อยอย่างมีเหตุผล
- รักษา architecture ตามที่กำหนด
- ไม่ขยาย scope เอง
- พัฒนาระบบให้พร้อม deploy และทดสอบได้จริง
- จัดทำ README ครบถ้วน
- แยกโค้ดให้ดูแลต่อได้

---

# 2. รูปแบบระบบที่ต้องรักษา

ผู้พัฒนาต้องคง Flow หลักนี้:

```text
External Source
        ↓
Secure Webhook Receiver
        ↓
Validation / Authentication
        ↓
Trigger Parser
        ↓
Metadata Enrichment
        ↓
Duplicate Check
        ↓
Central Storage
        ↓
Dashboard API
        ↓
Dashboard UI
        ↓
Update only allowed operational fields
```

---

# 3. แนวทางเลือก Technology Stack

โปรเจกต์จริงสามารถเลือก stack ได้ตามความเหมาะสม แต่ต้องระบุให้ชัดเจน

ตัวอย่าง stack ที่เหมาะกับระบบลักษณะนี้:

## Option A — Serverless-first
- Cloudflare Workers / Edge Functions
- TypeScript
- Static Web Assets
- External Spreadsheet / SaaS Storage / REST API

## Option B — Traditional Web App
- Laravel / Node.js / FastAPI
- MySQL / PostgreSQL
- Web UI
- Background queue ถ้าจำเป็น

## Option C — Hybrid
- Serverless Webhook Receiver
- Existing database/storage
- Separate web dashboard

> ผู้พัฒนาต้องเลือกให้เหมาะกับ Requirement จริง และห้ามเปลี่ยน architecture โดยไม่มีเหตุผล

---

# 4. Recommended Project Structure

ตัวอย่างโครงสร้างกลาง:

```text
project/
├─ src/
│  ├─ webhook/
│  │  ├─ receiver.ts
│  │  ├─ verification.ts
│  │  └─ parser.ts
│  ├─ integrations/
│  │  ├─ source-api.ts
│  │  └─ storage-api.ts
│  ├─ services/
│  │  ├─ event-intake-service.ts
│  │  ├─ duplicate-service.ts
│  │  └─ dashboard-service.ts
│  ├─ dashboard/
│  │  ├─ pages/
│  │  ├─ components/
│  │  └─ api-client/
│  ├─ auth/
│  │  ├─ login.ts
│  │  └─ session.ts
│  ├─ utils/
│  │  ├─ logger.ts
│  │  ├─ response.ts
│  │  └─ datetime.ts
│  └─ types.ts
├─ tests/
│  ├─ webhook.test.ts
│  ├─ parser.test.ts
│  ├─ duplicate.test.ts
│  ├─ auth.test.ts
│  └─ dashboard-api.test.ts
├─ README.md
├─ REQUIREMENTS.md
└─ SKILL.md
```

ปรับได้ตาม stack จริง แต่ต้องคง separation of concerns

---

# 5. Development Sequence

---

## Phase 0 — Read and Freeze the Contract
ก่อนเขียนโค้ด ต้องระบุให้ชัด:
- Source platform คืออะไร
- Event type ไหนที่รองรับ
- Trigger rules คืออะไร
- Storage คืออะไร
- Editable fields คืออะไร
- Authentication 방식ใดจะใช้
- What is out of scope

---

## Phase 1 — Build the Secure Receiver

ทำ:
1. สร้าง webhook endpoint
2. รับ raw body
3. verify request
4. parse payload หลัง verify สำเร็จ
5. จัดการ HTTP response ที่เหมาะสม

ข้อห้าม:
- ห้าม parse payload ก่อน verify ถ้า source platform ต้อง verify raw body
- ห้ามตอบ error ละเอียดจนเปิดเผยภายในระบบ

---

## Phase 2 — Build Trigger Parsing

ทำ:
1. สร้าง parser module
2. normalize input
3. ตรวจ trigger ที่เข้าเงื่อนไข
4. คืนค่า object กลางที่ระบบใช้ต่อได้

Parser ต้องแยกจาก controller/receiver อย่างชัดเจน

---

## Phase 3 — Build Metadata Enrichment

ทำ:
- เรียก source API เมื่อจำเป็น
- เก็บข้อมูลประกอบ เช่น sender/source/container
- มี fallback
- failure บางส่วนต้องไม่ทำให้ workflow ทั้งหมดล้ม ถ้า Requirement ไม่ได้บังคับ

---

## Phase 4 — Build Duplicate Prevention

ทำ:
- ใช้ external event id หรือ idempotency key
- ตรวจซ้ำก่อนสร้างรายการใหม่
- พฤติกรรมเมื่อพบรายการซ้ำต้องสม่ำเสมอ

---

## Phase 5 — Build Central Storage Integration

ทำ:
- เขียนข้อมูลตาม schema ที่ Requirement กำหนด
- ใช้ config/environment ไม่ hardcode
- มี error handling
- มี test mapping

---

## Phase 6 — Build Dashboard API

สร้าง endpoint อย่างน้อย:
- List
- Detail
- Update allowed fields only

ต้องมี:
- Authentication/Authorization
- Validation
- Pagination
- Search/filter
- Safe error responses

---

## Phase 7 — Build Dashboard UI

หน้าอย่างน้อย:
1. Login
2. List table
3. Edit modal/detail

ฟีเจอร์อย่างน้อย:
- Search
- Filters
- Pagination
- Status badge
- Save feedback
- Loading state
- Empty state
- Error state

---

## Phase 8 — Testing

ต้องมี:
- Unit tests
- Integration tests ที่คุ้มค่า
- Test parser
- Test auth
- Test update restrictions
- Test duplicate prevention
- Test storage mapping

---

## Phase 9 — Documentation and Deployment

README ต้องมี:
- ระบบทำอะไร
- Architecture
- Setup
- Environment variables / secrets
- Local development
- Test
- Deploy
- API summary
- Usage guide
- Known limitations

---

# 6. Coding Standards

ผู้พัฒนาต้อง:
- แยก business logic ออกจาก route/controller
- ใช้ type/model ที่ชัดเจน
- หลีกเลี่ยงฟังก์ชันยาวเกินไป
- หลีกเลี่ยง hardcode
- ใช้ environment variables
- ตั้งชื่อ function/variable ให้สื่อความหมาย
- มี error class หรือ error category ที่เหมาะสม
- log อย่างปลอดภัย

---

# 7. Configuration and Secrets

ตัวอย่างค่าที่มักต้องมี:

```text
SOURCE_PLATFORM_SECRET=
SOURCE_PLATFORM_ACCESS_TOKEN=
CENTRAL_STORAGE_ID=
CENTRAL_STORAGE_NAME=
DASHBOARD_USERNAME=
DASHBOARD_PASSWORD_HASH=
SESSION_SECRET=
TIMEZONE=
```

หลักการ:
- secret จริงต้องอยู่ใน secret manager / env / worker secret
- ห้าม commit ลง git
- มี `.env.example` หรือ `.dev.vars.example`
- README ต้องระบุวิธีตั้งค่า

---

# 8. Authentication Guidance for Dashboard

ถ้า Requirement ไม่ระบุทางเลือกอื่น ให้ implement อย่างใดอย่างหนึ่ง:

## แบบง่าย
- Username / Password
- Password hash
- HttpOnly secure cookie
- Session expiration
- Logout

## แบบองค์กร
- SSO
- Access Gateway
- Identity Provider

เลือกให้ตรงกับ context โปรเจกต์จริง

---

# 9. API Design Guidance

API ควร:
- ใช้ resource naming ที่สม่ำเสมอ
- แยก public webhook endpoint ออกจาก protected dashboard endpoint
- Validate body อย่างเข้ม
- Reject unsupported fields
- Return response ที่ predictable

ตัวอย่าง:
```http
POST /webhook
GET /api/items
GET /api/items/:id
PATCH /api/items/:id
POST /api/auth/login
POST /api/auth/logout
GET /api/auth/me
```

---

# 10. Update Restriction Pattern

Dashboard update ต้องทำตามหลักนี้:

```text
Read-only fields:
- source data
- trigger data
- external ids
- sender metadata

Editable fields:
- status
- assignee
- note
```

API ต้องไม่ยอมให้ client เขียนทับข้อมูลต้นทาง

---

# 11. Error Handling Standards

ต้องแยก error อย่างน้อย:
- invalid signature
- unsupported event
- duplicate event
- storage failure
- external API failure
- auth failure
- validation failure

Error ต่อ public:
- สั้น
- ปลอดภัย
- ไม่เปิดเผย secret

Error log ภายใน:
- พอ debug ได้
- ไม่รั่วข้อมูลลับ

---

# 12. Testing Checklist

## Intake
- Valid request
- Invalid signature
- Unsupported event ignored
- Trigger recognized
- Trigger ignored when not matching
- Duplicate event handling
- Metadata fallback

## Dashboard
- Login successful
- Login rejected
- Session required
- List items
- Search/filter
- Update allowed fields
- Reject invalid status
- Reject non-editable field update

---

# 13. Definition of Done

งานถือว่าเสร็จเมื่อ:
1. Build ผ่าน
2. Test ผ่าน
3. Deploy ได้
4. README ครบ
5. Secure receiver ใช้งานได้
6. Trigger parser ทำงานตรง Requirement
7. Duplicate prevention ทำงาน
8. Central storage เขียน/อ่านได้
9. Dashboard ใช้งานได้
10. Dashboard แก้เฉพาะฟิลด์ที่อนุญาต
11. ไม่มี secret จริงใน repo
12. ผู้ใช้หรือทีมติดตั้งต่อได้จากเอกสาร

---

# 14. สิ่งที่ผู้พัฒนาต้องไม่ทำเองโดยไม่ได้รับคำสั่ง

- เพิ่ม AI
- เพิ่ม notifications
- เพิ่ม analytics ขั้นสูง
- เพิ่ม multi-tenant
- เพิ่ม role ซับซ้อน
- เปลี่ยน storage
- เปลี่ยน source platform
- เปลี่ยน flow หลักของระบบ

---

# 15. Recommended Commit Plan

ตัวอย่าง:
1. `init project structure`
2. `implement secure webhook receiver`
3. `add trigger parser and mapping`
4. `add metadata enrichment and duplicate prevention`
5. `integrate central storage`
6. `build dashboard api`
7. `build dashboard ui`
8. `add tests and docs`

---

# 16. Final Report Expected from Developer

เมื่อทำเสร็จ ให้รายงาน:
- สิ่งที่พัฒนาเสร็จ
- Architecture ที่ใช้งานจริง
- Endpoint ที่มี
- Environment variables / secrets
- คำสั่งติดตั้ง
- คำสั่ง run/test/deploy
- Limitations
- งานต่อยอดที่ควรทำภายหลัง
