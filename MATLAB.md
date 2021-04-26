# 脚本 .m
* BianBuChang.m
>**步长加速法**  
&emsp;优化倾侧角的计算，不断迭代循环优化倾侧角，*变步长目前看不出来那里变?*，输出：
```m
倾侧角转折点优化结果：
gamma_v_mid                   V2
1:  20.000000          166.119325
2:  20.009000          166.108682
3:  20.027000          166.087395
4:  20.054000          166.055461
5:  20.090000          166.012878
6:  20.135000          165.959639
...
164:  82.224144          63.006891
165:  82.224144          63.006891
166:  82.225129          63.006891
167:  82.225129          63.006891
168:  82.225129          63.006891
```

* CAVH.m
>**CAV三自由度运动方程**  
&emsp;开头定义了模型的初始变量，例如升力系数、阻力系数的定义，升力阻力的计算等。采用龙格库塔法求解各个状态变量。最后得出各个状态变量关于t的时间曲线。


* CoordinateConvert.m
>**坐标之间的转换**  
&emsp;定义了一些坐标转换的关系，但是只定义了一些角度，*目前不知道什么意思?*  
```m
地面系与弹体系
地面系与弹道系
地面系与速度系
弹体系与速度系
速度系与弹道系
弹体系与弹道系
```
&emsp;注意.m开头没有`clear all`语句

* drawData.m
>**绘制数据的文件**  
&emsp;其中含有`flag` 、`Scale`这个全局变量，`flag`决定绘图的方式，`Scale`决定大小。开头先关闭了figure2~7，然后读取数据文件，位于`//data`文件夹中的：  



| 变量        | 变量名    |  数据来源  |
| :--------:   | :-----:   | ---- |
| 时间        | data_time      |   data\\time.txt    |
| 角度        | data_direction      |   data\\direction.txt    |
| 距离        | data_distance      |   data\\distance.txt    |
| 势场力        | data_potential      |   data\\potential.txt   |
| 速度        |data_speed      |   data\\speed.txt    |
| 位置        | data_position      |   data\\position.txt    |

&emsp;最后重画figure1~7

* drawMoveObstacle.m
>**绘制移动的障碍的文件**  
&emsp;
* drawYuan.m
>**绘制一个半圆曲面**  
&emsp;*目前还不知道什么用途，可能是测试文件？*

* LineNH.m
>**剩余航程信息**  
&emsp;*目前还不知道什么用途，得先弄懂生成数据的.m文件？*

* model.m
>**主函数**  
&emsp;开头定义全局变量 `Scale` `map` `d2r` `r2d` `quadSize` `flag`  
&emsp;使用golbal的优点：
* 传递大数据的参数
如果通过函数传参数的方式的话，系统会浪费过多的时间在复制数据的时间上，如果采用global的方式共享数据的话代码的效率会大大提高
*  过多的常量需要传递
如果每个量都作为函数函数的参数传递的话，代码参数列表就很长，如果采用global的话代码的可读性提高，函数调用也方便。

&emsp;主函数的流程  
1. 读取`data\\obstacleData.txt`的敌方防御信息，绘制障碍物地图
0. 初始化参数，如人工势场法的相关参数；并写入文本文件`data\\potential.txt` `data\\distance.txt` `data\\direction.txt` `data\\speed.txt` `data\\time.txt` `data\\position.txt` ，文件以及相关表头被创建。
0. 代入人工势场法，计算距离并进行路径规划，循环的终止条件为目标与飞行器的距离在接收范围内。①计算目标位置的距离。②障碍物检测，人工势场法的引力、各方向斥力以及合力的计算。③同时控制飞行的方向角，保证飞行平稳④控制飞行的速度，同时计算得到飞行器在地图上新的位置。⑤将上述计算过程的量写入对应文件中
0. 计算程序运行时间，是否符合在线规划的标准，运行`drawData`脚本，画出运行轨迹


* Trajectory.m
>**主函数，``model.m``是四旋翼版本的人工势场，`Trajectory.m`是高超的版本**  

更改的地方：
1. 人工势场法（APF）加入了伪目标来消除陷入局部极小值的问题
0. 龙格库塔四阶方法求解Y中的各个状态
0.  添加了目标以及障碍物会移动的部分，属于**动态环境**

与论文中对应的参数定义：
| 变量        | 变量值    |  定义  |
| :--------:   | :-----:   | ---- |
| 地图 map        | 3维矩阵  200 * 200 * 100     |   null    |
| 攻角 alpha_max        | 单值 20      |  °    |
| 攻角         alpha_maxLD        | 单值 10      |  °    |
| 无量纲剩余航程    s0    | 单值       |  根据公式（3.2）    |
| 伪平衡滑翔边界对应的倾侧角        | 单值   |  根据公式（2.44）    |
gamma_v_EQ1  初始的| 单值   |  由初始状态速度、高度   |
gamma_v_EQ2  末端的| 单值   |  由末端状态速度、高度    |
gamma_v_mid  中间值| 单值   |  优化而来    |
gamma_v_zf  带符号的偏航角 | 单值 | 由APF函数根据升力与侧立计算而来
gamma_v 经过高度平滑处理的σ | 单值 | **经过高度平滑处理？**
gamma gamma_v的集合 |数组 ，与Y一致 | 
totalPotential 侧向力合力 | 数组，数量比t大1 | 由APF函数输出给值
attPotential 引力侧向力 |数组，数量比t大1 | 由APF函数输出给值
repPotential 斥力侧向力  |数组，数量比t大1 | 由APF函数输出给值
distance_all_true 飞行器的真是航程| 单值 | 由经纬度直接计算，公式 SNL 报告（2.1）
distance_all 理论剩余航程 | 单值 | 直接距离，在APF函数中使用
tt **整个过程的无量纲时间？** | 数组，与Y大小相同 | 根据时间`t`以及步长`h`







