# Universe Overview — Tableau Extension

Dashboard extension สำหรับดูภาพรวมการขายแยกตาม Universe tier (ECO / MASS / PREMIUM / LUXURY) — KPI รวม, Sales by Universe, Sales by Product Group / Product Group Mer, Sales by Brand, Universe Performance (Flag_PrivateBrand mix รายเดือน), Top Products by Universe เทียบปีปัจจุบันกับปีก่อนหน้า

## โครงสร้างไฟล์

```
universe-overview/
  index.html              ไฟล์หลักของ extension (HTML + CSS + JS ในไฟล์เดียว)
  UniverseOverview.trex    ไฟล์ manifest สำหรับให้ Tableau รู้จัก extension นี้
tableau.extensions.1.latest.js   Tableau Extensions API (index.html เรียกใช้ไฟล์นี้)
```

> **หมายเหตุ:** ไฟล์ `universesample.xlsx` และ `data form ds new.xlsx` (ข้อมูลขายจริง) ถูกกันไว้ใน `.gitignore` ไม่ได้ขึ้น GitHub เพราะเป็นข้อมูลภายในและไม่จำเป็นต่อการรันตัว extension เช่นเดียวกับโฟลเดอร์ `preview/` (มีข้อมูลจริงฝังอยู่ในไฟล์ static HTML สำหรับใช้ตรวจ layout บนเครื่องเท่านั้น)

> **`index.html` ไม่มีข้อมูลตัวอย่าง/mock data ฝังอยู่เลย** — ไฟล์นี้จะแสดงผลได้ก็ต่อเมื่อรันอยู่ภายใน Tableau dashboard จริงเท่านั้น (ดึงข้อมูลสดจาก worksheet ตามสเปกในข้อ 3) ถ้าเปิดไฟล์ตรงๆด้วยเบราว์เซอร์จะเห็นแค่กรอบ layout เปล่าๆ กับข้อความแจ้งว่าต้องเปิดผ่าน Tableau — ใช้เช็คแค่โครงสร้าง/การจัดวางเท่านั้น ไม่มีตัวเลขให้ดู

---

## 1) เช็คโครงสร้าง Layout (ยังไม่ต้องต่อ Tableau)

เปิดไฟล์ `universe-overview/index.html` ตรงๆ ด้วยเบราว์เซอร์ (ดับเบิลคลิก หรือลากไฟล์เข้าเบราว์เซอร์) — จะเห็นข้อความ "This extension only renders inside a Tableau dashboard." เพื่อยืนยันว่าไฟล์โหลดไม่มี error

ถ้าต้องการดูข้อมูลจริงต้องเปิดผ่าน Tableau ตามข้อ 3

---

## 2) Deploy ขึ้น GitHub Pages

ขั้นตอนนี้ทำครั้งเดียวเพื่อให้ Tableau (ซึ่งต้องโหลด extension จาก URL แบบ `https://`) เข้าถึงไฟล์ `index.html` ได้

1. เข้า repo บน GitHub: `https://github.com/warinda-nor/port_product_universe`
2. ไปที่ **Settings → Pages**
3. ที่ **Source** เลือก **Deploy from a branch**
4. เลือก Branch เป็น **main** และ Folder เป็น **/ (root)** แล้วกด **Save**
5. รอ 1–2 นาที ให้ GitHub Pages build เสร็จ แล้วเข้าไปเช็คที่:
   ```
   https://warinda-nor.github.io/port_product_universe/universe-overview/index.html
   ```
   ถ้าเห็นข้อความ "This extension only renders inside a Tableau dashboard." แปลว่า deploy สำเร็จ

> URL ด้านบนต้องตรงกับค่าที่อยู่ใน `universe-overview/UniverseOverview.trex` (แท็ก `<source-location><url>`) เป๊ะๆ — ถ้าเปลี่ยนชื่อ repo หรือ path ต้องแก้ในไฟล์ `.trex` ให้ตรงกันด้วย

---

## 3) ติดตั้งใช้งานใน Tableau Desktop

