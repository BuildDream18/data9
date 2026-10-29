# 佰霖仪器与红外数据获取：NetViewX.OCX、NetClient.dll 安装及开发全指南

**简体中文版｜设备识别、下载渠道、Windows 11 环境、红外视频、热成像测温、SDK 开发与故障排查**  
**资料核查日期：** 2026 年 10 月 26 日  
**参考开发环境：** Windows 11 64 位；可选 Python 3.12；已获得合法访问权限的摄像机/仪器

> **首先说明：**「佰霖仪器」通常指 **河南佰霖仪器设备有限公司**。该公司网站销售工业检测及热成像类设备，包含 **BL-DT100 双光谱热成像测温设备**。
>
> 但是，目前**无法确认**您手里的佰霖设备一定使用 `NetViewX.OCX` 或 `NetClient.dll`。这两个文件名曾共同出现在**朗驰（Launch）早期网络摄像机/视频服务器**的资料中，不能据此推断佰霖产品也采用相同 SDK。目前也**没有找到可核实的官方公开独立下载链接**。
>
> 本指南会明确区分：**已证实的事实**、**适用于特定型号的条件性步骤**与**需要向厂商索取的资料**。不会编造 DLL 函数名、COM 类 ID、默认 IP、默认密码、红外测温公式或虚假的下载链接。

> **重点新增：测量物体表面温度。** 请优先阅读下文 **§12.4–12.11**：专门介绍 `NetViewX.OCX` + `NetClient.dll` 的配合、真实测温接口检查、物体 ROI 选择、预览坐标到温度阵列坐标转换、可运行的测温统计程序以及物理标定。**只有设备本身具备辐射测温能力、并且厂商 SDK 明确提供对应温度接口时，DLL 才能返回可信的 °C。**

## 1. 什么是「佰霖仪器」？

- **公司名称：** 河南佰霖仪器设备有限公司
- **英文：** Henan Bailin Instrument Equipment Co., Ltd.
- **网站列出的领域：** 工业在线热成像仪、工业内窥镜、水质/环境监测、工程检测与无损检测等

BL-DT100 的产品介绍涉及温度监测、火灾风险预警等应用。但公开产品页**没有明确公开**该设备使用 `NetViewX.OCX`、`NetClient.dll`、RTSP、ONVIF 或哪一种逐像素测温 SDK。**请以具体型号与固件说明书为准。**

同一家仪器供应商可能销售网口热成像仪、USB 检测仪、内窥镜或其他品牌的设备；它们**不一定使用同一个软件安装包**。

## 2. 您说的「获取红外线」是哪一种需求？

| 目标 | 硬件条件 | 正常可得到的数据 |
|---|---|---|
| **近红外夜视画面** | 普通摄像机具有近红外感光能力，通常配有红外补光 LED | 夜间黑白视频；**不能测温** |
| **热成像画面** | 具有热红外传感器的热像仪 | 伪彩色或灰度的热成像视频/图片 |
| **某点或某区域的真实温度** | 支持测温并完成标定的热像仪 | 厂商 SDK/元数据提供的 °C 等温度数据 |
| **逐像素温度矩阵/原始红外数据** | 支持辐射测温数据导出的设备与固件 | 温度阵列、辐射数据或原始传感器计数 |
| **开启/关闭红外补光** | 带有可控红外 LED、ICR 日夜模式的监控摄像机 | 厂商定义的开关或自动日夜切换 |

**关键区别：**

1. **红外补光夜视 ≠ 热成像。** 普通监控摄像机使用红外 LED 看清黑暗环境，并不因此具有温度测量能力。
2. **热成像彩色视频 ≠ 真实温度矩阵。** RTSP、H.264/H.265 等视频可能只包含已着色像素，不能直接换算成准确摄氏度。
3. **显示温度数字 ≠ 导出温度矩阵。** 有些设备只会在视频中叠加最高温、最低温或报警信息，未必开放逐像素数据接口。
4. **红外 SDK 必须和设备型号及固件匹配。** 单独两个文件不代表完整驱动或完整测温 SDK。

## 3. NetViewX.OCX 与 NetClient.dll 分别是什么？

| 文件 | 在历史摄像机软件中的典型作用 | 不应该作出的推断 |
|---|---|---|
| `NetViewX.OCX` | Windows **ActiveX/COM 视频播放控件**；历史上供网页实时预览 | 不代表能输出温度矩阵，也不代表兼容佰霖 |
| `NetClient.dll` | 视频客户端、网络连接或解码相关的**原生 DLL**；具体接口需要厂商文档 | 不能仅凭文件名推断函数、参数或红外控制命令 |
| `NetViewX.cab` | 老式 ActiveX **CAB 安装分发包**，可能包含 OCX 及关联 DLL | 不是各品牌、各固件通用的安装包 |

**证据边界：** 早期朗驰网络摄像机手册同时提到了 `NetClient.dll`、`NetViewX.cab`、`NetViewX.ocx`，并说明某些型号可以从自身网页重新下载播放控件。这说明其历史用途偏向**网络视频客户端/播放**，**不能证明**这些组件是佰霖热像仪的官方测温接口。

**安全提示：** OCX/DLL 属于原生可执行代码。下载来源不明的文件或以管理员权限注册它们，存在恶意软件、版本冲突等风险。不建议使用第三方「万能 DLL 下载」「缺失 DLL 修复」网站。

## 4. 下载 NetViewX.OCX 与 NetClient.dll

### 4.1 当前可验证的下载状态

| 文件 | 是否找到可信的官方独立直链 | 推荐获取方式 |
|---|---|---|
| `NetViewX.OCX` | **没有找到** | 向**实际设备生产厂商/OEM**索取匹配版本的签名安装包或 CAB |
| `NetClient.dll` | **没有找到** | 向设备厂商索取**完整客户端或 SDK 套件**，包括依赖库 |
| `NetViewX.cab` | 没有可确认的通用官方下载地址 | **只有设备说明书确认提供**时，才通过已授权设备的网页下载 |

