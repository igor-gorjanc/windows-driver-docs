---
title: Creating WDF HID Minidrivers
description: This topic describes how to create a Human Interface Device (HID) minidriver using Windows Driver Frameworks (WDF).
ms.assetid: 4FEDFE4B-F3B2-4B34-80DC-84BFFA4C612B
---

# Creating WDF HID Minidrivers


This topic describes how to create a Human Interface Device (HID) minidriver using Windows Driver Frameworks (WDF).

You can write a HID minidriver using either KMDF or UMDF. We recommend starting with the vhidmini2 minidriver sample. You can compile this sample driver using either KMDF or UMDF 2.x.

**What to provide**

1.  You'll write a lower filter driver under *MsHidUmdf.sys* (for UMDF) or *MsHidKmdf.sys* (for KMDF), both of which are included as part of the operating system.
2.  Download and review the [vhidmini2 sample](https://github.com/Microsoft/Windows-driver-samples/tree/master/hid/vhidmini2).

3.  Call [**WdfFdoInitSetFilter**](https://msdn.microsoft.com/library/windows/hardware/ff547273) from the driver's [*EvtDriverDeviceAdd*](https://msdn.microsoft.com/library/windows/hardware/ff541693) callback function.

4.  Create I/O queues to receive I/O requests that *MsHidUmdf.sys* or *MsHidKmdf.sys* pass from the class driver to your driver.

5.  Provide an [*EvtIoDeviceControl*](https://msdn.microsoft.com/library/windows/hardware/ff541758) callback function that branches to IOCTL-specific method handlers. Review the IOCTLs described in [WDF HID Minidriver IOCTLs](https://msdn.microsoft.com/library/windows/hardware/hh463977) and ensure that your driver handles the relevant ones for your device.
6.  For UMDF, if your driver is enumerated by ACPI, optionally enable selective suspend. In the device's hardware key, add a **EnableDefaultIdleNotificationHandler** subkey and set it to 1.
7.  For UMDF, set the following [INF directives](specifying-wdf-directives-in-inf-files.md) in a WDF-specific *DDInstall* section of your INF file:

    -   **UmdfKernelModeClientPolicy** to **AllowKernelModeClients** so that the kernel-mode pass-through driver can be loaded in the stack.
    -   **UmdfMethodNeitherAction** to **Copy** to allow UMDF to process IOCTLs of METHOD\_NEITHER type.
    -   **UmdfFileObjectPolicy** to **AllowNullAndUnknownFileObjects**
    -   **UmdfFsContextUsePolicy** to **CanUseFsContext2**

    For example:

    ```
    [hidumdf.NT.Wdf]
    UmdfKernelModeClientPolicy = AllowKernelModeClients
    UmdfMethodNeitherAction=Copy
    UmdfFileObjectPolicy=AllowNullAndUnknownFileObjects
    UmdfFsContextUsePolicy = CanUseFsContext2
    ```

If you are writing a UMDF HID minidriver for Windows 7, download [Windows Driver Kit (WDK) 8.1](https://go.microsoft.com/fwlink/p/?LinkId=733614) to obtain source code for *HidUmdf.sys*. Then, write a UMDF 1.11 driver and include *HidUmdf.sys* and UMDF 1.11 in your driver package.

## Architecture


The HID class driver (*HidClass.sys*) and the framework provide conflicting WDM dispatch routines to handle some I/O requests (such as Plug and Play and power management requests) for minidrivers. As a result, a HID minidriver cannot link to both the class driver and the framework. Therefore, Microsoft provides *MsHidUmdf.sys* and *MsHidKmdf.sys*, which are WDM drivers that reside between the class driver and the minidriver.

Both *MsHidUmdf.sys* and *MsHidKmdf.sys* call the HID class driver's [**HidRegisterMinidriver**](https://msdn.microsoft.com/library/windows/hardware/ff539835) routine to register as the actual HID minidriver. Although these drivers act as the device's function driver, they just pass I/O requests from the class driver to your driver (and are thus sometimes called *pass-through drivers*). For both KMDF and UMDF, the only component that you supply is the HID minidriver, which is a lower filter driver that sits under the pass-through driver.

|                                                                                     |                                                                                        |
|-------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------|
| UMDF architecture                                                                   | KMDF architecture                                                                      |
| ![location of hidumdf.sys in the driver stack](images/umdf-basedhidminidrivers.png) | ![location of mshidkmdf.sys in driver stack](images/framework-basedhidminidrivers.png) |

 

 

 

[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20%5Bwdf\wdf%5D:%20Creating%20WDF%20HID%20Minidrivers%20%20RELEASE:%20%284/5/2016%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/default.aspx. "Send comments about this topic to Microsoft")



