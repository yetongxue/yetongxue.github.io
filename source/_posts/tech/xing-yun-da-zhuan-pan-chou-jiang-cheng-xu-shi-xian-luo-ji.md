---
title: 幸运大转盘抽奖 抽奖算法 程序实现逻辑
date: 2017-10-26 00:00:00
sid: baf1c39d-d92f-474f-9d38-49a82389435b
tags: [Java,抽奖]
categories: 
  - 技术杂文
---
>近期碰到的一个需求，实现一个类似大转盘抽奖的功能，需自定义奖项，各奖项中奖概率，当日抽奖最大次数，抽奖成本等。分享一个简单的java代码的实现的思路，有不足之处感谢各位指正。
<!--more-->

![](http://ofbphtmkb.bkt.clouddn.com/qianxun/e/f3/3222d8e23d1b9889a6913d8d1fb14.png)

# 初步方法

首先要定义几个奖品，例如：
- iphone 中奖机率 10% 
- 100元购物卷 中奖机率 30% 
- 10元购物卷 中奖机率 50% 

总的中奖机率是 10%+30%+50%=90%

剩余10%是谢谢惠顾，不中奖的

## 设计思路
这个是把所有商品按照概率分配到数组里面

- A[0] = iphone
- A[1] = iphone 
- A[2] = iphone 
- ...
- A[10] = iphone

- A[11] = 100元购物卷 
- A[12] = 100元购物卷 
- ...

然后随机一个0到99的数字，例如现在随机的数字是2

那么A[2]就是中奖的商品A[2] = iphone

```java
//定义中奖率分母 百分之
int probabilityCount = 100;  
String[] prizesId = new String[probabilityCount];  
//获取商品列表
List<AdPrizeInfo> prizeInfoList = prizeInfoService.getPrizeInfo();  
int num = 0;  
//循环所有商品
for (AdPrizeInfo prize : prizeInfoList) {  
    Integer probability = prize.getOdds();
    //循环商品概率
    for (int i = 0; i < probability; i++) {
        prizesId[num] = prize.getId();
        num ++;
    }
}

//随机一个数字
int index = (int) (Math.random() * probabilityCount);  
//获取到随机商品ID
String prizeId = prizesId[index];  
```

# 优化方法

## 设计思路

以上方法如果大概率的话，是很吃内存的，整理优化为一下方法：

![](http://ofbphtmkb.bkt.clouddn.com/qianxun/c/73/ff890460e3fe6895416d64840985d.jpg)

使用范围算法

```java
//定义中奖率分母 百分之
int probabilityCount = 100;  
//最小概率值
String min = "min";  
//最大概率值
String max = "max";  
Integer tempInt = 0;  
//待中奖商品数组
Map<String,Map<String,Integer>> prizesMap = new HashMap<>();  
//获取商品列表
List<AdPrizeInfo> prizeInfoList = prizeInfoService.getPrizeInfo();  
for (AdPrizeInfo prize : prizeInfoList) {  
    Map<String,Integer> oddsMap = new HashMap<>();
    //最小概率值
    oddsMap.put(min,tempInt);
    tempInt = tempInt + prize.getOdds();
    //最大概率值
    oddsMap.put(max,tempInt);
    prizesMap.put(prize.getId(),oddsMap);
}


//随机一个数字
int index = (int) (Math.random() * probabilityCount);  
AdPrizeInfo prizeInfo = null;  
Set<String> prizesIds = prizesMap.keySet();  
for(String prizesId : prizesIds){  
    Map<String,Integer> oddsMap = prizesMap.get(prizesId);
    Integer minNum = oddsMap.get(min);
    Integer maxNum = oddsMap.get(max);
    //校验index 再哪个商品概率中间
    if(minNum <= index && maxNum > index){
        prizeInfo = prizeInfoService.selectByPrimaryKey(prizesId);
        break;
    }
}

//如果为空，则没中奖
if(prizeInfo == null){  
    prizeInfo = null;
}
```