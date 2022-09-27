# Запускаем ОС Linux на ноутбуке TCL B220G (он же TCL Book 14 Go B220G)

## Преамбула

Фирма TCL Communication Ltd из великой поднебесной империи выпустила ноутбук TCL Book 14 Go B220G он же просто B220G. Конкретно у меня модель B220G-3ALCRU1. Данный ноутбук оснащен процессором Qualcomm Snapdragon 7c Gen1 SC7180 8 ядер архитектуры ARMv8 (Aarch64, или ARM64) с частотой 2.4ГГц, 4ГБ оперативной памяти, 128ГБ твердотельным накопителем SSD,видеоускорителем Adreno 618 работающим на 14-и дюймовый дисплей разрешением 1366х768 точек с чатотой 60Гц. Работает под управлением Windows 11 Home. А еще есть двухдиапазонный Wi-Fi, Bluetooth и даже встроенный 4G/LTE радиомодуль. В целом все заявленное выполняется и работает. Бюджетная машика, с откровенно неудачным дисплеем. Неплохо собранная, не откровенно тормозная, но и звезд с неба хватать не собирающаяся. Одна из немногих (по состоянию на сентябрь 2022 года), работающих на архитектуре ARM64.

## Постановка задачи

Производитель предлагает безальтернативно Windows 11. Она работатет. Даже с двоичной трансляцией кодов x86. Но... На сегодня (сентябрь 2022 года) очень мало приложений работают нативно на архитектуре ARM64. В частности нет порта Libre Office. И в целом, не хорошо когда нет возможности выбора. Считаю необходимым обеспечить работу операционной сиситемы GNU Linux на этой машине. Предпосылки для этого есть.

## Беглый осмотр

По кнопке DEL входим в некий аналог BIOS как на системах x86. Нам доступно включение/выключение режима разработчика (звучит многообещающе) и Secure Boot (тоже не плохо). Ну еще можно задать пароль. И все. Выключение Windows с удержанием кнопки Shift позволяет попасть в некоторый аналог меню загрузки и выбрать источник загрузки. Ожидаемо только UEFI. 

Диспетчер усройств:
* накопитель Micron MT128GASAO4U21 с непонятным интерфейсом UFS 2.1. Плохо. Лучше бы привычные SATA или NVMe. Даже у Micron'а этот интерфейс докуметирован очень плохо.
* Qualcom(R) Blutooth UART Transport Driver намекает на то, что с Bluetoth проблем не ожидается
* Видеоадаптер Qualcomm(R) Ardeno(TM) 618 GPU
* AW9610x SAR Sensor - загадочная штука
* USB Camera c VID 0x13D3 PID 0x784B что опознается как IMG Network и дает надежду на то, что с драйверами проблем не будет
* Клавиатура HID что бы это не значило
* HID совместимая мышь (трекпад) на шине I2C - не плохо и не хорошо. Надо разбираться.
* Модем и WiFi особой надежды не дают к сожалению
* Накопители SD Storage и универсального флешь-накопителя (UFS) Qualcomm. Первый норм, второй грустно. Почему-то за версту несет неподдерживаемым GNU Linux решением.
* EDPBridge тонко намекает на то, что дисплей подключен через него, но в системе он виден как DisplayPort.
* и много другого по мелочи, особенно в системных устройствах...

## Осматриваемся глубже

Снимаем крышку. И буквально сразу видим DEBUG RX, DEBUG TX, DEBUG GND. Многообещающе... Рядом контрольные точки с напряжениями питаний, шинами SPI и I2C. Все компоненты распаяны на плате и если и подлежат замене, то только с паяльником. Нормальное состояние для бюджетной техники. Впрочем, аппаратный модинг - это в далеком будущем. Пока посотрим что нам пишет отадочный порт и позволяет ли хоть что-нить сделать.