1. เปิด Tableau Desktop แล้วเปิด Dashboard ที่ต้องการใส่ extension
2. ลาก object **Extension** จากแผง Objects มาวางในตำแหน่งที่ต้องการ
3. เลือก **My Extensions → Access Local Extensions** แล้วเลือกไฟล์ `universe-overview/UniverseOverview.trex`
   (หรือถ้า deploy ผ่าน GitHub Pages แล้ว จะสามารถแชร์ไฟล์ `.trex` นี้ให้คนอื่นใช้ได้เลยโดยไม่ต้องมีไฟล์ index.html อยู่ในเครื่อง เพราะ extension จะไปโหลดจาก URL บน GitHub Pages โดยตรง)
4. Dashboard ต้องมี Worksheet ทั้งหมด **2 ตัว** ตามสเปกด้านล่าง — **ตั้งชื่อ Worksheet เป็นอะไรก็ได้ตามใจ** เพราะ extension จะดูจาก **field ที่มีอยู่ใน worksheet นั้นๆ** เพื่อแยกว่าอันไหนคือ Detail อันไหนคือ Trend (ไม่ได้ดูจากชื่อ worksheet):
   - มี field **`Article Id`** → ถือเป็น **Detail**
   - มี field **`Day Month`** แต่ **ไม่มี** `Article Id` → ถือเป็น **Trend**
5. ถ้า field ที่ต้องใช้ขาดไปฝั่งใดฝั่งหนึ่ง extension จะโชว์ banner สีแดงบอกชื่อ field ที่ขาดแบบเจาะจง ไม่ใช่หน้าจอเปล่าๆ — ให้แก้ชื่อ field ใน Tableau (หรือแก้ค่าคงที่ `DETAIL_FIELDS`/`TREND_FIELDS` ใน `index.html`) ให้ตรงกัน

### Calculated Field ที่ต้องสร้างใน Tableau (CY/LY คำนวณสำเร็จรูปมาให้ extension เลย)

Net Inc Tax และ Sales Qty ทุกตัวต้อง split เป็นคอลัมน์ CY (Current Year) กับ LY (ปีก่อน ช่วงเดียวกัน) แยกกัน โดยขับด้วย Parameter `Start Date` / `End Date` บน Dashboard แล้ว extension จะ sum แต่ละคอลัมน์ตรงๆ ไม่มีการคำนวณช่วงวันที่เองอีกต่อไป (ช่วงวันที่/ปีที่แสดงบนหน้าจอ extension เอง **อนุมานจาก min/max ของ `Day Month` ที่พบจริงใน worksheet Trend** ไม่ได้อ่านจาก Parameter ตรงๆ ผ่าน Parameters API — ต่างจาก `port_vendor_fitting` ที่อ่าน Parameter ตรง ดูหัวข้อ "ข้อจำกัด" ด้านล่าง):

| Field ที่ต้องสร้าง | แนวคิดสูตร (ตัวอย่าง) |
|---|---|
| `Net Inc Tax - CY` | `IF [Time Date] >= [Start Date] AND [Time Date] <= [End Date] THEN [Net Inc Tax] END` |
| `Net Inc Tax - LY` | เหมือนกันแต่ใช้ `DATEADD('year', -1, [Start Date])` / `DATEADD('year', -1, [End Date])` |
| `Sales Qty - CY` / `Sales Qty - LY` | สูตรแบบเดียวกัน ใช้ `[Sales Qty]` |

**`Day Month`** — calculated field ใหม่ ใส่ใน worksheet "Trend" เท่านั้น เป็นวันที่รูปแบบ `YYYY-MM-DD` (หรือ date type ปกติ) ใช้เป็นแกนเวลาของกราฟ Trend (extension จะ group ให้เป็นรายเดือนเองจากวันที่) — worksheet "Detail" **ไม่ต้องมี field วันที่เลย**

### สเปก field ที่แต่ละ Worksheet ต้องมี

**Worksheet "Detail"** — grain รายบทความ (ต่อ Article Id) ไม่มีวันที่

