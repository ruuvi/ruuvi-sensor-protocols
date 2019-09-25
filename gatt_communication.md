# Ruuvi GATT Communication
*Lifecycle: Alpha*

Ruuvis present up to 3 GATT services, one of them BLE SIG standard and two Nordic Semiconductor -specific. 
The services are: 
 * Device Information Service (DIS) used for reading details such as firmware and hardware revisions of the device.
 * Device Firmware Update (DFU) for updating the firmware on Ruuvis. 
 * Nordic UART Service (NUS) for transferring custom data.

## Connecting to device
RuuviTags which are connectable will advertise themselves as _connectable_ and _scannable_.
The primary advertisement may contain anything or nothing, scan response has device name in format
"RuuviABCD" where ABCD is 2 last bytes of Mac address. The scan response has additionally UUID
of Nordic UART Service in either incomplete or complete list of 128-bit UUIDs. 

An example advertisement would be `0x0201041BFF99040512E26F65C78BFFF80004FFFC9D56002892DAB3673134BF11079ECADC240EE5A9E093F3A3B50100406E0A09527575766933344246`
where `0x0201041BFF99040512E26F65C78BFFF80004FFFC9D56002892DAB3673134BF` is the primary advertisement 
and `0x11079ECADC240EE5A9E093F3A3B50100406E0A09527575766933344246` is the scan response. 
Scan response is further divided into 
`0x11079ECADC240EE5A9E093F3A3B50100406E` which is the NUS UUID, LSB first and 
`0x0A09527575766933344246` which is ASCII for Ruuvi34BF.

Details of establishing connection are generally handled by your Bluetooth LE library, so
you don't need to worry about negotiating MTU, connection intervals, latencies etc. 

## Device Information Service
*Lifecycle: Alpha* 

Device Information Service on Ruuvi devices follows the BLE SIG [specification](https://www.bluetooth.org/docman/handlers/downloaddoc.ashx?doc_id=244369). 

The following characterisitcs are implemented:
 * Manufacturer Name String: "Ruuvi Innovations Ltd". 
 * Serial Number String: ID of the device, unique random 8 bytes. _Note: this might be removed or replaced for security reasons._
 * Hardware Revision String: For example "RuuviTag B". 
 * Firmware Revision String: For example "RuuviFW 3.26.1"

## Device Firmware Update
*Lifecycle: Alpha* 

The DFU Service has 2 stages: first stage is in application and it can be used to enter
the bootloader of the device via Buttonless DFU characteristic. Second stage is in bootloader
when the firmware can be updated via DFU Control Point and DFU Packet Characteristics. 

Nordic Infocenter describes the service and chraracteristics in detail. [First stage](https://infocenter.nordicsemi.com/index.jsp?topic=%2Fcom.nordic.infocenter.sdk5.v15.0.0%2Fservice_dfu.html) and [second stage](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v15.0.0/lib_dfu_transport_ble.html?cp=5_5_1_3_5_2_2).

The bootloader with BLE is *in production*, however buttonless entry to bootloader is at *alpha* stage and might be 
removed or locked down somehow due to security concerns.

## Nordic UART Service
*Lifecycle: Alpha* 

Nordic UART Service is used for communication with user application when there is no standard GATT service for the data.
The service details are described in detail in [Nordic Infocenter](https://infocenter.nordicsemi.com/index.jsp?topic=%2Fcom.nordic.infocenter.sdk5.v14.0.0%2Fble_sdk_app_nus_eval.html). 

The application layer on how to send and receive data is describet at [gatt formats page](./gatt_formats.mdß)
