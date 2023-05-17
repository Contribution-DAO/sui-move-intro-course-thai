# ประเภทความเป็นเจ้าของ ของ Sui Objects

แต่ละ Object ใน Sui จะมีฟิลด์ owner เพื่อใช้ระบุว่า object นี้ถูกเป็นเจ้าของได้อย่างไร ใน Sui Move จะมีรูปแบบความเป็นเจ้าของทั้งหมด 4 ประเภท

- Owned
    - Owned by an address
    - Owned by another object 
- Shared
    - Shared immutable
    - Shared mutable

## Owned Objects

ความเป็นเจ้าของสองแบบแรก อยู่ภายใต้หมวดหมู่ `Owned Objects` Owned Objects ใน Sui จะมีการทำงานที่แตกต่างจาก shared objects และไม่จำเป็นต้องมีการเรียกใช้งานคำสั่งจากด้านนอก (global ordering)

### Owned by an Address

มาต่อที่ตัวอย่าง `transacript` ของเรากัน ความเป็นเจ้าของประเภทนี้ค่อนข้างที่จะตรงไปตรงมาโดยที่วัตถุจะถูกครอบครองโดยแอดเดรสที่มันถูกโอนไปให้ตอนสร้าง ดังตัวอย่างข้างต้น ตรงบรรทัดนี้:


```rust
    transfer::transfer(transcriptObject, tx_context::sender(ctx)) // where tx_context::sender(ctx) is the recipient
```

จะเห็นว่า `transcriptObject` ถูกถ่ายโอนไปยังแอดเดรสที่ทำธุรกรรมนี้ ในขั้นตอนการสร้าง

### Owned by An Object

เพื่อให้วัตถุใดๆถูกครอบครองโดยวัตถุอื่น จะทำได้โดยการใช้ `dynamic_object_field` ซึ่งเดี๋ยวเราจะทำความรู้จักกับมันในตอนถัดๆ ไป โดยพื้นฐานแล้ว เมื่อวัตถุถูกครอบครองโดยวัตถุอื่น เราจะเรียกมันว่าวัตถุย่อย ตัววัตถุนี้สามารถถูกมองเห็นได้จากที่จัดเก็บข้อมูลภายนอก (global storage) โดยใช้ ID ของมัน

## Shared Objects

## Shared Immutable Objects

วัตถุบางอย่างใน Sui ไม่สามารถถูกเปลี่ยนแปลงได้ไม่ว่าโดยใครก็ตาม ด้วยเหตุนี้  วัตถุเหล่านั้นจึงไม่ได้มีใครเป็นเจ้าของมันโดยตรง โดยทุกๆแพ็คเกจ และโมดูล ที่ถูกเผยแพร่ให้ใช้งานใน Sui ถือเป็นวัตถุที่เปลี่ยนแปลงแก้ไขไม่ได้ (immutable objects)

เราสามารถทำให้วัตถุเป็น immutable ได้โดยการเรียกใช้ฟังก์ชั่นพิเศษนี้:

```rust
    transfer::freeze_object(obj);
```

## Shared Mutable Objects

Shared objects ใน Sui สามารถถูกอ่าน และแก้ไขโดยใครก็ได้ ธุรกรรมของ Shared objects จะต้องมีการเรีกใช้งานคำสั่งจากภายนอก (global ordering) ผ่านโปรโตคอลระดับฉันทามติ (consensus layer protocol) ซึ่งไม่เหมือนกับ owned objects.

ในการสร้าง shared object ทำได้โดยการเรียกฟังก์ชั่นนี้:

```rust
    transfer::share_object(obj);
```

เมื่อวัตถุนั้นเป็น shared object แล้ว มันจะยังคงความสามารถในการแก้ไขเปลี่ยนแปลงได้เอาไว้ และสามารถเข้าถึงโดยใครก็ได้เพื่อใช้ทำธุรกรรมไปยัง mutate object

