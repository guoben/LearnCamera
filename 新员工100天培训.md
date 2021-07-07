# camx与chi两个模块相互调用

## camx->chi
* camxhal3entry.cpp

    定义了HAL的设备抽象接口，此接口为Andorid系统定义的标准接口，与平台无关。
    ```c
    // 打开
    static hw_module_methods_t g_hwModuleMethods =
    {
        CamX::open
    };

    // 操作
    static camera3_device_ops_t g_camera3DeviceOps =
    {
        .initialize                         = CamX::initialize,
        .configure_streams                  = CamX::configure_streams,
        .construct_default_request_settings = CamX::construct_default_request_settings,
        .process_capture_request            = CamX::process_capture_request,
        .dump                               = CamX::dump,
        .flush                              = CamX::flush,
    };

    // 关闭
    static HwDeviceCloseOps g_hwDeviceCloseOps =
    {
        CamX::close
    };
    ```

    ```c
    CAMX_VISIBILITY_PUBLIC camera_module_t HAL_MODULE_INFO_SYM =
    {
        .common =
        {
            .tag                = HARDWARE_MODULE_TAG,
            .module_api_version = CAMERA_MODULE_API_VERSION_CURRENT,
            .hal_api_version    = HARDWARE_HAL_API_VERSION,
            .id                 = CAMERA_HARDWARE_MODULE_ID,
            .name               = "QTI Camera HAL",
            .author             = "Qualcomm Technologies, Inc.",
            .methods            = &CamX::g_hwModuleMethods
        },
        .get_number_of_cameras  = CamX::get_number_of_cameras,
        .get_camera_info        = CamX::get_camera_info,
        .set_callbacks          = CamX::set_callbacks,
        .get_vendor_tag_ops     = CamX::get_vendor_tag_ops,
        .open_legacy            = CamX::open_legacy,
        .set_torch_mode         = CamX::set_torch_mode,
        .init                   = CamX::init
    };
    ```

* camxhal3.cpp
    此文件提供了一个函数接口列表，可以理解为中间层，将Andorid需要的接口转化为高通平台上CamX架构的具体实现。
    ```c
    JumpTableHAL3 g_jumpTableHAL3 =
    {
        open,
        get_number_of_cameras,
        get_camera_info,
        set_callbacks,
        get_vendor_tag_ops,
        open_legacy,
        set_torch_mode,
        init,
        get_tag_count,
        get_all_tags,
        get_section_name,
        get_tag_name,
        get_tag_type,
        close,
        initialize,
        configure_streams,
        construct_default_request_settings,
        process_capture_request,
        dump,
        flush,
        camera_device_status_change,
        torch_mode_status_change,
        process_capture_result,
        notify
    };
    ```

* camxhaldevice.cpp/camxhaldevice.h

    该文件中定义了HALDevice类，它的每个实例代表了一个具体的Camera设备；此文件中函数，对应了具体的设备操作；
    ```c
    // 主要的接口：
    Initialize
    ConfigureStreams
    ProcessCaptureRequest
    ProcessCaptureResult
    Notify
    ```

* camxhal3module.cpp
    这个文件定义了HAL3Module类，它的构造函数是整个camx架构的起点。
    在构造函数时，会打开com.qcom.override.so这个库，并拿到chi_hal_override_entry这个句柄，然后获得操作chi-cdk模块的函数指针。
    ```c
    CHIHALOverrideEntry funcCHIHALOverrideEntry =
        reinterpret_cast<CHIHALOverrideEntry>(
            CamX::OsUtils::LibGetAddr(m_hChiOverrideModuleHandle, 
                                      "chi_hal_override_entry"));
    ```
