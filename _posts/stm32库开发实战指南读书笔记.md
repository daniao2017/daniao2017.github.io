[toc]
# stm32开发实战读书笔记（一）
## 部分目录
前面的基础部分，主要是将一些开发注意的事项，以及相应的知识点

部分目录如下:
![目录](./asserts/基础篇(一).png)

## 开发注意事项

- 推荐使用库函数开发
 可以查看`STM32F10x_StdPeriph_Driver_3.5.0(中文版).chm`官方的帮助文档加深理解

- 代码尽量写的规范些，注释要到位
  可以使用`doxygen`，生成文档

- 多练习与敲代码，心静

## 基本知识点

### 寄存器操作
- `<<` 
如左移,(1<<3)
- `&=`
 配合1xxx0xxx0,使用可以清空为0的位置上数
- `|=`
 配合0xxx01xx,使用可以将1位置上的数复制为1

- `^=`
 按位取反
- `volatile` 关键字
 表示易变变量，防止其值被优化

- c语言条件编译
- 结构体指针

### GPIO操作
GPIO_InitTypeDef结构参考

|数据结构|关键字|
|-|-|
|uint16_t|  GPIO_Pin |
|GPIOSpeed_TypeDef|  GPIO_Speed |
|GPIOMode_TypeDef|  GPIO_Mode |

- GPIOSpeed_TypeDef的枚举值
`GPIO_Speed_10MHz ` 
`GPIO_Speed_2MHz `  
`GPIO_Speed_50MHz ` 

- GPIOMode_TypeDef的枚举值
`GPIO_Mode_AIN `  
`GPIO_Mode_IN_FLOATING `  
`GPIO_Mode_IPD  ` 
`GPIO_Mode_IPU `  
`GPIO_Mode_Out_OD`   
`GPIO_Mode_Out_PP`   
`GPIO_Mode_AF_OD `  
`GPIO_Mode_AF_PP ` 

更多知识点可以查看官方的`帮助手册`，边学边进行总结

## 一些电路结构
### STM32F10X系统框图

![系统框图](./asserts/系统框图.jpg)

### GPIO结构图

![GPIO结构图](./asserts/GPIO结构图.jpg)

### GPIO输出模式

![推挽输出](./asserts/推挽等效电路.jpg)

![开漏输出](./asserts/开漏电路.jpg)

### GPIO硬件消抖

![硬件消抖](./asserts/硬件消抖.jpg)