# ตัวอย่างการหุ้มวัตถุ

เราจะ implement ตัวอย่างวิธีการหุ้มวัตถุ object wrapping ใน transcript ของเรากัน โดยเราจะทำให้ `WrappableTranscript` ถูกหุ้มด้วย `Folder` และสามารถแกะดูผ่าน `Folder` ได้เท่านั้น ดังนั้นตัว transcript ข้างในจะสามารถเข้าถึงได้แค่คน หรือแอดเดรสที่เราตั้งใจให้เข้าถึง

## แก้ไข `WrappableTranscript` และ `Folder`

ก่อนอื่น เราต้องแก้โค้ด `WrappableTranscript` กับ `Folder` จากตอนที่แล้วกันก่อน

1. ทำการเพิ่ม ability `key` ให้ `WrappableTranscript` เพื่อให้มันกลายเป็น assets และสามารถถูกโอนไปมาได้

อย่าลืมว่าตัวแปรประเภทที่มี abilities `key` กับ `store` จะถือว่าเป็น assets ใน Sui Move

```move
public struct WrappableTranscript has key, store {
    id: UID,
    history: u8,
    math: u8,
    literature: u8,
}
```

2. เราจำเป็นต้องเพิ่มฟิลด์ `intended_address` ให้กับ `Folder` เพื่อใช้ระบุว่าแอดเดรสไหนสามารถเข้าถึง transcript ที่ห่อไว้ข้างในได้

```move
public struct Folder has key {
    id: UID,
    transcript: WrappableTranscript,
    intended_address: address
}
```

## Request Transcript Method

```move
public fun request_transcript(transcript: WrappableTranscript, intended_address: address, ctx: &mut TxContext){
    let folder_object = Folder {
        id: object::new(ctx),
        transcript,
        intended_address
    };
    //We transfer the wrapped transcript object directly to the intended address
    transfer::transfer(folder_object, intended_address)
}
```

เมธอดนี้ทำการรับ `WrappableTranscript` และห่อมันไว้ใน `Folder` จากนั้นทำการโอน transacript ที่ถูกห่อไว้ไปยัง intended_address ที่ระบุเข้ามา

## Unwrap Transcript Method

```move
public fun unpack_wrapped_transcript(folder: Folder, ctx: &mut TxContext) {
    // Check that the person unpacking the transcript is the intended viewer
    assert!(folder.intended_address == ctx.sender(), ENotIntendedAddress);
    let Folder { id, transcript, .. } = folder;
    transfer::transfer(transcript, ctx.sender());
    id.delete();
}
```

เมธอดนี้ทำการแกะ `WrappableTranscript` ออกมาจาก `Folder` โดยมีเงื่อนไขว่าคนที่เรียกฟังก์ชั่นต้องเป็นแอดเดรสเดียวกับ intended_address ที่ระบุให้ตอนที่มันถูกห่อเท่านั้น และจากนั้นก็ทำการส่งไปในคนที่เรียกทำธุรกรรม

*แบบทดสอบ: ทำไมเราต้องเขียนโค้ดลบวัตถุตัวแม่ด้วยตัวเองด้วย? จะเกิดอะไรขึ้นถ้าเราไม่ลบมัน?*

### Assert

เราใช้คำสั่ง `assert!` เพื่อตรวจสอบว่าแอดเดรสที่เรียกทำธุรกรรมในการแกะ transcript นั้นเป็นแอดเดรสเดียวกันกับค่า `intended_address` ที่อยู่ใน `Folder` หรือไม่

`assert!` รับพารามิเตอร์สองตัวในรูปแบบนี้:

```
assert!(<bool expression>, <code>)
```

โดยที่ boolean expression ต้องได้ค่า true ไม่เช่นนั้นมันจะหยุดทำงานและแสดง error code `<code>` ออกมา

### Custom Errors

โดย default เราจะใช้เลข 0 เป็น error code ข้างบน แต่เราสามารถกำหนดตัวแปร error code ที่เป็นค่าคงที่ได้ด้วยวิธีนี้:

```move
const ENotIntendedAddress: u64 = 1;
```

Error code นี้สามารถใช้ได้ในระดับแอปพลิเคชั่น และถือเป็นวิธีการจัดการที่ถูกต้องเหมาะสม

**นี่คือโค้ดเต็มๆเวอร์ชั่นที่สองของสิ่งที่เราทำกันมาจนถึงตอนนี้: [WIP transcript.move](../example_projects/transcript/sources/transcript_2.move_wip)**
