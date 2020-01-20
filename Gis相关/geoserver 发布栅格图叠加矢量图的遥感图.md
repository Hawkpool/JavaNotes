在完成基础的矢量文件图层叠加图层组功能成功发布后,我们还需要使用shp文件与tif文件共同形成遥感地图进行发布
>在参考了大量网络教程后,目前还是只实现了shp文件从数据库读取,而tif栅格文件需要从文件目录一个个读取

遥感图的国家统一坐标系 是 CSCG2000/EPSG:4546 ,所以,为了保证shp文件与tif文件能一起展示,必须要保证shp文件和tif文件的坐标系是一致的 !!!

geoserver从Postgis中读取shp文件可以参考[上一篇文章](https://hacpai.com/article/1578378438066)

geoserver发布tif文件教程如下
geoserver默认支持读取tif栅格文件,但是文件必须在geoserver服务器本地目录下
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500857-1579501082275.png)![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500858-1579501084157.png)![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500859-1579501085756.png)

此处的路径是我在docker-compose文件中共享的文件目录,所以docker中的服务可以访问
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500860-1579501140235.png)![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500861-1579501141749.png)

配置完成后即可在Layer Preview中查看到显示效果
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500862-1579501155681.png)

再配置图层组与矢量网格图叠加

![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500863-1579501169727.png)
看到那个小黑点了吗,那就是我们的栅格图...

![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500864-1579501177874.png)
至此,基本的图层叠加已经完成,但是....看这密密麻麻的格子,总得有个批量导入栅格图的途径吧

原本是想要统一数据源,让shp和tif数据都从数据库读取,降低服务器存储空间的压力
在参考文档[GeoServer发布PostGIS数据库中的栅格数据](https://www.jianshu.com/p/ef9e37f0aed8)后
经过大量尝试,虽然tif文件入库成功,geoserver的imageMosaicJdbc插件也成功安装,但是在使用imageMosaicJdbc读取数据库数据的时候却一直无法连接,并一直报错
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500865-1579501198090.png)
经过问题排查和参数检查后,确认不是配置的问题,在百度和google之后终于在一个官方错误报告邮件记录上找到类似的错误
[Image Mosaic failing to create reader again](http://osgeo-org.1560.x6.nabble.com/Image-Mosaic-failing-to-create-reader-again-td5365868.html)
结果发现
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500866-1579501213342.png)
大致意思是...有10多年历史的设计错误,改起来很麻烦..所以就没改了.........
总而言之,使用插件的方式基本是宣告失败

但是栅格图的发布总不能真的一个一个的通过UI界面去操作吧
经过大量的搜索以及资料翻阅,终于在csdn上找到了一个号称可以操作[geoserver批量导入栅格图](https://download.csdn.net/download/qq_36178899/10560702)的Java项目代码

该项目下载下来后,能直接在idea中打开,但是却报了一个依赖错误
总的来说就是找不到这个依赖的jar
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500867-1579501300767.png)

经过修改Maven版本,重新导包,重新下包等一系列操作后发现问题根深蒂固,依旧存在,于是我决定去Maven仓库里面一探究竟
果然,依赖的jar包没有下载下来,只有几个lastupdated文件

![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500868-1579501313490.png)


这就好办了,去maven官网找爹去
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500869-1579501322009.png)

结果一找, 又懵逼了, 人家maven只有1.7的版本,还是16年发布的,现在都2020年了啊 !!!
我抱着死马当活马医的态度试着测试修改版本号,依旧不行

好吧, 那我找[geoserver-manager的仓库](https://repo.boundlessgeo.com/main/it/geosolutions/geoserver-manager/)去!!
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500870-1579501337728.png)
乍一看好像来对地方了,但事实却是十分残酷的
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500871-1579501348899.png)![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500872-1579501350069.png)

Maven仓库里的文件夹仍在,文件也能看到,但就是下载不了 !!!

但我不信, 一定还有其他解决方案, google一番后,发现github上好像有一个[同名的开源项目](https://github.com/geosolutions-it/geoserver-manager)

下载下来后,经过一键Maven构建打包四连后
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500873-1579501373802.png)
果然在仓库里出现了想要的画面
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500874-1579501380661.png)

回去一看批量导入项目的依赖问题也解决了

仔细一看这套批量导入的项目代码,其实就是调用了geoserver的rest接口
通过代码实现了UI界面上的一套操作
>创建工作区---轮询文件目录,创建数据源---发布图层

通过修改项目中的相关参数,成功实现了轮询本地目录读取tif文件,发布栅格图的功能
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/20/1579500875-1579501400881.png)

其中存在的问题:
* 该项目读取的目录与geoserver服务器需要是同一目录, 因此导致无法远程操作linux服务器下的geoserver批量发布
* 该项目没有一套成型的web操作界面,只能通过开发工具右键run java运行

目前来讲较好的解决方案: 
* 使用带有GUI操作界面的Linux服务器,并安装IDEA开发环境,方便操作栅格批量发布,及文件传输
* 使用windows server服务器发布geoserver,并安装IDEA开发环境,方便操作栅格批量发布,及文件传输



