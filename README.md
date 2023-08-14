# Game202_Cat
Game202 Coursework

## 实时阴影
    
### -Shadow Mapping,阴影贴图。
A 2-Pass Algorithm。先渲染点光源视角的深度缓冲贴图，再利用光源深度缓冲贴图来对视角可见的像素进行阴影判断。<br>
存在的问题:自遮挡(阴影纹路)和阴影边缘锯齿问题(深度缓冲贴图与可视空间的像素点无法做到完全适配)。<br>
问题解决方案:自遮挡(增加偏移量以减少自遮挡/Second-depth shadow mapping);阴影边缘锯齿(增大深度缓冲贴图分辨率，PCF百分比逼近滤波)。<br>
Second-depth shadow mapping,使用阴影贴图的第一深度和第二深度的中间点作为实际输出深度。实际上没有人用该方法:所有物体必须是"物体"而非"面"，此外计算第一与第二深度需要额外计算开销影响实时渲染性能。<br>
PCF(Percent Closer Filtering),其具体实现是在对点进行阴影判断时，同时考虑点周边对应的阴影缓冲点的值，并加权求平均。<br>
### -PCSS,百分比逼近软阴影。
软阴影相比于硬阴影，存在着从光明过渡到阴影的特性。这是因为光源往往不是单纯的一个点，而是面，因而受光源形状的影像，物体的阴影往往是软阴影。<br>
应用PCF技术，并将滤波器的范围调的很大，就可以实现软阴影了，惊不惊喜，意不意外？<br>
一个物体的软阴影的程度取决于阴影与物体的距离，而软阴影的程度又直接取决于滤波尺寸。<br>
<div align = center>
<img src="https://github.com/BIT-MJY/Active-SLAM-Based-on-Information-Theory/blob/master/img/1-2.png" width="180">
</div>
![PCSS](https://github.com/CsbDontLikeCode/Game202_Cat/blob/main/homework0/images/PCSS.png)  <br>
滤波尺寸计算方法: 滤波尺寸 = (光源到阴影点的距离 - 光源到遮挡点的距离) * 光源的宽度 / 光源到遮挡点的距离<br>
PCSS中生成深度缓冲的具体实现仍是以广源点为视点生成，只不过在判定阴影点时额外加入了光源宽度的参数。<br>
图画得不是很直观，光源到Block的深度可以直接在深度缓冲上查到，而阴影点与光源的距离也可以直接计算得出，没必要加垂直的虚线辅助理解(反倒增加了理解的成本)。<br>
完整的PCSS实现流程：<br>
1.Blocker search，getting the average blocker depth to determine filter size.好像是说选取一定范围内的深度缓冲纹理值求平均以计算出Blocker到光源的大概距离？？？Blocker范围选取可以根据光源面与阴影点的距离动态调节，但是怎么动态调节我没太整明白。<br>
2.Penumbra estimation(use the average blocker depth to determine filter size).<br>
3.Percentage Closer Filtering.<br>
<div align = center>
<img src="https://github.com/CsbDontLikeCode/Game202_Cat/blob/main/homework0/images/BlockerChosenRange.png" width="180" height="105">
</div>