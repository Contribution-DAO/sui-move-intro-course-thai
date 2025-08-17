# การใช้งาน Kiosk พื้นฐาน

## การสร้าง Kiosk

เริ่มต้นด้วยการ deploy smart contract ตัวอย่างของ kiosk และ export ค่า Package ID ไว้ใช้ต่อภายหลัง

```bash
export KIOSK_PACKAGE_ID=<Package ID of example kiosk smart contract>
```

```move
module kiosk::kiosk;
use sui::kiosk::{Self, Kiosk, KioskOwnerCap};

#[allow(lint(share_owned, self_transfer))]
/// สร้าง kiosk ใหม่
public fun new_kiosk(ctx: &mut TxContext) {
    let (kiosk, kiosk_owner_cap) = kiosk::new(ctx);
    transfer::public_share_object(kiosk);
    transfer::public_transfer(kiosk_owner_cap, ctx.sender());
}
```

มี 2 วิธีในการสร้าง kiosk ใหม่:

1. ใช้ `kiosk::new()` เพื่อสร้าง kiosk ใหม่ แต่เราต้องทำให้ `Kiosk` เป็น shared object และโอน `KioskOwnerCap` ให้กับผู้เรียกใช้เองโดยใช้ `sui::transfer`

```bash
sui client call --package $KIOSK_PACKAGE_ID --module kiosk --function new_kiosk
```

2. ใช้ `entry kiosk::default()` ซึ่งจะดำเนินการทุกขั้นตอนข้างต้นให้อัตโนมัติ

คุณสามารถ export `Kiosk` และ `KioskOwnerCap` ที่ถูกสร้างใหม่ไว้ใช้งานต่อได้

```bash
export KIOSK=<Object id ของ Kiosk ที่เพิ่งสร้างใหม่>
export KIOSK_OWNER_CAP=<Object id ของ KioskOwnerCap ที่เพิ่งสร้างใหม่>
```

_💡หมายเหตุ: Kiosk เป็น heterogeneous collection (รองรับหลายชนิด object) โดยค่าเริ่มต้น จึงไม่ต้องกำหนด type parameter สำหรับ item ที่ใส่_

## วาง Item ลงใน Kiosk

```move
public struct TShirt has key, store {
    id: UID,
}

public fun new_tshirt(ctx: &mut TxContext): TShirt {
    TShirt {
        id: object::new(ctx),
    }
}

/// วาง item ลงใน kiosk
public fun place(kiosk: &mut Kiosk, cap: &KioskOwnerCap, item: TShirt) {
    kiosk.place(cap, item)
}
```

เราสามารถใช้ API `kiosk::place()` เพื่อวาง item ลงใน kiosk ได้ อย่าลืมว่าเฉพาะเจ้าของ Kiosk เท่านั้นที่สามารถเข้าถึง API นี้ได้

## การถอน Item ออกจาก Kiosk

```move
/// ถอน item ออกจาก kiosk
public fun withdraw(
    kiosk: &mut Kiosk,
    cap: &KioskOwnerCap,
    item_id: object::ID,
): TShirt {
    kiosk.take(cap, item_id)
}
```

เราสามารถใช้ API `kiosk::take()` เพื่อถอน item ออกจาก kiosk ได้ อย่าลืมว่าเฉพาะเจ้าของ Kiosk เท่านั้นที่สามารถเข้าถึง API นี้ได้

## ลงรายการขาย

```move
/// ลงรายการขาย
public fun list(
    kiosk: &mut Kiosk,
    cap: &KioskOwnerCap,
    item_id: object::ID,
    price: u64,
) {
    kiosk.list<TShirt>(cap, item_id, price)
}
```

เราสามารถใช้ API `kiosk::take()` เพื่อประกาศขาย item ได้ อย่าลืมว่าเฉพาะเจ้าของ Kiosk เท่านั้นที่สามารถเข้าถึง API นี้ได้
