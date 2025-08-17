# รูปแบบ Hot Potato

Hot Potato คือ struct ที่ไม่มีความสามารถใดๆ ดังนั้นคุณจึงสามาถทำได้แค่ pack และ unpack มันภายในโมดูลของมันเท่านั้น รูปแบบนี้ใช้กลไกของ Programmable Transaction Block (PTB) เพื่อบังคับให้ผู้ใช้งานทำตามตรรกะทางธุรกิจที่ระบบกำหนดไว้ให้เสร็จสิ้นภายในธุรกรรมเดียว พูดง่าย ๆ ก็คือ หากคำสั่ง A ส่งคืนค่าที่เป็น hot potato ผู้ใช้จะต้อง "จัดการ" หรือ "ใช้" ค่านั้นให้เสร็จภายในคำสั่งถัดไปใน PTB เดียวกัน ไม่เช่นนั้นธุรกรรมจะล้มเหลว

## การกำหนดชนิดข้อมูล

```move
module flashloan::flashloan;

// === Imports ===
use sui::sui::SUI;
use sui::coin::{Self, Coin};
use sui::balance::{Self, Balance};
use sui::object::{UID};
use sui::tx_context::{TxContext};

/// เมื่อจำนวนที่ขอกู้เกินจากจำนวนใน pool
const ELoanAmountExceedPool: u64 = 0;
/// เมื่อจำนวนเงินที่ชำระคืนไม่ตรงกับที่กู้ไป
const ERepayAmountInvalid: u64 = 1;

/// พูลสินเชื่อแบบ "shared"
/// สำหรับการสาธิต สมมติว่า pool นี้รองรับเฉพาะ SUI
public struct LoanPool has key {
    id: UID,
    amount: Balance<SUI>,
}

/// ตำแหน่งเงินกู้
/// เป็น struct แบบ hot potato ที่บังคับให้ผู้ใช้
/// ต้องชำระหนี้ก่อนธุรกรรมจะสิ้นสุด หรือภายใน PTB เดียวกัน
public struct Loan {
    amount: u64,
}
```

เรามี `LoanPool` เป็น shared object ทำหน้าที่เสมือนคลังเงินให้ผู้ใช้สามารถกู้ได้ ซึ่ง pool นี้จะรองรับเฉพาะ SUI เพื่อความเรียบง่าย จากนั้นมี `Loan` ซึ่งเป็น struct แบบ hot potato ใช้เพื่อบังคับให้ผู้ใช้ต้องชำระหนี้ก่อนที่ธุรกรรมจะจบ
`Loan` มีเพียง field เดียวคือ `amount` ซึ่งแสดงจำนวนที่ถูกกู้ไป

We have a `LoanPool` shared object acting as a money vault ready for users to borrow. For simplicity sake, this pool only accepts SUI. Next, we have `Loan` which is a hot potato struct, we will use it to enforce users to repay the loan before transaction ends. `Loan` only has 1 field `amount` which is the borrowed amount.

## การยืม

```move
/// ฟังก์ชันให้ผู้ใช้กู้จาก loan pool
/// คืนค่าเป็น [`Coin<SUI>`] และ [`Loan`] position
/// ที่บังคับให้ผู้ใช้ต้องใช้ให้เสร็จก่อน PTB จะสิ้นสุด
public fun borrow(
    pool: &mut LoanPool,
    amount: u64,
    ctx: &mut TxContext,
): (Coin<SUI>, Loan) {
    assert!(amount <= pool.amount.value(), ELoanAmountExceedPool);
    (
        pool.amount.split(amount).into_coin(ctx),
        Loan {
            amount,
        },
    )
}
```
ผู้ใช้สามารถกู้ยืมเงินจาก `LoanPool` ได้โดยการเรียก `borrow()` โดยพื้นฐานแล้ว ฟังก์ชันจะคืนค่า `Coin<SUI>` ให้ผู้ใช้สามารถนำไปใช้กับฟังก์ชันอื่นต่อได้ พร้อมกันนั้นยังคืนค่า hot potato `Loan` กลับมาด้วย ดังที่กล่าวไว้ก่อนหน้า การ consume `Loan` ทำได้เพียงการ unpack ในฟังก์ชันที่อยู่ในโมดูลเดียวกัน ซึ่งทำให้เฉพาะแอปพลิเคชันเท่านั้นที่สามารถกำหนดวิธีการ consume hot potato ได้ ไม่ใช่บุคคลภายนอก

