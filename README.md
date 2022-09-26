### 安装 Windows 11，无法加入网络
官方 Windows 安装镜像缺少无线网卡驱动，可通过 `Shift-F10` 打开命令行使用 `OOBE\BYPASSNRO` 强制跳过联网步骤，或者插个 USB 网卡。

### 安装 Windows 11，强制要求登录微软账号
如果希望使用本地账户，可以使用 `test@test.com` 或 `a@b.com` 密码任意，直接进入下一步创建本地账户。

### Lenovo Commercial Vantage 安装好以后不显示设备信息
不要去官网下载，直接安装微软商店的 Commercial Vantage 即可，也无需先安装普通版 Vantage。

### 是不是需要去官网下载驱动
不需要，Windows Update 和 Vantage 就已足够。

### 系统更新后 dTPM/Pluton 炸了
由于联想的某个 BIOS 更新在安全选项卡里新增了选项，导致 TPM 设置被重置为关闭，需要重新手动打开。

### 1Password 无法打开 TPM 选项了
因为前面的原因 TPM 被重置，Windows Hello 数据被从 TPM 中移出，需要确保 TPM 被打开的前提下，使用 `certutil -DeleteHelloContainer` 并注销清空 Hello 数据并重新设置。

### OLED 屏幕过饱和
常开 HDR 模式即可。  
或者在 AMD 核显驱动中打开自定义颜色，并关闭色温控制。此时 SDR 模式为 sRGB 色域，HDR 模式不受影响，可按需切换。

### Microsoft Edge 看 4K HDR 视频掉帧
打开 `edge://flags/#enable-media-foundation-clear` 选项使用 Media Foundation 进行视频渲染。

### 雷电 Dock 在锁屏界面无法使用 USB
由于 Windows 新加入的安全机制 [内核 DMA 保护](https://docs.microsoft.com/zh-cn/windows/security/information-protection/kernel-dma-protection-for-thunderbolt)，不支持 DMA 重映射的驱动会被禁止加载。对于使用 ASMedia USB3.0 主控的设备，安装最新版本驱动即可解决（OEM 提供的驱动未必最新）。如果没有合适的驱动，可以去组策略中将 `计算机配置/管理模板/系统/内核 DMA 保护/与内核 DMA 保护不兼容的外部设备的枚举策略` 设置为 `允许所有`。

### 我想要极致的安全性
在 BIOS 的 Security 选项卡中进行以下设置：
- TSME - On
- Password - Supervisor Password - 设置管理员密码
- Password - Block SID Authentication - On
- Password - Lock UEFI BIOS Settings - On
- Password - Password at Unattended Boot - On
- Password - Password at Boot Device List - On
- Fingerprint - Predesktop Authenticaton - On
- Fingerprint - Security Mode - High
- Fingerprint - Password Authentication - On
- Security Chip - Security Chip Selection - Pluton TPM 2.0
- Security Chip - Security Chip - On
- Security Chip - Microsoft Pluton Processor Control - Enabled
- Memory Protection - Execution Prevention - On
- Virtualization - Enhanced Windows Biometric Security - On
- Device Guard - Device Guard - On

将组策略 `计算机配置/管理模板/系统/Device Guard/打开基于虚拟化的安全` 设置为：
- 选择平台安全级别 - 安全启动和 DMA 保护
- 基于虚拟化的代码完整性保护 - 使用 UEFI 锁启用
- Credential Guard 配置 - 使用 UEFI 锁启用
- 安全启动配置 - 已启用

### Enhanced Windows Biometric Security 正确开启方法
如果在该 BIOS 选项打开之前已经设置了 Windows Hello，需要清除 Windows Hello 数据后重新登录（方法见上文）。

此时 `certutil -csp "Microsoft Passport Key Storage Provider" -key -v | Select-String -Pattern "NgcKeyImplType"` 应显示代码 33。  

已知问题：打开该选项后人脸识别不可用。

### 插电待机时不定时自己开机并且风扇满转速，拔电后自动进入待机
推测是后台进行文件索引，不用理会即可。

### 从待机恢复后触控板有迟滞，手指不动时指针漂移
待解决

### USB4 与 HP Elite Thunderbolt 3 65W Dock 的兼容性汇总：
#### 不同状态下插拔 Dock 的现象
||合盖睡眠|合盖唤醒|开盖睡眠|开盖唤醒|
|--|--|--|--|--|
|插入|电源灯闪两下后保持睡眠|**一定概率USB失灵**|唤醒|**一定概率USB失灵**|
|拔出|保持待机|**保持唤醒**|保持待机|正常|

#### 未插入 Dock 时操作盖子和电源按钮的现象
||开盖睡眠|合盖睡眠|开盖唤醒|合盖唤醒|
|--|--|--|--|--|
|开合盖|保持睡眠|唤醒|睡眠|保持唤醒|
|按电源键|唤醒|**无反应**|睡眠|**无反应**|

#### 已插入 Dock 时操作盖子和电源按钮的现象
||开盖睡眠|合盖睡眠|开盖唤醒|合盖唤醒|
|--|--|--|--|--|
|开合盖|保持睡眠|唤醒|睡眠|保持唤醒|
|按电源键|唤醒|**唤醒，USB一定失灵**|睡眠|**睡眠**|

#### 奇妙小概率现象
- 合盖时插入 Dock，显示正常，键鼠正常，但解不开锁屏（点击鼠标或按键盘无法弹出 PIN 输入框），此时开盖使用机身触摸板和键盘也无法解开，只能重新进入待机再唤醒。
- 开盖睡眠时插入，显示器间歇黑屏，重启后无法复现。
- 开盖睡眠时插入，唤醒后掉电重启。
- 合盖唤醒时插入，掉电重启。
- 合盖睡眠时按电源键，掉电重启。
- 插入时手动重启，进入锁屏后掉电重启。
