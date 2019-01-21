---
layout: post
title: "Python爬蟲+SQL建置+自動化更新"
date: 2018-10-13
description: "yahoo股價資料爬取"
tag: [爬蟲 + SQL]
---

### 一、前言
因本人目前在幫忙建置理財機器人的部分，剛好有機會到後台玩玩，因此這篇簡易教學也就順手誕生了。本程式由Python撰寫，因Python執行速度快，在爬蟲方面也有完善的套件，可與SQL相連接，做自動化相當順手，可維護性極高。

### 二、爬蟲 & SQL匯入
本程式會由Yahoo Finance抓取資料以國外的ETF為主，大概講解一下程式架構，基本的Python語法和SQL語法就不一一贅述。

#### 套件

```python=
# 匯入套件
import pandas as pd
import numpy as np
import requests
import io
import os
import time
import datetime
import mysql.connector

# 設定路徑
os.chdir("C:/你的/路徑")
```

#### 是否重新抓取資料
* 這邊設定 **y** 時，程式會重新抓取2000年到現在的所有ETF資料，因為極度耗費時間，所以定期執行就好，不需要每天執行。
* 設定 **n** 時，程式則會每天更新近10天的股價資料。

```python=
# 設定是否重抓資料
recrawling_data = 'n'
```

#### 每日資料更新
* 注意時差問題，Yahoo Finance資料更新與台灣時間落後一天，因此抓資料時必須將時間往前推移一天。
* 本程式會自動判斷今日是否開盤，沒有開盤則不需要更新資料。（21行~58行，透過if else決定今天是否開盤）
* 觀察抓取網址（以SPY該檔ETF為例）

> https://query1.finance.yahoo.com/v7/finance/download/SPY?period1=1544893628&period2=1547572028&interval=1d&events=history&crumb=Vc9GVPFLZgc

