![](https://img.hacpai.com/bing/20190511.jpg?imageView2/1/w/960/h/540/interlace/1/q/100)

我是革命一块砖，哪里需要往哪搬！身为全栈做的活挺多了，不过公司最近要用 geoserver 发布地图服务，又要开始了一个陌生的领域的探索。

因为缺少经费，且需求暂不明确，暂且不考虑需要付费但功能强大的**ArcGis**,于是参考了[开源WebGIS架构](https://blog.csdn.net/lijie45655/article/details/89606143)

![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/image-65002c25-1579498253988.png)

> 经前端后端 ui 以及 cto 的联合商讨，大致确定
>
> * 由 ui 使用 qgis/arcgis 出图，出 shp 文件
> * 后端搭建**PostgreSQL/PostGIS** + **GeoServer** 处理 shp 文件入库及发布
> * 前端使用**Openlayers**进行地图展示

Geoserver 是一款开源免费的地图服务器，功能十分强大。或许我们会碰到这样一个场景，工作在内网下，不能使用外网的天地图资源，这时我们只能把需要的地图下载下来用 geoserver 发布了。然而对于多层级的数据地图数据发布方案不是很明确。我们用一些地图下载器下载的资源一般可以分为三种吧：大图拼接(.tif)格式，各种规范的瓦片，各种规范的瓦片包。

## 环境配置： docker 一键启动

Docker 是个好东西，有了它环境问题再也不用愁了~~
此处奉上使用 docker-compose 编排的一键启动脚本：

> 包含 **PostSql+PostGis+GeoServer+pgadmin(数据库图形化管理界面 Web 版)**

```
version: '3.1'  
services:  
 geoserver:  
 restart: always  
 image: kartoza/geoserver  
 container_name: geoserver  
 ports:  
 - 8888:8080  
 volumes:  
 - ./geoserver-data:/opt/geoserver/data_dir
  
 postgis:  
 restart: always  
 image: kartoza/postgis  
 container_name: postgis  
 ports:  
 - 5432:5432  
 environment:  
 POSTGRES_USER: hawk  
 POSTGRES_PASSWORD: 123456  
 volumes:  
 - ./postgis-data:/var/lib/postgresql/data  
  
 pgadmin4:   
 restart: always  
 image: dpage/pgadmin4  
 container_name: pgadmin  
 ports:   
 - 8090:80  
 environment:  
 PGADMIN_DEFAULT_EMAIL: hawk@XXXX.com  
 PGADMIN_DEFAULT_PASSWORD: 123456  
    
```

## 后端需要做的事主要包括

### 配置整体 GIS 环境(一键启动即可)

### 从 ui 获取 shp 文件，操作入库(也可让 ui 自行操作)

可以通过 **PostGis** 文件导入工具 导入 **PostSql**
![image.png](https://img.hacpai.com/file/2020/01/image-6d5eb642.png)

> 切记
>
> * shp 文件存放目录尽量简单，不要包含中文(否则容易报错)
> * 配置编码正确格式，包含中文的话尽量使用 UTF-8 或者 GBK
> * **shp 文件同时还有 dbf 文件、prj 等文件考到同一个目录下，名字要一致**
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/image-6d5eb642-1579498292071.png)
>   ![image.png](https://img.hacpai.com/file/2020/01/image-f1e62604.png)
>   **该工具需要 windows 本地安装 PostSql ,并安装 PostGis 方可链接数据库**
给大家分享一个 postgis-ui 的下载链接，使用这个不需要安装 pgAdmin  
链接：[下载链接](https://pan.baidu.com/s/1niJYnaZndn4Yy_7nXbTfHA) 密码：l1ml

### 配置 GeoServer

浏览器打开：`http://docker宿主机ip:8888/geoserver`

> 使用默认 admin 账户登录： admin/geoserver

#### 配置工作区

![image.png](https://img.hacpai.com/file/2020/01/image-946576f9.png)

#### 配置数据存储

geoserver 原生支持识别 shp 文件，但是仅支持事变本机目录文件，不支持远程上传
遂通过 postsql 导入，实现更佳的可用性
![image.png](https://img.hacpai.com/file/2020/01/image-96cb3957.png)
![image.png](https://img.hacpai.com/file/2020/01/image-643776df.png)
完成 PostGIS 配置后，会自动跳转到图层配置

#### 配置图层

数据库中的数据就是地图的矢量数据
图层有点像 ps 里的一个图层，通过解码对应坐标系解析矢量数据展示相应图像
图层组就像是 ps 中多个图层组合展示的画面
![image.png](https://img.hacpai.com/file/2020/01/image-138a7614.png)

##### 新增图层操作

先配置图层数据库，GeoServer 会自动加载数据库已上传的 shp 文件，即可选中相应图层点击发布
![image.png](https://img.hacpai.com/file/2020/01/image-f253a50f.png)
配置坐标系系数，计算边框，渲染地图
![image.png](https://img.hacpai.com/file/2020/01/image-b79cd416.png)
![image.png](https://img.hacpai.com/file/2020/01/image-6e0f3034.png)
保存成功后即可查看预览图

#### 查看图层预览图

![image.png](https://img.hacpai.com/file/2020/01/image-7ab3915e.png)
![image.png](https://img.hacpai.com/file/2020/01/image-0b666f71.png)
点击 OpenLayers 后会打开一个新的页面
![image.png](https://img.hacpai.com/file/2020/01/image-7161a893.png)

一个地图边框图层预览，GET!

#### 配置图层组

我们再通过上面操作配置一个点状图图层预览
![image.png](https://img.hacpai.com/file/2020/01/image-b4850e2c.png)
然后我们就可以配置图层组了
![image.png](https://img.hacpai.com/file/2020/01/image-f8f0c5bc.png)
![image.png](https://img.hacpai.com/file/2020/01/image-ade21549.png)
![image.png](https://img.hacpai.com/file/2020/01/image-5e923909.png)
![image.png](https://img.hacpai.com/file/2020/01/image-2005ac96.png)

完成后即可保存查看预览效果
![image.png](https://img.hacpai.com/file/2020/01/image-1399dd0e.png)

[windows相关环境及文件](https://pan.baidu.com/s/1P4M_uxy04c34VJuoLImepA)    提取码: arne

