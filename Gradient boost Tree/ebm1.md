# Explainable Boosting Machine

## Explainable Boosting Machineの使い方(引数)

### objective
ExplainableBoostingRegressor(回帰)のみの引数。
目的関数にあたるもので、デフォルトは*rmse*

### learning rate
学習率、デフォルト値は0.01

### interactions
抽出する交互作用項の数、デフォルト値は10

### max_leaves
各ブースティングステップで使用する木の最大葉数。デフォルト値は3

### max_bins
ヒストグラムのビンのようなもの、粗くするとシンプルになるが細かい特徴を逃してしまう、多くすると、プロコンが逆になる。デフォルト値は256

### early_stopping_rounds
早期学習停止条件、デフォルト値は50

### n_jobs
使用するコア数、デフォルト値は-1(全て)
