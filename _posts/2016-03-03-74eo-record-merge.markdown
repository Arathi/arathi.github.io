---
layout: post
title:  "关于ElectronicObserver记录的合并"
modified: 2016-03-03 15:10:00 +0800
categories: other
tags: 74eo
---

我有两台电脑一台平板，都使用ElectronicObserver(以下简称74eo)玩舰C，
因此想把三台设备中的建造、开发、捞船记录合并起来。

<!--more-->

打开Record目录，会发现以下文件：

1. ConstructionRecord.csv
2. DevelopmentRecord.csv
3. EnemyFleetRecord.csv
4. ResourceRecord.csv
5. ShipDropRecord.csv
6. ShipParameterRecord.csv
7. api_start2.json

其中这个api_start2.json是每次登陆游戏时获取到的，大概1M，包含了游戏中的大部分数据。这个我在开发Kanmulation时有接触过，结构（详见74eo中的[这篇文档](https://github.com/andanteyk/ElectronicObserver/blob/master/ElectronicObserver/Other/Information/apilist.txt)）是很熟悉了。

其他6个csv文件如下：

#### ConstructionRecord.csv

ConstructionRecord是建造记录，结构如下：

| 字段名  | 类型 | 备注 |
|:------:|:----:|:---:|
| 艦船ID  | int | |
|----
| 艦船名  | string | |
|----
| 建造日時 |string | 转换为date |
|----
| 燃料    | int | |
|----
| 弾薬    | int | |
|----
| 鋼材    | int | |
|----
| ボーキ   |int | |
|----
| 開発資材 | int | |
|----
| 大型建造 | int | 转换为bool |
|----
| 空ドック | int | 可能是建造的槽或者剩余空位数量 |
|----
| 旗艦ID   | int |
|----
| 旗艦名   | string | 冗余，可以由旗舰ID推测出 |
|----
| 司令部Lv | int | |
{: rules="groups"}

#### DevelopmentRecord.csv

DevelopmentRecord是开发记录，结构如下：

| 字段名  | 类型 | 备注 |
|:------:|:----:|:---:|
| 装備ID   | int ||
|----
| 装備名   | string ||
|----
| 開発日時 | string | 转换为date |
|----
| 燃料     | int ||
|----
| 弾薬     | int ||
|----
| 鋼材     | int ||
|----
| ボーキ   | int ||
|----
| 旗艦ID   | int ||
|----
| 旗艦名   | string | 冗余，可由旗艦ID推出 |
|----
| 旗艦艦種 | int | 冗余，可由旗艦ID推出 |
|----
| 司令部Lv | int |
{: rules="groups"}

#### EnemyFleetRecord.csv

EnemyFleetRecord是敌舰队数据（游戏数据，与玩家自身无关），结构如下：

| 字段名  | 类型 | 备注 |
|:------:|:----:|:---:|
| 敵編成ID | int ||
|----
| 敵艦隊名 | string ||
|----
| 海域 | int ||
|----
| 海域 | int ||
|----
| セル | int ||
|----
| 難易度 | string | 只有几个常量：甲、乙、丙、なし |
|----
| 陣形 | string | 只有几个常量：単縦陣、複縦陣、輪形陣、梯形陣、単横陣|
|----
| 敵1番艦 | int ||
| 敵2番艦 | int ||
| 敵3番艦 | int ||
| 敵4番艦 | int ||
| 敵5番艦 | int ||
| 敵6番艦 | int ||
|----
| 敵1番艦名 | string| 冗余|
| 敵2番艦名 | string| 冗余|
| 敵3番艦名 | string| 冗余|
| 敵4番艦名 | string| 冗余|
| 敵5番艦名 | string| 冗余|
| 敵6番艦名 | string| 冗余|
|----
| 経験値 | int | |
{: rules="groups"}

#### ResourceRecord.csv

ResourceRecord是各项资源记录，结构如下：

| 字段名  | 类型 | 备注 |
|:------:|:----:|:---:|
| 日時 | string | 可转为date |
|----
| 燃料 | int |
|----
| 弾薬 | int |
|----
| 鋼材 | int |
|----
| ボーキ | int |
|----
| 高速建造材 | int
|----
| 高速修復材 | int
|----
| 開発資材 | int 
|----
| 改修資材 | int
|----
| 司令部Lv | int
|----
| 提督Exp | int
{: rules="groups"}

#### ShipDropRecord.csv

ShipDropRecord是掉落记录，结构如下：

| 字段名  | 类型 | 备注 |
|:------:|:----:|:---:|
| 艦船ID | int
| 艦名 | string | 冗余，当艦船ID为-1时，这个字段的值为"(なし)"
|----
| アイテムID | int | 有些活动会捞到物品，比如菱饼、秋刀鱼
| アイテム名 | string | 冗余
|----
| 装備ID | int |
| 装備名 | string | 冗余
|----
| 入手日時 | string | 可以转为date
|----
| 海域 | int | 章
| 海域 | int | 节
| セル | int | 点
|----
| 難易度 | string
|----
| ボス | string | 可转为bool，表示是否为BOSS点，有"-"、"ボス"两个值
|----
| 敵編成 | int
|----
| ランク | string 
|----
| 司令部Lv | int
{: rules="groups"}

#### ShipParameterRecord.csv

ShipParameterRecord是舰娘属性记录（游戏数据，与玩家自身无关），这个表的信息量还是蛮大的，而且很多数据来源于api_start2.json，只有少部分数据（对潜aws、回避evasion、索敌los）在游戏过程中才能获取到。具体结构如下：

| 字段名  | 类型 | 备注 |
|:------:|:----:|:---:|
| 艦船ID | int
|----
| 艦船名 | string
|----
| 耐久初期 | int
| 耐久最大 | int
|----
| 火力初期 | int
| 火力最大 | int
|----
| 雷装初期 | int
| 雷装最大 | int
|----
| 対空初期 | int
| 対空最大 | int
|----
| 装甲初期 | int
| 装甲最大 | int
|----
| 対潜初期下限 | int
| 対潜初期上限 | int
| 対潜最大    | int
|----
| 回避初期下限 | int
| 回避初期上限 | int
| 回避最大    | int
|----
| 索敵初期下限 | int
| 索敵初期上限 | int
| 索敵最大    | int 
|----
| 運初期 | int
| 運最大 | int
|----
| 射程 | int
|----
| 装備1 | int 
| 装備2 | int 
| 装備3 | int 
| 装備4 | int 
| 装備5 | int 
|----
| 機数1 | int
| 機数2 | int
| 機数3 | int
| 機数4 | int
| 機数5 | int
|----
| ドロップ説明 | string
|----
| 図鑑説明 | string
{: rules="groups"}

注：  
*ShipParameterRecord中的数据可以从wiki上抓取*

#### 合并思路

ConstructionRecord、DevelopmentRecord、ResourceRecord、ShipDropRecord这四个文件都有时间字段，可以时间作为排序依据，而且时间精确到秒，两秒之内很难做同一件事情两次，基本上可以作为唯一索引。


