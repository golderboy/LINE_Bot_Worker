# REQUIREMENTS.md
# Generic Event Intake → Secure Receiver → Central Tracking Dashboard

สถานะเอกสาร: Generic Template  
วัตถุประสงค์: ใช้เป็นแม่แบบสำหรับพัฒนาระบบรับข้อมูลจากช่องทางภายนอกแบบอัตโนมัติ แล้วนำไปจัดเก็บและติดตามผลผ่าน Dashboard

---

# 1. แนวคิดของระบบ

พัฒนาระบบกลางสำหรับรับข้อความหรือเหตุการณ์จากแหล่งข้อมูลภายนอก เช่น

- กลุ่มแชต
- Messaging Platform
- Form Submission
- Webhook จากระบบภายนอก
- Event Notification จากบริการอื่น

จากนั้นระบบต้อง:

1. รับข้อมูลผ่าน Webhook หรือ Event Receiver
2. ตรวจสอบความถูกต้องและความปลอดภัยของ Request
3. คัดกรองเฉพาะเหตุการณ์ที่เข้าเงื่อนไข
4. แปลงข้อมูลให้อยู่ในรูปแบบรายการติดตามงาน
5. บันทึกลงแหล่งข้อมูลกลาง
6. แสดงผ่าน Dashboard ให้ทีมติดตามและอัปเดตสถานะได้

---

# 2. Workflow หลักของระบบ

```text
External Source / Messaging Channel
        ↓
Secure Webhook Receiver
        ↓
Event Validation + Signature Verification
        ↓
Rule Parser / Trigger Matcher
        ↓
Metadata Enrichment
        ↓
Duplicate Prevention
        ↓
Central Storage
        ↓
Web Dashboard
        ↓
Team updates only allowed operational fields
```

---

# 3. เป้าหมายของระบบ

ระบบต้องช่วยให้ทีม:

- ไม่พลาดรายการสำคัญจากหลายแหล่งข้อมูล
- เห็นรายการที่ต้องติดตามในจุดเดียว
- ตรวจสอบย้อนหลังได้
- เปลี่ยนสถานะหรือบันทึกความคืบหน้าได้
- เริ่มต้นง่ายและต่อยอดได้ในอนาคต
- แยกบทบาทระหว่าง “ข้อมูลที่ระบบรับมา” กับ “ข้อมูลที่ทีมอัปเดตเอง”

---

# 4. Scope ของระบบ

ระบบแบ่งออกเป็น 2 Phase หลัก

---

## 4.1 Phase 1 — Automated Intake

สร้างระบบรับเหตุการณ์จากภายนอกและบันทึกรายการเข้าสู่แหล่งข้อมูลกลาง

### สิ่งที่ต้องทำ
1. รับ Webhook/Event จากแหล่งภายนอก
2. ตรวจสอบ Request Signature หรือรูปแบบการยืนยันตัวตนที่แหล่งข้อมูลนั้นรองรับ
3. ประมวลผลเฉพาะ Event ที่เข้าเงื่อนไข
4. ตรวจจับข้อความ/คำสั่ง/Trigger ที่กำหนด
5. ดึง Metadata ที่จำเป็นจากแหล่งข้อมูลภายนอกเมื่อทำได้
6. ป้องกันการบันทึก Event ซ้ำ
7. บันทึกข้อมูลลง Central Storage
8. ตอบกลับ HTTP Status อย่างเหมาะสม
9. มี logging และ error handling ที่เหมาะสม
10. มีเอกสาร deploy และ configuration

---

## 4.2 Phase 2 — Tracking Dashboard

สร้าง Web Dashboard เพื่ออ่านข้อมูลจาก Central Storage และอัปเดตเฉพาะฟิลด์ที่อนุญาต

### สิ่งที่ต้องทำ
1. หน้า Login หรือกลไกป้องกันการเข้าถึง Dashboard
2. หน้าแสดงรายการทั้งหมด
3. Search / Filter / Pagination
4. หน้า Detail หรือ Modal แก้ไขรายการ
5. ฟิลด์ต้นทางทั้งหมดเป็น Read-only
6. อนุญาตให้ทีมแก้ไขเฉพาะฟิลด์เชิงปฏิบัติงาน เช่น
   - status
   - assignee
   - note
