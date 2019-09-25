## Data Format 2 and 4 protocol specification (URL)
*Lifecycle: Deprecated*

The data is packed in [Eddystone URL](https://developers.google.com/beacons/eddystone) 
with a base of https://ruu.vi/# and 9 [URL-safe base64](https://en.wikipedia.org/wiki/Base64#URL_applications) characters. 
Example URL is https://ruu.vi/#BFwaAMjlQ.  This is decoded to `0x045c1a00c8e5`

Offset | Allowed values | Description
-----|:-----:|-----------
 0 | 4 | Data format definition (4 = current sensor readings)
 1 | `0 ... 200` | Humidity (one lsb is 0.5%, e.g. 128 is 64%)
 2 | `-127 ... 127, signed` | Temperature (MSB is sign, next 7 bits are decimal value)
 3 | `0 ... 0` | Temperature (fraction, 1/100.). Not used, reads always as 0.
 4 - 5| `0 ... 65535` | Pressure (Most Significant Byte first, value - 50kPa). Rounded to 1 hPa accuracy.
 6 | `0..255` | Random id of tag, only present in format 4. NOTE! because of the URL limitation, only 6 most significant bits will be readable. 
 
### Data field descriptions
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

### Tag ID (only on format 4)
Contains a single random base 64 character used to identify tag.