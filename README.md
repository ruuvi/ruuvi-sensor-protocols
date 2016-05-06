# Sensor Protocol for Eddystone-URL

This specification is designed to offer simple and flexible way to broadcast different kind of sensor values in an [Eddystone-URL](https://github.com/google/eddystone/tree/master/eddystone-url) frame. Possible usage scenario would be:

RuuviTag sensor beacon broadcasts a URL address in an Eddystone-URL frame: `http://ruu.vi#53WG3vW`. Once user visit the link, **ruu.vi** website decodes the value `53WG3vW` and shows the data in a human-readable format.

The Eddystone-URL frame broadcasts the URL using a compressed encoding format, but this specification is only about the sensor readings. 

## Protocol Specification (Data Format 0)

The data section of the URL can be encoded in the firmware of the beacon. The most powerful way to encode the data is Base94 because the URL of the Eddystone-URL has a support for 94 different characters.

The decoded value contains only characters (decimals) `0-9`. First decimal defines what kind of data the URL contains.

Offset | Possible value | Description
-----|:-----:|-----------
 0 | `0-9` | Data format definition (0 = current sensor readings)
 1 | `0-9` | Temperature (1st decimal)
 2 | `0-9` | Temperature (2nd decimal)
 3 | `0-9` | Temperature (3rd decimal)
 4 | `0-9` | Humidity (1st decimal)
 5 | `0-9` | Humidity (2nd decimal)
 6 | `0-9` | Humidity (3rd decimal)
 7 | `0-9` | Atmospheric pressure (1st decimal)
 8 | `0-9` | Atmospheric pressure (2nd decimal)
 9 | `0-9` | Atmospheric pressure (3rd decimal)
10 | `0-9` | Atmospheric pressure (4th decimal)
11 | `0-9` | Atmospheric pressure (5th decimal)

### Temperature
Values supported: -30°C - 69.9°C with 0.1°C increments.
####Example
Value | Measured humidity
----|-----------
 `000` | -30°C
 `999` | 69.9°C

### Humidity
Values supported: 0.0% - 99.9% with 0.1% increments.
####Example
Value | Measured humidity
----|-----------
 `000` | 0%
 `999` | 99.9%

### Atmospheric Pressure
Values (Pascal, Pa) supported: 30 000Pa - 110 000Pa with 1Pa increments.
####Example
Value | Measured humidity
----|-----------
 `00000` | 30 000 Pa
 `99999` | 110 000 Pa

### Data Format Decimal (Offset 0)
The first decimal is the most important one because it tells the receiver (ie. website) what kind of data the URL has. Only the first one `(0)` is implemented so far, rest of the choices are proposals.

Decimal | Description
----|-----------
 0 | Current sensor readings
 1 | Minimum sensor readings
 2 | Maximum sensor readings
 3 | Acceleration
 4 | Reserved for future use
 5 | Reserved for future use
 6 | Reserved for future use
 7 | Reserved for future use
 8 | Reserved for future use
 9 | Reserved for future use
