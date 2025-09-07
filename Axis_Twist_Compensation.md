# 轴扭转补偿 (Axis Twist Compensation)

本文档介绍 `[axis_twist_compensation]` 模块。

某些打印机的X轴导轨可能存在轻微扭曲，这会导致安装在X滑座上的探针测量结果出现偏差。这种现象在Prusa MK3、Sovol SV06等设计的打印机上较为常见，并在[探针位置偏差](Probe_Calibrate.md#location-bias-check)一节中有进一步描述。它可能导致[床网校准](Bed_Mesh.md)、[螺丝倾斜调整](G-Codes.md#screws_tilt_adjust)、[Z轴倾斜调整](G-Codes.md#z_tilt_adjust)等探针操作返回不准确的床面数据。

该模块通过用户手动测量来校正探针的测量结果。请注意，如果您的轴存在明显扭曲，强烈建议先通过机械手段进行修复，再应用软件校正。

**警告**：该模块目前与可拆卸探针不兼容，如果使用，它会尝试在未连接探针的情况下探测热床。

## 补偿使用概述

> **提示**：请确保正确设置了[探针X和Y偏移量](Config_Reference.md#probe)，因为它们对校准影响很大。

### 基本用法：X轴校准
1.  设置好 `[axis_twist_compensation]` 模块后，运行：
    ```
    AXIS_TWIST_COMPENSATION_CALIBRATE
    ```
    此命令默认校准X轴。
    -   校准向导会提示您在热床的几个点上测量探针的Z偏移量。
    -   默认情况下，校准使用3个点，但您可以通过以下选项指定不同的数量：
        ```
        SAMPLE_COUNT=<值>
        ```

2.  **调整您的Z偏移量：**
    完成校准后，务必[调整您的Z偏移量](Probe_Calibrate.md#calibrating-probe-z-offset)。

3.  **执行床面调平操作：**
    根据需要使用基于探针的操作，例如：
    -   [螺丝倾斜调整](G-Codes.md#screws_tilt_adjust)
    -   [Z轴倾斜调整](G-Codes.md#z_tilt_adjust)

4.  **完成设置：**
    -   归位所有轴，如有必要，执行一次[床网校准](Bed_Mesh.md)。
    -   运行测试打印，如有需要，再进行任何[微调](Axis_Twist_Compensation.md#fine-tuning)。

### Y轴校准
Y轴的校准过程与X轴类似。要校准Y轴，请使用：
```
AXIS_TWIST_COMPENSATION_CALIBRATE AXIS=Y
```
这将引导您完成与X轴相同的测量过程。

> **提示**：床温、喷嘴温度和喷嘴尺寸似乎对校准过程没有影响。

## [axis_twist_compensation] 设置和命令

`[axis_twist_compensation]` 的配置选项可在[配置参考](Config_Reference.md#axis_twist_compensation)中找到。

`[axis_twist_compensation]` 的命令可在[G代码参考](G-Codes.md#axis_twist_compensation)中找到。
