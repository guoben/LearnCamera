# log系统详解

## 1.如何打开log开关
在camx架构下，打开log开关一共有4种方法：
* 方法1：修改camxsetting.xml文件
说明：
    - 测方法可以用于**代码提交**；
    - 验证比较麻烦；
* 方法2：直接写节点
说明：
    - 此方法只适用于**临时验证**；
    - xml文件中Dynamic属性为false时，此法失效；
    - persist开头的节点重启后依然有效；
    - vendor开头的节点重启后失效；
* 方法3：写camxoverridesettings.txt文件
说明：
    - 此方法只适用于**临时验证**；
    - 与xml配置无关，一直可以生效；
* 方法4：修改系统的debug节点
说明：
    - 测方法只用于**kernel测试**；

下面会结合各个模块实际情况进行说明。

## 2.Camx模块修改方法
> CamX模块指的是/vendor/qcom/proprietary/camx目录下的代码

在camx目录中，所有的log形式如下：
CAMX_LOG_INFO(CamxLogGroupSensor, "Power settings are not valid.");
CAMX_LOG_INFO指定了log对应的级别；
CamxLogGroupSensor指定了的需要打log的模块。

* 方法1：
如果使用这种方法，需要修改camx/src/settings/common/camxsettings.xml文件，需要先找到
    ```xml
    <setting>
        <Name>Info log mask</Name>
        <Help>Mask of which info logs are output<Help>
        <VariableName>logInfoMask</VariableName>
        <VariableType>UINT</VariableType>
        <SetpropKey>persist.vendor.camera.logInfoMask</SetpropKey>
        <DefaultValue>0</DefaultValue> <!--0x1C080-->
        <Dynamic>TRUE</Dynamic>
        <Public>TRUE</Public>
    </setting>
    ```
    然后，修改DefaultValue的值为02；
    接下来编译camera.qcom.so这个库，push进手机中的/vendor/lib64/camera/目录中，重启后验证；

* 方法2：
此方法使用`adb shell setprop propName propValue`设置即可。
其中propName是节点的名称，与log输出相关的节点如下所示，不同级别的log对应不同的节点，它应该是`<SetPropKey>`这个xml节点的值。
propValue是一个32位数，每一位（bit）代表一个模块，对应位被置1，则表示此模块被选择。 
比如，想让这句log打出来，
    - 首先选择CAMX_LOG_INFO对应的节点（persist.vendor.camera.logInfoMask）
    - 然后选择propValue,由于对应模块为CamxLogGroupSensor，所以对应值为0x2;
    - 也就是执行`adb shell setprop persist.vendor.camera.logInfoMask  2`
    - 然后重新打开相机应用即可。

* 方法3：
此方法将"propName=propValue"这个字符串写入/vendor/etc/camera/camxoverridesettings.txt文件中，重启即可；
其中，propName指的是需要控制的变量名称，对应xml中VariableName，应该与这个名字一致；propValue是一个32位数，每一位（bit）代表一个模块，对应位被置1，则表示此模块被选择，这个值可以使用16进制。
比如，想让这句log打出来，需要执行：
    - 找到CAMX_LOG_INFO对应的propName，即logInfoMask
    - 然后选择propValue,由于对应模块为CamxLogGroupSensor，所以对应值为0x2;
    - 执行`adb shell` 
    - 执行`echo "logInfoMask=0x2" > /vendor/etc/camera/camxoverridesettings.txt`
    - 然后重启手机即可。

* 方法4：
不适用与CamX子模块中；

```c
// camx中支持的log等级与节点对应关系如下：
persist.vendor.camera.logVerboseMask      // CAMX_LOG_VERBOSE
persist.vendor.camera.logWarningMask      // CAMX_LOG_WARNING
persist.vendor.camera.logConfigMask       // CAMX_LOG_CONFIG
persist.vendor.camera.logInfoMask         // CAMX_LOG_INFO
```

