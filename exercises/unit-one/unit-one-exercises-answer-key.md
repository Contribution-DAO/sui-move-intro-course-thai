# เฉลยแบบฝึกหัดหน่วยที่ 1

## คำถาม 1

1. `copy` + `drop`: ถูกต้อง
2. `copy` + `key`: ไม่ถูกต้อง, `copy` ขัดแย้งกับ `key`
3. `copy` + `store`: ถูกต้อง
4. `drop` + `key`: ไม่ถูกต้อง, `drop` ขัดแย้งกับ `key`
5. `drop` + `store`: ถูกต้อง
6. `store` + `key`: ถูกต้อง
7. `copy` + `drop` + `store`: ถูกต้อง
8. `copy` + `drop` + `key`: ไม่ถูกต้อง, `drop` ขัดแย้งกับ `key`
9. `copy` + `key` + `store`: ไม่ถูกต้อง, `copy` ขัดแย้งกับ `key`
10. `drop` + `key` + `store`: ไม่ถูกต้อง, `drop` ขัดแย้งกับ `key`
11. `copy` + `drop` + `key` + `store`: ไม่ถูกต้อง, `drop` ขัดแย้งกับ `key`

## Q2

`transfer` ต้องใช้กับ object ที่มี ability `key` และ object ต้องถูกกำหนด (defined) ภายในโมดูลเดียวกับที่เรียก `transfer`

`public_transfer` ต้องใช้กับ object ที่มีทั้ง `key` และ `store` สามารถเรียกใช้จากภายนอกโมดูล ที่ object นั้นถูกกำหนดได้
