# LGL
LGLはCVPR2019に発表されたHao Chengらの論文 [Local to Global Learning: Gradually Adding Classes for Training Deep Neural Networks] にて提案された、Deep Neural Network(DNN)における新たな学習パラダイムである。 <br>

論文URL: https://openaccess.thecvf.com/content_CVPR_2019/papers/Cheng_Local_to_Global_Learning_Gradually_Adding_Classes_for_Training_Deep_CVPR_2019_paper.pdf

## 論文について
### 直観的な話
私たちはものを学習することにおいて、全てを覚えるのではなく、まず単純なものから学びそこから様々なものを覚えていく。
(例えば、子供が犬という言葉と姿を知り、これも犬？と聞きながら猫を覚えていく。) <br>
この直感に基づいて「簡単なサンプルから学習を始め、徐々に学習するサンプルを複雑にしていく」学習手法、Curriculum Learning(CL)が2009年にBengioらによって提案された。
(参考：Evaluating capability of deep neural networks for image classification via information plane. )

### SPL
Self-Paced Learning(SPL)は、上記のCLを基に学習カリキュラムを自動的に生成して学習を行う手法で、この論文において比較対象とされる手法である。<br>
CL法では学習する「簡単なサンプル」から「複雑なサンプル」まで、手動で設定する必要があるが、SPL法では損失関数の値に基づいて各トレーニングイテレーションで「簡単なサンプル」を抽出して、それを学習する。
しかし、この抽出において新たなハイパーパラメータが必要となり、そのチューニングの手間がかかるという問題点がある。

### LGL
SPLの問題やCLの手動設定の問題を解決するために、LGLでは以下のように「簡単なサンプル」から「複雑なサンプル」までを学習するカリキュラムを提案した。<br>
1. 全サンプルから一部サンプルをランダム抽出して「簡単なサンプル」として学習する
1. その学習サンプルの学習を一定行ったあと、残ったサンプルから一定数のサンプルを抽出して「簡単なサンプル」に加える
1. 2を繰り返して最終的には全サンプルを学習する（「複雑なサンプル」の学習)
<br>
Haoらはこの手法を一般物体認識問題に適用させた学習を行い、その評価を行った。<br>
具体的には以下の手順である: <br>

1. 全サンプルをラベルごとに分類し、いくつかのラベルの学習データを「簡単なサンプル」にする
1. 残ったラベルの学習データからいくつかのラベルを抽出し、その学習データを「簡単なサンプル」に追加し、ネットワークの重みそのままで学習を始める
1. 最終的に全データをトレーニングデータとした学習を行う

この次のラベルの抽出において<br>
ランダム抽出・学習データのラベルに”似ている”ラベルを抽出・学習データのラベルに”似ていない”ラベルを抽出<br>
という３種類の抽出手法を挙げられたが、性能において大きな差は見られなかったが、どれも「複雑な」学習データ(ラベルの種類が多いetc)において、従来の学習モデル(VGG-16)を用いてより高い性能を出した。

### 論文の新規性・重要性
本論文では新しい学習カリキュラムの提案として、転移学習のような学習データを変更し拡張していく手法を提案した。(適切な重みで次の学習を初期化できる)<br>
LGL法は従来のCL法やSPL法にあげられるような手法とは異なり、新しいハイパーパラメータの導入やチューニング・データの手動処理の必要がなく、また学習の性能向上および収束も安定化させた。<br>
これらを数値的な明示だけではなく、情報理論的な観点から学習の安定性を議論していたところが CVPR通過のひと推しにつながったと考えられる。

## 本リポジトリの実装内容について
本リポジトリではGoogle Colaboratoryにおける実装を行った。
具体的には、論文の内容を元に同じような構造で以下の追実装を行った。
- VGG-16の実装
- CIFAR100を用いた学習
- ハイパーパラメータは基本論文と同じ
- ラベル抽出アルゴリズムには性能に大きな差がないためランダム抽出を選択した

ハイパーパラメータについて、各学習でのepoch数は論文では100とされているが、時間の都合上20で実装を行った。

### 改善点・改良点
- 過学習がいくらか見られ、初期値によっては損失関数が増加し学習が収束しないことがある。
    - Data Augumentationやハイパーパラメータの調整
        - epoch数の増加が影響するか 
    - 他の抽出アルゴリズムの実装
- 他の学習モデル(ResNetなど)の利用