Журнал загрузки
<pre>
Format: Log Type - Time(microsec) - Message - Optional Info
Log Type: B - Since Boot(Power On Reset),  D - Delta,  S - Statistic
S - QC_IMAGE_VERSION_STRING=BOOT.XF.3.1.1-00010-SC7180XWZB-1
S - IMAGE_VARIANT_STRING=RennellWP
S - OEM_IMAGE_VERSION_STRING=SM042022201
S - Boot Interface: UFS
S - Secure Boot: Off
S - Boot Config @ 0x00786070 = 0x000000c8
S - JTAG ID @ 0x00786130 = 0x1012c0e1
S - OEM ID @ 0x00786138 = 0x00000000
S - Serial Number @ 0x00786134 = 0x7cf14e03
S - OEM Config Row 0 @ 0x007841b8 = 0x0000000000000000
S - OEM Config Row 1 @ 0x007841c0 = 0x0000000000000000
S - Feature Config Row 0 @ 0x007841d0 = 0x0840200015300400
S - Feature Config Row 1 @ 0x007841d8 = 0x0014000000008080
S - Core 0 Frequency, 1516 MHz
S - PBL Patch Ver: 5
S - PBL freq: 600 MHZ
D -      5137 - pbl_apps_init_timestamp
D -     73192 - bootable_media_detect_timestamp
D -      1002 - bl_elf_metadata_loading_timestamp
D -       704 - bl_hash_seg_auth_timestamp
D -      8008 - bl_elf_loadable_segment_loading_timestamp
D -      5425 - bl_elf_segs_hash_verify_timestamp
D -      7215 - bl_sec_hash_seg_auth_timestamp
D -       889 - bl_sec_segs_hash_verify_timestamp
D -        37 - pbl_populate_shared_data_and_exit_timestamp
S -    101609 - PBL, End
B -    122915 - SBL1, Start
B -    241987 - SBL1 BUILD @ 14:16:36 on Jul 13 2022
B -    248087 - boot_flash_init, Start
D -        30 - boot_flash_init, Delta
B -    252997 - xblconfig_init, Start
B -    416843 - UFS INQUIRY ID: MICRON  MT128GASAO4U21  0101
D -      1159 - Auth Metadata
D -    196023 - xblconfig_init, Delta
B -    452620 - boot_config_data_table_init, Start
B -    456280 - Using default CDT
D -      4636 - boot_config_data_table_init, Delta - (54 Bytes)
B -    469578 - CDT Version:3,Platform ID:29,Major ID:1,Minor ID:0,Subtype:0
B -    476989 - pm_device_init, Start
B -    481991 - PM: PM 0=0x8000028000000080:0x800 
B -    482083 - PM: PM 2=0x8000018000000020:0x1000 
B -    486719 - PM: POWER ON by KPDPWR, POWER OFF by PS_HOLD
B -    570533 - PM: SET_VAL:Skip
B -    570807 - PM: PSI: b0x30_v0x17
B -    578310 - PM: Device Init # RTC: 0, #SDAM: 0
B -    578371 - PM: Device Init # SPMI Transn: 5144
D -    105957 - pm_device_init, Delta
B -    587704 - pm_driver_init, Start
B -    596366 - PM: Driver Init # SPMI Transn: 362
D -      5185 - pm_driver_init, Delta
B -    601063 - vsense_init, Start
D -         0 - vsense_init, Delta
B -    610976 - sbl1_ddr_set_params, Start
B -    611220 - Pre_DDR_clock_init, Start
D -        91 - Pre_DDR_clock_init, Delta
D -      8296 - sbl1_ddr_set_params, Delta
B -    622627 - sbl1_ddr_init, Start
D -      8052 - sbl1_ddr_init, Delta
B -    634644 - DSF version = 104.0.19, DSF SHRM version = 59.5
B -    643733 - Manufacturer ID = ff, Device Type = 7
B -    643885 - Rank 0 size = 4096 MB, Rank 1 size = 0 MB
B -    648948 - Row Hammer Check : DRAM supports unlimited MAC Value : MR24[OP2:0 = 0] & MR24[OP3 = 1] for CH0 & CS0
B -    659776 - Row Hammer Check : DRAM supports unlimited MAC Value : MR24[OP2:0 = 0] & MR24[OP3 = 1] for CH1 & CS0
B -    670420 - do_ddr_training, Start
B -    676551 - Frequency = 1555 MHz
D -      3660 - do_ddr_training, Delta
B -    684176 - pImem Init Start
D -     11468 - pImem Init End, Delta
B -    697199 - Relocate Pagetable to DDR, Start
B -    701012 - Relocate Pagetable to DDR, End
B -    705129 - External heap init, Start
B -    709430 - External heap init, End
B -    713273 - clock_init, Start
D -     20557 - clock_init, Delta
B -    737642 - Loading APDP Image
D -      4392 - Auth Metadata
D -       366 - Segments hash check
D -     11011 - Image Loaded, Delta - (21124 Bytes)
B -    752130 - usb: UFS Serial - 4e982bb5
B -    756735 - usb: fedl, vbus_low
B -    761066 - PM: SMEM Chgr Info Write Success
B -    764391 - Loading OEM_MISC Image
D -       915 - Auth Metadata
D -       335 - Segments hash check
D -     10370 - Image Loaded, Delta - (8024 Bytes)
B -    778085 - Loading QTI_MISC Image
D -      4636 - Image Loaded, Delta - (0 Bytes)
B -    791566 - PM: PM Total Mem Allocated: 1849 
B -    791597 - Loading AOP Image
D -       976 - Auth Metadata
D -      1830 - Segments hash check
D -     14152 - Image Loaded, Delta - (157928 Bytes)
B -    809073 - Loading QSEE Dev Config Image
D -      1128 - Auth Metadata
D -       610 - Segments hash check
D -     11773 - Image Loaded, Delta - (38124 Bytes)
B -    824171 - Loading QSEE Image
D -      4331 - Auth Metadata
D -     19123 - Segments hash check
D -     54534 - Image Loaded, Delta - (2967465 Bytes)
B -    882243 - Loading SEC Image
D -      4666 - Image Loaded, Delta - (0 Bytes)
B -    890051 - Loading QHEE Image
D -      1159 - Auth Metadata
D -      2653 - Segments hash check
D -     16836 - Image Loaded, Delta - (394336 Bytes)
B -    910211 - Loading STI Image
D -      4819 - Image Loaded, Delta - (0 Bytes)
B -    918294 - Loading APPSBL Image
D -      1464 - Auth Metadata
D -     10339 - Segments hash check
D -     30012 - Image Loaded, Delta - (2088960 Bytes)
B -    955199 - SBL1, End
D -    833595 - SBL1, Delta
S - Flash Throughput, 111000 KB/s  (5746011 Bytes,  51503 us)
S - DDR Frequency, 1555 MHz


