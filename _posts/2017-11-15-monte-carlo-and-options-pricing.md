---
layout: post
title: "蒙地卡羅模擬法（Monte Carlo）與選擇權定價（Options Pricing）"
date: 2017-11-15
description: "蒙地卡羅"
tag: [Python]
---

### 前言

![](/images/formula/price.png)


### 參數設定

```python=
from scipy.stats import norm
import numpy as np
```

```python=
# Black-Scholes Model參數
S0 = 100       # 資產價格
K = 100        # 履約價
r = 0.01       # 無風險利率
Tt = 10        # 時間
vol = 0.1      # 波動度
N = 20         # 期數
M = 10000      # 模擬次數
dt = Tt/N      # 一期時間長度
```

### 蒙地卡羅法
#### 步驟

![](/images/formula/monte_carlo_process.png)

```python=
# 建立空的矩陣
ST = np.zeros((M,N+1))
# 設定初始價格
ST[:,0] = np.log(S0)

# 執行蒙地卡羅
for n in range(N):
    e = np.random.standard_normal(M)
    ST[:,n+1] = ST[:,n] + (r-0.5*vol**2)*dt + vol*(dt**0.5)*e

# 取出第N期價格
ST_N = np.exp(ST[:, -1])
```

```python=
# 蒙地卡羅法的買權賣權價格
C0_M = np.mean(np.exp(-r*Tt)*np.maximum(ST_N-K,0))
P0_M = np.mean(np.exp(-r*Tt)*np.maximum(K-ST_N,0))
```
* 買權價格：17.395
* 賣權價格：7.936

```python=
# 計算蒙地卡羅準確率
C0_sd = (np.var(np.maximum(ST_N-K,0)))**0.5/(M**0.5)
P0_sd = (np.var(np.maximum(K-ST_N,0)))**0.5/(M**0.5)
```

### 與Black-Scholes Model比較

![](/images/formula/bs_model.png)

```python=
d1 = (np.log(S0/K)+(r+0.5*(vol**2))*Tt)/(vol*(Tt**0.5))
d2 = d1-vol*(Tt**0.5)
C0 = (-K)*np.exp(-r*Tt)*norm.cdf(d2) + S0*norm.cdf(d1)
P0 = K*np.exp(-r*Tt)*norm.cdf(-d2) - S0*norm.cdf(-d1)
```

* 買權價格：17.311
* 賣權價格：7.795

### 畫圖觀察模擬結果
#### 資產價格
![](https://i.imgur.com/luw0Jk8.png)

#### 買權報酬
![](https://i.imgur.com/S1XbiIP.png)

#### 賣權報酬
![](https://i.imgur.com/fWOkdpm.png)
