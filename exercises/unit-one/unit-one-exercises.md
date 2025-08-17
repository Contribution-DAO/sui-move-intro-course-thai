# แบบฝึกหัดบทที่ 1

## คำถาม 1

ในยูนิตนี้ เราได้แนะนำเรื่อง [abilities](../../unit-one/lessons/3_custom_types_and_abilities.md) ซึ่งเป็นหัวใจสำคัญในการกำหนดลักษณะของสินทรัพย์ (asset) ใน Move อย่างไรก็ตาม มีบางชุดของ 4 abilities ที่ไม่อนุญาตให้ใช้ร่วมกันใน Move เนื่องจากอาจนำไปสู่พฤติกรรมที่ไม่ปลอดภัยหรือขัดแย้งกันได้

จงระบุว่าการผสม abilities ต่อไปนี้ ถูกต้องหรือไม่ถูกต้อง:

1. `copy` + `drop`
2. `copy` + `key`
3. `copy` + `store`
4. `drop` + `key`
5. `drop` + `store`
6. `store` + `key`
7. `copy` + `drop` + `store`
8. `copy` + `drop` + `key`
9. `copy` + `key` + `store`
10. `drop` + `key` + `store`
11. `copy` + `drop` + `key` + `store`

สำหรับแต่ละชุดการผสมที่ไม่ถูกต้อง ให้อธิบายสั้นๆ ว่าจะเกิดอะไรขึ้นหากอนุญาตให้ใช้การจับคู่ที่ขัดแย้งนั้น

_คำใบ้: คุณสามารถทดลอง combinations เหล่านี้โดยใช้ตัว compiler_

## คำถาม 2

ความแตกต่างระหว่างๆ `sui::transfer::transfer` และ `sui::transfer::public_transfer` คืออะไร?