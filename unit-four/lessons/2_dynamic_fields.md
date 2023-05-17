# Dynamic Fields

เพื่อจะดูว่า collection ประเภท `Table` นั้นถูกนำมาใช้งานจริงอย่างไร เราต้องแนะนำให้รู้จักคอนเซปของ dynamic fields ใน Sui Move นั้น Dynamic fields คือฟิลด์แบบ heterogeneous ที่สามารถเพิ่ม หรือลบ ได้ในขณะรันไทม์ และสามารถตั้งชื่อได้ตามอำเภอใจ

dynamic fields สามารถแบ่งได้เป็นอีกสองประเภทย่อย:

  - **Dynamic Fields**  สามารถเก็บค่าอะไรก็ได้ที่มี ability `store` อย่างไรก็ตาม object ที่เก็บไว้ในฟิลด์ประเภทนี้จะถูกห่อหุ้มไว้ และไม่สามารถเข้าถึงได้โดยตรงโดยใช้ ID ของมันจากเครื่องมือภายนอก (explorers, wallets, และอื่นๆ)
  - **Dynamic Object Fields** values *ต้องเป็น* Sui objects (มี abilities  `key` และ `store` และมี `id: UID` เป็นฟิลด์แรก) แต่ยังคงสามารถเข้าถึงได้โดยตรงด้วย ID ของมัน

## Dynamic Field Operations

### การเพิ่ม Dynamic Field

เพื่อแสดงวิธีการทำงานกับ dynamic fields เราสามารถเขียน struct ได้ดังนี้:

```rust
   // Parent struct
    struct Parent has key {
        id: UID,
    }

    // Dynamic field child struct type containing a counter
    struct DFChild has store {
        count: u64
    }

    // Dynamic object field child struct type containing a counter
    struct DOFChild has key, store {
        id: UID,
        count: u64,
    }
```

นี่คือ API ที่ใช้สำหรับการเพิ่ม **dynamic fields** หรือ **dynamic object field** ให้กับ object::

```rust
  module collection::dynamic_fields {

      use sui::dynamic_object_field as ofield;
      use sui::dynamic_field as field;

    // Adds a DFChild to the parent object under the provided name
    public fun add_dfchild(parent: &mut Parent, child: DFChild, name: vector<u8>) {
        field::add(&mut parent.id, name, child);
    }

    // Adds a DOFChild to the parent object under the provided name
    public entry fun add_dofchild(parent: &mut Parent, child: DOFChild, name: vector<u8>) {
        ofield::add(&mut parent.id, name, child);
    }
  }
```

### การเข้าถึง และการแปลงรูป Dynamic Field

Dynamic fields และ dynamic object fields สามารถอ่าน หรือเข้าถึงได้ ดังนี้:

```rust
    // Borrows a reference to a DOFChild
    public fun borrow_dofchild(child: &DOFChild): &DOFChild {
        child
    }

    // Borrows a reference to a DFChild via its parent object
    public fun borrow_dfchild_via_parent(parent: &Parent, child_name: vector<u8>): &DFChild {
        field::borrow<vector<u8>, DFChild>(&parent.id, child_name)
    }

    // Borrows a reference to a DOFChild via its parent object
    public fun borrow_dofchild_via_parent(parent: &Parent, child_name: vector<u8>): &DOFChild {
        ofield::borrow<vector<u8>, DOFChild>(&parent.id, child_name)
    }
```

Dynamic fields และ dynamic object fields สามารถถูกแปลงรูปได้ ดังนี้:

```rust
    // Mutate a DOFChild directly
    public entry fun mutate_dofchild(child: &mut DOFChild) {
        child.count = child.count + 1;
    }

    // Mutate a DFChild directly
    public fun mutate_dfchild(child: &mut DFChild) {
        child.count = child.count + 1;
    }

    // Mutate a DFChild's counter via its parent object
    public entry fun mutate_dfchild_via_parent(parent: &mut Parent, child_name: vector<u8>) {
        let child = field::borrow_mut<vector<u8>, DFChild>(&mut parent.id, child_name);
        child.count = child.count + 1;
    }

    // Mutate a DOFChild's counter via its parent object
    public entry fun mutate_dofchild_via_parent(parent: &mut Parent, child_name: vector<u8>) {
        mutate_dofchild(ofield::borrow_mut<vector<u8>, DOFChild>(
            &mut parent.id,
            child_name,
        ));
    }
```
*แบบทดสอบ: ทำไม `mutate_dofchild` ถึงเป็นฟังก์ชั่นแรก โดยที่ไม่ใช่ `mutate_dfchild`?

### การลบ Dynamic Field

เราสามารถลบ dynamic field จาก object แม่ของมันได้ ดังนี้:

```rust
    // Removes a DFChild given its name and parent object's mutable reference, and returns it by value
    public fun remove_dfchild(parent: &mut Parent, child_name: vector<u8>): DFChild {
        field::remove<vector<u8>, DFChild>(&mut parent.id, child_name)
    }

    // Deletes a DOFChild given its name and parent object's mutable reference
    public entry fun delete_dofchild(parent: &mut Parent, child_name: vector<u8>) {
        let DOFChild { id, count: _ } = ofield::remove<vector<u8>, DOFChild>(
            &mut parent.id,
            child_name,
        );
        object::delete(id);
    }

    // Removes a DOFChild from the parent object and transfer it to the caller
    public entry fun reclaim_dofchild(parent: &mut Parent, child_name: vector<u8>, ctx: &mut TxContext) {
        let child = ofield::remove<vector<u8>, DOFChild>(
            &mut parent.id,
            child_name,
        );
        transfer::transfer(child, tx_context::sender(ctx));
    }
```

โปรดทราบว่าในกรณีของ dynamic object field นั้น เราสามารถลบ หรือโอนมันได้หลังจากถอดมันออกมาจาก object อื่น เนื่องจาก dynamic object field เป็น Sui object แต่เราไม่สามารถทำแบบนี้ได้กับ dynamic field เนื่องจากมันไม่มี ability `key` และไม่ใช่ Sui object

## Dynamic Field vs. Dynamic Object Field

เมื่อไหร่ที่คุณควรใช้ dynamic field เทียบกับ dynamic object field? พูดแบบง่ายๆคือ เราต้องการใช้ dynamic object fields เมื่อ type ลูกในโจทย์มี ability `key` และเราจะใช้ dynamic fields ในทางตรงกันข้าม

สำหรับคำอธิบายแบบเต็มๆ โปรดดูที่ [บทความนี้](https://forums.sui.io/t/dynamicfield-vs-dynamicobjectfield-why-do-we-have-both/2095) โดยคุณ @sblackshear.

## ทบทวนเรื่อง `Table`

ตอนนี้เราเข้าใจแล้วว่า dynamic fields ทำงานอย่างไร เราเปรียบได้ว่า `Table` เป็นเหมือนตัวที่ครอบการดำเนินการต่างๆของ dynamic fields เอาไว้ได้

คุณสามารถดู [ซอร์สโค้ด](https://github.com/MystenLabs/sui/blob/eb866def280bb050838d803f8f72e67e05bf1616/crates/sui-framework/packages/sui-framework/sources/table.move) ของ `Table` เป็นแบบฝึกหัดได้ และดูแต่ละ operations ก่อนหน้านี้เทียบกับ dynamic field operations และเพิ่มลอจิกบางอย่างเพื่อดูขนาดของ `Table`
