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

1. Build with `lake exe cache get` and `lake build`.
2. Second build under [Comparator](https://github.com/leanprover/comparator), which rebuilds the
   solution in a sandbox, compares the theorem statements with a challenge file, checks the axioms
   used, and replays the proofs in the Lean kernel.
3. HTML report (English / Japanese): summary of the build logs, the file structure of the project,
   and statistics of the network of proof objects.

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

1. `lake exe cache get` と `lake build` によるビルド．
2. [Comparator](https://github.com/leanprover/comparator) を付けた再ビルド．Comparator は解答をサンドボックス内で
   ビルドし直し，定理の主張を課題ファイルと比較し，使われた公理を検査し，証明を Lean のカーネルで再生します．
3. HTML レポート（英語／日本語）: ビルドログの要約，プロジェクトのファイル構造，証明オブジェクトのネットワークの統計．
