---
layout: post
title: "集群分析"
date: 2018-09-11
description: "R語言集群分析教學"
tag: [R]
---


### 一般集群分析
#### 要件：
* **Distance**：計算點與點以及群與群之間的相似程度，常見的演算法有：
    * 歐式距離
    * 曼哈頓距離
* **Centroid**：決定每群中心點的方法

#### 大致可分為：
* **階層式分群（Hierarchical Clustering）**
    * 聚合式 Agglomerative（AGNES）
    * 分割式 Divisive（DIANA）
    * 其他：CURE、BIRCH、CHAMELEON
* **切割式分群（Partitional Clustering）**
    * K-means
    * K-medoid／PAM（Patition around medoid）

### Distance
#### 歐式距離
在2維空間(平面)中，點x=(x1, x2)與點y=(y1, y2)之間的歐式距離為
![](/images/formula/euclidean1.png)
一般通式寫成：
![](/images/formula/euclidean2.png)

#### 曼哈頓距離（絕對誤差）
在2維空間(平面)中，點x=(x1, x2)與點y=(y1, y2)之間的曼哈頓距離為
![](/images/formula/manhattan1.png)
一般通式寫成：
![](/images/formula/manhattan2.png)

### 切割式分群（Partitional Clustering）
#### K-means
##### 原理
給定一個資料集，按照樣本之間的距離大小，將資料集切割成k群，讓群內的點盡量緊密地連在一起（越小越好），而群與群之間的距離越大越好
##### 流程
1. 給定群數k，隨機產生k個群聚中心點（資料空間內任意值）
2. 計算每個資料點與每個中心點的距離
3. 資料點與其最相近的中心點會被劃分成一群，形成k群
4. 利用目前得到的分群重新計算中心點（平均每個點的值）
5. 重複2~4的步驟直到收斂（達到最大迭代次數or群中心點每次更換的變動距離最小）
![](https://i.imgur.com/MA0Np0Q.png)
> 圖片來源：http://www.cnblogs.com/skyEva/p/5630820.html

##### 範例（鳶尾花資料集）
* 鳶尾花資料集(iris dataset)
    * Sepal.length（花萼長度）
    * Sepal.width（花萼寬度）
    * Petal.length（花瓣長度）
    * Petal.width（花瓣寬度）
    * Species（物種）：山鳶尾(setosa)、變色鳶尾(versicolor)、維吉尼亞鳶尾(virginica)

* 套件讀取

```
library(factoextra)
library(cluster)
```

* 資料處理

```
# 讀取資料
data("iris")
# 將特徵標準化以利分群
iris.x <- apply(iris[, -5], 2, function(x){(x-mean(x))/sd(x)})
```

* k-means

```
# k-means
km.iris <- kmeans(iris.x, centers = 3)
# 分群視覺化
fviz_cluster(km.iris, data = iris.x, geom = "point",
             stand = T, ellipse.type = "norm")
```

![](https://i.imgur.com/dksPbGI.png)

#### K-medoid / PAM（Patition around medoid）
##### 原理
與K-means的做法類似，差別在於==k-means==所選定的中心點為群內所有資料點之**平均值**，而==k-medoid==則是以群內某個**樣本點**作為中心點
##### 流程
1. 給定群數k，隨機選取k個群聚中心點（必須是樣本點）
2. 計算每個資料點到每個中心點的距離
3. 資料點與其最相近的中心點會被劃分成一群，形成k群
4. 利用目前得到的分群重新計算中心點
    1. 計算群內所有樣本點至某一個樣本點的曼哈頓距離和（絕對誤差）
    2. 選出使群內絕對誤差距離最小之樣本點作為新的中心點
5. 重複2~4的步驟直到收斂（達到最大迭代次數or群中心點每次更換的變動距離最小）


##### 範例（鳶尾花資料集）
```
# K-medoid
pam.iris <- pam(iris.x, k = 3)

# 分群視覺化
fviz_cluster(pam.iris, stand = T, geom = "point",
             ellipse.type = "norm")
```
![](https://i.imgur.com/Ee0WJk4.png)

### 階層式分群（Hierarchical Clustering）
##### 原理
透過一種階層架構的方式，將資料層層聚合或分裂，以產生樹狀結構
常見的方式有：
* 聚合（AGNES）：由樹的底層開始，由下而上將資料或群集合併
* 分裂（DIANA）：由樹的頂層開始，由上而下將群集逐次分裂
![](https://i.imgur.com/PLiEalJ.png)

* 樹狀圖
![](https://i.imgur.com/OWPZWgc.png)
> 圖片來源：[STHDA](http://www.sthda.com/english/articles/28-hierarchical-clustering-essentials/90-agglomerative-clustering-essentials/)

#### 聚合式 Agglomerative（AGNES）
![](https://i.imgur.com/MBLDIBt.png)
> 圖片來源：[藍鯨的網站分析筆記-層次聚類算法的原理及實現](http://bluewhale.cc/2016-04-19/hierarchical-clustering.html)

##### 流程
1. 將每個資料點各自視為一個群集
2. 找出所有群集間，兩個距離最相近的群集
3. 合併兩個群集成一個新的群集
4. 重複步驟2~3直至我們所設定的群數

* **Distance**：利用**歐式距離**判斷資料間的遠近
* **Centroid**：根據所算出之距離，進行資料的聚合，大致有下列方法：
    * 單一連結法 Single linkage（最近法）
        * 不同群中最接近之兩點距離
    * 完全連結法 Complete linkage（最遠法）
        * 不同群中最遠之兩點距離
    * 平均法 Average linkage
        * 不同群中各點間距離總和之平均
    * 中心法 Centroid linkage
        * 不同群中之中心點距離
    * 華德法 Ward Method
        * 將兩群合併後，各點到合併後的中心點之距離平方和
##### 範例（鳶尾花資料集）
* 建立距離矩陣

| 參數 | 說明
| -------- | --------
| method| euclidean(歐式距離)、manhattan(曼哈頓距離)、canberra、maximum、binary、minkowski

```
dist.res <- dist(iris.x, method = "euclidean")    # 歐式距離
```

* Hierarchical clustering

| 參數 | 說明
| -------- | --------
| method| single、complete、average、centroid、ward.D2

```
hc <- hclust(dist.res, method = "complete")

# 分群視覺化
plot(hc)
rect.hclust(hc, k = 3, border = 2:4)
```

![](https://i.imgur.com/2KEhCND.png)

#### 分割式 Divisive（DIANA）
##### 流程
與聚合式（AGNES）相反，不多贅述
##### 範例
請參考上方

### 最佳分群數
一般有三種方法可供判斷：
* Elbow method
* Average Silhouette method
* Gap statistic method

#### Elbow method
![](https://i.imgur.com/HR4clB8.png)
* 利用組內變異數SSE來衡量分群的好壞
* 隨著分群數的增加，組內變異數SSE會不斷地減少，此方法認為增加分群數，分群效果並不能增強，因此存在一個「拐點」，該點即為最佳分群數
* 分群數從1到3下降地最快，之後下降地很慢，因此3為最佳分群數
* 但「拐點」的認定相當主觀，這也是Elbow method的最大缺陷
* [Robert L. Thorndike (December 1953)](https://link.springer.com/article/10.1007%2FBF02289263)

##### k-means

```
fviz_nbclust(iris.x, kmeans, method = "wss") +
  geom_vline(xintercept = 3, linetype = 2)
```
##### k-medoid

```
fviz_nbclust(iris.x, pam, method = "wss") +
  geom_vline(xintercept = 3, linetype = 2)
```
##### Hierarchical clustering

```
fviz_nbclust(iris.x, hcut, method = "wss") +
  geom_vline(xintercept = 3, linetype = 2)
```

#### Average Silhouette method
* 結合內聚力與分散力，用來衡量群聚效果的好壞
* 其值越大，代表分群效果越好
* [Peter J. Rousseeuw (1986)](http://svn.donarmstrong.com/don/trunk/projects/research/papers_to_read/statistics/silhouettes_a_graphical_aid_to_the_interpretation_and_validation_of_cluster_analysis_rousseeuw_j_comp_app_math_20_53_1987.pdf)
##### 方法
1. 計算樣本點 i 至同群內其他樣本點的平均距離 ai，ai 越小，代表樣本點 i 越應該被歸類至該群
2. 計算樣本點 i 至其他某群Cj的所有樣本之平均距離 bij，bi 越大，代表樣本點 i 越不屬於該群
3. ai稱為組內不相似度，bi稱為組間不相似度，根據兩者定義**輪廓係數（Silhouette coefficient）**

![](/images/formula/silhouette_coefficient.png)

4. 判斷：
    * si接近1，代表樣本 i 的分群合理
    * si接近-1，代表樣本 i 更應該被分類至其他群內
    * si接近0，代表樣本 i 在兩個群的邊界上

##### k-means

```
fviz_nbclust(iris.x, kmeans, method = "silhouette")
```
![](https://i.imgur.com/3o4HDt2.png)

##### k-medoid

```
fviz_nbclust(iris.x, pam, method = "silhouette")
```
![](https://i.imgur.com/uJ91f4L.png)

##### Hierarchical clustering

```
fviz_nbclust(iris.x, hcut, method = "silhouette",
             hc_method = "complete")
```
![](https://i.imgur.com/kKFcQWj.png)

#### Gap statistic method
![](https://i.imgur.com/8ClQpRt.png)
* 將原本Elbow method中所提到的缺陷進行改良

![](/images/formula/gap.png)

* 當k最小，Gap值最大時，k即為最佳分群數
* [R. Tibshirani, G. Walther, and T. Hastie (Standford University, 2001)](http://web.stanford.edu/~hastie/Papers/gap.pdf)

##### k-means

```
gap_stat <- clusGap(iris.x, FUN = kmeans, nstart = 25,
                    K.max = 10, B = 50)
fviz_gap_stat(gap_stat)
```
![](https://i.imgur.com/oN7dADc.png)

##### k-medoid

```
gap_stat <- clusGap(iris.x, FUN = pam, K.max = 10, B = 50)
fviz_gap_stat(gap_stat)
```
![](https://i.imgur.com/bmWSvZ2.png)

##### Hierarchical clustering

```
gap_stat <- clusGap(iris.x, FUN = hcut, K.max = 10, B = 50)
fviz_gap_stat(gap_stat)
```
![](https://i.imgur.com/RpUJMJz.png)

---

### 時間序列集群分析
* [dtwclust套件使用說明](https://cran.r-project.org/web/packages/dtwclust/vignettes/dtwclust.pdf)
#### 常見集群方法
* **階層式分群（Hierarchical Clustering）**
    * 聚合式 Agglomerative（AGNES）
    * 分割式 Divisive（DIANA）
    * 其他：Any other suitable time-series centroid method
* **切割式分群（Partitional Clustering）**
    * PAM（Patition around medoid）
    * DBA+DTW


#### 要件
* **Distance**：
    * **動態時間校正 DTW（Dynamic time warping distance）**
        * 改寫自歐式距離
        * 找出一條路徑最短，使得兩不同長度序列的距離最短
        * 找出波形之間的相似度
        * [原理](http://waoffice.ee.kuas.edu.tw/download/%E5%BB%BA%E5%BE%B7%E7%A0%94%E7%A9%B6%E6%89%80%E8%B3%87%E6%96%99/%E4%B8%83%E6%9C%88%E8%AA%B2%E7%A8%8B/%E5%8B%95%E6%85%8B%E6%99%82%E9%96%93%E6%A0%A1%E6%AD%A3/%E5%8B%95%E6%85%8B%E6%99%82%E9%96%93%E6%A0%A1%E6%AD%A3.ppt)

* **Centroid**：
    * PAM
    * DBA（DTW barycenter averaging）
        * [原理第15頁](https://blog.csdn.net/I_am_huang/article/details/77345659)


#### 搭配用法

Prototyping function|Distance|Algorithm
:---:|:---:|---
PAM|歐式距離／DTW|Time-series with minimum sum of distances to the other series in the group
DBA|DTW|Average of points grouped according to DTW alignments


### 切割式分群（Partitional Clustering）
#### PAM
##### 原理 & 流程
基本上與一般集群分析的k-medoid雷同，請參考上方說明


##### 範例（以台灣市值前50大股票為例）
* 資料預處理

```
# 套件讀取（請自行安裝）
library(tidyverse)
library(tidyquant)
library(dtwclust)
library(cluster)

# 讀取資料
data <- read.table("stockdata.txt", sep = "\t", header = T, stringsAsFactors = F)
# 欄位命名
colnames(data) <- c("code", "name", "date", "open", "high", "low", "close", "volume", "mv")

# 取出2018年1月2日市值前50大的股票
top <- data %>%
  arrange(code, date) %>%
  group_by(code) %>%
  filter(date==20180102) %>%
  arrange(desc(mv)) %>%
  group_by() %>%
  slice(., 1:50) %>%
  pull(name)

# 建立日頻報酬率table
dayLogRet <- data %>%
  arrange(code, date) %>%
  group_by(name) %>%
  filter(name %in% top) %>%
  mutate(dayLogRet=log(close/lag(close, 1))) %>%
  group_by() %>%
  select(code, name, date, dayLogRet) %>%
  na.omit()

##################### 重要 #####################

# 將資料轉成 dtwclust套件的讀取格式
symbol <- unique(dayLogRet$name)
stockRetData <- lapply(symbol, function(iy){
  stockRet <- dayLogRet %>% filter(name==iy) %>% .$dayLogRet
  return(stockRet)
})
names(stockRetData) <- symbol

# 需先將資料進行標準化，以利更好的分群結果
stockRetData_z <- zscore(stockRetData)
```
* PAM

```
pc_k <- tsclust(stockRetData_z,
                k=3,
                distance="dtw_basic",
                centroid = "pam")

# 儲存分群資訊
pCluster <- data.frame(name=names(pc_k@datalist), group=pc_k@cluster)
pCluster <- pCluster %>% arrange(group)
```

* 參數說明

| 可調整參數 | 參數值 | 說明 |
|:---:|:---:| --- |
| series| stockRetData_z | 必須將data.frame轉成list格式放入 |
k | 3 | 群數，通常依經驗決定
distance|"dtw_basic"|距離演算法，預設為歐式距離，若為時間序列，建議使用dtw_basic
centroid|"pam"|決定每群中心點的方法

* 畫圖

```
plot(pc_k)
```

![](https://i.imgur.com/xEK5ylK.png)

#### DBA+DTW
##### 原理 & 流程
請見上方DBA與DTW原理

##### 範例（以台灣市值前50大股票為例）
* DBA+DTW分群

```
dba_k <- tsclust(stockRetData_z,
                 k=3,
                 distance = "dtw_basic",
                 centroid = "dba")

# 儲存分群資訊
dbCluster <- data.frame(name=names(dba_k@datalist), group=dba_k@cluster)
dbCluster <- dbCluster %>% arrange(group)
```

* 參數說明

| 可調整參數 | 參數值 | 說明 |
|:---:|:---:| --- |
| series| stockRetData_z | 必須將data.frame轉成list格式放入 |
k | 3 | 群數，通常依經驗決定
distance|"dtw_basic"|距離演算法，預設為歐式距離，若為時間序列，建議使用dtw_basic
centroid|"dba"|決定每群中心點的方法

* 畫圖

```
plot(dba_k)
```

![](https://i.imgur.com/Jrenb3b.png)

### 階層式分群（Hierarchical Clustering）
##### 原理 & 流程
基本上與一般集群分析方法雷同，請參考上方說明

##### 範例（以台灣市值前50大股票為例）

* 階層式分群（AGNES）

```
hcaCluster <- tsclust(stockRetData_z,
                      type="h",
                      k=3,
                      distance="dtw_basic",
                      control=hierarchical_control(method=agnes))
```

* 參數說明

| 可調整參數 | 參數值 | 說明 |
|:---:|:---:| --- |
| series| stockRetData_z | 必須將data.frame轉成list格式放入 |
type | "h" | h為階層式集群的代號
k | 3 | 群數，通常依經驗決定
distance|"dtw_basic"|距離演算法，通常用dtw_basic即可
control|hierarchical_control(method=agnes))|method可使用agnes或diana

* 樹狀圖（AGNES）

```
plot(hcaCluster)
```

![](https://i.imgur.com/EWt2bkv.png)


* 階層式分群（DIANA）

```
hcaCluster <- tsclust(stockRetData_z,
                      type="h",
                      k=3,
                      distance="dtw_basic",
                      control=hierarchical_control(method=diana))
```

* 樹狀圖（DIANA）

```
plot(hcaCluster)
```

![](https://i.imgur.com/1BgObls.png)

### 最佳分群數（CVI，Cluster validity indices）
* 利用dtwclust套件中的cvi函式
* cvi函式中有多項指標，詳細請見[dtwclust套件使用說明](https://cran.r-project.org/web/packages/dtwclust/vignettes/dtwclust.pdf)第24頁
* 以下僅示範cvi函式的用法
#### PAM

```
# 需要設定多群的k
pc_cvi <- tsclust(stockRetData_z,
                  k=2:10,
                  distance = "dtw_basic",
                  centroid = "pam")
# 將結果命名
names(pc_cvi) <- paste0("k_",2:10)

# 使用sapply搭配cvi函式同時計算多群的衡量效果
pc_eva <- sapply(pc_cvi, cvi, type = "internal")

# 使用Average Silhouette method衡量
plot(x=2:10, pc_eva[c("Sil"),], type="l")
```

![](https://i.imgur.com/klqgvOF.png)

#### DBA+DTW

```
# 需要設定多群的k
dba_k <- tsclust(stockRetData_z,
                 k=2:10,
                 distance = "dtw_basic",
                 centroid = "dba")
# 將結果命名
names(dba_cvi) <- paste0("k_",2:10)

# 使用sapply搭配cvi函式同時計算多群的衡量效果
dba_eva <- sapply(dba_cvi, cvi, type = "internal")

# 使用Average Silhouette method衡量
plot(x=2:10, dba_eva[c("Sil"),], type="l")
```

![](https://i.imgur.com/vUx8t7u.png)

#### Hierarchical clustering

```
# 需要設定多群的k
hcaCluster <- tsclust(stockRetData_z,
                      type="h",
                      k=2:10,
                      distance="dtw_basic",
                      control=hierarchical_control(method=diana))

# 將結果命名
names(hcaCluster) <- paste0("k_",2:10)

# 使用sapply搭配cvi函式同時計算多群的衡量效果
hca_eva <- sapply(hcaCluster, cvi, type = "internal")

# 使用Average Silhouette method衡量
plot(x=2:10, hca_eva[c("Sil"),], type="l")
```

![](https://i.imgur.com/jfgbTXk.png)


---

### 參考資料
1. [机器学习：K-means和K-medoids对比](https://blog.csdn.net/databatman/article/details/50445561)
2. [聚类算法——k-medoids算法](https://blog.csdn.net/coder_Gray/article/details/79705032)
3. [R筆記–(9)分群分析(Clustering)](https://rpubs.com/skydome20/R-Note9-Clustering)
4. [K means 演算法](https://dotblogs.com.tw/dragon229/2013/02/04/89919)
5. [K-Means聚类算法原理](https://www.cnblogs.com/pinard/p/6164214.html)
6. [【机器学习】确定最佳聚类数目的10种方法](https://zhuanlan.zhihu.com/p/24546995)
7. [聚类评估算法-轮廓系数（Silhouette Coefficient ）](https://blog.csdn.net/wangxiaopeng0329/article/details/53542606)
8. [层次聚类算法的原理及实现Hierarchical Clustering](http://bluewhale.cc/2016-04-19/hierarchical-clustering.html)
9. [Determining The Optimal Number Of Clusters: 3 Must Know Methods](http://www.sthda.com/english/articles/29-cluster-validation-essentials/96-determining-the-optimal-number-of-clusters-3-must-know-methods/#three-popular-methods-for-determining-the-optimal-number-of-clusters)
10. [Dynamic Time Warping](http://mirlab.org/jang/books/dcpr/dpDtw.asp?title=8-4%20Dynamic%20Time%20Warping&language=chinese)
11. [DTW Barycenter Averaging（DBA）——平均序列求法](https://blog.csdn.net/I_am_huang/article/details/77345659)

## Reference
1. [Robert L. Thorndike (December 1953)](https://link.springer.com/article/10.1007%2FBF02289263)
2. [Peter J. Rousseeuw (1986)](http://svn.donarmstrong.com/don/trunk/projects/research/papers_to_read/statistics/silhouettes_a_graphical_aid_to_the_interpretation_and_validation_of_cluster_analysis_rousseeuw_j_comp_app_math_20_53_1987.pdf)
3. [R. Tibshirani, G. Walther, and T. Hastie (Standford University, 2001)](http://web.stanford.edu/~hastie/Papers/gap.pdf)
4. [Paparrizos and Gravano (2015)](http://www1.cs.columbia.edu/~jopa/Papers/PaparrizosSIGMOD2015.pdf)
5. [Comparing Time-Series Clustering Algorithms in R Using the dtwclust Package](https://cran.r-project.org/web/packages/dtwclust/vignettes/dtwclust.pdf)