经纬度的处理：
在`obstacleData.txt`文件中，经度除于2，纬度除于4，这样处理后，才能axis ：[200 200 100],可不可以不这么处理  
但是goal[]里面的坐标并没有这么处理，还是原来的坐标

* v2d.m
>**坐标变换的各种矩阵** 

# 函数 fx
* Ackley.m
* APF.m
人工势场法
```matlab
function [angle1,attPotential_2,repPotential_2,totalPotential2,pathFound,gamma_v_zf]
= APF(Y,obstaclePosition,FL,distance_all,GoalFlag,goal)%,xbest)
...
end
```
输入参数：
1. Y：状态变量
0. obstaclePosition：障碍物位置
0. FL： 当前高度速度计算下的升力
0. distance_all：理论航程大小
0. GoalFlag：陷入局部最小判断位置，为`false`时将目标位置做小偏移，设置伪目标点
0. goal：目标点的坐标

中间变量：
| 变量        | 变量值    |  定义  |
| :--------:   | :-----:   | ---- |
| d0 静态斥力作用的最大距离   | 1000e3   | null |
| dv 动态斥力作用的最大距离   | 2000e3   | null |
| distanceGoal 高超与目标的距离   | 单值   | null |
distanceGoal_xy 高超与目标的距离，不计高度 | 单值   | null |
| distance 高超与各个障碍物之间的距离   | 根据障碍物个数   | 查看函数obstacleFinder |
| direction 高超与各个障碍物之间的角度   | 根据障碍物个数   | 查看函数obstacleFinder |
repPotential_temp 斥力的大小  | 根据障碍物个数 | 公式（3.16） Frep1（**但是有个次方不大对，4还是2**）
repPotential_temp1 障碍物斥力在高超的侧向力  | 单值 是repPotential_temp、psi_v等的表达式 | 公式 乘上航向角 **+/-？**
repPotential_1 障碍物斥力在高超的侧向力之和 | 单值 | repPotential_temp1带符号累加
moveObs 动态禁飞区标志位 | 根据障碍物的个数 bool型 | 0/1
angleGoal 高超与目标的角度计算 | 1*2 | ①XOY平面上:与X轴夹角 ②与XOY的夹角
attPotential_temp 引力计算 |单值|公式（4.8），如果是动目标点，则有附件引力，公式（4.9）
attPotential_1 引力乘上航向角| 单值 | 由attPotential_temp、angleGoal决定
totalPotential_1 合力计算 | 单值 | 由侧向的引力和斥力加法
gamma_v_zf 带符号的偏航角 | 介于10~81°之间 | 由totalPotential_1/FL决定


输出值：
1. angle1：angleGoal的第一个值，为与x的夹角
0. pathFound：为true时表示已经抵达终端
0. totalPotential2：=totalPotential_1 侧向力的合力计算
0. attPotential_2： = attPotential_1，引力侧向力的值
0. repPotential_2：= repPotential_1，斥力侧向力的值
0. gamma_v_zf：gamma_v_zf 带符号的偏航角值

* APF1.m
* APF2.m
* bas.m
* BBO.m
* ClearDups.m
* ComputeAveCost.m
* Conclude.m
* createMap.m
* define_yuanzhu.m
* drawObstacle.m
```matlab
function drawObstacle(obstaclePosition,goal)
...
end
```
输入参数：
1. obstaclePosition 通过读取`obstacleData.txt`文件得到，矩阵形式，共6列，每列的参数分别为障碍物的：经度坐标X，纬度Y，高度H（圆球区域为0），半径R，圆柱/球的判断位（值大小的影响目前还不知道）

2. goal:目标的三维坐标值

输出：画出障碍物figure

* dtCAV.m
* dtCAV1.m
>**CAV三自由度运动方程**  
```matlab
    function dY=dtCAV1(~,Y)
    ...
    end
```
&emsp;使用function函数句柄，建立CAV模型的三自由度运动方程，函数中的~表示参数t在方程中没有用到所以用~代替。  
&emsp;`dY`关于`v_dt`;`theta_dt`;`psi_v_dt`;`x_dt`;`z_dt`;`y_dt`;  
```matlab
Y(1,:)=[v/sqrt(g*R0),theta,psi_v,longitude,r,latitude];
%r其实是高度，theta速度倾角，psi_v航向角
```
&emsp;在`CAVH.m`中使用到

* dtCAV_S1.m
* EQtrajectoryS1.m
* Init.m
* K2G.m
* NewCallback.m
* obstacleFinder.m
```matlab
function [distance,direction]=obstacleFinder(obstaclePosition,currentPosition)
...
end
```
输入参数：
1. obstaclePosition：所有障碍物的信息
0. currentPosition：[longitude,latitude,H/1e3]，当前位置信息

中间参数：
1. r(i)：对于圆柱形的障碍物来说，通过球面三角函数作为禁飞区半径

输出参数：
1. distance：返回所有与障碍物的距离，减去了中间变量r(i)。对于圆柱形障碍物，一旦飞行高度高于障碍物高度，就直接设置为1e8
0. direction：返回所有与障碍物的角度，圆形区域有特别处理（目前看到的是距离直接设置为1e9，方向角设置为0）

* PopSort.m

