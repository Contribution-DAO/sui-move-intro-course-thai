# Marketplace Contract

ตอนนี้ เรามีความเข้าอกเข้าใจเกี่ยวกับวิธีการทำงานของ collections และ dynamic fields ประเภทต่างๆแล้ว เราสามารถเริ่มเขียนคอนแทรคสำหรับ on-chain marketplace ที่สามารถรองรับฟีเจอร์เหล่านี้ได้แล้ว:

- ลิสไอเทมต่างๆได้หลากหลายประเภท
- รับชำระเงินด้วย fungible token ทั้งแบบ custom และแบบ native
- สามารถอนุญาติให้ผู้ขายหลายๆคนลงรายการสินค้าของตน และรับชำระเงินได้อย่างปลอดภัย ไปพร้อมกัน

## Type Definitions

ขั้นแรก เราประกาศ struct `Marketplace` ได้ดังนี้:

```rust
    /// A shared `Marketplace`. Can be created by anyone using the
    /// `create` function. One instance of `Marketplace` accepts
    /// only one type of Coin - `COIN` for all its listings.
    struct Marketplace<phantom COIN> has key {
        id: UID,
        items: Bag,
        payments: Table<address, Coin<COIN>>
    }
```

`Marketplace` เป็น object ที่แบ่งปันให้ทุกคนสามารถเข้าถึง และแก้ไขมันได้ มันรับพารามิเตอร์เป็น generic type `COIN` เพื่อกำหนดประเภทของ [fungible token](../../unit-three/lessons/4_the_coin_resource_and_create_currency.md) ที่ใช้สำหรับรับชำระเงิน

ฟิลด์ `items` ใช้เก็บรายการต่างๆหลากหลายประเภท ดังนั้นเราจึงใช้ heterogeneous collection `Bag` ที่นี่

ฟิลด์ `payment` ใช้เก็บข้อมูลการชำระเงินที่ได้รับจากผู้ขายแต่ละราย สิ่งนี้สามารถแสดงด้วยรูปแบบ key-value โดยที่แอดเดรสของผู้ขายเป็น key และประเภทของเหรียญที่จ่ายเป็น value และเพราะว่าประเภทของ key และ value เป็น homogeneous และคงที่ เราจึงสามารถใช้คอลเลคชั่นแบบ `Table` กับฟิลด์นี้ได้

_แบบทดสอบ: เราสามารถแก้ไข struct นี้ได้อย่างไร เพื่อให้มันรับ fungible token ได้หลายประเภท?_

ถัดไป เราจะทำการประกาศ `Listing`:

```rust
    /// A single listing which contains the listed item and its
    /// price in [`Coin<COIN>`].
    struct Listing has key, store {
        id: UID,
        ask: u64,
        owner: address,
    }
```
struct นี้ทำการเก็บข้อมูลต่างๆที่จำเป็นเกี่ยวกับการสร้างรายการขาย เราจะทำการแนบไอเทมจริงๆที่จะทำการซื้อขายไปที่ `Listing` เป็น dynamic object field ดังนั้นเราจึงไม่ต้องกำหนดฟิลด์ของไอเทม หรือคอลเลคชั่นใดๆ ที่นี่

สังเกตว่า `Listing` มี ability `key` เพราะว่าเราต้องการที่จะใช้ id ของมันเป็น key เมื่อใส่มันเข้าคอลเลคชั่น

## Listing and Delisting

ต่อไป เราจะเขียนลอจิกสำหรับสร้างรายการขาย และเพิกถอนรายการขาย มาเริ่มที่การสร้างรายการกันก่อน:

```rust
   /// List an item at the Marketplace.
    public entry fun list<T: key + store, COIN>(
        marketplace: &mut Marketplace<COIN>,
        item: T,
        ask: u64,
        ctx: &mut TxContext
    ) {
        let item_id = object::id(&item);
        let listing = Listing {
            ask,
            id: object::new(ctx),
            owner: tx_context::sender(ctx),
        };

        ofield::add(&mut listing.id, true, item);
        bag::add(&mut marketplace.items, item_id, listing)
    }
```
อย่างที่กล่าวไปก่อนหน้านี้ เราจะใช้ dynamic object field เพื่อใส่รายการประเภทใดก็ได้ที่จะขาย จากนั้นก็ทำการใส่ object `Listing` ลงใน `Bag` โดยใช้ id ของ object เป็น key และ object `Listing` เป็น value (นั่นเป็นสาเหตุว่าทำไม `Listing` ถึงมี ability `store`)

สำหรับการลบรายการขาย เราสามารถเขียนได้ดังนี้:

```rust
   /// Internal function to remove listing and get an item back. Only owner can do that.
    fun delist<T: key + store, COIN>(
        marketplace: &mut Marketplace<COIN>,
        item_id: ID,
        ctx: &mut TxContext
    ): T {
        let Listing {
            id,
            owner,
            ask: _,
        } = bag::remove(&mut marketplace.items, item_id);

        assert!(tx_context::sender(ctx) == owner, ENotOwner);

        let item = ofield::remove(&mut id, true);
        object::delete(id);
        item
    }

    /// Call [`delist`] and transfer item to the sender.
    public entry fun delist_and_take<T: key + store, COIN>(
        marketplace: &mut Marketplace<COIN>,
        item_id: ID,
        ctx: &mut TxContext
    ) {
        let item = delist<T, COIN>(marketplace, item_id, ctx);
        transfer::public_transfer(item, tx_context::sender(ctx));
    }
```

