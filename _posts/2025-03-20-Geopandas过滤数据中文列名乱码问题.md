---
layout:     post
title:      Geopandas过滤数据中文列名乱码问题
subtitle:   
date:       2025-03-20
author:     WT
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - SHP
    - Geopandas
    - Pandas  
    - 中文乱码      
---

### 问题
SHP中保存数据时使用了中文列名，想用这个属性列过滤数据，但Geopandas读取列名不成功，导致过滤错误。

### 解答

读取的时候加上文件编码，<font size=3 color=Red>UTF-8不行的话试一试GBK</font>。
```
gdf = gpd.read_file(shapefile_path,encoding='gbk')
```

详细的代码如下：  
```
import geopandas as gpd
import os

# 读取Shapefile文件
shapefile_path = r"M:\A\X-100.shp"  # 替换为你的shp文件路径
gdf = gpd.read_file(shapefile_path,encoding='gbk')
#print(gdf)
gdf = gdf[gdf['淹没水深'] > 0.001]

# 检查原始数据的 CRS
print("Original CRS:", gdf.crs)

if gdf.crs is None:
    # 手动设置原始数据的 CRS
    gdf.set_crs('EPSG:4544', allow_override=True, inplace=True)
else:
    # 如果原始数据已有 CRS，尝试强制更改它
    gdf.to_crs('EPSG:4544', inplace=True)


# 合并所有多边形
#merged_geometry = gdf['geometry'].unary_union
merged_geometry = gdf['geometry'].union_all()

# 获取合并后的边界
#merged_boundary = merged_geometry.boundary

# 打印合并后的边界坐标
#boundary_coords = list(merged_boundary.coords)
#print("Merged boundary coordinates:", boundary_coords)

# 保存合并后的多边形或 MultiPolygon
merged_gdf = gpd.GeoDataFrame(geometry=[merged_geometry],crs=gdf.crs)

# 如果 CRS 未定义，请手动设置 CRS（如果你知道正确的CRS）
merged_gdf.set_crs('EPSG:4544', allow_override=True)
# 检查合并后的 CRS
print("Merged CRS:", merged_gdf.crs)

# 获取文件名（不带路径和扩展名）
file_name = os.path.splitext(os.path.basename(shapefile_path))[0]

# 将文件名添加为新的属性列
merged_gdf['file_name'] = file_name
output_file_name = f"{file_name}_淹没范围.shp"




# 获取当前运行的目录
current_directory = os.getcwd()

# 指定你想要检查的文件名


# 构造文件的完整路径
file_path = os.path.join(current_directory, output_file_name)

# 判断文件是否存在
if os.path.exists(file_path):
    print(f"文件 '{output_file_name}' 存在于当前目录：{current_directory}")
else:
    #print(f"文件 '{output_file_name}' 不存在于当前目录：{current_directory}")
    # 保存为新的Shapefile文件
    #merged_gdf.to_file("merged_polygon.shp")
    merged_gdf.to_file(output_file_name)

    # 打印合并后的多边形的几何信息
    print("Merged Polygon Geometry finished!",output_file_name)#将shp边界输出，即建模范围

```












