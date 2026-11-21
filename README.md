# lean-rh-alpoge

An independent build of the Lean 4 project distributed at <https://alpo.ge>, with a build report and
a statistical analysis of its file structure and proof network.

The project is a Lake package (175,187 `.lean` files, about 17.5 million lines) on top of Mathlib.
Its `Main.lean` states five theorems, the first of which is

```lean
theorem riemannHypothesis : RiemannHypothesis := rhOfAll
```

where `RiemannHypothesis` is the definition in Mathlib. This repository records what happens when
the project is built and checked; it makes no claim beyond what the reports show.

## Getting the data

The sources are distributed as 14 parts of one gzipped tarball (239,275,554 bytes in total):

```bash
for i in {01..14}; do 
  curl -fL --retry 3 -O "https://alpo.ge/lean.tar.gz.part${i}" || break
done
```

Concatenate the parts and extract them; this creates the directory `lean/`:

```bash
cat lean.tar.gz.part* | shasum -a 256   # 9631b4f020199e580ba86178cb56c6e9b09f690c2369a80dc4334aa93f49c7fc
cat lean.tar.gz.part* | tar -xzf -
```

The checksum is that of the parts downloaded on 2026-11-21.

## The Lean project

| Item | Value |
|---|---|
| Lean toolchain | `leanprover/lean4:v4.33.1` (`lean/lean-toolchain`) |
| Mathlib | revision `0df444a360eaa60ab8c11dca51a86af692955474` (`lean/lakefile.toml`) |
| Lake package | `tree`, source directory `lean/src`, 11 library targets |
| Size | 175,187 modules, 1.03 GB of source, 17,549,402 lines |
| Imports | 753,569 imports between project modules; the import graph is acyclic with depth 202; every module is imported, directly or indirectly, by `Main` |

`lean/src/Statements.lean` defines the four statements about Dirichlet L-functions and the zeta
function that `Main.lean` proves next to `RiemannHypothesis`:
`RiemannZetaZerosOnLine`, `DirichletLNonvanishingRightOfHalf`, `DirichletLZerosOnLine`,
`PrimitiveDirichletLZerosOnLine`.

To build it (Mathlib is fetched as a prebuilt cache, not compiled):

```bash
cd lean
lake exe cache get
lake build
```

## Reports

The build is run on the supercomputer RIKYU (RIKEN R-CCS; aarch64, 144 cores per node).
Reports are added here as each step finishes:

1. Build with `lake exe cache get` and `lake build` — **finished, see below**.
2. Second build under [Comparator](https://github.com/leanprover/comparator), which rebuilds the
   solution in a sandbox, compares the theorem statements with a challenge file, checks the axioms
   used, and replays the proofs in the Lean kernel — **running, no verdict yet** (see below).
3. HTML report (English / Japanese): summary of the build logs, the file structure of the project,
   and statistics of the network of proof objects — **<https://shosonoda.github.io/lean-rh-alpoge/>**
   (first edition; it will be updated when step 2 ends).

### Step 1: `lake exe cache get` and `lake build` (2026-11-21)

`lake build` exited with code 0 and all 175,187 modules of the project were compiled.

| Item | Result |
|---|---|
| Command | `lake exe cache get`, then `lake build -v` (default targets; Mathlib from the prebuilt cache, 8,690 files) |
| Exit code of `lake build` | 0 |
| Modules compiled | 175,187 of 175,187 (one `.olean` each, 88.1 GiB in total) |
| Failed Lake jobs | 0 |
| `error` lines in the log | 0 |
| `declaration uses 'sorry'` warnings | 0 |
| Warning lines in the log | 460,321 in the last run (a breakdown will be in the HTML report) |
| Machine | one RIKYU node (NVIDIA GB200 NVL4, Arm Neoverse-V2, Ubuntu 24.04), 64 cores and 800 GB allocated, 64 Lake threads |
| Time | 6 h 21 min of `lake build` in the last run, which compiled 170,014 modules; the first 5,173 modules were compiled by two short jobs (18 min and 11 min in total, on 32 cores) whose output was carried over |
| Cost | 13.3 GPU-hours of allocation in the three runs (about 4,000 yen at 300 yen per GPU-hour; the GPUs themselves were not used) |

The build was done in three Slurm jobs. Each job saved `.lake/build` when it stopped and the next
one continued from it, so Lake replayed the modules that were already built instead of compiling
them again.

This step shows that Lean 4.33.1 elaborates every file of the project without an error and
without a `sorry` warning. It does not by itself check which axioms the five theorems depend on,
nor that their statements are the intended ones; that is the purpose of step 2.

### Step 2: Comparator (running)

Comparator (tag v4.33.0, built with Lean 4.33.1) is given a challenge file that states the five
theorems of `Main.lean` with `sorry`, the solution module `Main`, and the permitted axioms
`propext`, `Quot.sound`, `Classical.choice`. It rebuilds the project inside a sandbox before it
exports and checks the proofs.

- The sandboxed rebuild took more than one batch job. The first job (8 hours) stopped with
  162,328 of 175,187 modules built; the second job continued from the saved build and Lake
  reported `Build completed successfully (183893 jobs)` at 11:01 on 2026-11-21.
- After the rebuild, Comparator exports the proofs (61.6 GB of text, measured separately on the
  build of step 1), compares the statements, checks the axioms and replays everything in the Lean
  kernel in a single thread. A sample of 960 modules suggests that the replay takes on the order
  of 15 to 20 hours; this is a rough estimate.
- The export ended at about 12:20 on 2026-11-21 and Comparator parsed it (its memory stopped
  growing at 296 GiB). One thread has been busy since then.
- Status at 06:54 on 2026-11-22: that thread has used 18.5 hours of CPU time and the run has not
  ended. The time limit of this job ends at 16:30, so a second copy of the run was started on
  another node with a time limit of 4 days.
- There is no verdict yet. This section and the report will be updated when the run ends.

### Step 3: HTML report

The report is published at **<https://shosonoda.github.io/lean-rh-alpoge/>**. It is one
self-contained file with an English / Japanese switch; the same file is in this repository as
[`report/index.html`](report/index.html). It contains the summary of the build logs, the file
structure of the sources, and the statistics of the dependency graph of declarations, read from
the `.olean` files of step 1. Some of its numbers:

| Item | Value |
|---|---|
| Declarations (constants) in the compiled environment | 2,701,132, of which 1,905,921 in the project (1,548,655 theorems, 331,347 definitions) and 795,211 in Lean, Mathlib and the other libraries |
| Dependency edges | 78,197,009 through types and 92,505,619 through values (proof terms) |
| Cone of the five theorems (everything they depend on) | 1,478,243 constants: 1,265,279 of the project (66.4% of it) and 212,964 of the libraries |
| Axioms in that cone | `propext`, `Quot.sound`, `Classical.choice`; no axiom is declared in a project module |
| Project constants outside the cone | 640,642 (33.6%) |
| Build time per module | median 6.7 s, 99% at most 33 s, slowest 657 s; 403 hours when summed over all modules |

The axiom count comes from our own traversal of the compiled environment. It is independent of
Comparator's check and does not replace it.

The report also draws the structure of the development:

- *Maps of the proof*: the upper layers of the dependency graph below the five theorems, the
  dominator tree (what the theorems use only through one result: seven results, nested in one
  chain, each dominate at least 10% of the cone, starting with `allChi` at 95.9%), the cone by
  distance from the theorems, and a map of the second-level directories.
- *How the sub-theories are used*: the sources and Mathlib are divided into 581 topics; for each
  topic the report gives how many of its results are used directly and its reach (the share of
  the project constants in the cone that depend on it, directly or indirectly), with a closer
  look at the number-theoretic and arithmetic-geometric topics (for example, 31.6% of the project
  constants in the cone depend on `Mathlib.NumberTheory.NumberField` and 34.8% on
  `Mathlib.RingTheory.DedekindDomain`), the most used results of each, and a cluster analysis of
  the dependency network (Leiden communities, modularity 0.80) compared with the directories.

These pictures describe the formal development as compiled. They are not an outline of the
mathematical argument and say nothing about its correctness.

---

# lean-rh-alpoge（日本語）

<https://alpo.ge> で配布されている Lean 4 プロジェクトを独立にビルドし，ビルドレポートと，ファイル構造・
証明ネットワークの統計分析をまとめるリポジトリです．

このプロジェクトは Mathlib の上に作られた Lake パッケージ（`.lean` ファイル 175,187 個，約 1,755 万行）で，
`Main.lean` は 5 つの定理を述べています．その 1 つ目が次のものです．

```lean
theorem riemannHypothesis : RiemannHypothesis := rhOfAll
```

`RiemannHypothesis` は Mathlib の定義です．このリポジトリは，プロジェクトをビルドして検査した結果を記録する
ものであり，レポートが示す以上のことは主張しません．

## データの入手

ソースは 1 つの gzip 圧縮 tar を 14 個に分割した形で配布されています（合計 239,275,554 バイト）．

```bash
for i in {01..14}; do 
  curl -fL --retry 3 -O "https://alpo.ge/lean.tar.gz.part${i}" || break
done
```

連結して展開すると `lean/` ディレクトリができます．

```bash
cat lean.tar.gz.part* | shasum -a 256   # 9631b4f020199e580ba86178cb56c6e9b09f690c2369a80dc4334aa93f49c7fc
cat lean.tar.gz.part* | tar -xzf -
```

チェックサムは 2026-11-21 に取得した分割ファイルのものです．

## Lean プロジェクト

| 項目 | 値 |
|---|---|
| Lean toolchain | `leanprover/lean4:v4.33.1`（`lean/lean-toolchain`） |
| Mathlib | リビジョン `0df444a360eaa60ab8c11dca51a86af692955474`（`lean/lakefile.toml`） |
| Lake パッケージ | `tree`，ソースは `lean/src`，ライブラリターゲット 11 個 |
| 規模 | 175,187 モジュール，ソース 1.03 GB，17,549,402 行 |
| import | プロジェクト内の import は 753,569 本．import グラフは閉路がなく深さ 202．すべてのモジュールが `Main` から直接または間接に import されている |

`lean/src/Statements.lean` は，`Main.lean` が `RiemannHypothesis` と並べて証明する 4 つの命題（ゼータ関数と
Dirichlet L 関数に関するもの）を定義しています: `RiemannZetaZerosOnLine`，`DirichletLNonvanishingRightOfHalf`，
`DirichletLZerosOnLine`，`PrimitiveDirichletLZerosOnLine`．

ビルド方法（Mathlib はコンパイルせず，ビルド済みキャッシュを取得します）:

```bash
cd lean
lake exe cache get
lake build
```

## レポート

ビルドはスーパーコンピュータ RIKYU（理研 R-CCS．aarch64，1 ノード 144 コア）で行います．
各ステップが終わるたびに，ここにレポートを追加します．

1. `lake exe cache get` と `lake build` によるビルド — **完了（下記）**．
2. [Comparator](https://github.com/leanprover/comparator) を付けた再ビルド．Comparator は解答をサンドボックス内で
   ビルドし直し，定理の主張を課題ファイルと比較し，使われた公理を検査し，証明を Lean のカーネルで再生します — **実行中．判定はまだ出ていません**（下記）．
3. HTML レポート（英語／日本語）: ビルドログの要約，プロジェクトのファイル構造，証明オブジェクトのネットワークの統計 —
   **<https://shosonoda.github.io/lean-rh-alpoge/>**（初版．ステップ 2 が終わったら更新します）．

### ステップ 1: `lake exe cache get` と `lake build`（2026-11-21）

`lake build` は終了コード 0 で終わり，プロジェクトの 175,187 モジュールすべてがコンパイルされました．

| 項目 | 結果 |
|---|---|
| コマンド | `lake exe cache get` の後に `lake build -v`（既定のターゲット．Mathlib はビルド済みキャッシュ 8,690 ファイルを使用） |
| `lake build` の終了コード | 0 |
| コンパイルされたモジュール | 175,187 個中 175,187 個（`.olean` が 1 個ずつ，合計 88.1 GiB） |
| 失敗した Lake ジョブ | 0 |
| ログ中の `error` 行 | 0 |
| `declaration uses 'sorry'` の警告 | 0 |
| ログ中の警告行 | 最後の実行で 460,321 行（内訳は HTML レポートに載せます） |
| 計算機 | RIKYU の 1 ノード（NVIDIA GB200 NVL4，Arm Neoverse-V2，Ubuntu 24.04）．64 コアと 800 GB を確保し，Lake は 64 スレッド |
| 時間 | 最後の実行の `lake build` は 6 時間 21 分で，170,014 モジュールをコンパイルしました．最初の 5,173 モジュールは，その前の短い 2 つのジョブ（32 コア．ジョブ全体で 18 分と 11 分）の出力を引き継いだものです |
| 費用 | 3 回の実行で確保した資源は 13.3 GPU 時間（1 GPU 時間 300 円で 約 4,000 円．GPU 自体は使っていません） |

ビルドは 3 つの Slurm ジョブに分けて行いました．各ジョブは停止時に `.lake/build` を保存し，次のジョブがそこから
続けたので，Lake はビルド済みのモジュールをコンパイルし直さずに再生しました．

このステップで分かるのは，Lean 4.33.1 がプロジェクトの全ファイルをエラーなし・`sorry` の警告なしで処理できる，
ということです．5 つの定理がどの公理に依存しているか，定理の主張が意図したものかどうかは，これだけでは
検査されません．それはステップ 2 の目的です．

### ステップ 2: Comparator（実行中）

Comparator（タグ v4.33.0．Lean 4.33.1 でビルド）には，`Main.lean` の 5 つの定理を `sorry` で述べた課題ファイル，
解答モジュール `Main`，許可する公理 `propext`・`Quot.sound`・`Classical.choice` を与えています．Comparator は
サンドボックスの中でプロジェクトをビルドし直してから，証明をエクスポートして検査します．

- サンドボックス内の再ビルドは 1 つのバッチジョブに収まりませんでした．最初のジョブ（8 時間）は 175,187 モジュール中
  162,328 個をビルドしたところで止まり，2 つ目のジョブが保存したビルドの続きを実行して，2026-11-21 の 11:01 に Lake が
  `Build completed successfully (183893 jobs)` と報告しました．
- 再ビルドの後，Comparator は証明をエクスポートし（テキストで 61.6 GB．ステップ 1 のビルドに対して別途測定），
  主張を比較し，公理を検査し，全体を Lean のカーネルで単一スレッドで再生します．960 モジュールの標本からは，再生に
  15〜20 時間程度かかると見込まれます（粗い見積もりです）．
- エクスポートは 2026-11-21 の 12:20 頃に終わり，Comparator がそれを解析しました（メモリは 296 GiB で増加が止まりました）．
  その後は 1 スレッドが動き続けています．
- 2026-11-22 06:54 時点の状況: そのスレッドの CPU 時間は 18.5 時間に達しましたが，実行はまだ終わっていません．
  このジョブの制限時間は 16:30 に来るので，制限時間を 4 日にした同じ実行を別のノードでも始めました．
- 判定はまだ出ていません．実行が終わったら，この節とレポートを更新します．

### ステップ 3: HTML レポート

レポートは **<https://shosonoda.github.io/lean-rh-alpoge/>** で公開しています．英語／日本語を切り替えられる
単一のファイルで，同じものをこのリポジトリの [`report/index.html`](report/index.html) にも置いています．ビルドログの要約，ソースのファイル構造，宣言の依存グラフの統計（ステップ 1 の `.olean`
ファイルから読み取ったもの）を載せています．主な数値:

| 項目 | 値 |
|---|---|
| コンパイル済み環境の宣言（定数） | 2,701,132 個．うちプロジェクトが 1,905,921 個（定理 1,548,655，定義 331,347），Lean・Mathlib などのライブラリが 795,211 個 |
| 依存の辺 | 型を通るものが 78,197,009 本，値（証明項）を通るものが 92,505,619 本 |
| 5 つの定理のコーン（依存先すべて） | 1,478,243 個．プロジェクトの定数が 1,265,279 個（プロジェクトの 66.4%），ライブラリの定数が 212,964 個 |
| コーンに含まれる公理 | `propext`，`Quot.sound`，`Classical.choice`．プロジェクトのモジュールで宣言された公理はありません |
| コーンの外にあるプロジェクトの定数 | 640,642 個（33.6%） |
| モジュールあたりのビルド時間 | 中央値 6.7 秒，99% が 33 秒以下，最長 657 秒．全モジュールの合計は 403 時間 |

公理の数は，コンパイル済みの環境をこちらでたどって数えたものです．Comparator の検査とは独立で，その代わりには
なりません．

レポートには，開発の構造を描いた図も載せています．

- 「証明の可視化」: 5 つの定理の下にある依存グラフの上層，支配木（定理がある 1 つの結果を通してしか使わない部分．
  コーンの 10% 以上を支配する結果は 7 個で，`allChi`（95.9%）から始まる 1 本の鎖になっています），定理からの距離別の
  コーン，第 2 階層のディレクトリの地図．
- 「部分理論の使われ方」: ソースと Mathlib を 581 個のトピックに分け，トピックごとに，直接使われている結果の数と
  到達率（コーンに含まれるプロジェクトの定数のうち，そのトピックに直接または間接に依存するものの割合）を示します．
  数論・数論幾何に関するトピックは詳しく扱っています（たとえば，コーンに含まれるプロジェクトの定数の 31.6% が
  `Mathlib.NumberTheory.NumberField` に，34.8% が `Mathlib.RingTheory.DedekindDomain` に依存しています）．
  各トピックでよく使われている結果の表と，依存ネットワークのクラスタ分析（Leiden 法のコミュニティ，モジュラリティ 0.80）
  とディレクトリ構造との比較も載せています．

これらの図は，コンパイルされた形式的な開発を記述するものです．数学的な議論の概略ではなく，その正しさについて
何かを述べるものでもありません．
