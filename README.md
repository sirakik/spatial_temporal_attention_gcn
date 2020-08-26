# データセットの準備

### ダウンロード  
- [NTU-RGB+D60](http://rose1.ntu.edu.sg/datasets/actionrecognition.asp)    
3D skeletons (body joints)のみ必要  
データセットを使う場合は申請が必要ですが,　skeleton dataのみは[Github](https://github.com/shahroudy/NTURGB-D)で公開中. 

- [NTU-RGB+D120](http://rose1.ntu.edu.sg/datasets/actionrecognition.asp)   
3D skeletons (body joints)のみ必要  
NTU-RGB+Dと同様にskeleton dataのみは[Github](https://github.com/shahroudy/NTURGB-D)から入手可能  
    - NTU-RGB+D120は, NTU-RGB+D60と合わせる事で完全なデータセットになります. NTU-RGB+D60の中身をNTU-RGB+D120のディレクトリ内にコピーしておいてください. 
    ```
    find ntu-rgb+d-skeletons60/ -name "*.skeleton" -print0 | xargs -0 -I {} mv {} ntu-rgb+d-skeletons120/
    ```

### train, testに分割 
各プログラムには変数`origin_path`(ダウンロードしてきたデータのパス)があるので, 適宜変更  
- NTU-RGB+D60  
```python Tools/Gen_dataset/ntu60.py```  

- NTU-RGB+D120  
```python Tools/Gen_dataset/ntu120.py```  

### マルチモーダルのデータを生成
- NTU-RGB+D60  
```python Tools/Gen_dataset/multi_modal.py --dataset ntu60```  

- NTU-RGB+D120  
```python Tools/Gen_dataset/multi_modal.py --dataset ntu120```



# 学習
- 学習やモデルのパラメータはconfigファイルに記述しています．  
  `Tools/Config`下にデータセットごとにconfigファイルを置いてあります．  
  configファイルを編集することで設定を変更できます.  主な変更箇所を挙げます.  
    - log_dir: ログを残すディレクトリ
    - device: GPU番号を設定, 配列で複数与えれば複数のGPUを使った学習を行います.
    - hop_size: グラフにおけるhop_size分離れた関節を接続します.
    - num_att_A: 動作ごとに生成するAttention_edgeの数
- configファイルを読み込んで学習を行います．
    - NTU-RGB+D60のxsubのcoordinate(joint coordinate, bone)データで学習
    ```
    python train.py --config Tools/Config/NTU-RGB+D60/xsub/coordinate.yaml
    ```
    
    - NTU-RGB+D60のxsubのveloicyt(joint velocity, bone velocity)データで学習
    ```
    python train.py --config Tools/Config/NTU-RGB+D60/xsub/velocity.yaml
    ```
    
    - NTU-RGB+D60のxsubのacceleration(joint acceleration, bone acceleration)データで学習
    ```
    python train.py --config Tools/Config/NTU-RGB+D60/xsub/acceleration.yaml
    ```
- 他のデータセットで学習する場合は違うconfigファイルを読み込みます.  
    - 例:  
        - NTU-RGB+D60のxviewのcoordinateデータ  
          `--config Tools/Config/NTU-RGB+D60/xview/coordinate.yaml`
        - NTU-RGB+D120のxsetupのvelocityデータ  
          `--config Tools/Config/NTU-RGB+D120/xsetup/velocity.yaml`
       
          
# 評価
- 学習したモデルを用いて評価を行います. 学習と同様のconfigファイルを読み込みます.
    - NTU-RGB+D60のxsubのcoordinate(joint coordinate, bone)データでテスト
    ```
    python test.py --config Tools/Config/NTU-RGB+D60/xsub/coordinate.yaml
    ```
    
    - NTU-RGB+D60のxsubのveloicyt(joint velocity, bone velocity)データでテスト
    ```
    python test.py --config Tools/Config/NTU-RGB+D60/xsub/velocity.yaml
    ```
    
    - NTU-RGB+D60のxsubのacceleration(joint acceleration, bone acceleration)データでテスト
    ```
    python test.py --config Tools/Config/NTU-RGB+D60/xsub/acceleration.yaml
    ```

- Attention graphを描画する場合は`--visualize`をつけます.  
  可視化結果の生成は時間がかかるため, testのみをしたい場合は`--visualize`をつけない事を薦めます.   
    - 例:  
        ```
        python test.py --config Tools/Config/NTU-RGB+D60/xsub/coordinate.yaml --visualize
        ```
  
  
        
# Mechanics-stream構造  
- `test.py`を動かすことで各logディレクトリに`test_score.pkl`が生成されています.   
  `test_score.pkl`を読み込んで，クラス確率の合計値を最終の推論値とさせます.    
  ``` 
  python ensemble.py --dataset ntu60_xsub --score1 coordinate --score2 velocity --score3 acceleration
  ```
    - `--dataset`:実行するデータセットを選択します. 以下から選択してください.  
    ntu60_xsub, ntu60_xview, ntu120_xsub, ntu120_xsetup
    - `--score1(2,3)`:`test_score.pkl`が保存されているディレクトリ名を与えます.   
    `--socre1 Log/NTU-RGB+D60/xsub/hogehoge/test_score.pkl`のようにパスを書くとエラーを起こします.`hogehoge`のみを与えてください. 
    