以下是**可核实的咨询或资料入口，不是 DLL 直接下载链接**：

### 4.2 正确的下载操作

1. 查看设备机身标签，记录**品牌、完整型号、硬件版本和固件版本**。公开传图时可遮住序列号。
2. 联系**设备真正的制造商/授权经销商**，确认是否使用这两种文件。
3. 请求**完整、与固件一致**的 Windows 客户端、CAB/OCX、DLL、依赖项和 SDK；优先获取**数字签名安装程序**。
4. 如果厂商明确表示「设备自身网页提供插件」，请在**有授权的隔离局域网**中访问设备网页，使用其实际提供的插件下载入口。
5. **不要猜测** 之类的路径；不同型号/固件不一定存在。
6. 校验文件来源、版本、SHA-256、数字签名，并保留原始压缩包。不要把别的型号的 DLL 混进去。

### 4.3 需要向厂家索取的资料

- 与您**确切型号/固件**对应的 Windows 安装包（x86/x64）。
- `NetViewX.OCX`、`NetClient.dll`、`NetViewX.cab` 及全部关联依赖。
- 厂商提供的 SHA-256、签名、发行说明及许可条件。
- OCX 的 **CLSID、ProgID、类型库（`.tlb`）、COM 接口说明和事件**。
- DLL 的 **C/C++ 头文件（`.h`）、导出函数说明、参数类型、调用约定、结构体定义与示例程序**。
- 是否支持 **RTSP、ONVIF、USB/UVC**；如果支持，提供真正的热成像通道 URL、端口、编码。
- 是否支持**逐像素测温、温度元数据、16 位原始数据或温度矩阵**。
- 温度单位、标定信息、发射率设置、精度、报警接口、离线使用要求。

## 5. 先选择正确的技术方案

| 您的目标 | 优先方案 | 是否需要旧 OCX/DLL |
|---|---|---|
| 只看热成像视频 | 厂商客户端，或经厂商确认的 **RTSP/ONVIF + VLC/OpenCV** | **不一定需要** |
| 读取真实温度、最高温、温度矩阵 | **设备专属辐射测温 SDK**、元数据接口或标准导出 | 仅当厂商文档明确规定时才需要 |
| 使用老网页观看视频 | **匹配型号的 ActiveX + Edge IE 模式/授权 COM 宿主** | 可能需要 |
| 自己写 C#、C++、Python 调用 NetClient | **真实 SDK 头文件 + 正确架构 + 示例程序** | 取决于设备 |
| 切换 IR LED/日夜模式 | 厂商客户端或公开的设备控制接口 | 通常不依赖这两个文件 |

**建议：** 如果只是需要读取热成像视频，先检查 RTSP/ONVIF 是否支持；如果需要可信温度数据，重点是**热像仪是否支持 radiometric / 辐射测温数据导出**。二者是不同的问题。

## 6. 佰霖热成像设备：从开箱到可用的一般安装流程

> 注意：下面是适用于工业热像仪的**通用安全部署流程**，不是未经验证的 BL-DT100 菜单逐项说明。

1. **确认硬件。** 检查型号、固件、供电标签、支持的接口、安装手册及产品许可。
2. **正确供电。** 严格依照厂家电压、极性、PoE 类型和接线图。**不要猜 DC 电压、接口针脚。**
3. **安装设备。** 固定在允许的温度/湿度环境中，保证可视范围、镜头清洁、测量距离合理。
4. **连接数据线。** 视型号使用以太网、USB、串口或配套 NVR/控制器。
5. **网络设置。** 先使用厂商手册或官方发现工具确认摄像机 IP，再配置 PC 到同一授权网段。**本文不提供未经证实的默认 IP/密码。**
6. **先用厂家客户端检查。** 确认设备在线、可正常登录、可看到可见光和/或热成像通道。
7. **如支持测温：** 根据手册配置**发射率、反射环境温度、距离、测温区域 ROI、量程及校准信息**。
8. **如支持报警：** 设置温度阈值、触发区域、延迟/滞回、报警联动，用已知安全目标验证。
9. **导出样本验证。** 保存普通图片、热成像图片、可能存在的温度 CSV/温度阵列、短视频，核对时间、单位、分辨率。
10. **建立记录。** 留存软件版本、固件、SDK、标定/参数、测试目标、测量误差。

**不要将未验证的图像温度或报警阈值直接用于生命安全/消防保护。**

## 7. Windows 11 安装前的环境准备

### 7.1 建议环境

| 项目 | 建议 |
|---|---|
| PC | Windows 11 x64；优先使用**隔离测试虚拟机/实验 PC** |
| 操作权限 | 仅在厂商确认时，使用管理员权限安装/注册受信任 OCX |
| 网络 | 与仪器通信的**隔离或授权 LAN**，不要直接暴露老旧 ActiveX 设备到公网 |
| 开发工具（可选） | Visual Studio / Windows SDK；Python 3.12（只做 RTSP 时） |
| 查看视频（可选） | VLC、FFmpeg/ffplay、OpenCV，前提是设备提供可用视频流 |
| COM/原生分析 | Windows SDK 的 `dumpbin`、OLE/COM Object Viewer（`oleview.exe`） |
| 官方程序 | 厂商对应的播放器、SDK、驱动与实际依赖库 |

**为什么用隔离环境？** ActiveX 是较旧的原生插件架构，且用户无法仅凭文件名识别其安全性。不要在主要工作电脑上安装来历不明的插件，也不要关闭整个系统的 SmartScreen/浏览器插件安全措施。

### 7.2 先取得厂商可信文件

