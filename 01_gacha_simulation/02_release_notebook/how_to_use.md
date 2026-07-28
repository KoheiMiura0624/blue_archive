# Blue Archive Gacha Simulator

ブルーアーカイブのガチャをモンテカルロシミュレーションするNotebookです。

## 主な用途

- 新旧ガチャシステムの比較
- ガチャ仕様変更時の期待値計算
- PU切替タイミングの検証
- 独自ガチャルールのシミュレーション

---

# クイックスタート

## 変更する項目

通常は以下の4項目のみ変更すれば利用できます。

1. `config`
2. `stop_condition`
3. `switch_condition`
4. `n_trials`

---

# 実行手順

## ① Configを設定

ガチャ仕様を設定します。

### 新ガチャ

```python
config = Config(
    system=GachaSystem.NEW,
    pickup_rate=0.007,
    star3_rate=0.03,
)
```

### 旧ガチャ

```python
config = Config(
    system=GachaSystem.OLD,
    pickup_rate=0.007,
    star3_rate=0.03,
)
```

> Configで省略した項目はデフォルト値が使用されます。

---

## ② ガチャ終了条件を設定

### 両PUを入手したら終了

```python
stop_condition = stop_when_both_obtained
```

### PU1を1人入手したら終了

```python
stop_condition = lambda state: state.pu1 >= 1
```

### 200連で終了

```python
stop_condition = lambda state: state.pulls >= 200
```

---

## ③ PU切替条件を設定

### PU1を入手したらPU2へ切替

```python
switch_condition = switch_when_pu1_obtained
```

### 切替なし

```python
switch_condition = lambda state: False
```

### 100連後に切替

```python
switch_condition = lambda state: state.pulls >= 100
```

---

## ④ シミュレーション回数を設定

|試行回数|用途|
|---:|---|
|10,000|動作確認|
|100,000|通常利用（推奨）|
|1,000,000|高精度比較|

```python
n_trials = 100000
```

---

## ⑤ Notebookを上から実行

実行後は以下の順番で結果を確認します。

```python
results = simulate(...)
```

↓

```python
summarize_results(results)
```

↓

```python
plot_pull_distribution(
    pull_counts_new,
    pull_counts_old,
)
```

---

# 出力結果

`summarize_results(results)` を実行すると、以下の項目を表示します。

|項目|説明|
|---|---|
|PU1入手率|PU1を1人以上入手した割合|
|PU2入手率|PU2を1人以上入手した割合|
|両方入手率|PU1・PU2を両方入手した割合|
|平均PU1人数|PU1の平均入手人数|
|平均PU2人数|PU2の平均入手人数|
|平均★3人数|★3生徒の平均入手人数|
|平均連数|ガチャを引いた平均回数|
|平均PU1募集連数|PU1募集で引いた平均連数|
|平均PU2募集連数|PU2募集で引いた平均連数|
|平均残チャージ数|終了時点の平均チャージ数|
|平均青輝石コスト|消費した青輝石の平均|
|平均実質ガチャ数|特典分を差し引いた実質ガチャ数の平均|

### 出力例

```
PU1入手率:          100.0000%
PU2入手率:          100.0000%
両方入手率:         100.0000%

平均PU1人数:        1.0410
平均PU2人数:        1.0274
平均★3人数:         6.7167

平均連数:           183.8283
平均PU1募集連数:    92.9936
平均PU2募集連数:    90.8347

平均残チャージ数:   3.8019

平均青輝石コスト:   18291.0000
平均実質ガチャ数:   152.4250
```

---

`plot_distribution()` を実行すると以下を表示します。

- ガチャ数分布
- 累積確率

---

# よく使う設定例

## 両PUを揃えるまで引く

```python
stop_condition = stop_when_both_obtained
switch_condition = switch_when_pu1_obtained
```

---

## PU1を3人引く

```python
stop_condition = lambda state: state.pu1 >= 3
```

---

## 300連まで引く

```python
stop_condition = lambda state: state.pulls >= 300
```

---

## 100連後にPUを切り替える

```python
switch_condition = lambda state: state.pulls >= 100
```

---

# Configリファレンス

|項目|説明|
|---|---|
|system|新ガチャ / 旧ガチャ|
|pickup_rate|PU排出率|
|other_pickup_rate|すり抜けPU排出率|
|star3_rate|★3排出率|
|batch_size|1回の募集数（単発・10連など）|
|initial_charge|初期チャージ数|
|charge_100_target_rate|100チャージ時のPU排出率|
|charge_100_other_pickup_rate|100チャージ時のすり抜けPU排出率|
|old_ceiling_points|旧ガチャの交換ポイント|

---

# カスタマイズ例

## ★3を10人入手したら終了

```python
stop_condition = lambda state: state.star3 >= 10
```

---

## 150連後にPUを切り替える

```python
switch_condition = lambda state: state.pulls >= 150
```

---

## 初期チャージを変更

```python
config = Config(
    initial_charge=80,
)
```

---

## PU率を変更

```python
config = Config(
    pickup_rate=0.01,
)
```

---

# 注意事項

- モンテカルロシミュレーションのため、試行回数が少ない場合は結果がばらつきます。
- 期待値比較には100,000回以上の試行を推奨します。
- Configで省略した項目にはデフォルト値が使用されます。