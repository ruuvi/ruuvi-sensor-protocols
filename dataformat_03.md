# Data Format 3 Protocol Specification (RAWv1)
*Lifecycle: Deprecated*. Use Data Format 5 instead.

The data is decoded from "Manufacturer Specific Data" -field, for more details please check [this article](http://www.argenox.com/a-ble-advertising-primer/) out.
Manufacturer ID is 0x0499. 
The actual data payload is: 

Offset | Allowed values | Description
-----|:-----:|-----------
 0 | 3 | Data format definition (3 = current sensor readings)
 1 | `0 ... 200` | Humidity (one lsb is 0.5%, e.g. 128 is 64%) Values above 100% indicate a fault in sensor.
 2 | `-127 ... 127, signed` | Temperature (MSB is sign, next 7 bits are decimal value)
 3 | `0 ... 99` | Temperature (fraction, 1/100.)
 4 - 5| `0 ... 65535` | Pressure (Most Significant Byte first, value - 50kPa)
 6-7 | `-32767 ... 32767, signed`  | Acceleration-X (Most Significant Byte first)
 8 - 9 | `-32767 ... 32767, signed`  | Acceleration-Y (Most Significant Byte first)
 10 - 11| `-32767 ... 32767, signed`  | Acceleration-Z (Most Significant Byte first)
 12 - 13| `0 ... 65535` | Battery voltage (millivolts). MSB First

## Data field descriptions 

### Data Format
The first byte tells the receiver (ie. website) what kind of type of data the packet has.
Please refer to the [Broadcast formats for details](./broadcast_formats)

### Temperature
Values supported: -127.99 °C to +127.99 °C in 0.01 °C increments.

_Example_ 

Value | Measurement
----|-----------
 `0x0000` | 0 °C
 `0x8145` | -1.69 °C
 `0x0145` | +1.69 °C

### Humidity
Values supported: 0.0 % to 100 % in 0.5 % increments.

_Example_

Value | Measurement
----|-----------
 `000` | 0%
 `128` | 64.0%
 `200` | 100.0%

### Atmospheric Pressure
Values supported: 50000 Pa to 115536 Pa in 1 Pa increments.

_Example_

Value | Measurement
----|-----------
 `00000` | 50000 Pa
 `51325` | 101325 Pa (average sea-level pressure)
 `65536` | 115536 Pa
 
### Acceleration
Values supported: -32000 to 32000 (mG), however the sensor on RuuviTag supports only 16 G max (2 G in default configuration).
Values are 2-complement int16_t, MSB first. All channels are identical.

_Example_

Value | Measurement
----|-----------
 `0xFC 0x18` | -1000 mG
 `0x03 0xE8` | 1000 mG

### Battery voltage
Values supported: 0 mV to 65536 mV in 1 mV increments, practically 1800 ... 3600 mV. 

## Test vectors
These test vectors are based on [ruuvitag-sensor](https://github.com/ttu/ruuvitag-sensor/tree/master/tests) project.
There is no specific value for invalid/not available sensor readings, it is suggested to send `0` if value is not available. 
The tests are bidirectional, decode-encode results in original raw data. 
Encode-decode must result in same values with given precision, but floating point rounding differences may occur.

### Case: valid data
Raw binary data: `0x03291A1ECE1EFC18F94202CA0B53`

Field | Value
------|------
Data format | `3`
Temperature | `26.3 C`
Pressure    | `102766`
Humidity    | `20.5 RH-%`
Acceleration X | `-1.000 G`
Acceleration Y | `-1.726 G`
Acceleration Z | `0.714 G`
Voltage     | `2.899 V`

### Case: maximum values
Raw binary data: `0x03FF7F63FFFF7FFF7FFF7FFFFFFF`

Field | Value
------|------
Data format | `3`
Temperature | `127.99 C`
Pressure    | `115535`
Humidity    | `127.5 RH-%`
Acceleration X | `32.767 G`
Acceleration Y | `32.767 G`
Acceleration Z | `32.767 G`
Voltage     | `65.535 V`

### Case: minimum values
Raw binary data: `0x0300FF6300008001800180010000`

Field | Value
------|------
Data format | `3`
Temperature | `-127.99 C`
Pressure    | `50000`
Humidity    | `0.0 RH-%`
Acceleration X | `-32.767 G`
Acceleration Y | `-32.767 G`
Acceleration Z | `-32.767 G`
Voltage     | `0.000 V`
