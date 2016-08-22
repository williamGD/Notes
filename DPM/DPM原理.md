# Filter channels Features

>Zhang, Shanshan, Rodrigo Benenson, and Bernt Schiele. "Filtered feature channels for pedestrian detection." Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 2015.

# DPM

>Object detection with discriminatively trained partbased models. IEEE Trans. PAMI, 32(9):1627–1645, 2010.

## DPM feature
![Alt text](.\Improve_HOG_byDPM.jpg "DPM")

1、8x8 Cell单元的概念；

2、归一化同周围组成的区域进行归一化；

3、计算梯度使用无符号（0-180°）和有符号（0-360°）结合策略，如果全部计算会达到 4x(9+18)=108 的维度；

4、提取无符号梯度，完整的cell会有4x9=36 维特征，进行近似PCA降维，前11个基本包含所有信息，仅仅取前9维，然后对4x9=36的矩阵，进行列的求和得到9个无符号梯度特征。 提取有符号梯度则进行列累加，有18个特征。还有对有符号梯度进行行的累加，有4个特征。最终获得9+18+4=31维特征。

## DPM model
 
![Alt text](.\DPMpedestrainmodel1.png "rootmodel title")
![Alt text](.\DPMpedestrainmodel2.png "partmodel title")
![Alt text](.\DPMpedestrainmodel3.png "partmodel cost title")

根模型，部件模型以及部件偏离损失。

## DPM detection

![Alt text](.\DPMdetectionprocess.png "DPM detection")

单一尺度下的检测流程

- 某一位置(x,y)与根模型/部件模型的响应得分，为该模型与该位置为锚点(左上角坐标)的子窗口区域内特征的内积，响应为特征与待匹配模型的相似程度；

- 响应变换：以锚点为参考位置，综合部件模型与特征的匹配程度和部件模型相对理想位置的偏离损失，得到的最优的部件模型位置和响应得分；

- 部件模型分辨率是根模型分辨率的1倍，因此部件模型需要在尺度层进行匹配考虑，即将锚点放大一倍。


DPM 速度慢，改进过的加速的算法有

>30Hz Object Detection with DPM V5