```c
// camx中支持的如下模块：
 CamxLogGroup CamxLogGroupAFD           = ((1) << 0);    // AFD
 CamxLogGroup CamxLogGroupSensor        = ((1) << 1);    // Sensor
 CamxLogGroup CamxLogGroupTracker       = ((1) << 2);    // Tracker
 CamxLogGroup CamxLogGroupISP           = ((1) << 3);    // ISP
 CamxLogGroup CamxLogGroupPProc         = ((1) << 4);    // Post Processor
 CamxLogGroup CamxLogGroupMemMgr        = ((1) << 5);    // MemMgr
 CamxLogGroup CamxLogGroupPower         = ((1) << 6);    // Power
 CamxLogGroup CamxLogGroupHAL           = ((1) << 7);    // HAL
 CamxLogGroup CamxLogGroupJPEG          = ((1) << 8);    // JPEG
 CamxLogGroup CamxLogGroupStats         = ((1) << 9);    // Stats
 CamxLogGroup CamxLogGroupCSL           = ((1) << 10);   // CSL
 CamxLogGroup CamxLogGroupApp           = ((1) << 11);   // Application
 CamxLogGroup CamxLogGroupUtils         = ((1) << 12);   // Utilities
 CamxLogGroup CamxLogGroupSync          = ((1) << 13);   // Sync
 CamxLogGroup CamxLogGroupMemSpy        = ((1) << 14);   // MemSpy
 CamxLogGroup CamxLogGroupFormat        = ((1) << 15);   // Format
 CamxLogGroup CamxLogGroupCore          = ((1) << 16);   // Core
 CamxLogGroup CamxLogGroupHWL           = ((1) << 17);   // HWL
 CamxLogGroup CamxLogGroupChi           = ((1) << 18);   // CHI
 CamxLogGroup CamxLogGroupDRQ           = ((1) << 19);   // DRQ
 CamxLogGroup CamxLogGroupFD            = ((1) << 20);   // FD
 CamxLogGroup CamxLogGroupIQMod         = ((1) << 21);   // IQ module
 CamxLogGroup CamxLogGroupLRME          = ((1) << 22);   // LRME
 CamxLogGroup CamxLogGroupCVP           = ((1) << 22);   // CVP
 CamxLogGroup CamxLogGroupNCS           = ((1) << 23);   // NCS
 CamxLogGroup CamxLogGroupMeta          = ((1) << 24);   // Metadata
 CamxLogGroup CamxLogGroupAEC           = ((1) << 25);   // AEC
 CamxLogGroup CamxLogGroupAWB           = ((1) << 26);   // AWB
 CamxLogGroup CamxLogGroupAF            = ((1) << 27);   // AF
 CamxLogGroup CamxLogGroupSWP           = ((1) << 28);   // SW Process
 CamxLogGroup CamxLogGroupHist          = ((1) << 29);   // Histogram Process
 CamxLogGroup CamxLogGroupBPS           = ((1) << 30);   // BPS
 CamxLogGroup CamxLogGroupDebugData     = ((1) << 31);   // Debug-Data
```

## 3.chi-cdk模块
> chi-cdk模块指的是/vendor/qcom/proprietary/chi-cdk下面的代码

在chi-cdk模块下，所有的log形式如下：
CHX_LOG_INFO("%s: Destroyed", IdentifierString())
其中，CHX_LOG_INFO代表了log级别，其对应关系如下所示。
```c
static const UINT32 CHX_LOG_ERROR_MASK   = 1;
static const UINT32 CHX_LOG_WARN_MASK    = 2;
static const UINT32 CHX_LOG_CONFIG_MASK  = 4;
static const UINT32 CHX_LOG_INFO_MASK    = 8;
static const UINT32 CHX_LOG_DUMP_MASK    = 16;
static const UINT32 CHX_LOG_VERBOSE_MASK = 32;
static const UINT32 CHX_LOG_MASK         = 64;
```
我们用**propValue**表示最终要设置的值，每个位(bit)代表一个log等级，在系统中设置后，即可生效；

* 方法1：
chi-cdk对应的xml节点如下：
    ```xml
    <setting>
        <Name>Set Log levels</Name>
        <Help>
            Bitmask of log levels,
            bit 0 - error,
            bit 1 - warning,
            bit 2 - config,
            bit 3 - info,
            bit 4 - dump
            bit 5 - verbose,
            bit 6 - log
            Set bit HIGH to enable
            Bits 0,1,2,3,4 are enabled by default
        </Help>
        <VariableName>overrideLogLevels</VariableName>
        <VariableType>UINT</VariableType>
        <SetpropKey>vendor.debug.camera.overrideLogLevels</SetpropKey>
        <DefaultValue>0</DefaultValue>  <!--0x1F-->
        <Dynamic>FALSE</Dynamic>
    </setting>
    ```
    修改DefaultValue的值后，编译生成`camera.qcom.so`，push进/vendor/lib64/hw/后重启即可生效。

* 方法2：
使用命令`adb shell setprop vendor.debug.camera.overrideLogLevels $(propValue)`
propValue的值要根据你希望打开log的级别计算。
注意：1.可以同时打开多个模块！2.重启后失效！

* 方法3：
将overrideLogLevels=$(propValue)写入/vendor/etc/camera/camxoverridesettings.txt中，重启之后即可生效；
注意：1.propValue的值要根据你希望打开log的级别计算。2.重启后才能生效！