示例文件夹（只是规划目录，不表示这些文件已经存在）：

```text
C:\BailinTest\Vendor\
  NetViewX.ocx
  NetClient.dll
  NetViewX.cab        # 仅在厂家提供时存在
  vendor-setup.exe    # 仅在厂家提供时存在
  additional-deps\
```

### 7.3 PowerShell：只读检查文件、签名、哈希

```powershell
$ocx = "C:\BailinTest\Vendor\NetViewX.ocx"
$dll = "C:\BailinTest\Vendor\NetClient.dll"

Test-Path $ocx
Test-Path $dll
Get-Item $ocx, $dll | Select-Object Name, Length, VersionInfo
Get-AuthenticodeSignature -FilePath $ocx
Get-AuthenticodeSignature -FilePath $dll
Get-FileHash -Algorithm SHA256 -Path $ocx
Get-FileHash -Algorithm SHA256 -Path $dll
```

请将 SHA-256 与**厂家公布的值**比较。**只有哈希值、但没有可信下载源，并不能保证文件安全。** 对签名缺失或异常应向厂家核实，而不是禁用系统安全校验。

### 7.4 查看 32/64 位及依赖项

在 Visual Studio 的 **Developer Command Prompt** 中：

```bat
dumpbin /headers "C:\BailinTest\Vendor\NetViewX.ocx"
dumpbin /headers "C:\BailinTest\Vendor\NetClient.dll"
dumpbin /dependents "C:\BailinTest\Vendor\NetViewX.ocx"
dumpbin /exports "C:\BailinTest\Vendor\NetClient.dll"
```

- `machine (x86)`：32 位文件；需要 32 位宿主程序。
- `machine (x64)`：64 位文件；需要 64 位宿主程序。
- `/dependents`：帮助查找缺失的依赖库。
- `/exports`：可以列出导出符号，但**看不出完整参数类型或函数语义**；不能以此猜测调用方法。

### 7.5 如果厂商提供了 NetViewX.cab

只进行**离线展开和检查**：

```powershell
New-Item -ItemType Directory -Force "C:\BailinTest\Extracted"
expand.exe -F:* "C:\BailinTest\Vendor\NetViewX.cab" "C:\BailinTest\Extracted"
```

检查其中的 `.inf`、OCX/DLL 版本、签名和依赖。若厂家提供**有数字签名的完整安装程序**，通常优先按厂家步骤使用安装程序，手工复制 OCX 可能漏掉必要配置。

## 8. 在 Windows 11 注册 NetViewX.OCX（仅限可信旧控件）

**前提：** 已确认文件来自真正的设备厂商、版本适配、它是**可自行注册的 COM/ActiveX 控件**；推荐在隔离测试 VM 中操作。注册/加载 OCX 会执行原生代码。

**64 位 Windows 安装 32 位 OCX：** 管理员命令提示符中执行：

```bat
C:\Windows\SysWOW64\regsvr32.exe "C:\BailinTest\Vendor\NetViewX.ocx"
```

**64 位 Windows 安装真正的 64 位 OCX：**

```bat
C:\Windows\System32\regsvr32.exe "C:\BailinTest\Vendor\NetViewX.ocx"
```

**Windows 的命名容易误解：** 在 x64 Windows 中，`SysWOW64` 实际包含 **32 位**系统程序，而 `System32` 包含 **64 位**程序。

**不要直接对 `NetClient.dll` 执行 `regsvr32`。** 它可能只是普通原生 DLL，没有 `DllRegisterServer`；必须以厂商安装说明为准。

遇到「模块无法加载」，检查文件路径、32/64 位和依赖 DLL；遇到「没有 DllRegisterServer」，可能该文件不是可注册 COM 控件，需要寻找正确安装程序。
## 9. 浏览器能运行 NetViewX.OCX 吗？

- **当前浏览器：** 不支持传统 ActiveX。在正常网页中仅用 HTML/JavaScript **无法直接加载 OCX**。
- **传统 IE 兼容宿主：** 部分受管理的 Windows 环境仍允许把选定的内网站点交给 IE 兼容宿主，以运行某些老旧 **ActiveX 控件**。站点须由组织策略加入。该宿主**不保证特定 OCX 一定可运行。**
- **Windows 桌面 COM 宿主程序：** 如果 OCX 的架构、注册及 COM 接口正确，也可能由合适的 Windows 程序承载；具体以厂商 SDK 为准。

**测试流程：**

1. 更新 Windows，并确认需要访问的是**可信授权局域网**中的设备网页。
2. 若组织仍提供传统 IE 兼容宿主，仅按批准的站点列表或内部流程加入该具体内部站点。
3. 使用厂商提供的**已验证兼容版本**安装/注册控件。
4. 在该兼容宿主中打开该设备的真实管理网页，按照厂家操作手册查看热成像或视频通道。
5. 如果预览失败，先排查位数、控件依赖、签名、权限、固件，而**不是**全局关闭浏览器安全防护。

**不要凭空编写 ActiveX HTML 的 `<object>` 类 ID。** 必须使用已知的真实 **CLSID/ProgID**，以及厂商公布的方法和属性。可使用微软 Windows SDK 的 **OLE/COM Object Viewer（`oleview.exe`）** 检查可信 OCX 的类型库。

## 10. 如何用 NetViewX.OCX 或 NetClient.dll 自己编程？

### 10.1 如果 NetViewX.OCX 是 COM/ActiveX 控件

1. 向厂家获取 OCX 的 **CLSID/ProgID**、类型库和 API 文档。
2. 在 Windows SDK 的 OLE/COM Object Viewer 中检查 COM **接口、方法、参数、事件、属性**。
3. 如果用 C# WinForms，可按厂商示例将 COM 组件添加为引用/控件。
4. **32 位 OCX 必须由兼容的 32 位进程承载**；不要误设为 Any CPU 后用 64 位进程加载。
5. 按厂商文档完成初始化 → 登录设备 → 选择通道 → 开始预览 → 停止 → 退出/释放资源。
6. 如果提供热成像温度方法，必须按**真实函数定义与测温标定规则**调用。

