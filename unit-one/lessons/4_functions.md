# ฟังก์ชัน

ในตอนนี้ เราจะทำความรู้จักกับฟังก์ชันใน Sui Move และเขียนฟังก์ชัน Sui Move แรกของเราซึ่งเป็นส่วนหนึ่งของตัวอย่าง Hello World

## การกำหนดขอบเขตการเข้าถึงของฟังก์ชัน (Function Visibility)

ฟังก์ชันใน Sui Move มีขอบเขตการเข้าถึง 3 ประเภท:

- **private**: เป็นค่าเริ่มต้น ฟังก์ชันสามารถถูกเรียกใช้ได้เฉพาะภายในโมดูลเดียวกันเท่านั้น
- **public**: ฟังก์ชันสามารถถูกเรียกใช้ได้จากทั้งภายในโมดูลเดียวกันและจากฟังก์ชันในโมดูลอื่น
- **public(package)**: ฟังก์ชันสามารถถูกเรียกใช้ได้จากฟังก์ชันในโมดูลที่อยู่ในแพ็กเกจเดียวกัน

## ค่าที่ฟังก์ชันส่งกลับ (Return Value)

ชนิดข้อมูลที่ฟังก์ชันจะส่งกลับจะถูกระบุไว้ในส่วนของ signature หลังจากพารามิเตอร์ โดยคั่นด้วยเครื่องหมาย colon

บรรทัดสุดท้ายของฟังก์ชันที่ไม่มีเครื่องหมายเซมิโคลอน คือค่าที่ฟังก์ชันส่งกลับ

ตัวอย่าง:

```move
public fun addition (a: u8, b: u8): u8 {
    a + b
}
```

<!--
## Entry Functions

In Sui Move, entry functions are simply functions that can be called by transactions. They must satisfy the following three requirements:

- Denoted by the keyword `entry`
- have no return value
- (optional) have a mutable reference to an instance of the `TxContext` type in the last parameter

-->

## Transaction Context

ฟังก์ชันที่ถูกเรียกโดยตรงผ่านธุรกรรมโดยปกติจะมีอินสแตนซ์ของ `TxContext` เป็นพารามิเตอร์ตัวสุดท้าย ซึ่งเป็นพารามิเตอร์พิเศษที่ VM ของ Sui Move สร้างให้อัตโนมัติ โดยผู้เรียกใช้ฟังก์ชันไม่จำเป็นต้องกำหนดเอง

ออปเจกต์ `TxContext` จะบรรจุ  [ข้อมูลสำคัญ](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/packages/sui-framework/sources/tx_context.move) เกี่ยวกับธุรกรรมที่เรียกฟังก์ชัน entry เช่น ที่อยู่ของผู้ส่ง (sender’s address), รหัสย่อยของธุรกรรม (digest ID), epoch ของธุรกรรม ฯลฯ

## การสร้างฟังก์ชัน `mint`

เราสามารถนิยามฟังก์ชันสำหรับ mint (สร้าง) ในตัวอย่าง Hello World ได้ดังนี้:

```move
public fun mint(ctx: &mut TxContext) {
    let object = HelloWorldObject {
        id: object::new(ctx),
        text: b"Hello World!".to_string()
    };
    transfer::public_transfer(object, ctx.sender());
}
```

ฟังก์ชันนี้จะสร้างอินสแตนซ์ใหม่ของ `HelloWorldObject` ซึ่งเป็นชนิดข้อมูลแบบกำหนดเอง จากนั้นใช้ฟังก์ชัน [`public_transfer`](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/transfer.md#function-public_transfer) ที่อยู่ในระบบของ Sui เพื่อส่งอ็อบเจกต์นี้ไปยังผู้ที่เรียกธุรกรรมนี้
