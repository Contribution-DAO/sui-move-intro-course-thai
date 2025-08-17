# Setup Development Environment

ยินดีต้อนรับสู่หลักสูตรเบื้องต้นสำหรับ Sui Move ในบทแรกนี้เราจะพาทุกคนไปดูวิธีการตั้งค่าเครื่องมือที่ใช้สำหรับพัฒนา Sui Move และสร้างโปรเจค Hello world แบบง่ายๆ เพื่อให้ทุกท่านค่อยๆ ได้ทำความรู้จักโลกของ Sui

## ติดตั้ง Sui

Move เป็นภาษาแบบต้อง compiled ดังนั้นคุณต้องติดตั้ง compiler เพื่อให้สามารถเขียนและรันโปรแกรม Move ได้ ซึ่ง compiler นี้รวมอยู่ใน Sui binary แล้ว และสามารถติดตั้งได้หลายวิธีดังด้านล่าง

### ติดตั้งผ่าน suiup (แนะนำ)

วิธีที่ดีที่สุดในการติดตั้ง Sui คือใช้ `suiup` ซึ่งเป็นเครื่องมือสำหรับติดตั้ง binary และจัดการเวอร์ชันต่างๆ (เช่น สำหรับ testnet หรือ mainnet)

ดูวิธีติดตั้ง `suiup` ได้ใน [README ของ repository](https://github.com/MystenLabs/suiup).


รันคำสั่งนี้เพื่อติดตั้ง Sui:

```bash
suiup install sui
```

### ดาวน์โหลด Binary

คุณสามารถดาวน์โหลด Sui binary เวอร์ชันล่าสุดได้จาก [หน้า Releases](https://github.com/MystenLabs/sui/releases) ซึ่งรองรับ macOS, Linux และ Windows หากใช้เพื่อการเรียนรู้หรือพัฒนาแนะนำให้ใช้เวอร์ชัน mainnet

### ติดตั้งด้วย Homebrew (macOS)

คุณสามารถติดตั้ง Sui ด้วย Homebrew ได้

```bash
brew install sui
```

### ติดตั้งด้วย Chocolatey (Windows)

สำหรับ Windows คุณสามารถติดตั้งผ่าน Chocolatey ได้

```bash
choco install sui
```

### Build ด้วย Cargo (macOS, Linux)

คุณสามารถติดตั้งและ build Sui บนเครื่องของคุณเองได้โดยใช้ตัวจัดการแพ็กเกจ Cargo (ต้องติดตั้ง Rust ก่อน)

```bash
cargo install --git https://github.com/MystenLabs/sui.git sui --branch mainnet
```

หากต้องการใช้เครือข่ายอื่น เช่น `testnet` หรือ `devnet` สามารถเปลี่ยนชื่อ branch ได้ตามต้องการ


ตรวจสอบให้แน่ใจว่าคุณใช้ Ruse เวอร์ชั่นล่าสุดด้วยคำสั่งด้านล่างนี้

```bash
rustup update stable
```

### ตรวจสอบการติดตั้ง

ตรวจสอบว่า binary ถูกติดตั้งเรียบร้อยแล้ว:

```bash
sui --version
```

หากติดตั้งสำเร็จ จะเห็นหมายเลขเวอร์ชันแสดงขึ้นใน terminal


### การแก้ปัญหา (Troubleshooting)

หากพบปัญหาในการติดตั้ง โปรดดูที่ [คำแนะนำการติดตั้ง Sui](https://docs.sui.io/build/install).

## ใช้ Docker Image ที่ติดตั้ง Sui Binaries มาให้แล้ว

1. [ติดตั้ง Docker](https://docs.docker.com/get-docker/)

2. Pull Sui docker image เวอร์ชันทางการ

   `docker pull mysten/sui-tools:devnet`

3. Start และ shell เข้าไปใน containerซ

   `docker run --name suidevcontainer -itd mysten/sui-tools:devnet`

   `docker exec -it suidevcontainer bash`

_💡Note: ถ้า Docker image นี้ไม่รองรับสถาปัตยกรรม CPU ของคุณ คุณสามารถเริ่มจาก base image ของ [Rust](https://hub.docker.com/_/rust) ที่เข้ากันได้กับเครื่อง แล้วติดตั้ง Sui และ dependencies ตามวิธีด้านบน_

## (ไม่จำเป็น) ติดตั้ง Move Analyzer บน VS Code

1. ติดตั้งปลั๊กอิน [Move Analyzer](https://marketplace.visualstudio.com/items?itemName=move.move-analyzer) จาก VS Marketplace

2. เพิ่มความเข้ากันได้กับ address สไตล์ Sui:

   `cargo install --git https://github.com/move-language/move move-analyzer --features "address20"`

## การใช้งาน Sui CLI ขั้นพื้นฐาน

[Reference Page](https://docs.sui.io/build/cli-client)

### เริ่มต้นใช้งาน (Initialization)

- พิมพ์ `Y` สำหรับคำถาม `do you want to connect to a Sui Full node server?` และกด `Enter` - เพื่อใช้ Sui Devnet full node
- พิมพ์ `0` สำหรับ key scheme เพื่อใช้ [`ed25519`](https://ed25519.cr.yp.to/)

### การจัดการ Network ต่างๆ

- การสลับ Network: `sui client switch --env [network alias]`
- ชื่อเล่นเครือข่ายเริ่มต้น (network aliases):
  - localnet: http://0.0.0.0:9000
  - devnet: https://fullnode.devnet.sui.io:443
- ดู network aliases ทั้งหมด: `sui client envs`
- เพิ่ม alias ใหม่: `sui client new-env --alias <ALIAS> --rpc <RPC>`
  - ทดลองเพิ่ม testnet alias ด้วยคำสั่ง: `sui client new-env --alias testnet --rpc https://fullnode.testnet.sui.io:443`

### ตรวจสอบ Address และ Gas Object

- ดู address ที่มีอยู่ใน keystore: `sui client addresses`
- ดู address ที่ใช้งานปัจจุบัน: `sui client active-address`
- ดู gas objects ทั้งหมดที่คุณควบคุม: `sui client gas`

## รับ SUI Token สำหรับ Devnet หรือ Testnet

1. [เข้าร่วม Sui Discord](https://discord.gg/sui)
2. ยืนยันตัวตนให้เรียบร้อย
3. ไปที่ห้อง [`#devnet-faucet`](https://discord.com/channels/916379725201563759/971488439931392130) สำหรับเหรียญ devnet หรือ [`#testnet-faucet`](https://discord.com/channels/916379725201563759/1037811694564560966) สำหรับเหรียญ testnet
4. พิมพ์คำสั่ง `!faucet <WALLET ADDRESS>`