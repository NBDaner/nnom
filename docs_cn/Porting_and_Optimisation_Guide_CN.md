# 移植

移植并不是必须的，因为NNoM是一个纯C框架。如果您的开发环境支持标准C库（libc），在默认的设置下，运行程序没有任何问题。

但是，在某些平台上，通过切换后端或者提供打印模型的信息并进行评估，发现移植可以获得更好的性能。

## 选项
通过修改 `port/` 下的 `nnom_port.h` 来简单地完成移植。

默认设置如下所示。
~~~C
// memory interfaces
#define nnom_malloc(n)      malloc(n) 
#define nnom_free(p)        free(p)
#define nnom_memset(p,v,s)  memset(p,v,s)

// runtime & debuges
#define nnom_us_get()       0
#define nnom_ms_get()       0
#define NNOM_LOG(...)       printf(__VA_ARGS__)

// NNoM configuration
#define NNOM_BLOCK_NUM      (8)		// maximum number of memory block  
#define DENSE_WEIGHT_OPT    (1)		// if used fully connected layer optimized weights. 

// Formate configuration
//#define NNOM_USING_CHW  			// using CHW format instead of default HWC
//#define NNOM_USING_CMSIS_NN       // uncomment if use CMSIS-NN for optimation 
~~~

### 内存接口
内存接口是必须的。

如果您的平台不支持std接口，请根据您的平台进行修改。关于RT-Thread的移植，请参见[example](#examples)。

### 运行 & 调试 
可选的。 

如果你的平台有控制台或者终端，建议你移植它们。它们将帮助你验证你的模型。

`nnom_us_get()` 用于运行时分析。如果提出，NNoM将能够记录每一层的时
间成本，并计算它们的效率。此方法应以us分辨率返回当前时间(16/32位无符
号值，值可能溢出)。

`nnom_ms_get()` 用于'nnom_utils.c'中的api求值。用该方法对整组测试
数据进行性能评价。

`LOG()` 用于打印模型编译信息和评估信息

### NNoM 配置
`NNOM_BLOCK_NUM` 是内存块的最大数量。内存块的使用情况将在编译时打印出来。
需要时进行调整。 

`DENSE_WEIGHT_OPT`, 重排序密度的权重将获得更好的性能。如果你的模型使用'nnom_utils.py'来部署，权重已经被重新排序。 


### 格式配置

神经网络的数据格式总共具有四个参数：`N`表示批量大小、`C`表示特征图通道数、`H`表示特征图的高、`W`表示特征图的宽。

机器学习中通常采用两种数据格式：`HWC` and `CHW`. 
HWC最后取通道数据, CHW先取通道数据. 

默认后端将运行 `HWC格式` ，这是针对CPU情况进行优化的。图像通常使用HWC格式存储在内存中。



**1. HWC 格式**

`NNOM_USING_CMSIS_NN` : 取消注释，它将启用CMSIS-NN后台加速。

在ARM-Cortex-M4/7/33/35P芯片上，当你启用它时，性能可以提高约5倍。
详情请查阅[paper](https://arxiv.org/pdf/1801.06601.pdf)

要将后台从本地后台切换到优化的CMSIS-NN/DSP，只需取消 `#define NNOM_USING_CMSIS_NN` 这一行注释。

然后，在你的项目中，你必须:

1. 在你的项目中包括CMSIS-NN和CMSIS-DSP。
2. 确保在CMSIS-NN/DSP上启用了优化。

**注意**

要求CMSIS版本在5.5.1+以上(NN版本> 1.1.0,DSP版本1.6.0)。

确保你的编译器正在使用新版本的“arm_math.h”。
在一个项目中可能会有一些重复，例如STM32 HAL有自己的“arm_math.h”版本

如果不能编译，也可以在预编译配置中定义芯片核心并启用FPU支持。
> *e.g. 当使用STM32L476时，你可以在你的项目中添加两个宏 ' ARM_MATH_CM4,  __FPU_PRESENT=1'*

毕竟，您可以尝试使用 'nnom_utils.c' 中的api来评估性能。

**2. CHW 格式t**

`NNOM_USING_CHW` : 取消注释它会将整个后端格式更改为 `CHW` 。

这种格式只在CPU模式下运行非常低效的卷积。但是，它与大多数硬件加速兼容，例如[K210](https://kendryte.com/)中的KPU. 

纯C实现完成。使用KPU的硬件加速还在开发中，很快就会推出。 

**Notes**

当启用CHW模型时，CMSIS-NN将被自动排除。 


## Examples

### Porting for RT-Thread

~~~C
// memory interfaces
#define nnom_malloc(n)      rt_malloc(n) 
#define nnom_free(p)        rt_free(p)
#define nnom_memset(p,v,s)  rt_memset(p,v,s)

// runtime & debuges
#define nnom_us_get()       0
#define nnom_ms_get()       rt_tick_get()	// when tick is set to 1000
#define NNOM_LOG(...)       rt_kprintf(__VA_ARGS__)

// NNoM configuration
#define NNOM_BLOCK_NUM      (8)		// maximum number of memory block  
#define DENSE_WEIGHT_OPT    (1)		// if used fully connected layer optimized weights. 

// Formate configuration
//#define NNOM_USING_CHW  			// using CHW format instead of default HWC
//#define NNOM_USING_CMSIS_NN       // uncomment if use CMSIS-NN for optimation 
~~~
































