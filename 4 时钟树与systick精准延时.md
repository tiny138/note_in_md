<!--
 * @Author: your name
 * @Date: 2021-01-23 21:11:19
 * @LastEditTime: 2021-01-25 10:46:54
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \markdown\4 时钟树与systick精准延时.md
-->
# 时钟树与systick精确延时
## 1 涉及外设
&emsp;RCC(复位时钟控制)、SysTick 定时器  
&emsp;GPIO 和串口，我们发现这两个外设初始化之前都会有一 个时钟初始化，并且不同的外设所挂载的总线下的时钟也不一样。

### 1.1 时钟系统简介
* 51 一个系统时钟
* stm32 多个时钟系统，因为：因为首先 STM32 本身非常复杂，外设非常的多，但是**并不是所有外设都需要系统时钟这 么高的频率**，比如看门狗以及 RTC 只需要几十 k 的时钟即可。**同一个电路， 时钟越快功耗越大，同时抗电磁干扰能力也会越弱，所以对于较为复杂的 MCU 一般都是采取多时钟源的方法来解决这些问题**

在 STM32F411 中有一共有 5 个时钟源，分别为 HSI、HSE、LSI、LSE、PLL。 这五个时钟源为整个时钟系统提供时钟源头。
1. LSI 是**低速内部时钟**，RC 振荡器，频率为 32kHz 左右。*供独立看门狗和 自动唤醒单元使用*。
2. LSE 是**低速外部时钟**，接频率为 32.768kHz 的石英晶体。 这个主要是 *RTC 的时钟源*。
3. HSE 是**高速外部时钟**，可接石英/陶瓷谐振器，或者接外部时钟源，频率 范围为 4MHz~26MHz。我们的 DragonFly 学习平台接的是 8M 的晶振。HSE 也可以直接做为*系统时钟或者 PLL 输入*。
4. HSI 是**高速内部时钟**，RC 振荡器，频率为 16MHz。可以直接作为*系统时钟或者用作 PLL 输入*。
5. PLL 为锁相环倍频输出。 STM32F411 有两个 PLL：
* 主 PLL(PLL)由 HSE 或者 HSI 提供时钟信号，并具有两个不同的输 出时钟。 第一个输出 PLLP 用于生成高速的系统时钟（最高 100MHz） 第二个输出 PLLQ 用于生成 USB OTG FS 的时钟（48MHz），随机数 发生器的时钟和 SDIO 时钟。
* 专用 PLL(PLLI2S)用于生成*精确时钟*，从而在 I2S 接口实现高品质 音频性能

&emsp;**其实以上讲的系统时钟初始化配置完全不需要我们手动配置，STM32 标准库 中都已用函数封装好了，并且在系统启动文件里自动调用，这也就是为什么前面 讲解 GPIO 和串口时没有初始化时钟但我们写的程序一样能运行。**

## 2 systick 系统定时器
&emsp;SysTick—系统定时器是属于 CM4 内核中的一个外设，定时器是属于 CM4 内核的外设，所以有关寄存器的定义和配置函数在`core_cm4.h` 文件中。

系统定时器是一个 24bit 的向下递减的计数器，一般我们设置系统时钟 SYSCLK 等于 100MHz。**当重装载数值寄存器的 值递减到 0 的时候，系统定时器就产生一次中断，以此循环往复**

* SysTick->VAL 寄存器的值每一个时钟周期（CLOCK）将会减1
* 如果 SysTick->VAL 被减到零时 SysTick->LODA 的值将会立即装载进入 SysTick->VAL 中，并且 SysTick->CTRL 的 COUNTFLAG将会置 1 
* 如果使能了定 时器中断，系统将会进入 SysTick 的中断。我们的延时原理：就是通过设置 SysTick->LODA 和时钟周期的数值来确定延时时间

## 3 软件设计
### 3.1 定时器使用思路
&emsp;可以调用`SysTick_Config()`;函数将定时器配置成每1ms中断一
次，因为计数的时钟周期为 10ns(T=1/100Mhz),也就是说 SysTick->VAL 每递减 一次数的时间就是 10ns。那么 1ms 的时间需要 SysTick->VAL 递减多少次呢？我 们知道 1ms = 1x10^6ns = 1x10^5x10ns,也就是说 SysTick->VAL 递减 100000 次 就是 1ms,所以 SysTick->LODA 就因该是 100000，也就是或 ticks 这个参数就必 须是 100000。1ms进一次中断：  
`SysTick_Config(SystemCoreClock/1000);`

我们定义一个全局变量` volatile uint32_t sysTickUptime = 0`;每一次中
断 `sysTickUptime++`  （**volatile 修饰符表示 sysTickUptime 变量是一个易变的变量，他告知编译器不要优化直接在内存中读取变量内容**）SysTick 定时器中断 函数在 stm32f4xx_it.c 中，stm32f4xx_it.c 中不只有 SysTick 定时器中断，而 是所有中断都在这个文件中，以后我们所有的中断也都会放在这个文件里面。 **SysTick 定时器实现微秒级计时中断**如下代码：
```c
    extern uint32_t systickuptime;
    void SysTick_Handler(void)
    {
        systickuptime++;
    }

    uint32_t millis(void)
    {
        return systickuptime;
    }
```
毫秒级也可以这样，但是**太消耗资源**，所以可以：  
**理解**
do{}while()里的循环体 ms = sysTickUptim；cycle_cnt = SysTick->VAL；仅在 sysTickUptime 的值被更新时 才时**循环两次**，其他情况就**只执行一次**就返回了微秒计数。**通过val读微妙的值，毫秒才进中断，节约资源**
```C
    void Delay_Init(void)
    {
        SysTick_Config(SystemCoreClock/1000);  //选择系统时钟（HCLK）作为SysTick定时器的时钟，开SysTick中断 1ms一次
        usTicks = SystemCoreClock/ 1000000; //100M/1M = 100次 也就是计数100次为1us
    }

    uint32_t micros(void)
    {
        register uint32_t ms, cycle_cnt;  //register请求编译器尽可能的将变量存在CPU内部寄存器，而不是通过内存寻址访问
        do 
        {
                ms = sysTickUptime;
                cycle_cnt = SysTick->VAL; // SysTick->VAL 的计数范围 100000 ~ 0就是1ms,递减计数器
        } while (ms != sysTickUptime);
        return (ms * 1000) + (100000 - cycle_cnt) / usTicks;
    }
```

这样延时器就可以这样写`delay()`
```C
    void delay_ms(uint32_t nms)
    {
        uint32_t Tm = 0;
        Tm = millis();
        while(millis()-Tm <= nms)
        {}
    }

    void delay_us(uint32_t nus)
    {
        uint32_t Tu = 0;
        Tu = micros();
        while(micros()-Tu <= nus)
        {}
    }
```


## 注意
中断函数是需要自己写的。


