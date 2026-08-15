# got you (懂你)

**A LINE-based AI booking assistant for hair salons.**

Live in production at **[gotyouyu.com](https://gotyouyu.com)** — used daily by working stylists, handling real customer traffic.

> **Source is private.** This is a production system handling real customer bookings and salon POS data. This repository documents the architecture and the reasoning behind it.

---

## What it does

A customer messages the salon's existing LINE account, in plain language:

> 「這週五下午想燙個頭髮」
> *("Free Friday afternoon for a perm?")*

The system then:

1. Classifies the intent
2. Extracts date, time window, and service
3. Checks the stylist's real availability
4. Quotes price and duration
5. Confirms, and writes the booking straight into the salon's POS

No app to install. No booking page. No new habit for the customer, and none for the stylist.

---

## The architecture principle

### The AI only labels. The program decides.

The language model classifies intent and tags the conversation. **It never writes what the customer reads.** Every sentence the customer sees is selected by deterministic code from a fixed library.

Why:

- In this business, the wrong tone loses a client. I can't ship sentences I can't control.
- Generated sentences can't be tested. Selected sentences can.
- When something goes wrong, *"which branch chose this sentence"* is answerable. *"Why did the model say that"* is not.

A customer message passes through twelve stages. Stage 6 assigns the labels. Stages 8–11 act on them. That boundary is deliberate, and it is enforced structurally — not by convention.

---

## Three states, not two

Availability is `AVAILABLE` / `UNAVAILABLE` / **`UNKNOWN`**.

"Not scanned yet" is not the same as "scanned, and there are no slots." Collapsing those two into one boolean is always a defect — it is how a system confidently tells a customer there is no space when it simply has not looked.

---

## The failure shape

The recurring bug in this system is not *"computed the wrong answer."* It is:

> **The answer was computed correctly, but never handed to the person who needed it.**

Found in production:

- Conversations flagged *"AI disclosure given"* on turns where the disclosure was suppressed by a card render. The flag was set. The sentence was never sent.
- A card telling the customer *"08/20 works"* while its own button booked **08/21** — a date the card never displayed.
- A time parser producing a window that ended before it started, silently filtering out every candidate slot.

Finding these changed how the system is tested: **don't watch the exit, watch the origin.** One derivation, one owner, and structural guards so a second deriver can't quietly appear.

---

## Engineering practice

- **~3,500 automated tests**, green before and after every change
- **Mutation verification** — after a fix, revert the source and confirm the test actually goes red. A test that passes against the bug is worse than no test.
- **Cost sets** — every fix carries a set of "must not break" behaviours, and those must be green *before* the fix lands
- **Measure, don't guess** — every number quoted here came from querying the production database, not from estimation

---

## Stack

LINE Messaging API · Python · Oracle Cloud · isoPOS integration · multi-model (Claude / Codex / Qwen)

---

## About

Built and maintained by a **practicing hairstylist with no engineering background**, using AI coding tools as the engineering team.

The problem it solves is one I have personally: answering booking messages between haircuts, and losing the clients whose messages I missed.

---

## 中文簡介

**got you（懂你）** 是給美髮沙龍用的 LINE AI 預約助理，已在 [gotyouyu.com](https://gotyouyu.com) 正式上線。

客人用日常語言在 LINE 說「這週五下午想燙頭髮」，系統判斷意圖、報價、確認時間，直接寫進店家的 POS。不用裝 App、不用開預約頁、設計師和客人都不用改習慣。

核心架構是 **「AI 貼標籤、程式動手」**：語意層只負責分類，客人看到的每一句話都由確定性的程式從固定句庫挑選，不是 AI 生成的。因為在這一行，語氣錯了就掉客人 —— 我不能出貨我控制不了的句子。

原始碼未公開（正式站處理真實客人資料）。本頁說明架構與背後的取捨。
