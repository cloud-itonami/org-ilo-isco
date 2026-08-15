# operator quickstart

この repo は **出所（ILO ISCO-08）を指す名前**であって実行系ではない。**走らせる物は
1 つも無い**（[README](../README.md)）。だから operator の仕事は「起動する」ことでは
なく、**名乗っていることが今も本当かを確かめる**ことである。

所要 2〜3 分（step 3 の npm を含めて 5 分）。必要なもの: `git`、`nbb`
（ClojureScript on Node）、`npm`。**この repo に install する物は無い** —— `deps.edn` も
無く、`kotoba/package.json` は step 3 のとおり install できない。

step 3・4・5 は**壊れていることの確認**である。緑を探しにいく手順ではない ——
「まだ壊れている」を確かめ、直ったときに気づけるようにするためのもの。

## 1. 取得する

```bash
git clone git@github.com:cloud-itonami/org-ilo-isco.git
cd org-ilo-isco
```

west workspace の中なら `west update --fetch smart org-ilo-isco`
（`orgs/cloud-itonami/org-ilo-isco` に展開される）。その場合 **remote 名は `origin`
ではなく `cloud-itonami`**（west が付ける名前）なので `git fetch origin` は通らない。

## 2. 何が在るかを数える

```bash
git ls-files | wc -l                    # → 18（うち 15 が継承した中身、3 が本 docs）
git log --oneline | wc -l               # → 2
git ls-files | grep -v '^docs/' | grep -v '^README.md$' | wc -l   # → 15（継承分だけ）
ls src test data 2>&1 | tail -1
```

`src` / `test` / `data` は**どれも無い**（`ls` が `No such file or directory`）。
TypeScript は `kotoba/src/` に在り、`src/` ではない —— この差は成熟度計測が
`src/**` を見ることと関係する（superproject ADR-2608052000）。

**継承分 15 ファイルは 2026-07-20 に `etzhayyim/root` から切り出されたまま、
1 バイトも変わっていない**（変わったのは `MIGRATION-TODO.md` の監査結果追記だけ）。
2 本目の commit はこの文書群であって、中身を直したものではない。

## 3. `kotoba/` が install できないことを確かめる

```bash
cd kotoba && npm install --no-audit --no-fund ; echo "exit=$?"
```

こう落ちる（**これが期待される結果**）:

```
npm error code EALLOWSCRIPTS
npm error --allow-scripts is not allowed in project-scoped installs.
npm error git dep preparation failed
exit=1
```

理由: 依存 `@etzhayyim/sdk` が git 依存で、その `package.json` が `prepare: tsc` を
持つ（`dist/` は commit されていないので install 時に build する必要がある）。npm の
新しい版は project-scoped install の中でこの準備工程を許さない。

- **`--ignore-scripts` を付けても同じ**（内部の準備工程が弾かれるので回避にならない）。
- 実測環境: **node v26.3.0 / npm 11.16.0**（2026-08-15）。**この値の隣に自分の
  `node -v` / `npm -v` を並べて読むこと** —— npm の版が違えば結果が変わりうる
  （このワークスペースでは「ローカルで赤」が「他のノードで赤」を意味しないことが
  実測されている）。
- **pin されている SDK commit 自体は生きている**。消えたわけではない:
  ```bash
  git ls-remote https://github.com/etzhayyim/com-etzhayyim-sdk.git | head -1
  ```
  （`etzhayyim/com-etzhayyim-sdk` は GitHub の redirect で `kotoba-lang/sdk` に解決する。）

したがって `kotoba/README.md` の `pnpm tsx src/seed.ts` / `src/query.ts` /
`src/verify.ts` は**今日実行できない**。時間を溶かす前にここで止まること。

## 4. `MIGRATION-TODO.md` の「substrate 違反」が誤検知であることを確かめる

`MIGRATION-TODO.md` は `kotoba/src/query.ts` を違反として開いたままにしている。
実際に何件当たるかを見る:

