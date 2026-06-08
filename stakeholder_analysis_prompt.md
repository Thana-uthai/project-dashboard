# PROMPT: Stakeholder Power-Interest Analysis Dashboard

ใช้ prompt นี้กับ Claude ใน Project ใหม่ เพื่อสร้าง Stakeholder Analysis ในรูปแบบเดียวกัน

---

## วิธีใช้

1. คัดลอก prompt ด้านล่างทั้งหมด
2. วางใน Claude chat ใหม่ (ไม่ต้องอยู่ใน project เดิม)
3. Claude จะเริ่มถามข้อมูลโครงการและ stakeholder ของคุณก่อน
4. ตอบคำถามตามที่รู้ — Claude จะสร้าง dashboard และไฟล์แก้คะแนนให้อัตโนมัติ

---

## PROMPT (คัดลอกตั้งแต่บรรทัดถัดไป)

---

ช่วยสร้าง **Stakeholder Power-Interest Analysis Dashboard** สำหรับโครงการของฉัน โดยออกมาเป็น 2 ไฟล์ HTML ที่ทำงานร่วมกัน

### ไฟล์ที่ต้องสร้าง

**ไฟล์ 1: `stakeholder_grid.html`** — กระดาน Power-Interest Grid แบบ interactive
- แบ่ง 4 quadrant: บน-ซ้าย "เข้าใจและรักษา" / บน-ขวา "ดึงร่วม — สำคัญสุด" / ล่าง-ซ้าย "ติดตาม" / ล่าง-ขวา "แจ้งข้อมูลสม่ำเสมอ"
- แกน X = Interest (1–5, ซ้ายต่ำ-ขวาสูง), แกน Y = Power (1–5, ล่างต่ำ-บนสูง)
- **สีแทนกลุ่มย่อย** (เช่น พื้นที่ปฏิบัติการ / องค์กร / สังกัด — ตามที่โครงการกำหนด)
- **ไอคอน/สัญลักษณ์แทนประเภท stakeholder** (เช่น ผู้บริหาร / เจ้าหน้าที่ / วิชาการ / ชุมชน)
- ปุ่มกรองด้านบน: "ทุกกลุ่ม" และแยกตามกลุ่มย่อย
- hover แต่ละจุดเพื่อดูชื่อ + Power + Interest
- legend สี + สัญลักษณ์ อยู่ด้านขวา
- ปุ่มลิงก์ไปยัง `stakeholder_scores.html`
- อ่านค่าจาก localStorage key `pi-scores` ถ้ามี

**ไฟล์ 2: `stakeholder_scores.html`** — หน้าแก้ไขคะแนน
- แบ่ง block ตามกลุ่มย่อย พร้อมสีขอบตรงกัน
- แต่ละ stakeholder มี slider 2 แถว: Power (1–5 step 0.5) และ Interest (1–5 step 0.5)
- แสดงตัวเลขกำกับ slider แบบ real-time
- ปุ่ม "บันทึก" → save ลง localStorage key `pi-scores` เพื่อส่งกลับไป grid
- ปุ่มกลับ dashboard

### Visual Design Spec (ใช้ให้ตรงกันทุกไฟล์)

```css
font-family: 'IBM Plex Sans Thai', sans-serif (Google Fonts)
font-family-mono: 'IBM Plex Mono'
background: #F7F6F3
white: #FFFFFF
border: #E0DDD6
text: #1A1A1A
text-2: #5A5A5A
text-3: #9A9890
border-radius: 12px (card), 8px (small)
```

**สีสำหรับกลุ่มย่อย** — ให้ Claude เลือกจากชุดนี้ตามจำนวนกลุ่ม:
- เขียว: `#1A7A56` / พื้น `#E6F5EE`
- น้ำเงิน: `#1A5FA8` / พื้น `#E6F0FB`
- ส้มอิฐ: `#B05000` / พื้น `#FFF0E6`
- ม่วง: `#7C5BBF` / พื้น `#F0ECFF`
- แดงเข้ม: `#A8221A` / พื้น `#FBE9E9`

**ตำแหน่งจุดใน grid:** คำนวณจากคะแนน
```
x% = 8 + (interest - 1) / 4 * 84
y% = 8 + (5 - power) / 4 * 84
```

