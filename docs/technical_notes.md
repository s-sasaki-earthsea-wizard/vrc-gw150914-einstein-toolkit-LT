# 技術メモ — 詰まった点と解決策

LT で「動かしてみた」の苦労話パートに使えるネタ集。詳細は元プロジェクトの
`CLAUDE.md` を参照。

## 1. Docker 環境構築の地雷 (Phase 1)

### 既製イメージが使えなかった

- `einsteintoolkit/jupyter-et` は Docker Hub に存在しない (誤った前提だった)
- `ndslabs/jupyter-et` は実在するが全タグが 4 年以上前で Kruskal release と乖離
- → **Ubuntu 20.04 ベースで自前ソースビルド** に方針転換

### MPI 実装の罠

公式 jupyter-et の `base.docker` は **MPICH** を採用している。
Ubuntu の `libscalapack-mpi-dev` は OpenMPI を依存に引き込み、
`update-alternatives` の auto モードでは Open MPI が選ばれてしまう。

```dockerfile
# Dockerfile 末尾で MPICH を明示固定
RUN update-alternatives --set mpirun /usr/bin/mpirun.mpich
```

HDF5 (`libhdf5_mpich.so.103`) や ADIOS2 が MPICH ABI でビルドされているため、
Open MPI で起動するとシンボル衝突や I/O 不整合のリスクあり。

### 起動時オプション

| オプション | 値 | 理由 |
| --- | --- | --- |
| `shm_size` | 4gb | MPICH のプロセス間共有メモリ通信用 (デフォルト 64MB では不足) |
| `cpuset` | 0-15 | 物理コアに固定 (NUMA 安定化) |
| `mem_limit` | 80g | ホスト 93GiB のうち 80GB をコンテナに割り当て |
| `USER_UID` / `USER_GID` | ホスト一致 | バインドマウント時のパーミッション一致 |

## 2. rpar grid と解像度 N の罠 (Phase 3a, 3b-ii)

公式 GW150914.rpar は **N=28 前提のグリッド設計**。N<28 では起動直後に
crash する。

### crash の真因 (Phase 3b-ii で特定)

`Interpolate2/test.cc:77` の制約:
**「refinement level > 0 のグリッドが inter-patch 境界マーク (Sn>=0) を
持つ点を 1 つでも含むと abort」**

事前仮説「`maxrls` を縮小すれば level-1 box が縮んで侵入しない」は
**実測で否定**:

| maxrls | rlsp/rlsm | 最外層 box | 結果 |
| --- | --- | --- | --- |
| 9 | 7 | 21.27 M | ❌ inter-patch crash |
| 8 | 6 | 10.64 M | ❌ 同 crash |
| 7 | 5 | 5.32 M | ❌ 同 crash |

box を縮小しても **Carpet の prolongation buffer + ghost zones が物理単位で
広がり、境界領域に侵入する**ため効果なし。

| | h₀ | buffer 厚 | sphere_inner_radius (公式) | 結果 |
| --- | --- | --- | --- | --- |
| N=28 | 1.224 M | ~6 M | 51.40 M | OK |
| N=16 | 2.143 M | ~10.7 M (1.75 倍) | 51.40 M | crash |

→ **inner Cartesian patch を物理的に拡大** (sphere_inner_radius を
51.40 M → 77.10 M = rpar 自然 step `i × h₀ = 8.566 M` の 9 倍に snap) で解決。

### 半整数倍制約

`Coordinates/patchsystem.cc:177`:
**`2 * sphere_inner_radius / h_cartesian` が整数 (誤差 1e-8 以内) でないと
起動時に MPI_ABORT**。
→ `scripts/generate_gw150914_n16_parfile.py::snap_inner_radius()` で
`i × h₀` 単位に ceil-snap して回避。

## 3. HDF5 1.10.4 checkpoint 書き込み問題 (Phase 3a, 3c-1)

複数 MPI rank が同一 checkpoint ファイルに同時 open すると
POSIX lock が衝突:

```
H5FD_sec2_lock: errno=11 'Resource temporarily unavailable'
```

- 出力先が local SSD (ext3) でも発生 (NFS 関係ない)
- `HDF5_USE_FILE_LOCKING=FALSE` env var は **通常 I/O には効くが
  checkpoint 経路では不十分**
- Phase 3a smoke では checkpoint 完全無効化で回避
- **Phase 3c-1 で `np=1 × OMP=16` 構成では再発しないことを確認** →
  本番採用構成。Phase 3a/3b で踏んだ lock 問題は np>=2 固有の現象だった

## 4. MPI / OMP 構成の最適化 (Phase 3b-i)

### N=28 の np スイープ実測 (TwoPunctures のみ)

| Config | Wall time | Peak mem | 完走 |
| --- | --- | --- | --- |
| **np=1 × OMP=16** | **6m22s** | **43 GiB** | ✅ |
| np=2 × OMP=8 | 26m33s | 80 GiB | ⚠ cleanup 時 OOM |
| np=4 × OMP=4 | 297m (OOM ハング) | 80 GiB | ❌ |

