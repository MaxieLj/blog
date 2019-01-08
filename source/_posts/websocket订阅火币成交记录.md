---
title: websocket订阅火币成交记录
date: 2018-06-19 18:15:57
tags: 折腾
categories: 折腾
toc: true
---
# python3
## 单线程：
### 文件一：
```python
# -*- coding: utf-8 -*-
#author: maxie_lj
import _thread
from websocket import create_connection
from mysqlOpertion import connect_mysql
import gzip
import time
import json

# 链接
def connect():
    while(1):
        try:
            print('')
            # 挂代理
            ws = create_connection("wss://www.huobi.br.com/-/s/pro/ws")
            #print('链接成功')
            return ws
            break
        except Exception as err:
            #print(err)
            #print('connect ws error,retry...')
            time.sleep(1)


def getsubconfig():
    # 统计参数
    tradeStr=[
              """{"sub": "market.elfusdt.trade.detail","id": "id1"}""",
              """{"sub": "market.btcusdt.trade.detail","id": "id2"}""",
              """{"sub": "market.htusdt.trade.detail","id": "id3"}""",
              """{"sub": "market.swftcbtc.trade.detail","id": "id4"}""",
              """{"sub": "market.topcbtc.trade.detail","id": "id6"}""",
              """{"sub": "market.bchusdt.trade.detail","id": "id7"}""",
              """{"sub": "market.ethusdt.trade.detail","id": "id8"}""",
              """{"sub": "market.etcusdt.trade.detail","id": "id9"}""",
              """{"sub": "market.eosusdt.trade.detail","id": "id9"}""",
              """{"sub": "market.ltcusdt.trade.detail","id": "id9"}"""
              ]
    return tradeStr

# 订阅
def sub(ws,tradeStr):
    ws.send(tradeStr)

# 获取返回
def getResponse(ws,tradeStr):
    db = connect_mysql.connect()
    while 1 :
        try :
            compressData=ws.recv()
        except Exception :
            #print('链接异常')
            run(tradeStr,1)
        try :
            result=gzip.decompress(compressData).decode('utf-8')
        except:
            #print('丢包解析异常')
            continue
        if result[:7] == '{"ping"':
            ts=result[8:21]
            #print('心跳包',ts)
            pong='{"pong":'+ts+'}'
            ws.send(pong)
        else:
            if result[:5] == '{"ch"':
                result = json.loads(result)
                #print(result)
                #print('------------------------------------------------------')
                n = 0
                while n < len(result['tick']['data']) :
                    connect_mysql.commit('%s' % result['ch'].split('.')[1],result['tick']['data'][n]['price'],result['tick']['data'][n]['amount'],"'%s'" % result["tick"]["data"][n]["direction"], "'%s'" % result['ch'].split('.')[1],db);
                    n += 1

def subCoin(tradeStr):
    ws=connect()
    sub(ws,tradeStr)
    getResponse(ws,tradeStr)

def run(tradeStr,test):
    subCoin(tradeStr)

def main():
    tradeStr=getsubconfig()
    try :
        tradeStr = getsubconfig()
        i = 0
        while i < len(tradeStr) :
            _thread.start_new_thread( run, (tradeStr[i],i) )
            i += 1
    except Exception as err :
        print(err)
    while 1:
        pass


if __name__ == '__main__':
    main()
```

# 多线程版本：
## 文件一：

```python
# -*- coding: utf-8 -*-
#author: maxie_lj
import _thread
from websocket import create_connection
from test1 import connect_mysql
import gzip
import time
import json

# 链接数据库
def connect():
    while(1):
        try:
            ws = create_connection("wss://www.huobi.br.com/-/s/pro/ws")
            # print('链接成功')
            return ws
            break
        except Exception as err:
            print(err)
            #print('connect ws error,retry...')
            time.sleep(1)


def getsubconfig():
    # 统计参数
    tradeStr=[
              """{"sub": "market.elfusdt.trade.detail"}""",
              """{"sub": "market.btcusdt.trade.detail"}""",
              """{"sub": "market.htusdt.trade.detail"}""",
              """{"sub": "market.swftcbtc.trade.detail"}""",
              """{"sub": "market.topcbtc.trade.detail"}""",
              """{"sub": "market.bchusdt.trade.detail"}""",
              """{"sub": "market.ethusdt.trade.detail"}""",
              """{"sub": "market.etcusdt.trade.detail"}""",
              """{"sub": "market.eosusdt.trade.detail"}""",
              """{"sub": "market.ltcusdt.trade.detail"}"""
              ]
    return tradeStr

# 订阅
def sub(ws,tradeStr):
    ws.send(tradeStr)

# 获取返回
def getResponse(ws):
    db = connect_mysql.connect()
    while 1 :
        try :
            compressData=ws.recv()
        except Exception :
            ws=connect()
            subCoin(ws)
            print('链接异常')
        try :
            result=gzip.decompress(compressData).decode('utf-8')
        except:
            print('丢包解析异常')
            continue
        if result[:7] == '{"ping"':
            ts=result[8:21]
            #print('心跳包',ts)
            pong='{"pong":'+ts+'}'
            ws.send(pong)
        else:
            if result[:5] == '{"ch"':
                result = json.loads(result)
                print(result)
                #print('------------------------------------------------------')
                n = 0
                while n < len(result['tick']['data']) :
                    connect_mysql.commit('%s' % result['ch'].split('.')[1],result['tick']['data'][n]['price'],result['tick']['data'][n]['amount'],"'%s'" % result["tick"]["data"][n]["direction"], "'%s'" % result['ch'].split('.')[1],db);
                    n += 1

# 订阅
def subCoin(ws):
    tradeStr=getsubconfig()
    i = 0
    while i < len(tradeStr) :
            sub(ws,tradeStr[i])
            i += 1
# 主函数
def main():
    ws=connect()
    subCoin(ws)
    getResponse(ws)


if __name__ == '__main__':
    main()
```


```
### 文件二:

```python
import json
import pymysql
import time

class connect_mysql():
    db = ''
    def connect() :
            db = connect_mysql.db = pymysql.connect("localhost", "root", "root", "huobi", charset='utf8' )
            return db

    def commit(table, price, amount, action, coin_type, db) :

        try:
            cursor = db.cursor()
            sql = "INSERT INTO %s_success_order(action,order_amount, price, coin_type)VALUES (%s, %.10f, %.10f,%s )" % (table,action, amount, price,coin_type )
            cursor.execute(sql)
           # 提交到数据库执行
            db.commit()
        except Exception as e:
            db = connect_mysql.db = pymysql.connect("localhost", "root", "root", "huobi", charset='utf8' )
            commit(table, price, amount, action, coin_type, db)
        return


    def connect_close() :
        connect_mysql.db.close()
```


文件二是单线程和多线程版的公用文件






# shell 脚本
```shell
#!/bin/bash
total=1
avaliable=`ps -ef | grep -w huobi.py | grep -v grep | wc -l`
diff=`expr $total - $avaliable`
#echo $diff >> /home/script/diff.text
#echo $total >> /home/script/diff.text
#echo $avaliable >> /home/script/diff.text
if [ $diff -gt 0 ];then
        for((i=0;i<$diff;i++));do
                 /usr/local/bin/python3 /home/script/huobi.py >> /home/script/huobi.out
        done
fi
```

# crontab 配置
```powershell
MAILTO=""

* * * * * sudo /bin/bash /home/script/huobi.sh
```

**注意事项**：
在写shell脚本时，尽量用绝对路径。
