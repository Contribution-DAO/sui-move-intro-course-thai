# การเข้ารหัส BCS

Binary Canonical Serialization หรือ BCS คือ รูปแบบการทำให้เป็นอนุกรม (serialization) ถูกพัฒนาขึ้นในบริบทของบล็อกเชน Diem และขณะนี้ถูกใช้งานอย่างกว้างขวางในบล็อกเชนส่วนใหญ่ที่ใช้ Move (Sui, Starcoin, Aptos, 0L) BCS ไม่เพียงแต่ใช้ใน Move VM เท่านั้น แต่ยังใช้ในการเขียนโค้ดธุรกรรม และเหตุการณ์ เช่น การ serializing รายการธุรกรรมก่อนลงนาม, หรือการแปลงค่าข้อมูลเหตุการณ์ เป็นต้น

การรู้ว่า BCS ทำงานอย่างไรนั้นมีความสำคัญอย่างยิ่ง หากคุณต้องการทำความเข้าใจว่า Move ทำงานอย่างไรในระดับที่ลึกขึ้น และกลายเป็นผู้เชี่ยวชาญในเรื่องนี้ มาลุยกันเถอะ

## ข้อมูลจำเพาะ และคุณสมบัติของ BCS

มีคุณสมบัติระดับสูงบางประการของการเข้ารหัส BCS ที่เราควรจะทราบ ในการที่จะอ่านบทเรียนที่เหลือ:

- BCS คือ การทำข้อมูลให้เป็นอนุกรม (data-serialization) โดยให้ผลลัพธ์เป็น bytes ที่ไม่มีข้อมูลประเภทใดๆ; ด้วยเหตุนี้ ฝั่งที่รับค่า bytes ที่เข้ารหัสนั้นจำเป็นต้องรู้วิธีการแกะข้อมูล (deserialize)
- ใน BCS นั้น ไม่มี structs (เนื่องจากมันไม่มี types); struct นั้นใช้เพียงแค่เพื่อกำหนดลำดับให้แต่ละฟิลด์ที่ถูก serialized
- ประเภทข้องตัวหุ้ม (Wrapper) จะถูกละเว้น ดังนั้น `OuterType` และ `UnnestedType` จะมีการแสดง BCS เหมือนกัน:

    ```move
    public struct OuterType {
        owner: InnerType
    }
    public struct InnerType {
        address: address
    }
    public struct UnnestedType {
        address: address
    }
    ```
- ตัวแปรประเภทที่มีฟิลด์เป็น generic types สามารถถูกแปลงได้จนถึงฟิลด์แรกที่เป็น generic type ดังนั้น แนวปฏิบัติที่ดีคือให้ใส่ตัวแปรที่เป็น generic type ไว้ท้ายสุด ถ้าเรามีตัวแปรแบบปรับแต่งเอง (custom type) ที่ต้องการจะทำ ser/de'd.
    ```move
    public struct BCSObject<T> has drop, copy {
        id: ID,
        owner: address,
        meta: Metadata,
        generic: T
    }
    ```
    ในตัวอย่างนี้ เราสามารถ deserialize ข้อมูลทุกอย่างได้จนถึงฟิลด์ meta
