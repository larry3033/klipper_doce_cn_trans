# 床网

床网模块可以用于补偿床面的不规则性，以实现更好的首层均一性。需要注意的是，基于软件的校正无法达到完美的结果，它只能近似地模拟床面的形状。床网模块也无法对机械和电气问题进行补偿。如果一个轴倾斜或探针不准确，那么床网模块将无法从探测过程中获得准确的结果。

在进行网格校准之前，需要先校准探针的 Z 偏移。如果使用限位开关进行Z轴定位，也需要对其进行校准。请参阅[探针校准](Probe_Calibrate.md)和[手动调平](Manual_Level.md)中的 Z_ENDSTOP_CALIBRATE 获取更多信息。

## 基本配置

### 矩形热床

此示例假定打印机具有 250 mm x 220 mm 矩形床和一个 x 偏移为 24 mm和 y 偏移为 5 mm的探针。

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
```

- `speed: 120` *默认值：50* 探针在两个点之间移动的速度。
- `horizontal_move_z: 5` *默认值：5* 探针前往下一个点之前Z需要抬升的高度。
- `mesh_min: 35,6` *（必须存在）*第一个探测的坐标，距离原点最近。该坐标就是探针所在的位置。
- `mesh_max: 240, 198` *Required* The probed coordinate farthest from the origin. This is not necessarily the last point probed, as the probing process occurs in a zig-zag fashion. As with `mesh_min`, this coordinate is relative to the probe's location.
- `probe_count: 5, 3` *默认值：3, 3* 每个轴上要探测的点数，指定为 X, Y 整数值。 在本示例中，将沿 X 轴探测 5 个点，沿 Y 轴探测 3 个点，总共探测 15 个点。 请注意，如果您想要一个方形网格，例如 3x3，可以将指定其为一个整数值，比如 `probe_count: 3`。 请注意，网格需要沿每个轴的最小 probe_count 为3。

下图演示了如何使用 `mesh_min`、`mesh_max` 和 `probe_count` 选项来生成探测点。 箭头表示探测过程的运动方向，从“mesh_min”开始。 图中所示，当探针位于“mesh_min”时，喷嘴将位于 (11, 1)，当探针位于“mesh_max”时，喷嘴将位于 (206, 193)。

![矩形网床基本配置](img/bedmesh_rect_basic.svg)

### 圆形热床

本示例假设打印机配备的圆床半径为 100 mm。 我们将使用与矩形网床示例相同的探针偏移来演示，X 偏移为 24 mm，Y 偏移为 5 mm。

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_radius: 75
mesh_origin: 0, 0
round_probe_count: 5
```

- `mesh_radius: 75` *必须配置* 探测网格范围的半径（单位：mm），相对于 `mesh_origin`。 请注意，探针的偏移会限制网格半径的大小。 在此示例中，大于 76 mm的半径会将打印头移动到打印机的范围之外。
- `mesh_origin: 0, 0` *默认值：0, 0* 探测网格的中心点。 该坐标相对于探针的位置。 虽然默认值为 0,0，但如果希望探测床的边角可以修改该值。 请参阅下图。
- `round_probe_count: 5` *默认值： 5* 这是一个整数值，用于限制沿 X 轴和 Y 轴的最大探测点数。 “最大”是指沿网格原点探测的点数。 该值必须是奇数，因为需要探测网格的中心。

下面的插图展示了如何生成探测点。如您所见，将 `mesh_origin` 设置为 (-10, 0) 允许我们指定更大的网格半径为 85。

![圆形网床基本配置](img/bedmesh_round_basic.svg)

## 高级配置

下面详细解释了更高级的配置选项。 每个示例都将建立在上面显示的基本矩形床配置之上。 每个高级选项都以相同的方式应用于圆床。

### 网格插值

