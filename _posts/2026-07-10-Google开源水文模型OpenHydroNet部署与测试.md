---
layout:     post
title:     Google开源水文模型OpenHydroNet部署与测试
subtitle:   
date:       2026-07-08
author:     WT
header-img: img/post-bg-flooding.jpg
catalog: true
tags:
    - Google
    - OpenHydroNet
    - 水文  
---


### 背景


Google Flood Hub 是 Google Research 开发的一个基于人工智能（AI）的全球洪水预警平台，其目标是利用机器学习、水文模型、遥感数据和天气预报，为政府、救灾机构和公众提供免费的洪水预测服务。它能够对<font size=3 color=Red>河流洪水</font>进行最长 7 天提前预报，帮助降低洪灾造成的人员伤亡和经济损失。Flood Hub 自 2018 年开始投入实际运行，并持续扩大覆盖范围，目前已成为 Google 应急响应体系的重要组成部分。

Google Flood Hub 主要由三部分组成，水文模型（Hydrologic Model）、淹没模型（Inundation Model）和可视化展示（Flood Hub UI）。水文模型预测未来河流流量，淹没模型预测哪里被淹，Flood Hub可视化部分进行地图展示和预警。
#### 水文模型
水文模型是Google洪水预报系统的第一层，负责回答未来几天河里的水会涨多少。
Google 使用了深度学习模型（包括 LSTM 等时序模型）与传统水文模型相结合来预测河流流量。  
输入数据包括天气预报、降雨预报、土壤湿度、地形、流域面积、上游汇流、历史水文数据。  
输出数据包括河流径流流量。

#### 淹没模型
淹没模型是Google洪水预报系统的第二层，负责回答水涨起来以后，哪些地方会被淹。
Google **不再完全依赖传统二维水动力模型**，而利用 AI 学习大量历史洪水与地形关系，实现快速推断淹没范围和水深。
输入数据包括  DEM、河道、河流流量、河宽、地形坡度。
输出数据包括 未来某个未知的预计积水深度和概率。

#### Flood Hub UI
Flood Hub UI 是Google洪水预报系统的可视化层，负责展示。  
展示内容主要是河流河段的流量，包括当前的流量与水位，未来7天的预测结果、峰现时间、是否超警。海提供洪水淹没概率图（分辨率约15m）

### Google洪水预报的数据来源
Google 采用的基本是公开的数据集。 天气预报包括了ECMWF、NOAA和其他全球天气预报产品，用于提供未来降雨预测。 DEM高程数据，用于计算河流的河谷、坡度和河流流向。卫星影像用于地表水体提取，进行历史洪水提取与洪水预报验证。河流网络数据主要使用了HydroSHEDS。还使用了各国公开的水文站点数据。

###  OpenHydroNet
OpenHydroNet 是 Google 开源的 AI 水文预测模型，对应于 Google Flood Hub 的水文模型层（Hydrologic Layer），承担洪水预报链条中降雨–径流模拟的核心任务。该模型融合多源气象预报和流域静态属性等信息，预测未来河流流量（Discharge），为后续水位计算、淹没范围模拟以及洪水预警提供关键的水文边界条件。需要指出的是，OpenHydroNet 仅实现了洪水预报系统中的水文预测模块，并不包含水位转换、淹没模拟、预警发布等完整洪水预警系统的其他组成部分。

**根据 Google 官方说明，当前 Flood Hub 水文模型采用集总流域（Lumped Catchment）建模方式，直接预测目标河段流量，而不显式模拟河网中的下游汇流过程。因此，该模型不进行基于河网拓扑的显式汇流计算或图结构信息传播。但官方并未说明各小流域之间完全独立，也未否认模型输入中可能包含上游或区域尺度的信息。**


#### 创建运行环境

```
# 将代码下载到本地
git clone https://github.com/google-research/flood-forecasting.git

cd flood-forecasting-main

#创建虚拟Python环境
conda env create -f environments/conda.yml

#激活虚拟环境
conda activate googlehydrology

#重要的步骤，因为后面修改文件后不需要再安装
pip install -e .

```
  
![wmts2](http://www.spatial.pro/img/D20260710_01.png)    


# 测试官方Demo

本人使用的操作系统环境：<font size=3 color=Red>Ubuntu 22.04.5 LTS</font>   
下载的代码中包含了官方的jupyter notebook 文件，位于目录flood-forecasting-main\flood-forecasting-main\tutorial下的OpenHydroNet_Tutorial.ipynb文件。
在googlehydrology虚拟环境中打开jupyter notebook，加载OpenHydroNet_Tutorial.ipynb，然后依次运行代码。    

![wmts2](http://www.spatial.pro/img/D20260710_02.png)  
注意：一定要运行以下的代码，否则输入结果会不正常     
%matplotlib inline  以内嵌方式显示图表  
![wmts2](http://www.spatial.pro/img/D20260710_03.png)     
代码中的文件存储目录需要修改为实际环境中的目录，除此之外，后面代码的配置文件（config.yml）也需要改动.  
![wmts2](http://www.spatial.pro/img/D20260710_04.png)   
![wmts2](http://www.spatial.pro/img/D20260710_05.png)      
![wmts2](http://www.spatial.pro/img/D20260710_06.png)     
![wmts2](http://www.spatial.pro/img/D20260710_06_01.png)      
加载训练和测试集     
![wmts2](http://www.spatial.pro/img/D20260710_07.png)   
使用的是CAMELS数据集的一部分，以下是QGIS中带影像底图的数据集范围。    
![wmts2](http://www.spatial.pro/img/D20260710_07_01.png)  


以下是基础模型的各流域评估结果，后续将使用了13235000的KGE对比微调前后的结果。  
![wmts2](http://www.spatial.pro/img/D20260710_08.png) 

基础模型中13235000的KGE的可视化结果  
![wmts2](http://www.spatial.pro/img/D20260710_09.png)   
微调后的模型结果，明显优于基础模型的结果。  
![wmts2](http://www.spatial.pro/img/D20260710_10.png)   
这个是13235000流域实际观测值、基础模型预测值和微调后的模型预测值间的对比，从峰现时间和洪水峰值流量上，微调后的模型  
![wmts2](http://www.spatial.pro/img/D20260710_11.png)   
流域面积是描述流域特征的重要静态属性之一。流域面积影响许多重要水文过程，比如：降雨汇流时间；径流形成过程；洪峰传播速度；流量响应尺度。因此，流域面积不仅是一个简单的几何参数，而是影响模型学习流域水文行为的重要物理特征。如果微调目标流域的面积明显偏离训练流域面积的主要分布范围（如直方图中显示的情况），则说明：
基础模型可能没有学习到适用于该尺度流域的有效特征表示。    
 流域面积分布分析的意义在于判断目标流域是否偏离预训练数据的尺度范围。如果存在明显尺度差异，基础模型中的静态属性编码层（static_attributes_fc）可能无法生成适合该流域的特征表示，因此通过针对该层进行微调，可以在保持原有水文预测能力的同时，提高模型对目标流域的适应性。  
![wmts2](http://www.spatial.pro/img/D20260710_12.png)     
   

引用文献：  
[Google 洪水预测系统解析：水文模型、淹没模型与国内落地方式](https://mp.weixin.qq.com/s/_Ty6nAQ1b9O8__E-nk4hvQ)    
[Google把洪水预测开源了！这事儿跟你的关系比你想象的大得多](https://mp.weixin.qq.com/s/3djTTU67e1xlHEr-M36NNQ)    
