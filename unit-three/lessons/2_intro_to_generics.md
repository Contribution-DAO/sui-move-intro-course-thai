# Intro to Generics

Generics คือตัวแปรไม่คงที่ เป็นนามธรรม ทำงานคล้ายๆกับ [generics in Rust](https://doc.rust-lang.org/stable/book/ch10-00-generics.html) ใช้เพื่อเพิ่มความยืดหยุ่นและหลีกเลี่ยงการเขียนลอจิกซ้ำๆ เวลาเขียนโค้ด Sui Move

Generic เป็นคอนเซปหลักของ Sui Move เป็นสิ่งสำคัญที่เราต้องทำความเข้าใจ และรู้หลักการทำงานของมัน ดังนั้นจงใช้เวลาของคุณให้เต็มที่ในการทำความเข้าใจเนื้อหาตอนนี้อย่างถ่องแท้

## การใช้งาน Generics

### การใช้งาน Generics ใน Structs

ลองดูตัวอย่างง่ายๆของการใช้ generics ในการสร้างคอนเทนเนอร์ `Box` ที่สามารถเก็บ type อะไรก็ได้

ก่อนอื่น ถ้าไม่ใช้ generics เราสามารถสร้าง `Box` ที่เก็บค่า `u64` ได้ดังนี้:

```rust
module Storage {
    struct Box {
        value: u64
    }
}
```

อย่างไรก็ตาม มันจะสามารถเก็บค่าที่เป็นประเภท `u64` ได้เท่านั้น เพื่อที่จะให้ `Box` ของเราสามารถเก็บค่าอะไรก็ได้ เราจะใช้ generics ซึ่งสามารถแก้โค้ดได้ เป็นดังนี้:

```rust
module Storage {
    struct Box<T> {
        value: T
    }
}
```

#### Ability Constraints

เราสามารถใส่เงื่อนไขเพื่อบังคับให้ประเภทของตัวแปรที่ส่งเข้าใน generic ต้องมี abilities ที่ต้องการได้ ซึ่ง syntax จะมีหน้าตาแบบนี้:

```rust
module Storage {
    // T must be copyable and droppable 
    struct Box<T: store + drop> has key, store {
        value: T
    }
}
```

💡สิ่งสำคัญที่ต้องทราบคือ type ของ `T` ด้านในวงเล็บ ในตัวอย่างด้านบนต้องมี ability constraints ตรงกับ type ของคอนเทนเนอร์ด้านนอก ในตัวอย่างนี้ `T` ต้องมี `store` ดังที่ `Box` มี `store` และ `key` อย่างไรก็ตาม `T` ก็ยังสามารถมี abilities ที่คอนเทนเนอร์ไม่มีได้ เช่น `drop` เป็นต้น

ลองคิดดูง่ายๆว่าถ้าคอนเทนเนอร์อนุญาตให้ตัวแปรด้านในมี type ที่ตัวเองไม่มี คอนเทนเนอร์ก็จะละเมิดกฎของตัวเอง แบบนี้ box จะใช้งานได้อย่างไรถ้าของข้างในมันไม่สามารถเก็บค่าได้?

เราจะได้เห็นในตอนถัดไปถึงวิธีการหลีกเลี่ยงกฎนี้ในบางกรณีโดยใช้คีย์เวิร์ดพิเศษ ชื่อว่า `phantom` 

*💡ดู [generics project](../example_projects/generics/) ในโฟลเดอร์ `example_projects` สำหรับตัวอย่างตัวอย่างของ generic types*

### การใช้งาน Generics ในฟังก์ชั่น

ในการเขียนฟังก์ชั่นเพื่อให้ return อินสแตนซ์ของ `Box` โดยที่สามารถรับพารามิเตอร์ `value` เป็น type อะไรก็ได้ เราก็ต้องใช้ generics ในการประกาศฟังก์ชั่นเช่นกัน โดยฟังก์ชั่นสามารถเขียนได้ดังนี้

```rust
public fun create_box<T>(value: T): Box<T> {
        Box<T> { value }
    }
```

หากเราต้องการจำกัดให้ฟังก์ชั่นรับค่า `value` เฉพาะ type ที่ต้องการ เราสามารถระบุ type ให้มันได้ดังนี้:

```rust
public fun create_box(value: u64): Box<u64> {
        Box<u64>{ value }
    }
```

สิ่งนี้จะทำให้เมธอด `create_box` รับค่า `u64` ได้เท่านั้น ในขณะที่ยังคงใช้ `Box` เป็นแบบ generic เหมือนเดิม

#### การเรียกใช้งานฟังก์ชั่นที่มี Generics

การเรียกใช้งานฟังก์ชั่นที่มี generics เราต้องระบุ type ในวงเล็บเหลี่ยม เหมือนกับใน syntax ดังนี้:

```rust
// value will be of type Storage::Box<bool>
    let bool_box = Storage::create_box<bool>(true);
// value will be of the type Storage::Box<u64>
    let u64_box = Storage::create_box<u64>(1000000);
```

#### การเรียกใช้งานฟังก์ชั่นที่มี Generics ด้วย Sui CLI

การเรียกใช้งานฟังก์ชั่นที่มี Generics ด้วย Sui CLI คุณต้องกำหนดประเภทของ argument ให้มัน โดยใช้ `--type-args`

ต่อไปนี้คือตัวอย่างของการเรียกฟังก์ชั่น `create_box` เพื่อสร้าง box ที่บรรจุเหรียญประเภท  `0x2::sui::SUI` เอาไว้:

```bash
sui client call --package $PACKAGE --module $MODULE --function "create_box" --args $OBJECT_ID --type-args 0x2::sui::SUI --gas-budget 10000
```

## Advanced Generics Syntax

สำหรับ syntax ขั้นสูงที่เกี่ยวข้องกับการใช้ generics ใน Sui Move เช่น generic หลากหลายประเภท โปรดดูที่ [section on generics in the Move Book](https://move-book.com/advanced-topics/understanding-generics.html). 

แต่ในบทเรียนของเราเกี่ยวกับ fungible token คุณมีความรู้เพียงพอแล้วว่า generics ทำงานอย่างไร
