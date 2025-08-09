# ตัวอย่าง Clock และ Locked Coin

ในตัวอย่างที่สองของโทเคนแบบเปลี่ยนมือได้ (fungible token) เราจะสาธิตวิธีการดึงข้อมูลเวลาแบบ on-chain บน Sui และวิธีใช้ข้อมูลนั้นเพื่อสร้างกลไกการล็อกเหรียญแบบปลดทีละส่วน (vesting mechanism)

## นาฬิกา (Clock)

Sui Framework มี [clock module](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/clock.md) ซึ่งเป็นโมดูลแบบ native ที่ช่วยให้สามารถเรียกดู timestamp ได้ภายใน smart contract ที่เขียนด้วยภาษา Move

เมธอดหลักที่คุณจะต้องเรียกใช้งานคือ:

```
public fun timestamp_ms(clock: &clock::Clock): u64
```

ฟังก์ชัน [`timestamp_ms`](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/clock.md#function-timestamp_ms) จะคืนค่าเวลาปัจจุบันในรูปแบบ timestamp เป็นมิลลิวินาที นับต่อเนื่องจากจุดเวลาหนึ่งในอดีต

ออปเจ็กต์ [`clock`](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/clock.md#sui_clock_Clock) มี identifier พิเศษที่สงวนไว้คือ `0x6` ซึ่งจำเป็นต้องส่งเข้าไปเป็นพารามิเตอร์เมื่อเรียกใช้ฟังก์ชัน

## เหรียญที่ถูกล็อก (Locked Coin)

เมื่อเราสามารถเข้าถึงเวลาบน chain ได้โดยใช้ `clock` การสร้างเหรียญแบบ vesting ก็จะสามารถทำได้ง่ายขึ้น

### `ชนิดข้อมูล `Locker` แบบกำหนดเอง

`locked_coin` ถูกพัฒนาต่อจาก `managed_coin` โดยเพิ่มชนิดข้อมูลใหม่ชื่อ `Locker`:

```move
/// วัตถุแบบโอนย้ายได้สำหรับจัดเก็บเหรียญที่มีการล็อกไว้
public struct Locker has key, store {
    id: UID,
    start_date: u64,
    final_date: u64,
    original_balance: u64,
    current_balance: Balance<LOCKED_COIN>

}
```

Locker คือ [ทรัพย์สิน](https://github.com/sui-foundation/sui-move-intro-course/blob/main/unit-one/lessons/3_custom_types_and_abilities.md#assets) แบบโอนย้ายได้ ซึ่งเก็บข้อมูลเกี่ยวกับตารางเวลาการปลดล็อกและสถานะของเหรียญ

`start_date` และ `final_date` คือ timestamp ที่ได้จาก `clock` เพื่อระบุช่วงเวลาเริ่มและสิ้นสุดของการปลดล็อก

`original_balance` คือจำนวนเหรียญตั้งต้นที่ใส่ไว้ใน `Locker` , `balance` คือจำนวนเหรียญปัจจุบันที่ยังคงเหลือหลังจากหักส่วนที่ปลดล็อกออกไปแล้ว

### การสร้างเหรียญ (Minting)

ในเมธอด `locked_mint` เราจะสร้างและส่ง `Locker` ที่บรรจุจำนวนเหรียญและตารางเวลาการล็อกไว้:

```move
/// สร้างและโอนวัตถุ locker พร้อมกับจำนวนเหรียญที่ระบุและตารางเวลาการปลดล็อก
public fun locked_mint(
    treasury_cap: &mut TreasuryCap<LOCKED_COIN>,
    recipient: address,
    amount: u64,
    lock_up_duration: u64,
    clock: &Clock,
    ctx: &mut TxContext,
) {
    let coin = treasury_cap.mint(amount, ctx);
    let start_date = clock.timestamp_ms();
    let final_date = start_date + lock_up_duration;

    transfer::public_transfer(
        Locker {
            id: object::new(ctx),
            start_date,
            final_date,
            original_balance: amount,
            current_balance: coin.into_balance(),
        },
        recipient,
    );
}
```

สังเกตว่า `clock` ถูกใช้เพื่อดึงเวลาปัจจุบัน

### การถอนเหรียญ (Withdrawing)

เมธอด `withdraw_vested` มีลอกจิกหลักๆที่ใช้สำหรับคำนวณจำนวนเหรียญที่สามารถปลดล็อกได้แล้ว

```move
/// ถอนจำนวนเหรียญที่ถูกปลดล็อกแบบเชิงเส้น (linear vesting)
public fun withdraw_vested(
    locker: &mut Locker,
    clock: &Clock,
    ctx: &mut TxContext,
) {
    let total_duration = locker.final_date - locker.start_date;
    let elapsed_duration = clock.timestamp_ms() - locker.start_date;
    let total_vested_amount = if (elapsed_duration > total_duration) {
        locker.original_balance
    } else {
        locker.original_balance * elapsed_duration / total_duration
    };
    let available_vested_amount =
        total_vested_amount - (locker.original_balance - locker.current_balance.value());
    transfer::public_transfer(
        coin::take(&mut locker.current_balance, available_vested_amount, ctx),
        ctx.sender(),
    )
}
```

ตัวอย่างนี้สมมติว่าการปลดล็อกเหรียญเป็นแบบเชิงเส้น (linear vesting) แต่สามารถปรับให้รองรับเงื่อนไขหรือรูปแบบอื่นๆได้ตามต้องการ

### สัญญาฉบับเต็ม

คุณสามารถดูสมาร์ทคอนแทรคฉบับเต็มของ [`locked_coin`](../example_projects/locked_coin/sources/locked_coin.move) ได้ที่โฟลเดอร์ [example_projects/locked_coin](../example_projects/locked_coin/)