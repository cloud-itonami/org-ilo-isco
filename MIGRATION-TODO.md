# Migration TODO (post-verification gap patch)

**Status**: ⚠️ **監査済み 2026-08-15 — 下記の「違反」は誤検知だった。**
[ADR-0001](docs/adr/0001-inherited-scaffold-not-a-catalog.md) /
[operator quickstart](docs/operator-quickstart.md) step 4 で再現できる。

This app was originally classified as ALIGN (or already migrated as TRANSFORM)
but post-migration verification scan detected substrate-boundary violations
that were not previously flagged.

## Detected violations (per re-scan 2026-05-21):

```
  - 60-apps/etzhayyim-project-open-isco/kotoba/src/query.ts
```

**監査結果（2026-08-15）**: `kotoba/src/query.ts` で禁止トークンに当たるのは
**1 行だけ**で、それは import ではなく **docstring の中で「置き換える前の書き方」を
説明している行**である（`kysely.selectFrom('vertex_open_isco_occupation')…`）。
実際の import は `@etzhayyim/sdk` / `node:fs/promises` / `./types.js` のみ。
**検査はコードではなく散文に反応している。**

## Required remediation (per CLAUDE.md substrate boundary):

**以下 4 項目は、この repo に対しては該当箇所が存在しない**（2026-08-15 実測。
`kotoba/src/*.ts` の全 import を確認済み）。**コードを書き換えて閉じてはならない** ——
docstring を消せば検査は黙るが、直る物は何も無く、この形にした理由の説明だけが失われる。

- [x] ~~Replace `@atproto/api` / `viem` / IPFS / Signal direct imports with `@etzhayyim/sdk`.~~
      → 該当なし。外部 import は `@etzhayyim/sdk` のみ。
- [x] ~~Strip RisingWave / Postgres / Kysely / Drizzle / Prisma → AT MST + IPFS + Base L2.~~
      → 該当なし（唯一の一致は上記 docstring）。
- [x] ~~Strip Stripe / PayPal / fiat → USDC + ERC-4337 + `etzhayyim-tithe-router`.~~
      → 該当なし。
- [x] ~~Remove GA4 / Meta Pixel / 3rd-party ad-tracking.~~ → 該当なし。
- [ ] Audit against Charter Rider v2.0 §2(a)-(h). — **未実施**（`CHARTER-RIDER.md` は
      切り出しで壊れた symlink になっており、この repo から本文を読めない）。

**本当に開いている問い**は substrate 違反ではなく、**`kotoba/` を維持するのか**である
（install 経路が npm 側の都合で構造的に壊れている。quickstart step 3）。判断は ADR-0001。

## Reference

- ADR-2605192100 / 2605192115 / 2605192200
- `/CLAUDE.md` § Substrate boundary
- This file added by Coverage Gap Patch task #15.
