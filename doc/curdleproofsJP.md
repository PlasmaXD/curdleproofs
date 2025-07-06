以下、日本語訳です。

# Curdleproofs のシャッフル証明

Curdleproofs の目的は、群要素のシャッフル（並べ替え）をゼロ知識で証明する議論（argument）を構築することです。具体的には、公開された群要素の並び

$$
\bm{R} = (R_1,\dots,R_\ell)\quad\text{と}\quad
\bm{S} = (S_1,\dots,S_\ell)
$$

に対して、シャッフラー（証明者）は新たに群要素の並び

$$
\bm{T} = (T_1,\dots,T_\ell)\quad\text{と}\quad
\bm{U} = (U_1,\dots,U_\ell)
$$

を計算し、次の条件をゼロ知識で証明します。すなわち、ある順列

$$
\sigma: \{1,\dots,\ell\}\to\{1,\dots,\ell\}
$$

と体元 $k\in F$ が存在して、

$$
T_i = k\,R_{\sigma(i)},\quad U_i = k\,S_{\sigma(i)}
$$

がすべての $i$ で成り立つことを示します。さらに、この順列 $\sigma$ は、群要素 $M\in G$ に対して何らかの乱数ベクトル $\bm{r}_M\in F^{n_{bl}}$ を用いて

$$
M = \sigma(1,2,\dots,\ell)\times\bm{g} \;+\;\bm{r}_M\times\bm{h}
$$

とコミットされています。

なお、$\ell$-DDH（分散離散対数）仮定の下では、シャッフル後の暗号文（$\bm{T},\bm{U}$）は真のランダム値と見分けが難しいとされています。

以上をまとめると、以下の関係に対するゼロ知識証明を定義していることになります：

$$
R_{\mathrm{shuffle}}
=
\left\{
(\bm{R},\bm{S},\bm{T},\bm{U},M)\;\Big|\;
\exists\,\sigma,\;k,\;\bm{r}_M\;\text{s.t.}\;
\begin{aligned}
&\bm{T} = \sigma(k\,\bm{R}),\\
&\bm{U} = \sigma(k\,\bm{S}),\\
&M = \sigma(1,\dots,\ell)\times\bm{g} + \bm{r}_M\times\bm{h}
\end{aligned}
\right\}.
$$

## サブ証明（Subarguments）

Curdleproofs では目的達成のために複数のサブ証明を組み合わせています。その証明フローを図示すると以下のようになります。

<center>
<img width="60%" src="https://github.com/asn-d6/curdleproofs/raw/main/doc/images/curdleproofs_overview.png"></img>
</center>

## プロトコル

*curdleproofs* 議論の形式的な手順は次の図のとおりです。

<center>
<img width="100%" src="https://github.com/asn-d6/curdleproofs/raw/main/doc/images/curdleproofs_prover.png"></img>
</center>

## 非公式なプロトコル概要

$\ell>1$ とします。証明者は公開情報 $\bm{R},\bm{S},\bm{T},\bm{U},M$ を入力として受け取り、順列 $\sigma$ とスカラー $k$ を知っていることを証明します。

1. **コミットメントの検証**
   $M$ は順列 $\sigma$ に対するコミットメントとして

   $$
   M = \sigma(1,2,\dots,\ell)\times\bm{g}
   $$

   と作られています。

2. **シャッフルの確かめ**
   $\bm{T} = \sigma(k\bm{R})$ および $\bm{U} = \sigma(k\bm{S})$ となっていることを示します。

まず、すべての公開入力をハッシュしてチャレンジベクトル $\bm{a}\in F^\ell$ を得ます。証明者はこれを使って

$$
A = \sigma(\bm{a}) \times \bm{g},\quad
T = \bm{a} \cdot (k\bm{R}),\quad
U = \bm{a} \cdot (k\bm{S})
$$

を計算し、検証者に送信します。

さらに、以下の３つの追加関係のゼロ知識証明アルゴリズムを用います。

* **SamePerm 議論** で、$A$ が $M$ にコミットされた順列 $\sigma$ に対して、$\sigma(\bm{a})$ のコミットメントであることを証明
* **SameMultiScalar 議論** で、あるベクトル $\bm{x}$ が存在し、

  $$
  A = \bm{x}\times\bm{g},\quad
  T = \bm{x}\times\bm{T},\quad
  U = \bm{x}\times\bm{U}
  $$

  が成り立つことを示す
* **SameScalar 議論** で、同じスカラー $k$ によって $T = k(\bm{a}\cdot\bm{R})$ および $U = k(\bm{a}\cdot\bm{S})$ が成り立つことを示す

これらを組み合わせることで、ランダムな $\bm{a}$ の下で $T_i = k\,R_{\sigma(i)}$（ほぼ確実に）および $U_i = k\,S_{\sigma(i)}$ が導かれます。

本来はゼロ知識性を保証するためのマスキング値も含まれますが、概要では便宜上省略しています。

## 完全ゼロ知識構成

以下では、ゼロ知識性を得るための追加手順を示します。

##### ステップ1

証明者と検証者は公開情報をハッシュして共通のランダムベクトル $\bm{a}\in F^\ell$ を決定します。秘密は関与せず、検証者はすべての入力が群や体の元であることをチェックします。

##### ステップ2

証明者は $\sigma(\bm{a})$ に対してコミットメント

$$
A = \sigma(\bm{a})\times\bm{g} + \bm{r}_A\times\bm{h}
$$

を生成します。ここで $\bm{r}_A\in F^{n_{bl}-2}$ はランダムなブラインダーです。Zero-knowledge 性を保つには $|\bm{r}_A|\ge2$ が必要なので、$n_{bl}\ge4$ とします。証明者はこれとともに SamePerm 議論 $\pi_{\mathrm{sameperm}}$ を検証者に送ります。

##### ステップ3

証明者はまず

$$
R = \bm{a}\cdot\bm{R},\quad S = \bm{a}\cdot\bm{S}
$$

を計算し、検証者が正しく計算されたことを確認します。その後、

$$
\mathit{com}_T = (r_TG_T,\;T + r_TH),\quad
\mathit{com}_U = (r_UG_U,\;U + r_UH)
$$

を生成し（$r_T,r_U$ はマスキング値）、SameScalar 議論 $\pi_{\mathrm{samescalar}}$ を付して検証者に送ります。

##### ステップ4

最後に証明者と検証者は

$$
A' = A + r_TG_T + r_UG_U
$$

と基底ベクトル $\bm{G}$ を拡張し、ブラインダーを含めたコミットメントとして扱います。同様に $\bm{T}',\bm{U}'$ を拡張した上で SameMultiScalar 議論 $\pi_{\mathrm{samemultiscalar}}$ を行い、最終的な証明

$$
\pi_{\mathrm{shuffle}}
= (A,\;\mathit{com}_T,\;\mathit{com}_U,\;R,\;S,\;\pi_{\mathrm{sameperm}},\;\pi_{\mathrm{samescalar}},\;\pi_{\mathrm{samemultiscalar}})
$$

を構成します。検証者はすべてのチェックが通れば受理します。

## 悪意あるランダマイザー攻撃

証明者がランダマイザー $k=0$ を選ぶと、出力暗号文がすべて無限遠点（消失）になりつつ証明が通ってしまう危険があります。これを防ぐため、検証者は少なくとも最初の暗号文が無限遠点でないことを確認する必要があります。
