# Sui Framework

การใช้งานโดยทั่วไปของสมาร์ทคอนแทรคคือการออกโทเคนแบบทดแทนได้ (fungible tokens) (เช่น ERC-20 บน Ethereum) เราจะมาดูวิธีการทำแบบนั้นบน Sui โดยใช้ Sui Framework และรู้จัก classic fungible tokens รูปแบบต่างๆ

## Sui Framework

[The Sui Framework](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/docs) คือการ implement ของ Move VM เพื่อมาใช้สำหรับ Sui โดยเฉพาะ ซึ่งประกอบไปด้วย native API ของ Sui รวมทั้งวิธีการใช้งานไลบารีมาตรฐานของ Move เช่นเดียวกันกับการดำเนินการใดๆ เฉพาะของ Sui อาทิ [crypto primitives](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/groth16.md) การใช้งาน [data structures](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/url.md) ในระดับ framework

วิธีการ implement custom fungible token ใน Sui จะได้ใช้งานไลบารีหลากหลายอย่างใน Sui Framework

## `sui::coin`

ไลบารีหลักที่เราจะใช้ในการสร้างโทเคนคือ [`sui::coin`](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/coin.md) module.

พวก resources หรือ methods ต่างๆที่เราจะได้ใช้ตรงๆในโทเคนของเราคือ:

- Resource: [Coin](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/coin.md#resource-coin)
- Resource: [TreasuryCap](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/coin.md#resource-treasurycap)
- Resource: [CoinMetadata](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/coin.md#resource-coinmetadata)
- Method: [coin::create_currency](https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/docs/sui/coin.md#0x2_coin_create_currency)

เราจะมาย้อนดูสิ่งเหล่านี้แบบเจาะลึกอีกครั้งหลังจากได้ทำความรู้จักคอนเซปใหม่ๆ ในบทถัดๆ ไป
