# Game202_Cat
Game202 Coursework

## 实时阴影
---  
### Shadow Mapping(阴影贴图)
一种2-Pass Algorithm。先渲染点光源视角的深度缓冲贴图，再利用光源深度缓冲贴图来对视角可见的像素进行阴影判断。<br>
存在的问题:自遮挡(阴影纹路)和阴影边缘锯齿问题(深度缓冲贴图与可视空间的像素点无法做到完全适配)。<br>
问题解决方案:自遮挡(增加偏移量以减少自遮挡/Second-depth shadow mapping);阴影边缘锯齿(增大深度缓冲贴图分辨率，PCF百分比逼近滤波)。<br>
Second-depth shadow mapping,使用阴影贴图的第一深度和第二深度的中间点作为实际输出深度。实际上没有人用该方法:所有物体必须是"物体"而非"面"，此外计算第一与第二深度需要额外计算开销影响实时渲染性能。<br>
PCF(Percent Closer Filtering),其具体实现是在对点进行阴影判断时，同时考虑点周边对应的阴影缓冲点的值，并加权求平均。<br>
### PCSS(百分比逼近软阴影)
软阴影相比于硬阴影，存在着从光明过渡到阴影的特性。这是因为光源往往不是单纯的一个点，而是面，因而受光源形状的影像，物体的阴影往往是软阴影。<br>
应用PCF技术，并将滤波器的范围调的很大，就可以实现软阴影了，惊不惊喜，意不意外？<br>
一个物体的软阴影的程度取决于阴影与物体的距离，而软阴影的程度又直接取决于滤波尺寸。<br>
<div align = center>
<img src="https://github.com/CsbDontLikeCode/Game202_Cat/blob/main/homework0/images/PCSS.png" width="240">
</div>
滤波尺寸计算方法: 滤波尺寸 = (光源到阴影点的距离 - 光源到遮挡点的距离) * 光源的宽度 / 光源到遮挡点的距离<br>
PCSS中生成深度缓冲的具体实现仍是以广源点为视点生成，只不过在判定阴影点时额外加入了光源宽度的参数。<br>
图画得不是很直观，光源到Block的深度可以直接在深度缓冲上查到，而阴影点与光源的距离也可以直接计算得出，没必要加垂直的虚线辅助理解(反倒增加了理解的成本)。<br>
完整的PCSS实现流程：<br>
1.Blocker search，getting the average blocker depth to determine filter size.<br>
2.Penumbra estimation(use the average blocker depth to determine filter size).<br>
3.Percentage Closer Filtering.<br>
<div align = center>
<img src="https://github.com/CsbDontLikeCode/Game202_Cat/blob/main/homework0/images/BlockerChosenRange.png" width="360">
</div>

### VSSM(Variance Soft Shadow Mapping)
VSSM的出现是为了应对PCSS实现流程中Blocker search和PCF操作所造成的极大性能消耗。这两个操作都需要进行滤波计算，需要消耗很大的计算性能。而VSSM能快速Blocker search和滤波。<br>
VSSM的做法：在计算阴影点滤波时，通过正态分布近似估计出概率结果。获取正态分布只需要得到均值和方差。<br>
如何获取均值呢？Mipmap，它在生成的时候就有作均值采样，很合适。但Mipmap也存在问题就是只适合正方形区域求解，而对于长方形区域，一种更好的方法是Summed Area Table(SAT)。<br>
如何获取方差呢？ Var(X) = E(X²) - E²(X); 此乃概率论公式，咋推导呢？我不到哇。<br>
紧接着又有个新问题：深度缓冲如何同时存下区域内的均值与方差呢？一张纹理图最多可以RGBA四个波段的信息，我用R,G波段分别存均值与方差，这河狸吗？这恒河狸。<br>
均值和方差有了，随之而来的问题是：要怎么通过均值和方法求出阴影点被遮挡的概率呢？由均值和方差可以推导出正态分布的概率密度函数(PDF),由概率密度函数又可获得累计分布函数(CDF)。而后，就可以通过累计分布函数得到概率的数值解,数值解可以调C++库直接计算。正儿八经去计算累计分布函数的数值解也多少有点费劲，不如直接试试切比雪夫不等式吧，反正就将就着获得一个解，切比雪夫不等式虽然不一定准，但是计算是真的快啊！<br>
切比雪夫不等式：P(x>t) <= σ² / (σ² + (t - μ)²) <br>
VSSM每个阴影点的判断运行时间复杂度都是O(1)，因为只需要查询深度缓冲上的一个点。<br>
以上方法仅仅能用于PCF阶段，而对于Blocker search,还需要用其他方法(没理解说的方法)。<br>
实话实说，看完了也没太整明白具体要怎么实现......算了，就这样吧先。<br>
我还有个想法：既然阴影点只是想要其在深度缓冲中对应的滤波的大概情况， 其实也没必要把整个滤波所包含的深度缓冲像素点全部考虑在内吧？象征性的间隔采样几下就OK了吧？减少一半的采样点，得到的结果大差不差，但计算开销却能减半。<br>
#### Summed Area Table(SAT) for Range Query
<div align = center>
<img src="https://github.com/CsbDontLikeCode/Game202_Cat/blob/main/homework0/images/SAT in 2D.png" width="720">
</div>

### Moment Shadow Mapping
在VSSM中，深度分布描述得不准确(因为都是基于假设的)。MSM是为了应对"深度信息描述不准确"这一问题而被提出的应对方法。<br>
VSSM是基于切比雪夫不等式来推断PCF的，通过记录前N阶矩(前四阶矩往往就能取得很好的效果)来最大限度的保留PCF的真实情况而非依靠纯粹的假设。<br>
<div align = center>
<img src="https://github.com/CsbDontLikeCode/Game202_Cat/blob/main/homework0/images/MSM.png" width="540">
</div>