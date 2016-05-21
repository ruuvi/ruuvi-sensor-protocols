# Sensor Protocol for Eddystone-URL

<img src="https://github.com/google/eddystone/blob/master/branding/assets/png/EddyStone_final-01.png" alt="Eddystone logo" width="300px" align="middle">

This specification is designed to offer simple and flexible way to broadcast different type of sensor values in an [Eddystone-URL](https://github.com/google/eddystone/tree/master/eddystone-url) frame. One possible usage scenario would be:

> [RuuviTag](http://ruuvitag.com) sensor beacon broadcasts an encoded URL address in an Eddystone-URL frame: `http://ruu.vi#53WG3vW`. Once user visits the link, **ruu.vi** website decodes the value `53WG3vW` and shows the data in a human-readable format.

[![Ruuvi Measurements](https://github.com/ruuvi/sensor-protocol-for-eddystone-url/blob/master/images/website.png)](http://ruuvi.com)

The data part of the URL can be encoded in firmware of the beacon. The most powerful way to encode the data is Base94 because the URL field of the Eddystone-URL has a support for 94 different characters. Normally it's mandatory to encode the data because of maximum length (18 characters) of the [Eddystone-URL](https://github.com/google/eddystone/tree/master/eddystone-url) frame's URL field.

## Protocol Specification (Data Format 0)

The decoded data is a list of decimal (0-9) characters. First number defines what kind of data the field contains.

Offset | Allowed values | Description
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
12 | `0-9` | Time format (0=sec, 1=min, 2=hours, 3=days)
13 | `0-9` | Time elapsed after last movement (1st decimal)
14 | `0-9` | Time elapsed after last movement (2nd decimal)
15 | `0-9` | Time elapsed after last movement (3rd decimal)

### Temperature
Values supported: -30°C to +69.9°C in 0.1°C increments.
####Example
Value | Measurement
----|-----------
 `000` | -30°C
 `550` | +25°C
 `999` | +69.9°C

### Humidity
Values supported: 0.0% to 99.9% in 0.1% increments.
####Example
Value | Measurement
----|-----------
 `000` | 0%
 `350` | 35.5%
 `999` | 99.9%

### Atmospheric Pressure
Values supported: 10000Pa to 109999Pa in 1Pa increments.
####Example
Value | Measurement
----|-----------
 `00000` | 10000Pa
 `91325` | 101325Pa (average sea-level pressure)
 `99999` | 109999Pa
 
### Time elapsed after last movement
Values supported: 0 sec to 999 days in sec/min/hour/day increments.
####Example
Value | Measurement
----|-----------
 `0000` | 0 secondes
 `0573` | 573 seconds
 `1141` | 141 minutes
 `2038` | 38 hours
 `3146` | 146 days

### Data Format Decimal (Offset 0)
The first decimal is the most important one because it tells the receiver (ie. website) what kind of type of data the URL has. Only the first one `(0)` is implemented so far, rest of the choices are preliminary proposals.

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
