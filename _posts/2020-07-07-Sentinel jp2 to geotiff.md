---
layout:     post
title:      Sentinel jp2 to geotiff
subtitle:   
date:       2020-07-07
author:     WT
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - SNAP
    - Sentinel 2
    - GeoTIFF  
---
### 导出波段的分辨率一致，SNAP Subset ###

1. 在Snap中加载需要导出的Sentinel 2 影像
2. Raster-> Subset,  设置Spatial Subset 和 Band Subset 确定导出的范围和波段，点击确定，左侧的列表框会出现裁剪和提取波段后的数据
3. 右键数据，选择 Open RGB Image Window，选择RGB合适的波段
4. 加载完毕后，右键图像，选择"Export View as Image",选择导出的区域(Image Region)、导出的分辨率(Image Resolution)和导出的数据格式,导出文件

如果不需要裁剪，可以直接使用 Raster--data conversion---band select即可导出自己需要的波段

it seems there are some issues with exporting S2 to GeoTiff. However, you can do the following.
Open the product and create a subset (Raster/Subset…) which contains only the required bands (B2,B3,B4) and the area you are interested in. Afterwards open the RGB image. Now right-click on the image and select “Export View as Image” from the context menu.



### 如果分辨率不一致，可能需要重采样 ###

t is basically a simple File > Export > 'GeoTiff / BigTiff' operation, however, it will throw an error as Sentinel-2 bands have various resolutions (10, 20, 60m).

Resample it first, then export.

Start Raster > Geometric Operations > Resampling. On the Resampling Parameters tab, you may choose any of B2, B3, B4 for the reference band, to resample all the bands to 10m resolution.
Create bands subset by Raster > Subset.
On the Product Explorer pane, click on the subset data to highlight then go to File > Export > 'GeoTiff / BigTiff'.
All bands such as the Band 1 (resolution: 60m) will be resampled to 10m in the first step. So this may not give you what you need, if the goal is to stack all the bands as-is.
  

引用文献：  
[Step Forum](https://forum.step.esa.int/t/exporting-rgb-image-as-a-geotiff/1071)   
[StackExchange](https://gis.stackexchange.com/questions/346322/export-sentinal-2-tiff-image-from-snap-with-all-the-bands-intact) 
         