7. API สำหรับอ่านรายการและอัปเดตฟิลด์ที่อนุญาต
8. Validation ค่าที่สามารถแก้ไขได้
9. มี error/loading/empty states
10. มีเอกสาร deploy และใช้งาน

---

# 5. นิยามส่วนที่ต้อง Configurable

เอกสารนี้เป็น Template กลาง ดังนั้นระบบจริงต้องกำหนดค่าต่อไปนี้ในโปรเจกต์ย่อย

## 5.1 Source Definition
ระบุว่า Event มาจากที่ใด เช่น
```text
[SOURCE_PLATFORM]
```

ตัวอย่าง:
- Messaging Platform
- Notification Service
- External Application Webhook

## 5.2 Trigger Rules
ระบุเงื่อนไขว่าข้อความหรือ Event แบบใดต้องถูกจับ

ตัวอย่าง placeholder:
```text
[TRIGGER_1]
[TRIGGER_2]
[TRIGGER_3]
```

แนวทางทั่วไป:
- จับเฉพาะข้อความที่ขึ้นต้นด้วย Trigger
- หรือจับเฉพาะ Event Type ที่กำหนด
- หรือจับเฉพาะ payload field ที่มีค่าเข้าเงื่อนไข

## 5.3 Central Storage
ระบุแหล่งข้อมูลกลาง เช่น
```text
[CENTRAL_STORAGE]
```

ตัวอย่าง:
- Spreadsheet
- Database
- Airtable
- Internal API

## 5.4 Editable Operational Fields
ระบุฟิลด์ที่ทีมสามารถแก้ไขได้จาก Dashboard

ค่าเริ่มต้นที่แนะนำ:
```text
status
assignee
note
```

---

# 6. Phase 1 Requirements — Secure Intake

---

## 6.1 Webhook Endpoint

ระบบต้องมี endpoint สำหรับรับข้อมูลภายนอก เช่น

```http
POST /webhook
```

หรือชื่ออื่นตามโปรเจกต์จริง

---

## 6.2 Request Verification

ก่อนประมวลผล payload ต้องตรวจสอบความถูกต้องของ request เสมอ

ระบบต้องรองรับอย่างน้อยหนึ่งแนวทาง:
- Signature verification
- Shared secret verification
- HMAC
- Token verification
- Scheme ที่ official API ของ source นั้นกำหนด

ข้อกำหนด:
- ต้อง verify จาก raw body เมื่อ source platform กำหนดแบบนั้น
- ห้าม parse และประมวลผลข้อมูลก่อน verify สำเร็จ
- Request ที่ไม่ถูกต้องต้อง reject ด้วย HTTP status ที่เหมาะสม

---

## 6.3 Supported Event Filtering

ระบบต้องประมวลผลเฉพาะ Event ที่กำหนดเท่านั้น

ตัวอย่างลักษณะ:
```text
event.type = [SUPPORTED_EVENT_TYPE]
message.type = [SUPPORTED_MESSAGE_TYPE]
source.type = [SUPPORTED_SOURCE_TYPE]
```

Event ที่ไม่เข้าเงื่อนไข:
- ignore
- ตอบ success ตามแนวทางของ webhook source ถ้าเหมาะสม
- ห้ามสร้างรายการใหม่

---

## 6.4 Trigger Parsing

ระบบต้องมี Rule Parser แยกต่างหาก เพื่อคัดกรอง Trigger ที่ต้องการ

แนวทางเริ่มต้นที่แนะนำ:
- Trim whitespace ด้านหน้า
- จับเฉพาะ trigger ที่ขึ้นต้นข้อความ
- รองรับรายละเอียดหลายบรรทัด
- รองรับกรณี Trigger ไม่มีรายละเอียด
- ไม่จับ trigger ที่ปรากฏกลางประโยคถ้า requirement ไม่อนุญาต

ตัวอย่างเชิง Template:
```text
[TRIGGER] [DETAIL]
```