UEFI Start     [ 1173] SEC
ASLR        : OFF [WARNING]
DEP         : ON (RTB)
Timer Delta : +1 mS
RAM Entry 0 : Base 0x0080000000  Size 0x003D800000
RAM Entry 1 : Base 0x00C0000000  Size 0x00C0000000
RAM Entry 2 : Base 0x00BD800000  Size 0x0000100000
Total Installed RAM : 4096 MB (0x0100000000)
UART Buffer size set to 0x8000 
Continue booting UEFI on Core 0
UEFI Ver    : 1.21
Build Info  : 64b Jul 29 2022 09:22:16
Boot Device : UFS
PROD Mode   : TRUE
Retail      : TRUE
npa_create_sync_client_ex start
PM0: 40, PM2: 31, 
UFS INQUIRY ID: MICRON  MT128GASAO4U21  0101
Get Card Info size = -864026624
LoadSecureApps: Embedded common libs load result 0
MDPLibProfile: Entry[MDPSetProperty()][6]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 10
MDPLibProfile: Exit[MDPPlatformConfigure()][86us function time][86us total time]
MDPLibProfile: Exit[MDPSetProperty()][150us function time][236us total time]
MDPLibProfile: Entry[MDPInit][0]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 6
SetupPlatformPanelConfig lt8911
SetupPlatformPanelConfig bDetectPanel = 0
MDPLibProfile: Exit[MDPPlatformConfigure()][120us function time][356us total time]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 5
MDPLibProfile: Exit[MDPPlatformConfigure()][37us function time][393us total time]
MDPLibProfile: Entry[MDPSetCoreClock()][0]
MDPLibProfile: Exit[MDPSetCoreClock()][564us function time][957us total time]
MDPLibProfile: Exit[MDPInit()][826us function time][1783us total time]
MDPLibProfile: Entry[MDPPower()][0]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 1
Panel_CLS_PowerUp
MDPLibProfile: Exit[MDPPlatformConfigure()][31816us function time][33599us total time]
MDPLibProfile: Exit[MDPPower()][31878us function time][65477us total time]
MDPLibProfile: Entry[MDPDetect()][0]
MDPLibProfile: Entry[MDPDetectPanel()][0]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 0
trace bDetectPanel = 0
GetPanelXmlConfig ok
MDPLibProfile: Exit[MDPPlatformConfigure()][4203us function time][69680us total time]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 14
Panel_CLS_Reset_LT8911
MDPLibProfile: Exit[MDPPlatformConfigure()][212404us function time][282084us total time]
EDPBridgeDriver_EDIDRead_LT8911
LT8911EXB edp bridge chip id 17 5 E0
LT8911 tx pll locked
Read eDP EDID......

