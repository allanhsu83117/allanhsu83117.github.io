---
layout: post
title: "梯度下降法 - Gradient Descent"
date: 2018-01-22
description: "梯度下降"
tag: [R]
---

### 前言
首先感謝台大李弘毅老師的youtube影片開啟了我對深度學習的學習之路，在影片中也針對梯度下降有了詳盡的[解說](https://www.youtube.com/watch?v=fegAeph9UaA&index=3&list=PLJV_el3uVTsPy9oCRY30oBPNLCo89yu49)，今天僅分享實作梯度下降的結果。

### 介紹
梯度下降法是優化／最佳化（Optimization）演算法的一種，深度學習中常透過這種方式，反覆迭代，找到損失函數的局部最佳解。

#### 資料
* 今天使用的資料集：**R的內建資料[diamonds](https://ggplot2.tidyverse.org/reference/diamonds.html)**，連結內有此資料集的欄位說明。

* 最後決定挑選**切割品質為Ideal**且**透明度為IF**的鑽石資料，只保留價格與克拉數的欄位，並且針對克拉數的欄位進行標準化。

```
library(tidyverse)
library(ggplot2)

data <- diamonds %>% filter((cut=="Ideal") & (clarity=="IF")) %>% select(price,carat)

# 標準化
data$carat_std <- (data$carat-mean(data$carat))/sd(data$carat)
x <- data %>% select(carat_std) %>% as.matrix()  
y <- data$price
```

### 一、全量梯度下降(Batch Gradient Descent，BGD)
每次迭代使用**全部的樣本**去計算損失函數的梯度，然後使用學習比率依據梯度的反方向更新參數，然而此方法存在一個缺點，當樣本數極大時，運算時間非常久，而且需要消耗許多記憶體，因此衍生出下方SGD的演算法。

* 程式碼如下：

```
# 參數
b <- 1000
w1 <- 5000
lr <- 0.0001        # 學習比率
iteration <- 10000  # 迭代次數

# 初始化
b_history <- b
w1_history <- w1
Loss_history <- sum((data$price-b-w1*data$carat_std)^2)
intercept_org <- data.frame(b=b_history,w1=w1_history,L=Loss_history)

for(i in 1:iteration){
  b_grad <- 0
  w1_grad <- 0

  b_grad <- sum(-2*(data$price - b - w1*data$carat_std)*1)
  w1_grad <- sum(-2*(data$price - b - w1*data$carat_std)*data$carat_std)

  # 更新參數
  b <- b - lr * b_grad
  w1 <- w1 - lr * w1_grad

  # 損失函數
  L <- sum((data$price-b-w1*data$carat_std)^2)


  # 儲存所有參數
  intcp <- cbind(b,w1,L)
  intercept_org <- rbind(intercept_org,intcp)

}
```

### 二、隨機梯度下降(SGD)
SGD全名為Stochastic Gradient Descent，相對於全量梯度下降，隨機梯度下降在每次迭代時**隨機挑選一組樣本**進行參數更新，更新速度極快，然而SGD也並非沒有缺點，SGD每次更新時並非朝著最佳解的方向進行更新，也因為更新頻繁，下圖可看到每次更新時的變動相當大，雖然速度快，但準確度也下降。

![](https://i.imgur.com/uG3Dkow.png)

* 程式碼如下：

```
# 參數
b <- 1000
w1 <- 5000
lr <- 0.001       # 學習比率
iteration <- 20   # 迭代次數

# 初始化
b_history <- b
w1_history <- w1
Loss_history <- sum((data$price-b-w1*data$carat_std)^2)
intercept_SGD <- data.frame(b=b_history,w1=w1_history,L=Loss_history)
intercept_SGD_all <- data.frame(b=b_history,w1=w1_history,L=Loss_history)


for(i in 1:iteration){

  for(ix in 1:nrow(data)){

    data_SGD <- data[ix,]

    b_grad <- -2*(data_SGD$price - b - w1*data_SGD$carat_std)*1
    w1_grad <- -2*(data_SGD$price - b - w1*data_SGD$carat_std)*data_SGD$carat_std

    # 更新參數
    b <- b - lr * b_grad
    w1 <- w1 - lr * w1_grad

    # 顯示進度條
    cat("第",i,"次疊代 ","第",ix,"樣本 ","b=",b," w=",w1,"\n")

    # 損失函數
    L <- (data_SGD$price-b-w1*data_SGD$carat_std)^2

    # 儲存每次迭代每筆樣本參數
    intcp_SGD_all <- cbind(b,w1,L)
    intercept_SGD_all <- rbind(intercept_SGD_all,intcp_SGD_all)

    # 僅儲存每次迭代的參數
    if(ix==nrow(data)){
      intcp_SGD <- cbind(b,w1,L)
      intercept_SGD <- rbind(intercept_SGD,intcp_SGD)
    }
  }
}

```

### 三、Adagrade
Adagrade全名為Adaptive Gradient Algorithm，此算法能針對稀疏特徵，增加其更新幅度，而對非稀疏特徵，縮小其更新幅度，因此此方法適合稀疏數據，另外Adagrade的另一個優點是能自動調整學習比率，減少手動調整的麻煩，然而這是優點也是缺點，可看到學習比率每次更新會一直累加，導致迭代到後期更新參數時，學習比率`lr/sqrt(b_lr)`變得很小。

* 程式碼如下：

```
# 參數
b <- 1000
w1 <- 5000
lr <- 1000           # 學習比率
iteration <- 10000   # 迭代次數
b_lr <- 0
w1_lr <- 0

# 初始化
b_history <- b
w1_history <- w1
Loss_history <- sum((data$price-b-w1*data$carat_std)^2)
intercept_ada <- data.frame(b=b_history,w1=w1_history,L=Loss_history)

for(i in 1:iteration){
  b_grad <- 0
  w1_grad <- 0

  b_grad <- sum(-2*(data$price - b - w1*data$carat_std)*1)
  w1_grad <- sum(-2*(data$price - b - w1*data$carat_std)*data$carat_std)


  # 調整學習比率
  b_lr <- b_lr + b_grad^2
  w1_lr <- w1_lr + w1_grad^2


  # 更新參數
  b <- b - lr/sqrt(b_lr) * b_grad
  w1 <- w1 - lr/sqrt(w1_lr) * w1_grad
  L <- sum((data$price-b-w1*data$carat_std)^2)

  # 儲存參數
  intcp <- cbind(b,w1,L)
  intercept_ada <- rbind(intercept_ada,intcp)
}
```

### 四、繪製等高線圖
* （上）藍色的線為Adagrade
* （中）紫色的線為BGD
* （下）粉紅色的線為SGD

![](https://i.imgur.com/JGWZzeY.png)

### 五、結論
除了這3個方法以外，還有其他很多種的梯度下降法，目前看到最好的梯度下降法似乎為**Adam(Adaptive Moment Estimation)**，但最重要的是了解資料的特性再來選擇適合的演算法才是做資料科學的最重要的步驟。

### 六、參考資料
1. https://blog.csdn.net/heyongluoyao8/article/details/52478715
2. https://hk.saowen.com/a/97823dc63110d2508082bc94fa390acc72503ebb23e0d4b4f490c58cde7fdf54
