# The `Coin` Resource and `create_currency` Method

ตอนนี้ เราได้รู้จักการทำงานของ generic และ witness แล้ว ทีนี้มาย้อนดูเรื่อง `Coin` และฟังก์ชั่น `create_currency` กัน

## The `Coin` Resource

ตอนนี้ หลังจากเราได้เข้าใจการทำงานของ generics เราสามารถกลับมาดูเรื่อง `Coin` ใน `sui::coin` ได้แล้ว ซึ่งมันถูก [defined](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/packages/sui-framework/sources/coin.move#L29) ไว้ดังนี้

```rust
struct Coin<phantom T> has key, store {
        id: UID,
        balance: Balance<T>
    }
```

ทรัพยากรประเภท `Coin` เป็น struct ที่มี generic type `T` และสองฟิลด์ `id` และ `balance` โดย `id` เป็นของ type `sui::object::UID` ซึ่งเราเคยเห็นกันมาแล้ว

`balance` เป็นของ type [`sui::balance::Balance`](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/balance.md#0x2_balance_Balance), ซึ่งถูก [defined](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/packages/sui-framework/sources/balance.move#L25) ดังนี้:

```rust
struct Balance<phantom T> has store {
    value: u64
}
```

หากย้อนดูที่เราคุยกันไว้เรื่อง [`phantom`](./3_witness_design_pattern.md#the-phantom-keyword), type `T` ถูกใช้ใน `Coin` เท่านั้น เพื่อเป็น argument ให้ phantom ตัวอื่นๆ สำหรับ `Balance` และใน `Balance` มันไม่ได้ถูกใช้ในฟิลด์ไหนเลย ดังนั้น `T` จึงเป็นพารามิเตอร์ประเภท `phantom`

`Coin<T>` เป็นตัวแทนของสินทรัพย์ (asset) ที่สามารถถ่ายโอนได้ของ fungible token ประเภท `T` ซึ่งสามารถถูกโอนระหว่างแอดเดรสหรือ consumed โดยการเรียกฟังก์ชั่นสมาร์ทคอนแทรค

## The `create_currency` Method

มาดูกันว่าจริงๆแล้วฟังก์ชั่น `coin::create_currency` สามารถทำอะไรได้บ้างใน [source code](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/packages/sui-framework/sources/coin.move#L251):

```rust
    public fun create_currency<T: drop>(
        witness: T,
        decimals: u8,
        symbol: vector<u8>,
        name: vector<u8>,
        description: vector<u8>,
        icon_url: Option<Url>,
        ctx: &mut TxContext
    ): (TreasuryCap<T>, CoinMetadata<T>) {
        // Make sure there's only one instance of the type T
        assert!(sui::types::is_one_time_witness(&witness), EBadWitness);

        // Emit Currency metadata as an event.
        event::emit(CurrencyCreated<T> {
            decimals
        });

        (
            TreasuryCap {
                id: object::new(ctx),
                total_supply: balance::create_supply(witness)
            },
            CoinMetadata {
                id: object::new(ctx),
                decimals,
                name: string::utf8(name),
                symbol: ascii::string(symbol),
                description: string::utf8(description),
                icon_url
            }
        )
    }
```

assert ใช้เช็คว่า witness ที่ถูกส่งเข้าไปคือ One Time Witness โดยใช้เมธอด [`sui::types::is_one_time_witness`](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/packages/sui-framework/sources/types.move) ของ Sui Framework

ฟังก์ชั่นทำการสร้าง และ return objects 2 ตัว หนึ่งคือ resource `TreasuryCap` และอีกอันคือ resource `CoinMetadata`

### `TreasuryCap`

`TreasuryCap` คือสินทรัพย์ (asset) และถูกรับประกันว่าเป็น singleton โดยรูปแบบ One Time Witness:

```rust
    /// Capability allowing the bearer to mint and burn
    /// coins of type `T`. Transferable
    struct TreasuryCap<phantom T> has key, store {
            id: UID,
            total_supply: Supply<T>
        }
```

มันมีตัวแปร singleton `total_supply` ของประเภท `Balance::Supply` อยู่ด้านใน:

```rust
/// A Supply of T. Used for minting and burning.
    /// Wrapped into a `TreasuryCap` in the `Coin` module.
    struct Supply<phantom T> has store {
        value: u64
    }
```

`Supply<T>` ใช้ติดตามจำนวนทั้งหมดของ custom fungible token `T` ที่กำลังหมุนเวียนอยู่ คุณจะเห็นว่าเหตุใดมันถึงต้องเป็น singleton เพราะเนื่องจากการมีอินสแตนซ์ `Supply` หลายรายการสำหรับโทเคนประเภทเดียวนั้นไม่สมเหตุสมผล

### `CoinMetadata`

นี่คือ resource ที่เก็บ metadata ของ fungible token ที่ถูกสร้างขึ้นมา ประกอบไปด้วยฟิลด์ต่างๆเหล่านี้::

- `decimals`: จำนวนหน่วยทศนิยม custom fungible token
- `name`: ชื่อของ custom fungible token
- `symbol`: คำอธิบายของ custom fungible token
- `description`: the description of this custom fungible token
- `icon_url`: url ของไฟล์ไอคอนของ custom fungible token

ข้อมูลต่างๆที่อยู่ใน `CoinMetadata` ถือเป็นข้อมูลมาตรฐานที่เป็นพื้นฐานของ fungible token ของ Sui และสามารถถูกใช้งานโดยกระเป๋า และ explorers เพื่อแสดง fungible token ที่ถูกสร้างโดยใช้โมดูล `sui::coin`
