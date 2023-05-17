# การส่งพารามิเตอร์ และการลบวัตถุ

## การส่งพารามิเตอร์ (โดย `value`, `ref` และ `mut ref`)

ถ้าคุณคุ้นเคยกับภาษา Rust คุณน่าจะคุ้นเคยกับระบบความเป็นเจ้าของของ Rust ด้วยเช่นกัน สิ่งนี้เป็นข้อได้เปรียบของภาษา Move เมื่อเทียบกับ Solidity คุณจะสามารถรู้ได้ทันทีว่าฟังก์ชั่นนั้นจะทำอะไรกับสินทรัพย์ (asset) ของเราเมื่อเราเรียกใช้งาน นี่คือตัวอย่างบางส่วน:

```rust
use sui::object::{Self};

// You are allowed to retrieve the score but cannot modify it
public fun view_score(transcriptObject: &TranscriptObject): u8{
    transcriptObject.literature
}

// You are allowed to view and edit the score but not allowed to delete it
public entry fun update_score(transcriptObject: &mut TranscriptObject, score: u8){
    transcriptObject.literature = score
}

// You are allowed to do anything with the score, including view, edit, delete the entire transcript itself.
public entry fun delete_transcript(transcriptObject: TranscriptObject){
    let TranscriptObject {id, history: _, math: _, literature: _ } = transcriptObject;
    object::delete(id);
}
```

## การลบวัตถุ และการแกะ Struct

ฟังก์ชั่น `delete_transcript` จากตัวอย่างข้างบนแสดงถึงวิธีการลบวัตถุใน Sui

1. ในการที่จะลบวัตถุได้ เราต้องแกะวัตถุเพื่อเอา ID ของมันออกมาก่อน การแกะวัตถุสามารถทำได้ภายในโมดูลที่ประกาศวัตถุนี้ขึ้นมาเท่านั้น เนื่องจากเป็นกฎการดำเนินการใดๆกับ struct ของภาษา Move:

- Struct สามารถถูกสร้าง (”บรรจุ”), ทำลาย (”แกะ”) ภายในโมดูลที่ประกาศมันขึ้นมาเท่านั้น
- ฟิลด์ต่างๆของ struct ก็สามารถเข้าถึงได้ภายในโมดูลที่ประกาศ struct ขึ้นมาเช่นกัน

ทำตามกฎเหล่านี้ถ้าคุณต้องการที่จะแก้ไข struct นอกโมดูลที่ประกาศมันขึ้นมา และคุณยังต้องมี public methods สำหรับการดำเนินการเหล่านั้นด้วย

2. หลังจากแกะ struct และได้ ID ออกมาแล้ว เราสามารถลบวัตถุนี้ได้อย่างง่ายดายโดยการเรียกฟังก์ชั่น `object::delete` และโยน ID มันเข้าไป

*💡หมายเหตุ: เครื่องหมายขีดล่าง `_` ในตัวอย่างข้างบนใช้แทนตัวแปร หรือพารามิเตอร์ที่เราไม่ได้ใช้*

**นี่คือโค้ดเต็มๆ เวอร์ชั่นแรกของสิ่งที่เรากำลังทำกันมาจนถึงตอนนี้: [WIP transcript.move](../example_projects/transcript/sources/transcript_1.move_wip)**