**当前不能给出一段已确认可运行的 OCX 连接/获取温度代码，原因是具体控件版本的真实 COM 接口未得到验证。** 随意填写 `classid`、`Login()`、`GetTemperature()` 等名称会制造不可运行的示例。

### 10.2 如果 NetClient.dll 提供普通原生 C 接口

1. 向厂家获取对应的 `NetClient.h`、SDK 说明和示例程序。
2. 查清楚每个导出函数的**返回值、参数类型、结构体内存布局、字符串编码、回调方式、调用约定（`cdecl` / `stdcall`）**。
3. 确认应用进程与 DLL 的架构匹配（x86 或 x64）。
4. **C/C++：** 按官方头文件链接相应导入库或受控地加载 DLL。
5. **C#：** 根据官方签名编写准确的 P/Invoke 声明；确认结构体对齐和回调生命周期。
6. **Python：** 仅在已取得官方函数原型之后使用 `ctypes.CDLL` 或 `ctypes.WinDLL`；不要猜测参数。
7. 按厂家给出的生命周期实现：初始化 → 设备登录 → 通道选择 → 视频/温度回调 → 结束预览 → 登出 → SDK 清理。

**不要把「成功加载 NetClient.dll」误认为「已经读取红外温度」。** 它可能只实现网络连接、登录或视频解码。

### 10.3 兼容性与开发语言

| 使用方式 | 必备条件 | 建议 |
|---|---|---|
| C++ 原生程序 | 官方头文件、导入库、架构一致、依赖项完整 | 最贴近老式设备 SDK |
| C# WinForms | 正式 COM 接口或完整 DLL P/Invoke 定义 | 适合 Windows 桌面 UI |
| Python 3.12 调用 DLL | 已验证的函数定义和 ABI；匹配 DLL 位数 | 数据处理可行，旧 SDK 兼容性需单独测试 |
| HTML + JavaScript | **不能仅靠浏览器原生 JS 加载 ActiveX DLL** | 优先使用设备提供的标准视频/Web API |
| Linux / ARM | 必须有厂商提供的 Linux/ARM SDK 或标准网络流 | Windows `.ocx` 无法直接原生运行在 Linux/ARM |

如果最终部署到 Linux/ARM、Zynq 或无 Windows 的设备上，应优先争取 **RTSP、ONVIF 或跨平台的温度/图像 API**。旧的 Windows ActiveX 控件通常不是合适的嵌入式部署方式。

## 11. 如何真正拿到「红外视频」：RTSP/ONVIF 方案

**先决条件：** 您的**准确设备型号**确实支持 RTSP 或 ONVIF，并且厂家给出真实的视频 URL 和通道说明。某些朗驰旧型号的 RTSP URL **不能**当作佰霖通用地址。

### 第一步：从厂家查出视频流信息

请确认：

- 设备真实 IP/域名、RTSP 端口、视频路径及登录方式。
- **热成像通道**与**可见光通道**的区别。
- 使用 H.264/H.265、分辨率、帧率、视频鉴权方式。
- 视频是否只有伪彩色画面，是否包含独立的温度元数据。
- 是否支持官方客户端、VLC、ffplay 或 ONVIF 设备发现。

### 第二步：先用视频播放器测试

在 VLC 的「打开网络串流」中输入**厂家实际提供**的 RTSP URL。正确解码后，确认画面来自热成像通道，而不是普通 RGB 通道。

如果使用 FFmpeg 的 `ffplay` 或 `ffprobe`，同样应使用设备手册明确给出的 URL；不要盲目尝试其他设备的默认路径或密码。

### 第三步：Windows 11 + Python 3.12 安装 OpenCV

打开 PowerShell：

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install opencv-python
$env:CAMERA_RTSP_URL = Read-Host "Enter the real RTSP URL from device vendor"
```

说明：

- `py -3.12` 使用 Windows 中已安装的 Python 3.12。
- `venv` 建立独立环境，避免影响其他 AI/视觉项目。
- `opencv-python` 用于**查看/保存视频帧**，不负责温度标定。
- URL 可能包含鉴权信息；**不要公开截图或提交到 Git 仓库**。

将下列内容保存为 `thermal_video.py`：

```python
import os
import cv2

url = os.environ.get("CAMERA_RTSP_URL")
if not url:
    raise SystemExit("请先设置实际设备的 CAMERA_RTSP_URL。")

cap = cv2.VideoCapture(url, cv2.CAP_FFMPEG)
if not cap.isOpened():
    raise SystemExit("无法打开该 RTSP 视频流；请检查真实 URL、鉴权、网络及编码。")

try:
    saved = False
    for _ in range(150):
        ok, frame = cap.read()
        if not ok:
            break

        if not saved:
            if not cv2.imwrite("camera_frame.png", frame):
                raise RuntimeError("视频帧保存失败。")
            saved = True

        cv2.imshow("Camera video - NOT calibrated temperature", frame)
        if (cv2.waitKey(1) & 0xFF) == ord("q"):
            break

    if not saved:
        print("打开流后没有得到可以解码的视频帧。")
finally:
    cap.release()
    cv2.destroyAllWindows()
