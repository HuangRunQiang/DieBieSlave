DieBieSlave - Hardware
===
# Introduction
在特文特大学生物力学工程学院的工作期间，我对EtherCAT及其应用产生了兴趣。我了解到，该学院的许多项目（以及其他机器人学领域的学院）都希望在独特的传感器和EtherCAT之间实现自定义的耦合，而现有的从站并不总是容易实现这种耦合。在我的工作中，我帮助设计了一款基于ET1100的EtherCAT从站，该ASIC采用BGA封装，需要两个外部以太网PHY和大量被动元件。ET1100完全能够将所有所需的应用与EtherCAT进行耦合，但在某些应用中，实施起来过于繁琐，因为知道LAN9252几乎可以完成相同的任务，并且将ASIC和2个PHY集成在一个QFN封装中。

得知LAN9252的存在，并听到需要一个“传感器 <--> EtherCAT”接口的需求，激发了我设计这个项目的动力; DieBieSlave，一个通用的EtherCAT从站，能够将“任何”基于I2C、SPI、UART、模拟、数字或CAN的传感器与EtherCAT相连。这个想法是，DieBieSlave对于任何传感器应用都是通用的，只需在一个PCB上具备所有“复杂”的EtherCAT、电源和微控制器硬件，而每个传感器实现唯一不同的是一个廉价且简单的子板。当然，每个传感器都需要其独特的代码来初始化和采样传感器。

The DieBieSlave looks like this:
![alt text](Binaries/Images/DieBieSlaveV0_2TOP0.png "DieBieSlave V0.2 TopView")
![alt text](Binaries/Images/DieBieSlaveV0_2BOT0.png "DieBieSlave V0.2 TopView")
![alt text](Binaries/Images/DieBieSlaveV0_2TOP1.png "DieBieSlave V0.2 TopView")
![alt text](Binaries/Images/DieBieSlaveV0_2BOT1.png "DieBieSlave V0.2 TopView")

顶部功能：
* 直流（10-30V）电源输入插孔。
* EtherCAT 输入/输出连接器。
* EtherCAT、电源和软件状态 LED 指示灯。
* BOOT0 硬件串行引导启用按钮。

底部功能：
* 微型 USB 连接器，连接到 STM32F303RET6 的硬件串行 UART1，用于一般串行通信和硬件引导程序。
* 微型 USB 连接器，连接到 STM32F303RET6 的硬件 USB 外围设备。
* 7Pin picoblade 调试连接器，连接到 STM32F303RET6 的 SWD 和 UART2 外围设备。

两侧功能：
DieBieSlave 接口两侧的孔连接到 STM32F303RET6 上所有暴露的（I2C、SPI、UART、模拟、数字和 CAN）外围设备，并应连接到所需的目标传感器。标准的 2.54 毫米排针可以焊接到 DieBieSlave 的两侧，从而可以使用标准的 2.54 毫米公头/母头连接器将标准原型板连接到顶部的小区域或底部的整个区域（实现紧凑或全面的接口）。连接到与传感器接口的一侧的板称为子板。

更多细节可以在原理图中找到。 [here](/Project%20Outputs%20for%20DB10012_UniversalSlave/DB10012_DieBieSlave.PDF).

### 最新硬件发布（生产文件）

* V0.1 初始硬件
   该版本存在以下错误：
   * LAN9252 的 INT 引脚被错误地连接到了 PC5（当 I2C INT 输入也被使用时无法切换到 INT 输入），但应该连接到 PB0（可以始终使用 INT 功能）。这个问题在 V0.2 中已修复。

* V0.2 V0.1的修复版本，并增加功能（当前版本）
   该版本增加了一些功能：
   * 直流-直流电源输入 10-30V 也路由到了引脚头，现在如果需要，DieBieSlave 可以由子板供电。

最新版本的生产数据可以在[这里](Project%20Outputs%20for%20DB10012_UniversalSlave)找到。PDF 格式的原理图可以在[这里](DB10012_UniversalSlave.PDF)找到。

### 特点
DieBieSlave 的设计使其易于制造，并具有相对较小的占地面积（由专业人士设计）。是否实现一个子板完全取决于用户。DieBieSlave PCB 具有以下功能：