โปรดสังเกตวิธีแกะ และลบ object `Listing` ที่ถูกเพิกถอนออกจากรายการ และไอเทมในรายการถูกรับผ่าน [`ofield::remove`](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/packages/sui-framework/sources/dynamic_object_field.move#L71) จำไว้ว่า assets ของ Sui ไม่สามารถถูกทำลายนอกโมดูลที่มันถูกสร้างได้ ดังนั้นเราจึงต้องโอนมันไปให้ผู้ที่ทำการเพิกถอนรายการ

## Purchasing and Payments

การซื้อไอเทมนั้นคล้ายๆกับการเพิกถอนรายการสินค้า แต่จะมีลอจิกเพิ่มเติมสำหรับการชำระเงิน

```rust
    /// Internal function to purchase an item using a known Listing. Payment is done in Coin<C>.
    /// Amount paid must match the requested amount. If conditions are met,
    /// owner of the item gets the payment and buyer receives their item.
    fun buy<T: key + store, COIN>(
        marketplace: &mut Marketplace<COIN>,
        item_id: ID,
        paid: Coin<COIN>,
    ): T {
        let Listing {
            id,
            ask,
            owner
        } = bag::remove(&mut marketplace.items, item_id);

        assert!(ask == coin::value(&paid), EAmountIncorrect);

        // Check if there's already a Coin hanging and merge `paid` with it.
        // Otherwise attach `paid` to the `Marketplace` under owner's `address`.
        if (table::contains<address, Coin<COIN>>(&marketplace.payments, owner)) {
            coin::join(
                table::borrow_mut<address, Coin<COIN>>(&mut marketplace.payments, owner),
                paid
            )
        } else {
            table::add(&mut marketplace.payments, owner, paid)
        };

        let item = ofield::remove(&mut id, true);
        object::delete(id);
        item
    }

    /// Call [`buy`] and transfer item to the sender.
    public entry fun buy_and_take<T: key + store, COIN>(
        marketplace: &mut Marketplace<COIN>,
        item_id: ID,
        paid: Coin<COIN>,
        ctx: &mut TxContext
    ) {
        transfer::transfer(
            buy<T, COIN>(marketplace, item_id, paid),
            tx_context::sender(ctx)
        )
    }

```

ในส่วนแรกนั้นจะเหมือนกับการเพิกถอนรายการ แต่เราจะทำการตรวจสอบด้วยว่าจำนวนเงินที่รับชำระเข้ามานั้นถูกต้องหรือไม่ ในส่วนที่สองจะทำการใส่เหรียญที่รับชำระเข้ามาลงใน `payment` `Table` และขึ้นอยู่กับว่าผู้ขายมียอดเงินคงเหลืออยู่บ้างหรือไม่ มันจะเรียกฟังก์ชั่น `table::add` หรือ `table::borrow_mut` และ `coin::join` เพื่อรวมการชำระเงินนี้ลงในยอดคงเหลือ

ฟังก์ชั่นแรกเริ่ม `buy_and_take` ทำการเรียก `buy` และโอนไอเทมที่ซื้อไปให้ผู้ซื้อ

### Taking Profit

อย่างสุดท้าย เราจะทำการเขียนฟังก์ชั่นเพื่อให้ผู้ขายทำการถอนเงินออกมาจาก marketplace

```rust
   /// Internal function to take profits from selling items on the `Marketplace`.
    fun take_profits<COIN>(
        marketplace: &mut Marketplace<COIN>,
        ctx: &mut TxContext
    ): Coin<COIN> {
        table::remove<address, Coin<COIN>>(&mut marketplace.payments, tx_context::sender(ctx))
    }

    /// Call [`take_profits`] and transfer Coin object to the sender.
    public entry fun take_profits_and_keep<COIN>(
        marketplace: &mut Marketplace<COIN>,
        ctx: &mut TxContext
    ) {
        transfer::transfer(
            take_profits(marketplace, ctx),
            tx_context::sender(ctx)
        )
    }
```

_แบบทดสอบ: ทำไมเราไม่จำเป็นต้องใช้ [Capability](../../unit-two/lessons/6_capability_design_pattern.md) เพื่อควบคุมการเข้าถึง ในการออกแบบ marketplace นี้? เราสามารถใช้มันได้หรือไม่? ถ้าใช้แล้ว marketplace จะได้คุณสมบัติอะไรเพิ่มเข้ามา?_

## Full Contract

คุณสามารถดูวิธีการ implement smart contract แบบเต็มๆของ generic marketplace ได้ในโฟลเดอร์ [`example_projects/marketplace`](../example_projects/marketplace/sources/marketplace.move)
