# 瓦片地图服务解析
![瓦片地图](https://i.loli.net/2017/11/16/5a0d8bee7805e.jpg
 "瓦片地图示意图")  
目前，在线地图服务商均采用瓦片题图的方式提供在线地图服务。瓦片地图(Tile Map)金字塔模型是一种多分辨率层次模型，从瓦片金字塔的底层到顶层，分辨率越来越低，但表示的地理范围不变。也就是采用分块的思想，将不同分辨率的地图均分成大小相同的块，这样可以大大降低我们加载地图时的数据量，是一种非常普及的题目服务模式。  
同时，我们认为该方式组织的地图另一大优势之一就是“自带”**定位属性**，瓦片地图服务(Tile Map Servce)采用墨卡托投影的方式，因此我们只需要知道目标位置的GPS坐标(WGS-84标准)，通过墨卡托投影变换，在根据给定的瓦片地图缩放级别，就可以确定目标位置位于当前缩放级别的哪一块地图的哪个位置之上，进而完成定位。  
下面，我们将就谷歌地图和高德地图做简要的介绍。

## 谷歌地图瓦片服务
![谷歌地图瓦片](https://i.loli.net/2017/11/16/5a0d8dc8be579.png "谷歌地图瓦片示意图")  
如图所示，可以看出当前缩放级别为1，整张地图由四个瓦片构成，其x和y坐标分别为(0,0),(0,1),(1,0),(1,1)。也就是说，我们假设当前缩放级别为z，那么该缩放级别下x方向和y方向上瓦片地图的数目均为$$2^z$$个，而整张地图则是以本初子午线和赤道的交点作为零坐标点，结合墨卡托投影的相关知识，我们就可以得出以下的经纬度与瓦片地图编号互转的公式：  
设瓦片地图坐标为(x,y,z),瓦片地图内像素坐标为(m,n),经纬度为$$(\lambda,\psi)$$,则有：  
$$\lambda=[2^{1-z}\cdotp(x+\dfrac{m}{256})-1]\cdotp180$$  
$$\psi=\dfrac{\arctan(e^{[1-2^{1-z}\cdotp(y+\dfrac{n}{256})]\cdotp\pi})}{\pi}-90$$  

$$x=2^{z-1}\cdotp(\dfrac{\lambda}{180}+1)$$  
$$y=2^{z-1}\cdotp(1-\dfrac{\ln{\tan(\dfrac{\pi\cdotp\psi}{180})+\sec(\dfrac{\pi\cdotp\psi}{180})}}{\pi})$$

## 高德瓦片地图服务
其实，国内各个在线地图服务提供商的瓦片地图规则和谷歌瓦片地图规则非常相似，但是有一点不同的就是经纬度标准从国际通用的WGS-84坐标改为了GCJ-02坐标系（包括国内的谷歌地图也采用了该坐标系，但是百度地图除外，百度地图在GCJ-02坐标系之上再度进行了一次加密，通常称之为BD-09坐标系）。GCJ-02坐标系从本质上来说就是基于国际通用WGS-84坐标系进行了一次非线性加偏处理之后得到的，GCJ-02是由国家测绘局开发出来的标准，要求所有国内地图提供商都必须采用该标准，而该算法的具体实现是保密的。也就是说地图数据提供商需要向国家测绘局购买相应的加偏算法对原始数据进行加偏，然后类似导航仪以及百度等厂商则要购买解密算法来消除偏差。当然，在本项目中，我们使用了网上开源的加密解密算法，目前来看误差很小，完全在可接受范围内。