```

运行：

```powershell
.\.venv\Scripts\python.exe .\thermal_video.py
```

**结果：** 当实际视频流是热成像通道时，可以弹出热成像视频预览，并保存 `camera_frame.png`。

**重要限制：** 图片是**视频帧**，并不包含可直接信任的逐像素摄氏度。OpenCV 解码需要系统和构建环境支持相应的 FFmpeg 后端、视频协议和鉴权。

## 12. 如何获取真实温度（°C）与红外原始数据？

这一步和获取视频**不一样**，需要支持辐射测温的硬件或相应 SDK。

### 12.1 获取温度的详细流程

1. **确认设备是否支持辐射测温（radiometric）。** 并非所有热成像仪都允许导出每个像素的温度。
2. 取得厂商的**测温 SDK、温度回调、私有数据协议、温度文件导出说明**。
3. 确认返回值到底是**摄氏度、开尔文、百分之一开尔文、原始探测器计数、16 位图像还是已标定温度**。
4. 确认二维阵列的**宽、高、每行字节数、字节序、每帧时间戳**。
5. 获取所需的测温参数与标定资料，包括**发射率、环境/反射温度、目标距离、空气透过率、传感器校正**。
6. 按厂商文档调用：设备登录 → 启动数据流 → 获取红外图像及**同步的温度数据** → 合法转换单位 → 释放资源。
7. 使用合适的温度基准和厂家精度指标，验证至少数个测量点与不同环境条件。
8. 验证后再存入 CSV、数据库、工业告警系统。

**绝对不要直接认为 `原始像素值 / 100 = 摄氏度`。** 不同厂家、型号与固件使用的编码、校准及温度偏移各不相同。

### 12.2 您的软件可设计成这种结果格式

下面是**您自己应用的 JSON 结构设计示例**，不是宣称 BL-DT100 SDK 返回此 JSON：

```json
{
  "device_model": "FROM_DEVICE",
  "captured_at_utc": "YYYY-MM-DDTHH:MM:SSZ",
  "thermal_width": 256,
  "thermal_height": 192,
  "temperature_unit": "C",
  "pixel_temperature_array": "ONLY_IF_VENDOR_SUPPORTS_RADIOMETRIC_EXPORT",
  "emissivity": "FROM_MEASUREMENT_CONFIGURATION"
}
```

其中 `256 × 192` 是纯粹的**格式示例**，**并非** BL-DT100 的已核实分辨率。

### 12.3 如果厂商只提供视频但没有温度 SDK

- 您仍然可以用 OpenCV/FFmpeg 做**图像处理、目标检测、运动分析和录像**。
- 伪彩色值受**色板、自动增益控制（AGC）、动态温度范围**等影响；不能可靠反推出温度。
- 如果需要监控真实过热温度，请选择**支持温度 API 的热像仪型号**，或询问厂家是否支持温度叠加字段/报警事件的标准数据导出。

### 12.4 必须先确认：NetClient.dll 是否真的支持测温？

**目标：** 在 `NetViewX.OCX` 视频中选择某个物体，然后用 `NetClient.dll` 读取这个物体**经过标定的表面温度（°C）**。**不能**把伪彩色视频的 RGB 颜色当作真实温度。

| 设备与 SDK 实际能力 | 能不能得到 °C？ | 两个文件的职责 |
|---|---|---|
| **辐射测温型热像仪** + NetClient 文档提供温度矩阵/点温/区域测温 API | **可以**，但需要正确标定和参数 | OCX 预览；DLL 返回温度 |
| 辐射测温型热像仪 + DLL 只支持点温/区域温度查询 | **可以**读取文档允许的点或区域 | OCX 选目标；DLL 接受传感器坐标并测温 |
| 只有热成像外观的视频，而 DLL 只提供视频连接/解码 | **没有可核实的温度数值接口** | 可以看视频；无法据此得到准确 °C |
| 普通摄像机的近红外补光夜视 | **不可以** | 设备硬件本身不能测量物体热辐射温度 |

**证据边界：** 出现这两个文件名的历史摄像机手册描述的是**网络视频浏览/播放**，不是逐像素测温。针对您的具体 DLL 版本，目前没有可核实的公开测温函数原型。因此不能凭空假设存在 `GetTemp()`、`GetObjectTemperature()` 等方法。

### 12.5 两个文件一起使用：完整实施顺序

```text
真实具备辐射测温能力的热像仪 / 已确认的热成像通道
  ├─ 视频 → NetViewX.OCX → 用户选中物体区域（屏幕坐标 ROI）
  │                           └─ 映射到真实热传感器坐标
  └─ 标定数据 → NetClient.dll（前提：官方 SDK 确认提供测温接口）
                              └─ 点测温 / 区域统计 / 温度矩阵
       校验摄像机、通道、同一时间的帧及正确 ROI
                        ↓
       物体表面温度：最低 / 平均 / 最高 / P95（摄氏度）
                        ↓
       屏幕显示、CSV/数据库记录、可选超温报警
