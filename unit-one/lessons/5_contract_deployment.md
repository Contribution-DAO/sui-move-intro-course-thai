# วิธีการ Deploy Contract และสาธิตโปรเจค Hello World

## ตัวอย่างโปรเจค Hello World ฉบับสมบูรณ์

ทุกคนสามารถดูตัวอย่างโปรเจค Hello World ฉบับสมบูรณ์ได้ [ที่นี่](../example_projects/hello_world). 

## การ Deploy Contract

เราจะใช้ Sui CLI ในการ deploy แพ็กเกจไปยัง Sui network เราสามารถ deploy ไปที่ devnet, testnet หรือ local node ก็ได้ เพียงแค่เซต Sui CLI ไปยัง network ที่ต้องการ และมีเหรียญเพียงพอสำหรับจ่ายค่าแก๊ส

คำสั่ง Sui CLI สำหรับ deploy แพ็คเกจมีดังนี้:

```bash
sui client publish [absolute file path to the package that needs to be published]
```

ถ้าไม่ระบุ path ระบบจะ deploy ไปที่ไดเรกทอรี่ปัจจุบัน `.` โดยอัตโนมัติ

ผลลัพธ์ที่ได้จะหน้าตาประมาณนี้ ถ้าการ deploy contract เสร็จสมบูรณ์

![Publish Output](../images/publish.png)

ตัวออปเจกต์ ID ที่อยู่ใน `Published Objects` คือ ID ของแพ็คเกจ Hello World ที่เราเพิ่งเอาขึ้นไป

ลอง export ออกมาเป็นตัวแปรดู

```bash
export PACKAGE_ID=<package object ID from previous output>
```

## การเรียกใช้งานเมธอดผ่านธุรกรรม

ต่อไป เราจะทำการ mint Hello World object โดยการเรียกฟังก์ชั่น `mint` ในสมาร์ทคอนแทรคที่เราเพิ่ง deploy ขึ้นไป

สังเกตว่าที่เราทำแบบนี้ได้เพราะฟังก์ชั่น `mint` เป็น entry function

คำสั่งสำหรับ Sui CLI คือ:

```bash
sui client call --function mint --module hello_world --package $PACKAGE_ID 
```

ถ้าเรียกฟังก์ชั่น `mint` สำเร็จ ควรจะได้ผลลัพธ์หน้าตาประมาณนี้บน console และ Hello World object จะถูกสร้างและโอนมาให้เรา

![Mint Output](../images/mint.png)

ตัว ID ที่อยู่ใน `Created Objects` คือ ID ของ Hello World object

## การดู Object ด้วย Sui Explorer

ลองใช้ [Sui Explorer](https://suiexplorer.com/) เพื่อดู Hello World object ที่เราเพิ่งสร้างและโอน

ทำการเลือก network ที่เราใช้งานผ่านเมนูที่มุมบนขวา

หากใช้ local dev node ให้เลือก `Custom RPC URL` และใส่ค่า:

```bash
http://127.0.0.1:9000
```

เอา Object ID จากผลลัพธ์ของธุรกรรมเมื่อสักครู่มาค้นหา แล้วคุณจะสามารถดูออปเจกต์ใน explorer ได้ดังนี้:

![Explorer Output](../images/explorer.png)

เราควรจะเห็นข้อความ "Hello World" อยู่ใต้หัวข้อ Properties

เยี่ยมมาก นี่คือข้อสรุปบทแรกของหลักสูตร
