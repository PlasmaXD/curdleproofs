# GrandProduct 論証

**GrandProduct 論証**は以下の関係を証明します：

$$
R_{gprod} =
\left\{
\begin{array}{l l|l}
	(B, p); & (\bm{b}, \bm{r}_B)
	&
	B =  \bm{b} \times \bm{g} + \bm{r}_B \times \bm{h}  \\
	& & p = \displaystyle\prod_{i=1}^{\ell} b_i
\end{array}
\right\}
$$

この構成では、[離散対数内積論証](crate::inner_product_argument)をサブプロトコルとして利用します。

## プロトコル

以下の図は、*gprod* 論証の形式的な記述を示しています。

<center>
<img width="80%" src="https://github.com/asn-d6/curdleproofs/raw/main/doc/images/gprod_prover.png"></img>
</center>

## 非公式概要

証明者（Prover）は入力として $B, p$ を受け取り、次を証明しようとします：

* $B = \bm{b} \times \bm{g}$ がベクトル $\bm{b}$ へのコミットメントであること
* $p = \prod_{i=1}^\ell b_i$ が $\bm{b}$ の総積（grandproduct）であること

高レベルでは、この関係を内積論証として表現します。手順は次のとおりです：

1. **分割（Separate）**：総積を複数の単一積の等式に分ける。
2. **圧縮（Compress）**：すべての等式を一つの多項式にまとめる。
3. **並び替え（Rearrange）**：多項式を内積の等式に組み替える。
4. **コンパイル（Compile）**：内積論証の入力となる各ベクトルへのコミットメントを取得して証明システムを構築する。

下図は、総積論証が内積論証へとコンパイルされる様子を示しています。

<center>
<img width="60%" src="https://github.com/asn-d6/curdleproofs/raw/main/doc/images/gprod_overview.png"></center>

### 分割（Separate）

総積

$$
p = \prod_{i=1}^\ell b_i
$$

は $\ell-1$ 回の掛け算で構成されます。最初にこれを次のように $\ell+1$ 個の乗算チェックへ**分割**します：

$$
c_{1} = 1,\quad
c_{i+1} = b_i\,c_i \quad (i \in [1,\ell)),\quad
p = b_\ell\,c_\ell
$$

これによりベクトル $\bm{c}$ が定義され、最後のチェックで $p = \prod_{i=1}^\ell b_i$ が保証されます。

### 圧縮（Compress）

各乗算チェックを確かめるため、次の一つの多項式等式に**圧縮**します：

$$
0 = (1 - c_1)
  + (b_1 c_1 - c_2)X
  + (b_2 c_2 - c_3)X^2
  + \cdots
  + (b_{\ell-1}c_{\ell-1} - c_\ell)X^{\ell-1}
  + (b_\ell c_\ell - p)X^\ell
$$

これはもとの $\ell+1$ 個の制約を、変元 $X$ における係数チェックとしてまとめたものです。

### 並び替え（Rearrange）

この多項式等式を、内積形式で表現できるよう**並び替え**ます。まず、

$$
pX^\ell - 1
= \sum_{i=1}^\ell c_i\,(X^i b_i - X^{i-1})
$$

と書き直せることがわかります。

### コンパイル（Compile）

Schwartz–Zippel の補題により、ランダムな点 $\beta$ で

$$
p\,\beta^\ell - 1
= \sum_{i=1}^\ell c_i\,(\beta^i b_i - \beta^{i-1})
$$

が成り立つなら、元の内積等式は高い確率で正しいと判断できます。すなわち、

$$
z = \bm{c} \cdot \bm{d},
\quad
z = p\,\beta^\ell - 1,\;\;
d_i = \beta^i b_i - \beta^{i-1}.
$$

ここで、$\bm{c}$ と $\bm{d}$ へのコミットメントが必要です。

1. 証明者はまず

   $$
   C = \bm{c} \times \bm{g}
   $$

   を提供し、これをハッシュしてチャレンジ値 $\beta$ を得ます。

2. 次に、既知のコミットメント

   $$
   B = \bm{b} \times \bm{g}
   $$

   をリスケールし、新たなキー $\bm{g}'$ の下で

   $$
   B = \bm{b}' \times \bm{g}',
   \quad
   \bm{b}' = (\beta^1 b_1,\ldots,\beta^\ell b_\ell),
   \;\;
   \bm{g}' = (\beta^{-1}g_1,\ldots,\beta^{-\ell}g_\ell).
   $$

3. ベクトル $\bm{d}$ は

   $$
   \bm{d} = \bm{b}' - (1,\beta,\ldots,\beta^{\ell-1})
   $$

   と表せるので、コミットメント

   $$
   D = B - \sum_{i=1}^\ell \beta^{i-1} g_i'
   $$

   を計算し、これが $\bm{d}$ へのコミットメントとなります。

最後に、証明者は離散対数内積論証を実行し、次の関係を主張します：

$$
C = \bm{c} \times \bm{g},\quad
D = \bm{d} \times \bm{g}',\quad
p\,\beta^\ell - 1 = \bm{c} \cdot \bm{d}.
$$

この過程には追加のブラインディング値が含まれ、ゼロ知識性が確保されます。

---

## 完全ゼロ知識な GrandProduct の構成

非公式概要に加えて、以下のステップでゼロ知識性を保ちます。

### ステップ1

インスタンス全体をハッシュしてランダム値 $\alpha$ を得ます。これにより、たとえ $\bm{r}_B=0$ の場合でも、次のステップでのブラインダーを隠すことが可能になります。

### ステップ2

ベクトル $\bm{c}$ をコミットする際、証明者はランダムなブラインダー $\bm{r}_C\in\mathbb{F}^{n_{bl}}$ を選びます。最終的な内積論証では、この $\bm{r}_C$ に対応するフィールド要素

$$
r_p = (\bm{r}_B + \alpha\,\bm{1}) \cdot \bm{r}_C
$$

を提供し、内積中のブラインダー寄与を打ち消します。

### ステップ3

コミットメントキーのブラインド用部分を

$$
\bm{h}' = \beta^{-(\ell+1)}\,\bm{h}
$$

として用意し、さらに

$$
\bm{r}_D = \beta^{\ell+1} (\bm{r}_B + \alpha\,\bm{1})
$$

を計算して

$$
D = \bm{d}\times\bm{g}' + \bm{r}_D\times\bm{h}'
$$

とすることで、$\bm{d}$ へのコミットメントを得ます。

### ステップ4

最終ステップでは、拡張コミットメントキーを

$$
\bm{G}=(\bm{g}\,\|\,\bm{h}),\quad
\bm{G}'=(\bm{g}'\,\|\,\bm{h}')
$$

とし、拡張ベクトル $(\bm{c}\,\|\,\bm{r}_C)$ と $(\bm{d}\,\|\,\bm{r}_D)$ の内積として

$$
z = p\,\beta^\ell + r_p\,\beta^{\ell+1} - 1
$$

$$
= (\bm{c}\,\|\,\bm{r}_C)\cdot(\bm{d}\,\|\,\bm{r}_D)
$$

を計算します。これにより、完全なゼロ知識性を維持しつつ総積の正当性を証明できます。
