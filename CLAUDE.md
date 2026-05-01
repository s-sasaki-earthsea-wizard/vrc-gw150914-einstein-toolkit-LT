# vrc-gw150914-einstein-toolkit-LT

## プロジェクト概要

**VRChat 物理学集会 LT 用スライド資料**。
[gw150914-einstein-toolkit](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit)
プロジェクト (Einstein Toolkit による GW150914 数値相対論シミュレーション再現)
の発表資料を `reveal.js` で作成する。

スライド本体・脚本は別途このプロジェクトで執筆する。`docs/` 以下に元プロジェクトから
移管した素材 (Phase 履歴、結果サマリ、技術メモ、代表 plot) を置いている。

## 関連リンク

- 元プロジェクト: <https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit>
- スライドテンプレート: <https://github.com/s-sasaki-earthsea-wizard/lt-slide-template>
- LIGO 公式論文: <https://doi.org/10.1103/PhysRevLett.116.061102>
- Einstein Toolkit 公式ギャラリー: <https://einsteintoolkit.org/gallery/bbh/index.html>
- Zenodo N=28 reference: <https://doi.org/10.5281/zenodo.155394>

## 言語設定

このプロジェクトでは**日本語**での応答を行ってください。コード内のコメント、ログメッセージ、
エラーメッセージ、ドキュメンテーション文字列なども日本語で記述してください。

## 開発ルール

### コーディング規約 (スライド資料 Markdown / HTML)

- インデント: スペース 2 文字 (HTML / JS) もしくは 4 文字 (Python ヘルパスクリプトを書く場合)
- Markdown は CommonMark 準拠

### スライド作成

- ベース: [reveal.js](https://github.com/hakimel/reveal.js) 5.1.0
- ディレクトリ: `slides-jp/`
- メインファイル: `slides-jp/index.html`
- 脚本: `slides-jp/yt_script.md` / `slides-jp/youtube_description.md`
- カスタムスタイル: `slides-jp/styles/custom-style.css`

### Git 運用

- ブランチ戦略: `feature/*`, `fix/*`, `refactor/*`
- コミットメッセージ: 英文を使用、動詞から始める
- PR は main ブランチへ

### コミットメッセージ規約

#### コミット粒度

- **1 コミット = 1 つの主要な変更**
- 関連する変更は 1 つのコミットにまとめる
- 大きな変更は段階的に分割

#### プレフィックスと絵文字

- ✨ feat: 新機能
- 🐞 fix: バグ修正
- 📚 docs: ドキュメント
- 🎨 style: コードスタイル修正
- 🛠️ refactor: リファクタリング
- ⚡ perf: パフォーマンス改善
- ✅ test: テスト追加・修正
- 🏗️ chore: ビルド・補助ツール
- 🚀 deploy: デプロイ
- 🔒 security: セキュリティ修正
- 📝 update: 更新・改善
- 🗑️ remove: 削除

**重要**: Claude Code を使用してコミットする場合は、必ず以下の署名を含める:

```text
🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## ディレクトリ構成

```text
.
├── CLAUDE.md                    # 本ファイル
├── README.md                    # プロジェクト README
├── Makefile                     # トップレベル make ターゲット
├── organizer.yaml               # GitHub branch protection 設定
├── docs/                        # 元プロジェクトから移管したドキュメント
│   ├── project_overview.md      # GW150914 シミュレーションの概要
│   ├── phase_summary.md         # Phase 0-4 タイムライン
│   ├── results.md               # Stage A/B/C 比較結果サマリ
│   ├── technical_notes.md       # 詰まった点と解決策
│   ├── comparison_method_n16_vs_n28.md  # 比較手法の設計
│   ├── img/phase4/*.png         # 代表 plot 4 枚
│   └── results/*.json           # Stage A/B/C 比較 JSON
└── slides-jp/                   # reveal.js スライド (本体は今後執筆)
    ├── index.html               # スライド本体 (TBA placeholder)
    ├── yt_script.md             # YouTube 脚本 (TBA)
    ├── youtube_description.md   # YouTube 動画概要文 (TBA)
    └── ...
```

## スライド作成の進め方 (今後)

1. `docs/project_overview.md` から発表ハイライトを抽出
2. `docs/results.md` の Stage C 全 7 check pass 結果を主要メッセージに据える
3. `docs/technical_notes.md` の苦労話から LT 向け 1〜2 トピックを選定
4. `docs/img/phase4/*.png` を `slides-jp/assets/images/` にコピーして埋め込み
5. `slides-jp/index.html` のセクションを書く
6. 脚本 (`yt_script.md` / `youtube_description.md`) を整える
