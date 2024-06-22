@[TOC]
# 一、什么是“软件I2C”和“硬件I2C”
## 1.1 什么是“软件I2C”
软件I2C (Software I2C)
软件I2C是通过软件代码模拟I2C协议的时序和信号来实现通信的。这种方法不依赖于特定的硬件I2C模块，因此几乎可以在任何微控制器上实现。以下是一些关键点：

 - 灵活性高：由于是通过软件实现，可以在没有硬件I2C支持的设备上使用。
 
 - 占用资源：软件I2C需要CPU周期来处理时序和数据传输，因此会占用更多的处理器时间，可能影响整体系统性能。
 
 - 速度限制：受限于软件执行速度，通常不能达到硬件I2C的最高速率。

 -  实现复杂度：需要编写代码来管理全部I2C时序，错误处理也需要额外的代码。
## 1.2 什么是“硬件I2C”
硬件I2C (Hardware I2C)
硬件I2C利用微控制器内置的I2C模块来实现通信。这种方式依赖于专门设计的硬件单元来处理I2C协议。以下是一些关键点：
 - 效率高：硬件I2C模块独立于CPU运行，可以大幅减轻CPU负担，提高整体系统效率。
 
 - 速度高：硬件I2C通常支持更高的通信速率，能够轻松达到I2C标准的最大速率（例如400kHz或更高）。
 
 - 可靠性高：由硬件处理时序和协议，通常更加稳定和可靠。

 - 配置简单：大多数微控制器提供易于使用的寄存器配置来启用和控制硬件I2C。
# 二、软件I2C和硬件I2C
​ 想要控制STM32产生I2C方式的通讯，可以采用软件模拟或硬件I2C 这两种方式。
硬件I2C直接使用外设来控制引脚，可以减轻 CPU 的负担。不过使用硬件I2C时必须使用某些固定的引脚作为 SCL 和 SDA，软件模拟IIC则可以使用任意GPIO引脚，相对比较灵活。在本开发板中，由于 STM32RCT6 芯片引脚较少，资源比较紧张，在设计硬件时不方便使用硬件I2C指定的引脚连接外部设备（EEPROM 存储器芯片），所以在控制程序上只能使用软件模拟 I2C的方式。
## 2.1 软件模拟
​ 所谓软件模拟，即直接使用 CPU 内核按照 IIC协议的要求控制 GPIO 输出高低电平。如控制产生IIC的起始信号时，先控制作为 SCL 线的 GPIO 引脚输出高电平，然后控制作为 SDA 线的 GPIO 引脚在此期间完成由高电平至低电平的切换，最后再控制 SCL 线切换为低电平，这样就输出了一个标准的IIC起始信号。
## 2.2硬件I2C
硬件I2C是指直接利用 STM32 芯片中的硬件I2C外设，该硬件I2C外设跟 USART串口外设类似，只要配置好对应的寄存器，外设就会产生标准串口协议的时序。使用它的IIC外设则可以方便地通过外设寄存器产生I2C协议方式的通讯，如初始化好I2C外设后，只需要把某寄存器位置 1，那么外设就会控制对应的 SCL 及 SDA 线自动产生IIC起始信号，而不需要内核直接控制引脚的电平。
# 三、配置STM32CubeMX

 - RCC设置：
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/ded2944778fb4a44a80633eb76e381d7.png#pic_center)

 - USART1设置：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/b5f3c5f8067740a7b955270dc4d07bbf.png#pic_center)

 - GPIO设置：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/6499c773e86a4434ad94a041d146d8d5.png#pic_center)

 - 时钟设置：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/a7a065e2f3e2401b9599c36636d9eb08.png#pic_center)

 - I2C1设置：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/58986c48362b420cb9370ec85a46fec2.png#pic_center)
