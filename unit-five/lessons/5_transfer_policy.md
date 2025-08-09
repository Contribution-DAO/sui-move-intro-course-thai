# Transfer Policy และการซื้อสินค้าจาก Kiosk

ในหัวข้อนี้ เราจะเรียนรู้วิธีสร้าง `TransferPolicy` และใช้งานเพื่อบังคับให้ผู้ซื้อปฏิบัติตามกฎที่กำหนดไว้ก่อนที่จะได้รับความเป็นเจ้าของสินค้าที่ซื้อ

## `TransferPolicy`

### สร้าง `TransferPolicy`

`TransferPolicy` สำหรับชนิด `T` ต้องถูกสร้างขึ้นเพื่อให้ `T` สามารถซื้อขายได้ในระบบ Kiosk `TransferPolicy` เป็น shared object ที่ทำหน้าที่เป็นศูนย์กลางในการบังคับตรวจสอบว่าแต่ละธุรกรรมการซื้อเป็นไปตามนโยบายที่กำหนดไว้ก่อนที่สินค้าจะถูกโอนให้ผู้ซื้อ

```move
use sui::transfer_policy::{Self, TransferRequest, TransferPolicy, TransferPolicyCap};
use sui::package::{Self, Publisher};

public struct KIOSK has drop {}

fun init(witness: KIOSK, ctx: &mut TxContext) {
    let publisher = package::claim(otw, ctx);
    transfer::public_transfer(publisher, ctx.sender());
}

#[allow(lint(share_owned, self_transfer))]
/// Create new policy for type `T`
public fun new_policy(publisher: &Publisher, ctx: &mut TxContext) {
    let (policy, policy_cap) = transfer_policy::new<TShirt>(publisher, ctx);
    transfer::public_share_object(policy);
    transfer::public_transfer(policy_cap, ctx.sender());
}
```

การสร้าง `TransferPolicy<T>` จำเป็นต้องใช้หลักฐานว่าเป็นผู้เผยแพร่ `Publisher` ของ module ที่มี type `T` อยู่ เพื่อให้แน่ใจว่ามีเพียงผู้สร้างชนิด `T` เท่านั้นที่สามารถสร้าง `TransferPolicy<T>` ได้ มี 2 วิธีในการสร้าง policy:

- ใช้ `transfer_policy::new()` เพื่อสร้าง policy ใหม่, ทำให้ `TransferPolicy` เป็น shared object และโอน `TransferPolicyCap` ให้กับผู้เรียกด้วย `sui::transfer`


```bash
sui client call --package $KIOSK_PACKAGE_ID --module kiosk --function new_policy --args $KIOSK_PUBLISHER
```

- ใช้ `entry transfer_policy::default()` เพื่อดำเนินการขั้นตอนทั้งหมดให้เราโดยอัตโนมัติ

เมื่อคุณ publish package แล้ว ควรได้รับ object `Publisher` มา ให้เรา export เอาไว้ใช้งานต่อ

```bash
export KIOSK_PUBLISHER=<Publisher object ID>
```

คุณควรจะเห็น object `TransferPolicy` และ `TransferPolicyCap` ที่เพิ่งถูกสร้าง ให้ export เอาไว้ใช้ต่ออีกเช่นกัน

```bash
export KIOSK_TRANSFER_POLICY=<TransferPolicy object ID>
export KIOSK_TRANSFER_POLICY_CAP=<TransferPolicyCap object ID>
```

### สร้างกฎการเก็บค่าธรรมเนียมคงที่

`TransferPolicy` จะไม่ทำงานหากไม่มี rule ใด ๆ กำหนดไว้ เราจะเรียนรู้วิธีสร้างกฎง่าย ๆ ใน module แยกต่างหากเพื่อบังคับให้ผู้ซื้อจ่ายค่าลิขสิทธิ์คงที่ (royalty fee) เพื่อให้การซื้อขายสำเร็จ

_💡หมายเหตุ: มี template มาตรฐานสำหรับสร้างกฎ ดูได้ที่ [rule template นี้](../example_projects/kiosk/sources/dummy_policy.move)_

#### Rule Witness & Rule Config

```move
module kiosk::fixed_royalty_rule;

/// The `amount_bp` passed is more than 100%.
const EIncorrectArgument: u64 = 0;
/// The `Coin` used for payment is not enough to cover the fee.
const EInsufficientAmount: u64 = 1;

/// Max value for the `amount_bp`.
const MAX_BPS: u16 = 10_000;

/// Rule Witness ใช้ยืนยัน policy
public struct Rule has drop {}

/// ค่าคอนฟิกสำหรับ Rule
public struct Config has store, drop {
    /// เปอร์เซ็นต์ค่าธรรมเนียม
    amount_bp: u16,
    /// ค่าธรรมเนียมขั้นต่ำ จะถูกใช้เมื่อค่าธรรมเนียมที่คำนวณได้น้อยกว่าค่านี้
    min_amount: u64,
}
```

