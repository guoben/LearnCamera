# CamX架构之Vendortag

## 一 熟悉的模型

verdortag    

```c
struct VendorTagInfo
{
    VendorTagSectionData* pVendorTagDataArray; 
    UINT32                numSections;
};
```

```c
enum MetadataTagType
{
    AndroidCameraTag, ///< camera metadata tag
    CamXVendorTag,    ///< vendor tag
    CamXProperty      ///< property
};
```

```c
enum class MetadataPoolSection : UINT32
{
    Input   = 0x0800, 
    Usecase = 0x2000,
    // 0x3000 - Result Prop
    // 0x4000 - Internal Prop
    // 0x5000 - Usecase Prop
    // 0x6000 - DebugData Prop
    Static  = 0x7000,
};
```

```c
enum class PropertyGroup : UINT32
{
    Result    = 0x3000, ///< Properties found in the result pool
    Internal  = 0x4000, ///< Properties found in the internal pool
    Usecase   = 0x5000, ///< Properties found in the usecase pool
    DebugData = 0x6000, ///< Properties found in the debug data pool
};
```





---

## 二 重要的数据结构



---

## 三 新增一个Vendortag



---

## 四 访问并使用



---

## 五 作用域