虽然可以直接使用简单的双线性插值来对探测矩阵进行采样，以确定探测点之间的 Z 值，但通常使用更高级的插值算法来插值额外的点，以增加网格密度，效果通常很好。这些算法会向网格添加曲率，试图模拟床的材料属性。床网提供拉格朗日和双三次插值来实现这一点。

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
mesh_pps: 2, 3
algorithm: bicubic
bicubic_tension: 0.2
```

- `mesh_pps: 2,3` *默认值：2,2*`mesh_pps` 选项是每段的网格点数的简写。 此选项指定沿 x 轴和 y 轴为每个线段插值的点数。 “段”被视为每个探测点之间的间隔。 与 `probe_count` 一样，`mesh_pps` 可以是 X, Y 整数对，也可以是同时应用于两个轴的单个整数。 在此示例中，沿 X 轴有 4 个线段，沿 Y 轴有 2 个线段。 这计算为沿 X 的 8 个插值点，沿 Y 的 6 个插值点，从而产生 13x8 网格。 请注意，如果 mesh_pps 设置为 0，则禁用网格插值，并且将直接对探测网格进行采样。
- `algorithm: lagrange` *默认值：lagrange* 用于插入网格的算法。 可能是 `lagrange` or `bicubic`。 拉格朗日插值最多为 6 个探测点，因为大量样本容易发生振荡。 双三次插值要求沿每个轴至少有 4 个探测点，如果指定的点少于 4 个，则强制拉格朗日采样。 如果 `mesh_pps` 设置为 0，则该值将被忽略，因为没有进行网格插值。
- `bicubic_tension: 0.2` *默认值：0.2* 双三次插值的张力值。如果`algorithm` 选项设置为双三次，则可以指定张力值。 张力越高，内插的斜率越大。 调整时要小心，因为较高的值也会产生更多的过冲，这将导致插值高于或低于探测点。

下图显示了如何使用上述选项生成网格插值。

![网床插值](img/bedmesh_interpolated.svg)

### 移动拆分

床网的工作原理是拦截 G 代码移动命令并对其 Z 坐标进行变换。长的移动必须被分割成较小的移动以正确地遵循床的形状。下面的选项可以控制分割的行为。

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
move_check_distance: 5
split_delta_z: .025
```

- `move_check_distance: 5` *默认值：5* 在执行拆分之前检查 Z 中需要变化的最小距离。 在此示例中，算法将遍历超过 5 毫米的移动。 每 5mm 将查找一次网格的Z ，并将其与前一次移动的 Z 值进行比较。 如果三角洲满足 `split_delta_z` 设置的阈值，则移动将被拆分并继续遍历。 重复此过程，直到到达移动结束处，在此将应用最终调整。 比 `move_check_distance` 短的移动将正确的 Z 调整直接应用于移动，无需遍历或拆分。
- `split_delta_z: .025` *默认值：.025* 如上所述，这是触发移动拆分所需的最小偏差。 在上面的示例中，任何偏差为 +/- .025 mm的 Z 值都将触发拆分。

通常这些选项的默认值已经足够了，事实上 `move_check_distance` 的默认值 5mm 可能过于保守。但是，高级用户可能希望尝试这些选项，以获取最佳的第一层效果。

### 网格淡出

启用“网格淡出”后，Z 轴的调整将在配置中定义的距离范围内逐步消失。 这是通过对层高进行小幅调整来实现的，根据床的形状增加或减少。 网格淡出完成后，不再使用 Z 调整，使打印的表面是平坦的而不是床弯曲的形状。 网格淡出也可能会产生一些不良表现，如果网格淡出过快，可能会导致打印件上出现可见的瑕疵（伪影）。 此外，如果您的床明显变形，网格淡出会缩小或拉伸打印件的 Z 高度。 因此，默认情况下禁用网格淡出。

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
fade_start: 1
fade_end: 10
fade_target: 0
```

- `fade_start: 1` *默认值：1* 开始网格淡出的值，在设定的fade_start值之后逐步停止调整Z的高度。 建议在打印几层之后再开始淡出层高。
- `fade_end: 10` *默认值：0* 网格淡出完成的 Z 高度。 如果此值低于`fade_start`，则禁用网格淡出。 该值可以根据打印表面的弯曲程度进行调整。 明显弯曲的表面应该在将网格淡出的距离长。 接近平坦的表面可能能够降低该值以更快地逐步淘汰。 如果对 `fade_start` 使用默认值 1，则 10mm 是一个合理的值。
- `fade_target: 0` *默认值：网格的平均 Z 值* `fade_target` 可以被视为在淡化完成后应用于整个床面的额外 Z 偏移量。一般来说，我们希望这个值为 0，但有些情况下不应该是这样的。例如，假设您在床上的归位位置是一个异常值，比床面的平均探测高度低 0.2 毫米。如果 `fade_target` 为 0，淡化将会使整个床面平均降低 0.2 毫米。通过将 `fade_target` 设置为 0.2，淡化区域将会提高到 0.2 毫米，但是，床面的其余部分将保持原大小。通常最好将 `fade_target` 留在配置中，以便使用网格的平均高度，但是如果您想在床面的特定部分上打印，则可能需要手动调整淡化目标。

### 配置零点参考位置

许多探针容易受到“漂移”的影响，即由热量或干扰引起的探测不准确。这使得计算探针的Z偏移量变得困难，尤其是在不同的热床温度下。因此，一些打印机使用限位开关来对Z轴进行归位，而使用探针来校准网格。在这种配置下，可以调整网格，使得(X, Y) `参考位置` 不进行任何调整。`参考位置` 应该是进行[Z_ENDSTOP_CALIBRATE](./Manual_Level.md#calibrating-a-z-endstop) 纸张测试时热床上的位置。bed_mesh模块提供了`zero_reference_position`选项来指定此坐标：

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
zero_reference_position: 125, 110
probe_count: 5, 3
```

