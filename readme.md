# B-HM-5E00-24V

基于先楫 HPM5E31（HPM5E31IPB1，eLQFP100）的 24V 电机驱动方案开发板：EtherCAT 从站双口 + FOC/BLDC 驱动 + 三类编码器接口 + OLED 显示。交付完整设计方案（原理图 + 驱动源码 + 应用例程），软件基于官方 hpm_sdk 例程，仅做板级宏适配（pinmux + board.h）。

> 状态：定版·已验证（对应原理图 HPM5E_FOC_ETHERCAT v1.2，EDA Bridge 实时网表核验）｜文档更新：2026-11-11

- 产品线：方案开发板（B）
- 产品编码：B-HM-5E00-24V（B=方案开发板线，HM=HPM 系列，5E00=芯片型号，24V=供电电压规格码）
- 适用客户：需要快速出电机产品的公司研发团队，以及电机驱动方向的个人开发者

## 功能特性

| # | 功能 | 说明 |
| --- | --- | --- |
| 1 | EtherCAT 从站双口 | P0(IN)+P1(OUT)，内置 ESC0，PHY=JL1111-NCG21 ×2，flash 模拟 EEPROM，PHY 复位/Link 指示齐全 |
| 2 | BLDC/FOC 电机驱动 | PWM0 六路官方互补对（PD0~PD5），FD6288Q 预驱 + NCE7560K 三相桥，硬件死区+互锁 |
| 3 | 三相电流采样 | IA/IB/IC（PF27/PF24/PF19），RS624XQ 差分调理、VREF=1.65V 偏置，ADC0/1 双通可同步采样 |
| 4 | 母线电流采样 | IBUS=PF22，母线分流电阻（1mΩ）差分检测，软件过流保护 |
| 5 | 母线电压与温度 | VBUS=PF21（24V 标定分压）、NTC=PF23，软件过欠压/过温保护 |
| 6 | 编码器接口 ×3 | QEI1 霍尔/ABZ（CN5 口，PB29/30/31）、SPI1 绝对值编码器（CN47，PC10~13）、多摩川 485（UART0+PA02 硬件 DE，SN65HVD75D） |
| 7 | PWM 占空比测量 | CN5 第 5 脚输入 → PC16（1K 串阻+下拉+滤波），TRGM 路由 GPTMR 输入捕获，测外部 PWM 频率/占空比 |
| 8 | CAN 总线 | MCAN3（PC14/15）经收发器至 CN10 端子 |
| 9 | 应用串口 | UART2（PC08/PC09），默认 115200，兼调试打印；H1 排针 + JTAG 座双引出 |
| 10 | OLED 显示 | 0.42 寸 I2C OLED（I2C3，PD6/PD7），状态参数可视化 |
| 11 | USB device | DP/DN 纯 device，ROM 下载 + CDC 虚拟串口 |
| 12 | JTAG 调试烧录 | H2 12 针座（TDO/TDI/TCK/TMS/RSTN），flash 烧录主通道 |
| 13 | EtherCAT IO 测试 | 2 输入（KEY1=PF18、KEY2=PF26，按下为高）+ 1 输出（SYS_LED=PA22），LED1/LED2（PF16/PF20）扩展指示 |

已明确不做：相电压采样（VA/VB/VC/COM）、QEO 编码器仿真、PWM 硬件故障输入、USB 主机（ID/OC/PWR）、BOOT 拨码开关、外挂 EEPROM。

## 硬件接口（对外接口与接线）

### 供电要求

| 项目 | 规格 |
| --- | --- |
| 供电电压 | 24V DC（宽压 12~36V 设计） |
| 电流 | 按电机功率 TBD |
| 接插件 | CN1 电源端子（DCBUS_IN/PGND_IN，防反接保护） |

板上电源树：24V → 12V → 5V → 3.3V/1.2V；内核 1.2V 由 TPS563201 旁路模式外供。

### 对外接口总览

