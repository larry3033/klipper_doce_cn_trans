# 安装

这些说明假定软件将运行在基于 Linux 的主机上，并且该主机运行着与 Klipper 兼容的前端。建议使用树莓派或基于 Debian 的 Linux 设备等小型单板计算机（SBC，Small Board Computer）作为主机（有关其他选项，请参见[常见问题](FAQ.md#can-i-run-klipper-on-something-other-than-a-raspberry-pi-3)）。

根据本说明，主机指 Linux 设备，mcu 指打印机主板。SBC 指小型单板计算机，例如树莓派。

## 获取 Klipper 配置文件

大多数 Klipper 设置由“打印机配置文件”printer.cfg 决定，该文件将存储在主机上。通常可以通过在 Klipper [config 目录](../config/) 中查找以“printer-”前缀开头且与目标打印机相对应的文件来找到适当的配置文件。Klipper 配置文件包含安装过程中需要的有关打印机的技术信息。

如果 Klipper 配置目录中没有合适的打印机配置文件，请尝试搜索打印机制造商的网站，看看他们是否有合适的 Klipper 配置文件。

如果找不到打印机的配置文件，但可以找到打印机控制板的类型，则可以查找以“generic-”前缀开头的适当 [配置文件](../config/)。这些示例打印机模板文件应该足以成功完成初始安装，但需要进行一些自定义才能获得完整的打印机功能。

也可以从头开始定义一个新的打印机配置。然而，这需要关于打印机及其电子系统的大量技术知识。建议大多数用户从一个适当的配置文件开始。如果需要创建一个新的自定义打印机配置文件，那么可以先从最接近的[配置文件](../config/)的例子开始，并从 Klipper [配置参考文档](Config_Reference.md)了解进一步信息。

## 与 Klipper 交互

Klipper 是一个 3D 打印机固件，因此需要某种方式让用户与其进行交互。

目前最好的选择是通过 [Moonraker web API](https://moonraker.readthedocs.io/) 检索信息的前端，也可以选择使用 [Octoprint](https://octoprint.org/) 来控制 Klipper。

用户可自行选择使用哪种工具，但底层的 Klipper 在所有情况下都是相同的。我们鼓励用户研究可用的选项并做出明智的决定。

## 获取 SBC 的操作系统映像

有多种方式可以获取用于 SBC 的 Klipper 操作系统镜像，具体方法大多取决于您希望使用的前端。一些 SBC 主板制造商也提供以 Klipper 为中心的预装镜像。

两个主要基于 Moonraker 的前端是 [Fluidd](https://docs.fluidd.xyz/) 和 [Mainsail](https://docs.mainsail.xyz/)，其中后者提供了一个预制作的安装镜像 ["MainsailOS"](https://docs-os.mainsail.xyz/)，支持树莓派和部分 OrangePi 变体。

Fluidd 可以通过 KIAUH（Klipper 安装和更新助手）进行安装，如下所述，它是所有 Klipper 的第三方安装程序。

OctoPrint 可以通过流行的 OctoPi 镜像或通过 KIAUH 安装，此过程在 <OctoPrint.md> 中有说明

## 通过 KIAUH 安装

通常，您应从 SBC 的基础镜像开始，例如 RPiOS Lite，或者对于 x86 架构的 Linux 设备，则使用 Ubuntu Server。请注意，不建议使用桌面版系统，因为某些辅助程序可能会阻止 Klipper 的部分功能运行，甚至屏蔽对某些打印机主板的访问。

KIAUH 可用于在运行 Debian 系统的各种基于 Linux 的设备上安装 Klipper 及其相关程序。更多信息请访问 https://github.com/dw-0/kiauh

## 构建和刷写微控制器

要编译微控制器代码，首先在主机设备上运行以下命令：

```
cd ~/klipper/
make menuconfig
```

[打印机配置文件](#obtain-a-klipper-configuration-file)的顶部注释应该描述了"make menuconfig"期间需要设置的设置。在网络浏览器或文本编辑器中打开该文件，在文件顶部附近寻找这些说明。一旦适当的"menuconfig"设置被配置好了，按"Q"退出，然后按"Y"保存，运行：

```
make
```

如果[打印机配置文件](#obtain-a-klipper-configuration-file)顶部的注释描述了将最终镜像“烧录”到打印机控制板的自定义步骤，则请按照这些步骤操作，然后继续进行[配置 OctoPrint 以使用 Klipper](#configuring-octoprint-to-use-klipper)。
否则，通常采用以下步骤来"flash"打印机控制板。首先，需要确定连接到微控制器的串行端口。然后，运行以下程序：

```
ls /dev/serial/by-id/*
```

它应该报告类似以下的内容：

```
/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
```

通常，每台打印机都有自己独特的串行端口名称。此唯一名称将在刷新微控制器时使用。上面的输出中可能有多行 - 如果是这样，请选择与微控制器相对应的行。如果列出了许多项目并且选择不明确，请拔下电路板并再次运行命令，缺少的项目将是您的打印板（有关更多信息，请参阅 [FAQ](FAQ.md#wheres-my-serial-port)）。

对于使用 STM32 或兼容芯片、LPC 芯片及其他常见微控制器的设备，通常需要通过 SD 卡进行首次 Klipper 固件烧录。

使用此方法烧录时，务必将打印主板与主机的 USB 连接断开，因为某些主板可能会通过 USB 反向供电，从而导致烧录失败。

请注意，大多数使用 SD 卡进行烧录的打印主板会实现某种形式的烧录循环保护机制，以防 SD 卡长期留在主板上导致重复烧录。常见的有两种方法：

需要更改文件名（通常是“原厂”打印主板）：

此类主板要求每次烧录时固件文件必须使用不同的文件名（例如 firmware1.bin、firmware2.bin 等）。如果重复使用相同的文件名，主板可能会忽略该文件而不进行更新。

自动文件重命名（通常是 aftermarket 市售改装打印主板）：

其他主板允许使用相同的文件名（通常为 firmware.bin），但在烧录完成后，主板会自动将该文件重命名为 firmware.cur。这有助于指示固件已成功烧录，并防止在下次启动时重复烧录。

在烧录前，请务必确认您的主板采用的是上述哪种机制。

对于使用 Atmega 芯片的常见微控制器，例如 2560，代码可以使用类似以下内容进行烧录：

```
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
sudo service klipper start
```

请务必用打印机的唯一串行端口名称来更新 FLASH_DEVICE 参数。

对于使用 RP2040 芯片的常见微控制器，代码可以使用类似以下方式烧录：

```
sudo service klipper stop
make flash FLASH_DEVICE=first
sudo service klipper start
```

需要注意的是，RP2040 芯片在执行此操作前可能需要先进入 Boot 模式。

## 配置 Klipper

下一步是将[打印机配置文件](#obtain-a-klipper-configuration-file)复制到主机。

或许最简便的方法是使用 Mainsail 或 Fluidd 内置的编辑器来设置 Klipper 配置文件。这些编辑器允许用户打开配置示例文件，并将其保存为 printer.cfg。

另一种选择是使用支持通过 "scp" 和/或 "sftp" 协议编辑文件的桌面编辑器。有许多免费工具支持此功能（例如，Notepad++、WinSCP 和 Cyberduck）。在编辑器中打开打印机配置文件，然后将其保存为 pi 用户主目录下的 "printer.cfg" 文件（即，/home/pi/printer.cfg）。

或者，也可以通过 SSH 直接在主机上复制并编辑该文件。操作可能如下所示（请务必修改命令以使用正确的打印机配置文件名）：

```
cp ~/klipper/config/example-cartesian.cfg ~/printer.cfg
nano ~/printer.cfg
```

通常每台打印机都有自己独特的微控制器名称。刷写Klipper后，名称可能会改变，所以即使在闪存时已经完成，也要重新运行这些步骤。运行：

```
ls /dev/serial/by-id/*
```

它应该报告类似以下的内容：

```
/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
```

然后用这个唯一的名字更新配置文件。例如，更新`[mcu]`部分，类似于：

```
[mcu]
serial: /dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
```

在创建并编辑该文件后，需要在命令控制台中发送“restart”命令以加载配置。如果 Klipper 配置文件被成功读取，并且微控制器被成功识别和配置，使用“status”命令将显示打印机已就绪。

在定制打印机配置文件时，Klipper 报告配置错误是很正常的情况。如果发生错误，请对打印机配置文件进行必要的修正，并发出"restart"，直到"status"报告打印机已准备就绪。

Klipper 通过命令控制台以及 Fluidd 和 Mainsail 中的弹窗报告错误信息。“status”命令可用于重新显示错误信息。日志文件可查看，通常位于 `~/printer_data/logs/klippy.log`。

在Klipper报告打印机已就绪后，继续进入[配置检查文件](Config_checks.md)，对配置文件中的定义进行一些基本检查。其他信息见主[文档参考](Overview.md)。