```

1. **确认摄像机能测真实温度。** 使用厂家原始客户端，对准已知安全温度的物体，检查是否有真正的 **°C 数值**。画面是彩色并不足以证明具备测温能力。
2. **获取同一套厂商 SDK。** 包括真实 `NetClient.h`、示例、`NetViewX.OCX` 类型库/CLSID、全部依赖项；版本必须与固件匹配。
3. **安装 Windows 组件。** 参考第 7–10 节，确认 OCX、DLL 和宿主程序为匹配的 x86/x64 架构；推荐隔离的测试环境。
4. **用 NetViewX.OCX 打开正确通道。** 必须确认正在显示**热成像通道**，不是双光谱设备的普通可见光通道。
5. **根据 NetClient.dll 的真实说明初始化、登录和启动数据。** 这可能需要独立于 OCX 的设备会话，必须使用厂家示例，不要猜测接口参数。
6. **启动厂商定义的测温数据流/查询。** 看返回的是**测温点、区域最小/最大/平均、已经换算成 °C 的数组，还是需要正式标定公式的原始探测器数据**。保存单位、热像分辨率、有效性状态、无效像素标识、时间戳与量程。
7. **用户选择具体物体。** 在 OCX 窗口画框、点击测点，或由目标检测算法得到物体掩码。按第 12.6 节转换到真实热传感器坐标。
8. **执行查询并对齐。** 如 SDK 有区域 API，就传入真实热传感器区域；如果有温度矩阵，就读取**同一帧**该物体区域的像素值。
9. **检查测量可信度。** 结合第 12.9 节发射率、反射、焦点、标定、背景污染、时间同步等规则，最后显示 **°C 与测量来源**。
10. **安全退出。** 按厂商顺序停止温度回调、停止 OCX 预览、登出、释放 DLL 与 COM 资源。

**硬性停止条件：** 如果第 6 步确认没有官方温度接口，那么**只用这两个文件不能测量物体温度**。需要额外辐射测温 SDK、开放了测温能力的固件，或者更换为支持温度输出的热像仪。

### 12.6 最关键的坐标换算：OCX 预览像素 ≠ 温度矩阵像素

用户在 `NetViewX.OCX` 视频窗口点击的坐标与真正热传感器像素坐标，可能属于不同分辨率。

以下公式**仅适用于没有裁剪、旋转、缩放平移，并且画面按比例完整显示（两侧可能有黑边）**的真实热成像图：

- `Vw,Vh`：OCX 的**实际视频画面区域**宽高，不含窗口标题和工具栏。
- `Tw,Th`：SDK 提供的**已标定温度矩阵**原始宽高。
- `vx,vy`：相对视频区域左上角的点击坐标。
- `scale = min(Vw/Tw, Vh/Th)`。
- `left = (Vw−Tw×scale)/2`，`top = (Vh−Th×scale)/2`。
- `tx = floor((vx−left)/scale)`，`ty = floor((vy−top)/scale)`。

要求 `0 ≤ tx < Tw`、`0 ≤ ty < Th`。黑边内的点击必须拒绝。对于矩形物体，转换两个角，使用右下坐标**不包含端点**的 `[x1,y1,x2,y2)` 表示方法。

**计算实例（只是示例，不是佰霖产品规格）：**

- OCX 显示区域为 `1280×720`，测温矩阵为 `320×240`。
- `scale = min(4,3) = 3`，实际显示热像是 `960×720`；两侧各有 `160` 像素黑边。
- 在视频中点击 `(640,360)` → 对应热传感器像素 **`(160,120)`**。
- 点击 `(50,360)` → 落在左侧黑边，**不能返回物体温度**。

**特殊情况必须校准：** 数码变焦、缩放、裁剪、平移、旋转、画中画、可见光/热成像融合显示都会改变几何关系。必须使用完整的**反向显示变换**或厂商的**可见光到热图像配准矩阵**，不能把普通 RGB 坐标直接乘一个比例。

如果以后接入 YOLO26-OBB/目标检测：优先把**旋转多边形或目标分割掩码**映射到热温度网格；单纯矩形框可能包含高温背景，导致「物体最高温」其实来自背景。

### 12.7 找到真正的 NetClient.dll 测温接口

只针对**厂商可信、版本匹配**的文件，在 Visual Studio / Windows SDK 开发命令提示符运行：

```bat
dumpbin /exports "C:\BailinTest\Vendor\NetClient.dll"
dumpbin /headers "C:\BailinTest\Vendor\NetClient.dll"
dumpbin /dependents "C:\BailinTest\Vendor\NetClient.dll"
```

使用 Windows SDK 的 `oleview.exe` 检查 `NetViewX.OCX` 的真实 **CLSID、ProgID、类型库、视频事件和可调用方法**。

向厂家准确确认以下接口：

- **温度矩阵：** 数组宽度、高度、行跨度、温度单位、数据类型、标定方式、无效数据、时间戳、回调机制。
- **点测温：** 指定热传感器 `(x,y)` 后返回对应表面温度（°C）。
- **区域测温：** 指定 `(x1,y1,x2,y2)` 或多边形后，返回最大、最小、平均等统计值。
- **视频配准：** OCX 视频帧与 NetClient 温度帧怎样匹配通道 ID、帧 ID 或时间戳。
- **原生 DLL ABI：** `NetClient.h` 中的函数原型、`cdecl`/`stdcall`、结构体对齐、回调线程、指针内存归属、错误码。

**请注意：** `dumpbin` 只能看到导出符号，**看不到完整函数参数与其真实意义**。没有厂商 C 头文件，不能可靠地编写 Python `ctypes`、C# P/Invoke 或 C++ 调用。

以下只是**实施调用顺序，不是实际 DLL 中的函数名**：

```text
OCX： [按厂商 CLSID 创建 COM 视频控件；预览热成像通道]
DLL： [按 NetClient.h 初始化 SDK；登录热像仪；开启测温数据]
OCX： [取得物体所选位置/框与预览帧时间]
程序：[根据真实几何映射出热传感器坐标]
DLL： [按厂商定义获取同一帧温度矩阵或查询点/区域 °C]
程序：[检查有效像素和时间同步；计算物体最低/平均/最高温]
界面：[显示结果、单位、时间戳、测量参数与来源]
清理：[结束回调、退出登录、释放 DLL 和 COM 资源]
```

### 12.8 可直接运行的 Python 3.12 物体温度统计代码

下面程序只需要 **Python 3.12 标准库**，**不需要额外安装包**。它计算**厂商 NetClient.dll 测温 SDK 已经提供的标定温度数据**，本身**不会加载 DLL，不会从 OCX 彩色视频推导温度，也不代表 NetClient.dll 真的会返回 JSON**。

请让您的**厂商 SDK 适配程序**把真实、经过标定的数据变成如下**自己定义的 JSON 格式**。这里的数值纯属演示，不是某台仪器的实测结果：

```json
{
  "source": "VENDOR_CALIBRATED_NETCLIENT_SDK",
  "timestamp_utc": "2026-10-26T15:00:00Z",
  "width": 4,
  "height": 3,
  "unit": "C",
  "temperature_c": [27.0, 28.0, 29.0, 26.0,
                    27.5, 34.0, 36.0, 26.5,
                    28.0, 33.0, 35.0, 27.0],
  "roi_xyxy": [1, 1, 3, 3],
  "emissivity": 0.95
}
```

把以下代码保存为 `object_temperature.py`：

```python
import json
import math
import statistics
import sys


