@[TOC]
# 一、µC/OS简介
##  1.1 μC／OS-II概述
μC／OS-II是一个完整的，可移植、可固化、可裁剪的占先式实时多任务内核；支持56个用户任务，支持信号量、邮箱、消息队列等常用的进程间通信机制；适用于各种微控制器和微处理器；所有代码用ANSI C语言编写，程序的可读性强，具有良好的可移植性，已被移植到多种处理器架构中，在某些实时性要求严格的领域中得到广泛应用。
## 1.2 工作原理
μC／OS-II的核心工作原理是：近似地让最高优先级的就绪任务处于运行状态。首先初始化MCU，再进行操作系统初始化，主要完成任务控制块TCB初始化，TCB优先级表初始化，TCB链表初始化，事件控制块(ECB)链表初始化，空任务的创建等。然后，开始创建新任务，并可在新创建的任务中再创建其他新任务。最后，诃用OSStart()函数启动多任务调度。在多任务调度开始后，启动时钟节拍源开始计时，此节拍源给系统提供周期性的时钟中断信号，实现延时和超时确认。
## 1.3 任务调度
操作系统在下面的情况下进行任务调度：中断(系统占用的时间片中断OSTimeTick()、用户使用的中断)和调用API函数(用户主动调用)。一种是当时钟中断来临时，系统把当前正在执行的任务挂起，保护现场，进行中断处理，判断有无任务延时到期；若没有别的任务进入就绪态，则恢复现场继续执行原任务。另一种调度方式是任务级的调度，即调用API函数(由用户主动调用)，足通过发软中断命令或依靠处理器在任务执行中调度。当没有任何任务进入就绪态时，就去执行空任务。
# 二、µC/OS-III 内核获取
官网下载啊地址：
[µC/OS-III获取](https://www.silabs.com/developers/micrium)
# 三、STM32CubeMX 工程配置

 - 配置RCC
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/f1d8677abb094c5b8bed675fa4b0ab38.png#pic_center)
 - 配置SYS且配置PC13为GPIO_Output
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/fca5ac19cdd54d9484538d15dcb4d1b7.png#pic_center)
 - 将与LED相连的两个端口PB0，PB1配置为GPIO_Output，可根据LED现象作为我们是否移植成功的依据。
# 四、代码移植
代码移植的具体操作请参考下面博客进行：

> https://blog.csdn.net/junseven164/article/details/121534916

# 五、代码修改
## 5.1 修改bsp.c和bsp.h文件
bsp.c文件

```c
// bsp.h
#ifndef  __BSP_H__
#define  __BSP_H__

#include "stm32f1xx_hal.h"

void BSP_Init(void);

#endif

```
bsp.h文件

```c
// bsp.c
#include "includes.h"

#define  DWT_CR      *(CPU_REG32 *)0xE0001000
#define  DWT_CYCCNT  *(CPU_REG32 *)0xE0001004
#define  DEM_CR      *(CPU_REG32 *)0xE000EDFC
#define  DBGMCU_CR   *(CPU_REG32 *)0xE0042004

#define  DEM_CR_TRCENA                   (1 << 24)
#define  DWT_CR_CYCCNTENA                (1 <<  0)

CPU_INT32U  BSP_CPU_ClkFreq (void)
{
    return HAL_RCC_GetHCLKFreq();
}

void BSP_Tick_Init(void)
{
	CPU_INT32U cpu_clk_freq;
	CPU_INT32U cnts;
	cpu_clk_freq = BSP_CPU_ClkFreq();
	
	#if(OS_VERSION>=3000u)
		cnts = cpu_clk_freq/(CPU_INT32U)OSCfg_TickRate_Hz;
	#else
		cnts = cpu_clk_freq/(CPU_INT32U)OS_TICKS_PER_SEC;
	#endif
	OS_CPU_SysTickInit(cnts);
}



void BSP_Init(void)
{
	BSP_Tick_Init();
	MX_GPIO_Init();
}


#if (CPU_CFG_TS_TMR_EN == DEF_ENABLED)
void  CPU_TS_TmrInit (void)
{
    CPU_INT32U  cpu_clk_freq_hz;


    DEM_CR         |= (CPU_INT32U)DEM_CR_TRCENA;                /* Enable Cortex-M3's DWT CYCCNT reg.                   */
    DWT_CYCCNT      = (CPU_INT32U)0u;
    DWT_CR         |= (CPU_INT32U)DWT_CR_CYCCNTENA;

    cpu_clk_freq_hz = BSP_CPU_ClkFreq();
    CPU_TS_TmrFreqSet(cpu_clk_freq_hz);
}
#endif


#if (CPU_CFG_TS_TMR_EN == DEF_ENABLED)
CPU_TS_TMR  CPU_TS_TmrRd (void)
{
    return ((CPU_TS_TMR)DWT_CYCCNT);
}
#endif


#if (CPU_CFG_TS_32_EN == DEF_ENABLED)
CPU_INT64U  CPU_TS32_to_uSec (CPU_TS32  ts_cnts)
{
	CPU_INT64U  ts_us;
  CPU_INT64U  fclk_freq;

 
  fclk_freq = BSP_CPU_ClkFreq();
  ts_us     = ts_cnts / (fclk_freq / DEF_TIME_NBR_uS_PER_SEC);

  return (ts_us);
}
#endif
 
 
#if (CPU_CFG_TS_64_EN == DEF_ENABLED)
CPU_INT64U  CPU_TS64_to_uSec (CPU_TS64  ts_cnts)
{
	CPU_INT64U  ts_us;
	CPU_INT64U  fclk_freq;


  fclk_freq = BSP_CPU_ClkFreq();
  ts_us     = ts_cnts / (fclk_freq / DEF_TIME_NBR_uS_PER_SEC);
	
  return (ts_us);
}
#endif

```
## 5.2 修改main.c代码

```c
/* USER CODE END Header */
/* Includes ------------------------------------------------------------------*/
#include "main.h"
#include "gpio.h"
#include "usart.h"
/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include <includes.h>
#include "stm32f1xx_hal.h"
/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */

/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */
/* 任务优先级 */
#define START_TASK_PRIO		3
#define LED0_TASK_PRIO		4
#define MSG_TASK_PRIO		5

/* 任务堆栈大小	*/
#define START_STK_SIZE 		64
#define LED0_STK_SIZE 		64
#define MSG_STK_SIZE 		64//任务堆大小过大会报错，可以试着改小一点

/* 任务栈 */	
CPU_STK START_TASK_STK[START_STK_SIZE];
CPU_STK LED0_TASK_STK[LED0_STK_SIZE];
CPU_STK MSG_TASK_STK[MSG_STK_SIZE];
/* 任务控制块 */
OS_TCB StartTaskTCB;
OS_TCB Led0TaskTCB;
OS_TCB MsgTaskTCB;
/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/

/* USER CODE BEGIN PV */

/* 任务函数定义 */
void start_task(void *p_arg);
static  void  AppTaskCreate(void);
static  void  AppObjCreate(void);
static  void  led_pc13(void *p_arg);
static  void  send_msg(void *p_arg);
/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
/* USER CODE BEGIN PFP */

/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
/**
  * @brief System Clock Configuration
  * @retval None
  */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /**Initializes the CPU, AHB and APB busses clocks 
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
  /**Initializes the CPU, AHB and APB busses clocks 
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

/* USER CODE END 0 */

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
	OS_ERR  err;
	OSInit(&err);
  HAL_Init();
	SystemClock_Config();
	//MX_GPIO_Init(); 这个在BSP的初始化里也会初始化
  MX_USART1_UART_Init();	
	/* 创建任务 */
	OSTaskCreate((OS_TCB     *)&StartTaskTCB,                /* Create the start task                                */
				 (CPU_CHAR   *)"start task",
				 (OS_TASK_PTR ) start_task,
				 (void       *) 0,
				 (OS_PRIO     ) START_TASK_PRIO,
				 (CPU_STK    *)&START_TASK_STK[0],
				 (CPU_STK_SIZE) START_STK_SIZE/10,
				 (CPU_STK_SIZE) START_STK_SIZE,
				 (OS_MSG_QTY  ) 0,
				 (OS_TICK     ) 0,
				 (void       *) 0,
				 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR),
				 (OS_ERR     *)&err);
	/* 启动多任务系统，控制权交给uC/OS-III */
	OSStart(&err);            /* Start multitasking (i.e. give control to uC/OS-III). */
               
}


void start_task(void *p_arg)
{
	OS_ERR err;
	CPU_SR_ALLOC();
	p_arg = p_arg;
	
	/* YangJie add 2021.05.20*/
  BSP_Init();                                                   /* Initialize BSP functions */
  //CPU_Init();
  //Mem_Init();                                                 /* Initialize Memory Management Module */

#if OS_CFG_STAT_TASK_EN > 0u
   OSStatTaskCPUUsageInit(&err);  		//统计任务                
#endif
	
#ifdef CPU_CFG_INT_DIS_MEAS_EN			//如果使能了测量中断关闭时间
    CPU_IntDisMeasMaxCurReset();	
#endif

#if	OS_CFG_SCHED_ROUND_ROBIN_EN  		//当使用时间片轮转的时候
	 //使能时间片轮转调度功能,时间片长度为1个系统时钟节拍，既1*5=5ms
	OSSchedRoundRobinCfg(DEF_ENABLED,1,&err);  
#endif		
	
	OS_CRITICAL_ENTER();	//进入临界区
	/* 创建LED0任务 */
	OSTaskCreate((OS_TCB 	* )&Led0TaskTCB,		
				 (CPU_CHAR	* )"led_pc13", 		
                 (OS_TASK_PTR )led_pc13, 			
                 (void		* )0,					
                 (OS_PRIO	  )LED0_TASK_PRIO,     
                 (CPU_STK   * )&LED0_TASK_STK[0],	
                 (CPU_STK_SIZE)LED0_STK_SIZE/10,	
                 (CPU_STK_SIZE)LED0_STK_SIZE,		
                 (OS_MSG_QTY  )0,					
                 (OS_TICK	  )0,					
                 (void   	* )0,					
                 (OS_OPT      )OS_OPT_TASK_STK_CHK|OS_OPT_TASK_STK_CLR,
                 (OS_ERR 	* )&err);				
				 
	/* 创建LED1任务 */
	OSTaskCreate((OS_TCB 	* )&MsgTaskTCB,		
				 (CPU_CHAR	* )"send_msg", 		
                 (OS_TASK_PTR )send_msg, 			
                 (void		* )0,					
                 (OS_PRIO	  )MSG_TASK_PRIO,     	
                 (CPU_STK   * )&MSG_TASK_STK[0],	
                 (CPU_STK_SIZE)MSG_STK_SIZE/10,	
                 (CPU_STK_SIZE)MSG_STK_SIZE,		
                 (OS_MSG_QTY  )0,					
                 (OS_TICK	  )0,					
                 (void   	* )0,				
                 (OS_OPT      )OS_OPT_TASK_STK_CHK|OS_OPT_TASK_STK_CLR, 
                 (OS_ERR 	* )&err);
				 
	OS_TaskSuspend((OS_TCB*)&StartTaskTCB,&err);		//挂起开始任务			 
	OS_CRITICAL_EXIT();	//进入临界区
}
/**
  * 函数功能: 启动任务函数体。
  * 输入参数: p_arg 是在创建该任务时传递的形参
  * 返 回 值: 无
  * 说    明：无
  */
static  void  led_pc13 (void *p_arg)
{
  OS_ERR      err;

  (void)p_arg;

  BSP_Init();                                                 /* Initialize BSP functions                             */
  CPU_Init();

  Mem_Init();                                                 /* Initialize Memory Management Module                  */

#if OS_CFG_STAT_TASK_EN > 0u
  OSStatTaskCPUUsageInit(&err);                               /* Compute CPU capacity with no task running            */
#endif

  CPU_IntDisMeasMaxCurReset();

  AppTaskCreate();                                            /* Create Application Tasks                             */

  AppObjCreate();                                             /* Create Application Objects                           */

  while (DEF_TRUE)
  {
		HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13,GPIO_PIN_RESET);
		OSTimeDlyHMSM(0, 0, 0, 500,OS_OPT_TIME_HMSM_STRICT,&err);
		HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13,GPIO_PIN_SET);
		OSTimeDlyHMSM(0, 0, 0, 500,OS_OPT_TIME_HMSM_STRICT,&err);
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
static  void  send_msg (void *p_arg)
{
  OS_ERR      err;

  (void)p_arg;

  BSP_Init();                                                 /* Initialize BSP functions                             */
  CPU_Init();

  Mem_Init();                                                 /* Initialize Memory Management Module                  */

#if OS_CFG_STAT_TASK_EN > 0u
  OSStatTaskCPUUsageInit(&err);                               /* Compute CPU capacity with no task running            */
#endif

  CPU_IntDisMeasMaxCurReset();

  AppTaskCreate();                                            /* Create Application Tasks                             */

  AppObjCreate();                                             /* Create Application Objects                           */

  while (DEF_TRUE)
  {
			printf("hello world \r\n");
		OSTimeDlyHMSM(0, 0, 0, 500,OS_OPT_TIME_HMSM_STRICT,&err);
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}


/* USER CODE BEGIN 4 */
/**
  * 函数功能: 创建应用任务
  * 输入参数: p_arg 是在创建该任务时传递的形参
  * 返 回 值: 无
  * 说    明：无
  */
static  void  AppTaskCreate (void)
{
  
}


/**
  * 函数功能: uCOSIII内核对象创建
  * 输入参数: 无
  * 返 回 值: 无
  * 说    明：无
  */
static  void  AppObjCreate (void)
{

}

/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */

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
     tex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */

/************************ (C) COPYRIGHT STMicroelectronics *****END OF FILE****/

```
## 5.3 修改剩余部分代码
修改startup_stm32f103xb.s文件：
将PendSV_Handler改写为OS_CPU_PendSVHandler
将Systick_Handler改写为OS_CPU_SysTickHandler。
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/d3ccd3dba3e849b49453405add614911.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7d7e6a26c4fc491481229f71c305a1d5.png#pic_center)
修改app_cfg.h文件
将#define APP_CFG_SERIAL_EN DEF_ENABLED改为#define APP_CFG_SERIAL_EN DEF_DISABLED
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/84ad5b8823bf4ae6bd58531def8918b4.png#pic_center)
将#define APP_TRACE BSP_Ser_Printf改为#define APP_TRACE (void)
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/0b6fe263007c43e7941a130dc2fff056.png#pic_center)
修改includes.h文件
在#include <bsp.h>后面添加#include “gpio.h”和#include “app_cfg.h”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/d6222d4b9afc49908c1364031e75214d.png#pic_center)
将#include <stm32f10x_lib.h> 改为#include “stm32f1xx_hal.h”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/0dc36a2430724cd6bcac47cedbd694d7.png#pic_center)
修改lib_cfg.h文件
lib_cfg.h里修改堆空间的宏定义，它原本是 27K，但这块板子RAM一共才20k，修改为5k

```c
#define  LIB_MEM_CFG_HEAP_SIZE          5u * 1024u  

```
最后进行烧录
# 六、总结
通过本实验，掌握了以下关键技能：

 - µC/OS-III 的移植：学会了将实时操作系统移植到指定硬件平台，并进行了必要的配置和初始化。
 - 多任务管理：了解了如何创建和管理多个任务，实现了不同任务的周期性调度和执行。
 - 调试与优化：学会了如何调试嵌入式系统中的任务代码，解决了堆栈溢出和任务调度问题。
