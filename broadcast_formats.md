# Official Ruuvi Broadcast Advertisement Formats

RuuviTags are Bluetooth LE sensor beacons. While Bluetooth SIG has defined standard formats
to represent sensor data via GATT connection, there is no widely adopted standard for
broadcasting the sensor information in one-to-many configuration. Therefore a custom representation of the data is required.

The data from various sensors on the RuuviTag are broadcast in a single advertisment.

A Disadvantage of this is that the transmitting beacon does not receive an acknowledgement 
that a message is received and cannot retransmit packets not received (this is similar to UDP). 
This is mitigated by having each packet transmitted multiple times.
A sequence number in the packet allows the receiver to detect recent duplicate as well as missing packets. 
An additional consideration is that the environmental information provided by the sensors changes
slowly compared to the transmittion interval, so several packets will contain the same values.
Depending on the use case, the values reported by the accelerometer 
may change faster than the transmission interval even without consideration for missing packets.

Advantages of broadcasting sensor data in advertisments are simplicity, there is no connection negotiation
required, and density, a single receiver can listen to many sensors. 
Additionaly multiple receivers, perhaps located at various locations, may overlap coverage and can recieve packets 
from the same transmitter and consolidate packets.

A typical RuuviTag Bluetooth broadcast will contain: 
 - Preamble
 - Access Address
 - Advertising Channel Protocol Data Unit
   * MAC address of the tag
   * Flags of the advertisement
   * Payload: Ruuvi data
 - Cyclic redundancy checksum

For a detailed explanation on Bluetooth advertisement packets please refer to Bluetooth 
SIG [post](https://www.bluetooth.com/blog/bluetooth-low-energy-it-starts-with-advertising/).

RuuviTags broadcast the data from all its sensors in a single advertisement packet as manufacturer
specific data. The format is identified by the first byte of the payload. 

Here is an example packet:
0201061BFF99040505941A5BC7B1FFE0001C043867366F2497ED4DFAE75678. 

This is interpreted as:
 - 020106
   * 02 : length  
   * 01 : Type Flags. 
   * 06 : Flag value: LE General Discoverable Mode, BR/EDR not supported
 - 1BFF9904
   * 1B : length 27 bytes payload
   * FF : Type Manufacturer Specific data
   * 9904 : Manufacturer: Ruuvi Innovations (Least Significant Byte first)
 - 05941A5BC7B1FFE0001C043867366F2497ED4DFAE7567
   * 05 : format as the first byte of the payload. 
   *  remaining bytes as described in the format specification.
   
Most scanners will return only Advertising channel PDU to the application.

If you need to parse the packet select only packets with Manufacturer of Ruuvi Innovations. 
Then examine the first byte of the manufacturer specific data for a matching Ruuvi data format.
It should be noted that Ruuvi data formats can be used in many different transport layers.
It is allowed to send other data alongside same advertisement packet using BLE5 extended
advertising or interleaving the advertisements with other data formats. Therefore parsers
cannot identify Ruuvi data by length of the BLE advertisement or blacklist devices sending
different data. The advertisements may be connectable or non-connectable. 


# Data format 5 (also known as RAWv2)
**Lifecycle: In production.**

RAWv2 is a primary data format in 2.x and 3.x firmware. It includes temperature, humidity, atmospheric pressure, X/Y/Z accelerometer, batteryVoltage, TX power, a movement counter, a sequence number and the MAC address.
It has been available for update since January 2019 and is included in tags shipped since October 2019. Format is described in detail [here](./dataformat_05.md).

# Data format 8 (Encrypted environmental data)
**Lifecycle: Proposal.**

Other formats do not provide protection against spoofing the data or Man-In-The-Middle attacks.
Encrypted data format will be based on format 5. Expeced to be implemented in 3.X firmware once specification is agreed on. 
Proposal is described in detail [here](./dataformat_08.md)


# Data format 3 (also known as RAWv1)
**Lifecycle: deprecated. (There are many deployed RuuviTags using this format).**

RAWv1 is a primary format in 1.x and 2.x version firmware. It includes temperature, humidity, atmospheric pressure, X/Y/Z accelerometer and battery fields.  
Only version 2.x RuuviTags allow the user to select between format 3 or 5 by tapping the B button.
RAWv1 was included in tags shipped from June 2018 until October 2019. 
This did not include the MAC address needed for applications running on Apple products. Described in detail [here](./dataformat_03.md).

# Data format 1 
**Lifecycle: Obsolete.**

Data format 1 used base91 encoding to pack data into URL parameter. However it wasn't
compatible with many browsers and was therefore discontinued.

# Data format 2
**Lifecycle: Obsolete.**

Data format 2 was used on Kickstarter devices. It is described in detail [here](./dataformat_04.md).
The data is encoded on Eddystone URL with a URL-safe base64 encoding and interpreted via server-side javascript
of the website of the URL. Google has discontinued their Nearby notfications and
the URL dataformats are depreciated.

# Data format 4
**Lifecycle: Obsolete.**

The data is encoded on Eddystone URL with a URL-safe base64 encoding and interpreted via server-side javascript
of the website of the URL which is no longer operating.
Format 4 was the primary data format in RuuviTags shipped before June 2018. It is described in detail [here](./dataformat_04.md).

# Unofficial Ruuvi data formats*

If you wish to use your own data formats on RuuviTags, your are allowed to use 
Ruuvi Innovations Bluetooth SIG ID 0x0499 under following conditions:
 - The hardware is manufactured by Ruuvi Innovations
 - The the format is in range of 0xF0 ... 0xFF
