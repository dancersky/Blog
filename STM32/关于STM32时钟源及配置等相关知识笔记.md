# 关于STM32时钟源及配置等相关知识笔记

### 1，时钟源

​	HSI
    	HSE
    	LSI
    	LSE
    	PLL
    	MSI （L系列特有）

### 2，主要的几个时钟

​	SYSCLK --->系统时钟
	PLLCLK --->锁相环时钟
	HCLK --->AHB总线时钟
	PCLK1 --->APB1总线时钟
	PCLK2 --->APB2总线时钟

### 3，各时钟来源

​	SYSCLK，系统时钟来源，可直接选择HSI,HSE,MSI直接作为系统时钟源，也可经过PLL倍频输出后的PLL作为系统时钟。
	PLLCLK,锁相环时钟，一般是HSI,HSE,MSI时钟经过锁相环倍频后的输出时钟。
	HCLK, AHB总线时钟，经过SYSCLK预分频后的输出时钟。
	PCLK1,外设低速时钟，经过HCLK预分频后的输出时钟。
	PCLK2,外设高速时钟，经过HCLK预分频后的输出时钟。

### 4，各时钟对应的外设

​	一般从数据手册中我们可以找到详细的各总线对应的外设情况，如下：

​	![外设对应图]()

### 5，一个简单的系统时钟配置（HAL库,部分代码）

```c
	/*
	SYSCLK---32MHZ
	PLLCLK---32MHZ
	HCLK---4MHZ
	PCLK1---4MHZ
	PCLK2---4MHZ
	*/
	//选用HSI，16MHZ 内部高速晶振
    RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
    RCC_OscInitStruct.HSIState = RCC_HSI_ON;
    RCC_OscInitStruct.HSICalibrationValue = 16;
	//锁相环开启，倍频4，分频2，16*4/2 = 32MHZ
    RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
    RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI;
    RCC_OscInitStruct.PLL.PLLMUL = RCC_PLLMUL_4;
    RCC_OscInitStruct.PLL.PLLDIV = RCC_PLLDIV_2;
    if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
    {
    	Error_Handler();
    }
    RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
    |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
	//选取系统时钟为锁相环时钟PLLCLK
    RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
	//HCLK 8分频  32/8 = 4MHZ
    RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV8;
	//PCLK1 PCLK2 分频1  4/1 = 4MHZ
    RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;
    RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;
    if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_0) != HAL_OK)
    {
    	Error_Handler();
    }
```

