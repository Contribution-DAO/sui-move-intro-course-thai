# หลักสูตร Sui Move เบื้องต้น

หลักสูตรเบื้องต้นเกี่ยวกับภาษา Sui Move ดูแลโดย [the Sui Foundation](https://suifoundation.org/)

## เนื้อหา

- **บทที่หนึ่ง: การตั้งค่าเครื่องมือ และสวัสดีชาวโลก**
    - [การตั้งค่าเครื่องมือ](./unit-one/lessons/1_set_up_environment.md)
    - [โครงสร้างโปรเจคของ Sui](./unit-one/lessons/2_sui_project_structure.md)
    - [Types และ Abilities แบบปรับแต่งเอง](./unit-one/lessons/3_custom_types_and_abilities.md)
    - [ฟังก์ชั่น](./unit-one/lessons/4_functions.md)
    - [การ Deploy Contract](./unit-one/lessons/5_contract_deployment.md)
    - [แบบฝึกหัดบทที่หนึ่ง](./exercises/unit-one/unit-one-exercises.md)
    - [เฉลยแบบฝึกหัดบทที่หนึ่ง](./exercises/unit-one/unit-one-exercises-answer-key.md)
- **บทที่สอง: การทำงานกับ Sui Objects**
    - [แนะนำ](./unit-two/lessons/1_working_wiith_sui_objects.md)
    - [ความเป็นเจ้าของ](./unit-two/lessons/2_ownership.md)
    - [การส่งพารามิเตอร์ และการลบวัตถุ](./unit-two/lessons/3_parameter_passing_and_object_deletion.md)
    - [การหุ้มวัตถุ](./unit-two/lessons/4_object_wrapping.md)
    - [ตัวอย่างการหุ้มวัตถุ](./unit-two/lessons/5_object_wrapping_example.md)
    - [รูปแบบการออกแบบความสามารถ](./unit-two/lessons/6_capability_design_pattern.md)
    - [เหตุการณ์](./unit-two/lessons/7_events.md)
- **บทที่สาม: โทเคนแบบทดแทนได้**
    - [เฟรมเวิร์ก Sui](./unit-three/lessons/1_sui_framework.md)
    - [แนะนำ Generics](./unit-three/lessons/2_intro_to_generics.md)
    - [รูปแบบการออกแบบพยาน](./unit-three/lessons/3_witness_design_pattern.md)
    - [ทรัพยากร `Coin` และเมธอด  `create_currency`](./unit-three/lessons/4_the_coin_resource_and_create_currency.md)
    - [ตัวอย่างการจัดการเหรียญ](./unit-three/lessons/5_managed_coin.md)
    - [`Clock` และ Locked Coin](./unit-three/lessons/6_clock_and_locked_coin.md)
    - [การทำ Unit Test](./unit-three/lessons/6_unit_testing.md)
    - [[ส่วนเสริม] Closed Loop Token](./unit-three/lessons/8_closed_loop_token.md) 
- **บทที่สี่: มาร์เก็ตเพลส**
    - [คอลเลคชั่นที่เป็นเนื้อเดียวกัน](./unit-four/lessons/1_homogeneous_collections.md)
    - [ฟิลด์แบบยืดหยุ่น](./unit-four/lessons/2_dynamic_fields.md)
    - [คอลเลคชั่นที่แตกต่างกัน](./unit-four/lessons/3_heterogeneous_collections.md)
    - [สัญญามาร์เก็ตเพลส](./unit-four/lessons/4_marketplace_contract.md)
    - [การนำไปใช้ และการทดสอบ](./unit-four/lessons/5_deployment_and_testing.md)
    - [แบบฝึกหัดบทที่สี่](./exercises/unit-four/unit-four-exercises.md)
    - [เฉลยแบบฝึกหัดบทที่สี่](./exercises/unit-four/unit-four-exercises-answer-key.md)
- **Unit Five: Sui Kiosk**
    - [Programmable Transaction Block](./unit-five/lessons/1_programmable_transaction_block.md)
    - [รูปแบบการออกแบบ Hot Potato](./unit-five/lessons/2_hot_potato_pattern.md)
    - [แนวคิดพื้นฐานเรื่อง Sui Kiosk](./unit-five/lessons/3_kiosk_basics.md)
    - [การใช้งาน Sui Kiosk ขั้นพื้นฐาน](./unit-five/lessons/4_kiosk_basic_usage.md)
    - [นโยบายการโอน](./unit-five/lessons/5_transfer_policy.md)
- **หัวข้อขั้นสูง**
    - [การเข้ารหัส BCS](./advanced-topics/BCS_encoding/lessons/BCS_encoding.md)

## สิ่งที่ต้องทำ

- [x] เขียนเนื้อหา BCS สำหรับหัวข้อขั้นสูง
- [ ] สร้าง Docker image สำหรับหลากหลายแพลทฟอร์ม
- [ ] สร้างแบบทดสอบสำหรับแต่ละบท
- [ ] เพิ่มบทที่ห้าเกี่ยวกับมาตรฐานการแสดง NFT และการ implement SDK/frontend

## แหล่งข้อมูลทั่วไปสำหรับนักพัฒนา

- [เอกสารสำหรับนักพัฒนา Sui](https://docs.sui.io/build)
- [Sui GitHub](https://github.com/MystenLabs/sui)
- [เอกสารสำหรับเฟรมเวิร์ก Sui](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/docs)
- [Sui Typescript SDK (ทางการ)](https://github.com/MystenLabs/sui/tree/main/sdk/typescript)
- [Sui Typescript SDK (ชุมชน)](https://github.com/scallop-io/sui-kit)
- [Sui Rust SDK (ทางการ)](https://github.com/MystenLabs/sui/tree/main/crates/sui-sdk)
- [Sui Golang SDK (ชุมชน)](https://github.com/coming-chat/go-sui-sdk)
- [Sui Python SDK (ชุมชน)](https://github.com/FrankC01/pysui)
- [Sui Java SDK (ชุมชน)](https://github.com/GrapeBaBa/sui4j)
- [Sui Kotlin SDK (ชุมชน)](https://github.com/cosmostation/suikotlin)
- [Sui C# SDK (ชุมชน)](https://github.com/d-moos/SuiNet)
- [Sui Explorer](https://explorer.sui.io/)

## สังคมและชุมชน

ถ้าคุณต้องการที่จะเข้าร่วมชุมชนออนไลน์ของ Sui สามารถเข้าร่วมโดยลิ้งค์ต่อไปนี้:

- [เครือข่ายทวิตเตอร์](https://twitter.com/SuiNetwork) 
- [ดิสคอร์ดทางการ](https://discord.gg/sui)
- [ฟอรัมนักพัฒนา](https://forums.sui.io/)

## การแปล Repo

- [x] ภาษาอังกฤษ
- [x] [ภาษาจีน](https://github.com/RandyPen/sui-move-intro-course-zh)

โปรด [ติดต่อมา](mailto:henry@sui.io) ถ้าคุณต้องการช่วยแปลเป็นภาษาอื่นๆ

## วิดิโอ และรูปแบบอื่นๆ

- [x] Encode Club Video Series (ภาษาอังกฤษ)
- [x] [BuidlerDAO Video Series](https://www.bilibili.com/video/BV1RY411v7YU) (ภาษาจีน)
- [x] MoveBit Move บนคอร์ส Sui (ภาษาอังกฤษ) [เพลย์ลิสต์ YouTube](https://www.youtube.com/playlist?list=PL3id4Z64z2sNED_aH7UYIFFwy6MsvKCN9) | [บทเรียนและโค้ด](https://github.com/movebit/sui-course-2023)

## คำถามที่พบบ่อย

1. ฉันสามารถใช้เนื้อหาของ repo นี้เพื่อผลิตเนื้อหาด้านการศึกษาอื่นๆ ที่เกี่ยวข้องกับภาษาโปรแกรม Sui หรือ Sui Move ได้หรือไม่?

    ได้ นั่นคือความตั้งใจแรกเริ่มของ repo นี้ เพื่อให้ผู้สร้างเนื้อหา และแพลทฟอร์มการศึกษาใช้งาน และขยายเนื้อหาภายใน repo นี้ เพื่อสร้างรูปแบบต่างๆ ของสื่อ หรือเนื้อหาทางเทคนิคเกี่ยวกับภาษา Sui หรือ Sui Move

    repo ได้รับอนุญาตภายใต้ Creative Common License; [CC-BY-SA-4.0 license](https://github.com/sui-foundation/sui-move-intro-course/blob/main/LICENSE), เพื่อความชัดเจนมากขึ้น สิ่งนี้ทำให้ทุกคนแก้ไข เปลี่ยนแปลง สร้าง หรือแบ่งปันเนื้อหาใน repo นี้ ไม่ว่าด้วยวัตถุประสงค์ใดก็ตาม

2. ฉันจะมีส่วนร่วมใน repo นี้ได้อย่างไร?

    Fork และเปิด PR กลับเข้ามา พวกเราเปิดรับทุกๆการสนับสนุนจากชุมชน