**np を増やすほど悪化** (qc0 と真逆)。理由: Multipole / WeylScal4 の
球面調和バッファが rank ごとに複製される。GW150914 N=28 / N=16 は
**np=1 × OMP=16 が唯一安定**。

### `OMP_NUM_THREADS` 明示は必須

未指定だと Cactus が全コアスレッド化して `mpirun -np N` と掛け合わせて
N² スレッドで oversubscription。`makefile/sim.mk` で
`mpirun -np $(SIM_MPI_PROCS) -genv OMP_NUM_THREADS $(SIM_OMP_THREADS)` を
明示展開している。

## 5. Stage 跨ぎ中断再開フローの事故 (Phase 3c-3)

Stage B 本番 run の途中 (it_69124, 約 154 M physical time) で
中断・再投入した際、3 つの問題が同時発生:

1. **Stage A の checkpoint から再起動**: `run-gw150914-n16-stage-b` は
   `--continue-from A` を渡すため `recover_dir = ../checkpoints/gw150914-n16-stage-a`
   を生成。`recover = autoprobe` は `recover_dir` のみ参照するため、Stage B の
   it_69124 ckpt は無視されて **Stage A 終端 (it_26568) からやり直し**になる
2. **cactus_sim 並行二重起動**: ユーザが `nohup make ... &` を 2 度叩いた
   結果、22 秒差で 2 系統の mpirun が起動。同じ output dir に並行書き込みで
   解析対象データが破損
3. **MPI 構成が `np=4 × OMP=1`**: sim.mk default が qc0 ベースラインの
   ままだった。N=16 では np>=2 で OOM + HDF5 lock 衝突のリスクあり

### 対策

- `run-*` (cross-stage continuity) と `resume-*` (同 stage 中断再開) を分離
- `cactus_sim` の二重起動 pre-check (alive プロセス検出で abort)
- sim.mk default を `np=1 × OMP=16` に変更
- `recover_dir` が `recover = autoprobe` の唯一の参照先であることを明文化

## 6. 比較設計の知見 (Phase 4)

### 完走判定は puncturetracker の t_max を使う

`BH_diagnostics.ah1` は merger 後に出力停止 (Zenodo N=28 で t≈910 M で停止
確認)。Stage B / C target=1000 / 1700 M いずれにも対応する判定指標として
puncture が安定。

### 軌道数は両 sim の共通開始時刻で計算

N=16 self-run の puncturetracker 出力は trigger 仕様で **t≈261 M から**、
Zenodo は t=0 から。同一時間窓で計算しないと N=16 だけが早期 ≈1 軌道分を
欠落させて見える。

### Stage A→B 境界の puncturetracker ギャップ

Stage A は puncturetracker を t=0 から連続出力するが、Stage B (Stage A
checkpoint からの restart) は trigger 仕様で **t=261.16 M から再出力**
されるため、t=99.26 → 261.16 の **162 M 区間でデータ抜け**が発生。
`np.unwrap` がギャップ越しに位相 jump を誤計算し、軌道数が −0.85 ズレる
現象が発生。

→ `compare_stage_c.effective_pt_t_min()` で「最大ギャップ以降の連続区間 start」
(= 261.16 M) を有効 t_min として採用する自動補正で解消。
中央値 dt の 5 倍を超えるギャップを「大ギャップ」と判定する閾値
`MAX_GAP_FACTOR = 5.0`。

### TARGET_TIME_SNAP_TOLERANCE_M = 1.0 M

Cactus の ASCII 出力頻度の都合で、実 simulation が target に到達していても
ASCII 最終サンプルが手前に止まることがある (Stage A 実測: 100.013 M 完走 /
ASCII 最終 99.26 M)。許容差以内なら最終サンプル値にスナップして比較する。

## 7. その他細かい知見

### Makefile の `${PIPESTATUS[0]}` が dash で壊れる

`${PIPESTATUS[0]}` は **bash 専用**。Make のデフォルト SHELL (= `/bin/sh` →
Debian/Ubuntu で dash) では `Bad substitution` エラーで exit code が壊れる。
`makefiles/sim.mk` 冒頭に `SHELL := /bin/bash` を追加して解消。

### MPICH mpirun の OOM 挙動

1 rank が OOM kill されても残り rank が `MPI_Wait` で **無限停止**する
(np=4 試行で 297 分ハング)。Phase 3c ではタイムアウト付き実行を推奨。

### sec/iter の見積もり vs 実測ズレ

| Phase | 想定 sec/iter | 実測 sec/iter | 備考 |
| --- | ---: | ---: | --- |
| 3b-ii (16 iter まで) | 1.84 | — | AMR 立ち上がり込み |
| 3c-1 以降 (steady state) | — | 0.89 | post-merger は更に軽量化 |

Phase 3b-ii の 1.84 は最初の 16 iter (AMR 立ち上がり込み) の値で、steady
state の倍程度のオーバーヘッドが含まれていた。
本番予想 wall time は **(1.84 → 0.89 で) 半分に短縮** (Stage C の 4.7 日が
2.7 日相当)。実測は 27.9 時間 (1.16 日) で更に短かった
— post-merger で AMR が単純化するため。
