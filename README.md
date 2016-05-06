## Idea

RuuviTag sensor beacon broadcasts Eddystone-URL frame. The URL looks like `http://ruu.vi#53WG3vW`. Once user clicks the link, ruu.vi website decodes the value `53WG3vW` and shows the sensor values in a human-readable format.

`53WG3vW -> 53442571350`

## Protocol Specification

The URL is encoded in the firmware of the beacon according these specs. The decoded value contains only characters `0-9` and is **minimum of 12 decimals** totally. More values can be added in the future.

Offset | Possible value | Description
-----|:-----:|-----------
 0 | `0-9` | 0 = current values
 1 | `0-9` | Temperature (1nd decimal)
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
The specification supports temperature values from -30.0 celcius to 69.9 celcius with 0.1 degree increments.
####Example: `000` = -30C and `999` = 69.9C

### Humidity
Humidity readings can be between 0.0% and 99.9% with 0.1% increments.
####Example: `000` = 0% and `999` = 99.9%

### Air pressure
Atmospheric pressure values (Pascal, Pa) can be between 30 000 and 110 000 with 1Pa increments.
####Example: `00000` = 30 000 Pa and `80000` = 110 000 Pa

### Altitude
The altitude can be calculated using temperature and air pressure values. For this reason we won't transfer the altitude data, but it can be shown on website anyways.

### First value (offset 0)
The first decimal tells the receiver (ie. website) what kind of data the URL has. Only the first one `(0)` is implemented, rest of the choices are proposals.

Decimal | Description
----|-----------
 0 | Current sensor readings
 1 | Minimum sensor readings
 2 | Maximum sensor readings
 3 | Acceleration
 4 | For future use
 5 | For future use
 6 | For future use
 7 | For future use
 8 | For future use
 9 | For future use

## License
All the files in this repository are licensed under Apache 2.0.

The encoder/decoder uses a code snippet called [ShortURL](https://github.com/delight-im/ShortURL).