ผลลัพธ์ parser ควรคืนค่า:
```json
{
  "trigger": "[TRIGGER]",
  "detail": "[DETAIL]",
  "original_content": "[RAW_TEXT]"
}
```

---

## 6.5 Metadata Enrichment

เมื่อ Event ผ่านเงื่อนไขแล้ว ระบบควรพยายามดึง metadata ที่จำเป็นเพิ่มเติม เช่น

- source/container name
- sender/display name
- sender id
- thread/group/channel id
- timestamp
- event id

หากบาง metadata ดึงไม่ได้:
- ต้องใช้ fallback value
- ห้ามทำให้การบันทึกรายการหลักล้มเหลวโดยไม่จำเป็น

---

## 6.6 Duplicate Prevention

ระบบต้องป้องกันการสร้างรายการซ้ำในกรณี webhook redelivery หรือ event ถูกส่งซ้ำ

แนวทางทั่วไป:
- ใช้ unique event id จาก source
- หรือสร้าง idempotency key
- หรือ hash จาก payload ที่เสถียร

ต้องมีตรรกะ:
1. ตรวจสอบก่อนบันทึก
2. ถ้าเคยมีแล้ว ไม่บันทึกซ้ำ
3. ตอบ success เพื่อไม่ให้เกิด retry loop โดยไม่จำเป็น

---

## 6.7 Central Storage Schema

Central Storage ต้องรองรับข้อมูลขั้นต่ำดังนี้

| Field | ความหมาย |
|---|---|
| received_at | วันเวลาได้รับรายการ |
| trigger_type | ประเภท Trigger / ประเภทงาน |
| source_name | ชื่อแหล่งที่มา |
| detail | รายละเอียดที่แยกจากข้อความ |
| original_content | ข้อความหรือ payload หลักแบบอ่านได้ |
| status | สถานะเริ่มต้น |
| assignee | ผู้รับผิดชอบ |
| note | หมายเหตุ |
| external_event_id | รหัส event สำหรับป้องกันซ้ำ |
| source_id | รหัสแหล่งข้อมูล |
| sender_name | ชื่อผู้ส่ง |
| sender_id | รหัสผู้ส่ง |
| updated_at | เวลาอัปเดตล่าสุด ถ้ามี |

---

## 6.8 Default Values

เมื่อสร้างรายการใหม่:
```text
status = [DEFAULT_STATUS]
assignee = empty
note = empty
```

ค่าเริ่มต้นของสถานะที่แนะนำ:
```text
ใหม่
```

---

## 6.9 HTTP Responses

ตัวอย่างแนวทาง:
| กรณี | Response |
|---|---|
| Request ถูกต้องและบันทึกสำเร็จ | 200 |
| Request ถูกต้องแต่ไม่มี Event เข้าเงื่อนไข | 200 |
| Event ซ้ำ | 200 |
| Signature ไม่ผ่าน | 401 |
| Method ไม่รองรับ | 405 |
| Error ภายในที่ต้อง retry | 500 |

---

## 6.10 Security Requirements

ต้องมี:
- Secret อยู่ใน environment/secret store เท่านั้น
- ห้าม hardcode secret
- ห้าม commit secret ลง repository
- ห้าม log private key, access token, password หรือ signature เต็ม
- Error message ต่อ public ต้องไม่เปิดเผยข้อมูลภายในมากเกินไป

---

# 7. Phase 2 Requirements — Dashboard

---

## 7.1 Authentication

Dashboard ต้องไม่เปิดสาธารณะโดยไม่มีการป้องกัน

อย่างน้อยต้องมีหนึ่งทางเลือก:
- Application-level login
- SSO
- Access Gateway
- Auth Proxy
- Internal network restriction

Template เริ่มต้นให้รองรับ:
- Username / Password
- Session cookie ที่ปลอดภัย
- Logout
- API authorization

---

## 7.2 Dashboard Pages

### 7.2.1 Login Page
- Username
- Password
- Sign in button
- Error state

### 7.2.2 Main Dashboard
- Summary counts แบบไม่ซับซ้อน
- Search box
- Filter ตาม status
- Filter ตาม trigger/type
- Table รายการ
- Pagination