### Node Design Spec — วงกลม + ไอคอน + ชื่อ

แต่ละ node ประกอบด้วย 3 ส่วนซ้อนกันในแนวตั้ง จัดกึ่งกลางแนวนอน:

```
        ┌─────────┐
        │  emoji  │  ← วงกลม (avatar)
        └─────────┘
        │  label  │  ← ชื่อ (text badge)
```

**1. วงกลม (avatar)**
```css
width: 36px
height: 36px
border-radius: 50%
background: [สีของกลุ่มย่อย เช่น #1A7A56]
display: flex
align-items: center
justify-content: center
font-size: 16px          /* ขนาด emoji */
border: 2.5px solid #FFFFFF
box-shadow: 0 0 0 1.5px [สีกลุ่มย่อย]60  /* วงแหวนรอบนอก ความโปร่ง 40% */
```

- สีพื้นหลังวงกลม = สีของกลุ่มย่อย (ไม่ใช่ประเภท)
- ไอคอน emoji = แทนประเภท stakeholder ตามตารางด้านล่าง
- เมื่อ hover: `transform: scale(1.15)` transition 0.15s

**2. ชื่อ (label badge)**
```css
font-size: 11px
font-weight: 500
white-space: nowrap
padding: 2px 6px
border-radius: 4px
background: #FFFFFF
border: 0.5px solid [สีกลุ่มย่อย]50   /* โปร่ง 30% */
color: [สีกลุ่มย่อย]                   /* ตัวอักษรสีเดียวกับกลุ่ม */
margin-top: 3px
```

- ข้อความ = ชื่อ/ตำแหน่งย่อ (ไม่เกิน 14 ตัวอักษร — ถ้าเกินให้ตัดและใส่ "…")
- ไม่ขึ้นบรรทัดใหม่ (white-space: nowrap)
- อยู่ใต้วงกลม ไม่ใช่ข้างๆ

**3. โครงสร้าง HTML ของ node หนึ่งตัว**
```html
<div class="node" style="position:absolute; transform:translate(-50%,-50%);
     left:[x%]; top:[y%]; display:flex; flex-direction:column;
     align-items:center; gap:3px; cursor:default; z-index:5;">

  <div class="avatar" style="width:36px; height:36px; border-radius:50%;
       background:[site-color]; display:flex; align-items:center;
       justify-content:center; font-size:16px;
       border:2.5px solid #fff;
       box-shadow:0 0 0 1.5px [site-color]60;">
    [emoji]
  </div>

  <div class="node-label" style="font-size:11px; font-weight:500;
       white-space:nowrap; padding:2px 6px; border-radius:4px;
       background:#fff; border:0.5px solid [site-color]50;
       color:[site-color];">
    [ชื่อย่อ]
  </div>

</div>
```

**4. ตาราง emoji ตามประเภท stakeholder (ค่า default)**

| ประเภท | Emoji | ใช้เมื่อ |
|---|---|---|
| ผู้บริหาร / นักการเมืองท้องถิ่น | 🏛 | นายก รองนายก ผู้ว่า CEO ผู้อำนวยการ |
| เจ้าหน้าที่ / ผู้ปฏิบัติงาน | 📋 | เจ้าหน้าที่ กองสวัสดิการ ฝ่ายต่างๆ |
| สาธารณสุข / การแพทย์ | 🏥 | หมอ พยาบาล รพ.สต. อนามัย |
| วิชาการ / การศึกษา | 🎓 | อาจารย์ นักศึกษา นักวิจัย |
| ชุมชน / อาสาสมัคร | 👥 | อสม. แกนนำชุมชน ผู้ใหญ่บ้าน |
| ภาคเอกชน / ธุรกิจ | 🏢 | บริษัท ผู้ประกอบการ สปอนเซอร์ |
| ผู้รับประโยชน์ / กลุ่มเป้าหมาย | 🧑 | ผู้สูงอายุ ผู้พิการ กลุ่มเปราะบาง |
| NGO / ภาคประชาสังคม | 🤝 | มูลนิธิ องค์กรพัฒนา |
| สื่อ / การสื่อสาร | 📢 | นักข่าว PR ผู้ประสานสื่อ |

