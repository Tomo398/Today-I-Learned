## AIが書いたコードで素晴らしかったもの

### ある文字を含むセルをnanに置換

```
#明示的にnanと置き換える
df_1days.loc[
    df_1days["気温"].astype(str).str.contains("月", na=False),
    "気温"
] = np.nan
```

置換で*loc[]*を使おうという発想がまずない、その上*str.contains("月")*というメゾットを使おうとも思わない

素晴らしい発想だと思う、見習いたい