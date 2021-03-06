---
layout: post
title: "簡單運用財報因子選股也能創造好績效"
date: 2018-01-12
description: "財報因子選股"
tag: [Investment]
---

### 一、前言
試想投資如果是一件簡單的事情該有多好，如同Warren Buffett所言：「投資是一門簡單的事情，但做得好往往很少。」投資確實可以很簡單也可以很困難，今天嘗試使用簡單的方式帶著讀者們一起選股，與其隨波逐流、大海撈針，不如先過濾一些體質比較差的股票，避免雜亂無章的選股。
如標題所示，今天會使用兩方面的因子做股票篩選，分別是財報因子與技術因子，財報與技術分析基本上對於投資人都是投資股票的參考依據，接著再搭配資產配置的方式，也就是持有一段時間後賣出，長期持有比起短線波段操作往往更能賺錢，而資產配置的另一個好處是分散風險，避免投資人重押特定股票產生劇烈虧損。

### 二、財報因子
在進入正題前，首先要先了解財報公佈日的問題，在每年的3/31、5/15、8/14、11/14會公布前一季的財報，也就是說`第一季財報5/15公布`，`第二季財報8/14公布`、`第三季財報11/14公布`、`第四季財報(年報)於隔年3/31公布`，記好這些時間後，接著我們開始介紹今天所使用的財報因子。



| 季別 | 公布日 |
|:-------:|:------:|
| 第一季財報 | 05/15 |
| 第二季財報 | 08/14 |
| 第三季財報 | 11/14 |
| 第四季財報(年報) | 03/31 |

#### **選定因子**

| 編號 | 因子 |
|:-------:|:------:|
| 1 | $$\dfrac{營收(Sales)}{企業價值(EV)}$$ |
| 2 | $$現金殖利率=\dfrac{每股自由現金流}{價格}$$ |

1. 　
* 講解第一個因子前，先了解何謂企業價值（Enterprise Value，EV），企業價值乃是併購一間企業需要付出的成本，一般計算企業價值的公式為：權益市值＋債務市值－現金，由此公式可知道企業價值涵蓋了股東、債權人等種種的訴求，透漏著一些帳面數字無法揭露的資訊。
* 往往驅動企業價值（EV）的因素大多來自企業的營收（Sales），會選用營收（Sales）是因為這數字比起稅前息前利益（EBIT）、稅後淨利（NI）被操弄的機會來的小，看過損益表的讀者們應該知道，營收（Sales）是損益表的第一個數字，減去一堆費用後才會得到稅前息前利益（EBIT）與稅後淨利（NI），因此筆者才會選定`營收/企業價值`當作其中一個財報因子。

2. 現金殖利率通常被投資人拿來當作選擇定存股的依據，也有一些投資人也會拿現金殖利率判斷現在的價格是否太便宜或是太貴，因此筆者想透過這兩種指標挑選體質不錯的股票。

#### **篩選方式**
* 為避免流動性問題而買不到股票，因此先選擇市值前300大的股票作為起始的股票池（pool）。
* 接著使用第一個指標`營收/企業價值`由大到小排列，從股票池中再篩選出前100大的股票。
* 最後使用第二個指標`現金殖利率`由大到小排列，選出前10%的股票，也就是最後會選出10檔股票。
#### **買賣時點**
根據上述所說的財報公布日，我們簡單將這些日期作為買賣時點的操作，簡單來說我們每季更換投資標的，重新挑選股票。
* 買進：財報公布日的下一個交易日買進
* 賣出：下個財報公布日的前一個交易日賣出
#### **資產配置**
接著配置我們選出來的10檔股票，資產配置方法大致有以下方法：
1. **等權重配置（Equal weight）**
2. **等風險配置（Equal risk）**
3. **最小變異組合（Global MinVar）**
4. **波動擇時組合（Volatility Timing）**
5. **切線投資組合（Tangency Portfolio）**

#### **績效回測**
由夏普值來看，波動擇時組合（VT）是表現最好的，最大回撤率也僅有8.8%，平均而言年化報酬率約在6%上下，且從累積報酬圖來看各個投資組合皆擊敗大盤，顯示這樣的選股策略能創造一定的獲利品質。
* 回測期間：2010/01/04~2018/5/31


![](https://i.imgur.com/6deFh8i.png)

> 註：圖中紫色的線為大盤


| 投資組合 | 總報酬率 | 年化報酬率 | 最大回撤率 | 勝率 | 夏普值 |
|:------:|:------:|:------:|:------:|:------:|:------:|
| 等權重配置（EW） | 70.27% | 6.09% | -15.5% | 55.0% | 0.69 |
| 等風險配置（ER） | 76.21% | 6.49% | -12.2% | 55.3% | 0.76 |
| 最小變異組合（GMV） | 61.42% | 5.46% | -9.5% | 57.0% | 0.81 |
| 波動擇時組合（VT） | 79.36% | 6.71% | -8.8% | 55.0% | 0.86 |
| 切線投資組合（TP） | 71.70% | 6.14% | -18.3% | 52.6% | 0.70 |

#### **年度分析（以等權重配置（EW）為例）**
從各年度來看，虧損的年度僅有2010年與2015年，獲利比較好的年份為2016年，其餘年分約莫也有3~5%的平均報酬率。

| 年度 | 平均季報酬率 | 最大報酬 | 最小報酬 |  勝率  | 波動度 | 夏普值 |
|:----:|:----------:|:--------:|:--------:|:------:|:------:|:------:|
| 2010 | -1.18%     | 18.77%   | -16.21%  | 52.50% | 13.75% | -0.34  |
| 2011 | 0.85%      | 23.66%   | -17.07%  | 55.00% | 11.20% | 0.30   |
| 2012 | 3.09%      | 39.97%   | -20.79%  | 60.00% | 19.26% | 0.64   |
| 2013 | 3.95%      | 41.93%   | -15.96%  | 57.50% | 18.74% | 0.84   |
| 2014 | 5.31%      | 39.34%   | -20.91%  | 67.50% | 14.89% | 1.43   |
| 2015 | -3.56%     | 29.55%   | -28.15%  | 40.00% | 20.47% | -0.69  |
| 2016 | 8.59%      | 70.56%   | -8.42%   | 68.06% | 20.28% | 1.69   |
| 2017 | 5.05%      | 44.01%   | -12.52%  | 57.50% | 12.00% | 1.68   |
| 2018 | 5.54%      | 30.66%   | -25.68%  | 54.70% | 18.36% | 0.60   |

### 三、後續改進
本次回測並未考慮停損機制，若再加入停損，績效勢必再往上提升，對於一般投資人來說，這樣的選股邏輯確實能有較穩定的獲利，唯獨需要做好資金管理與風險控管，避免在下跌趨勢產生巨大虧損。

---
**溫馨提醒：**
1. 以上實證皆為本人使用R/Python撰寫，請勿抄襲。
2. 投資有賺有賠，本次實作僅為研究使用，投資人投資前應審慎評估，本人不負任何投資賠償責任。