- ประเภทแบบดั้งเดิม (Primitive types) เช่น unsigned int จะถูกเข้ารหัสในรูปแบบ Little Endian
- เวกเตอร์ จะถูก serialized ด้วย [ULEB128](https://en.wikipedia.org/wiki/LEB128) (โดยมีความยาวสูงสุดถึง `u32`) ตามด้วยเนื้อหาของเวกเตอร์

ข้อมูลจำเพาะของ BCS แบบเต็มๆ สามารถดูได้ที่ [the BCS repository](https://github.com/zefchain/bcs).

## การใช้งานไลบารีจาวาสคริป `@mysten/bcs`

### การติดตั้ง

ไลบารีที่คุณต้องทำการติดตั้งในส่วนนี้คือ [@mysten/bcs library](https://www.npmjs.com/package/@mysten/bcs) คุณสามารถติดตั้งด้วยการพิมพ์คำสั่งในไดเรกทอรี่นอกสุดของโปรเจค node::

```bash
npm i @mysten/bcs
```

### ตัวอย่างพื้นฐาน

ไปลองใช้ไลบารีจาวาสคริปเพื่อ serialize และ de-serialize ของประเภทข้อมูลอย่างง่ายกันก่อน:

```javascript
import { BCS, getSuiMoveConfig } from "@mysten/bcs";

// initialize the serializer with default Sui Move configurations
const bcs = new BCS(getSuiMoveConfig());

// Define some test data types
const integer = 10;
const array = [1, 2, 3, 4];
const string = "test string"

// use bcs.ser() to serialize data
const ser_integer = bcs.ser(BCS.U16, integer);
const ser_array = bcs.ser("vector<u8>", array);
const ser_string = bcs.ser(BCS.STRING, string);

// use bcs.de() to deserialize data
const de_integer = bcs.de(BCS.U16, ser_integer.toBytes());
const de_array = bcs.de("vector<u8>", ser_array.toBytes());
const de_string = bcs.de(BCS.STRING, ser_string.toBytes());

```

เราสามารถเริ่มต้นอินสแตนซ์ด้วยค่าเริ่มต้นพื้นฐานสำหรับ Sui Move โดยการใช้ syntax ด้านบน, `new BCS(getSuiMoveConfig())`.

ตัวไลบารีมี ENUMs มาให้ในตัวที่สามารถใช้สำหรับประเภทต่างๆของ Sui Move เช่น `BCS.U16`, `BCS.STRING`, และอื่นๆ สำหรับ [generic types](../../../unit-three/lessons/2_intro_to_generics.md), สามารถประกาศโดยการใช้ syntax เดียวกับ Sui Move เช่น `vector<u8>` ในตัวอย่างด้านบน

ลองมาดูแต่และฟิลด์ที่ถูก serialzied และ deserialized อย่างละเอียด:

```bash
# ints are little endian hexadecimals
0a00
10
# the first element of a vector indicates the total length,
# then it's just whatever elements are in the vector
0401020304
1,2,3,4
# strings are just vectors of u8's, with the first element equal to the length of the string
0b7465737420737472696e67
test string
```

### Type Registration

เราสามารถลงทะเบียนตัว custom types ที่เราจะทำงานด้วย ด้วย syntax ด้านล่างนี้::

```javascript
import { BCS, getSuiMoveConfig } from "@mysten/bcs";
const bcs = new BCS(getSuiMoveConfig());

// Register the Metadata Type
bcs.registerStructType("Metadata", {
  name: BCS.STRING,
});

// Same for the main object that we intend to read
bcs.registerStructType("BCSObject", {
  // BCS.ADDRESS is used for ID types as well as address types
  id: BCS.ADDRESS,
  owner: BCS.ADDRESS,
  meta: "Metadata",
});
```

## การใช้งาน `bcs` ในสมาร์ทคอนแทรคของ Sui

มาต่อกันที่ตัวอย่างข้างบนด้วย structs

### การประกาศ Struct

เราจะเริ่มด้วยการประกาศ struct ที่สอดคล้องกับสัญญาใน Sui Move

```move
public struct Metadata has copy, drop {
    name: ascii::String,
}

public struct BCSObject has copy, drop {
    id: ID,
    owner: address,
    meta: Metadata,
}
```

### Deserialization

ทีนี้ มาลองเขียนฟังก์ชั่นเพื่อ deserialize วัตถุในคอนแทรคของ Sui

```move
public fun object_from_bytes(bcs_bytes: vector<u8>): BCSObject {
    let mut bcs = bcs::new(bcs_bytes);

    // Use `peel_*` functions to peel values from the serialized bytes.
    // Order has to be the same as we used in serialization!
    let (address, owner, meta) = (
        bcs.peel_address(),
        bcs.peel_address(),
        bcs.peel_vec_u8(),
    );
    // Pack a BCSObject struct with the results of serialization
    BCSObject {
        id: address.to_id(),
        owner,
        meta: Metadata { name: meta.to_ascii_string() },
    }
}
```

เมธอด `peel_*` เมธอด peel_ ในเฟรมเวิร์ก Sui [`bcs` module](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/bcs.md) ใช้เพื่อ “ปอก/ลอก” แต่ละฟิลด์จาก BCS serialized bytes โปรจำไว้ว่าลำดับที่เราทำการลอกฟิลด์ต้องเหมือนกับลำดับของฟิลด์ตอนประกาศ struct แบบเป๊ะๆ

_แบบทดสอบ: ทำไมการเรียกฟังก์ชั่น `peel_address` ในสองครั้งแรกถึงได้ผลลัพธ์ไม่เหมือนกัน ทั้งที่ใช้วัตถุ `bcs` เดียวกัน?_

และอย่าลืมสังเกตวิธีที่เราแปลง types จาก `address` เป็น `id`, และจาก `vector<8>` เป็น `std::ascii::string` ด้วยฟังก์ชั่นช่วยเหลือต่างๆ

_แบบทดสอบ: จะเกิดอะไรขึ้นถ้า `BSCObject` มี `UID` แทนที่จะเป็น `ID`?_

## ตัวอย่างฉบับสมบูรณ์ของการ Ser/De

สามารถดูตัวอย่างโค้ดจาวาสคริป และ Sui Move ได้ในโฟลเดอร์[`example_projects`](../example_projects/)

อย่างแรก เราทำการ serialize วัตถุทดสอบด้วยโปรแกรมจาวาสคริป:

```javascript
// We construct a test object to serialize, note that we can specify the format of the output to hex
let _bytes = bcs
  .ser("BCSObject", {
    id: "0x0000000000000000000000000000000000000005",
    owner: "0x000000000000000000000000000000000000000a",
    meta: {name: "aaa"}
  })
  .toString("hex");
```

เราต้องการให้ผลลัพธ์ของตัวเขียน BCS อยู่ในรูปแบบเลขฐานสิบหก ซึ่งสามารถระบุได้ดังด้านบน

แปะ `0x` นำหน้าผลลัพธ์ และ export เป็นตัวแปร:

```bash
export OBJECT_HEXSTRING=0x0000000000000000000000000000000000000005000000000000000000000000000000000000000a03616161
```

ตอนนี้ เราสามารถรัน unit tests เพื่อเช็คความถูกต้อง:

```bash
sui move test
```

คุณควรจะเห็นสิ่งนี้บนคอนโซล:

```bash
BUILDING bcs_move
Running Move unit tests
[ PASS    ] 0x0::bcs_object::test_deserialization
Test result: OK. Total tests: 1; passed: 1; failed: 0
```
หรือเราสามารถเผยแพร่โมดูล (และ export PACKAGE_ID) และเรียกเมธอด `emit_object` โดยใช้ BCS serialized hexstring ด้านบน:

```bash
sui client call --function emit_object --module bcs_object --package $PACKAGE_ID --args $OBJECT_HEXSTRING
```

จากนั้นเราสามารถตรวจสอบได้ที่เเท็บ `Events` ภายในธุรกรรมบน Sui Explorer เพื่อตรวจสอบความถูกต้องของข้อมูล `BCSObject`:

![Event](../images/event.png)


