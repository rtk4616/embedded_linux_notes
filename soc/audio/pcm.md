# PCM族

## PCM：

>PCM是一种`四线式`的音讯接口，采用特殊的调变、解调变方式实现多声道功能，将讯号的强度依照同样的间距分成数段，然后再采用特殊的数字符号来量化(Quantization)，PCM通常被应用于数字电信系统上，PCM主要可分三个过程，取样、量化及解碼，早期PCM技术并不流行于DVD或DVR等消费性电子产品上，主要原因是因为PCM需要使用大量的位率，相较之下在DVD设备上使用压缩过的音讯会更符合效率，但是新世代的蓝光光盘技术崛起，使得PCM音讯处理得以大量应用，随着蓝光光盘的普遍，PCM的应用会渐渐增加。


## 硬件接口

四线的信号接口

### SCLK

同步频率(`脉冲`)

### FS

帧同步频率(包含了FSO/FSI) 

### DR

输入数据

### DX

输出数据 


## 音频格式


### DSP mode A

### DSP mode B

### TDM 1 A

### TDM 2 B


## SCLK--脉冲占空比调试

设置:

T_SYNCL : 设置SCLK时钟保持高电平的时间,n将保持n个bit的时间.
T_ISYNC	: 反转同步,比如设置的占空比为30%, 使能该位后,占空比发生反转变为70% 
