# Báo cáo điều tra máy tự bật lại sau shutdown

Ngày kiểm tra: 2026-09-02  
Máy: `DESKTOP-3T2R9GC`  
Hệ điều hành: Windows 10 Pro 10.0.19045  
Mục tiêu: Giữ Wake-on-LAN hoạt động bằng Magic Packet

## Kết luận

Máy thực sự shutdown sạch xuống S5 rồi được firmware hoặc phần cứng power-on trở lại. Các lần gần đây không phải resume từ sleep/hibernate, crash/reboot, Fast Startup hay hybrid shutdown.

Windows Event Log không lưu chính xác tác nhân bật máy từ S5. Realtek hiện chỉ được phép wake bằng Magic Packet trong Power Management API và registry. Chưa có evidence cho thấy traffic thông thường, pattern match, USB hoặc Scheduled Task gây power-on.

SMBIOS của phiên boot được kiểm tra báo `WakeUpType = 6` (`Power Switch`). Đây là evidence hỗ trợ nhưng không đủ kết luận độc lập vì độ tin cậy phụ thuộc BIOS và chỉ phản ánh boot hiện tại.

## Timeline gần nhất

```text
2026-09-02 07:59:51  User32 1074         shutdown.exe initiated shutdown
2026-09-02 07:59:57  EventLog 6006       Event Log stopped cleanly
2026-09-02 07:59:58  Kernel-Power 109    ShutdownActionType=4
2026-09-02 07:59:58  Kernel-General 13   OS shutting down

— máy tắt khoảng 19 phút 45 giây —

2026-09-02 08:19:43  Kernel-General 12   new operating-system start
2026-09-02 08:19:43  Kernel-Boot 27      BootType=0
2026-09-02 08:19:53  EventLog 6005       Event Log started
```

Các lần tương tự:

```text
2026-09-01 23:54:19 shutdown sạch -> 2026-09-02 05:43:54 full boot
2026-09-01 17:39:40 shutdown sạch -> 2026-09-01 22:47:30 full boot
2026-09-01 10:30:48 shutdown sạch -> 2026-09-01 12:11:32 full boot
2026-08-30 15:50:06 shutdown sạch -> 2026-08-30 15:53:38 full boot
```

Không có Event 41/6008 gần các lần này. Không có Power-Troubleshooter 1 hay chuỗi Kernel-Power 42/107 tương ứng. Fast Startup bị policy vô hiệu hóa và `HiberbootEnabled=0`.

## Realtek PCIe GbE

```text
Adapter: Realtek PCIe GbE Family Controller
MAC: F0-2F-74-51-17-33
Driver: 10.77.50.807 (2025-08-07)
Link: 1 Gbps

WakeOnMagicPacket: Enabled
WakeOnPattern: Disabled
Shutdown Wake-On-Lan: Enabled
WOL & Shutdown Link Speed: 10 Mbps First
```

Registry tại `HKLM\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\0001`:

```text
*WakeOnMagicPacket           = 1
*WakeOnPattern               = 0
S5WakeOnLan                  = 1
WolShutdownLinkSpeed         = 0
*ModernStandbyWoLMagicPacket = 0
PnPCapabilities              = 256
```

WOL từ S5 đang bật đúng, pattern match đã tắt trong API, advanced property và registry. Windows chỉ đăng ký Magic Packet. ARP/NS Offload đang bật nhưng không được đăng ký làm wake pattern.

## Wake timers, tasks và USB

`powercfg /waketimers` không có active wake timer. Sáu Scheduled Task có thuộc tính `WakeToRun` đều disabled, bao gồm UpdateOrchestrator, InstallService, .NET NGEN và SharedPC.

Các thiết bị `wake_armed` gồm keyboard, mouse và Realtek NIC. HID có thể đánh thức S3, nhưng danh sách này không chứng minh chúng có thể bật máy từ S5. Nếu BIOS cho phép USB power-on from S5 thì đó là cơ chế firmware, bên ngoài Event Log của Windows.

## Firmware

```text
Mainboard: ASUS PRIME Z490-P Rev 1.xx
BIOS: American Megatrends 1410
BIOS date: 2020-09-04
```

Windows thấy `ACPI Wake Alarm` là phần cứng hỗ trợ wake nhưng không có active timer. Cần kiểm tra trực tiếp BIOS đối với Power On By RTC, Restore AC Power Loss, USB power-on from S5 và PCIe PME wake.

## Test quyết định tiếp theo

1. Shutdown bình thường rồi rút LAN ngay khi máy tắt.
2. Nếu không còn tự bật, NIC/PCIe là đường power-on; kiểm tra thiết bị hoặc dịch vụ đang gửi Magic Packet.
3. Nếu vẫn tự bật, rút keyboard/mouse/USB và kiểm tra power switch, RTC wake, AC restore.
4. Nếu cần test nút nguồn, chỉ tháo đầu `PWR_SW` cho một chu kỳ khi có thể thao tác phần cứng an toàn.

Giữ nguyên cấu hình tối thiểu:

```text
Wake on Magic Packet  = Enabled
Shutdown Wake-On-Lan  = Enabled
Wake on pattern match = Disabled
```

Không tắt Shutdown Wake-On-Lan hoặc bỏ NIC khỏi wake configuration vì sẽ làm mất WOL từ S5.

## Lệnh điều tra chính

```powershell
powercfg /a
powercfg /lastwake
powercfg /waketimers
powercfg /devicequery wake_armed
powercfg /devicequery wake_programmable
powercfg /devicequery wake_from_S3_supported
powercfg /query SCHEME_CURRENT SUB_SLEEP RTCWAKE
powercfg /requests

Get-WinEvent ...
Get-NetAdapter
Get-NetAdapterPowerManagement
Get-NetAdapterAdvancedProperty
Get-ScheduledTask
Get-ScheduledTaskInfo
Get-PnpDevice
Get-CimInstance Win32_OperatingSystem
Get-CimInstance Win32_ComputerSystem
Get-CimInstance Win32_BIOS
Get-CimInstance Win32_BaseBoard
Get-CimInstance Win32_OSRecoveryConfiguration
```

Không có cấu hình Windows, registry, NIC, task hoặc BIOS nào bị thay đổi trong quá trình điều tra.
