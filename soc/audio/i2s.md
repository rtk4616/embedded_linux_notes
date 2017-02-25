# I2S

## 定义:

>I2S(Inter—IC Sound)总线, 又称 集成电路内置音频总线，是飞利浦公司为数字音频设备之间的音频数据传输而制定的一种总线标准，该总线专责于音频设备之间的数据传输，广泛应用于各种多媒体系统。它采用了沿独立的导线传输时钟与数据信号的设计，通过将数据和时钟信号分离，避免了因时差诱发的失真，为用户节省了购买抵抗音频抖动的专业设备的费用。


## 硬件接口

三个主要的信号线

```
     --- 		Left		----------------------
  >		|					|					 |					LRCLK = 44.1kHZ
		---------------------		Right		 ------------
      --    --    --    --   
  >	    |  |  |  |  |  |  |  |  									BCLK = 2 * 44.1Khz * 24 = 2.1168MHz
	     --    --    --    --

  > ... |  1  |  2  |  3  | ...										SDATA
```
### BCLK

> 时钟是方波的形式

串行时钟SCLK，也叫位时钟（BCLK），即对应数字音频的每一位数据，SCLK都有1个脉冲。`SCLK的频率=2×采样频率×采样位数`。

如采样频率=44.1Khz  采样位数=24bit

SCLK = 2 * 44.1kHz * 24 = 2.1168MHz

### LRCLK

帧时钟LRCK，(也称WS)，用于切换左右声道的数据。LRCK为“1”表示正在传输的是`右声道`的数据，为“0”则表示正在传输的是`左声道`的数据。`LRCK的频率 = 采样频率`。
在目前的测试中主要为SYNC_CLK

LRCLK=44.1kHz

### SDATA

串行数据SDATA，就是用二进制补码表示的音频数据。

>有时为了使系统间能够更好地同步，还需要另外传输一个信号`MCLK`，称为主时钟，也叫系统时钟（Sys Clock），是采样频率的`256`倍或`384`倍。


## 音频格式

### I2S


### LJ (Left Justified)


### RJ (Left Justified)




