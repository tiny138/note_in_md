<!--
 * @Author: your name
 * @Date: 2021-01-21 19:04:11
 * @LastEditTime: 2021-01-21 21:47:48
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \markdown\2 点亮LED灯.md
-->
# 点亮LED
## 1 涉及的外设
GPIO(通用输入输出),RCC（复位和时钟控制）。

### 1.1 GPIO（后续有用的再来补充）
&emsp;
DragonFly 学习平台这次用的是 STM32F411CCU6 一共有 48 个引脚。共分为 3 的端口 GPIO 分别为 GPIOA、GPIOB、 GPIOC，每个端口 16 个 I/O 引脚。  
&emsp;
将 GPIO 分为八种模式。八种模式分别为：
1. 输入浮空 
2. 输入上拉 
3. 输入下拉 
4. 模拟输入 
5. 具有上拉或下拉功能的开漏输出 
6. 具有上拉或下拉功能的推挽输出 
7. 具有上拉或下拉功能的复用功能推挽 
8. 具有上拉或下拉功能的复用功能开漏  

&emsp; 
**输入**部分整个图的
上半部分 1、2、3、4，**输出**部分整个图的下半部分 5、6、7、8。

### 1.1 GPIO初始化过程
<img src = picture\LED电路图.PNG>

* 使能GPIOB 的时钟 
* 配置GPIOB 端口到第 8 个 IO 
* 配置 PB8 的为输出模式（配置GPIOB_MODER） 
* 配置 PB8 的输出类型为推挽（配置GPIOB_OTYPER） 
* 配置 PB8 的输出速度为 100MHz（配置GPIOB_OSPEEDR） 
* 初始化GPIOB（初始化相应的寄存器）

`void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO _InitStruct);`  
其中 ``GPIO_InitTypeDef``   

```C
    typedef struct {
        uint32_t GPIO_Pin;   //引脚
        GPIOMode_TypeDef GPIO_Mode;  //输入or输出
        GPIOSpeed_TypeDef GPIO_Speed;   //输出速度
        GPIOOType_TypeDef GPIO_OType;   //推挽or开漏
        GPIOPuPd_TypeDef GPIO_PuPd;    //上拉/下拉
    }GPIO_InitTypeDef;
```   

**初始化模板**   
    
```C
    GPIO_InitTypeDef  GPIO_InitStructure;  //声明结构体变量
    RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOB, ENABLE);  //使能GPIOB的时钟
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_8;   //上面结构体变量赋值
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_OUT; 
    GPIO_InitStructure.GPIO_OType = GPIO_OType_PP; 
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_100MHz; 
    //   GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_UP;
    GPIO_Init(GPIOB, &GPIO_InitStructure); 
```
GPIO_InitTypeDef 的每个成员，其实也就是将我们上面讲的 GPIO 的配置寄存器的对应位给封装进结构体了，让我们无需了解底层寄存器的操作一样可以达到初始化 GPIO 的目的；最后调用 GPIO_Init();初始化 GPIO 为高速推挽输出模式；**注意**：其中上/下拉模式一般是应用在输入模式，输出模式配置不配置没有影响；所以可以注释掉。
    
### 1.2 GPIO置位与复位
`void GPIO_SetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);`  
此函数是 GPIO 置位函数，**第一个参数**是端口号，我们填 GPIOB;     **第二个是
IO 号**，我们填 GPIO_Pin_8。调用此函数并传入前面俩参数 **PB8** 引脚就会被置位**高电平**

`void GPIO_ResetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pi n);`  
