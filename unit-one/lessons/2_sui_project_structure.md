# โครงสร้างโปรเจคของ Sui

## โมดูล และแพ็คเกจ

- โมดูล คือชุดของฟังก์ชั่น และประเภทต่างๆ ที่ถูกรวบรวมไว้ด้วยกันสำหรับให้นักพัฒนาเผยแพร่ไปยังแอดเดรสที่ระบุ
- ไลบารี่มาตรฐานของ Sui จะอยู่ภายใต้แอดเดรส `0x2` ส่วนโมดูลที่นักพัฒนาทั่วไปสร้างมาใช้งานจะถูกสุ่มแอดเดรสให้โดยระบบปฏิบัติการเสมือนของ Sui Move (Sui Move VM)
- โมดูลจะขึ้นต้นด้วยคีย์เวิร์ดคำว่า `module` ตามด้วยชื่อโมดูล และเครื่องหมายปีกกาเปิด-ปิด ภายในปีกกาคือเนื้อหาของโมดูล

    ```rust
    module hello_world {
        // module contents
    }
    ```

- โมดูลที่ได้ทำการเผยแพร่ออกไปใช้งานแล้วถือเป็น immutable objects; immutable objects หมายถึงวัตถุที่ไม่สามารถ เปลี่ยนรูป, โอนย้าย หรือ ลบทิ้งได้ ความสามารถในการเปลี่ยนแปลงแก้ไขไม่ได้นี้ ทำให้มันไม่มีเจ้าของ และมันจะถูกใช้งานโดยใครก็ได้
- แพ็คเกจใน Move คือชุดของโมดูลที่มีไฟล์ manifest (ไฟล์ที่เอาไว้อธิบายข้อมูลต่างๆ ของแพ็คเกจนั้น) ชื่อ Move.toml


## สร้างแพ็คเกจ Sui Move

ใช้คำสั่งข้างล่างเพื่อขึ้นโครงสร้างแพ็คเกจ Sui:

`sui move new <PACKAGE NAME>`

สำหรับตัวอย่างของเราในบทนี้ เราจะเริ่มด้วยโปรเจค Hello World

`sui move new hello_world`

สิ่งที่ได้:

- โฟลเดอร์นอกสุดชื่อ `hello_world`
- ไฟล์ manifest `Move.toml`
- โฟลเดอร์ย่อย `sources` ที่มีไฟล์สมาร์ทคอนแทรคต่างๆ ของ Sui move อยู่ในนั้น


### โครงสร้างไฟล์ `Move.toml`

`Move.toml` เป็นไฟล์ที่เอาไว้อธิบายข้อมูลต่างๆ ในแพ็คเกจ มันจะถูกสร้างขึ้นมาให้อัตโนมัติ และเก็บไว้ที่โฟลเดอร์ชั้นนอกสุด

`Move.toml` ประกอบไปด้วยสามส่วน:

- `[package]` ใช้กำหนดชื่อ และเลขเวอร์ชั่นของแพ็คเกจ
- `[dependencies]` ใช้ลิสรายชื่อแพ็คเกจอื่นๆ ที่แพ็คเกจนี้มีการเรียกใช้งาน เช่นพวกไลบารี่มาตรฐานของ Sui เอง หรือไลบารี่ของผู้พัฒนาคนอื่นๆ ถ้ามีการเรียกใช้ก็นำมาใส่ไว้ตรงนี้เช่นกัน
- `[addresses]`ใช้กำหนดชื่อเล่นให้แต่ละแอดเดรส สำหรับเรียกใช้งานในซอร์สโค้ด

#### ตัวอย่างไฟล์ `Move.toml`

นี่คือไฟล์ `Move.toml` ที่ถูกสร้างขึ้นโดย Sui CLI ซึ่งมีชื่อแพ็คเกจคือ `hello_world`:


```rust
[package]
name = "hello_world"
version = "0.0.1"

[dependencies]
Sui = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "devnet" }

[addresses]
hello_world =  "0x0"
sui =  "0x2"
```

จะเห็นว่าในส่วนของ dependencies มีการเรียกใช้งานไลบารีมาตรฐานของทาง Sui โดยกำหนดด้วย Github repo ทั้งนี้ เรายังสามารถเรียกใช้งานไบนารีบนเครื่องได้ด้วยเช่นกัน โดยสามารถระบุเส้นทางแบบเต็มๆ หรือระบุบางส่วนโดยอ้างอิงจากตำแหน่งปัจจุบันก็ได้ ตัวอย่างเช่น

```rust
[dependencies]
Sui = { local = "../sui/crates/sui-framework/packages/sui-framework" } 
```

## การตั้งชื่อโมดูล และแพ็คเกจ

- หลักการตั้งชื่อโมดูล และแพ็คเจค ใน Sui Move นั้น จะใช้วิธีการที่เรียกว่า สเนคเคส ดังตัวอย่าง this_is_snake_casing
- ชื่อโมดูลใน Sui จะใช้เครื่องหมาย `::` ของภาษา Rust เพื่อใช้แบ่งชื่อโมดูล กับชื่อแพ็คเกจออกจากกัน ตัวอย่าง:
    1. `unit_one::hello_world` - โมดูลชื่อ `hello_world` ในแพ็คเกจชื่อ `unit_one`
    2. `capy::capy` - โมดูลชื่อ `capy`  ในแพ็คเกจชื่อ `capy`
- สำหรับข้อมูลเพิ่มเติมเกี่ยวกับหลักการตั้งชื่อบน Move ให้ดูได้ที่ [เว็บนี้](https://move-language.github.io/move/coding-conventions.html#naming)
