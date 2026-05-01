# プロジェクト概要

**GW150914 を Einstein Toolkit で再現する試み** — VRChat 物理学集会 LT 用資料。

## 何を発表するか

2015 年 9 月 14 日に LIGO が直接検出した最初の重力波イベント **GW150914**
（連星ブラックホール合体）を、数値相対論コード **Einstein Toolkit** (Kruskal release,
`ET_2025_05`) で **自宅環境 (16 コア / 93 GiB RAM)** から再現できるか挑戦した
プロジェクトの成果報告。

- 元プロジェクト: <https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit>
- 公式ギャラリー (参照): <https://einsteintoolkit.org/gallery/bbh/index.html>
- Zenodo reference (N=28 シミュレーション結果): <https://doi.org/10.5281/zenodo.155394>
- 観測論文 (参考): [Phys. Rev. Lett. 116, 061102](http://dx.doi.org/10.1103/PhysRevLett.116.061102)

## 研究と教育の境界

本プロジェクトは **研究目的の数値相対論ではない**。
公式ギャラリーの解像度 `N=28` は 128 コア × 2.8 日のリソースを前提としており、
個人マシンでは実現できない。**有名な重力波イベントを「試しに動かしてみる」**
ことを主眼に、解像度を `N=16` に落として軌道・波形の **定性的な再現** を狙った。

| | 公式ギャラリー (N=28) | 本プロジェクト (N=16) |
| --- | --- | --- |
| 並列度 | 128 MPI プロセス | 1 MPI × OMP=16 |
| メモリ | 98 GB | 28.76 GiB (peak, Stage B) |
| 実行時間 | 2.8 日 (128 コア = 8700 コア時間) | Stage A+B+C = 84h |
| 空間刻み h₀ | 1.224 M | 2.143 M (1.75 倍粗い) |

それでも結果は健闘した — 詳しくは [results.md](results.md) を参照。

## GW150914 の物理パラメータ

| 項目 | 値 |
| --- | --- |
| 初期分離 D | 10 M |
| 質量比 q = m₁/m₂ | 36/29 ≈ 1.24 |
| BH1 スピン χ₁ = a₁/m₁ | +0.31 |
| BH2 スピン χ₂ = a₂/m₂ | −0.46 |

## 結論先取り

N=16 解像度でも **マージャー時刻 +2.94%、最終 BH 質量誤差 0.10%、
最終 BH スピン誤差 0.5%、ψ4 peak 振幅誤差 −1.79%** という高精度で公式
ギャラリー値および Zenodo N=28 reference と一致した。
**プロジェクト目標 (有名重力波イベントの数値再現) は達成。**

## 関連ドキュメント

- [phase_summary.md](phase_summary.md) — Phase 0〜4 の進行履歴
- [results.md](results.md) — Stage A/B/C の比較結果サマリ
- [technical_notes.md](technical_notes.md) — 技術的に詰まった点と解決策
- [comparison_method_n16_vs_n28.md](comparison_method_n16_vs_n28.md) — N=16 vs N=28 比較手法の設計
- [img/phase4/](img/phase4/) — 代表 plot 4 枚
- [results/](results/) — Stage A/B/C 比較 JSON
