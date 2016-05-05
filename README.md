## Idea

RuuviTag sensor beacon broadcasts Eddystone-URL frame. The URL looks like `http://ruu.vi#wCY5Nd7`. Once user clicks the link, ruu.vi website decodes the value `wCY5Nd7` and shows the sensor values in a human-readable format.

`wCY5Nd7 -> 432635101256`

## Protocol Specification

The URL is encoded in the firmware of the beacon according these specs. The decoded value contains only characters `0-9` and is **minimum of 12 characters** totally. More values can be added in the future.

Byte offset | Value | Description
-----|:-----:|-----------
 0 | `0-9` | Temperature (1st value)
 1 | `0-9` | Temperature (2nd value)
 2 | `0-9` | Temperature (3rd value)
 3 | `0-9` | Humidity (1st value)
 4 | `0-9` | Humidity (2nd value)
 5 | `0-9` | Humidity (3rd value) 
 6 | `0-9` | Atmospheric pressure (1st value)
 7 | `0-9` | Atmospheric pressure (2nd value)
 8 | `0-9` | Atmospheric pressure (3rd value)
 9 | `0-9` | Atmospheric pressure (4th value)
10 | `0-9` | Atmospheric pressure (5th value)
11 | `0-9` | Atmospheric pressure (6th value)

### Temperature
The specification supports temperature values from -30.0 celcius to 69.9 celcius with 0.1 degree increments.
**Example: `000` = -30C and `999` = 69.9C**

### Humidity
Humidity readings can be between 0.0% and 99.9% with 0.1% increments.
**Example: `000` = 0% and `999` = 99.9%**

### Air pressure
Atmospheric pressure values (Pascal, Pa) can be between 30 000 and 110 000 with 1Pa increments.
**Example: `000000` = 30 000 Pa and `800000` = 110 000 Pa**

### Altitude

The altitude can be calculated using temperature and air pressure values. For this reason we won't transfer the altitude data, but it can be shown on website anyways.

## License

All the files in this repository are licensed under Apache 2.0.

The encoder uses a code snippet called [ShortURL](https://github.com/delight-im/ShortURL) licensed under Apache 2.0 which is not a copyleft license.
