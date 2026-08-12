## CNN気づき

### layers.Conv1D(filters,kernel_size,padding="causal")
1次元のデータの畳み込みに使う<br>
padding="causal"に設定すると、t時点の出力をする際に未来の値を見ないようになる<br>
基本的に**畳み込みたい次元がいくつあるか**でConv1DかConv2Dか使い分ける<br>
基本的に、構造化データの時系列モデルは畳み込むのは時間軸のみなのでConv1D