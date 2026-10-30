# ⛺ Camp Vote — โหวตวันไปแคมป์กับแก๊ง

เว็บโหวตวันไปแคมป์ปิ้ง เป็นไฟล์เดียว (`index.html`) ดีไซน์ขาวคลีนสไตล์ Apple เปิดใช้ได้ทันที ไม่ต้องติดตั้งอะไร

- ใส่ชื่อ แล้วเลือกวันที่ว่างได้หลายวัน
- ใส่ชื่อเดิมซ้ำ จะโหลดโหวตเดิมมาให้แก้ แล้วบันทึกทับ
- ผลโหวตอัปเดตแบบเรียลไทม์ (Firebase Firestore) พร้อมไฮไลต์วันที่คนว่างตรงกันมากที่สุด
- ถ้ายังไม่ใส่ค่า Firebase จะใช้ **โหมดทดลอง** เก็บข้อมูลไว้ใน `localStorage` ของเครื่องตัวเอง

## 1. ตั้งค่า Firebase (ฐานข้อมูล)

1. เข้า <https://console.firebase.google.com> → สร้างโปรเจกต์ใหม่
2. เมนู **Build → Firestore Database** → Create database
3. **Project settings → Your apps → เพิ่ม Web app (`</>`)** → คัดลอกค่า `firebaseConfig`
4. เปิด `index.html` แล้ววางค่าลงใน `FIREBASE_CONFIG` ด้านบนของสคริปต์
5. ตั้ง **Firestore Rules** (ตัวอย่างสำหรับกลุ่มเพื่อน ใครมีลิงก์ก็โหวตได้):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /votes/{voteId} {
      allow read: if true;
      allow delete: if true;
      allow create, update: if request.resource.data.name is string
        && request.resource.data.name.size() > 0
        && request.resource.data.name.size() <= 30
        && request.resource.data.dates is list
        && request.resource.data.dates.size() <= 50;
    }
  }
}
```

> ค่า `apiKey` ของ Firebase Web ใส่ในโค้ดสาธารณะได้ตามปกติ ความปลอดภัยของข้อมูลมาจาก Rules ด้านบน

## 2. เปิดเว็บด้วย GitHub Pages

1. ไปที่ repo บน GitHub → **Settings → Pages**
2. Source: **Deploy from a branch** → เลือก branch `main` และโฟลเดอร์ `/ (root)`
3. รอสักครู่ จะได้ลิงก์ `https://<username>.github.io/<repo>/` ส่งให้เพื่อนได้เลย

## แก้ไขวันที่ให้เลือก

แก้ array `MONTHS` ใน `index.html` ได้เลย แต่ละวันมี `id` (ห้ามซ้ำ), `days` (ข้อความที่แสดง) และ `note` (ไม่ใส่ก็ได้)

## โครงสร้างข้อมูลใน Firestore

collection `votes` — 1 document ต่อ 1 คน, document ID = ชื่อตัวพิมพ์เล็กแบบ encode

| ฟิลด์ | ชนิด | คำอธิบาย |
|---|---|---|
| `name` | string | ชื่อที่แสดง |
| `dates` | array | id ของวันที่ว่าง |
| `updatedAt` | timestamp | เวลาบันทึกล่าสุด (serverTimestamp) |
