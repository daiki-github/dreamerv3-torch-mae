# 視覚基盤モデルを用いた世界モデルの学習効率化

# TL; DR
 - ImageNetで事前訓練したMAEを用いて，DreamerV3において再構成を伴わない世界モデルの学習を試みる．
 - DeepMind Control Walkerにおいて検証し，通常のDreamerV3と比較して顕著な性能低下が見られた．

Code: https://github.com/daiki-github/dreamerv3-torch-mae

# Introduction
 - [DreamerV3](https://arxiv.org/abs/2301.04104) や[TD-MPC2](https://arxiv.org/abs/2310.16828) をはじめとする世界モデルでは，観測の再構成を伴っている．こうした観測の再構成は，世界モデルにタスクに無関係な特徴やピクセル単位の細部特徴の学習を強制し，サンプル効率の低下につながる．また，観測エンコーダの学習はエージェントの方策に条件づけられており，未知環境における汎化性能が低下する可能性がある．<br>
 - [SupPVR](https://openreview.net/forum?id=LvAy07mCxU)では[CLIP](https://arxiv.org/abs/2103.00020)や[DINO](https://arxiv.org/abs/2104.14294)をはじめとした事前訓練した視覚基盤モデルの表現（Pre-trained Visual Representation, **PVR**）を用いて観測エンコーダを置き換える試みが検証されている．この研究では，PVRがDreamerV3やTD-MPC2のサンプル効率をむしろ悪化させることが報告されている．その要因として，観測エンコーダと比較して，視覚基盤モデルの表現空間内の報酬構造がentangledしていることが指摘されている．<br>
 - 一方で，[SupPVR](https://openreview.net/forum?id=LvAy07mCxU)で検証されているPVRは，画像の大域的な特徴抽出に優れたものが多い．これは，エージェントの局所的な姿勢や物体の位置関係などの密な予測が必要となる場合には適切でない．そこで本課題では，DreamerV3の訓練において，[MAE](https://arxiv.org/abs/2111.06377)を用いたPVRの有効性を検証する．MAEは入力画像の非マスク領域からマスク領域を予測する自己教師あり学習手法であり，DINOやCLIPと比較して物体検出やセグメンテーションなどの密な予測タスクにおいて優れた性能を達成している．

# Methods & Experiments

![](https://user-images.githubusercontent.com/2111293/217355673-4abc0ce5-1a4b-4366-a08d-64754289d659.png)<br>
 上図に示すように，本課題では自己符号化器によって構成されるDreamerV3の観測モデルを, PVRに置き換える．本課題を通して，PVRとしてMAE (ViT-B/16)を用いる．具体的には，MAE Encoderの特徴表現に対して，線形層を追加し観測エンコーダを置き換える．世界モデルの訓練時にはMAEの重みは凍結し，線形層のみ訓練する．<br><br>

 主な実験設定を以下に示す．<br>
 - タスク
   - DeepMind Controll, Walker (Walk)
 - 環境ステップ数：1M steps
 - 画像サイズ: (64, 64) for original DreamerV3, (224, 224) for PVR
 - GPUs: 1x RTX 5090

# Experiments
 以下に，訓練時の収益値の遷移を示す．<br>
![](https://scrapbox.io/files/68d9088ebb02ebc56bbc0c9f.png)<br>
 - 青：Original DreamerV3<br>
 - 黒：DreamerV3 with MAE (rep_scale=0.1)<br>
 - 赤：DreamerV3 with MAE (rep_scale=0)<br>

# References
 - https://github.com/NM512/dreamerv3-torch
 - https://huggingface.co/docs/transformers/model_doc/vit_mae




