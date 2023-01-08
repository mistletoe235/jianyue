> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Mark_md/article/details/117898801?spm=1001.2014.3001.5501)

ODrive 对无刷电机进行闭环控制，需要提前获取电机和编码器的参数。

电机极对数
=====

电机需要配置的参数为 `极对数`、`最大电流`、`校准电流`、`电机类型`。其他参数如`相电阻`、`电感`等可由驱动器自动测量。

极对数 = 极数 / 2，极数 = 电机转子的磁铁个数。  
我选用的`DJI-3512`电机，极对数为 7。

借用张店家的图，其中右侧为转子，上面分布着 14 颗磁铁，则极数为 14，极对数为 7。  
![](https://img-blog.csdnimg.cn/20210614104535669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)  
电机电流不清楚。以前 DJI 还卖散装电机的时候，官网是有电机的参数页的，现在统统下架了。不过看这粗壮的铜线，20~30A 应该还是有的。  
  

编码器分辨率（线束）
==========

编码器我使用的是 `TLE5012B-E1000`，是一款磁编码器，支持`SPI接口`和`ABI接口`两种输出方式。与 ODrive 的连接使用`ABI接口`。

当 `TLE5012B-E1000` 的输出方式为 ABI 时，其默认分辨率为 12 位，即`4096`线 / 圈。  
![](https://img-blog.csdnimg.cn/20210614105217633.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)  
参阅 [ODrive 在线文档](https://docs.odriverobotics.com/)，CPR 值应为 PPR 值的 4 倍。

所以我的 CPR 要设置为 4096x4 = `16384`。  
（CPR：每转一圈，ODrive 的编码器计数）  
（PPR：每转一圈，编码器输出的脉冲数（线数））

为什么是 4 倍？因为 ODrive 的主控 MCU 是 stm32，编码器计数使用 Timer 的编码器模式，输入捕获配置为同时捕获 AB 两相的上升 / 下降沿，即为 4 个计数 / pulse。  
![](https://img-blog.csdnimg.cn/20210614105820361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)  
为了验证上面说的对不对，打开`odrivetool`。

使用`odrv0.axis0.encoder.shadow_count`指令，查询编码器计数。

查询当前编码器计数，计数值为 20774。  
接着将电机转子记上记号，用手拨动一整圈，再次查询为 37163。

那么应设置的 CPR 值为 37163 - 20774 = 16389 ≈ `16384`。

有些误差很正常，毕竟不能保证电机每次都转到相同的位置。但记住 CPR = 编码器线束 * 4。  
![](https://img-blog.csdnimg.cn/20210614111042198.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

知道了电机和编码器的大致参数，下面使用调试工具对控制器进行配置。

推荐两个视频教程：  
[创客基地 ODrive 视频教程（过于详细，从入门到退坑）](https://space.bilibili.com/393688975/video?keyword=oDrive)  
[灯哥 ODrive 视频教程（拖更严重）](https://www.bilibili.com/video/BV19v411W7hv/?spm_id_from=333.788.recommend_more_video.0)

关于磁编码器原理，及如何用 TLE5012B 构成一款伺服电机的博文。  
[TLE5012B 磁编码器原理](https://blog.csdn.net/Mark_md/article/details/114889305?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162366290016780357249933%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=162366290016780357249933&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-114889305.nonecase&utm_term=%E7%BC%96%E7%A0%81%E5%99%A8&spm=1018.2226.3001.4450)

ODrive 教程资源导航目录
===============

[ODrive 踩坑（一）windows 下使用环境的搭建，odrivetool 及 USB 驱动的安装](https://blog.csdn.net/Mark_md/article/details/117885698?spm=1001.2014.3001.5501)

[ODrive 踩坑（二）3508 电机和 TLE5012B 磁编码器参数配置、校准、位置闭环模式转动电机（TLE5012B - ABI）](https://blog.csdn.net/Mark_md/article/details/117898801?spm=1001.2014.3001.5501)

[ODrive 踩坑（三）AS5047P 磁编码器的 ABI 接口](https://blog.csdn.net/Mark_md/article/details/119524819?spm=1001.2014.3001.5501)

[ODrive 踩坑（四）AS5047P-SPI 绝对值磁编码器，不需每次上电校准无刷电机，直接上电可用](https://blog.csdn.net/Mark_md/article/details/119774663?spm=1001.2014.3001.5501)

[ODrive 踩坑（五）驱动云台电机、低齿槽转矩电机实现高精度定位](https://blog.csdn.net/Mark_md/article/details/119860059?spm=1001.2014.3001.5501)

接线、上电
=====

*   电机编码器连接右侧的接口，电机三根线接在 M0 位置。
    
*   主电源插电，USB 连电脑。AUX 端口上的制动电阻可以不接。  
    ![](https://img-blog.csdnimg.cn/20210614175334640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)  
    ![](https://img-blog.csdnimg.cn/20210614175642916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)
    
*   cmd 打开命令行窗口，启动 odrivetool，确认连接。
    

一、电机校准前的配置
==========

1.1、恢复出厂配置
----------

问过了驱动器的卖家，出厂的固件为`v0.5.1`，有参数。适配自己的电机时，需要先`恢复出厂配置`。

```
# 恢复出厂配置
odrv0.erase_configuration()
```

这期间，ODrive 会断连一次并自动重新连接电脑。  
![](https://img-blog.csdnimg.cn/2021061411254169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

1.2、配置主板参数
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

![](https://img-blog.csdnimg.cn/2021061413424516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

1.3、配置电机参数
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

![](https://img-blog.csdnimg.cn/20210614134436462.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

1.4、配置编码器参数
-----------

```
# 配置电机0编码器类型。当前使用的是ABI正交（增量）编码器。
odrv0.axis0.encoder.config.mode = ENCODER_MODE_INCREMENTAL

# 配置电机0编码器CPR（每转一圈，编码器的计数），为编码器线束*4，博客开篇有讲
odrv0.axis0.encoder.config.cpr = 16384

# 保存参数
odrv0.save_configuration()
```

![](https://img-blog.csdnimg.cn/20210614134549729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

1.5、配置控制器参数（位置闭环模式、配置 PID 参数）
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
![](https://img-blog.csdnimg.cn/20210614134844336.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

二、电机和编码器校准
==========

电机和编码器校准，可以分为两条指令单独做，也可以用一条指令一起做。

**分开做：（分开做和一起做只选择一组进行操作即可）**

```
# 进行电机参数校准（运行后电机会发出哔~的一声）
odrv0.axis0.requested_state = AXIS_STATE_MOTOR_CALIBRATION

# 设置电机预校准。（不用每次上电都哔~的一声）
# 驱动器会将本次校准值保存，避免上电启动后自动校准，以加快启动速度。
odrv0.axis0.motor.config.pre_calibrated = True

# 进行编码器校准（运行后，电机会正转一圈再反转一圈）
odrv0.axis0.requested_state = AXIS_STATE_ENCODER_OFFSET_CALIBRATION
```

**一起做：**

```
# 进行电机参数和编码器校准（运行后电机会发出哔~的一声，并电机会正转一圈再反转一圈）
odrv0.axis0.requested_state = AXIS_STATE_FULL_CALIBRATION_SEQUENCE

# 设置电机预校准。（不用每次上电都哔~的一声）
# 驱动器会将本次校准值保存，避免上电启动后自动校准，以加快启动速度。
odrv0.axis0.motor.config.pre_calibrated = True
```

**之后进入闭环模式：**

```
# 配置电机为闭环模式
odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
```

进入闭环模式后，电机会有轻微的滋滋电流声。用手扭动电机，电机会产生反抗的力矩，松手后，电机会转回原来的位置。

如手拨后，如电机未回到原来位置，或在原来位置反复运动，或者能明显感受到过冲，都需要重调 1.5 章节的 PID 参数。  
  

**最后设置为上电自动校准，自动进入闭环。**

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

重启后，电机会自动进行编码器校准，正转一圈反转一圈归位，并自动进入闭环模式。

之后可直接发送控制指令，来让电机运动。  
![](https://img-blog.csdnimg.cn/20210614135644211.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hcmtfbWQ=,size_16,color_FFFFFF,t_70)

三、下发控制指令、控制电机运行
===============

发送控制指令，控制电机的旋转位置。

```
# 控制电机运行到100圈的位置
odrv0.axis0.controller.input_pos = 100

# 控制电机运行到0圈的位置
odrv0.axis0.controller.input_pos = 0

# 控制电机运行到-100圈的位置
odrv0.axis0.controller.input_pos = -100

# 释放电机（释放后，电机退出闭环模式，外力可轻松拨动电机）
odrv0.axis0.requested_state = AXIS_STATE_IDLE

# 配置电机为闭环模式
odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
```

释放电机后，用手可拨动电机到任意位置。  
重新进入闭环模式，电机并不会返回原来位置，请放心操作。

![](https://img-blog.csdnimg.cn/20210614184015467.gif)  
定位效果很好，迅速且精准。

现在的 PID 只是根据手感进行的粗调，略微偏软，抽空结合 GUI 工具进行细调，期望钉钉子的效果。  
（主要是内环速度 P 略微一高就震荡，不敢加高。不过也可能和我没有把磁编码器的磁铁与电机轴固定好有关，现在只是让径向磁铁吸附在电机轴上，并没有用胶水固定，兴许震荡的原因和这个有些关联）

上面的操作和指令同样适用于`电机1`，只要将 axis0 替换为 axis1。

另外命令行与终端是一样的，用键盘上下键，可快速切换到上 / 下一条指令。

1.5 章节的 PID，是我根据自己的电机调试的，并不一定适合其他电机，可能会引发震荡。  
如控制效果不理想，请重调 PID。细调 PID 需要 GUI 工具，来绘制位置 / 速度 / 电流的波形，作为 PID 调整的参考。详见下一节。