1. 可以發現粗體的地方download/**SPY**?為該ETF的代號，因此若要抓取所有的ETF勢必要有代碼。
2. **period1=1544893628**&**period2=1547572028** 由此發現period1為起始日，period2為結束日，另外各自後面的一串數字為日期的unix格式，因此在爬取前必須將日期設定好並轉成unix格式。
3. **interval=1d** 為資料頻率，在yahoo finance可設定周頻或是月頻，不過我們要抓取日頻資料，因此不需做修改。

```
if recrawling_data == 'N' or recrawling_data == 'n':
    # 日期設定
    # 今日日期(台灣)
    today_date = datetime.datetime.today().strftime('%Y-%m-%d')
    # 交易日期(美國)
    df_last_date = (datetime.date.today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d')
    # 資料抓取起始日
    start_date = (datetime.date.today() - datetime.timedelta(days=10)).strftime('%Y-%m-%d')
    # 資料抓取起始日前一天(為了刪除資料庫內的資料而設定)
    bf_st_date = (datetime.date.today() - datetime.timedelta(days=11)).strftime('%Y-%m-%d')

    # 將日期轉成unix格式
    st_unix = str(int(time.mktime(datetime.datetime.strptime(start_date, '%Y-%m-%d').timetuple())))
    ed_unix = str(int(time.mktime(datetime.datetime.strptime(today_date, '%Y-%m-%d').timetuple())))

    # 爬蟲設定
    s = requests.Session()
    # 請求頭設定
    cookies = dict(B='2dlj1k5dn0vg0&b=3&s=g3')
    headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36'}

    # 抓取大盤資料(S&P500)確定今天是否開盤
    bench_url = 'https://query1.finance.yahoo.com/v7/finance/download/%5EGSPC?period1=' + st_unix + '&period2=' + ed_unix + '&interval=1d&events=history&crumb=xxy94FXoLdp'
    r = s.get(bench_url, cookies=cookies, headers=headers).content
    bench_data = pd.read_csv(io.StringIO(r.decode('utf-8')))

    error_ticker = []

    # 若今日有開盤則開始更新資料
    if bench_data['Date'].iloc[-1] == df_last_date:

        # 匯入ETF代碼資料
        code_list = open('ETF_code.txt')
        ticker = code_list.readlines()
        code_list.close()
        ticker = list(map(lambda s: s.strip(), ticker))
        # 開始爬資料
        for i in ticker:
            print("Running to : ", ticker.index(i), "/", (len(ticker)-1), "\n")
            url='https://query1.finance.yahoo.com/v7/finance/download/'+ i +'?period1=' + st_unix + '&period2=' + ed_unix + '&interval=1d&events=history&crumb=xxy94FXoLdp'
            r = s.get(url, cookies=cookies, headers=headers).content

            # 紀錄下市名單
            if len(r) < 500:
                error_ticker.append(i)
                next

            # 合併ETF價格資料
            else:
                if i == ticker[0]:
                    data = pd.read_csv(io.StringIO(r.decode('utf-8')))
                    data['code'] = i
                else:
                    df = pd.read_csv(io.StringIO(r.decode('utf-8')))
                    df['code'] = i
                    data = data.append(df)

            #time.sleep(5)
        # 重新命名欄位
        data.columns = ['date','open','high','low','close','adjusted','volume','code']
        # 針對日期格式處理以便匯入SQL資料庫
        data['date'] = data['date'].str.replace("-", "")
        data.to_csv('ETF_Data_new.csv', sep=',', encoding='utf-8', index=False)
        print("Saving Complete!")

    # 若今日沒有開盤則不更新資料
    else :
        print("Today is holiday!")

    # 連接 mysql
    etfdb = mysql.connector.connect(
      host="xxx.xxx.xx.xxx",
      user="xxxxx",
      passwd="xxxxxxxxxxxxxx"
    )
    etfcursor = etfdb.cursor()

    # 刪除某段時間的資料
    bf_st_date = bf_st_date.replace("-", "")
    df_last_date = df_last_date.replace("-", "")
    # 送入SQL指令將資料刪除
    query ="DELETE FROM etf.historical_data WHERE date >= " + bf_st_date + " and date <= " + df_last_date + ";"
    etfcursor.execute(query)
    etfdb.commit()
    print("Data Deleted!")

    # 送入SQL指令將資料匯入
    query = "LOAD DATA LOCAL INFILE 'E:/NSYSU/昭文團隊/ETF/ETF_Data_new.csv' INTO TABLE etf.historical_data FIELDS TERMINATED BY ',' LINES TERMINATED BY '\r\n' IGNORE 1 ROWS;"
    etfcursor.execute(query)
    etfdb.commit()
    print("Data Inserted!")
```


#### 重新更新所有資料
* 寫法與上段程式碼相近，僅需更改資料起始日，並將是否開盤的判斷條件拿掉即可。

```
elif recrawling_data == 'Y' or recrawling_data == 'y':
    # 日期設定
    today_date = datetime.datetime.today().strftime('%Y-%m-%d')
    df_last_date = (datetime.date.today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d')
    start_date = '2000-01-01'

    st_unix = str(int(time.mktime(datetime.datetime.strptime(start_date, '%Y-%m-%d').timetuple())))
    ed_unix = str(int(time.mktime(datetime.datetime.strptime(today_date, '%Y-%m-%d').timetuple())))

    # 爬蟲設定
    s = requests.Session()

    cookies = dict(B='2dlj1k5dn0vg0&b=3&s=g3')
    headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36'}


    error_ticker = []

    # 匯入ETF代碼資料
    code_list = open('ETF_code.txt')
    ticker = code_list.readlines()
    code_list.close()
    ticker = list(map(lambda s: s.strip(), ticker))
        # 開始爬資料
    for i in ticker:
        print("Running to : ", ticker.index(i), "/", (len(ticker)-1), "\n")
        url='https://query1.finance.yahoo.com/v7/finance/download/'+ i +'?period1=' + st_unix + '&period2=' + ed_unix + '&interval=1d&events=history&crumb=xxy94FXoLdp'
        r = s.get(url, cookies=cookies, headers=headers).content

        if len(r) < 500:
            error_ticker.append(i)
            next
        else:
            if i == ticker[0]:
                data = pd.read_csv(io.StringIO(r.decode('utf-8')))
                data['code'] = i
            else:
                df = pd.read_csv(io.StringIO(r.decode('utf-8')))
                df['code'] = i
                data = data.append(df)

            #time.sleep(5)
    data.columns = ['date','open','high','low','close','adjusted','volume','code']
    data['date'] = data['date'].str.replace("-", "")
    data.to_csv('ETF_Data.csv', sep=',', encoding='utf-8', index=False)
    print("Saving Complete!")

    # 連接 mysql
    etfdb = mysql.connector.connect(
      host="xxx.xxx.x.xxx",
      user="xxxxx",
      passwd="xxxxxxxxxxxxx"
    )
    etfcursor = etfdb.cursor()

    # 刪除 MySQL內的表格
    query ="TRUNCATE TABLE etf.historical_data;"
    etfcursor.execute(query)
    etfdb.commit()
    print("Data Truncated!")

    # 將資料匯入
    query = "LOAD DATA LOCAL INFILE 'E:/NSYSU/昭文團隊/ETF/ETF_Data.csv' INTO TABLE etf.historical_data FIELDS TERMINATED BY ',' LINES TERMINATED BY '\r\n' IGNORE 1 ROWS;"
    etfcursor.execute(query)
    etfdb.commit()
    print("Data Inserted!")
```

### 三、自動化更新
* 將以上的檔案存成.py檔後，利用windows的自動排程系統定時執行爬蟲的script
