# 目的

- xgboost v2.0が公開されたので変更点などを調査


# 参考情報

- [XGBoost 2.0 is Here](https://analyticsindiamag.com/xgboost-2-0-is-here/)
- [github-xgboost](https://github.com/dmlc/xgboost/releases)
- [Multiple Outputs - Multiple Outputs](https://xgboost.readthedocs.io/en/stable/tutorials/multioutput.html#training-with-vector-leaf)
- [A demo for multi-output regression](https://xgboost.readthedocs.io/en/stable/python/examples/multioutput_regression.html#sphx-glr-python-examples-multioutput-regression-py)
- [Distributed XGBoost with PySpark](https://xgboost.readthedocs.io/en/stable/tutorials/spark_estimator.html#distributed-xgboost-with-pyspark)

# サマリ

## Multi-target trees機能
- 必要に応じて多出力ツリーを構築することができるように。
- 多出力ツリーとは、単一の決定木が複数のターゲットまたは出力変数を予測する能力を持つツリーのことを指す。（NNのマルチヘッドモデルと近しい概念と思われる）
  - e.g.) 気象データをもとに、同時に最高気温と最低気温を予測、ある病気の存在確率とその病気が進行するリスクを同時に予測など。
- `hist` メソッドを使用する場合、葉のサイズ（すなわち、葉の出力の数）はターゲットの数と等しくすることができる。
- multi_strategyトレーニングパラメータで制御でき、ターゲットごとに1つのモデルを構築する場合はone_output_per_tree（デフォルト）、複数の出力木を構築する場合はmulti_output_treeの値を与える。

## pysparkへの対応
- xgboostはv1.7よりpysparkへの対応を開始している
- v2.0ではGPUベースの予測、データ初期化の改善、特徴量寄与の予測のサポート、Python型付けのサポート、トレーニングのログの改善など、多くの新機能と最適化を受けるように。
- [Distributed XGBoost with PySpark](https://xgboost.readthedocs.io/en/stable/tutorials/spark_estimator.html#distributed-xgboost-with-pyspark)に色々書いてあるので参考に。

## デバイスパラメータの追加
- `gpu_id`、`gpu_hist`、`gpu_predictor`、`cpu_predictor`、`gpu_coord_descent`、PySpark固有の`use_gpu`が追加。

## デフォルトのツリーメソッド
- XGBoost 2.0から、ツリーメソッドのデフォルト設定が`hist`に。
  - それ以前のバージョンでは、入力データや学習環境に基づいて、`approx`または`exact`のツリーメソッドを自動的に選択していた。
  - `hist`をデフォルトにすることで、モデルのトレーニングの効率や一貫性を向上させることが期待できる。

## メモリ・フットプリントの最適化 
- 新しいパラメータ「max_cached_hist_node」が導入され、ユーザーがヒストグラムのCPUキャッシュ・サイズを制限できるように。
- 特に深いツリーにおいて、ヒストグラムの積極的なキャッシュを防ぐのに役立つ。
- 分散システムにおける`hist`および`approx`ツリーメソッドのメモリ使用量が半分になる効果がある。


# セットアップ
## dockerをインストール
### docker desktopをインストールする場合
- [ここ](https://docs.docker.com/desktop/)などを参考にdocker desktopをインストール

### CLIでインストールする場合
- ec2でubuntuインスタンスを立ち上げた場合などを想定
```shell
sudo apt-get update
sudo apt-get install docker.io
sudo gpasswd -a {ユーザ名:ubuntu} docker
docker --version #表示されればセットアップ完了
exit #gpasswdでユーザを付与した後に再ログイン必要
```

## git clone
```shell
git clone https://github.com/takuya-tokumoto/tutorial-xgboost-v2.0.git
cd env-analysis
```

## dorkerの起動して環境構築

```shell
./build.sh
./run.sh #windows環境の場合は run_for_win.sh を実行
```

## jupyterlabへアクセス

- ローカルPCで立ち上げた場合はブラウザから`locathost:8888`
- EC2の場合はブラウザから`{パブリック IPv4 DNS}:8888`
