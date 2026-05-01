# Phase 進行履歴サマリ

元プロジェクト [gw150914-einstein-toolkit](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit)
の Phase 0〜4 完了までの簡潔な記録。

## Phase 一覧

| Phase | 内容 | Issue | 主な成果 |
| --- | --- | --- | --- |
| 0 | プロジェクト初期化 | - | スコープ確定 (N=16 + 自宅 16 コア) |
| 1 | Docker 環境構築 | [#1](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/1) | Ubuntu 20.04 + Kruskal release `ET_2025_05` のソースビルド完成 |
| 2 | パラメータファイル取得・テスト基盤 | [#2](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/2) | rpar 取得 + Level 1/2 テスト整備、N=28 で TwoPunctures 6 分 18 秒 |
| 3a | qc0-mclachlan による ET feasibility 確認 | [#10](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/10) | smoke 26 分で完走、make ターゲット化 |
| 3b-i | N=28 メモリ・時間 feasibility 計測 | - | np=1 OMP=16 で 50 GiB / 16 日見込み → **N=16 方針へ転換** |
| 3b-ii | N=16 対応の rpar grid 改変 | [#9](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/9) | sphere_inner_radius 拡大で 1.84 sec/iter, ringdown 5.7 日見込み |
| 3c-1 | checkpoint write/restart 動作確認 | [#3](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/3) | np=1 で POSIX lock 不発、walltime + on_terminate 両経路 OK |
| 3c-2 | Stage A (0 → 100 M) | [#3](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/3) | 6h51m, 100.013 M, peak 26.91 GiB |
| 3c-3 | Stage B (100 → 1000 M) | [#21](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/21) | 49h39m, 1000.01 M, peak 28.76 GiB |
| 3c-4 | Stage C (1000 → 1700 M) | [#3](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/3) | 27h54m, 1700.01 M, peak 22.79 GiB |
| 4 | 軌道・波形の抽出とプロット | [#4](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/4) | Stage A/B/C 比較完走、Stage C overall_pass=True |
| 5 | 3D 可視化 | [#5](https://github.com/s-sasaki-einstein-toolkit/issues/5) | 未着手 (オプション) |

## 全体タイムライン (2026 年)

```
04 月 04 日頃  Phase 0 開始 (プロジェクト発案、スコープ確定)
04 月        Phase 1 完了 (Docker 環境構築)
04 月 24 日   Phase 2 完了 (rpar 取得 + テスト基盤)
04 月 24 日   Phase 3a 完了 (qc0 smoke 26 分)
              Phase 3b-i 完了 (N=28 feasibility 確認 → N=16 方針確定)
04 月 25 日   Phase 3b-ii 完了 (N=16 grid 改変 + sphere_inner_radius 拡大)
04 月 25 日   Phase 3c-1 完了 (checkpoint 動作確認)
04 月 26 日   Phase 3c-2 完了 (Stage A 100 M)
04 月 29 日   Phase 3c-3 完了 (Stage B 1000 M)
05 月 01 日   Phase 3c-4 完了 (Stage C 1700 M)
05 月 01 日   Phase 4 完了 (Stage A/B/C 全比較 + 全 7 check pass)
```

## 設計判断のターニングポイント

### 1. N=28 → N=16 への解像度引き下げ (Phase 3b-i)

N=28 は 16 コアでも物理的にメモリは足りる (peak 50 GiB, 80 GB 上限内)。
しかし **ringdown まで到達するのに wall time 16 日以上** 必要と判明。
ユーザ目標 (~2 週間) では完走不可能。
→ 解像度 N=16 (空間刻み 1.75 倍粗い) に落として 1〜3 日で完走させる方針へ転換。
**検証は Zenodo 10.5281/zenodo.155394 の N=28 reference データとの比較**
で代替する。

### 2. rpar grid 改変の真因特定 (Phase 3b-ii)

公式 GW150914.rpar は **N=28 専用の grid 設計**。N<28 では Carpet 内部の
prolongation buffer が物理単位で 1.75 倍に広がり、inter-patch 境界に侵入
して `Interpolate2/test.cc:77` で abort する。
事前仮説「`maxrls` を縮小すれば level-1 box が縮んで侵入しない」は実測で否定。
**真の解決策は inner Cartesian patch を物理的に拡大すること** (sphere_inner_radius
51.40 M → 77.10 M = rpar 自然 step の 9 倍に snap)。

### 3. Phase 3c の staged validation (3c-1 〜 3c-4)

1700 M まで一気に走らせて Zenodo と一括比較するのは効率が悪い
(失敗箇所の局所化困難 + 早期失敗時のコスト過大)。
**3c-1 (checkpoint) → 3c-2 (Stage A 100 M) → 3c-3 (Stage B 1000 M)
→ 3c-4 (Stage C 1700 M)** の段階制にして、各 stage 終了後に Zenodo
比較で go/no-go 判定する方式を採用。

### 4. 中断再開フローの教訓 (Phase 3c-3)

Stage B 途中の中断・再投入で **Stage A の checkpoint から再起動してしまう**
事故が発生。`recover = autoprobe` は `recover_dir` のみ参照するため、
`run-*` (cross-stage continuity 用) と `resume-*` (同 stage 中断再開用) の
ターゲットを分離。`cactus_sim` 二重起動 pre-check も追加。

## Zenodo N=28 reference データの内訳調査

[Phase 4 タスク A1〜A5] で確認した要点:

- アーカイブは 466 MB tar.xz (展開後 720 MB)
- 6 個のディレクトリ `output-0000` 〜 `output-0005` は **checkpoint ではなく
  SimFactory walltime restart segment**。output-0000 (0–246 M) が
  Stage A 100 M をカバー
- `mp_psi4.h5` は **81 (l, m) modes × 7 抽出半径** [100, 115, 136, 167, 214, 300, 500] M
- t=100 M reference: D=9.773 M, |ψ4₂₂(r=100)|=1.67e-5,
  χ₁=+0.3102, χ₂=−0.4604 (rpar 設定値と完全一致、drift < 3e-4)
- QLM 慣習: `qlm_spin` は **角運動量 J (次元あり)**、
  無次元スピン χ = J/M_horizon² で換算する必要あり
- **constraint norm 出力 (Hamiltonian/Momentum) は Zenodo 側になし**
  → self-consistency 検証は N=16 自前 run のみで実施