* PCB 两侧均设有电源指示灯。
* PCB 两侧均设有 EtherCAT 运行指示灯。
* 软件状态指示灯连接到引脚 PB15，位于 PCB 两侧。
* 直流输入范围为 10-30V（可能更广，但未经测试），可简单应用于12V和24V系统。
* 板载 +5V 和 +3V3 开关电源，可供子板头使用。
* 配备强大的微控制器，带有 DSP 和 FPU，STM32F303RET6（cortex-M4，512K字节闪存和64K字节静态随机存取存储器）。
* RJ45 EtherCAT 输入和输出连接器。
* 基于 QFN 封装的 LAN9252 EtherCAT ASIC。

### 电气规格
* 直流输入电源范围为 10-30V，具有反极性保护。
* +5V 电源可提供高达 400mA 输出电流。
* +3V3 电源可提供高达 200mA 输出电流。

# 实现
正如介绍中所述，DieBieSlave 是一个通用的 EtherCAT 从站实现，始终需要一个子板向 DieBieSlave 提供应该放在总线上的信息。以下是一个 DieBieSlave 和子板的示例实现：
![alt text](Binaries/Images/DieBieSlave_V0_2_07.jpg "分离的子板和DieBieSlave")
![alt text](Binaries/Images/DieBieSlave_V0_2_08.jpg "连接的子板和DieBieSlave")
如图所示，子板可以是标准的穿孔原型板，也可以是携带传感器的定制设计板。
#### LAN9252 vs ET1100
尽管 LAN9252 似乎是 ET1100 的完美替代品，具有两个内置的 PHY、QFN 封装和少量外部元件，但在软件方面存在一个重大缺陷。虽然 ET1100 通过 SPI 具有对其内部 RAM 的透明接口，而 LAN9252 没有，需要某种形式的地址转换，这会阻止大块（PDO）数据被一次性写入，这阻止了在微控制器和 LAN9252 之间进行 DMA 同步的使用。唯一的替代方案是相对耗费软件资源（地址转换需要在软件中完成），与其 DMA 替代方案相比，这需要更多的软件工作量。对于传感器采集来说这并不是致命问题（传感器需要非常少的处理器性能），但在像电机控制器这样的实时系统中将是一个缺点。

#### 使用的技术
使用的集成电路及其相应功能：
* STM32F303RET6 -> 主微控制器。
* LAN9252/ML -> 带有内置 PHY 的EtherCAT 从站控制器。
* M24C16-WMN6P -> 用于 LAN9252 的 EEPROM。
* LM25011MY/NOPB -> 直流电源输入到 +5V 转换器。
* LMR10510XMFE/NOPB -> +5V 到 +3V3 转换器。
* 74980111211 -> 带有集成磁性元件的 RJ45 连接器。
* CP2104-F03-GM -> 用于引导加载程序和一般串行通信的 USB 串行转换器。

![alt text](Binaries/Images/DieBieSlave_V0_2_06.jpg "DieBieSlave V0.2 双层 PCB 图片")
![alt text](Binaries/Images/DieBieSlave_V0_2_02.jpg "DieBieSlave V0.2 底部概览")
![alt text](Binaries/Images/DieBieSlave_V0_2_03.jpg "DieBieSlave V0.2 顶部概览")
![alt text](Binaries/Images/DieBieSlave_V0_2_04.jpg "DieBieSlave V0.2 底部元件概览")
![alt text](Binaries/Images/DieBieSlave_V0_2_05.jpg "DieBieSlave V0.2 底部元件概览")

# 示例用法
#### 将 MPU9250/NunChuck 连接到 EtherCAT
将 Nun-chuck 控制器和 MPU9250 传感器与 EtherCAT 进行接口连接：

[![VIDEO01](http://img.youtube.com/vi/i7gFqLQb0EA/0.jpg)](http://www.youtube.com/watch?v=i7gFqLQb0EA)

#### 更多图片
![alt text](Binaries/Images/DieBieSlave_V0_2_09.jpg "NunChuck 从站和双 MPU9250 从站")
![alt text](Binaries/Images/DieBieSlave_V0_2_10.jpg "NunChuck 从站和双 MPU9250 从站")
![alt text](Binaries/Images/DieBieSlave_V0_2_11.jpg "一个 Shield 示例的视图")
![alt text](Binaries/Images/DieBieSlave_V0_2TwinCAT_01.png "TwinCAT 中连接的两个从站示例的截图")
![alt text](Binaries/Images/DieBieSlave_V0_2SlaveEditor_01.png "SOES 配置的从站编辑器视图")
