# Homogeneous Collections 

ก่อนที่เราจะเจาะลึกลงไปในหัวข้อหลักของการสร้าง marketplace บน Sui เราจะมาเรียนวิธีการสร้าง collections ใน Move กันก่อน

## Vectors

`Vector` ใน Move นั้นมีความคล้ายคลึงกับในภาษาอื่นๆ เช่น C++ มันใช้เพื่อจับจองหน่วยความจำในขณะ runtime และจัดการกับชุดของตัวแปรที่เป็นประเภทเดียวกัน โดยจะเป็น type แบบเจาะจงหรือเป็น [generic type](../../unit-three/lessons/2_intro_to_generics.md) ก็ได้

ดูโค้ดตัวอย่างสำหรับการสร้าง `vector` และการดำเนินการพื้นฐานต่างๆได้ที่นี่

```rust
module collection::vector {

    use std::vector;

    struct Widget {
    }

    // Vector for a specified  type
    struct WidgetVector {
        widgets: vector<Widget>
    }

    // Vector for a generic type 
    struct GenericVector<T> {
        values: vector<T>
    }

    // Creates a GenericVector that hold a generic type T
    public fun create<T>(): GenericVector<T> {
        GenericVector<T> {
            values: vector::empty<T>()
        }
    }

    // Push a value of type T into a GenericVector
    public fun put<T>(vec: &mut GenericVector<T>, value: T) {
        vector::push_back<T>(&mut vec.values, value);
    }

    // Pops a value of type T from a GenericVector
    public fun remove<T>(vec: &mut GenericVector<T>): T {
        vector::pop_back<T>(&mut vec.values)
    }

    // Returns the size of a given GenericVector
    public fun size<T>(vec: &mut GenericVector<T>): u64 {
        vector::length<T>(&vec.values)
    }
}

```

เรื่องสำคัญที่ต้องทราบคือ แม้ว่าจะประกาศ vector ด้วย generic type ที่สามารถรับ object *type อะไรก็ได้* แต่ทุกๆ object ใน collection นั้นยังคงต้องเป็น *type เดียวกันทั้งหมด* เราจึงเรียก collection นี้ว่า *homogeneous*

## Table

`Table` เป็น collection ที่คล้ายกับ map ที่มีการเก็บค่าแบบ key-value แต่จะไม่เหมือน map แบบเก่า โดย keys และ values ของมันจะไม่ได้ถูกเก็บไว้ในตาราง แต่จะถูกเก็บไว้โดยใช้ระบบของ Sui object แทน ตัว struct `Table` ทำหน้าที่แค่จัดการระบบ object เพื่อดึง keys และ values เหล่านั้นออกมาใช้งานเท่านั้น

ตัว `key` ของ `Table` ต้องมี ability constraint `copy + drop + store` และ `value` ต้องมี ability constraint `store` 

`Table` ยังเป็น collection ประเภท *homogeneous* อีกด้วย โดยที่ key และ value สามารถเป็น types แบบเจาะจง หรือ generic ก็ได้ แต่ทุก values และ keys ใน `Table` ต้องเป็นประเภท *เดียวกัน*

*แบบทดสอบ: Table สองตัวที่ประกอบไปด้วย key-value ที่เหมือนกันนั้น มีค่าเท่ากันมั๊ยเมื่อเช็คด้วยเครื่องหมาย `===` ลองไปเล่นดูสิ*

ดูตัวอย่างข้างล่างสำหรับการทำงานกับ `Table`:

```rust
module collection::table {

    use sui::table::{Table, Self};
    use sui::tx_context::{TxContext};

    // Defining a table with specified types for the key and value
    struct IntegerTable {
        table_values: Table<u8, u8>
    }

    // Defining a table with generic types for the key and value 
    struct GenericTable<phantom K: copy + drop + store, phantom V: store> {
        table_values: Table<K, V>
    }

    // Create a new, empty GenericTable with key type K, and value type V
    public fun create<K: copy + drop + store, V: store>(ctx: &mut TxContext): GenericTable<K, V> {
        GenericTable<K, V> {
            table_values: table::new<K, V>(ctx)
        }
    }

    // Adds a key-value pair to GenericTable
    public fun add<K: copy + drop + store, V: store>(table: &mut GenericTable<K, V>, k: K, v: V) {
        table::add(&mut table.table_values, k, v);
    }

    /// Removes the key-value pair in the GenericTable `table: &mut Table<K, V>` and returns the value.   
    public fun remove<K: copy + drop + store, V: store>(table: &mut GenericTable<K, V>, k: K): V {
        table::remove(&mut table.table_values, k)
    }

    // Borrows an immutable reference to the value associated with the key in GenericTable
    public fun borrow<K: copy + drop + store, V: store>(table: &GenericTable<K, V>, k: K): &V {
        table::borrow(&table.table_values, k)
    }

    /// Borrows a mutable reference to the value associated with the key in GenericTable
    public fun borrow_mut<K: copy + drop + store, V: store>(table: &mut GenericTable<K, V>, k: K): &mut V {
        table::borrow_mut(&mut table.table_values, k)
    }

    /// Check if a value associated with the key exists in the GenericTable
    public fun contains<K: copy + drop + store, V: store>(table: &GenericTable<K, V>, k: K): bool {
        table::contains<K, V>(&table.table_values, k)
    }

    /// Returns the size of the GenericTable, the number of key-value pairs
    public fun length<K: copy + drop + store, V: store>(table: &GenericTable<K, V>): u64 {
        table::length(&table.table_values)
    }

}
```
