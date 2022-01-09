# RTOS-9
**嵌入式实时操作系统9——中断系统**

## 1.中断是什么

中断是计算机中一个非常重要的概念，现代计算机中毫无例外地都要采用中断技术。

早期的计算机没有中断系统，人们往往需要等上一个任务运行结束才能运行下一个任务，这极大的限制了计算机的执行效率。早期计算机如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/6de1a7a80b074358beb88ecd0ee08686.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)

事实上中断系统出现的很早，Intel的传奇中断控制芯片8259在1976年就被用在8085系列产品中。现代计算机中毫无例外地都要采用中断技术！

当计算机在运行时，很多事情在“同时”发生，磁盘在快速读写，网络在收发数据，而用户也往往同时运行了多个应用程序。这些能够发生，中断系统扮演了关键角色。

中断是一种基于硬件的机制，用于通知CPU发生了一个异步事件。CPU确认中断后，跳转并执行一个特殊的函数，这个特殊的函数叫中断服务程序。

![请添加图片描述](https://img-blog.csdnimg.cn/952302e80b104f77ba75715854effbc3.gif)

根据中断发生时间可以讲中断分为**同步中断**和**异步中断**：

> 1、同步中断是执行指令时由CPU产生的，之所以称之为同步，是因为只有在一条特殊指令被执行后CPU才会发出中断（如执行了一条未定义指令），这类型的中断也被称为异常。
> 2、异步中断时由其他硬件设备随机触发CPU产生的，这类型的中断也被称为外部中断。

根据中断服务函数入口地址可以将中断分为**向量中断**和**非向量中断**：

> 1、在向量中断中，不同的中断号由不同的入口地址。 
> 2、在非向量中断中，多个中断共享一个入口地址。

## 2.中断硬件实现

中断是一种基于硬件的机制，目前的主流处理器都是向量中断结构。以ARM 体系处理为参考分析中断机制的硬件实现。
在ARM向量中断体系中有一个向量中断控制器VIC,其特点如下：
1、支持多个中断请求输入。
2、支持动态分配优先级。
3、支持向量地址配置。
向量中断控制器VIC的模型框图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/6df9dcfdced544c99453872561829b7e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

通俗的理解：向量中断控制器VIC根据中断输入信号，给CPU产生一个中断信号，同时将中断的服务的地址加载给PC，从而跳转到中断服务程序中。
中断机制是通过硬件来实现。







## 3.操作系统中断服务程序

操作系统的任务有优先级，处理器中断也有优先级，可以将这两个优先级汇总在一起，框图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/01bbaaa04032428a9c6dbb53a20f07dc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_11,color_FFFFFF,t_70,g_se,x_16)

由图可知中断优先级高于任务优先级。
操作系统中中断服务程序可以分为两类：**操作系统内核不参入的中断服务程序**和**操作系统内核参入的中断服务程序**。

**操作系统内核不参入的中断服务程序**
在很多情况下，中断服务程序需要做的处理非常简单，并不需要向任务发布消息或信息（使用操作系统API），这种情况下的中断服务程序的代码框架如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/7e7d10b60cd04c02a126416eeacf2e3a.png)

进入中断服务程序后，直接完成需要的处理操作，完成操作后硬件自动退出中断返回用户程序，继续执行被打断的程序。操作系统内核不参入的中断服务程序的运行关系图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/f9369edc56eb43c4b73b59769a6fc118.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_18,color_FFFFFF,t_70,g_se,x_16)


**操作系统内核参入的中断服务程序**
在有些情况下，中断服务程序需要向任务发布消息或信号（使用操作系统API），因为发布布消息或信号有可能会导致高优先级的任务就绪，当完成中断操作后这时操作系统不能直接返回刚刚被中断打断的任务，而是需要执行高优先级任务，因此在**操作系统内核参入的中断服务程序末尾我们需要增加一种任务切换的机制。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/e288bde40297412cacb427d16f19ba31.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_17,color_FFFFFF,t_70,g_se,x_16)

进入中断服务程序后，完成需要的处理操作，向任务发布消息或信号，执行任务切换（当有高优先级任务就绪时切换任务）。操作系统内核参入的中断服务程序的运行关系图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/8d25fa5af8404303b0384aa757a97622.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)



## 4.中断嵌套

中断中存在一种较为复杂的情况：中断嵌套。中断嵌套就是一个中断程序正在处理时，又发了一个高优先级的中断，中断嵌套的关系图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b4d4443dca234f889c8d461c8d739ffc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)