| 接口 | 类型 | 端子/信号 | 电气规格 | 说明 |
| --- | --- | --- | --- | --- |
| 电源输入 | CN1 端子 | DCBUS_IN / PGND_IN | 24V | 防反接保护 |
| EtherCAT IN (P0) | RJ45 | PHY0 JL1111 ← ESC0 Port0 | 100BASE-TX | 从站输入口，PHY 地址 01 |
| EtherCAT OUT (P1) | RJ45 | PHY1 JL1111 ← ESC0 Port1 | 100BASE-TX | 级联输出口，PHY 地址 15 |
| 电机相线 | CN41 端子 | U/V/W（MA/MB/MC） | 按 24V 电机功率 TBD | FD6288Q + NCE7560K 三相桥，硬件死区 |
| 霍尔/ABZ 编码器 | CN5 8pin | 1=GND，2~4=MA/MB/MC（QEI1），5=PWM 测量输入，6=5V，7/8=GND | 5V 上拉、1K 串阻+1nF 滤波+钳位 | 霍尔/增量编码器 + PWM 占空比测量共用 |
| SPI 编码器 | CN47 8pin | VCC + SPI1 四线（PC10~13）+ GND×3 | 3.3V | 绝对值编码器 |
| 多摩川 485 | RS485 口 | UART0：TX=PA00、RX=PA01、DE=PA02 经 SN65HVD75D | 2.5Mbps 半双工 | 硬件 DE 自动方向；带自恢复保险丝+TVS |
| CAN | CN10 4pin | CAN_H / CAN_L / GND×2 | 经 TJA1051TK/3 | 经典 CAN |
| OLED 显示 | U11 4pin | VCC / GND / SCL(PD7) / SDA(PD6) | I2C3，3.3V | 0.42 寸白色 OLED |
| 应用串口 | H1 排针 + H2 座 | UART2：TX=PC08、RX=PC09 | 3.3V TTL | H1 第 9/7 脚，同时引到 H2（JTAG 座）第 6/8 脚 |
| USB | USB 座 | DP=68 / DN=67 | USB 2.0 HS device | ROM 下载 + 虚拟串口 |
| JTAG | H2 12pin | TDO/TDI/TCK/TMS/RSTN + UART2 复用 | 3.3V | 调试 + flash 烧录主通道；TRST（PA08）经 R18 10K 上拉，未引出 |
| ECAT IO 输入 | 板载按键 | KEY1=PF18、KEY2=PF26 | 3.3V，10K 下拉、按下为高 | ecat_io 例程 TxPDO 输入 |
| ECAT IO 输出/LED | 板载 | SYS_LED=PA22；LED1=PF16、LED2=PF20 | 3.3V | ecat_io RxPDO 输出兼状态指示 |
| 启动配置 | 板上电阻 | PA2/PA3 各 10K 下拉 | 复位采样 [PA03:PA02]=00 | 固定 NOR flash 启动；串口 ISP 经 485 口（UART0）或 USB0 |
| PWM 测量输入 | CN5 第 5 脚 | → R103 1K → PC16 | 3.3V 域，R105 1K 下拉 + C85 100nF | TRGM 路由 GPTMR 输入捕获；C85 截止约 3kHz，高频 PWM 建议改 1nF |

> 电流/电压/温度采样点（IA=PF27、IB=PF24、IC=PF19、IBUS=PF22、VBUS=PF21、NTC=PF23）均在板内，板上留有对应单针测试点（CN20 IA / CN21 IB / CN14 IBUS / CN40 VBUS / CN13 VREF / CN16 偏置中点等）。

### 接线步骤

1. 接 24V 电源到 CN1（注意 DCBUS_IN/PGND_IN 极性，防反接）
2. 接电机三相线 U/V/W 到 CN41
3. 按编码器类型接对应座：霍尔/ABZ → CN5；SPI 绝对值 → CN47；多摩川 485 → RS485 口
4. OLED 接 U11 座
5. 调试/烧录：JTAG 接 H2 座（主通道），或 USB 连 PC
6. EtherCAT 级联：主站网线接 P0（IN），下一级从站接 P1（OUT）

### 指示与保护

- 状态 LED：SYS_LED（PA22，EtherCAT IO 输出兼用）；LED1/LED2（PF16/PF20）扩展指示
- EtherCAT Link 灯：RJ45 内置灯由 PHY JL1111 LED1 脚驱动，MCU 经 PB27/PB28 读取链路状态
- 电源指示灯：PWR_LED（板载）
- 输入保护：防反接 MOS + TVS + 自恢复保险丝（485 口）
- 软件保护：IBUS（PF22）过流检测、VBUS（PF21）过欠压检测、NTC（PF23）过温保护

## 软件接口与协议

| 协议/接口 | 说明 |
| --- | --- |
| EtherCAT 从站 | 官方 SSC 移植（hpm_sdk samples/ethercat），flash 模拟 EEPROM，ecat_io / ecat_cia402 例程 |
| 电机控制 | hpm_sdk bldc/foc 例程，按板级差异改宏（pinmux + board.h）后直跑 |
| 应用串口 | UART2 默认 115200，协议命令表 TBD |
| 多摩川编码器 | 官方 uart_tamagawa 例程，UART0 + TRGM0 触发（TRIG0 组），2.5Mbps，PA02 硬件 DE |
| PWM 占空比测量 | GPTMR 输入捕获（input_capture 例程）+ TRGM 路由 PC16，测频率/占空比 |
| OLED | I2C3 主控（SDK i2c 例程），地址/驱动按所选屏 TBD |
| CAN | MCAN3 + TJA1051TK/3，应用协议 TBD |
| USB | ROM 下载 + CDC 虚拟串口（device 模式） |

通信协议与命令：TBD（UART 命令表、CAN 报文定义、EtherCAT 对象字典索引，定型后填）。

## 快速上手

1. 上电：检查 PWR_LED；启动配置已固定 NOR flash，无需拨码
2. 运行验证：JTAG（H2 座）烧录 ecat_io 例程 → 主站扫描从站进入 OP → 按 KEY1/KEY2 看 TxPDO 位翻转、主站写 RxPDO 看 SYS_LED 亮灭
3. 电机调试：烧录 bldc/foc 例程，OLED 可显示运行状态；外部 PWM 接 CN5 第 5 脚可测占空比
4. EtherCAT：主站（TwinCAT 等）加载 ESI 描述；IO 测试用 ecat_io 例程

## 相关文档

设计文档（设计细节与原理图说明）位于原金山云文档同目录；本仓库 README 由「功能描述」与「软硬件接口说明」合并整理。