* chxextensioninterface.cpp
    此文件主要提供chi_hal_override_entry的具体实现
    ```c
    void chi_hal_override_entry(
        chi_hal_callback_ops_t* callbacks)
    {
        ExtensionModule* pExtensionModule = ExtensionModule::GetInstance();

        CHX_ASSERT(NULL != callbacks);

        if (NULL != pExtensionModule)
        {
            callbacks->chi_get_num_cameras              = chi_get_num_cameras;
            callbacks->chi_get_camera_info              = chi_get_camera_info;
            callbacks->chi_get_info                     = chi_get_info;
            callbacks->chi_initialize_override_session  = chi_initialize_override_session;
            callbacks->chi_finalize_override_session    = chi_finalize_override_session;
            callbacks->chi_override_process_request     = chi_override_process_request;
            callbacks->chi_teardown_override_session    = chi_teardown_override_session;
            callbacks->chi_extend_open                  = chi_extend_open;
            callbacks->chi_extend_close                 = chi_extend_close;
            callbacks->chi_remap_camera_id              = chi_remap_camera_id;
            callbacks->chi_modify_settings              = chi_modify_settings;
            callbacks->chi_get_default_request_settings = chi_get_default_request_settings;
            callbacks->chi_override_flush               = chi_override_flush;
            callbacks->chi_override_dump                = chi_override_dump;
        }
    }
    ```

---

## chi->cdk

* camxchi.cpp

    此文件定义了一个函数入口，可以通过此接口操作CamX架构中的函数。
    ```c
    CAMX_VISIBILITY_PUBLIC VOID ChiEntry(
        ChiContextOps* pChiContextOps)
    {
        if (NULL != pChiContextOps)
        {
            pChiContextOps->size                       = sizeof(ChiContextOps);

            pChiContextOps->majorVersion               = CHI_API_MAJOR_VERSION;
            pChiContextOps->minorVersion               = CHI_API_MINOR_VERSION;
            pChiContextOps->pOpenContext               = CamX::ChiOpenContext;
            pChiContextOps->pCloseContext              = CamX::ChiCloseContext;
            pChiContextOps->pGetNumCameras             = CamX::ChiGetNumCameras;
            pChiContextOps->pGetCameraInfo             = CamX::ChiGetCameraInfo;
            pChiContextOps->pEnumerateSensorModes      = CamX::ChiEnumerateSensorModes;
            pChiContextOps->pCreatePipelineDescriptor  = CamX::ChiCreatePipelineDescriptor;
            pChiContextOps->pDestroyPipelineDescriptor = CamX::ChiDestroyPipelineDescriptor;
            pChiContextOps->pCreateSession             = CamX::ChiCreateSession;
            pChiContextOps->pDestroySession            = CamX::ChiDestroySession;
            pChiContextOps->pFlushSession              = CamX::ChiFlushSession;
            pChiContextOps->pActivatePipeline          = CamX::ChiActivatePipeline;
            pChiContextOps->pDeactivatePipeline        = CamX::ChiDeactivatePipeline;
            pChiContextOps->pSubmitPipelineRequest     = CamX::ChiSubmitPipelineRequest;
            pChiContextOps->pQueryPipelineMetadataInfo = CamX::ChiQueryPipelineMetadataInfo;
            pChiContextOps->pTagOps                    = CamX::ChiGetTagOps;
            pChiContextOps->pGetFenceOps               = CamX::ChiGetFenceOps;
            pChiContextOps->pMetadataOps               = CamX::ChiGetMetadataOps;
            pChiContextOps->pGetBufferManagerOps       = CamX::ChiGetBufferManagerOps;
        }

        CamX::g_vendorTagOps.get_all_tags     = CamX::ChiGetAllTags;
        CamX::g_vendorTagOps.get_section_name = CamX::ChiGetSectionName;
        CamX::g_vendorTagOps.get_tag_count    = CamX::ChiGetTagCount;
        CamX::g_vendorTagOps.get_tag_name     = CamX::ChiGetTagName;
        CamX::g_vendorTagOps.get_tag_type     = CamX::ChiGetTagType;

        set_camera_metadata_vendor_ops(&(CamX::g_vendorTagOps));
    }
    ```
* chxextensionmodule.cpp
    此文件中定义了ExtensionModule类，它最主要的作用是打开camera.qcom.so获取ChiEntry句柄，用来操作CamX模块
    ```c
    PCHIENTRY funcPChiEntry = reinterpret_cast<PCHIENTRY>(ChxUtils::LibGetAddr(handle, "ChiEntry")    
    ```
* 