`Rule` คือ witness สำหรับผูกกับ `TransferPolicy` มันช่วยในการระบุและแยกแยะเวลามาหลายๆ rule เพิ่มเข้ามาใน policy เดียวกัน `Config` คือค่าคอนฟิกของ `Rule` ซึ่งในกรณีนี้เราใช้เพื่อกำหนดค่าธรรมเนียมลิขสิทธิ์แบบคงที่ (fixed royalty fee) ดังนั้นการตั้งค่านี้ควรรวมถึงเปอร์เซ็นต์ที่ต้องการหักจากจำนวนเงินที่ชำระทั้งหมด

#### เพิ่ม Rule เข้า TransferPolicy

```move
/// ฟังก์ชันสำหรับเพิ่ม Rule เข้าไปใน `TransferPolicy`
/// ต้องใช้ `TransferPolicyCap` เพื่อให้แน่ใจว่า
/// สามารถเพิ่มกฎได้เฉพาะผู้เผยแพร่ (publisher) ของ T เท่านั้น
public fun add<T>(
    policy: &mut TransferPolicy<T>,
    cap: &TransferPolicyCap<T>,
    amount_bp: u16,
    min_amount: u64

) {
    assert!(amount_bp <= MAX_BPS, EIncorrectArgument);
    transfer_policy::add_rule(
        Rule {},
        policy,
        cap,
        Config { amount_bp, min_amount },
    )
}
```

ใช้ `transfer_policy::add_rule()` เพิ่ม rule พร้อมคอนฟิกเข้าสู่ policy

จากฝั่ง client ให้เรียกฟังก์ชันนี้เพื่อเพิ่ม `Rule` ลงใน `TransferPolicy` ไม่เช่นนั้น policy จะยังไม่มีการบังคับใช้ ตัวอย่างนี้กำหนดค่าธรรมเนียมไว้ที่ `0.1%` หรือ `10 basis points` และค่าขั้นต่ำที่ `100 MIST`

```bash
sui client call --package $KIOSK_PACKAGE_ID --module fixed_royalty_rule --function add --args $KIOSK_TRANSFER_POLICY $KIOSK_TRANSFER_POLICY_CAP 10 100 --type-args $KIOSK_PACKAGE_ID::kiosk::TShirt
```

#### ปฏิบัติตามกฎ (Satisfy the Rule)

```move
/// การกระทำของผู้ซื้อ: ชำระค่าธรรมเนียมลิขสิทธิ์สำหรับการโอน
public fun pay<T: key + store>(
    policy: &mut TransferPolicy<T>,
    request: &mut TransferRequest<T>,
    payment: Coin<SUI>,
) {
    let paid = transfer_policy::paid(request);
    let amount = fee_amount(policy, paid);

    assert!(payment.value() == amount, EInsufficientAmount);

    transfer_policy::add_to_balance(Rule {}, policy, payment);
    transfer_policy::add_receipt(Rule {}, request)
}

/// ฟังก์ชันช่วยคำนวณจำนวนเงินที่จะต้องจ่ายสำหรับการโอน
/// สามารถใช้รันแบบจำลอง (dry-run) เพื่อประเมินค่าธรรมเนียมจากราคาที่ประกาศขายใน Kiosk ได้
public fun fee_amount<T: key + store>(
    policy: &TransferPolicy<T>,
    paid: u64,
): u64 {
    let config: &Config = transfer_policy::get_rule(Rule {}, policy);
    let mut amount = (
        ((paid as u128) * (config.amount_bp as u128) / 10_000) as u64,
    );

    // ถ้าจำนวนเงินน้อยกว่าค่าขั้นต่ำ ให้ใช้ค่าขั้นต่ำแทน
    if (amount < config.min_amount) {
        amount = config.min_amount
    };

    amount
}
```

เราต้องการฟังก์ชัน `fee_amount()` เพื่อช่วยคำนวณค่าธรรมเนียมลิขสิทธิ์ตามนโยบาย (`policy`) และจำนวนเงินที่ชำระ โดยใช้ `transfer_policy::get_rule()` เพื่อดึงค่าการตั้งค่าและใช้ในการคำนวณ

`pay()` เป็นฟังก์ชันที่ผู้ใช้จะต้องเรียกด้วยตนเองเพื่อเติมเต็ม `TransferRequest` (ซึ่งจะกล่าวถึงในส่วนถัดไป) ก่อนเรียก `transfer_policy::confirm_request()` โดย `transfer_policy::paid()` จะคืนค่าการชำระเงินดั้งเดิมจากการเทรดที่อยู่ใน `TransferRequest` หลังจากคำนวณค่าธรรมเนียมแล้ว เราจะเพิ่มค่าธรรมเนียมเข้าไปใน policy ด้วย `transfer_policy::add_to_balance()` โดยค่าธรรมเนียมที่สะสมไว้ใน policy นี้ เจ้าของ `TransferPolicyCap` สามารถถอนออกไปภายหลังได้ สุดท้าย เราใช้ `transfer_policy::add_receipt()` เพื่อระบุว่า `TransferRequest` นี้ผ่านกฎแล้ว และพร้อมสำหรับการยืนยันด้วย `transfer_policy::confirm_request()`