```bash
grep -rn "kysely\|drizzle\|prisma\|risingwave\|postgres\|stripe\|paypal\|viem" kotoba/src/*.ts
```

**当たるのは 1 行だけ**で、それは import ではなく docstring である:

```
kotoba/src/query.ts:6: * (kysely.selectFrom('vertex_open_isco_occupation').where(...))
```

実際の import を並べると、外部依存は `@etzhayyim/sdk` だけだと分かる:

```bash
grep -h "^import" kotoba/src/*.ts | sort -u
```

```
import type { Occupation } from "./types.js";
import { Etzhayyim } from "@etzhayyim/sdk";
import { readFile } from "node:fs/promises";
```

**3 行で全部**。外部依存は `@etzhayyim/sdk` だけである。

**つまり検査はコードではなく散文に反応している。** 「置き換えた前の書き方」を説明する
コメントを消せば TODO は閉じるが、それは検査を黙らせるだけで何も直さない。判断は
[ADR-0001](adr/0001-inherited-scaffold-not-a-catalog.md)。

## 5. `PROJECT.jsonld` の分類が ISCO-08 とずれていることを確かめる

**前提なし**（この repo だけで実行できる）。ISCO-08 の sub-major group は **43 件**、
うち軍隊は `01` / `02` / `03` の 3 群である:

```bash
nbb -e '(ns p (:require ["fs" :as fs]))
(let [g (aget (js/JSON.parse (fs/readFileSync "PROJECT.jsonld" "utf8")) "etzhayyim:subMajorGroups")
      codes (set (map #(aget % "code") g))]
  (println "listed:" (count g))
  (doseq [c ["01" "02" "03" "00"]] (println " " c (if (codes c) "present" "ABSENT"))))'
```

```
listed: 41
  01 ABSENT
  02 ABSENT
  03 ABSENT
  00 present
```

**41 = 43 − 3 + 1。** 標準の軍隊 3 群が、標準に無い `00` 1 件へ潰れている。
`etzhayyim:totalComponents` も 41 と書いてあるので、**内部では整合している** ——
外部の標準と照らして初めてずれが出る。内部整合だけを見る検査では捕まらない類である。

**catalog と突き合わせる**（前提: `cloud-itonami/isco` が手元に在ること。west workspace
なら `orgs/cloud-itonami/isco`）:

```bash
nbb -e '(ns p (:require [clojure.edn :as edn] ["fs" :as fs]))
(let [seed (:seed (edn/read-string (fs/readFileSync "../isco/data/isco-occupations.edn" "utf8")))
      codes (map :isco.occupation/code seed)]
  (println "total:" (count seed))
  (println "by code length:" (pr-str (into (sorted-map) (frequencies (map count codes)))))
  (println "duplicates:" (- (count codes) (count (set codes))))
  (println "orphans:" (count (remove #(or (= 1 (count %)) ((set codes) (subs % 0 (dec (count %))))) codes))))'
```

```
total: 619
by code length: {1 10, 2 43, 3 130, 4 436}
duplicates: 0
orphans: 0
```

これが ISCO-08 の 10 / 43 / 130 / 436 = 619 で、**この repo が名乗っている出所境界の
実体**である。今それは隣の repo に在る（[README](../README.md) の境界表）。

**ILO 本体から取る手順は書けない** —— `www.ilo.org` も `ilostat.ilo.org` も自動
クライアントに **403** を返す（2026-08-15 実測）。取得できない URL を手順に書かない。

## 直ったかどうかの見分け方

| step | 今日 | 直ったら |
|---|---|---|
| 3 | `EALLOWSCRIPTS` で install 不能 | install が通る、または `kotoba/` が撤去されている |
| 4 | 誤検知の TODO が開いたまま | `MIGRATION-TODO.md` が閉じている |
| 5 | `listed: 41` | `listed: 43` かつ `01`/`02`/`03` present |

**step 2 の継承分カウントが 15 のままなら、中身については何も起きていない**
（commit 数は文書を足しただけでも増えるので、そちらを進捗と読まないこと）。
