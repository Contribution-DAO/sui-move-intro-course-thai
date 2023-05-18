# การตั้งค่าเครื่องมือ

ยินดีต้อนรับสู่หลักสูตรเบื้องต้นสำหรับ Sui Move ในบทแรกนี้เราจะพาทุกคนไปดูวิธีการตั้งค่าเครื่องมือที่ใช้สำหรับพัฒนา Sui Move และสร้างโปรเจค Hello world แบบง่ายๆ เพื่อให้ทุกท่านค่อยๆ ได้ทำความรู้จักโลกของ Sui

## ตั้งค่าเครื่องมือที่ใช้สำหรับพัฒนา

[หน้าอ้างอิง](https://docs.sui.io/build/install#install-sui-binaries)

1. [ติดตั้งเครื่องมือที่จำเป็นกันก่อน](https://docs.sui.io/build/install#prerequisites) (ขึ้นอยู่กับระบบปฏิบัติการที่ใช้)

2. ติดตั้ง sui binaries ด้วยคำสั่ง

    `cargo install --locked --git https://github.com/MystenLabs/sui.git --branch devnet sui`

3. เช็คว่าติดตั้งสำเร็จหรือไม่:

    `sui --version`

    ถ้าการติดตั้งสำเร็จจะเห็นหมายเลขเวอร์ชั่นของ sui binaries ขึ้นมาบนเทอมินอล

## ใช้งานผ่าน Docker image ที่มี Sui Binaries ติดตั้งมาให้แล้ว

1. [วิธีติดตั้ง Docker](https://docs.docker.com/get-docker/)

2. ดาวน์โหลด Docker image ที่เตรียมไว้ให้สำหรับคอร์สนี้

    `docker pull hyd628/sui-move-intro-course:latest`

3. สั่งรัน docker ขึ้นมา และ shell เข้าไปในคอนเทนเนอร์

    `docker run --entrypoint /bin/sh -itd hyd628/sui-move-intro-course:latest`
    `docker exec -it <container ID> bash`

*💡หมายเหตุ: ถ้า docker image ด้านบน ใช้กับสถาปัตกรรมซีพียูเครื่องเราไม่ได้ คุณสามารถใช้ base [Rust](https://hub.docker.com/_/rust) Docker image *ที่เหมาะสมกับ CPU ของคุณแทน จากนั้นก็ทำการติดตั้ง Sui binaries และเครื่องมือต่างๆที่จำเป็นดังที่อธิบายไว้ด้านบน*

## ตั้งค่า VS Code และปลั๊กอิน Move Analyzer

1. ติดตั้งปลั๊กอิน [Move Analyzer plugin](https://marketplace.visualstudio.com/items?itemName=move.move-analyzer) จากมาร์เกตเพลส

2. เพิ่มคำสั่งให้สามารถอ่านหมายเลขกระเป๋าของ Sui ได้:

    `cargo install --git https://github.com/move-language/move move-analyzer --features "address20"`

## วิธีการใช้ Sui CLI แบบพื้นฐาน

[หน้าอ้างอิง](https://docs.sui.io/build/cli-client)

### การจัดการเครือข่าย

- สลับเครือข่าย: `sui client switch --env [network alias]`
- เครือข่ายมีชื่อเล่นเริ่มต้นดังนี้:
    - localnet: http://0.0.0.0:9000
    - devnet: https://fullnode.devnet.sui.io:443
- แสดงชื่อเล่นทั้งหมดของเครือข่าย: `sui client envs`
- เพิ่มชื่อเล่นให้เครือข่าย: `sui client new-env --alias <ALIAS> --rpc <RPC>`

### เช็คแอดเดรสที่ใช้งานอยู่ และแก๊ส

- เช็คแอดเดรสปัจจุบันในคีย์สโตร์: `sui client addresses`
- เช็คแอดเดรสที่กำลังใช้งาน: `sui client active-address`
- แสดงรายชื่อแก๊สทั้งหมด: `sui client gas`

### มิ้นท์ NFT สำหรับทดสอบ

- มิ้นท์ NFT สำหรับทดสอบที่เครือข่ายปัจจุบันด้วยคำสั่ง: `sui client create-example-nft` ผลลัพธ์ที่ได้ควรจะออกมาหน้าตาใกล้เคียงกับรูปข้างล่างนี้

![Demo NFT](../images/demo-nft.png)

## การขอรับเหรียญ Sui บน Devnet

1. [เข้าร่วมดิสคอร์ด](https://discord.gg/sui)
2. ยืนยันตัวตนให้เรียบร้อย
3. ไปที่ห้อง `#devnet-faucet`
4. พิมพ์คำสั่ง `!faucet <WALLET ADDRESS>`

## การขอรับเหรียญ Sui บน Testnet

1. [เข้าร่วมดิสคอร์ด](https://discord.gg/sui)
2. ยืนยันตัวตนให้เรียบร้อย
3. ไปที่ห้อง `#testnet-faucet`
4. พิมพ์คำสั่ง `!faucet <WALLET ADDRESS>`
