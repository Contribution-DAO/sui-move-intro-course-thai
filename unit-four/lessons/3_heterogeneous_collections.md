# Heterogeneous Collections


Homogeneous collections อย่างเช่น `Vector` หรือ `Table` สามารถนำไปใช้ได้กับ marketplace (หรือแอปพลิเคชันประเภทอื่นๆ) ซึ่งเราจำเป็นต้องเก็บ collection ของ objects ที่เป็นประเภทเดียวกัน แต่ถ้าเราต้องเก็บ objects ที่ต่างประเภทกันล่ะ? หรือถ้าเราไม่ทราบล่วงหน้าว่าประเภทของ objects ที่เราจะเก็บคืออะไรล่ะ?

สำหรับ marketplaces ประเภทนี้ เราจำเป็นต้องใช้คอลเลคชั่น *heterogenerous* เพื่อเก็บ items ที่ถูกขาย เราได้ทำความเข้าใจเรื่อง dynamic fields กันอย่างหนักหน่วงแล้ว heterogenerous collection ควรจะเข้าใจได้อย่างง่ายดาย เราจะดูคอลเลคชั่นประเภท `Bag` อย่างละเอียดที่นี่

## The `Bag` Type

`Bag` เป็นคอลเลคชั่นคล้าย map แบบ heterogeneous ซึ่ง collection นี้คล้ายๆกับ `Table` โดยที่ keys และ values ไม่ได้ถูกเก็บไว้ใน `Bag` แต่เก็บไว้ในระบบ object ของ Sui แทน และตัว struct `Bag` จะทำหน้าที่ควบคุมการรับส่งค่า keys กับ values เหล่านั้นเท่านั้น

### Common `Bag` Operations

โค้ดตัวอย่างการดำเนินการต่างๆของ `Bag`:

```rust
module collection::bag {

    use sui::bag::{Bag, Self};
    use sui::tx_context::{TxContext};

    // Defining a table with generic types for the key and value
    struct GenericBag {
       items: Bag
    }

    // Create a new, empty GenericBag
    public fun create(ctx: &mut TxContext): GenericBag {
        GenericBag{
            items: bag::new(ctx)
        }
    }

    // Adds a key-value pair to GenericBag
    public fun add<K: copy + drop + store, V: store>(bag: &mut GenericBag, k: K, v: V) {
       bag::add(&mut bag.items, k, v);
    }

    /// Removes the key-value pair from the GenericBag with the provided key and returns the value.
    public fun remove<K: copy + drop + store, V: store>(bag: &mut GenericBag, k: K): V {
        bag::remove(&mut bag.items, k)
    }

    // Borrows an immutable reference to the value associated with the key in GenericBag
    public fun borrow<K: copy + drop + store, V: store>(bag: &GenericBag, k: K): &V {
        bag::borrow(&bag.items, k)
    }

    /// Borrows a mutable reference to the value associated with the key in GenericBag
    public fun borrow_mut<K: copy + drop + store, V: store>(bag: &mut GenericBag, k: K): &mut V {
        bag::borrow_mut(&mut bag.items, k)
    }

    /// Check if a value associated with the key exists in the GenericBag
    public fun contains<K: copy + drop + store>(bag: &GenericBag, k: K): bool {
        bag::contains<K>(&bag.items, k)
    }

    /// Returns the size of the GenericBag, the number of key-value pairs
    public fun length(bag: &GenericBag): u64 {
        bag::length(&bag.items)
    }
}
```

อย่างที่คุณเห็น functions signatures สำหรับการโต้ตอบกับคอลเลคชั่น `Bag` ค่อนข้างคล้ายกับวิธีการโต้ตอบกับคอลเลคชั่น `Table` โดยมีข้อแตกต่างที่สำคัญคือเราไม่จำเป็นต้องประกาศ type ให้มันตอนทำการสร้าง `Bag` อันใหม่ และประเภทของ key-value ที่เพิ่มเข้าไปไม่จำเป็นต้องเป็นประเภทเดียวกัน