对于操作系统内核不参入的中断服务程序，出现嵌套现象不会产生影响。
对于操作系统内核参入的中断服务程序，出现嵌套现象会产生影响。
前文我们提到操作系统内核参入的中断服务程序末尾有任务切换操作。这样将会出现以下多次切换任务的情况，同时任务切换1还有导致在中断嵌套未处理完成直接切换到用任务，导致中断阻塞，中断嵌套的关系图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2dcc9be3fe4445eb9165ce2b02762ea9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)









因此我们需要有一个中断嵌套机制避免上述问题，解决方法有两种：
**1、中断屏蔽
2、低优先级任务切换**

**中断屏蔽**
为将中断分为两种内核不参入和内核参入如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/24fe3e548e224cdb9d47f8f591eb3ba2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_11,color_FFFFFF,t_70,g_se,x_16)

因此我们需要无内核参入的中断放在最高优先级，无内核参入的中断服务需要“短平快”，有内核参入的中断服务使用低优先级，同时有内核参入的**中断服务进中断后会屏蔽所有有内核参入的中断**，使得只有一个中断能“**独享**”操作系统内核。利用这种机制就可以解决中断嵌套给操作系统内核带来的中断返回问题。


















**低优先级任务切换**
将在内核参入的中断返回时切换任务策略修改为，内核参入的中断返回时设置一个最低优先级中断标志，等内核参入的中断完成后，由一个低优先级中断完成任务切换操作。
以CORTEX-M4为例，cortex-M4核有一个PendSV(可挂起的系统调用)异常，其异常编号为14并且具有可编程的优先级。当软件将PendSV设置成挂起时，程序将进入PendSV异常（可中断）。系统优先级如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/81cdd476d8614a47ac2a50e312a5d723.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_11,color_FFFFFF,t_70,g_se,x_16)

将PendSV异常优先级设置为最低，这样其他的中断函数都可以正常响应中断，不会受到PendSV异常影响，在PendSV异常中执行任务切换时序框图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/4bfa3d1291444da2a355f57b32cdde0a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_17,color_FFFFFF,t_70,g_se,x_16)

中断服务程序中操作内核API，使得高优先级任务就绪，此时设置PendSV标志位，等所有中断完成后，系统才进入最低优先级中断PendSV进行任务切换。




## 5.FreeRTOS示例

在硬件为cortex-m3的FREERTOS系统中，一个串口中断服务如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/f2defcf323dd40249f0f36ed21a69693.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

在串口中断服务中，处理了串口硬件的数据，然后给相关任务发送了信号，使得相关任务就绪，在中断返回时调用了切换任务接口。任务切换接口代码如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c57b9c2b7cc47ffb9f4131791ef808d.png)

portYIELD_FROM_ISR宏最终将设置PendSV中断的标志位，当其他中断结束后，PendSV中断会进行任务切换（PendSV中断被设计成任务切换功能）。




## 6.源码

```c
void UART4_IRQHandler(void)
{
	static portBASE_TYPE xHigherPriorityTaskWoken;	
	__disable_irq();
	xHigherPriorityTaskWoken = pdFALSE;	
	if(USART_GetITStatus(UART4, USART_IT_IDLE) == SET)
	{	

		useless_ram_data = UART4->DR; 	
		DMA_Cmd(DMA2_Channel3,DISABLE);    	
		Common_Cache_Data(CACHE_BUFF_NUM - DMA_GetCurrDataCounter(DMA2_Channel3),&rs485_commnuication_frame);				
		DMA2_Channel3->CNDTR=CACHE_BUFF_NUM;			
		DMA_Cmd(DMA2_Channel3,ENABLE); 
		/* 释放信号 */
		xSemaphoreGiveFromISR(xSem_uart4_com_recv, &xHigherPriorityTaskWoken);	
		
		USART_ClearITPendingBit(UART4, USART_IT_TC);
		USART_ClearITPendingBit(UART4, USART_IT_IDLE);
		
	}

	__enable_irq();
	portYIELD_FROM_ISR( xHigherPriorityTaskWoken ); /* 切换任务 */
}


#define portYIELD_FROM_ISR( x ) portEND_SWITCHING_ISR( x )

#define portEND_SWITCHING_ISR( xSwitchRequired ) if( xSwitchRequired != pdFALSE ) portYIELD()

/* 切换任务 */
#define portYIELD() 															\
{																				\
	/* 设置PendSV 标志位 */								\
	portNVIC_INT_CTRL_REG = portNVIC_PENDSVSET_BIT;								\
	__dsb( portSY_FULL_READ_WRITE );											\
	__isb( portSY_FULL_READ_WRITE );											\
}
```
**未完待续…

**实时操作系统系列将持续更新

**创作不易希望朋友们点赞，转发，评论，关注。

**您的点赞，转发，评论，关注将是我持续更新的动力

**作者：李巍

**Github：liyinuoman2017
