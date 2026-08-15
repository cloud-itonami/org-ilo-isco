# org-ilo-isco — ILO ISCO-08 の**出所境界**（今は catalog を持っていない）

この repo は **ILO（ilo.org）の ISCO-08 職業分類**という外部標準を指す origin 面の名前
（`org-ilo` + 主題 `isco`）を持つ。cloud-itonami の workforce まわりで
「ISCO-08 の**標準そのもの**はどこか」と訊かれたときの行き先として名乗られている。

**しかし 2026-08-15 現在、この repo に ISCO-08 の catalog は入っていない。**
名乗りと中身が一致していない。この README はその差を隠さずに書く。

## 最近接 repo との境界

| repo | 持っているもの |
|---|---|
| **`cloud-itonami/org-ilo-isco`**（ここ） | 出所（ILO ISCO-08）を指す名前。**catalog の実体は未搭載** |
| `cloud-itonami/isco` | workforce coordinator。**619 ノードの catalog 実体**が `data/isco-occupations.edn` に在る |
| `cloud-itonami/cloud-itonami-isco-*` | 職業ごとの operational blueprint（`-0110` `-1111` …） |

`cloud-itonami/isco` の README は「the source-standard catalog boundary is
`cloud-itonami/org-ilo-isco`」と宣言している。**その宣言の行き先がここ**だが、
実際の catalog は宣言した側（`isco`）に置かれたままである。どちらが正しい配置かは
まだ決めていない（[ADR-0001](docs/adr/0001-inherited-scaffold-not-a-catalog.md) §決めていないこと）。

## 今ここに在るもの（実測 2026-08-15、継承分は commit `e2463f7`）

**継承した中身は 15 ファイル・約 33 KB・commit 1 本きり**。`etzhayyim/root` の
`60-apps/etzhayyim-project-open-isco` を 2026-07-20 に切り出したもので、
切り出し以降の変更は無い（`migration.edn` が source revision `c3a74d2` を記録している）。
この README と `docs/` はその上に足した**文書だけ**の commit で、中身は直していない。

```
CLAUDE.md            PROJECT.jsonld       README.edn      MIGRATION-TODO.md
migration.edn        kotoba/              （TypeScript scaffold 5 ファイル）
```

- **`src/` は無い**（TS は `kotoba/src/`）。**`test/` も `data/` も無い。**
- **走るものが 1 つも無い。** `deps.edn` も無く、`kotoba/` は install できない（下記）。

## 既知の欠陥 —— 踏む前に読む

いずれも [operator quickstart](docs/operator-quickstart.md) で**そのまま再現できる**。
経緯と判断は [ADR-0001](docs/adr/0001-inherited-scaffold-not-a-catalog.md)。

1. **`kotoba/` は install できない。** git 依存 `@etzhayyim/sdk` が `prepare: tsc` を
   持つため、npm 11.16.0 / node 26.3.0 では `EALLOWSCRIPTS` で止まる
   （`--ignore-scripts` を付けても同じ。git 依存の準備工程が内部で弾かれる）。
   **したがって `kotoba/README.md` に書かれた `pnpm tsx src/seed.ts` 等は今日
   実行できない。**
2. **`MIGRATION-TODO.md` が開けている「substrate 違反」は誤検知である。**
   `kotoba/src/query.ts` の唯一の該当箇所は **docstring の中で「置き換えた前の書き方」を
   説明している行**（`kysely.selectFrom(...)`）で、実際の import は
   `@etzhayyim/sdk` / `node:fs/promises` / `./types.js` だけ。**コードではなく散文に
   反応した検査**なので、この TODO を満たそうとしてコードを書き換える必要は無い。
3. **`PROJECT.jsonld` の分類が ISCO-08 と一致していない。** sub-major group を **41 件**
   列挙しているが、ISCO-08 の sub-major は **43 件**。軍隊の 3 群
   （`01` Commissioned / `02` Non-commissioned / `03` Other Ranks）が、標準に無い
   `00` 1 件に潰れている。**出所境界を名乗る repo の分類データが出所と違う**ので、
   ここを引用する前に確認すること（`cloud-itonami-isco-0110` / `-0210` / `-0310` の
   blueprint 家族は標準どおり 3 群に分かれている）。
4. **`kotoba/README.md` のリンクは全部切れている**（`../appview/` /
   `../../../90-docs/adr/…` / `../../../20-actors/etzhayyim-sdk/`）。切り出し前の
   monorepo のパスを指したまま運ばれてきた。参照している `data/isco08.sample.json` も
   **存在しない**。`kotoba/CHARTER-RIDER.md` は**壊れた symlink**（`../../../CHARTER-RIDER.md`）。
5. **`kotoba/README.md` の「SDK はまだ not yet implemented を投げる」は古い。**
   pin されている SDK commit `12314a0c` では `read` / `write` / `verify` は実装されて
   おり、`NotImplementedError` は `charter-compliance-gate.ts` の sub-gate 群に在る。

## 名前についての未検証事項

`org-ilo` は「ilo.org のラベル逆順」と読めるが、**`manifest/origin-domains.edn` に
この repo の記録が無い**。workspace の規則（CLAUDE.md「記録が無い = UNVERIFIED であって
CONFORMANT ではない」）に従い、ここでは**名前からドメインを補完しない**。
記録の追加は未実施（follow-up）。

## 出所

- ILO ISCO-08 — International Standard Classification of Occupations (Resolution 2008)。
  **ilo.org は自動クライアントに 403 を返す**（2026-08-15 実測: `www.ilo.org` /
  `ilostat.ilo.org` とも 403）ので、`curl` で取得する手順は書けない。
  workspace 内の機械可読な写しは `cloud-itonami/isco` の `data/isco-occupations.edn`
  （10 major / 43 sub-major / 130 minor / 436 unit = 619、実測で重複 0・親欠け 0）。
