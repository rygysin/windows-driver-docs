---
title: The Filter Driver's Control Device Object
description: The Filter Driver's Control Device Object
keywords:
- control device objects WDK file system
- CDOs WDK file system
ms.date: 04/20/2017
---

# The Filter Driver's Control Device Object


## <span id="ddk_the_filter_drivers_control_device_object_if"></span><span id="DDK_THE_FILTER_DRIVERS_CONTROL_DEVICE_OBJECT_IF"></span>


Unlike a file system, which is required to create and use a named control device object (CDO), a file system filter driver is not required to have a CDO. If it does, this CDO, which can optionally be named, represents the filter driver to the system. Its role is to receive I/O requests from a user-mode application (or, less commonly, another kernel-mode driver), and to act on them appropriately.

Most file system filter drivers create and use a CDO. However, support for I/O requests on the CDO is optional. To provide this support, when the filter driver calls [**IoCreateDevice**](/windows-hardware/drivers/ddi/wdm/nf-wdm-iocreatedevice) to create the CDO, it must supply a device name for the object. The user-mode application can then obtain a handle to the named CDO by calling [**CreateFile**](/windows/win32/api/fileapi/nf-fileapi-createfilea), supplying the user-mode version of the device name.

For example, consider a hypothetical "MyLegacyFilter" kernel-mode driver. This driver can create a CDO with the name:

```cpp
\Device\MyLegacyFilter
```

and calls [**IoCreateSymbolicLink**](/windows-hardware/drivers/ddi/wdm/nf-wdm-iocreatesymboliclink) to link this name to an equivalent user-mode-visible name. This is done so that MyLegacyFilter's user-mode application can open a handle to the kernel-mode driver's CDO by supplying the name:

```cpp
\\.\MyLegacyFilter
```

when it calls [**CreateFile**](/windows/win32/api/fileapi/nf-fileapi-createfilea).

### <span id="types_of_i_o_requests_that_are_sent_to_the_filter_driver_s_control_dev"></span><span id="TYPES_OF_I_O_REQUESTS_THAT_ARE_SENT_TO_THE_FILTER_DRIVER_S_CONTROL_DEV"></span>Types of I/O Requests That Are Sent to the Filter Driver's Control Device Object

File system filter drivers are not required to support any I/O operations on the control device object (CDO). However, most filters permit the following types of I/O requests to be sent to the filter's CDO:

-   [**IRP\_MJ\_CREATE**](./irp-mj-create.md) (to open a handle to the target device object, and give that handle to a user application)

-   [**IRP\_MJ\_CLEANUP**](./irp-mj-cleanup.md) (to close a user-mode application's handle to the target device object)

-   [**IRP\_MJ\_CLOSE**](./irp-mj-close.md) (to close the last remaining open handle to the target device object)

-   [**IRP\_MJ\_DEVICE\_CONTROL**](./irp-mj-device-control.md), [**IRP\_MJ\_FILE\_SYSTEM\_CONTROL**](./irp-mj-file-system-control.md), or **FastIoDeviceControl** (to send private IOCTLs or FSCTLs to the filter driver)

Note that, unlike all other device objects that a file system filter driver creates, the CDO is not attached to a driver stack. No device objects are attached above or below the filter driver's CDO. Thus, for any I/O request it receives, the CDO can safely assume that it is the sole intended recipient. This is not true for filter device objects or file system CDOs. Accordingly, the CDO must eventually complete every IRP it receives. For fast I/O requests, it must return **TRUE** or **FALSE**.

 