| Field ใน Tableau | ใช้ทำอะไร |
|---|---|
| `Article Id` | นับจำนวน SKU / ใช้แยกว่านี่คือ worksheet Detail |
| `Article Name Th` | ชื่อสินค้าในตาราง Top Products |
| `Sls Grp Desc` | Sales Channel |
| `Product Group` | Sales by Product Group |
| `Product Group Mer` | Sales by Product Group Mer — สลับดูได้ผ่าน Toggle บนการ์ดเดียวกัน |
| `Mc Desc` | คอลัมน์ MC Desc ในตาราง Top Products |
| `Brand` | Sales by Brand and Universe (top 5 brand) |
| `Vendor Name` | ใช้แสดงชื่อ vendor ในตาราง Detail |
| `Universe` | Sales by Universe (ECO / MASS / PREMIUM / LUXURY — ค่าอื่น/ว่างจัดเป็น UNCLASSIFIED) |
| `Net Inc Tax - CY`, `Net Inc Tax - LY` | ยอดขาย ปีปัจจุบัน/ปีก่อน |
| `Sales Qty - CY`, `Sales Qty - LY` | จำนวนขาย ปีปัจจุบัน/ปีก่อน |

**Worksheet "Trend"** — grain รายวัน ไม่มี Article Id

| Field ใน Tableau | ใช้ทำอะไร |
|---|---|
| `Day Month` | แกนเวลาของกราฟ Trend / ใช้อนุมานช่วงวันที่ที่แสดงบนหน้าจอ / ใช้แยกว่านี่คือ worksheet Trend |
| `Sls Ofc Desc` | Sales Office |
| `Flag_PrivateBrand` | Universe Performance mix (Market Brand / Private Brand) |
| `Universe` | ใช้กรอง/สรุปยอดตาม Universe รายเดือน |
| `Net Inc Tax - CY`, `Net Inc Tax - LY` | KPI ยอดขาย + กราฟ Trend รายเดือน |
| `Sales Qty - CY`, `Sales Qty - LY` | KPI จำนวนขาย + กราฟ Trend รายเดือน |

> ชื่อ field ต้องตรงกับในตาราง **เป๊ะๆ** (ตรงตามค่าคงที่ `DETAIL_FIELDS` / `TREND_FIELDS` ท้ายไฟล์ `index.html`) ถ้าใน data source ใช้ชื่อคอลัมน์ต่างจากนี้ ให้แก้ค่าในตัวแปรเหล่านั้นให้ตรงกับ data source จริง

---

## ข้อจำกัดที่ควรรู้

- ช่วงวันที่/ปีที่แสดงบนหน้าจอ (CY/LY) **อนุมานจาก min/max ของ `Day Month` ที่พบจริงใน worksheet Trend** ไม่ได้อ่านจาก Tableau Parameter ผ่าน Parameters API ตรงๆ — ถ้าต้องการให้ label ตรงกับ Parameter `Start Date`/`End Date` แบบ `port_vendor_fitting` ต้องปรับโค้ดเพิ่ม (ยังไม่ทำในรอบนี้ตามที่ตกลงไว้)
- แถวที่ `Universe` เป็นค่าว่าง/ไม่ใช่ 4 tier มาตรฐาน จะถูกจัดเป็น `UNCLASSIFIED` และโชว์แบบลดความเด่นในตาราง/stacked bar เท่านั้น — โดนัทชาร์ตกับกริด "Universe Performance" ไม่รวม UNCLASSIFIED
- ทุกครั้งที่แก้ `universe-overview/index.html` แล้ว push ขึ้น GitHub ต้องรอ GitHub Pages build ใหม่ (ปกติ 1–2 นาที) ก่อนที่ Tableau จะเห็นเวอร์ชันล่าสุด — ถ้าไม่เห็นการเปลี่ยนแปลง ให้ลอง hard refresh หรือปิด-เปิด dashboard ใหม่
- ยังไม่ได้ทดสอบกับ Tableau Desktop จริง (`getSummaryDataReaderAsync`, การแยก Detail/Trend อัตโนมัติ, event listener refresh) — ตรวจสอบตอนติดตั้งจริงตามข้อ 3