> **หมายเหตุ:** ถ้าโครงการมีประเภทที่ไม่อยู่ในตาราง ให้ Claude เลือก emoji ที่สื่อความหมายชัดที่สุด และแจ้งผู้ใช้ก่อน

**5. การจัดการ node ที่ซ้อนทับ**

ถ้า stakeholder หลายคนมีค่า Power และ Interest เท่ากัน ให้เหลื่อมตำแหน่งดังนี้:
```
node คนที่ 1: x, y (ตำแหน่งปกติ)
node คนที่ 2: x+3%, y-2%
node คนที่ 3: x-3%, y+2%
node คนที่ 4: x+4%, y+3%
```
ไม่ให้ node ซ้อนทับกันจนอ่านชื่อไม่ออก

### ขั้นตอนที่ Claude ต้องทำก่อนสร้างไฟล์

**ขั้น 1 — ถามบริบทโครงการ:**
- ชื่อโครงการคืออะไร?
- มีกลุ่มย่อยกี่กลุ่ม? (เช่น พื้นที่ปฏิบัติการ / องค์กร) และชื่ออะไรบ้าง?

**ขั้น 2 — ถามประเภท stakeholder:**
- stakeholder ในโครงการนี้มีกี่ประเภท? และแต่ละประเภทใช้สัญลักษณ์อะไร?
- (ถ้าไม่แน่ใจ Claude แนะนำตัวเลือก เช่น ผู้บริหาร / เจ้าหน้าที่ / วิชาการ / ชุมชน / เอกชน)

**ขั้น 3 — รวบรวม stakeholder รายคน:**
สำหรับแต่ละคน ถามตามธรรมชาติ:
- ชื่อ/ตำแหน่ง
- สังกัดกลุ่มย่อยไหน
- ประเภทไหน
- Power กี่คะแนน? (พร้อมถามเหตุผลสั้นๆ)
- Interest กี่คะแนน? (พร้อมถามเหตุผลสั้นๆ)

**หลักการให้คะแนน Power (ใช้ reference นี้):**
- 5 = ชี้ขาดโครงการได้คนเดียว (นายก / CEO / ผู้ว่า)
- 4 = มีอำนาจสูง ผลักดันหรือขัดขวางได้ (รองนายก / ผู้อำนวยการ)
- 3 = ดำเนินการได้แต่ไม่ชี้ขาด (เจ้าหน้าที่ระดับกลาง)
- 2 = มีผลทางอ้อม (นักศึกษา / อสม. / อาสาสมัคร)
- 1 = ผู้รับประโยชน์ ไม่มีอำนาจในกระบวนการ

**หลักการให้คะแนน Interest:**
- 5 = ได้รับผลกระทบโดยตรง หรือผลักดันเต็มที่
- 4 = เห็นประโยชน์ชัด มีส่วนร่วมในกิจกรรม
- 3 = รู้เรื่องและมีความเห็น แต่ไม่ได้ผลักดัน
- 2 = รู้เรื่องแต่ไม่ได้ริเริ่ม
- 1 = ไม่รู้เรื่อง หรือไม่เกี่ยวข้อง

**ขั้น 4 — สร้างไฟล์** ทั้ง 2 ไฟล์พร้อมกัน

---

### หมายเหตุสำหรับ Claude

- ถามทีละขั้นตอน ไม่ต้องถามทุกอย่างพร้อมกัน
- ถ้ากลุ่มย่อยมีมากกว่า 5 กลุ่ม ให้รวมสีหรือใช้ความเข้มต่าง
- ถ้า stakeholder คนเดียวกันอยู่หลายกลุ่มย่อย ให้สร้าง node แยก
- ตำแหน่ง node ที่ค่าเท่ากันให้เหลื่อมกันเล็กน้อย (±2%) ไม่ให้ซ้อนทับ
- ใช้ localStorage key `pi-scores` เป็น JSON array ของ node objects เพื่อส่งค่าระหว่างไฟล์

---

*Prompt นี้มาจากโครงการหลักสูตรขบวนการชุมชน R69-00219 สถาบันอาศรมศิลป์*