## การซื้อสินค้าจาก Kiosk

```move
use sui::transfer_policy::{Self, TransferRequest, TransferPolicy};

/// ซื้อสินค้าที่ประกาศขายไว้
public fun buy(
    kiosk: &mut Kiosk,
    item_id: object::ID,
    payment: Coin<SUI>,
): (TShirt, TransferRequest<TShirt>) {
    kiosk.purchase(item_id, payment)
}

/// ยืนยัน TransferRequest
public fun confirm_request(
    policy: &TransferPolicy<TShirt>,
    req: TransferRequest<TShirt>,
) {
    policy.confirm_request(req);
}
```

เมื่อผู้ซื้อทำการซื้อสินค้าผ่าน API `kiosk::purchase()` จะได้รับสินค้าพร้อมกับ `TransferRequest` กลับมา `TransferRequest` นี้เป็นลักษณะของ *hot potato* ซึ่งบังคับให้เราต้องใช้มันผ่าน `transfer_policy::confirm_request()` เท่านั้น หน้าที่ของ `transfer_policy::confirm_request()` คือการตรวจสอบว่าผู้ใช้ได้ปฏิบัติตามกฎทั้งหมดที่ถูกตั้งค่าและเปิดใช้งานใน `TransferPolicy` แล้วหรือยัง หากมีกฎใดกฎหนึ่งไม่ถูกปฏิบัติตตาม ฟังก์ชันนี้จะเกิด error ทำให้ธุรกรรมล้มเหลว และส่งผลให้แม้ผู้ซื้อจะโอนสินค้ามายังบัญชีของตนก่อนหน้านั้นแล้ว แต่จะยังไม่ถือว่ามีเป็นเจ้าของสินค้านั้น

_💡หมายเหตุ: ผู้ใช้จะต้องรวมทุกการเรียกฟังก์ชันที่จำเป็นไว้ใน PTB (Programmable Transaction Block) เพื่อทำให้ TransferRequest สมบูรณ์ก่อนที่จะเรียก `confirm_request()`_

ลำดับการทำงานจะเป็นดังนี้:

_ผู้ซื้อ -> `kiosk::purchase()` -> `Item` + `TransferRequest` -> เรียกฟังก์ชันที่เกี่ยวข้องเพื่อเติมเต็ม `TransferRequest` -> `transfer_policy::confirm_request()` -> โอนกรรมสิทธิ์ของ `Item`_

## ตัวอย่างการซื้อสินค้าจาก Kiosk แบบครบขั้นตอน

จาก section ก่อนหน้า สินค้าต้องถูกวางไว้ใน Kiosk และต้องถูกประกาศขายก่อนจึงจะสามารถซื้อได้ สมมุติว่าได้มีการประกาศขายแล้วในราคาสินค้า `10_000 MIST` ให้เราสร้างตัวแปรใน terminal ที่เก็บค่า Object ID ของสินค้าที่ถูกประกาศขาย:

```bash
export KIOSK_TSHIRT=<Object ID of the listed TShirt>
```

มาสร้าง PTB เพื่อทำการซื้อขายกัน ขั้นตอนค่อนข้างตรงไปตรงมา คือ เราซื้อสินค้าที่ถูกประกาศขายจาก kiosk จากนั้นจะได้รับสินค้าพร้อมกับ `TransferRequest` กลับมา แล้วเราจะเรียก `fixed_royalty_fee::pay` เพื่อชำระและทำให้ `TransferRequest` สมบูรณ์ จากนั้นยืนยัน TransferRequest ด้วย `transfer_policy::confirm_request()` ก่อนที่จะโอน `Item` ให้กับผู้ซื้อในขั้นตอนสุดท้าย

```bash
sui client ptb \
--assign price 10000 \
--split-coins gas "[price]" \
--assign coin \
--move-call $KIOSK_PACKAGE_ID::kiosk::buy @$KIOSK @$KIOSK_TSHIRT coin.0 \
--assign buy_res \
--move-call $KIOSK_PACKAGE_ID::fixed_royalty_rule::fee_amount "<$KIOSK_PACKAGE_ID::kiosk::TShirt>" @$KIOSK_TRANSFER_POLICY price \
--assign fee_amount \
--split-coins gas "[fee_amount]"\
--assign coin \
--move-call $KIOSK_PACKAGE_ID::fixed_royalty_rule::pay "<$KIOSK_PACKAGE_ID::kiosk::TShirt>" @$KIOSK_TRANSFER_POLICY buy_res.1 coin.0 \
--move-call $KIOSK_PACKAGE_ID::kiosk::confirm_request  @$KIOSK_TRANSFER_POLICY buy_res.1 \
--move-call 0x2::transfer::public_transfer "<$KIOSK_PACKAGE_ID::kiosk::TShirt>" buy_res.0 <buyer address> \

```
