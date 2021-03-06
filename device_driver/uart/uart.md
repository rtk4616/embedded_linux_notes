# UART

| 对比的项 | 所属类型 | 数据位个数 | 主要作用和功能 | 备注 |
| :--------:|:------:| :------: | :---------: | :--------: |
Serial	软件概念		串行，概念上属于“时分复用”，数据是随着时间不同，慢慢的传送过去的，且大多数是以一个一个bit位的形式发送的。USART, UART, RS232, USB, SPI, I2C, TTL等等，都属于串行方面的协议或概念。	 

UART	硬件设备，电子电路，物理上的模块	1	最常用的一种串行协议。处理串行接口之间的通信，只不过此串行接口，往往都是RS232接口而已。	由于每传输一个字节，都要通过自己的起始位的下降沿起去同步，因此才叫做异步通信。

RS232	电气接口规范	2	串口通信的协议，定义了，DCE和DTE之间的，电气方面的特性：硬件接口即引脚和其功能，信号时序和含义等	更严格的说法应该把RS232叫做EIA-232；对于远距离通信，5V不可靠，所以才会加大电压采用12V，即+12V表示0，-12V表示1

## 参考

1. [RS232串口协议详解](https://www.crifan.com/files/doc/docbook/rs232_serial_intro/release/webhelp/rs232_vs_uart.html)
2. [从tty到uart层，分析uart数据流程（一）](http://blog.csdn.net/changqing1990/article/details/44171201)
3. [UART时序分析](http://blog.csdn.net/liuhan33025/article/details/51131120)
