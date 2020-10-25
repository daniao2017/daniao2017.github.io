[toc]
# stm32库开发实战指南读书笔记（二）

该部分主要是函数功能的实现,如`按键点亮LED`的相应知识点。
主要实现目录如下:
![stm32读书笔记二](./asserts/stm32读书笔记（二）.png)

## 基本硬件结构
一个基本的stm32应该包含如下：
- 主芯片
- 上电复位电路
- 时钟电路
- 电源供电电路组成

博主使用的是`正点原子`的`stm32 mini`开发板,实际开发过程中,**引脚的宏定义应根据实际电路来设置**

基本硬件如下:

### 复位电路及时钟电路

![复位电路及时钟电路](./asserts/复位电路及时钟电路.png)

### 串口输入及供电

该部分包括了RS232转ttl电路，实现如下:

![串口输入](./asserts/串口输入.png)

![串口输入及供电](./asserts/串口输入及供电.png)

### led及key电路
**注意**
LED0 -> PAout(8)	
LED1 -> PDout(2)
KEY0 -> GPIO_ReadInputDataBit(GPIOC,GPIO_Pin_5)//读取按键0
KEY1 ->  GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_15)//读取按键1
WK_UP ->  GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0)//读取按键2 

![led及key电路](./asserts/led及按钮外设.png)

## 编写自己的库函数
### .h文件
.h中一般放的是同名.c文件中定义的变量、数组、函数的声明，需要让.c外部使用的声明。这个声明有啥用？只是让需要用这些声明的地方方便引用。因为#include "xx.h" 这个宏其实际意思就是把当前这一行删掉，把xx.h 中的内容原封不动的插入在当前行的位置。
如本次写的led库函数以及按钮库函数的.h文件如下
#### key.h
包含俩个函数的。
- void KEY_Init(void);//IO初始化 
- u8 KEY_Scan(u8 mode);  //按键扫描函数	
```
#ifndef __KEY_H
#define __KEY_H	 
#include "sys.h"
 

//#define KEY0 PCin(5)   	
//#define KEY1 PAin(15)	 
//#define WK_UP  PAin(0)	 

#define KEY0  GPIO_ReadInputDataBit(GPIOC,GPIO_Pin_5)//读取按键0
#define KEY1  GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_15)//读取按键1
#define WK_UP   GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0)//读取按键2 
 
#define KEY0_PRES	1		//KEY0  
#define KEY1_PRES	2		//KEY1 
#define WKUP_PRES	3		//WK_UP  

void KEY_Init(void);//IO初始化
u8 KEY_Scan(u8 mode);  	//按键扫描函数		   
#endif

```
#### led.h
- void LED_Init(void);//io初始化
```
#ifndef __LED_H
#define __LED_H	 
#include "sys.h"

#define LED0 PAout(8)	// PA8
#define LED1 PDout(2)	// PD2	
void LED_Init(void);//初始化	 				    
#endif
```

### .c文件

**函数实现的内部逻辑**

