# README — ode23tb の次数（収束次数）を数値実験で確認する

このミニプロジェクトは、**`ode23tb`（TR-BDF2, 2次・陰解法・可変ステップ）**の**観測次数**を、非剛性の既知解問題で**収束実験（convergence study）**により確認するものです。比較対象として **固定ステップの**

* `ode1be`（後退オイラー, 1次）
* `ode2`（Heun, 2次）
* `ode3`（Bogacki–Shampine, 3次）
  も同一フレームで評価します。

> 参考スクリプト（この README が紹介するもの）
>
> * `compare_ode1be_ode2_ode3_ode23tb.m`
> * `compare_ode2_ode3_ode23tb.m`（後退オイラーなしの簡易版）

---

## 1. 目的（What this does）

* 既知解 ( y'=-y,\ y(0)=1,\ T=1 ) を用いて、
  ステップ幅 (h) を段階的に半減しながら**最終時刻誤差** (|e(h)|=|y_h(T)-e^{-T}|) を測定。
* `loglog(h, |e(h)|)` を直線近似し、**傾き ≒ 観測次数**を得る。
* `ode23tb` は可変ステップ＆誤差制御のため、**`tspan=0:h:T` + `MaxStep=h` + `RelTol≈C·h^2`** の設定で“ほぼ固定”に寄せ、次数を観測しやすくする。

---

## 2. 対象スクリプトの概要

### `compare_ode1be_ode2_ode3_ode23tb.m`

* 問題設定：( y'=-y,\ y(0)=1,\ T=1 )（真値 (e^{-1})）
* ステップ幅：(h = 1/8, 1/16, \dots, 1/1024)
* 各法で最終誤差を計算し、`loglog` で可視化・`polyfit` で傾きを推定
* 期待結果（観測次数の目安）

  * `ode1be`：≈ 1
  * `ode2`（Heun）：≈ 2
  * `ode3`（BS3）：≈ 3
  * `ode23tb`（TR-BDF2）：≈ 2

> `ode23tb` の設定例（スクリプト内）
>
> ```matlab
> C = 0.1;
> RelTol = max(1e-12, C*h^2);
> AbsTol = 1e-12;
> tspan  = 0:h:T;
> opts   = odeset('RelTol',RelTol,'AbsTol',AbsTol,'InitialStep',h,'MaxStep',h);
> [~, Ytb] = ode23tb(f, tspan, y0, opts);
> err     = abs(Ytb(end) - yT);
> ```

---

## 3. 必要環境（Requirements）

* MATLAB（R2020 以降を推奨）
* 追加ツールボックス不要

---

## 4. 使い方（How to run）

1. スクリプトファイル（`.m`）を MATLAB の作業フォルダに置く
2. コマンドウィンドウで関数を呼び出し

   ```matlab
   compare_ode1be_ode2_ode3_ode23tb
   ```
3. 生成物

   * `loglog` 誤差曲線（h 対 |error|）
   * 観測次数（傾き）のコンソール出力
   * 各 h における誤差の数表

---

## 5. 結果の読み取り（Interpreting the plot）

* `loglog` の直線部の**傾き ≒ 次数**
* h を極端に小さくすると、**丸め誤差や補間器・tol の床**で横ばいに入ることがあるため、**床に当たる前の点群**で回帰するのがコツです。

---

## 6. プロット外観の調整（例）

* 線幅（LineWidth）の指定：

  ```matlab
  loglog(hs, E2, 'o-', 'LineWidth', 1.8, 'DisplayName', 'ode2 Heun');
  ```
* 凡例位置や軸ラベルはスクリプト中の `legend`, `xlabel`, `ylabel`, `grid on` を適宜調整

---

## 7. よくある質問（FAQ / Tips）

* **`ode23tb` の誤差が横ばい（傾き 0）になる**
  → tol が厳しすぎると、内部で細分化され、誤差が **tol に支配**されます。
  `RelTol≈C·h^2`（例：`C=0.1`）に緩め、`tspan=0:h:T` と `MaxStep=h` を併用してください。
* **剛性問題でも試したい**
  → 明示法（ode2/ode3）は安定性のため極小 h を要求。`ode23tb` や `ode1be` の優位（安定）を示すには、剛性の線形問題等に切り替えて同様の枠組みで比較してください。

---

## 8. ライセンス / クレジット

* 本スクリプトは学習・検証用途を想定したサンプルです。
* 必要に応じて、社内文書やスライドへの転記・改変可。

---

## 9. 変更履歴（Change Log）

* v1.0: 初版作成（`ode1be/ode2/ode3/ode23tb` の同一プロット比較、傾き自動推定を実装）

---

ご要望があれば、**剛性版テスト関数**追加、**ログの自動保存**、**PNG/PDF 図の自動出力**にも対応した発展版を用意します。