# 四、配置keil代码
[AHT20芯片代码官网下载](http://www.aosong.com/class-36-2.html)
## 4.1 创建文件
在创建好的项目文件中创建一个新文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/81c409333cd943f4bec04f11d8cce0db.png#pic_center)
## 4.2 复制文件
将在官网下载的文件复制到新建文件夹中
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/9a2a5bf6d381433c82e1b7d5347974bc.png#pic_center)
## 4.3 在keil中添加文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/c12e4067699043db89d97183b5f7a985.png#pic_center)
## 4.4 添加路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/c47e96d0651147aa837284a993054439.png#pic_center)
## 4.5 代码修改

 - 修改AHT20-21_DEMO_V1_3.h

```c
#ifndef _AHT20_DEMO_
#define _AHT20_DEMO_

#include "main.h"  

void Delay_N10us(uint32_t t);//延时函数
void SensorDelay_us(uint32_t t);//延时函数
void Delay_4us(void);		//延时函数
void Delay_5us(void);		//延时函数
void Delay_1ms(uint32_t t);	
void AHT20_Clock_Init(void);		//延时函数
void SDA_Pin_Output_High(void)  ; //将PB15配置为输出 ， 并设置为高电平， PB15作为I2C的SDA
void SDA_Pin_Output_Low(void);  //将P15配置为输出  并设置为低电平
void SDA_Pin_IN_FLOATING(void);  //SDA配置为浮空输入
void SCL_Pin_Output_High(void); //SCL输出高电平，P14作为I2C的SCL
void SCL_Pin_Output_Low(void); //SCL输出低电平
void Init_I2C_Sensor_Port(void); //初始化I2C接口,输出为高电平
void I2C_Start(void);		 //I2C主机发送START信号
void AHT20_WR_Byte(uint8_t Byte); //往AHT20写一个字节
uint8_t AHT20_RD_Byte(void);//从AHT20读取一个字节
uint8_t Receive_ACK(void);   //看AHT20是否有回复ACK
void Send_ACK(void)	;	  //主机回复ACK信号
void Send_NOT_ACK(void);	//主机不回复ACK
void Stop_I2C(void);	  //一条协议结束
uint8_t AHT20_Read_Status(void);//读取AHT20的状态寄存器
uint8_t AHT20_Read_Cal_Enable(void);  //查询cal enable位有没有使能
void AHT20_SendAC(void); //向AHT20发送AC命令
uint8_t Calc_CRC8(uint8_t *message,uint8_t Num);
void AHT20_Read_CTdata(uint32_t *ct); //没有CRC校验，直接读取AHT20的温度和湿度数据
void AHT20_Read_CTdata_crc(uint32_t *ct); //CRC校验后，读取AHT20的温度和湿度数据
void AHT20_Init(void);   //初始化AHT20
void JH_Reset_REG(uint8_t addr);///重置寄存器
void AHT20_Start_Init(void);///上电初始化进入正常测量状态
#endif

```

 - 修改AHT20-21_DEMO_V1_3.c

```c
/*******************************************/
/*@版权所有：广州奥松电子有限公司          */
/*@作者：温湿度传感器事业部                */
/*@版本：V1.2                              */
/*******************************************/
//#include "main.h" 
#include "AHT20-21_DEMO_V1_3.h" 
#include "gpio.h"
#include "i2c.h"


void Delay_N10us(uint32_t t)//延时函数
{
  uint32_t k;

   while(t--)
  {
    for (k = 0; k < 2; k++);//110
  }
}

void SensorDelay_us(uint32_t t)//延时函数
{
		
	for(t = t-2; t>0; t--)
	{
		Delay_N10us(1);
	}
}

void Delay_4us(void)		//延时函数
{	
	Delay_N10us(1);
	Delay_N10us(1);
	Delay_N10us(1);
	Delay_N10us(1);
}
void Delay_5us(void)		//延时函数
{	
	Delay_N10us(1);
	Delay_N10us(1);
	Delay_N10us(1);
	Delay_N10us(1);
	Delay_N10us(1);

}

void Delay_1ms(uint32_t t)		//延时函数
{
   while(t--)
  {
    SensorDelay_us(1000);//延时1ms
  }
}


//void AHT20_Clock_Init(void)		//延时函数
//{
//	RCC_APB2PeriphClockCmd(CC_APB2Periph_GPIOB,ENABLE);
//}

void SDA_Pin_Output_High(void)   //将PB7配置为输出 ， 并设置为高电平， PB7作为I2C的SDA
{
	GPIO_InitTypeDef  GPIO_InitStruct;
	GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;//推挽输出
	GPIO_InitStruct.Pin = GPIO_PIN_7;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
	HAL_GPIO_Init(GPIOB,& GPIO_InitStruct);
	HAL_GPIO_WritePin(GPIOB,GPIO_PIN_7,GPIO_PIN_SET);
}

void SDA_Pin_Output_Low(void)  //将P7配置为输出  并设置为低电平
{
	GPIO_InitTypeDef  GPIO_InitStruct;
	GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;//推挽输出
	GPIO_InitStruct.Pin = GPIO_PIN_7;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
	HAL_GPIO_Init(GPIOB,& GPIO_InitStruct);
	HAL_GPIO_WritePin(GPIOB,GPIO_PIN_7,GPIO_PIN_RESET);
}

void SDA_Pin_IN_FLOATING(void)  //SDA配置为浮空输入
{
	GPIO_InitTypeDef  GPIO_InitStruct;
	GPIO_InitStruct.Mode = GPIO_MODE_INPUT;//浮空
	GPIO_InitStruct.Pin = GPIO_PIN_7;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
	HAL_GPIO_Init( GPIOB,&GPIO_InitStruct);
}


void SCL_Pin_Output_High(void) //SCL输出高电平，P14作为I2C的SCL
{
	HAL_GPIO_WritePin(GPIOB,GPIO_PIN_6,GPIO_PIN_SET);
}

void SCL_Pin_Output_Low(void) //SCL输出低电平
{
	HAL_GPIO_WritePin(GPIOB,GPIO_PIN_6,GPIO_PIN_RESET);
}

void Init_I2C_Sensor_Port(void) //初始化I2C接口,输出为高电平
{	
	GPIO_InitTypeDef  GPIO_InitStruct;
	GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;//推挽输出
	GPIO_InitStruct.Pin = GPIO_PIN_7;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
	HAL_GPIO_Init(GPIOB,& GPIO_InitStruct);
	HAL_GPIO_WritePin(GPIOB,GPIO_PIN_15,GPIO_PIN_SET);

	
	GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;//推挽输出
	GPIO_InitStruct.Pin = GPIO_PIN_6;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
	HAL_GPIO_Init(GPIOB,& GPIO_InitStruct);
	HAL_GPIO_WritePin(GPIOB,GPIO_PIN_15,GPIO_PIN_SET);
	
}
void I2C_Start(void)		 //I2C主机发送START信号
{
	SDA_Pin_Output_High();
	SensorDelay_us(8);
	SCL_Pin_Output_High();
	SensorDelay_us(8);
	SDA_Pin_Output_Low();
	SensorDelay_us(8);
	SCL_Pin_Output_Low();
	SensorDelay_us(8);   
}


void AHT20_WR_Byte(uint8_t Byte) //往AHT20写一个字节
{
	uint8_t Data,N,i;	
	Data=Byte;
	i = 0x80;
	for(N=0;N<8;N++)
	{
		SCL_Pin_Output_Low(); 
		Delay_4us();	
		if(i&Data)
		{
			SDA_Pin_Output_High();
		}
		else
		{
			SDA_Pin_Output_Low();
		}	
			
    SCL_Pin_Output_High();
		Delay_4us();
		Data <<= 1;
		 
	}
	SCL_Pin_Output_Low();
	SensorDelay_us(8);   
	SDA_Pin_IN_FLOATING();
	SensorDelay_us(8);	
}	


uint8_t AHT20_RD_Byte(void)//从AHT20读取一个字节
{
		uint8_t Byte,i,a;
	Byte = 0;
	SCL_Pin_Output_Low();
	
	SDA_Pin_IN_FLOATING();
	SensorDelay_us(8);	
	
	for(i=0;i<8;i++)
	{
    SCL_Pin_Output_High();
		
		Delay_5us();
		a=0;
		
		//if(GPIO_ReadInputDataBit(GPIOB,GPIO_Pin_15)) a=1;
		if(HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_7)) a=1;
		Byte = (Byte<<1)|a;
		
		//SCL_Pin_Output_Low();
		HAL_GPIO_WritePin(GPIOB,GPIO_PIN_6,GPIO_PIN_RESET);
		Delay_5us();
	}
  SDA_Pin_IN_FLOATING();
	SensorDelay_us(8);	
	return Byte;
}


uint8_t Receive_ACK(void)   //看AHT20是否有回复ACK
{
	uint16_t CNT;
	CNT = 0;
	SCL_Pin_Output_Low();	
	SDA_Pin_IN_FLOATING();
	SensorDelay_us(8);	
	SCL_Pin_Output_High();	
	SensorDelay_us(8);	
	while((HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_7))  && CNT < 100) 
	CNT++;
	if(CNT == 100)
	{
		return 0;
	}
 	SCL_Pin_Output_Low();	
	SensorDelay_us(8);	
	return 1;
}

void Send_ACK(void)		  //主机回复ACK信号
{
	SCL_Pin_Output_Low();	
	SensorDelay_us(8);	
	SDA_Pin_Output_Low();
	SensorDelay_us(8);	
	SCL_Pin_Output_High();	
	SensorDelay_us(8);
	SCL_Pin_Output_Low();	
	SensorDelay_us(8);
	SDA_Pin_IN_FLOATING();
	SensorDelay_us(8);
}

void Send_NOT_ACK(void)	//主机不回复ACK
{
	SCL_Pin_Output_Low();	
	SensorDelay_us(8);
	SDA_Pin_Output_High();
	SensorDelay_us(8);
	SCL_Pin_Output_High();	
	SensorDelay_us(8);		
	SCL_Pin_Output_Low();	
	SensorDelay_us(8);
    SDA_Pin_Output_Low();
	SensorDelay_us(8);
}

void Stop_I2C(void)	  //一条协议结束
{
	SDA_Pin_Output_Low();
	SensorDelay_us(8);
	SCL_Pin_Output_High();	
	SensorDelay_us(8);
	SDA_Pin_Output_High();
	SensorDelay_us(8);
}

uint8_t AHT20_Read_Status(void)//读取AHT20的状态寄存器
{

	uint8_t Byte_first;	
	I2C_Start();
	AHT20_WR_Byte(0x71);
	Receive_ACK();
	Byte_first = AHT20_RD_Byte();
	Send_NOT_ACK();
	Stop_I2C();
	return Byte_first;
}

uint8_t AHT20_Read_Cal_Enable(void)  //查询cal enable位有没有使能
{
	uint8_t val = 0;//ret = 0,
  val = AHT20_Read_Status();
	 if((val & 0x68)==0x08)
		 return 1;
   else  return 0;
 }

void AHT20_SendAC(void) //向AHT20发送AC命令
{

	I2C_Start();
	AHT20_WR_Byte(0x70);
	Receive_ACK();
	AHT20_WR_Byte(0xac);//0xAC采集命令
	Receive_ACK();
	AHT20_WR_Byte(0x33);
	Receive_ACK();
	AHT20_WR_Byte(0x00);
	Receive_ACK();
	Stop_I2C();

}

//CRC校验类型：CRC8/MAXIM
//多项式：X8+X5+X4+1
//Poly：0011 0001  0x31
//高位放到后面就变成 1000 1100 0x8c
//C现实代码：
uint8_t Calc_CRC8(uint8_t *message,uint8_t Num)
{
	uint8_t i;
	uint8_t byte;
	uint8_t crc=0xFF;
  for(byte=0; byte<Num; byte++)
  {
    crc^=(message[byte]);
    for(i=8;i>0;--i)
    {
      if(crc&0x80) crc=(crc<<1)^0x31;
      else crc=(crc<<1);
    }
  }
        return crc;
}

void AHT20_Read_CTdata(uint32_t *ct) //没有CRC校验，直接读取AHT20的温度和湿度数据
{
		volatile uint8_t  Byte_1th=0;
	volatile uint8_t  Byte_2th=0;
	volatile uint8_t  Byte_3th=0;
	volatile uint8_t  Byte_4th=0;
	volatile uint8_t  Byte_5th=0;
	volatile uint8_t  Byte_6th=0;
	 uint32_t RetuData = 0;
	uint16_t cnt = 0;
	AHT20_SendAC();//向AHT10发送AC命令
	Delay_1ms(80);//延时80ms左右	
    cnt = 0;
	while(((AHT20_Read_Status()&0x80)==0x80))//直到状态bit[7]为0，表示为空闲状态，若为1，表示忙状态
	{
		SensorDelay_us(1508);
		if(cnt++>=100)
		{
		 break;
		 }
	}
	I2C_Start();
	AHT20_WR_Byte(0x71);
	Receive_ACK();
	Byte_1th = AHT20_RD_Byte();//状态字，查询到状态为0x98,表示为忙状态，bit[7]为1；状态为0x1C，或者0x0C，或者0x08表示为空闲状态，bit[7]为0
	Send_ACK();
	Byte_2th = AHT20_RD_Byte();//湿度
	Send_ACK();
	Byte_3th = AHT20_RD_Byte();//湿度
	Send_ACK();
	Byte_4th = AHT20_RD_Byte();//湿度/温度
	Send_ACK();
	Byte_5th = AHT20_RD_Byte();//温度
	Send_ACK();
	Byte_6th = AHT20_RD_Byte();//温度
	Send_NOT_ACK();
	Stop_I2C();

	RetuData = (RetuData|Byte_2th)<<8;
	RetuData = (RetuData|Byte_3th)<<8;
	RetuData = (RetuData|Byte_4th);
	RetuData =RetuData >>4;
	ct[0] = RetuData;//湿度
	RetuData = 0;
	RetuData = (RetuData|Byte_4th)<<8;
	RetuData = (RetuData|Byte_5th)<<8;
	RetuData = (RetuData|Byte_6th);
	RetuData = RetuData&0xfffff;
	ct[1] =RetuData; //温度

}


void AHT20_Read_CTdata_crc(uint32_t *ct) //CRC校验后，读取AHT20的温度和湿度数据
{
		volatile uint8_t  Byte_1th=0;
	volatile uint8_t  Byte_2th=0;
	volatile uint8_t  Byte_3th=0;
	volatile uint8_t  Byte_4th=0;
	volatile uint8_t  Byte_5th=0;
	volatile uint8_t  Byte_6th=0;
	volatile uint8_t  Byte_7th=0;
	 uint32_t RetuData = 0;
	 uint16_t cnt = 0;
	// uint8_t  CRCDATA=0;
	 uint8_t  CTDATA[6]={0};//用于CRC传递数组
	
	AHT20_SendAC();//向AHT10发送AC命令
	Delay_1ms(80);//延时80ms左右	
    cnt = 0;
	while(((AHT20_Read_Status()&0x80)==0x80))//直到状态bit[7]为0，表示为空闲状态，若为1，表示忙状态
	{
		SensorDelay_us(1508);
		if(cnt++>=100)
		{
		 break;
		}
	}
	
	I2C_Start();

	AHT20_WR_Byte(0x71);
	Receive_ACK();
	CTDATA[0]=Byte_1th = AHT20_RD_Byte();//状态字，查询到状态为0x98,表示为忙状态，bit[7]为1；状态为0x1C，或者0x0C，或者0x08表示为空闲状态，bit[7]为0
	Send_ACK();
	CTDATA[1]=Byte_2th = AHT20_RD_Byte();//湿度
	Send_ACK();
	CTDATA[2]=Byte_3th = AHT20_RD_Byte();//湿度
	Send_ACK();
	CTDATA[3]=Byte_4th = AHT20_RD_Byte();//湿度/温度
	Send_ACK();
	CTDATA[4]=Byte_5th = AHT20_RD_Byte();//温度
	Send_ACK();
	CTDATA[5]=Byte_6th = AHT20_RD_Byte();//温度
	Send_ACK();
	Byte_7th = AHT20_RD_Byte();//CRC数据
	Send_NOT_ACK();                           //注意: 最后是发送NAK
	Stop_I2C();
	
	if(Calc_CRC8(CTDATA,6)==Byte_7th)
	{
	RetuData = (RetuData|Byte_2th)<<8;
	RetuData = (RetuData|Byte_3th)<<8;
	RetuData = (RetuData|Byte_4th);
	RetuData =RetuData >>4;
	ct[0] = RetuData;//湿度
	RetuData = 0;
	RetuData = (RetuData|Byte_4th)<<8;
	RetuData = (RetuData|Byte_5th)<<8;
	RetuData = (RetuData|Byte_6th);
	RetuData = RetuData&0xfffff;
	ct[1] =RetuData; //温度
		
	}
	else
	{
		ct[0]=0x00;
		ct[1]=0x00;//校验错误返回值，客户可以根据自己需要更改
	}//CRC数据
}


void AHT20_Init(void)   //初始化AHT20
{	
	Init_I2C_Sensor_Port();
	I2C_Start();
	AHT20_WR_Byte(0x70);
	Receive_ACK();
	AHT20_WR_Byte(0xa8);//0xA8进入NOR工作模式
	Receive_ACK();
	AHT20_WR_Byte(0x00);
	Receive_ACK();
	AHT20_WR_Byte(0x00);
	Receive_ACK();
	Stop_I2C();

	Delay_1ms(10);//延时10ms左右

	I2C_Start();
	AHT20_WR_Byte(0x70);
	Receive_ACK();
	AHT20_WR_Byte(0xbe);//0xBE初始化命令，AHT20的初始化命令是0xBE,   AHT10的初始化命令是0xE1
	Receive_ACK();
	AHT20_WR_Byte(0x08);//相关寄存器bit[3]置1，为校准输出
	Receive_ACK();
	AHT20_WR_Byte(0x00);
	Receive_ACK();
	Stop_I2C();
	Delay_1ms(10);//延时10ms左右
}
void JH_Reset_REG(uint8_t addr)
{
	
	uint8_t Byte_first,Byte_second,Byte_third;
	I2C_Start();
	AHT20_WR_Byte(0x70);//原来是0x70
	Receive_ACK();
	AHT20_WR_Byte(addr);
	Receive_ACK();
	AHT20_WR_Byte(0x00);
	Receive_ACK();
	AHT20_WR_Byte(0x00);
	Receive_ACK();
	Stop_I2C();

	Delay_1ms(5);//延时5ms左右
	I2C_Start();
	AHT20_WR_Byte(0x71);//
	Receive_ACK();
	Byte_first = AHT20_RD_Byte();
	Send_ACK();
	Byte_second = AHT20_RD_Byte();
	Send_ACK();
	Byte_third = AHT20_RD_Byte();
	Send_NOT_ACK();
	Stop_I2C();
	
  Delay_1ms(10);//延时10ms左右
	I2C_Start();
	AHT20_WR_Byte(0x70);///
	Receive_ACK();
	AHT20_WR_Byte(0xB0|addr);//寄存器命令
	Receive_ACK();
	AHT20_WR_Byte(Byte_second);
	Receive_ACK();
	AHT20_WR_Byte(Byte_third);
	Receive_ACK();
	Stop_I2C();
	
	Byte_second=0x00;
	Byte_third =0x00;
}

void AHT20_Start_Init(void)
{
	JH_Reset_REG(0x1b);
	JH_Reset_REG(0x1c);
	JH_Reset_REG(0x1e);
}

```

 - 修改mian.c

```c
/* USER CODE BEGIN Header */
/**
  ******************************************************************************
  * @file           : main.c
  * @brief          : Main program body
  ******************************************************************************
  * @attention
  *
  * Copyright (c) 2022 STMicroelectronics.
  * All rights reserved.
  *
  * This software is licensed under terms that can be found in the LICENSE file
  * in the root directory of this software component.
  * If no LICENSE file comes with this software, it is provided AS-IS.
  *
  ******************************************************************************
  */
/* USER CODE END Header */
/* Includes ------------------------------------------------------------------*/
#include "main.h"
#include "dma.h"
#include "i2c.h"
#include "usart.h"
#include "gpio.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include<stdio.h>
#include "AHT20-21_DEMO_V1_3.h" 
/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */
int fputc(int ch,FILE *f)
{
    HAL_UART_Transmit(&huart1,(uint8_t *)&ch,1,0xFFFF);    
		//等待发送结束	
		while(__HAL_UART_GET_FLAG(&huart1,UART_FLAG_TC)!=SET){
		}		

    return ch;
}
/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */
/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/

/* USER CODE BEGIN PV */

/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
/* USER CODE BEGIN PFP */

/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */

/* USER CODE END 0 */

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
  /* USER CODE BEGIN 1 */
	uint32_t CT_data[2]={0,0};
	volatile int  c1,t1;
	Delay_1ms(500);
  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  
  MX_USART1_UART_Init();
  MX_DMA_Init();
  MX_I2C1_Init();	
  MX_USART1_UART_Init();
	
  /* USER CODE BEGIN 2 */
  AHT20_Init();
	Delay_1ms(500);

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */
		//AHT20_Read_CTdata(CT_data);       //不经过CRC校验，直接读取AHT20的温度和湿度数据    推荐每隔大于1S读一次
		AHT20_Read_CTdata_crc(CT_data);  //crc校验后，读取AHT20的温度和湿度数据 
	

		c1 = CT_data[0]*1000/1024/1024;  //计算得到湿度值c1（放大了10倍）
		t1 = CT_data[1]*2000/1024/1024-500;//计算得到温度值t1（放大了10倍）
		printf("正在检测");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		printf("\r\n");
		HAL_Delay(1000);
		printf("温度:%d%d.%d",t1/100,(t1/10)%10,t1%10);
		printf("湿度:%d%d.%d",c1/100,(c1/10)%10,c1%10);
		printf("\r\n");
		printf("等待");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		HAL_Delay(100);
		printf(".");
		printf("\r\n");
		HAL_Delay(1000);
    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}

/**
  * @brief System Clock Configuration
  * @retval None
  */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
  RCC_OscInitStruct.HSEState = RCC_HSE_ON;
  RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  /** Initializes the CPU, AHB and APB buses clocks
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
  {
    Error_Handler();
  }
}

/* USER CODE BEGIN 4 */

/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */
  __disable_irq();
  while (1)
  {
  }
  /* USER CODE END Error_Handler_Debug */
}

#ifdef  USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{
  /* USER CODE BEGIN 6 */
  /* User can add his own implementation to report the file name and line number,
     ex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */

```
在**main.c**和**usart.c**中添加头文件 **#include "stdio.h"** 之后，勾选 **Target** 中的 **use MicroLIB**
完成后点击编译
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/047356a3e1d3498a986a96ce4e9dcb9b.png#pic_center)
# 五、硬件连接
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/e5f4a559e1534b4f85c2bdc8accad5e3.png#pic_center)
连接图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/572825b5289643ffadfa3a6bf5e5a9f8.jpeg#pic_center)
# 六、总结
通过本次实验，熟悉了 STM32 I2C 外设的配置和使用，掌握了如何与外部传感器进行通信并采集数据。同时，了解了 AHT20 传感器的工作原理和数据格式