#### led.c
```
#include "led.h"
/***
	* @brief LED IO初始化
	* @param 无 
	* @retval 无
	* @note 初始化PA8和PD52为输出口.并使能这两个口的时钟	
	* @note_time 2020-10-24,小刘同学制作
***/
void LED_Init(void)
{
 
 GPIO_InitTypeDef  GPIO_InitStructure;
 RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_GPIOD, ENABLE);	 //使能PA,PD端口时钟

 GPIO_InitStructure.GPIO_Pin = GPIO_Pin_8;				 //LED0-->PA.8 端口配置
 GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; 		 //推挽输出
 GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 //IO口速度为50MHz
 GPIO_Init(GPIOA, &GPIO_InitStructure);					 //根据设定参数初始化GPIOA.8
 GPIO_SetBits(GPIOA,GPIO_Pin_8);						 //PA.8 输出高

 GPIO_InitStructure.GPIO_Pin = GPIO_Pin_2;	    		 //LED1-->PD.2 端口配置, 推挽输出
 GPIO_Init(GPIOD, &GPIO_InitStructure);	  				 //推挽输出 ，IO口速度为50MHz
 GPIO_SetBits(GPIOD,GPIO_Pin_2); 						 //PD.2 输出高 
}
```
#### key.c
按键初始化,以及相应的处理函数
```
#include "key.h"
#include "delay.h"

/***
	* @brief 按键初始化函数 
	* @param 无 
	* @retval 无
	* @note PA0.15和PC5 设置成输入.并使能这两个口的时钟	
	* @note_time 2020-10-24,小刘同学制作
***/
void KEY_Init(void)
{
	
	GPIO_InitTypeDef GPIO_InitStructure;

 	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_GPIOC,ENABLE);//使能PORTA,PORTC时钟

	GPIO_PinRemapConfig(GPIO_Remap_SWJ_JTAGDisable, ENABLE);//关闭jtag，使能SWD，可以用SWD模式调试
	
	GPIO_InitStructure.GPIO_Pin  = GPIO_Pin_15;//PA15
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU; //设置成上拉输入
 	GPIO_Init(GPIOA, &GPIO_InitStructure);//初始化GPIOA15
	
	GPIO_InitStructure.GPIO_Pin  = GPIO_Pin_5;//PC5
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU; //设置成上拉输入
 	GPIO_Init(GPIOC, &GPIO_InitStructure);//初始化GPIOC5
 
	GPIO_InitStructure.GPIO_Pin  = GPIO_Pin_0;//PA0
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPD; //PA0设置成输入，默认下拉	  
	GPIO_Init(GPIOA, &GPIO_InitStructure);//初始化GPIOA.0
	
} 

/***
	* @brief 按键处理函数 
	* @param mode:
		0,不支持连续按;
		1,支持连续按;
	* @retval 
		KEY0_PRES，KEY0按下
		KEY1_PRES，KEY1按下
		WKUP_PRES，WK_UP按下 
	* @note 注意此函数有响应优先级,KEY0>KEY1>WK_UP!!
	* @note_time 2020-10-24,小刘同学制作
***/
u8 KEY_Scan(u8 mode)
{	 
	static u8 key_up=1;//按键按松开标志
	if(mode)key_up=1;  //支持连按		  
	if(key_up&&(KEY0==0||KEY1==0||WK_UP==1))
	{
		delay_ms(10);//去抖动 
		key_up=0;
		if(KEY0==0)return KEY0_PRES;
		else if(KEY1==0)return KEY1_PRES;
		else if(WK_UP==1)return WKUP_PRES; 
	}else if(KEY0==1&&KEY1==1&&WK_UP==0)key_up=1; 	     
	return 0;// 无按键按下
}

```
## 具体实现机理

完整的项目步骤如下：
- 新建项目，参考原子哥模块，配置好魔法棒
- 包含c/c++的c与h的路径
- 编译下载即可

### 主函数
```
/*---------------包含头文件----------------------*/
#include "stm32f10x.h"
#include "sys.h"
#include "delay.h"
#include "key.h"
#include "led.h"

/*-------------------主函数---------------------------*/
/***
	* @brief 循环读取按键的值，并用led状态表示
	* @param 外部按钮状态
		key_up 两个灯状态都反转
		key1 led1状态反转
		key0 led0状态反转
	* @retval 
		无
	* @note  初始状态led0被点亮
	* @note_time 2020-10-24,小刘同学制作
***/
 int main(void)
 {	
	u8 t=0;	  
	delay_init();	    	 //延时函数初始化	  
	LED_Init();		  	 	//初始化与LED连接的硬件接口
	KEY_Init();          	//初始化与按键连接的硬件接口
	LED0=0;					//点亮LED
	while(1)
	{
		t=KEY_Scan(0);		//得到键值
		switch(t)
		{				 
			case KEY0_PRES:
				LED0=!LED0;
				break;
			case KEY1_PRES:
				LED1=!LED1;
				break;
			case WKUP_PRES:				
				LED0=!LED0;
				LED1=!LED1;
				break;
			default:
				delay_ms(10);	
		} 
	}		
 }
```
### 小结
仔细分析上面模块，可以基本了解了如何控制stm32的输入与输出。以及项目代码的编写规范。总的来说还是需要多加练习。