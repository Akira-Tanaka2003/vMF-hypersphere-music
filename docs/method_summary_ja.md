# Method Summary 日本語版：Conformer-vMF 音楽表現・生成

この文書は、GitHub 上で数式が崩れにくいように、すべての主要数式を `math` ブロックで記述した日本語版の Method Summary です。  
Google 翻訳が有効な状態では数式が崩れる場合があるため、確認時はブラウザ翻訳をオフにすることを推奨します。

## 1. 目的

本研究では、MIDI 演奏に含まれる音高、五度圏、音高遷移、拍節位置、和声中心を vMF 超球面上の方向ベクトルとして表現する。  
Conformer-vMF は、この方向表現を入力として、root、コードらしさ、コード template、拍節文脈、velocity、timing などをマルチタスクに学習し、生成時には和声中心との方向一致度を用いて旋律・伴奏・強弱を制御する。

## 2. 音高クラスと五度圏座標

MIDI 音高を `m_{t,i}` とする。  
時刻 `t` の第 `i` 音について、pitch class は次で定義する。

```math
p_{t,i}=m_{t,i}\bmod 12.
```

五度圏上のインデックスは、完全五度が 7 半音であることを用いて次のように定義する。

```math
q_{t,i}=7p_{t,i}\bmod 12.
```

音高円上の角度と五度圏上の角度は次の通りである。

```math
\theta^{pc}_{t,i}=\frac{2\pi}{12}p_{t,i},
\qquad
\theta^{5}_{t,i}=\frac{2\pi}{12}q_{t,i}.
```

それぞれの周期座標は次で与える。

```math
u^{pc}_{t,i}
=
\left(
\cos \theta^{pc}_{t,i},
\sin \theta^{pc}_{t,i}
\right).
```

```math
u^{5}_{t,i}
=
\left(
\cos \theta^{5}_{t,i},
\sin \theta^{5}_{t,i}
\right).
```

## 3. 音高遷移の表現

隣接する音符間の実音高差を次で定義する。

```math
\Delta m_{t,i}=m_{t,i}-m_{t-1,i}.
```

pitch class 上の遷移は次で定義する。

```math
\Delta p_{t,i}
=
(p_{t,i}-p_{t-1,i})\bmod 12.
```

遷移角度と遷移方向ベクトルは次の通りである。

```math
\theta^{\Delta}_{t,i}
=
\frac{2\pi}{12}\Delta p_{t,i}.
```

```math
u^{\Delta}_{t,i}
=
\left(
\cos \theta^{\Delta}_{t,i},
\sin \theta^{\Delta}_{t,i}
\right).
```

実際の跳躍量は、過度に大きな値にならないように正規化する。

```math
d_{t,i}
=
\tanh\left(
\frac{\Delta m_{t,i}}{12}
\right).
```

## 4. 拍節位置の周期表現

1 拍あたりの tick 数を `r`、1 小節の拍数を `k` とする。  
小節内位置に基づく角度を次で定義する。

```math
\theta^{bar}_{t,i}
=
2\pi
\frac{(o_{t,i}-b_t)\bmod kr}{kr}.
```

ここで、`o_{t,i}` は onset tick、`b_t` は小節開始 tick である。  
拍節位置の周期座標は次の通りである。

```math
u^{onset}_{t,i}
=
\left(
\cos \theta^{bar}_{t,i},
\sin \theta^{bar}_{t,i}
\right).
```

## 5. vMF 音符方向ベクトル

音高円座標、五度圏座標、遷移座標、実音高差、音域成分、拍節位置などを結合し、音符特徴ベクトルを作る。

```math
x_{t,i}
=
[
u^{pc}_{t,i},
u^{5}_{t,i},
u^{\Delta}_{t,i},
d_{t,i},
h(m_{t,i}),
u^{onset}_{t,i},
\phi^{beat}_{t},
\phi^{bar}_{t}
].
```