def measure(frame):
    if frame.get("source") != "VENDOR_CALIBRATED_NETCLIENT_SDK":
        raise ValueError("No verified calibrated SDK temperature source.")
    if frame.get("unit") != "C":
        raise ValueError("Expected vendor-converted degrees Celsius.")

    w, h = int(frame["width"]), int(frame["height"])
    temperatures_c = frame["temperature_c"]
    if w <= 0 or h <= 0 or len(temperatures_c) != w * h:
        raise ValueError("Bad calibrated thermal array dimensions.")
    x1, y1, x2, y2 = map(int, frame["roi_xyxy"])
    if not (0 <= x1 < x2 <= w and 0 <= y1 < y2 <= h):
        raise ValueError("ROI is outside the thermal array.")

    mask = frame.get("mask")  # Optional w*h object segmentation mask
    if mask is not None and len(mask) != w * h:
        raise ValueError("Invalid ROI mask length.")

    samples, selected = [], 0
    for y in range(y1, y2):
        for x in range(x1, x2):
            i = y * w + x
            if mask is not None and not mask[i]:
                continue
            selected += 1
            val = temperatures_c[i]
            if val is None:  # OEM invalid pixel must be normalized to None
                continue
            t = float(val)
            if math.isfinite(t) and t > -273.15:
                samples.append(t)

    if not selected or len(samples) / selected < 0.90:
        raise ValueError("Insufficient valid target pixels.")
    ordered = sorted(samples)
    p95 = ordered[math.ceil(len(ordered) * 0.95) - 1]
    return {
        "min_c": ordered[0],
        "mean_c": statistics.fmean(ordered),
        "max_c": ordered[-1],
        "p95_c": p95,
        "valid_pixels": len(ordered),
        "roi_xyxy": [x1, y1, x2, y2],
        "timestamp_utc": frame.get("timestamp_utc"),
        "emissivity": frame.get("emissivity")
    }


if __name__ == "__main__":
    if len(sys.argv) != 2:
        raise SystemExit("Usage: py -3.12 object_temperature.py calibrated_frame.json")
    with open(sys.argv[1], encoding="utf-8") as f:
        print(json.dumps(measure(json.load(f)), indent=2))
