# ตั้งค่า Firebase ให้ห้องออฟฟิศแชร์กันได้จริงบน GitHub Pages

ใช้ **Firebase Firestore (แผนฟรี Spark)** เป็นฐานข้อมูลกลาง ทุกคนที่เปิดเว็บจะเห็นตัวละคร
สถานะ และประกาศของกันและกัน**แบบเรียลไทม์ทันที** ไม่ต้องรีเฟรชหน้าเลย — และยังใช้กับ
GitHub Pages (static hosting) ได้ปกติ เพราะ Firebase ทำงานฝั่ง client ล้วน ๆ

## ขั้นตอนที่ 1: สร้างโปรเจกต์ Firebase (ฟรี)

1. ไปที่ https://console.firebase.google.com
2. กด **Add project** → ตั้งชื่อโปรเจกต์ (เช่น `qa-office`) → ปิด Google Analytics ก็ได้ (ไม่จำเป็น) → Create project

## ขั้นตอนที่ 2: เปิดใช้งาน Firestore Database

1. ในเมนูซ้าย ไปที่ **Build > Firestore Database**
2. กด **Create database**
3. เลือก **Start in production mode** (จะตั้ง security rules เองในขั้นตอนถัดไป)
4. เลือก region ที่ใกล้ที่สุด (เช่น `asia-southeast1`) → Enable

## ขั้นตอนที่ 3: ตั้งค่า Security Rules

ในหน้า Firestore Database ไปที่แท็บ **Rules** แล้ววางโค้ดนี้แทนของเดิม:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /office/{docId} {
      allow read: if true;
      allow write: if true;
    }
    match /announcements/{docId} {
      allow read: if true;
      allow write: if request.resource.data.text is string
                    && request.resource.data.text.size() < 200;
    }
  }
}
```

กด **Publish**

> ⚠️ กติกานี้เปิดให้ทุกคนอ่าน/เขียนได้อย่างอิสระ (เหมือนของเดิมที่ "แก้ไขได้แค่ของตัวเอง" ควบคุมแค่
> ในหน้าเว็บ ไม่ใช่ระดับเซิร์ฟเวอร์) เหมาะกับใช้ภายในทีมที่ไว้ใจกัน ถ้าต้องการความปลอดภัยเข้มขึ้น
> (ล็อกด้วยบัญชีจริง) ต้องเพิ่ม Firebase Authentication — บอกได้เลยถ้าอยากให้ช่วยทำต่อ

## ขั้นตอนที่ 4: เอาค่า config มาใส่ในไฟล์

1. ไปที่ไอคอนเฟือง (⚙️) มุมซ้ายบน → **Project settings**
2. เลื่อนลงมาที่ **Your apps** → กดไอคอน **</>** (Web)
3. ตั้งชื่อแอป (เช่น `qa-office-web`) → **ไม่ต้อง**ติ๊ก Firebase Hosting → Register app
4. จะเห็นโค้ดแบบนี้ — **copy ค่าทั้งหมด**:
   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "qa-office-xxxxx.firebaseapp.com",
     projectId: "qa-office-xxxxx",
     storageBucket: "qa-office-xxxxx.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef123456"
   };
   ```
5. เปิดไฟล์ `index.html` ที่แนบมา หาบรรทัด `const firebaseConfig = {`
   (อยู่ใกล้ ๆ ด้านบนของ `<script>` แรก) แล้ว**แทนที่ค่าทั้งหมด**ด้วยค่าที่ copy มา

## ขั้นตอนที่ 5: อัปโหลดขึ้น GitHub Pages

1. เอาไฟล์ `index.html` (ที่แก้ config แล้ว) ไปแทนที่ไฟล์เดิมในรีโปของคุณ
   (`khwanl/Qa-office-`)
2. Commit + Push ขึ้น GitHub
3. รอ 1-2 นาที ให้ GitHub Pages build ใหม่ แล้วเปิด
   https://khwanl.github.io/Qa-office-/ อีกครั้ง

## ทดสอบว่าทำงานจริง

เปิดเว็บ 2 แท็บ (หรือ 2 เครื่อง) ตั้งชื่อคนละชื่อ แต่งตัวแล้วกด "บันทึกลุค" —
อีกฝั่งควรเห็นตัวละครโผล่ขึ้นมาในห้องออฟฟิศ**ทันทีโดยไม่ต้องรีเฟรช**

## แผนฟรีของ Firebase พอไหม

Spark plan (ฟรี) ให้ 50,000 reads / 20,000 writes ต่อวัน — สำหรับทีมขนาดเล็ก-กลาง
(สิบ-หลายสิบคน) ใช้งานปกติทั้งวันสบาย ๆ ไม่มีค่าใช้จ่าย

## ⚠️ อย่าลืม: โฟลเดอร์ assets/

แพ็กนี้มีไฟล์ `assets/office-bg.jpg` (ภาพพื้นหลังห้องออฟฟิศ) — ตอน commit ขึ้น GitHub
ต้องอัปโหลดทั้งโฟลเดอร์ `assets/` ไปด้วย ไม่ใช่แค่ `index.html` ไม่งั้นพื้นหลังจะไม่ขึ้น
(เห็นแค่พื้นสีทึบแทน)

ภาพพื้นหลังนี้มาจาก Freepik (ผู้วาด: upklyak) — ถ้าดาวน์โหลดด้วยบัญชีฟรี (ไม่ได้ subscribe)
ต้องมีเครดิตแสดงบนหน้าเว็บตามเงื่อนไขของ Freepik ซึ่งใส่ไว้ให้แล้วท้ายหน้าห้องออฟฟิศ
ถ้าใช้บัญชี Premium จะลบบรรทัดเครดิตนี้ออกจาก `index.html` ก็ได้