このベクトルを L2 正規化し、超球面上の単位方向ベクトルに変換する。

```math
e_{t,i}
=
\frac{x_{t,i}}
{\|x_{t,i}\|_2+\epsilon}.
```

ここで、`epsilon` は 0 除算を避けるための小さな正数である。

## 6. 和声中心方向

同時刻または近接時刻の音集合を `C_t` とする。  
各音の重みを `w_{t,i}` として、和声中心方向を次で定義する。

```math
\mu_t
=
\frac{
\sum_{i\in C_t} w_{t,i}e_{t,i}
}{
\left\|
\sum_{i\in C_t} w_{t,i}e_{t,i}
\right\|_2+\epsilon
}.
```

この `mu_t` は、その時刻における和声全体の中心方向を表す。

## 7. 方向一致度

各音符方向 `e_{t,i}` と和声中心方向 `mu_t` の一致度を内積で定義する。

```math
A_{t,i}
=
e_{t,i}^{\top}\mu_t.
```

`A_{t,i}` が大きいほど、その音は現在の和声中心に近い。  
生成時には、この値を velocity 補正や chord-tone 優先に利用する。

## 8. vMF 分布

単位超球面上の方向データに対して、vMF 分布は次で表される。

```math
p(e\mid \mu,\kappa)
=
C_D(\kappa)
\exp(
\kappa \mu^{\top}e
).
```

ここで、`mu` は平均方向、`kappa` は集中度である。  
`kappa` が大きいほど方向が `mu` の近くに集中し、`kappa` が小さいほど分布は広がる。

## 9. Conformer-vMF の学習

Conformer-vMF は、時系列入力から隠れ状態 `h_t` を得る。

```math
h_{1:T}
=
\mathrm{Conformer}
(e_{1:T},\ \mu_{1:T},\ \kappa_{1:T},\ \text{context}).
```

各 head は、root、chord-like、template、triad、seventh、beat、bar、onset、velocity、timing などを予測する。

```math
\hat{r}_t
=
\mathrm{softmax}(W_rh_t+b_r).
```

```math
\hat{y}^{cl}_t
=
\sigma(W_{cl}h_t+b_{cl}).
```

```math
\hat{c}_t
=
\mathrm{softmax}(W_ch_t+b_c).
```

```math
\hat{\mu}_t
=
\frac{W_{\mu}h_t+b_{\mu}}
{\|W_{\mu}h_t+b_{\mu}\|_2+\epsilon}.
```

```math
\hat{\kappa}_t
=
\mathrm{softplus}(W_{\kappa}h_t+b_{\kappa}).
```

## 10. マルチタスク損失

全体の学習損失は、方向回帰、集中度回帰、分類損失、連続値回帰を重み付きで足し合わせる。

```math
\mathcal{L}
=
\lambda_{\mu}\mathcal{L}_{\mu}
+
\lambda_{\kappa}\mathcal{L}_{\kappa}
+
\lambda_{root}\mathcal{L}_{root}
+
\lambda_{cl}\mathcal{L}_{cl}
+
\lambda_{temp}\mathcal{L}_{temp}
+
\lambda_{beat}\mathcal{L}_{beat}
+
\lambda_{bar}\mathcal{L}_{bar}
+
\lambda_{vel}\mathcal{L}_{vel}
+
\lambda_{time}\mathcal{L}_{time}.
```

方向損失は、予測方向と正解方向の cosine loss として定義できる。

```math
\mathcal{L}_{\mu}
=
1-\hat{\mu}_t^{\top}\mu_t.
```

root や template などの多クラス分類には Cross Entropy を用いる。

```math
\mathcal{L}_{root}
=
-\sum_{c=1}^{12}
y^{root}_{t,c}
\log \hat{y}^{root}_{t,c}.
```

chord-like や onset などの 2 値分類には Binary Cross Entropy を用いる。

