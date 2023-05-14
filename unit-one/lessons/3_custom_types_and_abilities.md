# Types และ Abilities แบบปรับแต่งเอง (Custom Types and Abilities)

ในตอนนี้ เราจะเริ่มทดลองสร้างคอนแทรค Hello World แบบไปทีละขั้นตอน และอธิบายแนวคิดพื้นฐานต่างๆใน Sui Move เช่น types และ abilities แบบปรับแต่งเอง

## สร้างแพ็คเกจ

(ถ้าคุณข้ามตอนก่อนหน้านี้มา) คุณสามารถสร้างแพ็คเกจ Hello World ด้วยคำสั่งด้านล่างนี้ หลังจาก [ติดตั้ง Sui binaries](./1_set_up_environment.md#install-sui-binaries-locally):

`sui move new hello_world`

## สร้างไฟล์ซอร์สโค้ดของคอนแทรค

เปิดโปรเจคบน editor ที่ถนัด จากนั้นสร้างไฟล์ชื่อ `hello.move` ในโฟลเดอร์ `sources`

จากนั้นสร้างโมดูลเปล่าตามนี้

```rust
module hello_world::hello {
    // module contents
}
```

### คำสั่ง Import

ใน Move เราสามารถ import โมดูลต่างๆโดยตรงได้ด้วยแอดเดรสของมัน แต่เพื่อทำให้โค้ดอ่านง่ายขึ้น เราสามารถใช้คีย์เวิร์ด `use` เพื่อจัดระเบียบการ import ได้

```rust
use <Address/Alias>::<ModuleName>;
```

ในตัวอย่างของเรา เราจะอิมพอร์ทโมดูลต่างๆดังนี้

```rust
use std::string;
use sui::object::{Self, UID};
use sui::transfer;
use sui::tx_context::{Self, TxContext};
```

## Types แบบปรับแต่งเอง (Custom Types)

structure ใน Sui Move คือ type แบบปรับแต่ง ซึ่งประกอบไปด้วยคู่ key-value โดยที่ key คือชื่อ และ value คือค่าที่เก็บไว้ สามารถสร้างด้วยการใช้คีย์เวิร์ด `struct` โดยที่โครงสร้างของมันสามารถมีได้สูงสุด 4 abilities

### ความสามารถ (Abilities)

Abilities คือคีย์เวิร์ดใน Sui Move ที่ใช้กำหนดว่าแต่ละ types จะทำงานยังไงในคอมไพเลอร์

Abilities มีความสำคัญอย่างมากในการกำหนดพฤติกรรมของวัตถุใน Sui Move การผสมผสานความสามารถต่างๆ มีรูปแบบการออกแบบที่เป็นเอกลักษณ์เฉพาะของตัวมันเอง เราจะได้ศึกษาแต่ละความสามารถและวิธีการใช้งานมันตลอดทั้งหลักสูตรนี้

ณ ตอนนี้ ใน Sui Move มีเพียงแค่ 4 ความสามารถ นี้เท่านั้น

- **Copy:** สามารถถูกคัดลอก (หรือโคลน) ได้
- **Drop:** สามารถถูกลบออกเมื่อสิ้นสุดการทำงานได้
- **Key:** สามารถใช้เป็นคีย์ สำหรับการดำเนินการใดๆกับที่จัดเก็บข้อมูลข้างนอก (golbal storage) ได้
- **Store:** สามารถถูกจัดเก็บ ไว้ในที่จัดเก็บข้อมูลข้างนอก (global storage) ได้

types ที่ปรับแต่งเองใดๆ ที่มี abilities ทั้ง `key` และ `store` อยู่ด้วยกัน จะถือว่าเป็น **สินทรัพย์ (assets)** สินทรัพย์จะถูกจัดเก็บไว้ที่พื้นที่เก็บข้อมูลด้านนอก (global storage) และสามารถถูกโอนไปมาระหว่างหลายๆบัญชีได้

### Type Hello World แบบปรับแต่งเอง

เรานิยามวัตถุในตัวอย่าง Hello World ของเราดังนี้:

```rust
/// An object that contains an arbitrary string
struct HelloWorldObject has key, store {
  	id: UID,
  	/// A string contained in the object
  	text: string::String
}
```

UID ในที่นี้เป็น type ของ Sui Framework (sui::object::UID) ที่กำหนดไอดีเฉพาะให้กับวัตถุ และ Type ที่ปรับแต่งเองที่มี ability `key` จำเป็นที่จะต้องมีฟิลด์ ID