```

在 Windows 11 PowerShell 中运行（**先保证真实厂商 SDK 已生成文件**）：

```powershell
py -3.12 .\object_temperature.py .\calibrated_frame.json
```

示例 ROI `[1,1,3,3]` 包含 `34、36、33、35` °C，程序计算得到：**最低温 33°C、平均温 34.5°C、最高温 36°C**。代码中「90% 有效像素」是**程序质量规则示例**，不是厂家精度指标。

如果厂商 DLL **只返回某点温度/某区域温度**而不是完整矩阵，请直接存储真实 SDK 的温度结果，**不要**虚构一个逐像素数组。

### 12.9 物体表面温度怎么测才准确？

热像仪通常测量的是物体**外表面**的辐射对应温度，不是物体内部、体核或者一定正确的实际温度。

1. **尺寸与光学分辨率：** 物体必须占据足够的真实热传感器像素，符合厂商的测量距离/点尺寸要求；小目标和边缘混合像素容易失真。
2. **发射率 Emissivity（`ε`）：** 按实际材质、粗糙度和涂层选择。光亮金属反射严重；**不能统一设为 0.95**。
3. **反射表观温度、环境温度、测量距离、湿度/透过率：** 按厂商温度模型设定。室温和反射表观温度不一定相同。
4. **镜头对焦、热稳定和自动非均匀性校正：** 按说明书进行预热、对焦、NUC/FFC，确认处于标定的测温量程。
5. **遮挡：** 普通玻璃及许多塑料在相关长波红外波段并不透明；镜头可能测到的是玻璃而非后面的物体。
6. **目标掩码：** 从物体内部取有效像素，区分最高/最低/平均/P95；剔除背景、遮挡或无效像素。
7. **独立验证：** 在安全条件下用合适的接触式探头或校准黑体做对照，结合具体设备误差指标；不要为了改善发射率而接触、涂覆危险的带电/高温物体。
8. **帧同步：** 在时间 `t1` 选中的物体不能随意套用时间 `t2` 的温度数据。必须核对摄像机、通道、时间戳/帧 ID。
9. **失败应明确提示：** 标定、接口、ROI 映射、同步或温度数据缺失时显示 **「温度不可用」**，**不能**返回根据画面颜色猜出来的数字。

### 12.10 真正交付前必须通过的 5 项测试

| 验证内容 | 通过标准 |
|---|---|
| **A. 硬件** | 设备在安全的已知冷热参考目标上提供真实数值；普通近红外 CCTV 不合格 |
| **B. NetClient.dll** | 官方 SDK 文档和例程确认能返回经标定的温度/单位/时间/错误状态，而不只是视频 |
| **C. NetViewX.OCX 对齐** | OCX 在画面中央及边缘点击的物体，与 DLL 查询的真实热传感器区域一致 |
| **D. 物理复现** | 在控制好的条件下多次测量，与**该型号官方精度指标**一致 |
| **E. 异常处理** | 缺少温度 API、ROI 超出范围、时间不匹配或测量无效时提示「不可用」，绝不误报 |

### 12.11 向厂家索取哪些资料，才能编写真正可运行的 DLL+OCX 程序？

请提供：**`NetClient.h`、测温 API 示例、`NetViewX.OCX` 类型库/CLSID、设备准确型号、固件及对应 SDK 版本**。明确要求：

- 点测温/区域测温/温度矩阵**真实函数原型**、参数、结构体和回调类型。
- 每个像素的温度单位、原始数据标定公式/转换 API、无效/过量程标记。
- 热像通道 ID、传感器分辨率、OCX 画面与热像数据的映射，以及时间同步。
- 发射率/反射/距离参数、测量精度、Windows x86/x64 兼容性、依赖和许可条件。

**如果厂家确认这份 NetClient.dll 只有网络视频功能而没有测温函数，那么只靠这两个文件无法测物体真实温度。** 正确方案是取得专用辐射测温 SDK，或更换具备温度数据输出能力的热像仪，而不是编造函数。

## 13. 如果您要的是「打开红外灯 / 红外夜视」

这个问题主要针对**近红外安防摄像机**，不是长波热像仪：

1. 检查型号是否配有红外 LED、红外截止滤镜（IR-cut）和日夜模式。
2. 通过**厂家应用**寻找「日夜切换 / 自动 / 夜间 / 红外补光 / IR LED」等选项。
3. 在低照度但安全的环境中测试自动切换。
4. 如果需要远程控制，向厂家索取对应的 HTTP/ONVIF/SDK 控制接口及参数。
5. 依据**您具体型号**的 API 执行，不要从 `NetViewX.OCX` 或 `NetClient.dll` 文件名凭空猜测命令。

**红外灯开启不等于温度测量。**

## 14. 常见问题与处理方法

| 问题 | 常见原因 | 正确处理 |
|---|---|---|
| 找不到 NetViewX.OCX / NetClient.dll 官方下载 | 老式专有组件、版本与固件绑定 | 联系设备原厂家，索取完整匹配套件 |
| `regsvr32` 报「无法加载模块」 | 架构不一致、缺少依赖、文件损坏 | 用 `dumpbin /dependents` 检查 DLL 链和 x86/x64 |
| `DllRegisterServer` 不存在 | DLL 不属于可自行注册的 COM 服务器 | **不要**反复对 `NetClient.dll` 注册，使用厂商安装程序 |
| `0x80040154 Class not registered` | CLSID 不对、未注册、32/64 位宿主不匹配 | 核对厂家真实 CLSID、注册方式和位数 |
| Browser 预览黑屏/空白 | Browser 无 ActiveX 支持 | 使用厂商现代视频接口，或经授权配置 Edge IE 模式 |
| IE 模式阻止未签名控件 | 安全签名/发行者校验失败 | 向厂商申请可信签名安装包，不关闭系统保护 |
| 能看视频却读不到温度 | 流可能只包含伪彩色视频 | 获取热成像温度 SDK 或辐射数据接口 |
| 画面有红/黄颜色，但温度不准 | 视频使用色板和 AGC | **不要**由 RGB 颜色换算摄氏度 |
| 连不上摄像机 | IP、网段、真实 URL、凭据、端口或固件差异 | 按型号手册及授权网络配置排查 |
| Python 调用 DLL 崩溃 | `ctypes` 类型、调用约定、结构体或架构错误 | 使用正式 C 头文件和厂商示例，停止猜测接口 |
| 测得的温度明显偏差 | 发射率、环境反射、距离、标定不正确 | 按设备测温规范重新配置并做实际标定测试 |

## 15. 如何让我继续给您写「完全可运行」的代码？

请提供以下信息（可隐藏私密信息）：

1. **设备机身型号照片**，例如具体产品名和型号，而不仅仅是「佰霖仪器」。
2. **固件版本**，设备连接方式：以太网、USB、串口、配套 NVR 等。
3. 您需要哪种结果：**夜视视频 / 热成像视频 / 某点温度 / 每像素温度矩阵 / 原始红外数据 / 打开红外灯**？
4. 是在哪里看到 `NetViewX.OCX`、`NetClient.dll` 的？（安装包、软件目录、设备网页、文档截图）
5. 这两个文件是否已经在您电脑上？有无 SDK 压缩包、`NetClient.h`、`.tlb` 或例程？
6. 想用 **Python 3.12、C#、C++、HTML/JS** 的哪一种开发？最终运行在 Windows 11 还是 Linux/ARM/Zynq？

拿到**真实 SDK 头文件/控件类型库**后，才能准确编写匹配版本的登录、红外视频读取、温度读取以及资源清理代码。

### 可直接发给佰霖或设备厂家的一段咨询信息

> 您好！我需要在 Windows 11 环境下采集贵司设备的红外热成像图像和真实温度数据。请确认我提供的设备型号与固件是否使用 NetViewX.OCX、NetClient.dll、NetViewX.cab。如果使用，烦请提供官方可信安装包、完整依赖、x86/x64 说明、SDK 头文件、COM CLSID/ProgID、接口文档、函数示例和数字签名/校验值。另请明确是否支持 RTSP/ONVIF 热成像视频、温度叠加信息、逐像素温度矩阵/16 位原始数据、测温标定和离线运行。谢谢！

## 16. 参考资料与官方链接

资料核查日期：2026 年 10 月 26 日。

## 最终建议

1. **先确认确切的设备型号。** 佰霖产品页无法证明它使用 NetViewX/NetClient。
2. **视频与真实温度要分开处理。** 普通网络视频容易集成；真实温度需要厂商开放并校准的数据接口。
3. **旧 ActiveX/DLL 只能使用可信、型号匹配的版本。** 目前没有核实到安全官方直链，不应使用不明下载站。
4. **如果您打算移植到 Linux/ARM/Zynq，尽量优先采用厂商支持的 RTSP、ONVIF 或跨平台测温 SDK。**

