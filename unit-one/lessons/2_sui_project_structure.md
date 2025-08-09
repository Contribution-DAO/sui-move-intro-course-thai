# โครงสร้างโปรเจคของ Sui

## โมดูลและแพ็กเกจของ Sui

- Sui module คือชุดของฟังก์ชันและชนิดข้อมูล (types) ที่ถูกรวมเข้าด้วยกัน และเผยแพร่ (publish) โดยนักพัฒนาภายใต้ address ที่ระบุไว้

- Sui standard library ถูกเผยแพร่ภายใต้ address `0x2` ในขณะที่โมดูลที่ผู้ใช้สร้างขึ้นจะถูกเผยแพร่ภายใต้ address แบบสุ่มที่ถูกกำหนดโดย Sui Move VM

- โมดูลจะเริ่มต้นด้วยคีย์เวิร์ด `module` ตามด้วยชื่อโมดูล และวงเล็บปีกกา ซึ่งใช้ใส่เนื้อหาของโมดูล:

  ```move
  module hello_world::hello_world;
  // เนื้อหาภายในโมดูล
  ```

- โมดูลที่เผยแพร่แล้วใน Sui จะเป็น object ที่เปลี่ยนแปลงไม่ได้ (Immutable object) ซึ่งหมายถึงไม่สามารถแก้ไข, โอนย้าย หรือถูกลบได้ และเนื่องจากไม่มีใครเป็นเจ้าของ มันจึงสามารถถูกใช้งานได้โดยใครก็ได้

- Move package คือกลุ่มของโมดูลที่ถูกรวมไว้ในโฟลเดอร์เดียวกัน พร้อมไฟล์ manifest ชื่อว่า Move.toml

## การเริ่มต้น Sui Move Package

ใช้คำสั่งของ Sui CLI ต่อไปนี้เพื่อสร้างโครงสร้างพื้นฐานของแพ็กเกจ Sui:

`sui move new <PACKAGE NAME>`

ตัวอย่างในบทนี้ เราจะเริ่มต้นด้วยโปรเจกต์ Hello World:

`sui move new hello_world`

คำสั่งนี้จะสร้าง:

- root โฟลเดอร์ของโปรเจกต์ชื่อ `hello_world`
- ไฟล์ manifest `Move.toml` ที่เก็บ metadata ของแพ็กเกจ
- โฟลเดอร์ย่อย `sources/` สำหรับเก็บซอร์สโค้ดของ smart contract ที่เขียนด้วย Sui Move
- โฟลเดอร์ย่อย `tests/` สำหรับเก็บโค้ดทดสอบของแพ็กเกจ โค้ดในโฟลเดอร์นี้จะไม่ถูกเผยแพร่ขึ้น chain ใช้สำหรับทดสอบเท่านั้น

### โครงสร้างไฟล์ `Move.toml`

`Move.toml` เป็นไฟล์ manifest ของแพ็กเกจ และถูกสร้างขึ้นอัตโนมัติใน root โฟลเดอร์ของโปรเจกต์

`Move.toml` ประกอบด้วยหลาย section:

- `[package]` อธิบายข้อมูลของแพ็กเกจ เช่น name (ใช้เมื่อ import), version (สำหรับจัดการ release), edition (เวอร์ชันภาษาของ Move ซึ่งปัจจุบันคือ 2024)
- `[dependencies]` ระบุ dependencies ของโปรเจกต์ โดยแต่ละ dependency อาจเป็น Git repository หรือ path ของไดเรกทอรีภายในเครื่องก็ได้ แพ็กเกจยังสามารถนำเข้า address จาก dependencies ได้ด้วยเช่นกัน
- `[dev-dependencies]` ใช้สำหรับ override dependencies ในโหมดพัฒนา (dev) และโหมดทดสอบ (test) ตัวอย่างเช่น หากคุณต้องการใช้เวอร์ชันที่แตกต่างของแพ็กเกจ Sui ในโหมด dev คุณสามารถเพิ่มการกำหนด dependency แบบกำหนดเองลงในส่วน [dev-dependencies] ได้
- `[addresses]` ตั้งชื่อเล่น (alias) ให้กับ address เพื่อความสะดวกในโค้ด
- `[dev-addresses]` ใช้ override ชื่อ alias ของ address เฉพาะในโหมด dev กับ test เท่านั้น ไม่สามารถเพิ่ม alias ใหม่ได้

#### ตัวอย่างไฟล์ `Move.toml`

ด้านล่างคือตัวอย่างไฟล์ `Move.toml` ที่ถูกสร้างขึ้นโดย Sui CLI เมื่อใช้ชื่อแพ็กเกจว่า `hello_world`:

```toml
[package]
name = "hello_world"
edition = "2024.beta" # edition = "legacy" เพื่อใช้ Move เวอร์ชันเก่า (ก่อนปี 2024)
# license = ""           # เช่น "MIT", "GPL", "Apache 2.0"
# authors = ["..."]      # เช่น ["Joe Smith (joesmith@noemail.com)", "John Snow (johnsnow@noemail.com)"]

[dependencies]
# สำหรับ import จาก Git ใช้รูปแบบ { git = "...", subdir = "...", rev = "..." }
# rev สามารถเป็น branch, tag หรือ commit hash
# MyRemotePackage = { git = "https://some.remote/host.git", subdir = "remote/path", rev = "main" }

# สำหรับ dependency ในเครื่องใช้ local path
# Local = { local = "../path/to" }

# ถ้ามี version conflict สามารถบังคับเวอร์ชันด้วย override
# Override = { local = "../conflicting/version", override = true }

[addresses]
hello_world = "0x0"

# Alias ของ address จะสามารถใช้งานใน Move ได้ในรูปแบบ `@name` และสามารถ export ออกไปได้ด้วย
# ตัวอย่าง: std = "0x1" มาจาก Standard Library
# alice = "0xA11CE"

[dev-dependencies]
# สำหรับระบุ dependency ที่ใช้เฉพาะในโหมด `--test` และ `--dev`
# Local = { local = "../path/to/dev-build" }

[dev-addresses]
# ใช้เพื่อ override address alias ในโหมด `--test` และ `--dev` เท่านั้น
# alice = "0xB0B"

```

## Dependencies

ส่วนของ `[dependencies]` จะใช้สำหรับระบุ dependencies ของโปรเจกต์ โดยแต่ละ dependency จะถูกกำหนดในรูปแบบ key-value ซึ่ง key คือชื่อของ dependency และ value คือข้อมูลจำเพาะของ dependency นั้น โดยข้อมูลจำเพาะนี้อาจเป็น URL ของ Git repository หรือ path ไปยังไดเรกทอรีภายในเครื่องก็ได้

```toml
# git repository
Example = { git = "https://github.com/example/example.git", subdir = "path/to/package", rev = "framework/testnet" }

# local directory
MyPackage = { local = "../my-package" }
```

แพ็กเกจยังสามารถ import address จากแพ็กเกจอื่นได้ เช่น dependency ของ Sui จะเพิ่ม address `std` และ `sui` ให้ใช้งานได้ในโปรเจกต์ ซึ่งสามารถใช้เป็น alias ในโค้ดได้

ตั้งแต่ Sui CLI เวอร์ชัน 1.45 ขึ้นไป แพ็กเกจระบบของ Sui (`std`, `sui`, `system`, `bridge`, and `deepbook`) จะถูกเพิ่มเข้ามาให้อัตโนมัติถึงแม้จะไม่ได้ระบุไว้

### การจัดการปัญหาเวอร์ชัน Conflict ด้วย Override

บางครั้ง dependencies อาจมีเวอร์ชันที่ขัดแย้งกันของแพ็กเกจเดียวกัน ตัวอย่างเช่น หากคุณมี dependencies สองตัวที่ใช้แพ็กเกจ Example คนละเวอร์ชัน คุณสามารถ override dependency นั้นได้ในส่วน `[dependencies]` โดยให้เพิ่มฟิลด์ `override` เข้าไปใน dependency ที่ต้องการ จากนั้นระบบจะใช้เวอร์ชันที่ระบุไว้ใน `[dependencies]` แทนเวอร์ชันที่ถูกระบุไว้ภายในตัว dependency เอง

```toml
[dependencies]
Example = { override = true, git = "https://github.com/example/example.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }
```

## การตั้งชื่อ Sui Module และ Package

- การตั้งชื่อโมดูลและแพ็กเกจใน Sui Move จะใช้ snake case (ตัวพิมพ์เล็กและขีดล่าง) เช่น this_is_snake_casing

- ชื่อของ Sui module ใช้ตัวคั่น `::` เหมือนภาษา Rust เพื่อแยกชื่อแพ็กเกจกับชื่อโมดูล เช่น:

  1. `unit_one::hello_world` - หมายถึงโมดูล `hello_world` ในแพ็กเกจ `unit_one`
  2. `capy::capy` - โมดูล `capy` ในแพ็กเกจ `capy`

- สามารถดูแนวทางการตั้งชื่อเพิ่มเติมได้จาก [หัวข้อ Style](https://move-language.github.io/move/coding-conventions.html#naming) ในหนังสือ Move
