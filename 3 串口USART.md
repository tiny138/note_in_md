<!--
 * @Author: your name
 * @Date: 2021-01-23 10:52:51
 * @LastEditTime: 2021-01-23 19:28:26
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \markdown\3 串口USART.md
-->
# 串口USART
## 1 基本概念

&emsp;串口通讯是一种设备间非常常用的串行通讯方式，因为它简单便捷，大部分电子设备都支持该通讯方式。并且它也是一个**很好的调试工具**，我们可以利用它输出调试信息到电脑屏幕。

## 2 涉及的外设
&emsp;涉及外设：GPIO(复用输入输出)，RCC（复位和时钟控制），USART(串口)

### 2.1 通信方式
&emsp;同步、异步 
1. **同步通信：**发送端和接收端必须使**用统一时钟**，它一种连续传送数据
的通信方式，一次通信传送多个字符数据，称为一帧信息。
<img src = picture\3.1.PNG>
**同步串行通信帧**：是将许多字符组成一个**信息帧**，这样，字符可以一 个接一个地传输，但是，在每帧信息的开始要加上同步字符，在**没有信息要传输时，要填上空字符**，因为同步传输不允许有间隙
<img src = picture\3.2.PNG>

&emsp;
**同步串行通信特点**：必须有同步时钟，传输信息量大，传输速率高， 但是传输设备复杂，技术要求高

0. **异步通信**：是指发送和接收端使用**各自的时钟**，并且他是一种不连续 传输的通信方式，**一次通信只传一个字符数据**，称为**字符帧**。字符帧之间 的间隙可以是任意间隙。**此次 USART 用的就是异步通信方式。**
<img src = picture\3.3.PNG>
异步串行通信帧：是将一个字节数据加上**起始位、校验位、停止位**， 构成的字符帧。**由于异步通信没有同步时钟，所以接收端要时刻处接收状态**.  
**起始位**:在没有数据传送时既空闲状态，此时通信线上为逻辑“1”状 态。**当发送端要发送 1 个字符数据时**，首先**发送一个逻辑“0”信号**，这个 低电平就是帧格式的**起始位**。其作用就是告诉接收端开始发送一帧数据。 接收端检测到这个低电平后，就准备接收数据信号  
**数据位**：在起始位之后，发送端发出的就是数据位，数据位的**位数没
有严格限制** 5~8 位都行。低位在前-，高位在后，由低位到高位逐位传送。 USART 一般是 8 位。  
**校验位**:  
**停止位**：  
<img src = picture\3.4.PNG>

&emsp;异步串行通信**特点**：不需要时钟同步，通信实现简单，设备开销小。
但是传输速率不高


### 2.2传输方向
&emsp;根据串行数据的传输方向，我们可以将通信分为单工，半双工， 双工。
1) 单工：是指数据传输仅能沿一个方向，不能实现反向传输。 
2) 半双工：是指数据传输可以沿两个方向，但需要分时进行传输。 
3) 全双工：是指数据可以同时进行双向传输
<img src = picture\3.5.PNG>

### 2.3传输速率
比特率：每秒钟传送的二进制位数。bps  bits per second  
波特率：每秒钟调制信号变化的次数。Baud   
串行通信常用波特率表示数据传输率。   
波特率与比特率的关系为：
`比特率 = 波特率 x 单个调制状态对应的二进制位数`

### 2.3 USART
&emsp;STM32 芯片具有多个 USART 外设用于串口通讯，USART（通用同步异步收发
器）能够灵活地与外部设备进行全双工数据通信。  
&emsp;**相关引脚**：  
1. Tx:发送数据输出引脚 
2. Rx：接收数据输入引脚
3. SW_Rx:数据接收引脚，只用于单线和智能卡模式，属于内部引脚，没有具体外部引脚
0. nRST：请求以发送(Request To Send)， n 表示低电平有效。
3. nCTS：清除以发送(Clear To Send)， n 表示低电平有效。
2. SCLK：发送器时钟输出引脚。**这个引脚仅适用于同步模式**

&emsp;STM32F411CCU6 一共有 **3 个 USART**,  *USART1 和 USART6* 时钟来源与 *APB2*总线时钟，其最大频率为 100MHz, *USART2* 来源于 *APB1* 总线时钟，其最大 频率为 50MHz。也就意味着他们的最大通讯速率也是不同的。

### 2.3 USART波特率发生器
&emsp;USART波特率发生器(USART_BRR)  
16位： 整数[15:5]、小数[0:4]  
USART 波特率计算公式如下:
<img src = picture\3.6.PNG>
**fck**:上述总线上的时间  
**0VER8**:是由 USART_CR1 的第 15 位设置。O:16 倍过采样；1:8 倍过
采样；   
**USARTDIV**:波特率分频系数，USART_BRR 配置得到。  
USARTDIV 的计算公式： USARTDIV = DIV_Mantissa + (DIV_Fraction/8 * (2-OVER8))

**例如**  
<img src = picture\3.7.PNG>

## 3 USART使用
### 3.1 初始化
**硬件原理图**
<img src = picture\3.8.PNG>  
**注意 e-link上的RX和TX相反**