```math
\mathcal{L}_{cl}
=
-
y^{cl}_{t}\log \hat{y}^{cl}_{t}
-
(1-y^{cl}_{t})
\log(1-\hat{y}^{cl}_{t}).
```

velocity や timing residual などの連続値には MAE、MSE、Smooth L1 などを用いる。

```math
\mathcal{L}_{vel}
=
\left|
v_{t,i}-\hat{v}_{t,i}
\right|.
```

## 11. P2OT prototype lens

P2OT は生成に必須の部品ではなく、Conformer 表現を prototype 確率として読むための補助的な構造レンズとして扱う。  
prototype を `z_k`、隠れ状態を `h_t` とすると、距離に基づく prototype 分布を次のように置く。

```math
\gamma_{t,k}
=
\frac{
\exp(-d(h_t,z_k)/\tau)
}{
\sum_{j=1}^{K}
\exp(-d(h_t,z_j)/\tau)
}.
```

ここで、`K` は prototype 数、`tau` は温度である。  
`gamma_t` は確率単体上の分布として解釈できる。

## 12. 生成時の pitch score

生成時には、候補音 `m` に対して、和声一致度、コード構成音らしさ、反復抑制、大跳躍抑制、方向連続抑制を組み合わせて score を作る。

```math
S(m)
=
\beta_A A(m)
+
\beta_{tone}T(m)
-
\alpha_{rep}R(m)
-
\alpha_{leap}L(m)
-
\alpha_{same}D(m).
```

ここで、`A(m)` は和声中心との一致度、`T(m)` はコード構成音らしさ、`R(m)` は同音反復ペナルティ、`L(m)` は大跳躍ペナルティ、`D(m)` は同方向連続ペナルティである。

候補音のサンプリング確率は温度付き softmax で与える。

```math
P(m)
=
\frac{
\exp(S(m)/\tau_{mel})
}{
\sum_{m'}
\exp(S(m')/\tau_{mel})
}.
```

## 13. velocity 生成

生成 velocity は、基本 velocity、和声一致度、メロディ補正、ベース補正、ランダム揺らぎを足し合わせて決定する。

```math
v_{t,i}
=
\mathrm{clip}
\left(
v_0
+
\beta_A A_{t,i}
+
\beta_{mel}M_{t,i}
+
\beta_{bass}B_{t,i}
+
\epsilon_v,
1,
127
\right).
```

ここで、`M_{t,i}` はメロディ音フラグ、`B_{t,i}` はベース音フラグ、`epsilon_v` は自然な強弱の揺らぎである。

## 14. timing と duration

発音タイミングは、グリッド位置に timing residual を加えて決定する。

```math
\tilde{o}_{t,i}
=
o^{grid}_{t,i}
+
\hat{\delta}^{time}_{t,i}
+
\epsilon^{time}_{t,i}.
```

音価は、予測 duration またはリズム template から得られる。

```math
\ell_{t,i}
=
\mathrm{clip}
\left(
\hat{\ell}_{t,i},
\ell_{min},
\ell_{max}
\right).
```

offset は次で与える。

```math
f_{t,i}
=
\tilde{o}_{t,i}
+
\ell_{t,i}.
```

## 15. 生成時の全体手順

生成時には、まず root、template、bar/beat 文脈を得る。  
次に候補音の score を計算し、melody、bass、chord part を生成する。  
最後に velocity、timing、duration を決定し、MIDI として書き出す。

```math
\text{MIDI}
=
\mathrm{Render}
(
m_{t,i},
\tilde{o}_{t,i},
f_{t,i},
v_{t,i}
).
```

## 16. 本手法の要点

Conformer-vMF では、音高やコードを単なる離散ラベルとして扱うだけでなく、五度圏、遷移、拍節、和声中心を方向として統合する。  
これにより、各音が現在の和声中心にどれだけ近いかを `A_{t,i}` として定量化できる。  
生成時には、この方向一致度を用いて、和声的に自然な音を強め、大跳躍や同音反復を抑えた旋律・伴奏生成を行う。
