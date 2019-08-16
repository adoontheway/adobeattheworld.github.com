---
title: Python读取Excel数据
author: ado
date: 2019-08-16 20:27:23
description: 导入Excel数据，进行分析存档，然后绘制推表
tags: [python]

---

# Preface

今天跟人讨论了一些问题，有个问题，明明某方面数据跟理论很接近，但是的出来的结果却差距太大。但是这个问题能影响到结果的地方实在是太少了，我就说可能是另一因素导致的。大哥表示不信啊，理论就是理论。我为了证明给他看我的假设，为了证明理论也是依赖实际来完善的，于是就抓了几万条数据下来，整理成表以支持我的说法。

# Content

代码如下：

```python
# *-* encoding:utf8 *-*

import xlrd
import pymongo
import redis
from threading import Thread
import math
from time import time
import matplotlib.pyplot as plt
import numpy as np

mongo_client = None
my_db = None
units_per_thread = 5000
max_score = 0
min_score = 99999999
def check_score(value):
    global max_score, min_score
    if value > max_score:
        max_score = value
    if value < min_score:
        min_score = value

class Game001(Thread):

    def __init__(self,coll=None,red=None,startIdx=0, endIdx=0,sheet=None):
        super().__init__()
        self.head = None
        self.item_len = 0
        self.my_coll = coll
        self.red = red
        self.sheet = sheet
        self.startIdx = startIdx
        self.endIdx = endIdx

    def run(self):
        self.head = self.sheet.row(1)
        self.parse_head(self.head)
        for i in range(self.startIdx,self.endIdx):
            self.parse_row(self.sheet.row(i))

    def parse_head(self,row):
        self.item_len = len(row)
        for i in range(self.item_len):
            self.head[i] = row[i].value

    def parse_row(self,row):
        # print(row)
        item = {}
        for i in range(self.item_len):
            item[self.head[i]]=row[i].value
            if self.head[i] == "winscore":
                if row[i].value > 0:
                    self.red.incr("game001:lostcnt")
                    self.red.rpush("game001:loses",row[i].value)
                    check_score(row[i].value)
                elif row[i].value < 0:
                    self.red.incr("game001:wincnt")
                    self.red.rpush("game001:wins", -row[i].value)
                    check_score(-row[i].value)
                else:
                    self.red.incr("game001:tiecnt")
        self.my_coll.insert_one(item)

def init_mongo():
    mongo_client = pymongo.MongoClient("mongodb://192.168.1.91:27017/")
    return mongo_client["data"]

def init_redis():
    return redis.StrictRedis(host="192.168.1.91", port=7001, db=0, password="123456", decode_responses=True)

def parse_sheet(datapath):
    wb = xlrd.open_workbook(datapath)
    return wb.sheet_by_index(0)

def start_parse(r):
    start = time()
    sheet = parse_sheet("./data.xls")
    total_lines = sheet.nrows - 2
    r.hset("game001", "totalline", total_lines)
    my_db = init_mongo()
    num_threads = math.ceil(total_lines / units_per_thread)
    print("total:%d,need threads:%d"%(total_lines,num_threads))
    for i in range(num_threads):
        endIdx = (i+1) * units_per_thread
        endIdx = endIdx > total_lines+2 and total_lines+2 or endIdx
        startIdx = i * units_per_thread
        startIdx = startIdx == 0 and 2 or startIdx
        tbnn = Game001(coll=my_db["game001"], red=r, sheet=sheet,
                    startIdx=startIdx, endIdx=endIdx)
        tbnn.start()
        tbnn.join()
    end = time()
    r.set("game001:max_score",max_score)
    r.set("game001:min_score", min_score)
    print("Total cost:%.2f second"%(end-start))

def start_analyse(r):
    max_score = float(r.get("game001:max_score"))
    min_score = float(r.get("game001:min_score"))
    print("min_score: %d, max_score :%d"%(min_score, max_score))
    gap = max_score - min_score
    units = 10
    while gap/units < 1:
        units = math.floor(units/4)
    print("units:%d"%units)
    unit = math.ceil(gap / units)
    x_labels = [str(min_score+i*unit) for i in range(units)]
    win_data = [0 for i in range(units)]
    lose_data = [0 for i in range(units)]
    losts = r.lrange("game001:loses",0,-1)
    wins = r.lrange("game001:wins",0,-1)
    win_count = len(wins)
    lost_count = len(losts)
    for i in range(win_count):
        score = float(wins[i])
        idx = math.floor((score-min_score)/unit)
        win_data[idx] = win_data[idx] + 1

    for i in range(lost_count):
        score = float(losts[i])
        idx = math.floor((score-min_score)/unit)
        lose_data[idx] = lose_data[idx] + 1

    x = np.arange(len(x_labels))
    width = 0.35
    plt.figure(figsize=(10,6))
    fig,ax = plt.subplots()
    rect1 = ax.bar(x - width/2, win_data, width, label="wins")
    rect2 = ax.bar(x + width/2, lose_data, width, label="lose")
    ax.set_ylabel("Score")
    ax.set_title("Game001 score")
    ax.set_xticks(x)
    ax.set_xticklabels(x_labels)
    def auto_label(rects):
        for rect in rects:
            height = rect.get_height()
            ax.annotate('{}'.format(height),
                        xy=(rect.get_x() + rect.get_width()/2, height),
                        xytext=(0,3),
                        textcoords="offset points",
                        ha="center",
                        va="bottom")
    auto_label(rect1)
    auto_label(rect2)
    # fig.tight_layout()
    '''
    plt.scatter(wins,wins,marker=".",color="red",label="win")
    plt.scatter(losts, losts, marker=".", color="green", label="lose")
    plt.legend(loc="best")
    '''
    plt.show()

if __name__ == '__main__':
    r = init_redis()
    # start_parse(r)
    start_analyse(r)
```

## Result

![result](pychart.png)