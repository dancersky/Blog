# 关于STM32串口printf输出调试信息问题

##### 																		         																   By  Sky.J  	2018.06.01

### 1,遇到的问题（使用HAL库）

​	在STM32使用过程中，我们程序调试时一般都会用到printf重定向串口输出调试信息来进行程序开发调试，从网上我们找到了重定向的代码部分加入到串口代码文件中，如下：

```c
UART_HandleTypeDef husart_printf;

#ifdef __GNUC__  
/* With GCC/RAISONANCE, small printf (option LD Linker->Libraries->Small printf  
set to 'Yes') calls __io_putchar() */  
#define PUTCHAR_PROTOTYPE int __io_putchar(int ch)  
#else  
#define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)  
#endif /* __GNUC__ */  

PUTCHAR_PROTOTYPE
{
  /* Place your implementation of fputc here */
  /* e.g. write a character to the USART2 and Loop until the end of transmission */
	HAL_UART_Transmit(&husart_printf, (uint8_t *)&ch, 1, 0xFFFF);
  return ch;
}
```

​	程序编辑好，本以为可以直接打印出消息从串口输出，但是程序下进去并未打印出任何信息到串口，这就奇怪了，难道是上面重定向部分不对吗？



### 2，找到问题

​	在网上又搜索，发现半主机模式这样一个模式。半主机是这么一种机制，它使得在ARM目标上跑的代码，如果主机电脑运行了调试/仿真器，那么该代码可以使用该主机电脑的输入输出设备，即直接可以使用主机电脑的外设实现输入输出调试，并不是使用ARM目标器件的外设输入输出设备。简单来说就是，半主机需要依靠仿真器才可输入输出调试信息。而我们是使用单片机的串口外设，即不使用仿真器，那么我们就必须禁用半主机模式。



### 3，解决问题

​	通过上面的了解，我们知道像printf()这类函数是使用了半主机模式，我们现在只需不在半主机模式下使用printf就可以解决问题了。

​	解决方法1：

​		使用微库MicroLIB，MicroLIB是一个C99标准库的微缩版，简称微库，使用微库则不会使用半主机模式故可正常运行并打印调试信息。在使用keil5编程时，配置上勾选Use MicroLIB即可，关于MicroLIB详细信息可以自行搜索。

​	解决方法2：

​		使用标准库，但是禁用半主机模式，如何实现，只需在主程序后添加如下代码即可：

```c
#pragma import(__use_no_semihosting) 
int _sys_exit(int x) 
{ 
	x = x; 
} 
struct __FILE 
{ 
int handle; 
/* Whatever you require here. If the only file you are using is */ 
/* standard output using printf() for debugging, no file handling */ 
/* is required. */ 
}; 
/* FILE is typedef’ d in stdio.h. */ 
FILE __stdout;


/**********************************************说明*********************************************
__use_no_semihosting_swi，即不使用半主机模式，防止程序进入软件中断。在嵌入式程序编译时如果出现printf、fopen、fclose等文件操作，因程序中并没有对这些函数的底层实现，使得设备运行时会进入软件中断BAEB处，这时就需要__use_no_semihosting_swi这 个声明，使程序遇到这些文件操作函数时不停在此中断处.
**********************************************************************************************/
```

​		