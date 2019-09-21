# GATT data format
*Lifecycle: alpha*
As there are many different use cases which require different setup from RuuviTags,
we're building extensible and generic protocol for configuring and reading data from RuuviTag.
As always, any design decisions are compromises between complexity, reliablity and coverage for
features. 

We intend to keep the RuuviTags working as they were once they come to your doorstep:
Broadcasting data in the advertisements. 
For those of you who have special requirements, the upcoming protocol allows for configuration of
devices so that they'll better suit your needs without the need of adjusting firmware itself.

## GATT Message Structure
The GATT messages have a format of 3 bytes of header + 8 bytes payload. This is to be as compatible as possible
with BLE Mesh which will have a header of C0...FF, FF, 04 and 8 bytes of payload. 

The first byte of the header is a `destination`, second byte is a `source` and the third byte is a `type` of the message.
Messages are routed to application first by their `destination` and then by their `type`. 

The payload is defined primarily by the `type` and secondarily by `source`.

## GATT communication flow
The user application acts as a _central_ and Ruuvi device acts as a _peripheral_
The _central_ is responsible for initiating the connection and registering to Nordic UART Service
TX Notifications as described in [GATT Communication page](./gatt_communication.md). 

Central sends queries and commands to the peripheral which then replies with acknowledgements
and requested data. Central may send write or read commands to peripheral, but peripheral 
always responds with a write command which tells the central about the state of the peripheral after the 
operation. 

Examples for data exchanges are in the sections below. 


## Supported and planned functionality

- Sensor configuration write/read: *planned*
- Sensor offset write/read: *planned*
- Sensor log configuration write/read: *planned*
- Sensor value write/read: *planned*
- Sensor log write/read: *alpha*

### Sensor configuration writing and reading.
*Lifecycle: planned*
Sensor configuration message is one standard 8-byte payload.
After the sensor has been configured, sensor will start sampling, and possibly applies given DSP function to data.

Sampling applies only to the sensor itself, the application will not read the data at new rate unless the application
is configured to do so. 

```
typedef struct __attribute__((__packed__)){
 uint8_t samplerate;
 uint8_t resolution;
 uint8_t scale;
 uint8_t dsp_function;
 uint8_t dsp_parameter;
 uint8_t mode;
 uint8_t reserved0;
 uint8_t reserved1;
}configuration_payload_t;
```

#### Example - TODO

### Sensor log write/read
*Lifecycle: alpha*
As of 9/2019 and firmware version 3.26.x logging interval and values are hardcoded. 
_[Demo](https://www.youtube.com/watch?v=mVeEGJYrvNA&feature=youtu.be)_
_Try it out: Install [RuuviFW 3.26.0](https://jenkins.ruuvi.com/job/ruuvi.firmware.c/30/artifact/targets/ruuvitag_b/armgcc/ruuvitag_b_armgcc_ruuvifw_test_v3.26.0_full.hex) and read the logs with [RuuviWebBLE](https://ojousima.github.io/ojousima.ruuviwebble.ts/)_

The log is read with command that has a header destination set to target value, source to any and type as a log read. The payload is two 32-bit timestamps, seconds since unix epoch. First timestamp is current time, and second time is the lower bound of log data timestamps. For example if the timestamps are 1567047917 and 1566047917 data from time between 2019-08-13 13:18 and 2019-08-29 03:05 is sent.
The logs are sent in format where header destination is the source of read command, source is the temperature and type is log write. Payload is 4 bytes of timestamp and 4 bytes of int32_t, where unit is dependent on value if being read. On temperature the unit is 0.01C, on humidity the unit is 0.01 RH-% and on pressure the unit is 1 Pa. _TODO: Use milli-SI unit for all values?_

When the log buffer is sent entirely, a special message with the entire payload set to 0xFF is sent. It should be noted that there is no way to send only “missing sections” of data from the middle of the logs, logs are always retrieved to the end of the stored data.

Constants for endpoints and associated units are listed at [Ruuvi Endpoints](./ruuvi_endpoints.md).

### Example

Device     | Header        | Payload              | Description
-----------|---------------|----------------------|-------------
Central    | 0x3A 3A 11    | 0x5D6740ED 5D57FEAD  | “To: Environmental. From: Environmental. Type: Read logs. Clock is 2019-08-29 03:05 now, start from 2019-08-13 13:18”
Peripheral | 0x3A 30 10    | 0x5D57FEAD 0000098D  | “To: Environmental. From: Temperature. Type: Write logs.  Value at 2019-08-13 13:18 24.45 C“
Peripheral | 0x3A 31 10    | 0x5D57FEAD 000010EE  | “To: Environmental. From: Humidity. Type: Write logs.  Value at 2019-08-13 13:18 43.34 RH-%“
Peripheral | 0x3A 32 10    | 0x5D57FEAD 00018623  | “To: Environmental. From: Pressure. Type: Write logs.  Value at 2019-08-13 13:18 99875 Pa“
Peripheral | .    | .  | Log entry
Peripheral | .    | .  | Log entry
Peripheral | .    | .  | Log entry
Peripheral | 0x3A 3A 10    | 0xFFFFFFFF FFFFFFFF  | “To: Environmental. From: Environmental. Type: Write logs. Value: No more data“
