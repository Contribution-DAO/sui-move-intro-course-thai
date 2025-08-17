# ประเภทข้อมูลและความสามารถที่กำหนดเอง (Custom Types and Abilities)

ในตอนนี้ เราจะเริ่มสร้างสมาร์ตคอนแทรกต์ตัวอย่าง Hello World ทีละขั้นตอน พร้อมอธิบายแนวคิดพื้นฐานใน Sui Move เมื่อพบเจอ เช่น custom types และ abilities

## การเริ่มต้นสร้างแพ็กเกจ

(หากคุณข้าม section ก่อนหน้านี้ไป) คุณสามารถเริ่มต้นแพ็กเกจ Sui สำหรับ Hello World ได้ด้วยคำสั่งต่อไปนี้ใน command line หลังจากที่ [ติดตั้ง Sui binaries](./1_set_up_environment.md#install-sui-binaries-locally) แล้ว:

`sui move new hello_world`

## สร้าง Contract Source File

ใช้ editor ที่คุณถนัด เพื่อสร้างไฟล์ต้นฉบับของสมาร์ตคอนแทรกต์ Move ชื่อ `hello.move` ภายใต้โฟลเดอร์ย่อย `sources`

จากนั้นสร้าง module ว่าง ๆ ดังนี้:

```move
module hello_world::hello_world;
// เนื้อหาใน module
```

### คำสั่ง Import

คุณสามารถ import โมดูลใน Move โดยระบุที่อยู่ของโมดูลนั้นโดยตรง แต่เพื่อให้โค้ดอ่านง่ายขึ้น เราสามารถจัดการการ import ด้วยคำสั่ง `use`.

```move
use <Address/Alias>::<ModuleName>;
```

ในตัวอย่างของเรา เราต้อง import โมดูลต่างๆดังนี้:

```move
use std::string;
```

### Implicit Imports

บางโมดูลจะถูก import โดยอัตโนมัติ และสามารถใช้งานได้ภายใน module โดยไม่ต้องใช้ `use` สำหรับ Standard Library โมดูลและชนิดข้อมูลต่างๆเหล่านั้นได้แก่:

- `std::vector`
- `std::option`
- `std::option::Option`

เช่นเดียวกับ Standard Library โมดูลและชนิดข้อมูลบางอย่างใน Sui Framework ก็ถูก import โดยไม่ต้องใช้ `use` ด้วยเช่นกัน:

- `sui::object`
- `sui::object::ID`
- `sui::object::UID`
- `sui::tx_context`
- `sui::tx_context::TxContext`
- `sui::transfer`

### Method Call Syntax

Move รองรับ syntax การเรียกเมธอดโดยใช้เครื่องหมาย `.` เพื่อความสะดวกทางไวยากรณ์ ค่าในฝั่งซ้ายของเครื่องหมายจุด `.` จะถูกส่งเป็นอาร์กิวเมนต์ตัวแรกของฟังก์ชันโดยอัตโนมัติ การเรียกเมธอดทั้งหมดใน Move จะถูกกำหนดแบบคงที่ (statically) ตั้งแต่ขั้นตอนคอมไพล์

```move
// การเรียกฟังก์ชันแบบดั้งเดิม
let sender = tx_context::sender(ctx);
let text = string::utf8(b"Hello World!");

// การเรียกเมธอดด้วย dot syntax
let sender = ctx.sender();
let text = b"Hello World!".to_string();
```

syntax การเรียกเมธอดจะทำงานเมื่อพารามิเตอร์ตัวแรกของฟังก์ชันตรงกับชนิดข้อมูลก่อนเครื่องหมายจุด คอมไพเลอร์จะสร้าง alias ของเมธอดให้โดยอัตโนมัติในโมดูลที่กำหนด และจะทำการ borrow ค่า receiver โดยอัตโนมัติหากจำเป็น

## Custom Types

โครงสร้างข้อมูลใน Sui Move คือประเภทข้อมูลที่กำหนดเองซึ่งเก็บข้อมูลในรูปแบบ key-value โดยที่ key คือชื่อของ property และ value คือค่าที่เก็บ ใช้คีย์เวิร์ด `struct` เพื่อกำหนด และสามารถระบุความสามารถ (abilities) ได้สูงสุด 4 อย่าง

### ความสามารถ (Abilities)

Abilities คือคีย์เวิร์ดใน Sui Move ที่กำหนดพฤติกรรมของชนิดข้อมูลในระดับของคอมไพเลอร์

Abilities มีความสำคัญมากในการกำหนดพฤติกรรมของอ็อบเจกต์ใน Sui Move ในระดับภาษา การรวมกันของ abilities แต่ละแบบจะมี design pattern ที่แตกต่างกัน เราจะศึกษา abilities และการใช้งานใน Sui Move ตลอดทั้งคอร์สนี้

สำหรับตอนนี้ ให้รู้ไว้ว่ามี abilities 4 อย่างใน Sui Move:

- **copy**: ค่านี้สามารถถูก copy (หรือ clone แบบ by value) ได้
- **drop**: ค่านี้สามารถถูกทำลายทิ้งได้เมื่อจบ scope
- **key**: ค่านี้สามารถใช้เป็น key สำหรับการจัดเก็บข้อมูลแบบ global ได้
- **store**: ค่านี้สามารถเก็บอยู่ภายใน struct ที่ถูกจัดเก็บแบบ global ได้

#### สินทรัพย์ (Assets)

ประเภทข้อมูลที่กำหนดเองซึ่งมี abilities `key` และ `store` จะถือว่าเป็น **สินทรัพย์** ใน Sui Move โดยสินทรัพย์จะถูกเก็บไว้ใน global storage และสามารถโอนระหว่างบัญชีได้

### ประเภทข้อมูล Hello World

เราจะกำหนดอ็อบเจกต์ในตัวอย่าง Hello World ดังนี้:

```move
/// อ็อบเจกต์ที่มีข้อความ string ใด ๆ
public struct HelloWorldObject has key, store {
  	id: UID,
  	/// ข้อความ string ที่เก็บอยู่ในอ็อบเจกต์
  	text: string::String
}
```

UID คือชนิดข้อมูลจาก Sui Framework (sui::object::UID) ซึ่งใช้กำหนดหมายเลขประจำตัวที่ไม่ซ้ำกันในระดับ global ของอ็อบเจกต์ สำหรับประเภทข้อมูลที่มี ability `key` จำเป็นต้องมีฟิลด์ ID เสมอ
