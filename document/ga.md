# 遺伝的アルゴリズムとそのJulia実装

遺伝的アルゴリズムをJuliaで実装するべく改めてまとめた.

## 遺伝的アルゴリズムの概要
遺伝的アルゴリズムは数理最適化の一種で,生物の遺伝をイメージとしたものである.

最初に大量の生物を用意し,子供を作らせ、時間がたったあとに生き残った生物が一番よい遺伝子を持っているだろうという期待に基づくものである.

理論的な精度保証はあまり知られていない。(少なくとも私は知らない)が、実際に使ってみると精度が高いことが多く、よく使われる手法の一つである。

## 用語
生物の進化に基づくこともあり、それっぽい用語が出てくるので、最初にそれを説明しておきます

- 遺伝子(gene) : 決定変数の一つ.$X + Y$が目的関数の場合,$X$や$Y$のこと
- 個体(individual) : 決定変数の組
- 個体集合(population) : 個体を集めたセット。現世代(population)から子孫(offspring)を作ります。
- 世代(generation) : 子孫を作る操作回数
- 適応度(fitness) : 個体に対する目的関数の値。
- 選択(selection) : 現世代から次世代の生成に向けた淘汰のこと。適応度の高いものを優先的に選択する、(ただし、選び方は多数ある)
- 交叉(crossover) : 2個体間の遺伝子の入れ替えのこと。生物が交配によって子孫を残すことをモデル化したもの
- 突然変異(mutation) : 個体の遺伝子をランダムに変化させること
- エリート保存(elitsism):　優秀な子孫は次世代も必ず保存して残すこと.

## アルゴリズム

1. 初期の個体集合を生成
1. 選択
1. 交差
1. 突然変異
1. エリート保存
1. 選択からエリート保存を一定回数ループ

がアルゴリズムのメインの処理である.

遺伝的アルゴリズムでは,選択,交差,突然変異等を具体的に何を選ぶかを考える必要がある.

代表的なものは存在するが、問題ドリブンでその状況に応じた関数を選ぶべきこともある.
特に問題にある制約条件を満たすためには、交差や突然変異を自分で作る部分が生まれる。

また、現実の問題を解く場合は、そもそもモデリングを変更する余地があるため,シミュレーテッドアニーリングよりもモデリングが難しいとされる。


### 選択(select)
- input: array of individual
- output: array of individual

ただし、outputを減らすとindividualが減っていくので、inputとoutputの配列の長さは基本一致する。
(エリート選択や必要なミニマムは保てれば減らすといったアルゴリズムもありえるので、あくまで基本である)

驚いたのは重要なのはoutputの要素には優秀な遺伝子を持つindividualが複数回選ばれることである。

__余談__

individualを人間とイメージすると、ある人間が選択を経て、分身したことになり、違和感があった。
人間というよりは種族(哺乳類、霊長類)ぐらいのイメージでいると自然かもしれない。

#### ルーレット選択
- 個人のfitnessに応じて選ばれる確率を変え、ランダムに選ぶ方法。

[deap](https://github.com/DEAP/deap/blob/master/deap/tools/selection.py#L94)を見ると,一様乱数をかけ、その値を超えるものを選ぶよう.

他にも以下のような方法が有名.
- 期待値選択
- ランキング
- トーナメント方式

子供を選ぶ方法.
selectに従って, `individual`が出会い,子供を作る.

### 交差
- input: array of individual
- output: array of individual

#### 一点交差
2つのindividualを`x1`,`x2`をランダムで選ぶ
$i$を1からindividualの配列の長さからランダムで選び,
```python
y1 = x1[:i] + x2[i:]
y2 = x2[:i] + x1[i:]
```
とする方法.


### mating
- input: individual
- output individual

遺伝子を少しいじる方法.
http://www.sist.ac.jp/~kanakubo/research/evolutionary_computing/ga_operators.html
をみると

> 実装上は、各数字ごとに乱数を振り、例えば5％以下の確率で、突然変異の操作を行なう。二箇所必要な方法では、何らかのランダムな方法で箇所を決める。
他に、或る遺伝子の一部を他の遺伝子に割り込ませる（交叉でもある）挿入、或る遺伝子の一部が無くなる欠失もある（これらは遺伝子長が変わる）。:


とある。実際にどういうmatingをするべきかはあまりわからないが、二値の場合はランダムで選んだ決定変数の0,1を置き換える事が多い

[deap](https://github.com/DEAP/deap/blob/master/deap/tools/mutation.py)をみると、gaussian noiseを加えたりもしている。

## 設計
実際にJuliaでの実装を考える.
`sklearn`に近いかたちとする.

- `fit(model, X ,y)`:　数理最適化に`fit`はないので、そのままmodelを返す.
- `precit(model, X)`: メイン処理

### ファイルの概要(未実装含む)
`select.jl`:selectのメソッドを実装
`mutate.jl`:mutateのメソッドを実装
`mating.jl`:matingのメソッドを実装
`elite.jl`:eliteのメソッドを実装
`ga.jl`:遺伝的アルゴリズムのメインのループ処理を実装
`individual.jl`:決定変数の関する実装,pkg側にはいれるのは難しい.
`customizefunc.jl`:問題ごとのカスタマイズした関数等の実装、余力があれば
`script.jl`:実際に人が実行するスクリプト


### 人間がいじるもの
- gafunction
  - どのmutation,mating,elightsim,selectを選ぶか
- gaparatemr
  - 計算回数
  - mutation rate, mating rate
- 遺伝子(決系変数)
  - 遺伝子の制約条件
  - 遺伝子の評価関数
- populationのサイズ