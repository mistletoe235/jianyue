> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Mark_md/article/details/119774663?spm=1001.2014.3001.5501)

前几篇介绍了`ODrive`在`Windows`下的使用环境搭建，以及`TLE5012B`、`AS5047P`的 ABI 配置。

ODrive 教程资源导航
=============

[ODrive 踩坑（一）windows 下使用环境的搭建，odrivetool 及 USB 驱动的安装](https://blog.csdn.net/Mark_md/article/details/117885698?spm=1001.2014.3001.5501)

[ODrive 踩坑（二）3508 电机和 TLE5012B 磁编码器参数配置、校准、位置闭环模式转动电机（TLE5012B - ABI）](https://blog.csdn.net/Mark_md/article/details/117898801?spm=1001.2014.3001.5501)

[ODrive 踩坑（三）AS5047P 磁编码器的 ABI 接口](https://blog.csdn.net/Mark_md/article/details/119524819?spm=1001.2014.3001.5501)

[ODrive 踩坑（四）AS5047P-SPI 绝对值磁编码器，不需每次上电校准无刷电机，直接上电可用](https://blog.csdn.net/Mark_md/article/details/119774663?spm=1001.2014.3001.5501)

[ODrive 踩坑（五）驱动云台电机、低齿槽转矩电机实现高精度定位](https://blog.csdn.net/Mark_md/article/details/119860059?spm=1001.2014.3001.5501)

苦于使用 `ABI编码器`，每次上电都要`编码器校准`，电机左转一圈再右转一圈。浪费时间不说，运动过程还可能导致工件误触，导致上电意外。如果想要设备上电不经过`编码器校准`，通电后直接就能用，可能要用到 `SPI绝对值编码器`。

`ODrive` 支持两种类型的 `SPI绝对值编码器`：

*   **CUI 协议**：兼容 AMT23xx 系列 (AMT232A, AMT232B, AMT233A, AMT233B)。
*   **AMS 协议**：兼容 AS5047P 和 AS5048A。

（**注意**：`ODrive` 并不支持 `TLE5012B` 的 SPI 接口，仅能使用它的 ABI）

1、ODrive 连接 AS5047P，电机安装
========================

  为了方便测试 `AS5047P-SPI绝对值编码器`，也便于扩展不同的电机，就有了下面这块万能转接板，支持 2208、2212、3508、5008、6010、6374、42 步进、57 步进 等不同电机的定位安装。

  **图中 AS5047P 转接板购买链接，我的淘宝小店**：[AS5047P SPI 磁编码器 3206 云台无刷电机 带径向磁铁 Odrive 电机](https://m.tb.cn/h.f84uwDQ?sm=0bbfa8)

  店铺详情内有安装孔位、原理图、教程、资料，手机端可能因没做适配看不到，建议用电脑打开。  
![](https://img-blog.csdnimg.cn/6dd784e6f30a46a898d4013596501182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/0291212510ef435b88b7316ae98f6704.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATWFya19tZA==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

![](https://img-blog.csdnimg.cn/10e0eeea9bfe4da1968bbeff3d42860e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATWFya19tZA==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

  将电机与磁编码器正确安装，`强磁`与`AS5047P`芯片的间距不建议超过 3mm。手册中建议值为 0.5~3mm。

电机的型号与第（二）章的不同，但极对数还是 7 对，其他参数也基本一致。  
测试使用与 DJI3508 相同的 PID 参数，控制效果竟比原来要好不少。  
![](https://img-blog.csdnimg.cn/1dd09980002a4f33bb374fe13639e800.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)  
连接 SPI 接口，SCK、MISO、MOSI 一一对应，CS 插到 `ODrive` 的`4脚`。  
![](https://img-blog.csdnimg.cn/947e375c0c6f44b58c7b11487d24ac01.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/77ad3df35d5c4046a26dbf88c83f6005.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

2、ODrive 连接电脑，进行电机校准前的配置
========================

将系统上电，ODrive 连接电脑。

2.1、恢复出厂配置
----------

```
# 恢复出厂配置
odrv0.erase_configuration()
```

2.2、配置主板参数
----------

*   限制参数主要用于保护主板、电机不会受损，包括最大电流、保护电压等参数。
*   根据自己适配的电机，酌情调整大小。

```
# 配置AUX接口上的制动电阻值（常见的为0.47、2.0Ω），如没接则配置为0
odrv0.config.brake_resistance = 0

# 配置低压保护阈值（V）
odrv0.config.dc_bus_undervoltage_trip_level = 8.0

# 配置高压保护阈值（V）
odrv0.config.dc_bus_overvoltage_trip_level = 56.0

# 配置过流保护阈值（A）
odrv0.config.dc_max_positive_current = 50.0

# 配置反向电流阈值（电机制动产生的反向电流）（A）
odrv0.config.dc_max_negative_current = -5.0

# 配置回充电流值（根据供电电池的参数配置，开关电源供电配置为0）（A）
odrv0.config.max_regen_current = 0

# 保存参数
odrv0.save_configuration()
```

![](https://img-blog.csdnimg.cn/84dccaadb30a4b71855be0e440a4b1a5.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

2.3、配置电机参数
----------

```
# 配置电机0极对数，根据博客开篇的介绍，自己去数磁极个数，然后/2
odrv0.axis0.motor.config.pole_pairs = 7

# 配置电机0的限制电流（A）
odrv0.axis0.motor.config.current_lim = 35

# 配置电机0的电流采样阈值（A）
odrv0.axis0.motor.config.requested_current_range = 60

# 配置电机0校准时的电流阈值（根据自己电机的负载状况酌情配置）（A）
odrv0.axis0.motor.config.calibration_current = 10

# 配置电机0类型。
# 目前支持两种电机：大电流电机（MOTOR_TYPE_HIGH_CURRENT）和云台电机（MOTOR_TYPE_GIMBAL）
odrv0.axis0.motor.config.motor_type = MOTOR_TYPE_HIGH_CURRENT

# 保存参数
odrv0.save_configuration()
```

![](https://img-blog.csdnimg.cn/a69e74cf7526416a848777ad9ce124e7.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

2.4、配置编码器参数
-----------

```
# 配置电机0编码器类型。ENCODER_MODE_INCREMENTAL 使用的是ABI正交（增量）编码器。
# 值 ENCODER_MODE_SPI_ABS_AMS 使用AMS磁编码器-AS5047/AS5048。
odrv0.axis0.encoder.config.mode = ENCODER_MODE_SPI_ABS_AMS

# 设置CSn片选的引脚，ODrive J3接口GPIO3-8任选一个作为CS，这里我使用GPIO4
odrv0.axis0.encoder.config.abs_spi_cs_gpio_pin = 4

# 配置电机0编码器CPR（每转一圈，编码器的计数），AS5047P的最大分辨率为14位
odrv0.axis0.encoder.config.cpr = 2**14

# 编码器带宽设置，CPR值越高带宽设置的也越高
odrv0.axis0.encoder.config.bandwidth = 3000

# 编码器精度，类型为 [float]，单位为 [圆周角度∠] （这个值可以适当的大一些，避免环境干扰）
# 电机实际转动角度和开环移动距离之间允许的最大误差，超过此误差将报错ERROR_CPR_OUT_OF_RANGE。
odrv0.axis0.encoder.config.calib_range = 10

# 保存参数
odrv0.save_configuration()
```

如果因编码器精度误差，而导致失步，会使 `ODrive` 报错不运行。

建议将`odrv0.axis0.encoder.config.calib_range`调大，尤其是磁编码器，避免因环境干扰出现误差。（AS5047P 的手册对器件的精度描述为 ±0.1°，故`odrv0.axis0.encoder.config.calib_range`的值最小不能小于 0.1）  
![](https://img-blog.csdnimg.cn/95c381a51e334373a1e24a5fab63f295.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

2.5、配置控制器参数（位置闭环模式、配置 PID 参数）
-----------------------------

```
# 配置电机0控制模式，为位置闭环控制
odrv0.axis0.controller.config.control_mode = CONTROL_MODE_POSITION_CONTROL

# 配置电机0最大转速（转/秒）（电机kv值 * 电压 / 60）
odrv0.axis0.controller.config.vel_limit = 120

# 配置位置环增益：20
odrv0.axis0.controller.config.pos_gain = 20

# 配置速度环增益：0.05
odrv0.axis0.controller.config.vel_gain = 0.05

# 配置积分增益：0.02
odrv0.axis0.controller.config.vel_integrator_gain = 0.02

# 配置输入模式为：梯形轨迹模式
odrv0.axis0.controller.config.input_mode = INPUT_MODE_TRAP_TRAJ

# 配置梯形模式下的电机转速阈值（转/秒）
odrv0.axis0.trap_traj.config.vel_limit = 50

# 配置梯形运动模式下的加速加速度
# 数值大小影响动作跟随效果，大则跟随快；小则跟随慢。
odrv0.axis0.trap_traj.config.accel_limit = 10

# 配置梯形运动模式下的减速加速度
# 数值大小影响动作跟随效果，大则跟随快；小则跟随慢。
odrv0.axis0.trap_traj.config.decel_limit = 10

# 保存参数
odrv0.save_configuration()

# 重启驱动器
odrv0.reboot()
```

上述配置完成后，会重启控制器，留意最后一条指令。

![](https://img-blog.csdnimg.cn/ae1b32d742894170978ded748bae6bc1.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)  
  

三、电机和编码器校准
==========

使用 `AS5047-SPI绝对值磁编码器` 最大的好处，就是可以不用上电后运动校准。只要校准过一次，以后便可直接上电使用，缩短了上电到正常工作的时间，也避免了校准过程对驱动部件的影响。

**注意**：以下操作请务必按照规定的顺序执行，不然很容易造成校准后电机不能进入闭环，或者电机在重新启动后不能自动进入闭环。`AS5047P-SPI磁编码器`最难配置，也最容易出错的地方就在校准的步骤，其中试验了好几十次，基本是差了几步顺序就会导致重启后不能正常进入闭环。请务必按照以下操作进行！！！

**注意**：如有重启后不能自动进入闭环、或者电机参数校准出错 的现象，请断电重启后重试。我的设备有时必须`断电重启`才可用，reboot 命令的重启不管用。（用`odrv0.axis0.error`查询是否出错）

下面三种配置方法，建议依次顺序试验一次。可以只选择一种进行设置，但按顺序来一般不会有错。  
  

3.1 每次上电后都自动校准编码器 的配置方法
-----------------------

这种方式的运行现象与 ABI 接口的一致，每次上电后都自动左转一圈右转一圈，自动进行`编码器校准`。

```
# 进行电机参数校准（运行后电机会发出哔~的一声）
odrv0.axis0.requested_state = AXIS_STATE_MOTOR_CALIBRATION

# 设置电机预校准。（不用每次上电都哔~的一声）
# 驱动器会将本次校准值保存，避免上电启动后自动校准，以加快启动速度。
odrv0.axis0.motor.config.pre_calibrated = True

# 进行编码器校准（运行后，电机会正转一圈再反转一圈）
odrv0.axis0.requested_state = AXIS_STATE_ENCODER_OFFSET_CALIBRATION

# 查看错误，如果为0，则为无错。否则请断电后重启，重试校准。
odrv0.axis0.error
```

如校准的第一步无反应，建议先断电重启后重试。

如断电重启后校准仍无效，用 `odrv0.axis0.encoder.shadow_count` 来测试 `AS5047P-SPI磁编码器` 是否可以正确读数，如读值始终为 0，则说明 AS5047P 硬件故障或者连线有误，不能进行后续的校准操作。（磁编码器容易配置失败，大多数都是这个问题，读不到值，一直为 0）

**注意**：上面的配置 没有保存参数，有保存参数会让后面的闭环无法运行，不清楚什么原因。下面继续。  
![](https://img-blog.csdnimg.cn/c3dfd5addcd342aeb21d027326baa3a7.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

**之后进入闭环模式，电机会保持位置，用手扭动电机，电机会产生反抗并回到原来位置：**

```
# 配置电机为闭环模式
odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
```

**测试运动。电机会按照之前设置的梯形轨迹运行到指定位置：**

```
# 控制电机运行到10圈的位置
odrv0.axis0.controller.input_pos = 10

# 控制电机运行到0圈的位置
odrv0.axis0.controller.input_pos = 0
```

经过上面的校准后，机器已经能够用`AS5047P磁编码器`进行闭环控制。

但并不会在重启后自动进入闭环，仍需在重启后手动进入闭环，略有不便。下面设置上电自动校准并闭环运行。

**设置为上电自动校准，自动进入闭环。**

```
# 设置ODrive上电启动时，自动校准编码器
odrv0.axis0.config.startup_encoder_offset_calibration = True

# 设置ODrive上电启动时，自动进入闭环模式
odrv0.axis0.config.startup_closed_loop_control = True

# 保存参数
odrv0.save_configuration()

# 重启驱动器
odrv0.reboot()
```

重启后，就会看到电机右转一圈又左转一圈，自动校准磁编码器。

因为上面设置了 `odrv0.axis0.motor.config.pre_calibrated = True`，会省去上电启动后自动校准电机参数的过程（哔~ 的一声），节约了一部分启动时间。  
  

3.2 每次上电后，只自动校准电机，不校准编码器
------------------------

上面的配置是上电后自动校准编码器，不仅导致启动时间变长，也会导致机械部件碰到零件的意外。

这次使用的`AS5047P磁编码器`，支持一圈内的绝对值定位，相较于普通的 ABI 正交编码器，可以配置为上电不校准编码器，大大节约了上电启动时间，也避免了运动过程对其他零件的影响。

下面照做一次。如第一步电机校准无反应，重新上电后重试。

```
# 进行电机参数校准（运行后电机会发出哔~的一声）
odrv0.axis0.requested_state = AXIS_STATE_MOTOR_CALIBRATION

# 设置ODrive上电启动后，自动校准电机
odrv0.axis0.config.startup_motor_calibration = True

# 进行编码器校准（运行后，电机会正转一圈再反转一圈）
odrv0.axis0.requested_state = AXIS_STATE_ENCODER_OFFSET_CALIBRATION

# 设置编码器预校准。（不用每次上电都右转一圈又左转一圈）
# 驱动器会将本次校准值保存，避免上电启动后自动校准，以加快启动速度。
odrv0.axis0.encoder.config.pre_calibrated = True

# 关闭ODrive上电启动时，自动校准编码器
odrv0.axis0.config.startup_encoder_offset_calibration = False

# 配置电机为闭环模式
odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL

# 设置ODrive上电启动时，自动进入闭环模式
odrv0.axis0.config.startup_closed_loop_control = True

# 保存参数
odrv0.save_configuration()

# 重启驱动器
odrv0.reboot()
```

如果因编码器精度误差，而导致失步，会使 `ODrive` 报错不运行。

建议将下值调大，尤其是磁编码器，容易受环境干扰出现误差。

（AS5047P 的手册对器件的精度描述为 ±0.1°，故`odrv0.axis0.encoder.config.calib_range`的值最小不能小于 0.1）

```
# 编码器精度，类型为 [float]，单位为 [圆周角度∠] 
# 电机实际转动角度和开环移动距离之间允许的最大误差，超过此误差将报错ERROR_CPR_OUT_OF_RANGE。
odrv0.axis0.encoder.config.calib_range = 0.1
odrv0.axis0.encoder.config.calib_range = 10

# 保存参数
odrv0.save_configuration()
```

3.3 每次上电后，不需任何校准，直接自动进入闭环
-------------------------

在 3.2 的基础上，继续这样设置：（可能会失败，但用 3.2 保底，可以避免每次上电都校准编码器）

```
# 进行电机参数校准（运行后电机会发出哔~的一声）
odrv0.axis0.requested_state = AXIS_STATE_MOTOR_CALIBRATION

# 设置电机预校准。（不用每次上电都哔~的一声）
# 驱动器会将本次校准值保存，避免上电启动后自动校准，以加快启动速度。
odrv0.axis0.motor.config.pre_calibrated = True

# 关闭ODrive上电启动后，自动校准电机
odrv0.axis0.config.startup_motor_calibration = False

# 进行编码器校准（运行后，电机会正转一圈再反转一圈）
odrv0.axis0.requested_state = AXIS_STATE_ENCODER_OFFSET_CALIBRATION

# 设置编码器预校准。（不用每次上电都右转一圈又左转一圈）
# 驱动器会将本次校准值保存，避免上电启动后自动校准，以加快启动速度。
odrv0.axis0.encoder.config.pre_calibrated = True

# 保存参数
odrv0.save_configuration()

# 重启驱动器
odrv0.reboot()
```

四、错误修复、注意事项
===========

多用`odrv0.axis0.error`去检错，多用`odrv0.axis0.motor`、`odrv0.axis0.encoder`去检查参数。

用`odrv0.axis0.encoder.shadow_count`可以测试 AS5047P-SPI 磁编码器能否正常读数。

`odrv0.vbus_voltage`：检查 ODrive 的供电电压。

如果你的`ODrive`无法正常工作，用如下查看错误列表：  
`dump_errors(odrv0)` 查看错误  
`dump_errors(odrv0, True)` 清除错误（如果报错 ODrive 不会继续执行电机旋转指令）

如需重新对 AS5047P 进行软硬件设计，有以下文章可供参考：  
[AS5047P 磁编码器应用设计大全解：硬件电路设计、SPI 通信时序、逻辑波形分析、注意事项](https://blog.csdn.net/Mark_md/article/details/119645201?spm=1001.2014.3001.5501)

相关传感器：  
[TLE5012B 硬件电路设计、4 线 SPI 通信，驱动完美兼容 4 线 SPI 不用改 MOSI 开漏推挽输出](https://blog.csdn.net/Mark_md/article/details/119806139?spm=1001.2014.3001.5501)