### 7.2.3 Edit Detail / Modal
Read-only:
- received_at
- trigger_type
- source_name
- detail
- original_content
- sender_name
- source_id
- external_event_id

Editable:
- status
- assignee
- note

---

## 7.3 Editable Fields Restriction

ระบบต้องอนุญาตให้แก้ไขเฉพาะฟิลด์ที่กำหนดใน config เท่านั้น

ค่าเริ่มต้น:
```text
status
assignee
note
```

ห้าม API รับหรืออัปเดตฟิลด์ read-only โดยเด็ดขาด

---

## 7.4 Recommended Status Values

สถานะมาตรฐานที่สามารถนำไปปรับใช้:
```text
ใหม่
รับเรื่องแล้ว
กำลังดำเนินการ
เสร็จสิ้น
```

หรือให้โปรเจกต์จริงกำหนดเองผ่าน Requirement ย่อย

---

## 7.5 API Requirements

ตัวอย่าง API:

```http
GET /api/items
GET /api/items/:id
PATCH /api/items/:id
```

`PATCH /api/items/:id`
รับเฉพาะ:
```json
{
  "status": "...",
  "assignee": "...",
  "note": "..."
}
```

---

## 7.6 Search / Filter / Pagination

Dashboard ต้องรองรับอย่างน้อย:
- Search จาก source_name, detail, assignee, note
- Filter ตาม status
- Filter ตาม trigger_type
- เรียงล่าสุดก่อน
- Pagination

---

# 8. Non-functional Requirements

ระบบควร:
- แยก module ชัดเจน
- ดูแลต่อได้
- มี error handling ที่เข้าใจง่าย
- มี test สำหรับ logic สำคัญ
- รองรับการต่อยอดในอนาคต
- UI อ่านง่าย ใช้งานได้จริง
- ไม่ทำ feature เกิน scope

---

# 9. Logging Requirements

อนุญาตให้ log:
- request id
- external event id
- trigger type
- source id
- response code
- error category

ห้าม log:
- secrets
- raw credentials
- private keys
- full session token
- full authorization header

---

# 10. Testing Requirements

อย่างน้อยต้องมี test สำหรับ:

## Phase 1
- Request verification pass/fail
- Parser จับ trigger ถูกต้อง
- Parser ไม่จับข้อความผิดเงื่อนไข
- Mapping เป็น storage schema
- Duplicate prevention
- Fallback metadata

## Phase 2
- Login success/fail
- Session check
- List API
- Filter/search logic
- Validation editable fields
- Update เฉพาะ field ที่อนุญาต
- Read-only fields ไม่ถูกแก้

---

# 11. สิ่งที่ไม่อยู่ใน Scope โดยค่าเริ่มต้น

เว้นแต่โปรเจกต์จริงระบุเพิ่ม

- AI Classification
- Automatic assignment
- Notifications
- SLA Engine
- File upload
- Full audit trail
- Multi-tenant architecture
- Role-based permission ละเอียด
- Mobile App
- Advanced analytics

---

# 12. Acceptance Criteria

ระบบถือว่าสำเร็จเมื่อ:

## Phase 1
1. รับ webhook/event ได้
2. Verify request ได้
3. จับ trigger ที่กำหนดได้
4. ไม่จับข้อความที่ไม่เข้าเงื่อนไข
5. บันทึกรายการลง Central Storage ได้
6. กัน event ซ้ำได้
7. Metadata หลักถูกบันทึก
8. Error handling เหมาะสม

## Phase 2
1. Dashboard เปิดใช้งานได้
2. มี authentication
3. โหลดข้อมูลจาก Central Storage ได้
4. Search/filter/page ได้
5. แก้ได้เฉพาะ operational fields
6. ฟิลด์ต้นทางเป็น read-only
7. Validation ทำงาน
8. Test ผ่าน
9. README ครบ
10. deploy ได้จริง

---

# 13. แนวทางต่อยอดในอนาคต

ระบบนี้สามารถพัฒนาเพิ่มได้ภายหลัง เช่น:
- Notification ไปยังทีม
- SLA monitoring
- Auto assignment
- Configurable triggers
- Dashboard analytics
- AI-assisted classification
- Export report
- Integration กับระบบอื่น
