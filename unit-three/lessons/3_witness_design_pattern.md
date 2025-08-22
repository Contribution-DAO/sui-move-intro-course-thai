# รูปแบบการออกแบบพยาน (The Witness Design Pattern)

ต่อไป เราจะมาทำความเข้าใจรูปแบบการออกแบบระบบพยานเพื่อดูว่า fungible token ถูก implement อย่างไรใน Sui Move

พยาน (Witness) คือรูปแบบการออกแบบเพื่อใช้พิสูจน์ resource หรือ type ในคำถาม `A` สามารถเริ่มต้นได้หลังจากทรัพยากร `witness` ถูกใช้งานแล้วเท่านั้น โดยที่ `witness` ต้องถูก consumed หรือ dropped ทันทีหลังใช้งานเพื่อให้แน่ใจว่ามันจะไม่สามารถนำกลับมาใช้สร้างอินซแตนซ์ `A` ซ้ำได้อีก

## Witness Pattern Example

ในตัวอย่างข้างล่าง `witness` คือ `PEACE` ขณะที่ `A` คือตัวที่เราใช้ควบคุมการสร้างอินสแตนซ์ `Guardian`

ตัว `witness` ต้องมีคีย์เวิร์ด `drop` เพื่อให้มันถูกลบทิ้งหลังจากส่งให้ฟังก์ชั่น เราจะเห็นว่าอินสแตนซ์ `PEACE` ถูกส่งเข้าไปในฟังก์ชั่น `create_guardian` และทำการลบทิ้ง (สังเกตุที่เครื่องหมาย underscore ก่อน `witness`) ทำให้แน่ใจว่าจะมี `Guardian` เพียงตัวเดียวที่ถูกสร้างขึ้นมา

```move
/// Module that defines a generic type `Guardian<T>` which can only be
/// instantiated with a witness.
module witness::peace;

/// Phantom parameter T can only be initialized in the `create_guardian`
/// function. But the types passed here must have `drop`.
public struct Guardian<phantom T: drop> has key, store {
    id: UID
}

/// This type is the witness resource and is intended to be used only once.
public struct PEACE has drop {}

/// The first argument of this function is an actual instance of the
/// type T with `drop` ability. It is dropped as soon as received.
public fun create_guardian<T: drop>(_: T, ctx: &mut TxContext): Guardian<T> {
    Guardian { id: object::new(ctx) }
}

/// Module initializer is the best way to ensure that the
/// code is called only once. With `Witness` pattern it is
/// often the best practice.
fun init(witness: PEACE, ctx: &mut TxContext) {
    transfer::public_transfer(create_guardian(witness, ctx), ctx.sender())
}
```

*ตัวอย่างข้างบนถูกดัดแปลงมาจากสุดยอดหนังสือ [Sui Move by Example](https://examples.sui.io/patterns/witness.html) โดย [Damir Shamanaev](https://github.com/damirka).*

### คีย์เวิร์ด `phantom`

ในตัวอย่างด้านบน เราต้องการให้ `Guardian` มี ability `key` กับ `store` เพื่อให้มันเป็นสินทรัพย์ (asset) ที่สามารถโอนถ่ายไปมาได้ และอยู่ในที่จัดเก็บข้อมูลด้านนอก (global storage)

นอกจากนี้เรายังต้องการส่ง `witness`, `PEACE` ไปใน `Guardian` แต่ `PEACE` นั้นมีแค่ ability `drop` ซึ่งหากย้อนไปดูเนื้อหาก่อนหน้านี้เรื่อง [ability constraints](./2_intro_to_generics.md#ability-constraints) และ types ด้านใน กฎบอกไว้ว่า `PEACE` ควรจะมี `key` และ `storage` ด้วย เนื่องจากตัว type `Guardian` ด้านนอกมี แต่ในกรณีนี้ เราไม่ต้องการที่จะเพิ่ม abilities ที่ไม่จำเป็นให้กับ `witness` เพราะว่าการทำแบบนั้นอาจทำให้เกิดพฤติกรรมที่ไม่ต้องการและมีช่องโหว่ได้

เราสามารถใช้คีย์เวิร์ด `phantom` เพื่อแก้ไขสถานการณ์นี้ เมื่อพารามิเตอร์ไม่ถูกใช้ภายใน struct หรือใช้เป็น argument ให้พารามิเตอร์ `phantom` ตัวอื่นๆ เราสามารถใช้คีย์เวิร์ด `phantom` เพื่อสั่งให้ระบบ type ของ Move ผ่อนคลายกฎ ability constraint บน types ด้านใน เราจะเห็นว่า `Guardian` ไม่ได้ใช้ type `T` บนฟิลด์ไหนเลย ดังนั้นเราจึงสามารถประกาศ `T` เป็น type `phantom` ได้อย่างปลอดภัย

สำหรับคำอธิบายเชิงลึกเพิ่มเติมของคีย์เวิร์ด `phantom` โปรดดูที่ [relevant section](https://github.com/move-language/move/blob/main/language/documentation/book/src/generics.md#phantom-type-parameters) ของเอกสารภาษา Move

## One Time Witness

พยานแบบใช้ครั้งเดียว One Time Witness (OTW) คือรูปแบบย่อยของระบบพยาน โดยที่เราจะใช้ฟังก์ชั่น `init` ของโมดูล เพื่อให้แน่ใจว่าจะมีการสร้าง `witness` มาเพียงตัวเดียว (ดังนั้น type `A` จะการันตีได้ว่าเป็น singleton)

ใน Sui Move ตัว type จะถือว่าเป็น OTW เมื่อมีคุณสมบัติดังต่อไปนี้:

- type ถูกตั้งชื่อตามชื่อโมดูล แต่เป็นตัวพิมพ์ใหญ่
- type นั้นมี ability `drop` แค่เพียงอย่างเดียว

ในการรับอินสแตนซ์ของ type นี้ คุณต้องเพิ่มมันเป็น argument แรกให้ฟังก์ชั่น `init` ของโมดูล ดังในตัวอย่างด้านบน Sui runtime จะสร้าง struct OTW โดยอัตโนมัติในตอนที่เผยแพร่โมดูล

ตัวอย่างด้านบนมีการใช้รูปแบบการออกแบบ One Time Witness เพื่อการันตีว่า `Guardian` จะเป็น singleton
