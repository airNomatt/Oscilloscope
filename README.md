# Oscilloscope
This project built a simple oscilloscope by Labview,based on NI DAQ.
# NI数据采集卡上位机设计（Labview）

>本文设计的上位机主要有以下特点：
>
>1. 使用NI DAQmx 函数套件读取NI公司数据采集卡内容；
>2. 利用XY图绘图，实现了波形大小可调且X轴标签合理对应（**波形图表数据缓存区无法在程序面板进行调节，运行过程中无法调节波形大小；波形图可通过传入数据大小调节波形大小，但是X轴标签不可调**）。

==注：本文档主要陈述程序的设计思路，实现过程可以直接看代码。==

## 一、软件UI界面介绍

![OSC-UI界面](https://github.com/airNomatt/Oscilloscope/blob/main/images/OSC-UI%E7%95%8C%E9%9D%A2.png)

Signal实时显示信号波形，Frequency显示信号1s数据的频谱。

六档旋钮控制显示波形的大小，Channel选择物理通道，START、STOP控制信号采集的开始和结束。



## 二、线程设计

为了程序具有更好的执行效能，将UI界面控制程序作为主线程，接受用户的命令并控制数据处理相关操作；而将数据处理和显示部分作为子线程，由主线程调度。

线程的功能分配如下：

![OSC线程设计](https://images.cnblogs.com/cnblogs_com/blogs/734322/galleries/2208430/o_220825064411_OSC%E7%BA%BF%E7%A8%8B%E8%AE%BE%E8%AE%A1.png)

Labview后面板中，位于同一层的程序图在执行时具有相同的优先级，即它们是并行执行的，所以Labview实现多线程无需特殊函数，只要将两个程序图放在同一层，他们就能各自独立执行。



## 三、NI DAQmx套件的使用

DAQmx函数套件是Labview中，专门用于与NI公司的数据采集硬件设备进行交互的函数套件。可以在安装Labview时勾选相关组件，或者安装后通过VI Package Manager进行添加。

![DAQmx](https://github.com/airNomatt/Oscilloscope/blob/main/images/DAQmx.png)

使用该套件进行数据读取的方法如下：

依次选择函数：创建通道(Create Channel)、采样时钟(Timing）、开始任务(Start)、读取数据(Read)、停止任务(Stop)、清楚任务(Clear)。

![DAQ连接](https://github.com/airNomatt/Oscilloscope/blob/main/images/DAQ%E8%BF%9E%E6%8E%A5.png)

连接方式如上，数据一般要进行多次读取，所以Read函数一般放在循环结构中。**各个函数其他必要参数可查看帮助文档**，下面对上述连接方式做几点简要说明：

1. 创建通道函数实现了将硬件采集设备的通道映射到上位机，可以理解为这个创建通道函数就是我们的硬件，它可以源源不断地输出数据，Timing函数则用于选取这些数据，也就是采样，Timing使用的采样时钟一般是上位机的时钟。
2. Timing函数有连续采样(Continuous Samples)和有限采样(Finite Samples)，连续采样即持续进行采样，有限采样则根据参数“每通道采样数(Samples per channel)”来采集固定数量的点，该参数在连续采样模式下用于配置缓存区大小。
3. 读取函数有单点读取(Single Sample)和多点读取(Multiple Samples)两种，单点读取每次从缓冲区读取一个点，**输出为数据点（double或者波形数据点类型）**；多点读取每次从缓冲区读取一组数据点，读取量可配置，**输出为数组**。



## 四、波形显示

为了实现波形大小可调，本程序使用如下方法构建XY图所需数据

![XY图数据处理](https://github.com/airNomatt/Oscilloscope/blob/main/images/XY%E5%9B%BE%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86.png)

## 五、程序的拓展项

本程序只是一个简单的示意，仍存在许多可以补充的地方，欢迎大家二次开发，方向不限于：

1. 本程序将Read函数读取的数据存入到一个缓存数组，但为对该数组的大小进行限制，是因为考虑到没有长时程显示数据的需求，所以没有对其进行处理。如果数据量比较大，可以考虑构建循环数组；
2. 本程序设置的波形大小调节只有6档，有需要的可以添加更大的调节范围；
3. 本程序主要演示数据波形，所以计算信号的FFT时，简单地采用了1s内采集的数据进行计算，如果希望获得更好的频谱展现效果，可以增大FFT计算使用的数据量；
4. 如需要数字滤波，可以将队列中的数据先输入Labview中的数字滤波器函数，然后将结果显示出来；
5. 可以添加波形数据保存功能；
6. 对于心电数据，可以添加一个心率计算功能（寻找频谱中幅度最大的点，取它的频率值乘以60即心率，如果需要更精准的预测，可以把一段时间内计算的心率存起来，然后使用数学工具中的线性拟合降低误差）。
