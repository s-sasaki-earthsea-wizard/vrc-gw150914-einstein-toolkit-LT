# 結果サマリ — Stage A / B / C

N=16 自前 run (Stage A+B+C 連結) を Zenodo N=28 reference (10.5281/zenodo.155394)
および公式ギャラリー値と比較した結果。最終的に **Stage C overall_pass=True**
で 7 つの全 check を通過。

## ヘッドラインメトリクス

| 量 | 公式 (N=28) | **N=16 (本研究)** | N=28 (Zenodo) | N=16 vs N=28 |
| --- | ---: | ---: | ---: | ---: |
| 軌道数 | 6 | **5.08 + 早期 ≈ 6** | 4.92 + 早期 ≈ 6 | +0.15 軌道 |
| マージャーまでの時間 | 899 M | **925.1 M** | 898.7 M | **+2.94%** |
| 最終 BH 質量 M_f | 0.95 M | **0.9518 M** | 0.9527 M | **−0.10%** |
| 最終 BH スピン χ_f | 0.69 | **0.6930** | 0.6877 | +0.0054 abs |
| ψ4 peak amplitude (r=100 M) | — | **7.21e-4** | 7.34e-4 | **−1.79%** |
| ψ4 peak time after merger | — | 113.72 M | 113.99 M | −0.28 M |

**N=16 が公式ギャラリー値・Zenodo N=28 reference の両者と高精度で一致した。**

## Stage C 全 7 check の合否

| Check | N=16 | N=28 (Zenodo) | 差分 | 閾値 | Pass |
| --- | ---: | ---: | ---: | ---: | :---: |
| completion (target=1700 M) | 1699.953 M | — | — | ≥ 1700 ± 1 M | ✓ |
| merger_time | 925.145 M | 898.712 M | +2.94% | ±5% | ✓ |
| m_final | 0.9518 | 0.9527 | −0.10% | ±2% | ✓ |
| chi_final | 0.6930 | 0.6877 | +0.0054 abs | ±0.02 | ✓ |
| n_orbit | 5.078 | 4.923 | +0.15 | ±0.5 | ✓ |
| ψ4 peak amplitude (r=100 M) | 7.21e-4 | 7.34e-4 | −1.79% | ±10% | ✓ |
| ψ4 peak time after merger | 113.72 M | 113.99 M | −0.28 M | ±20 M | ✓ |

詳細 JSON: [results/stage_c_pass_fail.json](results/stage_c_pass_fail.json)

## Self-consistency (拡張 ringdown 窓)

N=16 単体でリングダウン中の最終 BH 質量・スピンが時間とともにどれだけ
ドリフトするかを評価。merger 後 50–500 M の窓に 467 サンプル:

| 量 | drift | 閾値 | Pass |
| --- | ---: | ---: | :---: |
| m_final drift | 0.0038% | 1.0% | ✓ |
| chi_final drift | 0.000494 abs | 0.05 | ✓ |

**極めて安定**で、ringdown が真の Kerr 解 (M_f, χ_f 一定) に漸近していることを
定量的に確認。

## 計算リソース実測

| 段階 | wall time | peak メモリ | 終了原因 |
| --- | --- | --- | --- |
| Stage A (0 → 100 M) | 6h51m | 26.91 GiB | cctk_final_time |
| Stage B (100 → 1000 M) | 49h39m | 28.76 GiB | cctk_final_time |
| Stage C (1000 → 1700 M) | 27h54m | 22.79 GiB | cctk_final_time |
| **合計** | **84h24m** | — | — |

Stage C のメモリが Stage A/B より小さいのは、merger 後に punctures が消失して
高 refinement レベルが common horizon 周辺のみになり、AMR 構造が単純化したため。

## 物理的解釈

### merger time +2.94% 遅延

N=16 解像度で数値消散が大きく radiation reaction が underestimate される
系統的傾向と整合。公式 ±5% 閾値内に収まっている。

### M_f / χ_f の異常な一致 (0.10% / 0.5%)

ringdown 早期の値は merger dynamics により決まる「保存的」量であり、
AMR が高解像度層を merger 近傍に集中させるため解像度依存性が小さい。
Phase 3b-ii で行った `sphere_inner_radius` 拡大が物理にネガティブな影響を
与えていない証拠でもある。

### ψ4 peak amplitude の −1.79% 一致

Stage B 比較時点では merger + 74 M しか到達していなかったため
peak (merger + 100 M で発生) を捕捉できず provisional 扱いだった。
Stage C で初めて真の peak (merger + 113 M ≈ 1039 M) が検証でき、
**振幅・時刻の両方で N=28 reference と高精度で一致**することが示された。
波形 morphology が解像度に依らないことの強い証拠。

## 代表 plot

| ファイル | 内容 |
| --- | --- |
| [img/phase4/orbit_trajectory.png](img/phase4/orbit_trajectory.png) | 連星 BH の軌道 (xy 平面) |
| [img/phase4/orbit_separation_aligned.png](img/phase4/orbit_separation_aligned.png) | 軌道分離 D(t)、merger 時刻で時間軸を整列 |
| [img/phase4/psi4_22_amplitude_aligned.png](img/phase4/psi4_22_amplitude_aligned.png) | ψ4₂₂ 振幅 (r=100 M)、merger 整列 |
| [img/phase4/ringdown_aligned.png](img/phase4/ringdown_aligned.png) | リングダウン波形拡大 |

## 元データへのリンク

- 比較スクリプト: [`scripts/analyze/compare_stage_c.py`](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/blob/main/scripts/analyze/compare_stage_c.py)
- Phase 4 完了 PR: [#24](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/pull/24)
- Phase 4 関連 Issue: [#4](https://github.com/s-sasaki-earthsea-wizard/gw150914-einstein-toolkit/issues/4)
