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
   used, and replays the proofs in the Lean kernel — running.
3. HTML report (English / Japanese): summary of the build logs, the file structure of the project,
   and statistics of the network of proof objects — in preparation.

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
   ビルドし直し，定理の主張を課題ファイルと比較し，使われた公理を検査し，証明を Lean のカーネルで再生します — 実行中．
3. HTML レポート（英語／日本語）: ビルドログの要約，プロジェクトのファイル構造，証明オブジェクトのネットワークの統計 — 準備中．

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
