## Data Format 5 Protocol Specification (RAWv2)
*Lifecycle: Beta testing* 

The data is decoded from "Manufacturer Specific Data" -field, for more details please check [this article](http://www.argenox.com/a-ble-advertising-primer/) out.
Manufacturer ID is 0x0499. 
The actual data payload is: 

Offset | Allowed values | Description
-------|:--------------:|-----------
0      | `5`            | Data format (8bit)
1-2    | `-32767 ... 32767` |Temperature in 0.005 degrees
3-4    | `0 ... 40 000`  | Humidity (16bit unsigned) in 0.0025% (0-163.83% range, though realistically 0-100%)
5-6    | `0 ... 65534` |   Pressure (16bit unsigned) in 1 Pa units, with offset of -50 000 Pa
7-8    | `-32767 ... 32767`  | Acceleration-X (Most Significant Byte first)
9-10   | `-32767 ... 32767`  | Acceleration-Y (Most Significant Byte first)
11-12  | `-32767 ... 32767`  | Acceleration-Z (Most Significant Byte first)
13-14  | `0 ... 2046`, `0 ... 30` | Power info (11+5bit unsigned), first 11 bits is the battery voltage above 1.6V, in millivolts (1.6V to 3.646V range). Last 5 bits unsigned are the TX power above -40dBm, in 2dBm steps. (-40dBm to +20dBm range)
15     | `0 ... 254`| Movement counter (8 bit unsigned), incremented by motion detection interrupts from accelerometer
16-17  | `0 ... 65534`| Measurement sequence number (16 bit unsigned), each time a measurement is taken, this is incremented by one, used for measurement de-duplication. Depending on the transmit interval, multiple packets with the same measurements can be sent, and there may be measurements that never were sent.
18-23  | `Any valid mac` | 48bit MAC address. 

_Not available_ is signified by largest presentable number for unsigned values, smallest presentable number for signed values and all bits set for mac. 

### Data field descriptions 
#### Temperature
Values supported:  (-163.835 °C to +163.835 °C in 0.005 °C increments.

_Example_ 

Value | Measurement
----|-----------
 `0x0000` | 0 °C
 `0x01C3` | +2.255 °C
 `0xFE3D` | -2.255 °C
 `0x8000` | Invalid / not available

#### Humidity
Values supported: 0.0 % to 100 % in 0.0025 % increments. Higher values than 100 % are 
possible, but they generally indicate a faulty or miscalibrated sensor. 

_Example_

Value | Measurement
----|-----------
 `000` | 0%
 `10010` | 25.050%
 `40000` | 100.0%
 `65535` | Invalid / not available

#### Atmospheric Pressure
Values supported: 50000 Pa to 115536 Pa in 1 Pa increments.

_Example_

Value | Measurement
----|-----------
 `00000` | 50000 Pa
 `51325` | 101325 Pa (average sea-level pressure)
 `65534` | 115534 Pa
 `65535` | Invalid / not available
 
#### Acceleration
Values supported: -32767 to 32767 (mG), however the sensor on RuuviTag supports only 16 G max (2 G in default configuration).
Values are 2-complement int16_t, MSB first. All channels are identical.

_Example_

Value | Measurement
----|-----------
 `0xFC18` | -1000 mG
 `0x03E8` | 1000 mG
 `0x8000` | Invalid / not available

#### Battery voltage
Values supported: 1600 mV to 3647 mV in 1 mV increments, practically 1800 ... 3600 mV.

_Example_

Value | Measurement
----|-----------
 `0000` | 1600 mV
 `1400` | 3000 mV
 `2047` | Invalid / not available

#### Tx Power
Values supported: -40 dBm to +22 dBm in 2 dBm increments.

_Example_

Value | Measurement
----|-----------
 `00` | -40 dBm
 `22` | +4 dBm
 `31` | Invalid / not available

#### Movement counter
Movement counter is one-byte counter which gets triggered when LIS2DH12 gives "activity interrupt". 
Sensitivity depends on the firmware, by default the sensitivity is a movement over 64 mG. 
The counter will roll over. Movement is deduced by "rate of change". Please note that the highest valid value is 254, and 255 is reserved for the "not available".

_Example_

Value | Measurement
----|-----------
 `00` | 0 counts
 `100` | 100 counts
 `255` | Invalid / not available

#### Measurement sequence number
Mesurement sequence number gets incremented by one for every measurement. 
It can be used to gauge signal quality and packet loss as well as to deduplicated data entries. You should note that the measurement sequence refers to data rather than transmission, so you might receive many transmissions with the same measurement sequence number.
Please note that the highest valid value is 65534, and 65535 is reserved for the "not available".

_Example_

Value | Measurement
----|-----------
 `00` | 0 counts
 `1000` | 1000 counts
 `65535` | Invalid / not available

## Test vectors
These test vectors are based on [ruuvitag-sensor](https://github.com/ttu/ruuvitag-sensor/tree/master/tests) project.
The tests are bidirectional, decode-encode results in original raw data. 
Encode-decode must result in same values with given precision, but floating point rounding differences may occur.

### Case: valid data
Raw binary data: `0x0512FC5394C37C0004FFFC040CAC364200CDCBB8334C884F`

Field | Value
------|------
Data format | `5`
Temperature | `24.3 C`
Pressure    | `100044`
Humidity    | `53.49 RH-%`
Acceleration X | `0.004 G`
Acceleration Y | `-0.004 G`
Acceleration Z | `1.036 G`
TX Power    | `4 dBm`
Voltage     | `2.977 V`
Movement counter | `66`
Measurement Sequence | `205`
MAC | `CB B8 33 4C 88 4F`

### Case: maximum values
Raw binary data: `0x057FFFFFFEFFFE7FFF7FFF7FFFFFDEFEFFFECBB8334C884F`

Field | Value
------|------
Data format | `5`
Temperature | `163.835 C`
Pressure    | `115534`
Humidity    | `163.8375 RH-%`
Acceleration X | `32.767 G`
Acceleration Y | `32.767 G`
Acceleration Z | `32.767 G`
TX Power    | `20 dBm`
Voltage     | `3.646 V`
Movement counter | `254`
Measurement Sequence | `65534`
MAC | `CB B8 33 4C 88 4F`

### Case: minimum values
Raw binary data: `0x05FFFE00000000FFFEFFFEFFFE0000000000CBB8334C884F`

Field | Value
------|------
Data format | `5`
Temperature | `-163.835 C`
Pressure    | `50000`
Humidity    | `0.000 RH-%`
Acceleration X | `-32.767 G`
Acceleration Y | `-32.767 G`
Acceleration Z | `-32.767 G`
TX Power    | `-40 dBm`
Voltage     | `1.600 V`
Movement counter | `0`
Measurement Sequence | `0`
MAC | `CB B8 33 4C 88 4F`

# Case: Invalid values
Raw binary data: `0x058000FFFFFFFF800080008000FFFFFFFFFFFFFFFFFFFFFF`

Field | Value
------|------
Data format | `5`
Temperature | `NAN`
Pressure    | `NAN`
Humidity    | `NAN`
Acceleration X | `NAN`
Acceleration Y | `NAN`
Acceleration Z | `NAN`
TX Power    | `NAN`
Voltage     | `NAN`
Movement counter | `NAN`
Measurement Sequence | `NAN`
MAC | `NAN`
