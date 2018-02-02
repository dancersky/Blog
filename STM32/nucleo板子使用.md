## 官方STM32的NUCLEO开发板使用



**概述**：因工作需求，先买了stm32的NUCLEO开发板做测试板，本人使用的是L073RZ这块板子，也是第一次接触stm32，摸索状态。这里记录一下使用过程，做笔记的同时希望也可以对别人有点帮助。



#### 1，准备

​	既然要使用这块板子，那么对这块板子我们肯定是需要基本了解一下的，怎么测试，如何测试，供电问题，调试问题我们都需要看看官方文档。

​	这里贴一个NUCLEO开发板手册下载地址[http://www.stmcu.org/document/detail/index/id-214946](http://www.stmcu.org/document/detail/index/id-214946).里面详细的说了开发板的功能，如何使用，原理图等，有不懂的不明白的查查这手册。

​	在手册的5.1节，就说了如何开始使用。我把原文贴下，顺便简单翻译一下，本人英语垃圾，只是表达下意思，不对的还请指正，还有本核心板已经集成ST-LINK/V2-1：

> Follow the sequence below to configure the STM32 Nucleo board and launch the demo 
> software:（按照下面的步骤配置STM32 NUCLEO核心板并启动演示软件）
>
> 1. Check the jumper position on the board, JP1 off, JP5 (PWR) on U5V, JP6 on (IDD), 
>   CN2 on (NUCLEO) selected.（检差板子上跳线帽位置，JP1不接跳线帽，JP5跳线帽接U5V插针，JP6接跳线帽，CN2接跳线帽用于t调试选择NUCLEO板子）
> 2. For correct identification of all device interfaces from the host PC, install the Nucleo 
>   USB driver available from the www.st.com/stm32nucleo webpage, prior to connecting 
>   the board.（在连接板子之前，为了主机PC能识别设备所有接口，需安装Nucleo USB驱动，可从www.st.com/stm32nucleo 获取）
> 3. Connect the STM32 Nucleo board to a PC with a USB cable ‘Type-A to Mini-B’ through 
>   USB connector CN1 to power the board. The red LED LD3 (PWR) and LD1 (COM) 
>   should light up. LD1 (COM) and green LED LD2 should blink.（将STM32核板连接到带有USB线缆的PC上，即A到mini B。 USB连接器CN1为板供电。红色LED LD3 (PWR)和LD1 (COM) 应该点亮。LD1 (COM)和绿色LED LD2应该闪烁）
> 4. Press button B1 (left button).（按下B1键（左键））
> 5. Observe the blinking frequency of the three LEDs LD1 to LD3, by clicking on the button 
>   B1.（通过点击按键 B1，观察三个led LD1到LD3的闪烁频率）
> 6. The demonstration software and several software examples on how to use the STM32 
>   Nucleo board features are available at the www.st.com/stm32nucleo webpage.（演示软件和几个关于如何使用STM32 NUCLEO核心板的软件示例可在www.st.com/stm32nucleo网页上找到）
> 7. Develop the application using the available examples.（使用可用示例开发应用程序）



#### 2，开始使用

​	准备完毕，我们就要开始按步骤开始使用调试。

**第一步，检查跳线帽，略过了。**

**第二步，安装ST-LINK/V2-1驱动**

​	1，官网下载驱动相关软件，[驱动地址](http://www.st.com/content/st_com/en/search.html#q=ST-LINK/V2-1-t=tools-page=1).页面如下所示，选择对应版本下载，同时记得下载一下STSW-LINK007软件包，这个是用于ST-LINK/V2-1固件更新的。

![](https://github.com/dancersky/Blog/blob/master/STM32/img/%E9%A9%B1%E5%8A%A8%E9%A1%B5%E9%9D%A2%E4%B8%8B%E8%BD%BD.png)



​	2，我下载的驱动是STSW-LINK009，解压后以管理员身份运行stlink_winusb_install.bat文件，安装好驱动。

**第三步，供电并查看驱动是否可用**

​	我们将USB的mini口接上板子，另一端接到电脑，这时我们可以在设备管理器查看，我们应该可以看到下图所示的界面，一个ST-LINK Debug的串行控制总线和一个COM口。同时我们也看到绿色的LED灯闪烁。

![](https://github.com/dancersky/Blog/blob/master/STM32/img/%E9%A9%B1%E5%8A%A8%E6%98%BE%E7%A4%BA.png)



**第四步，按B1键并观察LED灯闪烁频率**

​	发现绿色LED闪烁频率变化。

**第五步，官网下载示例程序并下载到板子测试运行（默认你已经配置安装好keil5）**

​	1，官方示例包下载地址：[示例包地址](http://www.st.com/content/st_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-configurators-and-code-generators/stm32cubemx.html)，选择对应系列即可，我的事L0就下载的STM32CubeL0的软件包。

​	2，对应芯片keil5 pack包下载地址：[地址](http://www.keil.com/dd2/Pack/)，选择对应芯片系列下载即可，然后安装即可。

​	3，使用usart测试示例.

​		（1）解压我们下载的STM32CubeL0的软件包，使用keil5打开对应板子的UART程序。我打开的程序路			径是：

```
STM32Cube_FW_L0_V1.10.0\Projects\STM32L073RZ-Nucleo\Examples\UART\UART_TwoBoards_ComPolling\MDK-ARM.
```

​		（2）修改源码，我们如果看手册的话，可以知道usb集成的串口是USART2,而我源代码里面是使用的USART1,这样我们串口输出不能用已经集成的串口输出了，所以我改了一下main.h的宏定义，改后如下：

```c
/* Definition for USARTx clock resources */
#define USARTx                           USART2
#define USARTx_CLK_ENABLE()              __HAL_RCC_USART2_CLK_ENABLE()
#define USARTx_RX_GPIO_CLK_ENABLE()      __HAL_RCC_GPIOA_CLK_ENABLE()
#define USARTx_TX_GPIO_CLK_ENABLE()      __HAL_RCC_GPIOA_CLK_ENABLE()

#define USARTx_FORCE_RESET()             __HAL_RCC_USART2_FORCE_RESET()
#define USARTx_RELEASE_RESET()           __HAL_RCC_USART2_RELEASE_RESET()

/* Definition for USARTx Pins */
#define USARTx_TX_PIN                    GPIO_PIN_2
#define USARTx_TX_GPIO_PORT              GPIOA
#define USARTx_TX_AF                     GPIO_AF4_USART2
#define USARTx_RX_PIN                    GPIO_PIN_3
#define USARTx_RX_GPIO_PORT              GPIOA
#define USARTx_RX_AF                     GPIO_AF4_USART2
```

​		（3）使用keil5编译软件，设置Device为对应的芯片，设置Debug选项为ST-LINK,下载程序到板子，下载成功后我们打开调试助手，选择我们对应的串口打开，设置9600波特率，按板子复位键，再按B1键，就可以看到收到了设备的信息如下所示（这里主要是keil5软件下载调试部分，就不祥说了）:

​		![](https://github.com/dancersky/Blog/blob/master/STM32/img/%E4%B8%B2%E5%8F%A3%E6%98%BE%E7%A4%BA%E4%BF%A1%E6%81%AF.png)



**第六步，使用示例程序开发**

​	这就根据自己需要，去玩喽。。。

#### 3，使用总结

​	整体来说，官方的板子用起来还是挺方便的，我一个没用过STM32的使用cubeMX 加 keil5可以很快的写一个小程序出来并使用。之前主要做是嵌入式linux，使用HAL库非常方便快捷，不用关心底层，速度较快。