* 方法4：
不适用于chi-cdk模块中。

## 4.Kernel模块
> Kernel模块是指kernel目录中的文件

* kernel模块中的log格式如下：
CAM_DBG(CAM_SENSOR, " Rxed Req Id:%lld", csl_packet->header.request_id);
CAM_SENSOR是对应要打印的模块，支持的模块如下所示。

* 修改的节点：sys/module/cam_debug_util/parameters/debug_mdl
* 要设置的值，也是每个bit对应一个模块，具体对应关系如下：
    ```c
    #define CAM_CDM        (1 << 0)
    #define CAM_CORE       (1 << 1)
    #define CAM_CPAS       (1 << 2)
    #define CAM_ISP        (1 << 3)
    #define CAM_CRM        (1 << 4)
    #define CAM_SENSOR     (1 << 5)
    #define CAM_SMMU       (1 << 6)
    #define CAM_SYNC       (1 << 7)
    #define CAM_ICP        (1 << 8)
    #define CAM_JPEG       (1 << 9)
    #define CAM_FD         (1 << 10)
    #define CAM_LRME       (1 << 11)
    #define CAM_FLASH      (1 << 12)
    #define CAM_ACTUATOR   (1 << 13)
    #define CAM_CCI        (1 << 14)
    #define CAM_CSIPHY     (1 << 15)
    #define CAM_EEPROM     (1 << 16)
    #define CAM_UTIL       (1 << 17)
    #define CAM_HFI        (1 << 18)
    #define CAM_CTXT       (1 << 19)
    #define CAM_OIS        (1 << 20)
    #define CAM_RES        (1 << 21)
    #define CAM_MEM        (1 << 22)
    #define CAM_IRQ_CTRL   (1 << 23)
    #define CAM_REQ        (1 << 24)
    #define CAM_PERF       (1 << 25)
    #define CAM_HYP        (1 << 26)
    #define CAM_IR_LED     (1 << 27)
    ```
* 比如要打开CAM_SENSOR模块的log，只需要设置
`adb shell "echo 32 > sys/module/cam_debug_util/parameters/debug_mdl"`
但是要注意，这样设置重启后会失效；
* 如果要重启后依然有效，需要修改cam_debug_util.c文件中，debug_mdl初始值修改为32，然后编译一个bootimage镜像，刷入系统后即可生效。

## 5.扩展
* 生效机制：
camxsettings.xml中原生节点如下：
```xml
 <setting>
     <Name>Output Format</Name>
     <Help>Output format for IPE Preview and Video output</Help>
     <VariableName>outputFormat</VariableName>
     <VariableType>OutputFormatType</VariableType> 
     <SetpropKey>persist.vendor.camera.outputFormat</SetpropKey>
     <DefaultValue>UBWCNV12</DefaultValue>
     <Dynamic>FALSE</Dynamic>
     <Public>TRUE</Public>
 </setting>
```
g_camxsettings.cpp中的关键函数如下，对应节点会被编译成如下代码：
```c
// 处理字符串型数据的函数
CamxResult SettingsManager::LoadOverrideSettings(
    IOverrideSettingsStore* pOverrideSettingsStore)
{
    // 该函数被调用时，所有流程都会被调用到，所以不存在不会更新的情况
    pOverrideSettingsStore->ReadSettingEnum(
        StringHashOutputFormat,
        reinterpret_cast<INT*>(&m_pStaticSettings->outputFormat),
        OutputFormatTypeEnumeratorToHashMap,
        static_cast<UINT>(CAMX_ARRAY_SIZE(OutputFormatTypeEnumeratorToHashMap)));
    ...
}
```

```c
// 处理persist节点的函数
CamxResult SettingsManager::LoadOverrideProperties(
    IOverrideSettingsStore* pOverrideSettingsStore,
    BOOL                    updateStatic)
{
    // 入参updateStatic在第一次调用是为true，以后更新的时候为false
    // 这样就导致在更新的时候下面的代码不会执行，从而不会动态更新。
    // 此处对应xml中的Dynamic这个节点，为FALSE会生成这个判断，为TRUE则不会。
    if (TRUE == updateStatic)
    {
        pOverrideSettingsStore->ReadSettingEnum(
            PropStringHashOutputFormat,
            reinterpret_cast<INT*>(&m_pStaticSettings->outputFormat),
            OutputFormatTypeEnumeratorToHashMap,
            static_cast<UINT>(CAMX_ARRAY_SIZE(OutputFormatTypeEnumeratorToHashMap)));

    }
    ...
}
```
