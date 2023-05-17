# การหุ้มวัตถุ (object wrapping)

ใน Sui Move มีหลากหลายวิธีในการซ้อนวัตถุไว้ในวัตถุอื่น วิธีแรกที่เราอยากแนะนำเรียกว่า object wrapping

มาต่อกันที่ตัวอย่างเดิมของเรา เราประกาศ `WrappableTranscript` และ wrapper ที่จะใช้งานชื่อว่า `Folder` 

```rust
struct WrappableTranscript has store {
    history: u8,
    math: u8,
    literature: u8,
}

struct Folder has key {
    id: UID,
    transcript: WrappableTranscript,
}
```

ในตัวอย่างข้างบน `Folder` จะทำการหุ้ม `WrappableTranscript` เอาไว้ และ `Folder` สามารถระบุตำแหน่งได้ด้วยไอดีของมันเนื่องจากมันมี ability `key`

## คุณสมบัติต่างๆของ Object Wrapping

สำหรับตัวแปรประเภท struct เพื่อให้มันสามารถถูกฝังอยู่ใน Sui object ได้ โดยทั่วไปแล้วจะต้องมี ability `key` และที่เพิ่มเติมจากนั้นคือต้องมี ability `store`

เมื่อวัตถุถูกหุ้มไว้แล้ว วัตถุนั้นจะไม่สามารถเข้าถึงได้โดยตรงด้วย ID ของมันอีกต่อไป แต่ตัวมันจะถือเป็นส่วนหนึ่งของวัตถุที่หุ้มมันอีกที ที่สำคัญยิ่งกว่านั้น วัตถุตัวลูกที่อยู่ข้างในจะไม่สามารถใช้ส่งเป็น argument ในการเรียกฟังก์ชั่นของ Move ได้ และทางเดียวที่จะเข้าถึงมันได้คือผ่านตัวแม่ของมัน

ด้วยคุณสมบัตินี้ การหุ้มวัตถุ (object wrapping) เป็นวิธีที่เราสามารถใช้เมื่อไม่ต้องการให้มันถูกเรียกจากนอกคอนแทรค สำหรับข้อมูลเพิ่มเติมเกี่ยวกับเรื่องนี้สามารถดูได้ [ที่นี่](https://docs.sui.io/devnet/build/programming-with-objects/ch4-object-wrapping). 