<img src = picture\3.9.PNG>
PA9 和 PA10 当做 USART1 的 Tx 和 Rx 引脚来 用，  
PA9 和 PA10 已不是通用 IO 的作用了，而是芯片内部的 外设的接口引脚。   
STM32 将这种应用叫做“**I0引脚复用**”，这也就说 PA9 将被 配置成“复用推挽输出模式”，PA10 将被配置成“复用上拉输入模式

作为片内外设使用的时候，就叫做复用。并且***片内外设的功能引脚也不是随意复用的***  
外设复用库函数  
`void GPIO_PinAFConfig(GPIO_TypeDef* GPIOx, uint16_t GPIO_PinSource, uint8_t GPIO_AF)`  
其中，GPIO_AF指的是复用外设哪个功能

**①GPIOA管脚整体初始化**：
`void GPIO_CoNFIG(void);`
```C
    void GPIO_CoNFIG(void)
    {	
        //1.初始化时钟
        GPIO_InitTypeDef GPIO_InitStructure;
        RCC_AHB1PeriphResetCmd(RCC_AHB1Periph_GPIOA,ENABLE);

        //2.管脚复用
        GPIO_PinAFConfig(GPIOA,GPIO_PinSource9,GPIO_AF_USART1); 
        GPIO_PinAFConfig(GPIOA,GPIO_PinSource10,GPIO_AF_USART1); 

        //3.PA9与PA10管脚初始化
        //PA9->tx (复用推挽输出模式)
        GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
        GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF;
        GPIO_InitStructure.GPIO_OType = GPIO_OType_PP;
        GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
        GPIO_Init(GPIOA,&GPIO_InitStructure);

        //PA10->rx (复用上拉输入)
        GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;
        GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF;
        GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_UP;
        GPIO_Init(GPIOA,&GPIO_InitStructure);
    }
```
**②USART外设初始化**：
USART 的初始化所需要调用到 stm32f4xx_usart.c 和 stm32f4xx_usar
t.h 文件，其中.h 文件中存放了关于 USART 的所有功能接口函数，大家需要仔细研究，需要用导函数`void USART_init(int32_t baud);`
```C
    void USART_init(int32_t baud)
    {
        USART_InitTypeDef USART_InitStructure;

        //1.首先配置USART1所需要的引脚
        GPIO_CoNFIG();

        //2.使能USART1时钟
        RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1,ENABLE);

        //3.初始化USART1
        USART_InitStructure.USART_BaudRate = baud;                    //波特率
        USART_InitStructure.USART_WordLength = USART_WordLength_8b;//字长8位
        USART_InitStructure.USART_Parity = USART_Parity_No;//无校验位
        USART_InitStructure.USART_StopBits = USART_StopBits_1; //一位停止位
        USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
        USART_InitStructure.USART_Mode = USART_Mode_Rx | USART_Mode_Tx; //发送模式
        USART_Init(USART1,&USART_InitStructure);  //USART1c初始化

        //4.使能USART1
        USART_Cmd(USART1,ENABLE);


    }
```



### 3.2 发送过程
*  首先要**使能**发送即 **USART_CR1** 的 TE 位置 1；
* 接着内部总线的数据的一个字节**写入**“发送数据寄存器（**TDR**）”； （该操作将清零 TXE 位也就是发送数据寄存器非空，数据其他数 据不可以写入）
* 紧接着“发送数据寄存器（TDR）”中的数据一次性复制进入“发送移位寄存器”；（将 TXE 位置既发送数据寄存器为空，后续数据 可以接着写入）
* “发送移位寄存器”将刚才“发送数据寄存器（TDR）”复制得数 据一位一位的送到 Tx 引脚
* 循环执行上面的操作，直到总线将最后一个数据写入“发送数据 寄存器（TDR）”后，等待 TC=1。这表明最后一帧的传送已完成

**经过3.1的流程我们已经完成了所有的初始化工作**，我们下面就可以直接
调用串口数据发送函数 `USART_SendData()`;和串口数据接收函数 `USART_Receive Data()`;来时实现 DragonFly 学习平台与电脑的通信实验，也就是通过 USART1 将数据打印到电脑

### 3.3 接收过程
USART 接收过程： Rx 引脚有数据输入时，

* 首先要使能接收即 USART_CR1 的 RE 位置 1； 
* 然后 Rx 引脚移入数据的最低有效位，到“接收移位寄存器”； 
* 当“接收移位寄存器”8 位满时，将数据一次性写入“接收数据寄 存器（RDR）”;(该操作将 RXNE 置 1 既接收数据寄存器非空，总线 可读取)
* 总线发现 RXNE=1 时立即读取数据并将 RXNE 置零（注意接收期间 每接收一个字节 RXNE 都置 1）
* 循环执行上面操作，直到 Rx 引脚将最后一字节数据传送入“接收 数据寄存器（RDR）”后，等待总线读取完成



## 4 注意
1. 函数写完需要单击 的图标，然后定位到 Target 界面勾选 Use Micro LIB。如图 5-19：

0. 可能配置上有一些问题，要添加.c文件

















