# vrc-gw150914-einstein-toolkit-LT

VRChat 物理学集会 LT 用スライド資料 — **Einstein Toolkit による GW150914
（連星ブラックホール合体）数値相対論シミュレーション再現** の発表用プロジェクト。

スライド本体は [reveal.js](https://github.com/hakimel/reveal.js) で構成し、
[lt-slide-template](https://github.com/s-sasaki-earthsea-wizard/lt-slide-template) を
ベースにしている。

## 概要

2015 年 9 月 14 日に LIGO が直接検出した最初の重力波イベント **GW150914** を、
個人マシン (16 コア / 93 GiB RAM) 上の Docker 環境で **Einstein Toolkit** Kruskal release
を用いて再現したプロジェクトの成果報告。

- 元プロジェクト: <https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit>
- 公式ギャラリー (参照): <https://einsteintoolkit.org/gallery/bbh/index.html>
- Zenodo N=28 reference: <https://doi.org/10.5281/zenodo.155394>

公式ギャラリーの解像度 N=28 (128 コア × 2.8 日) は実現不可能なため、
**N=16 に落として 84 時間で完走**。それでも以下の精度で N=28 reference と一致した:

| 量 | 公式 (N=28) | N=16 (本研究) | 誤差 |
| --- | ---: | ---: | ---: |
| マージャーまでの時間 | 899 M | 925.1 M | +2.94% |
| 最終 BH 質量 | 0.95 M | 0.9518 M | -0.10% |
| 最終 BH スピン | 0.69 | 0.6930 | +0.0054 abs |
| ψ4 peak amplitude | — | 7.21e-4 | -1.79% |

詳細は [docs/results.md](docs/results.md) 参照。

## ディレクトリ構成

```text
.
├── docs/             # 元プロジェクトから移管したドキュメント
│   ├── project_overview.md
│   ├── phase_summary.md
│   ├── results.md
│   ├── technical_notes.md
│   ├── comparison_method_n16_vs_n28.md
│   ├── img/phase4/   # 代表 plot 4 枚
│   └── results/      # Stage A/B/C 比較 JSON
└── slides-jp/        # reveal.js スライド (本体は今後執筆)
    ├── index.html
    ├── yt_script.md
    └── youtube_description.md
```

## 開発環境

- Node.js 18 以上 (reveal.js 5.x の要求)
- 必要に応じて [decktape](https://github.com/astefanutti/decktape) (PDF エクスポート用)

## 使い方

### スライドのプレビュー

```bash
cd slides-jp
npm install      # 初回のみ
npm start
```

ブラウザで <http://localhost:8000> にアクセス。

### スライドの PDF エクスポート

```bash
cd slides-jp
npm install -g decktape   # 初回のみ
decktape --size 1920x1080 index.html slides.pdf
```

## 関連ドキュメント

- [docs/project_overview.md](docs/project_overview.md) — プロジェクトの目的とスコープ
- [docs/phase_summary.md](docs/phase_summary.md) — Phase 0〜4 進行履歴
- [docs/results.md](docs/results.md) — Stage A/B/C の比較結果
- [docs/technical_notes.md](docs/technical_notes.md) — 詰まった点と解決策
- [docs/comparison_method_n16_vs_n28.md](docs/comparison_method_n16_vs_n28.md) — 比較手法の設計

## 免責事項

本スライドの内容に従ったいかなる結果においても著者は一切の責任を負いません。

## LICENSE

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

This project uses [reveal.js](https://github.com/hakimel/reveal.js) which is also licensed
under the MIT License.