- `ZERO_REFERENCE_POSITION：`*默认值：无(禁用)*`ZERO_REFERENCE_POSITION`期望(X，Y)坐标与上面描述的`参考位置`匹配。如果坐标位于网格内，则网格将偏移，因此参考位置应用零点调整。如果坐标位于网格之外，则将在校准后探测该坐标，并将生成的z值用作z偏移。请注意，如果需要探测，则此坐标不能位于指定为`FAULTY_REGION`的位置。

#### 不推荐使用的Relative_Reference_Index

使用`Relative_Reference_index`选项的现有配置必须更新为使用`ZERO_REFERENCE_Position`。对[BED_MESH_OUTPUT PGP=1](#output)GCODE命令的响应将包括与索引相关的(X，Y)坐标；该位置可用`ZERO_REFERENCE_POSITION`的值。输出将如下所示：

```
// bed_mesh: generated points
// Index | Tool Adjusted | Probe
// 0 | (1.0, 1.0) | (24.0, 6.0)
// 1 | (36.7, 1.0) | (59.7, 6.0)
// 2 | (72.3, 1.0) | (95.3, 6.0)
// 3 | (108.0, 1.0) | (131.0, 6.0)
... (additional generated points)
// bed_mesh: relative_reference_index 24 is (131.5, 108.0)
```

*注意：上述输出在初始化时也会打印在`klippy.log`中。*

在上面的例子中，我们看到`Relative_Reference_index`与它的坐标一起打印。因此，`ZERO_REFERENCE_Position`是`131.5,108`。

### 故障区域

由于特定位置的“故障”，热床的某些区域在探测时可能会报告不准确的结果。 最好的例子是带有用弹簧钢板的磁铁热床。 这些磁铁处和周围的磁场可能干扰探针触发的高度，从而导致网格无法准确表示这些位置的表面。 **注意：不要与探头位置偏差导致探测结果不准确的结果混淆。**

可以配置 `faulty_region` 选项来避免这种影响。 如果生成的点位于故障区域内，热床网格将尝试在该区域的边界处探测最多 4 个点。 这些探测的平均值将插入网床中作为生成的 (X, Y) 坐标处的 Z 值。

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
faulty_region_1_min: 130.0, 0.0
faulty_region_1_max: 145.0, 40.0
faulty_region_2_min: 225.0, 0.0
faulty_region_2_max: 250.0, 25.0
faulty_region_3_min: 165.0, 95.0
faulty_region_3_max: 205.0, 110.0
faulty_region_4_min: 30.0, 170.0
faulty_region_4_max: 45.0, 210.0
```

- `faulty_region_{1...99}_min` `faulty_region_{1...99}_max` *默认值：None （无）(disabled（禁用）)* 故障区域的定义方式类似床网本身，必须为每个区域指定最小和最大（X, Y）坐标。一个故障区域可以延伸到网格之外，但是产生的替代探测点总是在网格的边界内。两个区域不可以重叠。

下面的图片说明了当一个生成的探测点位于一个故障区域内时，如何生成替代探测点。所显示的区域与上述样本配置中的区域一致。替代点和它们的坐标以绿色标识。

![bedmesh_interpolated](img/bedmesh_faulty_regions.svg)

### 自适应网格 (Adaptive Meshes)

自适应床网格是一种通过仅探测打印对象所占用区域来加速床网格生成的方法。启用后，该方法会根据定义的打印对象所占据的区域自动调整网格参数。

自适应网格区域将根据所有定义的打印对象边界所定义的区域计算得出，以覆盖每个对象，包括配置中定义的任何边距。计算出区域后，探测点的数量将根据默认网格区域与自适应网格区域的比率进行缩放。举例说明：

对于一个150mm x 150mm的热床，`mesh_min`设置为`25,25`，`mesh_max`设置为`125,125`，则默认网格区域是一个100mm x 100mm的正方形。如果自适应网格区域为`50,50`，则自适应区域与默认网格区域的比率为`0.5x0.5`。

如果`bed_mesh`配置中`probe_count`指定为`7x7`，那么自适应床网格将使用4x4个探测点（7 * 0.5 向上取整）。

[adaptive_bedmesh](img/adaptive_bed_mesh.svg)

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
adaptive_margin: 5
```

- `adaptive_margin` *默认值：0* 在定义的打印对象所占用的热床区域周围添加的边距（单位：毫米）。下图显示了`adaptive_margin`为5mm的自适应床网格区域。自适应网格区域（绿色区域）的计算方式为使用的热床区域（蓝色区域）加上定义的边距。

   [adaptive_bedmesh_margin](img/adaptive_bed_mesh_margin.svg)

本质上，自适应床网格使用正在打印的G代码文件中定义的对象。因此，每个G代码文件都会生成一个探测热床不同区域的网格。因此，不应重复使用自适应网格。如果使用自适应网格，预期是每次打印都会生成一个新的网格。

还需要注意的是，自适应床网格最适合那些通常能够探测整个热床并使最大偏差小于或等于1层高度的机器。对于存在机械问题、通常需要全床网格来补偿的机器，在**超出**探测区域进行打印移动时可能会产生不良结果。如果全床网格的偏差大于1层高度，在使用自适应床网格并尝试在网格区域外进行打印移动时必须格外小心。

## 表面扫描 (Surface Scans)

某些探针（例如[涡流探针](./Eddy_Probe.md)）能够“扫描”热床表面，即这些探针可以在不抬起工具的情况下对网格进行采样。要激活扫描模式，应在`BED_MESH_CALIBRATE` G代码命令中传递`METHOD=scan`或`METHOD=rapid_scan`探针参数。

### 扫描高度 (Scan Height)

扫描高度由`[bed_mesh]`中的`horizontal_move_z`选项设置。此外，也可以通过`BED_MESH_CALIBRATE` G代码命令中的`HORIZONTAL_MOVE_Z`参数提供。

扫描高度必须足够低，以避免扫描错误。通常，2mm的高度（例如：`HORIZONTAL_MOVE_Z=2`）效果良好，前提是探针安装正确。

需要注意的是，如果探针高于表面超过4mm，则结果将无效。因此，对于表面偏差严重或倾斜度极大的热床（且未校正），无法进行扫描。

### 快速（连续）扫描 (Rapid (Continuous) Scanning)

执行`rapid_scan`时，应记住结果会存在一定误差。这种误差应足够小，以便在具有合理厚层高的大面积打印区域上使用。某些探针可能比其他探针更容易产生误差。

不建议在快速模式下扫描“密集”的网格。快速扫描引入的一些误差可能来自传感器的高斯噪声，而密集的网格会反映这种噪声（即会出现峰谷）。

Bed Mesh会尝试优化行进路径，以根据配置提供最佳可能的结果。这包括在采集样本时避开故障区域，以及在改变方向时“超出”网格。这种超出（overshoot）可以改善网格边缘的采样，但它要求网格配置方式允许工具在网格外移动。

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5
scan_overshoot: 8
```

- `scan_overshoot` *默认值：0（禁用）* 网格外可用的最大移动距离（单位：毫米）。对于矩形热床，这适用于X轴方向的移动；对于圆形热床，它适用于整个半径。工具必须能够在网格外移动指定的距离。此值用于在执行“快速扫描”时优化行进路径。可指定的最小值为1。默认无超出。

如果未配置扫描超出，则在改变方向时不会应用行进路径优化。

## 床网 G代码

### 校准

`BED_MESH_CALIBRATE PROFILE=<name> METHOD=[manual | automatic | scan | rapid_scan] \ [<probe_parameter>=<value>] [<mesh_parameter>=<value>] [ADAPTIVE=[0|1] \ [ADAPTIVE_MARGIN=<value>]` *Default Profile: default* *Default Method: automatic if a probe is detected, otherwise manual*  *Default Adaptive: 0*  *Default Adaptive Margin: 0*

启动床网校准的探测程序。

当命令完成时，网格将立即准备好使用，并保存到由`PROFILE`参数指定的配置文件中，如果没有指定，则保存到`default`配置文件。`METHOD`参数接受以下值之一：

- `METHOD=manual`：启用使用喷嘴和纸张测试的手动探测。
- `METHOD=automatic`：自动（标准）探测。这是默认设置。
- `METHOD=scan`：启用表面扫描。工具将在每个位置上方暂停以收集样本。
- `METHOD=rapid_scan`：启用连续表面扫描。

当选择了`manual`以外的探测方法时，XY位置会自动调整以包含X和/或Y偏移量。这意味着在进行非手动探测时，系统会考虑到任何已定义的偏移量，从而确保探测准确性和网格质量。这为用户提供了灵活性，无论是需要精确控制的手动校准，还是追求效率和速度的自动或扫描方法，都可以根据实际需求来选择。

可以通过指定网格参数来修改探测区域。以下参数可用：

- 矩形打印床（笛卡尔 Cartesian）：
   - `MESH_MIN`
   - `MESH_MAX`
   - `PROBE_COUNT`
- 圆形打印床（三角洲 delta）：
   - `MESH_RADIUS`
   - `MESH_ORIGIN`
   - `ROUND_PROBE_COUNT`
- 全部打印床：
   - `MESH_PPS`
   - `ALGORITHM`
   - `ADAPTIVE`
   - `ADAPTIVE_MARGIN`

有关在网格中使用的配置参数详见配置文档。

### 配置

`BED_MESH_PROFILE SAVE=<名称> LOAD=<名称> REMOVE=<名称>`

在执行 BED_MESH_CALIBRATE 后，可以将当前网格状态保存到一个命名的配置中。这样不需要重新探测打印床就可以载入一个网格。在使用`BED_MESH_PROFILE SAVE=<名称>`保存了一个配置文件后，可以执行`SAVE_CONFIG` G代码将配置写入 printer.cfg。

可以通过运行 `BED_MESH_PROFILE LOAD=<名称>` 来载入配置。

需要注意的是，每次进行 BED_MESH_CALIBRATE 时，当前状态会自动保存到 *default* 配置文件中。可以按以下方式删除 *default* 配置文件：

`BED_MESH_PROFILE REMOVE=default`

任何其他保存的配置也可以用相同的方式删除，用你想删除的配置名称替换*default*。

#### 加载默认配置文件

以前版本的`bed_mesh`如果(default)默认配置存在，则始终在启动时加载名为*default*的配置文件。现已删除此行为，以允许用户确定何时加载配置文件。如果用户希望加载`default`配置文件，则建议将 `BED_MESH_PROFILE LOAD=default` 添加到其 `START_PRINT` 宏或其切片软件的“启动 G代码”配置中，视情况而定。

Note that this is not required if a new mesh is generated with `BED_MESH_CALIBRATE` in the `START_PRINT` macro or the slicer's "Start G-Code" and may produce unexpected results, especially with adaptive meshing.

或者可以通过添加`[delayed_gcode]`恢复在启动时加载配置文件的旧行为：

```ini
[delayed_gcode bed_mesh_init]
initial_duration: .01
gcode:
  BED_MESH_PROFILE LOAD=default
```

### 输出

`BED_MESH_OUTPUT PGP=[0 | 1]`

将当前网格状态输出到终端。请注意，输出的是网格本身

PGP 参数是“打印生成的点”的简写。如果设置了`PGP=1`，生成的探测点将输出到终端：

```
// bed_mesh: generated points
// Index | Tool Adjusted | Probe
// 0 | (11.0, 1.0) | (35.0, 6.0)
// 1 | (62.2, 1.0) | (86.2, 6.0)
// 2 | (113.5, 1.0) | (137.5, 6.0)
// 3 | (164.8, 1.0) | (188.8, 6.0)
// 4 | (216.0, 1.0) | (240.0, 6.0)
// 5 | (216.0, 97.0) | (240.0, 102.0)
// 6 | (164.8, 97.0) | (188.8, 102.0)
// 7 | (113.5, 97.0) | (137.5, 102.0)
// 8 | (62.2, 97.0) | (86.2, 102.0)
// 9 | (11.0, 97.0) | (35.0, 102.0)
// 10 | (11.0, 193.0) | (35.0, 198.0)
// 11 | (62.2, 193.0) | (86.2, 198.0)
// 12 | (113.5, 193.0) | (137.5, 198.0)
// 13 | (164.8, 193.0) | (188.8, 198.0)
// 14 | (216.0, 193.0) | (240.0, 198.0)
```

"Tool Adjusted"（工具调整）点指每个点的喷嘴位置，"Probe"（探针）点指探头位置。请注意，手动探测时"Probe"（探针）点时将同时指工具和喷嘴位置。

### 清除网格状态

`BED_MESH_CLEAR`

此 gcode 可用于清除内部网格状态。

### 应用X/Y偏移量

`BED_MESH_OFFSET [X=<value>] [Y=<value>] [ZFADE=<value>]`

这对于拥有多个独立挤出机的打印机非常有用，因为在换工具后需要偏移量来进行正确的Z轴调整。偏移量应相对于主挤出机来指定。也就是说，如果副挤出机安装在主挤出机的右侧，则应指定正的X偏移量；如果副挤出机安装在主挤出机“后面”，则应指定正的Y偏移量；如果副挤出机的喷嘴高于主挤出机的喷嘴，则应指定正的ZFADE偏移量。

请注意，ZFADE偏移量*不会*直接应用额外的调整。它的目的是在启用[网格渐变](#mesh-fade)时补偿`gcode offset`。例如，如果副挤出机高于主挤出机并且需要负的gcode偏移量，即`SET_GCODE_OFFSET Z=-.2`，则可以在`bed_mesh`中通过`BED_MESH_OFFSET ZFADE=.2`来补偿。

## 床网格Webhooks API

### 导出网格数据

`{"id": 123, "method": "bed_mesh/dump_mesh"}`

导出当前网格和所有已保存配置文件的配置和状态。

`dump_mesh`端点接受一个可选参数`mesh_args`。该参数必须是一个对象，其键和值是[BED_MESH_CALIBRATE](#bed_mesh_calibrate)可用的参数。这将在返回结果之前使用提供的参数更新网格配置和探测点。除非希望在执行`BED_MESH_CALIBRATE`之前可视化探测点和/或行进路径，否则建议省略网格参数。

## 可视化与分析

大多数用户可能会发现，像Mainsail、Fluidd和Octoprint这样的应用程序中包含的可视化工具足以满足基本分析需求。然而，Klipper的`scripts`文件夹中包含`graph_mesh.py`脚本，可用于进行额外的可视化和更详细的分析，对于调试硬件或`bed_mesh`产生的结果特别有用：

```
usage: graph_mesh.py [-h] {list,plot,analyze,dump} ...

Graph Bed Mesh Data

positional arguments:
  {list,plot,analyze,dump}
    list                列出可用的绘图类型
    plot                绘制指定类型
    analyze             对网格数据执行分析
    dump                将API响应转储到json文件

options:
  -h, --help            显示此帮助信息并退出
```

### 先决条件

与Klipper提供的大多数绘图工具一样，`graph_mesh.py`需要`matplotlib`和`numpy` python依赖项。此外，通过Moonraker的websocket连接到Klipper需要`websockets` python依赖项。虽然所有可视化都可以输出到`svg`文件，但`graph_mesh.py`提供的大多数可视化在桌面级PC的实时预览模式下查看效果更佳。例如，3D可视化可以在预览模式下旋转和缩放，路径可视化可以选择在预览模式下动画播放。

### 绘制网格数据

`graph_mesh.py`工具可以绘制几种类型的可视化。通过运行`graph_mesh.py list`可以显示可用的类型：

```
graph_mesh.py list
points    绘制原始生成的点
path      绘制探测行进路径
rapid     绘制快速扫描行进路径
probedz   绘制探测的Z值
meshz     绘制网格Z值
overlay   将当前探测的网格与配置文件叠加绘制
delta     绘制当前探测网格与配置文件之间的差值
```

绘制可视化时有几种可用选项：

```
usage: graph_mesh.py plot [-h] [-a] [-s] [-p PROFILE_NAME] [-o OUTPUT] <plot type> <input>

positional arguments:
  <plot type>           要绘制的数据类型
  <input>               Klipper套接字的路径/URL或json文件的路径

options:
  -h, --help            显示此帮助信息并退出
  -a, --animate         在实时预览中为路径添加动画
  -s, --scale-plot      使用Klipper报告的axis_minimum和axis_maximum值来缩放绘图的X/Y轴
  -p PROFILE_NAME, --profile-name PROFILE_NAME
                        为'probedz'生成3D网格可视化时可选的配置文件名称
  -o OUTPUT, --output OUTPUT
                        输出文件路径
```

以下是每个参数的描述：

- `plot type`：指定要生成的可视化类型的必需位置参数。必须是`graph_mesh.py list`命令输出的类型之一。
- `input`：包含输入源路径或URL的必需位置参数。必须是以下之一：
   - Klipper的Unix域套接字路径
   - Moonraker实例的URL
   - 由`graph_mesh.py dump <input>`生成的json文件路径
- `-a`：为`path`和`rapid`可视化类型提供可选的动画。动画仅适用于实时预览。
- `-s`：可选地使用dump文件生成时Klipper的`toolhead`对象报告的`axis_minimum`和`axis_maximum`值来缩放绘图。
- `-p`：生成`probedz` 3D网格可视化时可指定的配置文件名称。生成`overlay`或`delta`可视化时必须提供此参数。
- `-o`：可选的文件路径，指示脚本应将可视化保存到此位置而不是以预览模式运行。图像以`svg`格式保存。

例如，要通过Klipper的unix套接字绘制动画快速路径：

```
graph_mesh.py plot -a rapid ~/printer_data/comms/klippy.sock
```

或者通过Moonraker绘制网格的3D可视化：

```
graph_mesh.py plot meshz http://my-printer.local
```

### 床网格分析

`graph_mesh.py`工具也可用于对[bed_mesh/dump_mesh](#dumping-mesh-data) API提供的数据执行分析：

```
graph_mesh.py analyze <input>
```

与`plot`命令一样，`<input>`必须是Klipper的unix套接字路径、Moonraker实例的URL或由dump命令生成的json文件路径。

分析开始时，将对`bed_mesh`在dump时生成的点和探测路径执行各种检查。这包括以下内容：

- 生成的探测点数量，不包括任何添加
- 生成的探测点数量，包括因故障区域和/或配置的零参考位置而生成的任何点
- 执行快速扫描时生成的探测点数量
- 快速扫描生成的总移动次数
- 验证快速扫描生成的探测点与标准探测过程生成的探测点是否相同
- 对标准探测路径和快速扫描路径进行“回溯”检查。回溯可定义为在探测过程中多次移动到同一位置。标准探测中不应发生回溯。在快速扫描中，故障区域*可能*会导致回溯，这是为了避免在接近或离开探测位置时进入故障区域，但其他情况下不应发生。

接下来，将分析dump中存在的每个探测网格，从dump时加载的网格（如果存在）开始，然后是任何已保存的配置文件。提取以下数据：

- 网格形状（最小X,Y，最大X,Y 探测点数）
- 网格Z范围（最小Z，最大Z）
- 网格中的平均Z值
- 网格中Z值的标准偏差

除了上述内容外，还会对形状相同的网格执行差值分析，报告以下内容：

- 两个网格之间差值的范围（最小值和最大值）
- 平均差值
- 差值的标准偏差
- 绝对最大差值
- 绝对平均差值

### 将网格数据保存到文件

`dump`命令可用于将响应保存到文件中，以便在故障排除时共享进行分析：

```
graph_mesh.py dump -o <输出文件名> <输入>
```

`<input>`应为Klipper的unix套接字路径或Moonraker实例的URL。`-o`选项可用于指定输出文件的路径。如果省略，文件将保存在工作目录中，文件名格式如下：

`klipper-bedmesh-{年}{月}{日}{小时}{分钟}{秒}.json`
