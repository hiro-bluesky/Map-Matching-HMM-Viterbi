## 概要
以下の4種類のマップマッチングを行う．

| |  最寄り割り当てアルゴリズム  |  HMM + Viterbiアルゴリズム  |
| ----| ---- | ---- |
|ノード（交差点）割り当て|  ①nearest-node matching |  ③nearest-node HMM matching  |
|エッジ（道路リンク）割り当て|  ②nearest-edge matching  |  ④nearest-edge HMM matching  |

最寄り割り当ては，各GPS測位点を最も近いノード（エッジ）に割りあてるアルゴリズムで，非常にシンプルなため計算量が低いのが特徴です．大規模データに対しても簡単に適用することができます．その反面，一つの誤差に弱いというデメリットがあります．

そこで，GPS測位点の「**時系列を考慮した**」ものがHMMによるマップマッチングです．**HMM**（隠れマルコフモデル）とは，

「真の状態（移動者の実際の位置）がわからないもとで，観測系列（移動者のGPSデータ）から状態系列を推定する」

手法です．各GPS測位点の割り当て先を**近傍**$\mathbf{K}$**個**まで緩和し，その中で，尤もらしい経路を**動的計画法**（Viterbiアルゴリズム）で求めます．

## ①Nearest-Node Matching 
各GPS測位点を最寄りのノード（交差点）に割り当てるマッチングアルゴリズム．``ox.distance.nearest_nodes``を用いて最近傍ノード探索を行います．

<div style="border:2px solid black; padding:10px; border-radius:8px;">
<b>交差点の定義：</b>  
osmnxで取得した道路ネットワークの中で，<b>次数3以上</b>のノード．
</div>

## ②Nearest-Edge Matching
各GPS測位点を最寄りのエッジ（道路リンク）に割り当てるマッチングアルゴリズム．``ox.distance.nearest_edges``を用いて最近傍エッジ探索を行います．

## ③Nearest-Node HMM Matching
近傍$K$個のノードをマッチング候補として，HMM（隠れマルコフモデル）を用いて最尤経路を求めます．

## ④Nearest-Edge HMM Matching
近傍$K$個のエッジをマッチング候補として，HMM（隠れマルコフモデル）を用いて最尤経路を求めます．

# HMM + Viterbi Map Matching 
## モデルの構造
- **隠れ系列**: $O=(o_1,o_2,...,o_T)$ = 「GPS座標系列」
- **隠れ状態系列**: $S = (s_1,s_2,...,s_T)$ = 「実際の座標の系列」（真の座標はわからないので，“隠れ”状態）
- **観測確率**: P(o_t|s_t) = 「GPS座標が“そのノード（エッジ）に観測される”確率」
- **遷移確率**: 「道路ネットワークに沿って状態がどのように遷移するか」

最終的には，$S^* = \arg \max_S P(S|O)$，すなわち「**与えられたGPS系列を最もよく説明する状態列**」を求めたい．

## HMM遷移確率関数
<img width="336" height="405" alt="image" src="https://github.com/user-attachments/assets/59c7af1d-c7a6-45d0-a5e7-5b8e730dbad8" />

<img width="271" height="307" alt="image" src="https://github.com/user-attachments/assets/d249db76-f441-4182-9486-651dbd3a811d" />

## Viterbi アルゴリズム
HMMに基づいて，最尤経路を**動的計画法**で求める．


- **初期化** $\delta_0(k) = \log P_{\text{emis}}(p_0|n_{0,k})$

- **再帰ステップ**  $\delta_t(j) = \max_i \left[ \delta_{t-1}(i) + \log P_{\text{trans}}(n_{t,j}|n_{t-1,j}) \right] + \log P_{\text{emis}}(p_t|n_{t,j})$

- **対応するバックポインタ**  $\psi_t(j) = \arg \max_i \left[ \delta_{t-1}(i) + \log P_{\text{trans}}(n_{t,j}|n_{t-1,j}) \right]$
- **最尤系列**  

<p align="center">
  <img src="https://latex.codecogs.com/svg.latex?n_t^*%20=%20\psi_{t+1}(n_{t+1}^*),%20\quad%20n_T^*%20=%20\arg\max_j%20\delta_T(j)" alt="formula"/>
</p>


本手法は **隠れマルコフモデル (HMM)** を用いたマップマッチングであり、GPS 観測点列から最も尤もらしい道路上の経路を復元する。  
観測は GPS 点列、隠れ状態は道路ネットワーク上のノード（またはエッジの位置）として定式化される。


### 記号の意味
- $\delta_t(j)$: 時刻 $t$ における候補ノード $j$ の最尤スコア（対数尤度）  
- $P_{\text{emis}}(p_t|n_{t,j})$: GPS 観測 $p_t$ が候補ノード $n_{t,j}$ に対応する確率（位置誤差や方位の整合性を考慮）  
- $P_{\text{trans}}(n_{t,j}|n_{t-1,i})$: 時刻 $t-1$ の候補 $i$ から時刻 $t$ の候補 $j$ への遷移確率（道路ネットワーク距離や速度整合性を考慮）  
- $\psi_t(j)$: バックポインタ．最適経路を復元するためのポインタを保持  

---