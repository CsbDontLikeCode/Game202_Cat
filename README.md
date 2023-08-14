# Game202_Cat
Game202 Coursework

实时阴影
    
-Shadow Mapping,阴影贴图。
    A 2-Pass Algorithm。先渲染点光源视角的深度缓冲贴图，再利用光源深度缓冲贴图来对视角可见的像素进行阴影判断。
    存在的问题:自遮挡(阴影纹路)和阴影边缘锯齿问题(深度缓冲贴图与可视空间的像素点无法做到完全适配)。
    问题解决方案:自遮挡(增加偏移量以减少自遮挡/Second-depth shadow mapping);阴影边缘锯齿(增大深度缓冲贴图分辨率，PCF百分比逼近滤波)。
    Second-depth shadow mapping,使用阴影贴图的第一深度和第二深度的中间点作为实际输出深度。实际上没有人用该方法:所有物体必须是"物体"而非"面"，此外计算第一与第二深度需要额外计算开销影响实时渲染性能。
    PCF(Percent Closer Filtering),其具体实现是在对点进行阴影判断时，同时考虑点周边对应的阴影缓冲点的值，并加权求平均。

-PCSS,百分比逼近软阴影。
    软阴影相比于硬阴影，存在着从光明过渡到阴影的特性。这是因为光源往往不是单纯的一个点，而是面，因而受光源形状的影像，物体的阴影往往是软阴影。
    应用PCF技术，并将滤波器的范围调的很大，就可以实现软阴影了，惊不惊喜，意不意外？
    一个物体的软阴影的程度取决于阴影与物体的距离，而软阴影的程度又直接取决于滤波尺寸。
    
    ![Image text](https://github.com/CsbDontLikeCode/Game202_Cat/blob/main/homework0/images/PCSS.png) 