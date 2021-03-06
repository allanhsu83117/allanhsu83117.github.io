---
layout: post
title: "利用線性回歸、Lasso、彈性網針對4種商品建構策略"
date: 2018-09-30
description: "線性模型與財務策略"
tag: [Machine Learning + Investment]
---

### 一、迴歸模型

$$
{y_i = \alpha + \beta_1x_{1, i} +...+\beta_nx_{n, i}}
$$

#### MSE(Loss Function)

$$
{\min L = (\sum_{n = 1}^N \beta_ix_i - \alpha - y_i)^2}
$$

- Loss Function 分布

![](https://i.imgur.com/7ApX4CN.png)


### 二、正規化模型（Regularization Model)
#### 核心思想
在模型訓練的過程中，越是非線性的參數修改的越嚴重(如高次方$$x$$之係數$$\beta$$)，故隨著模型的次方數提高，模型會更複雜，模型會為了追求更小的損失進而有over-fitting的現象。
為了抑制over-fitting，加入了懲罰項，用以限制模型的複雜度。

#### 1. Lasso Regularization ($$L_1$$)

##### 處罰函數


$$
L = {(\sum_{n = 1}^N \beta_ix_i - \alpha - y_i)^2} + {\sum_{n = 1}^N \vert \beta_n \vert}
$$

![](https://i.imgur.com/uozB5Qg.png)

##### 意義
因為Lasso之值域為絕對值相加，故其分布為多邊形(圖示為二維，若更高維則會產生更多邊角)，故值域與最低Loss Function之交點應為最低點，故多為頂點、或是易在最接近邊上滑動。

因Lasso所述的特性，使得最佳點常為頂點，意即Lasso會降低影響力較低、不顯著之變數的權重($$\beta$$)，甚至降低至0。

#### 2. Ridge Regularization ($$L_2$$)

##### 處罰函數
$$
L = {(\sum_{n = 1}^N \beta_ix_i - \alpha - y_i)^2} + {\sum_{n = 1}^N \beta_i^2}
$$



![](https://i.imgur.com/T75qSzw.png)

##### 意義
由於是每個係數$$\beta$$的平方項總和值域為一個圓，故接近最低時會較穩定，且不會使任一係數為0。
故較為常用。

#### 3. Elastic Net

##### 處罰函數

$$
L = {(\sum_{n = 1}^N \beta_ix_i - \alpha - y_i)^2} + \lambda
{\sum_{n = 1}^N \vert \beta_n \vert}+
(1-\lambda)
{\sum_{n = 1}^N \beta_i^2}
$$

##### 意義

$$L_1$$ + $$L_2$$ 之組合，使迴歸式具有彈性。


---
### 三、資料來源與建模流程
#### 1. Data
- S&P500：SPDR S&P 500 ETF (SPY)
- 台灣50： TEJ (0050tw)
- 黃金價格：SPDR Gold Shares (GLD)
- 美元對台幣：Invesco DB US Dollar Bullish (UUP)
#### 2. Rolling
每三個月，以過去500天的價格資料，重新train一次模型，概念如下
![](https://i.imgur.com/xw5jK6Y.jpg)

### 四、績效
#### 1. Linear Model

資產代碼|平均報酬|年化報酬|累積報酬|標準差|夏普值|最大報酬|最小報酬|
:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
0050tw|0.53%|3.36%|34.16%|2.71%|0.34|11.88%|-6.53%|
GLD|-0.86%|-8.92%|-39.16%|4.56%|-0.79|6.02%|-26.26%|
SPY|1.13%|8.29%|52.71%|2.27%|0.83|10.62%|-4.78%|
UUP|0.15%|-0.57%|-2.99%|1.65%|-0.11|7.65%|-4.01%|


|資產代碼|勝率|交易次數|最大回撤率|獲利因子|
| :---:|:---:|:---:|:---:|:---:|:---: |
| 0050tw | 61.81%|55|18.42%|1.92|
| GLD | 35.55%|45|41.00%|0.43|
| SPY | 72.00%|50|10.56%|6.67|
| UUP | 53.7%|55|16.47%|1.35|


交易策略報酬率直方圖，X軸為單次交易之報酬率，Y軸為交易次數，如此可方便檢視該產品在特定方法下，報酬率的分布狀況。

![](https://i.imgur.com/mKYKb2V.png)![](https://i.imgur.com/uadmwd2.png)![](https://i.imgur.com/1SOdn5x.png)![](https://i.imgur.com/1NlnXk3.png)


#### 2. Lasso

資產代碼|平均報酬|年化報酬|累積報酬|標準差|夏普值|最大報酬|最小報酬|
:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:
0050tw|1.03%|7.00%|43.36%|3.77%|0.58|16.49%|-10.69%|
GLD|-1.97%|-7.57%|-34.21%|7.36%|-0.79|5.31%|-26.63%|
SPY|2.20%|11.63%|79.52%|2.20%|1.05|35.30%|-7.32%|
UUP|0.24%|0.12%|0.63%|1.83%|0.02|3.57%|-4.47%|

|資產代碼|勝率|交易次數|最大回撤率|獲利因子|
| :---:|:---:|:---:|:---:|:---:|:---: |
| 0050tw | 76.08%|46|20.11%|2.80|
| GLD | 50.00%|18|34.34%|0.32|
| SPY | 64.70%|17|16.02%|4.48|
| UUP | 60.0%|20|8.88%|1.48|

![](https://i.imgur.com/LaXK66W.png)![](https://i.imgur.com/sKUWZFw.png)![](https://i.imgur.com/ptr0TFJ.png)![](https://i.imgur.com/QkqZnBf.png)


#### 3. Elastic Net

資產代碼|平均報酬|年化報酬|累積報酬|標準差|夏普值|最大報酬|最小報酬|
:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
0050tw|1.00%|6.65%|40.86%|3.87%|0.55|19.57%|-10.70%|
GLD|-2.59%|-6.18%|-28.79%|8.30%|-0.72|2.27%|-26.63%|
SPY|2.52%|11.39%|77.46%|9.86%|1.02|35.29%|-9.01%|
UUP|0.17%|-0.20%|-1.09%|1.53%|-0.03|3.57%|-4.39%|

|資產代碼|勝率|交易次數|最大回撤率|獲利因子|
| :---:|:---:|:---:|:---:|:---:|:---: |
| 0050tw | 71.74%|46|20.23%|2.92|
| GLD | 45.45%|11|28.80%|0.24|
| SPY | 71.42%|14|17.17%|4.11|
| UUP | 63.3%|30|10.85%|1.43|

![](https://i.imgur.com/Itxa6lF.png)![](https://i.imgur.com/PbE0BL1.png)![](https://i.imgur.com/CQ5FNpy.png)![](https://i.imgur.com/Lanh93M.png)

#### 4. 各產品績效方法比較
##### a. 0050tw

可以發現正規化後平均報酬、累積報酬、年化報酬、夏普比率、最大報酬、勝率等有提升，但一方面最小報酬與最大回撤率也變大了，交易次數降低幅度不大。

方法|平均報酬|年化報酬|累積報酬|標準差|夏普值|
:---:|:---:|:---:|:---:|:---:|:---:|
Linear Model|0.53%|3.36%|34.16%|2.71%|0.34|
Lasso|1.03%|7.00%|43.36%|3.77%|0.58|
Elastic Net|1.00%|6.65%|40.86%|3.87%|0.55|

|方法|最大報酬|最小報酬|勝率|交易次數|最大回撤率|獲利因子|
| :---:|:---:|:---:|:---:|:---:|:---: |:---:|:---:|
| Linear Model |11.88%|-6.53%|61.81%|55|18.42%|1.92|
| Lasso |16.49%|-10.69%|76.08%|46|20.11%|2.80|
| Elastic Net |19.57%|-10.70%|71.42%|14|17.17%|4.11|

##### b. GLD

正規化後平均報酬率表現反而不佳，是四項產品中表現最差，標準差漸變大，夏普值差異不大，最大報酬率降低，勝率皆不超過50%，交易次數大幅降低，但最大回撤率也有降低。

方法|平均報酬|年化報酬|累積報酬|標準差|夏普值|
:---:|:---:|:---:|:---:|:---:|:---:|
Linear Model|-0.86%|-8.92%|-39.16%|4.56%|-0.79|
Lasso|-1.97%|-7.57%|-34.21%|7.36%|-0.79|
Elastic Net|-2.59%|-6.18%|-28.79%|8.30%|-0.72|

|方法|最大報酬|最小報酬|勝率|交易次數|最大回撤率|獲利因子|
| :---:|:---:|:---:|:---:|:---:|:---: |:---:|:---:|
| Linear Model |6.02%|-26.26%|35.55%|45|41.00%|0.43|
| Lasso |16.49%|5.31%|-26.63%|50.00%|18|34.34%|0.32|
| Elastic Net |2.27%|-26.63%|45.45%|11|28.80%|0.24|

##### c. SPY

累積報酬為四項產品中最高，皆超過50%以上；但標準差在彈性網模型下反而飆很高；勝率皆在6成4以上，與獲利因子同為四項產品中最高，但Lasso模型下勝率最低；交易次數大幅降低。

方法|平均報酬|年化報酬|累積報酬|標準差|夏普值|
:---:|:---:|:---:|:---:|:---:|:---:|
Linear Model|1.13%|8.29%|52.71%|2.27%|0.83|
Lasso|2.20%|11.63%|79.52%|2.20%|1.05|
Elastic Net|2.52%|11.39%|77.46%|9.86%|1.02|

|方法|最大報酬|最小報酬|勝率|交易次數|最大回撤率|獲利因子|
| :---:|:---:|:---:|:---:|:---:|:---: |:---:|:---:|
| Linear Model |10.62%|-4.78%|72.00%|50|10.56%|6.67|
| Lasso |16.49%|35.30%|-7.32%|64.70%|17|16.02%|4.48|
| Elastic Net |35.29%|-9.01%|71.42%|14|17.17%|4.11|

##### d. UUP

最大回測率四項產品中最低，年化報酬率表現最接近0，綜觀整體績效指標，相較其他產品，三個方法間的結果差異最不明顯，但整體而言Lasso下績效表現最好。

方法|平均報酬|年化報酬|累積報酬|標準差|夏普值|
:---:|:---:|:---:|:---:|:---:|:---:|
Linear Model|0.15%|-0.57%|-2.99%|1.65%|-0.11|
Lasso|0.24%|0.12%|0.63%|1.83%|0.02|
Elastic Net|0.17%|-0.20%|-1.09%|1.53%|-0.03|

|方法|最大報酬|最小報酬|勝率|交易次數|最大回撤率|獲利因子|
| :---:|:---:|:---:|:---:|:---:|:---: |:---:|:---:|
| Linear Model |7.65%|-4.01%|53.7%|55|16.47%|1.35|
| Lasso |16.49%|3.57%|-4.47%|60.0%|20|8.88%|1.48|
| Elastic Net |3.57%|-4.39%|63.3%|30|10.85%|1.43|

### 五、基金績效圖

#### 1. 0050tw
* Linear Model
![](https://i.imgur.com/jJGytXu.png)
* Lasso
![](https://i.imgur.com/v2QWvMt.png)
* Elastic Net
![](https://i.imgur.com/JdBoh4w.png)

#### 2. GLD
* Linear Model
![](https://i.imgur.com/b5IMNM2.png)
* Lasso
![](https://i.imgur.com/lTGkQuG.png)
* Elastic Net
![](https://i.imgur.com/8lYJrjb.png)

#### 3. SPY
* Linear Model
![](https://i.imgur.com/efcVKP5.png)
* Lasso
![](https://i.imgur.com/FBFapyf.png)
* Elastic Net
![](https://i.imgur.com/lyGDij7.png)

#### 4. UUP
* Linear Model
![](https://i.imgur.com/BDB9y2Q.png)
* Lasso
![](https://i.imgur.com/2JfbSAx.png)
* Elastic Net
![](https://i.imgur.com/7hktDOG.png)

### 六、結論

本次實證利用線性模型對四種商品建構策略，結果可發現在回測績效上並沒有特別亮麗，線性模型較為簡單，而金融市場充斥著複雜的訊息，並非所有訊息都能靠著線性模型萃取而出，雖然如此，本次實，也算是個不錯的嘗試，本方法仍有值得許多可以改進的空間。

---
### Reference

- [L1 / L2 正规化 (Regularization)](https://morvanzhou.github.io/tutorials/machine-learning/ML-intro/3-09-l1l2regularization/)
- [[資料分析&機器學習] 第5.4講: 機器學習進階實用技巧-正規化](https://medium.com/@yehjames/%E8%B3%87%E6%96%99%E5%88%86%E6%9E%90-%E6%A9%9F%E5%99%A8%E5%AD%B8%E7%BF%92-%E7%AC%AC5-4%E8%AC%9B-%E6%A9%9F%E5%99%A8%E5%AD%B8%E7%BF%92%E9%80%B2%E9%9A%8E%E5%AF%A6%E7%94%A8%E6%8A%80%E5%B7%A7-%E6%AD%A3%E8%A6%8F%E5%8C%96-8dd14fcd3140)
- [机器学习中正则化项L1和L2的直观理解](https://blog.csdn.net/jinping_shi/article/details/52433975)

---
**溫馨提醒：**
1. 以上實證皆為本人使用R/Python撰寫，請勿抄襲。
2. 投資有賺有賠，本次實作僅為研究使用，投資人投資前應審慎評估，本人不負任何投資賠償責任。
