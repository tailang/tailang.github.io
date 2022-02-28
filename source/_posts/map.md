---
title: 骑行路线数据采集小工具
date: 2021-08-14 17:26:08
categories:
    - tech
tags:
    - web
---
2011年开始喜欢上了骑行，到今年（2021年）刚好10年🎉。在这段时间内估摸着已经骑了10000+公里的路，由于早期没有用GPS码表、智能手表或运动App，导致之前的骑行路线没有留下数据（经纬度等）。  

所以写了一个简单的小工具，通过手动采集数据（经纬度、海拔、总距离等）复原曾经的线路。

### 项目地址
项目地址：https://github.com/tailang/RiderMap
RiderMap是一个采集骑行路线数据，并能导出GPX或GeoJson文件的小工具。

### 如何使用


1、注意项

（1）项目中使用的地图数据来自MapBox，这是一个商业地图服务商，对应free用户有很多请求的限制，比如Map Loads for Web每月支持50000次，Tilequery Api每月支持100000次、每分钟最多请求600次等限制。所以推荐注册一个自己的MapBox账号并生成对应的token，替换项目中的token



（2）采集到的经纬度数据是采用WGS-84标准，也就是大家常说的”地球坐标“，而国内使用的地图，如高德，苹果中国，谷歌中国等一般使用GCJ-02标准（”火星坐标“，在WGS-84上做了一次加密），百度地图使用BD-09标准（在GCJ-02上又做了一次加密）。为什么这么做？应该是出于国家安全考虑吧。所以如果将导出的gpx或geoJson文件用国内地图加载打开，会出现偏差，需要通过一些网上工具进行纠偏处理。



（3）又是出于国家安全，国内的地图现在不支持海拔查询，所以这个小工具采用了mapbox的瓦片api获取海拔数据，这个海拔值只是一个参考值，不一定准确



2、clone项目，找到rideRoute.js文件中的mapbox_token替换成自己的token
```
const mapbox_token = '替换成你的token'
```

3、找到rideRoute.html文件,点击使用浏览器打开就可以开始采集数据了。



### 一个🌰

下图是使用小工具采集的我2014年环太湖的数据，并导出了GPX和GeoJson文件
![tool](/images/map_tool.png)  
导出的GPX或GeoJson文件可以通过一些软件方便的反复查看您的骑行路线

下图是通过VSCode的Geo data viewer插件打开的geoJson文件
![geojson](/images/map_geo.png)  

下图是通过”风日影踪“App 打开的GPX文件  
![gpx](/images/map_gpx.png)