0x00,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x00,0x09,0xE5,0x0D,0x09,0x00,0x00,0x00,0x00,
0x1C,0x20,0x01,0x04,0x95,0x1F,0x11,0x78,0x03,0xF8,0x45,0x96,0x57,0x54,0x92,0x28,
0x23,0x50,0x54,0x00,0x00,0x00,0x01,0x01,0x01,0x01,0x01,0x01,0x01,0x01,0x01,0x01,
0x01,0x01,0x01,0x01,0x01,0x01,0x2B,0x1B,0x56,0x78,0x50,0x00,0x0C,0x30,0x30,0x20,
0x36,0x00,0x35,0xAE,0x10,0x00,0x00,0x1A,0x1D,0x12,0x56,0x78,0x50,0x00,0x0C,0x30,
0x30,0x20,0x36,0x00,0x35,0xAE,0x10,0x00,0x00,0x1A,0x00,0x00,0x00,0x00,0x00,0x00,
0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x02,
0x00,0x0D,0x40,0xFF,0x0A,0x3C,0x7D,0x11,0x11,0x21,0x7D,0x00,0x00,0x00,0x00,0x52,

eDP Timing = { H_FP / H_pluse / H_BP / H_act / H_tol / V_FP / V_pluse / V_BP / V_act / V_tol / D_
eDP Timing = { 48, 32, 40, 1366, 1486, 3, 6, 3, 768, 780, 6955 }
 BitDeepPara:(208,0,0)
EDID_H:(1366,48,40,32,0)
EDID_V:(768,3,3,6,6973)
EDID_Board:(0,0,0,0,0x3C0000)
eDP_lane:(1)
EdpConfig: 0x00000000BD800000
PanelCfg print finish,totle lengh: 3942
EdpOffSetAddr=-1115684864
DisplayDxe: Resolution 1366x768 (1 intf)
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 5
MDPLibProfile: Exit[MDPPlatformConfigure()][36us function time][282120us total time]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 6
MDPLibProfile: Exit[MDPPlatformConfigure()][82us function time][282202us total time]
MDPLibProfile: Exit[MDPDetectPanel()][114us function time][282316us total time]
MDPLibProfile: Exit[MDPDetect()][141us function time][282457us total time]
MDPLibProfile: Entry[MDPSetProperty()][3]
MDPLibProfile: Exit[MDPSetProperty()][22us function time][282479us total time]
MDPLibProfile: Entry[MDPSetProperty()][4]
MDPLibProfile: Exit[MDPSetProperty()][22us function time][282501us total time]
HW Wdog Setting from PCD : Disabled
ScmSipSysCall() failed, Status = (0x7)
Device booted from UFS

 APC1 Total 66
LoadSys  TIME 0ms
tsens  TIME 0ms
scm  TIME 0ms
LmhIsenseInit Pre CPU  TIME 0ms
GLD APC TOTAL IDDQ 66
LmhIsenseSubSysEntry Post SubSysEntryCb  TIME 0ms
LmhIsenseSubSysEntry Post LmhIsenseSubSysInit  TIME 0ms
LmhIsenseInitSubSys Post SubSysEntry  TIME 0ms
Core 0, CDYN=2250pF, IDDQ FUSE Volt 750, IDDQ Temp 30
Volt(mV)	Freq(KHz)	IDyn(mA)	ILeak(mA)	ITotal(mA)	Temp(c)	CorePair	ADC
828	1555200	2897	75	2973	32	0	149
828	1708800	3183	73	3256	31	0	157
828	1900800	3541	73	3614	31	0	174
828	2112000	3934	75	4010	32	0	182
828	2208000	4113	75	4189	32	0	201
828	2323200	4328	75	4403	32	0	217
828	2400000	4471	75	4547	32	0	205
CoreIdx	Slope	Intercept
0	181	-71
TIME TO MEASURE 3ms OpPoints 7
LmhIsenseInitSubSys Post SubSysTrim  TIME 4ms
LmhIsenseInitSubSys Post SubSysExit  TIME 4ms
LmhIsenseInit Post CPU  TIME 4ms
LmhIsenseInit Post Store  TIME 5ms
isense  TIME 6ms
ISENSE TOTAL TIME 6ms
qusb2_1: hstx: 2
PLL1 locked: 5
usb_lane: 0
EC Version = 1.13.
I2CKeyboardEntryPoint.
[DppDxe] Failed to read DPP block data with RawFSProtocol.
: BlockGetDPP returned Not Found
[LoadSHIPFromDPP] DppProtocol->GetDPP failed. Status: 14.
Gpio_Init_1!
SwitchUsbModeEntryPoint start
[LoadECFromDPP] WriteEcRegsI2C Set USB device mode SUCC
[LoadECFromDPP] DppProtocol->GetDPP EcFileBuffer: 48
SwitchUsbModeEntryPoint end
MDPLibProfile: Entry[MDPSetMode()][0]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 8
MDPLibProfile: Exit[MDPPlatformConfigure()][45us function time][282546us total time]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 8
MDPLibProfile: Exit[MDPPlatformConfigure()][33us function time][282579us total time]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 8
MDPLibProfile: Exit[MDPPlatformConfigure()][35us function time][282614us total time]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 8
MDPLibProfile: Exit[MDPPlatformConfigure()][32us function time][282646us total time]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 7
MDPLibProfile: Exit[MDPPlatformConfigure()][102us function time][282748us total time]
EDPBridgeDriver_SetMode_LT8911
UsbEnumeratePort: new device connected at port 0, HubIfc =0xFD2FAB98 
LT8911 tx pll locked
link train on going...
Link train success, address 0x82 val= 62 
panel link rate: 10 
panel link count: 129 
LT8911 video check: mipi byteclk = 52028 
LT8911 video check: Vtotal = 780 

video check: Hact(word counter) = 1366 
 
video check: Vact = 768 
 
Reg0xD087 = 18
PCR Clock stable
MDPLibProfile: Exit[MDPSetMode()][474476us function time][757224us total time]
MDPLibProfile: Entry[MDPSetProperty()][0]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 4
MDPLibProfile: Exit[MDPPlatformConfigure()][21608us function time][778832us total time]
MDPLibProfile: Exit[MDPSetProperty()][21668us function time][800500us total time]
MDPLibProfile: Entry[MDPSetProperty()][2]
MDPLibProfile: Entry[MDPPlatformConfigure()][0]
MDPPlatformConfigure type = 5
MDPLibProfile: Exit[MDPPlatformConfigure()][31us function time][800531us total time]
MDPLibProfile: Exit[MDPSetProperty()][1214us function time][801745us total time]
WaitParallelThreads InIt [ 4055] 
-----------------------------
Platform Init  [ 4055] BDS
UEFI Ver   : 1.21
Platform           : CLS
Subtype            : 0
Boot Device        : UFS
Chip Name          : SC7180
Chip Ver           : 1.1
Chip Serial Number : 0x7CF14E03
-----------------------------
UsbEnumeratePort: new device connected at port 3, HubIfc =0xFD21FE18 
UsbSelectConfig: failed to connect driver Not Found, ignored
UsbSelectConfig: failed to connect driver Not Found, ignored
Successfully loaded Tools FV
Current is disabled
WaitForTimeout:0
WaitForTimeout:0
WaitForTimeout:0
WaitForTimeout:0
WaitForTimeout:0
BattShow  don't need to show low batt tips
ScmSlowsyscall() failed, Status = (0x2)
[DppDxe] Failed to read DPP block data with RawFSProtocol.
: BlockGetDPP returned Not Found
LoadEC failed. Status: Not Found.
ENEFlashApp force to flash EC.bin failed. Status: Not Found.
Start image failed
Failed to launch ENEAutoFlash app, status: Not Found
No pending capsules found in EFI Raw file
ACPI: Loaded 1 tables
TPM type 0x6654504D successfully updated to ACPI variable.
[DppDxe] Failed to read DPP block data with RawFSProtocol.
: BlockGetDPP returned Not Found
: DppProtocol->GetDPP returned Not Found
 HandleMorPpi delta: 13
Platform Init End : 5579
-----------------------------
JLQ_Gpio_Init_2
[QcomBds] InitOption=0,USBFirst=0,PxeBootFirst=0
Booting option 0:(Boot0003) "Windows Boot Manager"
UEFI Total : 4460 ms
POST Time      [ 5633] OS Loader
Start Boot Windows Boot Manager
ScmSlowsyscall() failed, Status = (0x2)
ScmSlowsyscall() failed, Status = (0x2)
ScmSlowsyscall() failed, Status = (0x2)
ConvertPages: range FCCCA000 - FCE70FFF covers multiple entries
ConvertPages: range FCBF9000 - FCCC8FFF covers multiple entries
ScmSlowsyscall() failed, Status = (0x2)
Start EBS        [ 6559] 
BDS: LogFs sync skipped, Unsupported
App Log Flush : 0 ms
Exit EBS        [ 6593] UEFI End
</pre>

## Найденные комбинации клавиш управления загрузкой

* F2 или DEL - вызов некоего аналога BIOS, позволяющего включить/выключить Secure Boot, Developer Mode (на данный момент неизвестно что этот режим дает), а так же запустить режим поставки или Shipping mode (в этом режиме включение возможно _ТОЛЬКО_ при наличии подключенного сетевого адаптера, что исключает возможность случайного включения в процессе транпортировки и посадки батареи в ноль)
* F7 - вызов загрузочного меню UEFI (наконец-то не надо больше перезагружать Windows удерживая Shift для выбора источника загрузки). Загрузка возможна _ТОЛЬКО_ в режиме UEFI для архитектуры ARM64, т.е. ищем на накопителе файл EFI/BOOT/BOOTAA64.EFI и только по его наличию определяем диск как загрузочный.
* F5 или Стрелка Вниз - ощутимо "тормозят" процесс загрузки на отображении логотипа TCL. Вероятно имеет смысл посмотреть на то, что в таком режиме происходит в опладочной консоли, но пока ясности нет.

Пока все догадки видны. Попытки хоть как-топродвинуться дальше поваливаются. Флешка с Ubuntu Desktop ARM64 вообще отказывается стартовать, Debian 11 вполне себе запускает GRUB но и все. Логично, нет тут дерева устройств. Его только предстоит создать. Благо процессор поддерживается ванильным ядром.

Собственно в этот момент я и решил что стоит выложить идеи на github. Возможно не я один этим озадачен и вместе получится быстрее. Пользуясь случаем - ищу контакты в TCL. Они могут подсказать очень многое и облегчить работу. Не надо схему - главный вопрос что к какому интерфейсу подключено. Пока на этом все.