## การชำระคืน

```move
/// ชำระคืนเงินกู้
/// ผู้ใช้ต้องเรียกฟังก์ชันนี้เพื่อให้แน่ใจว่าเงินกู้ถูกชำระคืน
/// ก่อนธุรกรรมสิ้นสุด
public fun repay(pool: &mut LoanPool, loan: Loan, payment: Coin<SUI>) {
    let Loan { amount } = loan;
    assert!(payment.value() == amount, ERepayAmountInvalid);

    pool.amount.join(payment.into_balance());
}
```

ผู้ใช้จะต้องเรียก `repay()` เพื่อชำระคืนเงินกู้ก่อน PTB จะสิ้นสุด เรา consume `Loan` โดยการ unpack มิฉะนั้นคุณจะได้ error จากคอมไพเลอร์หากพยายามเข้าถึงฟิลด์ `loan.amount` โดยตรง เนื่องจาก `Loan` เป็น non-`drop` หลังจาก unpack แล้ว เราจะตรวจสอบจำนวนการชำระคืน และอัปเดตค่าของ `LoanPool` ให้ถูกต้อง

## ตัวอย่าง

ลองสร้างตัวอย่าง flashloan โดยเราจะยืม SUI จำนวนหนึ่ง นำไป mint NFT จำลอง และขายเพื่อนำเงินมาชำระหนี้ เราจะเรียนรู้การใช้ PTB ร่วมกับ Sui CLI เพื่อดำเนินการสิ่งนี้ทั้งหมดในธุรกรรมเดียว

```move
/// NFT จำลองเพื่อแสดงการทำงานของ flashloan
public struct NFT has key {
    id: UID,
    price: Balance<SUI>,
}

/// Mint NFT
public fun mint_nft(payment: Coin<SUI>, ctx: &mut TxContext): NFT {
    NFT {
        id: object::new(ctx),
        price: payment.into_balance(),
    }
}

/// ขาย NFT
public fun sell_nft(nft: NFT, ctx: &mut TxContext): Coin<SUI> {
    let NFT { id, price } = nft;
    id.delete();
    price.into_coin(ctx)
}
```

คุณควรจะสามารถ deploy smart contract ตามคู่มือก่อนหน้าได้ หลังการ deploy เราจะได้ package ID และ shared `LoanPool` object ให้เรา export ไว้เพื่อใช้ภายหลัง

```bash
export LOAN_PACKAGE_ID=<package id>
export LOAN_POOL_ID=<object id of the loan pool>
```

คุณต้องฝาก SUI จำนวนหนึ่งโดยใช้ฟังก์ชัน `flashloan::deposit_pool` ตัวอย่างนี้จะฝาก 10_000 MIST เข้าพูลเงินกู้ (loan pool)

```bash
sui client ptb \
--split-coins gas "[10000]" \
--assign coin \
--move-call $LOAN_PACKAGE_ID::flashloan::deposit_pool @$LOAN_POOL_ID coin.0 \

```

ตอนนี้เราจะสร้าง PTB ที่ทำงานตามลำดับ `borrow() -> mint_nft() -> sell_nft() -> repay()`.

```bash
sui client ptb \
--move-call $LOAN_PACKAGE_ID::flashloan::borrow @$LOAN_POOL_ID 10000 \
--assign borrow_res \
--move-call $LOAN_PACKAGE_ID::flashloan::mint_nft borrow_res.0 \
--assign nft \
--move-call $LOAN_PACKAGE_ID::flashloan::sell_nft nft \
--assign repay_coin \
--move-call $LOAN_PACKAGE_ID::flashloan::repay @$LOAN_POOL_ID borrow_res.1 repay_coin \

```

_แบบทดสอบ: จะเกิดอะไรขึ้นถ้าคุณไม่เรียก `repay()` ตอนท้ายของ PTB ลองทดสอบดูด้วยตัวเอง_

_💡หมายเหตุ: คุณอาจตรวจสอบรายละเอียดของ PTB เพิ่มเติมได้ที่ [SuiVision](https://testnet.suivision.xyz/) หรือ [SuiScan](https://suiscan.xyz/testnet/home)_
