# ADR-0001 — この repo は出所境界を名乗るが、中身は継承した scaffold である

- **status**: accepted
- **date**: 2026-08-15
- **参照**: superproject ADR-2608052000（成熟度の測り方）/ ADR-2608080000（1 反復 1 軸）
- **先例**: `cloud-itonami/marine-insurance` ADR-0001（同型の「名乗りと中身のずれ」）

## 文脈

`org-ilo-isco` は 2026-07-20 に `etzhayyim/root` の
`60-apps/etzhayyim-project-open-isco` を切り出したもので（`migration.edn` の source
revision `c3a74d2`）、**切り出し以降 commit は 1 本も足されていない**。

名前は origin 面（`org-ilo` = 出所 + `isco` = 主題）を主張し、隣の
`cloud-itonami/isco` の README は「the source-standard catalog boundary is
`cloud-itonami/org-ilo-isco`」と**この repo を名指しで宣言している**。

しかし **README が無かった。** 宣言に従って辿り着いた読み手が最初に見るのは、
retired された etzhayyim appview のことを書いた `AGENTS.md` と、動かない
TypeScript scaffold である。**名乗りを確かめる入口が無い**状態だった。

## 測ってわかったこと（2026-08-15、commit `e2463f7`）

1. **catalog が無い。** 出所境界を名乗るのに ISCO-08 の分類データを持っていない。
   実体（619 ノード = 10/43/130/436、重複 0・親欠け 0 を実測）は
   **宣言した側の `cloud-itonami/isco/data/isco-occupations.edn` に在る。**
2. **`kotoba/` は install できない。** git 依存 `@etzhayyim/sdk` が `prepare: tsc` を
   持ち、npm 11.16.0 / node 26.3.0 では `EALLOWSCRIPTS` で止まる（`--ignore-scripts`
   でも同じ）。**pin された SDK commit `12314a0c` 自体は生きている** ——
   壊れているのは install 経路であって参照先ではない。
3. **`MIGRATION-TODO.md` の「substrate 違反」は誤検知だった。** 開かれている違反は
   `kotoba/src/query.ts` 1 件だが、該当するのは**「置き換えた前の書き方」を説明する
   docstring の行**（`kysely.selectFrom(...)`）で、実 import は `@etzhayyim/sdk` /
   `node:fs/promises` / `./types.js` だけ。**検査がコードではなく散文に反応している。**
4. **`PROJECT.jsonld` が ISCO-08 と一致しない。** sub-major group を 41 件列挙するが
   標準は 43 件。軍隊の `01`/`02`/`03` が標準に無い `00` 1 件へ潰れている。
   `etzhayyim:totalComponents` も 41 なので **内部では整合しており、外部の標準と
   照らして初めてずれる**（内部整合だけを見る検査では捕まらない）。
   なお `cloud-itonami-isco-0110` / `-0210` / `-0310` の blueprint 家族は標準どおり。
5. **継承された参照がほぼ全部切れている。** `kotoba/README.md` のリンク 3 種
   （`../appview/` / `../../../90-docs/adr/…` / `../../../20-actors/etzhayyim-sdk/`）、
   参照される `data/isco08.sample.json`、`kotoba/CHARTER-RIDER.md`（symlink）——
   **すべて解決しない。** 切り出しがパスを運ばなかった。
6. **`kotoba/README.md` の「SDK は not yet implemented を投げる」は古い。** pin 先では
   `read`/`write`/`verify` は実装済みで、`NotImplementedError` は
   `charter-compliance-gate.ts` の sub-gate 群に在る。

## 決めたこと

1. **README と operator quickstart を置く。** 名乗りと中身がずれているなら、
   **ずれを書くのが入口の仕事**である。隠して「ISCO-08 の出所境界」とだけ書くと、
   catalog を探しに来た読み手が 33 KB の死んだ scaffold を掘ることになる。
2. **`MIGRATION-TODO.md` の違反を「コードを直して」閉じない。** 誤検知なので、
   直す対象が無い。docstring を消せば検査は黙るが、**それは検査を黙らせるだけで
   何も直さない**（消せば「なぜこの形なのか」の説明が失われる分だけ悪化する）。
   TODO には監査済みである旨と本 ADR への参照を書き、**開いたまま残す** ——
   閉じるべきは「違反」ではなく、`kotoba/` を維持するのかという上位の問い。
3. **`PROJECT.jsonld` はこの反復では直さない。** 41→43 の修正は文書ではなくデータの
   変更で、`etzhayyim:` 名前空間の consumer を確かめずに触ると、**内部整合していた
   ものを片側だけ動かす**ことになる。欠陥として README / quickstart / ここに記録し、
   再現手順を添えるところまでを本反復の範囲とする。
4. **`kotoba/` を撤去しない。** 動かないが、`types.ts` は lexicon
   `com.etzhayyim.apps.openIsco.occupation` の record 形を、`query.ts` は MST 前置詞
   走査の設計意図を保持している。**撤去は catalog をどこに置くかを決めてから**。

## 決めていないこと（次の反復が判断する）

- **catalog の実体をここへ移すのか、`cloud-itonami/isco` に置いたままにして
  この repo の名乗りを畳むのか。** どちらでもよいが、**両方が「境界はあちら」と
  言っている今の状態だけは不可**。移すなら `isco` の README の宣言文が正になり、
  畳むなら宣言文を書き換える必要がある。
- **`kotoba/`（TypeScript）を維持するのか。** workspace の規則は新規実装を
  `.cljc` / `.kotoba` に寄せており（AGENTS.md「ランタイム優先順位」）、TS scaffold の
  復活はその方向と逆を向く。install 経路が npm 側の都合で壊れていることは、
  維持コストの実測値として数えてよい。
- **`manifest/origin-domains.edn` への登録。** この repo の記録が無く、規則により
  **UNVERIFIED**（名前からドメインを補完してはならない）。superproject 側の変更に
  なるため本反復では触っていない。

## 影響

- 出所境界を探す読み手が、**1 分で正しい行き先（`cloud-itonami/isco` の catalog）に
  着く**。
- `kotoba/` を動かそうとする者が、**install が構造的に不能であることを最初に知る**。
- 誤検知の TODO を満たすためにコードを書き換える作業が**起きなくなる**。
