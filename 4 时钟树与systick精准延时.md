<!--
 * @Author: your name
 * @Date: 2021-01-23 21:11:19
 * @LastEditTime: 2021-01-23 21:16:54
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
* stm32 多个时钟系统，因为：因为首先 STM32 本身非常复杂，外设非常的多，但是**并不是所有外设都需要系统时钟这 么高的频率**，比如看门狗以及 RTC 只需要几十 k 的时钟即可。同一个电路， 时钟越快功耗越大，同时抗电磁干扰能力也会越弱，所以对于较为复杂的 MCU 一般都是采取多时钟源的方法来解决这些问题