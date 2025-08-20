# การทำงานกับ Sui Objects

## เกริ่นนำ

Sui Move เป็นภาษาที่เน้น object เป็นศูนย์กลาง ทุกๆธุรกรรมบน Sui จะถือเป็นการดำเนินการที่ข้อมูลที่ป้อนเข้าไป และผลลัพธ์ที่ได้ออกมา เป็น objects ทั้งคู่ ดังที่เราได้กล่าวไปแล้วสั้นๆ เกี่ยวกับแนวคิดนี้ใน [บทที่ 1 ตอนที่ 4](../../unit-one/lessons/4_custom_types_and_abilities.md#custome-types-and-abilities) Sui objects เป็นหน่วยพื้นฐานของการจัดเก็บข้อมูลใน Sui ซึ่งทุกอันจะขึ้นต้นด้วยคีย์เวิร์ดคำว่า `struct`

ก่อนอื่น เรามาเริ่มด้วยตัวอย่างที่แสดงถึงใบรับรองผลการเรียนที่บันทึกผลการเรียนของนักเรียน:

```move
public struct Transcript {
    history: u8,
    math: u8,
    literature: u8,
}
```

โค้ดข้างบนเป็นการประกาศ struct ใน Move แบบปกติ แต่ยังไม่ใช่ Sui object เพื่อที่จะทำให้ custom type นี้สร้างอินสแตนซ์ของ Sui object ในที่จัดเก็บข้อมูลภายนอก (global storage) ได้ เราต้องเพิ่ม ability `key` และฟิลด์ `id: UID` ที่ไม่ซ้ำใครในการประกาศ struct

```move
public struct TranscriptObject has key {
    id: UID,
    history: u8,
    math: u8,
    literature: u8,
}
```

## สร้าง Sui Object

การสร้าง Sui object จำเป็นที่จำต้องมี ID ที่ไม่ซ้ำใคร เราจะใช้ฟังก์ชั่น `sui::object::new` ในการสร้าง ID และส่งให้ `TxContext`

ใน Sui ทุก object ต้องมีเจ้าของ ซึ่งสามารถเป็นได้ทั้ง แอดเดรส, object ตัวอื่น หรือ “shared” ในตัวอย่างของเรา เราทำการสร้าง `transcriptObject` ใหม่ โดยมีเจ้าของคือคนทำธุรกรรม ซึ่งสามารถทำได้โดยการใช้ฟังก์ชั่น `transfer` ของ Sui framework และใช้ฟังก์ชั่น `tx_context::sender` เพื่อรับเอาค่าแอดเดรสของคนทำธุรกรรมนั้นมา

เราจะเจาะลึกเรื่องเกี่ยวกับความเป็นเจ้าของในตอนถัดไป

```move
public fun create_transcript_object(history: u8, math: u8, literature: u8, ctx: &mut TxContext) {
  let transcript_object = TranscriptObject {
    id: object::new(ctx),
    history,
    math,
    literature,
  };
  transfer::transfer(transcript_object, ctx.sender())
}
```
*💡หมายเหตุ: โค้ดตัวอย่างที่ให้มาจะสร้างข้อความแจ้งเตือนว่า warning[Lint W01001]: non-composable transfer to sender สำหรับรายละเอียดเพิ่มเติม โปรดดูบทความ "[Sui Linters and Warnings Update Increases Coder Velocity](https://blog.sui.io/linter-compile-warnings-update/)"* 
*หมายเหตุ: Move รองรับการทำ field punning ซึ่งทำให้เราไม่ต้องใส่ค่าของฟิลด์ได้ ถ้าทั้งชื่อฟิลด์ และค่านั้น มีค่าเดียวกัน*
