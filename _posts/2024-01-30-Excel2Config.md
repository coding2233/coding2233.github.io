---
title: Excel转Protobuf/Json通用配置文件
author: wanderer
date: 2024-01-30 11:47:00
categories: [Game-Dev, Config, .Net]
tags: [.net,excel,c#,config,protobuf,json,yaml]
render_with_liquid: false
---

# 使用场景
最近工作流中有将Excel转Protobuf作为配置文件的技术方案。具体实现是先定一个proto文件，再在一个对应excel表中定义对应字段，由策划在excel进行更改。proto文件可以生成对应语言的脚本，然后将excel转成对应protobuf的binary。  

我的想法就是优化掉自定义proto文件的步骤，根据约束在excel中定义数据类型，在导出数据前，自动导出对应的proto文件，以及生成对应的脚本文件。

# Excel定义约束
Excel已定义的关键词，以下的所有的配置都可以放在任何sheet里面

|关键词|描述|
|------|---|
|#message|同protobuf message|
|#enum|同protobuf enum|
|#package|同protobuf package|
|#config|标识当前message导出为config|
|#desc|描述/注释|
|#type|数据类型, 基本同Protobuf类型，有自定义|
|#var|变量名称|
|#value|枚举的变量|

## message
基本的Message结构如下,RarityType参考下方的枚举定义,支持map和list,list同protobuf repeated  
|#message|Card|  |  |  |  |  |
|--------|----|----|---|-----------|------------------------|-----------------|
|#type|int32|string|bool|RarityType|map#string:string#sep=,|list#string#sep=,|
|#var|id|name|hide|raity|sounds|animations|
|#desc|唯一标识|名称|隐藏|罕见程度|音效|动画|
||10001|名称01|false|2|apply:aa.ogg,walk:bb.ogg|run,attack,idle|
||10002|名称02|false|1|apply:aa.ogg,walk:bb.ogg|run,attack,idle|

## enmu
枚举的变量赋值必须从0开始,受限于protobuf的限制  
|#enmu|	RarityType|||
|---|---|---|--|		
|#desc|	罕见程度|||		
||	#var	|#desc	|#value|
||	Basic	|基础	|0|
||	Common	|普通	|1|
||	Rare 	|稀有	|2|
||	Epic 	|史诗	|3|
||	Legendary 	|传说	|4|
||	Fixed 	|固定	|5|

## config
`#config`是加在#message上方的表示，后面表格的内容为当前的配置名称  
|#config|	collect|
|-------|----------|
|#message|	Collect|
|#type|	list#Card|
|#var|	CardList|
|#desc|	列表|

## type
基本类型同protobuf的基本类型，比如int32、string等，list、map参考上方的message示例。如果是引用其他的类型结构，直接添加对应的类型名称即可， 具体的数据读取对应类型定义下方填写的数据，参考上方的config示例。

# 工具实现

都需要安装`.net6`或者以上的环境，工具在windows下可以直接调用Excel2Config执行，`linux/mac`环境，可以调用`dotnet Excel2Config.dll --help`

|Excel2Config| --help|
|--|--|
|--help|Show this text.|
|--version|           Show version info. 0.1.0.|
|--excel_path=|       The path to the excel file or folder.|
|--recursive,-R|      Traverse all the subfolders of the excel folder.|
|--output_path=|      Setting the output directory. If it is not set, it is the folder path of excel.|
|--to_json|           Convert to a json configuration file.|
|--to_protobuf=|      Convert to a protobuf configuration file. Input parameter proto|textproto|binaryproto|all, all is recommended.|
|--protoc=|           Set the path to the protoc execution file.Environment variables are used by default protoc.|
|--shell=|            Set the path to the shell execution file.Environment variables are used by default sh.|
|--protoc_cmd=|       By default, the output file path of proto is set, and other protoc commands that need to be executed are added.|

## 使用示例 

```shell
Excel2Config --excel_path=Excel/ --to_json --to_protobuf=all --protoc_cmd="--csharp_out=Excel/" --shell="C:\\Program Files\\Git\\bin\\bash.exe" --protoc="D:\\protoc.exe"
```

### Excel配置

![](/assets/images/excel_01.png)  
![](/assets/images/excel_02.png)  
![](/assets/images/excel_03.png)  

### 导出文件

对应的protobuf的binary、proto文件、c#脚本，以及json配置文件  
![](/assets/images/excel_04.png)  
![](/assets/images/excel_05.png)  


# 夹带私货
* 虽然支持json导出，但是还是推荐导出protobuf作为配置使用，json只是作为可视化参考  
* 有json可以利用其他工具转成,yaml等配置文件  
* 这里利用`shell`环境去调用protoc的命令，即使在windows下也需要设置shell环境,开发都安装了`git-bash`环境，所以这里的shell环境也不是啥大问题。windows下的路径确实有点恶心。

# ToDo
* 需要支持配置大文件的分割，并使用同一个结构脚本