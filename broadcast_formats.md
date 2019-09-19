# Official Ruuvi Broadcast formats

RuuviTags are Bluetooth LE sensor beacons. While Bluetooth SIG has defined standard ways
to represent sensor data via GATT connection, there is no widely adopted standard for
broadcasting the sensor information in one-to-many configuration.

Advantages of broadcasting sensor data are simplicity, as there is no connection negotiation
required and density, as a single gateway can listen to hundreds of the sensors.

Disadvantages are that there is no guarantee or acknowledgement that a message is received
and theer is no protection against spoofing the data or MITM attacks.  

By default RuuviTags broadcast the sensor data in advertisement packet as a manufacturer
specific data. Some of the first versions of RuuviTags had the data encoded into 
an Eddystone URL, however since the Google has discontinued their Nearby notfications
the URL dataformats are depreciated.

The data formats are identified by the first byte of the advertisement payload. A typical
RuuviTag Bluetooth broadcast will contain: 
 - Preamble
 - Access Address
 - Advertising Channel Protocol Data Unit
   * MAC address of the tag
   * Flags of the advertisement
   * Ruuvi data
 - Cyclic redundancy checksum

For a detailed explanation on Bluetooth advertisement packets please refer to Bluetooth 
SIG [post](https://www.bluetooth.com/blog/bluetooth-low-energy-it-starts-with-advertising/).

Most scanners will return only Advertising channel PDU to the application, a typical advertised
value could be `0x0201061BFF99040505941A5BC7B1FFE0001C043867366F2497ED4DFAE75678`. This is
interpreted as:
 - `0x020106`
   * `0x02`: 1 byte length + 2 bytes payload for total of 3 bytes
   * `0x01`: Type Flags. 
   * `0x06`: Flag value: LE General Discoverable Mode, BR/EDR not supported
 - `0x1BFF99040505941A5BC7B1FFE0001C043867366F2497ED4DFAE75678`
   * `0x1B`: 1 byte length + 27 bytes payload for a total of 28 bytes
   * `0xFF`: Type Manufacturer Specific data
   * `9904`: Manufacturer: Ruuvi Innovations, Least Significant Byte first.
   * `0x05`: Header byte of Ruuvi data format, 0x05. 
   * `5941A5BC7B1FFE0001C043867366F2497ED4DFAE75678`: Data payload, described under section Data Format 5. 

It should be noted that Ruuvi data formats can be used in many different transport layers.
It is allowed to send other data alongside same advertisement packet using BLE5 extended
advertising or interleaving the advertisements with other data formats. Therefore parsers
cannot identify Ruuvi data by length of the BLE advertisement or blacklist devices sending
different data. The advertisements may be connectable or non-connectable. 

Correct way to parse the data is to identify if there is a manufacturer specific data field
from Ruuvi Innovations and then check if the header of the data payload matches Ruuvi data formats.

# Data format 1 
Lifecycle: Obsolete

Data format 1 used base91 encoding to pack data into URL parameter. However it wasn't
compatible with many browsers and was therefore discontinued

# Data format 2
Lifecycle: Obsolete
Data format 2 was used on Kickstarter devices. It is described in detail [here](./dataformat_04.md).
The data format was appended with a ID byte on data format 4, otherwise it is identical to data format 4. 

# Data format 3 (also known as RAWv1)
Lifecycle: In production, primary dataformat in 1.x branch firmware. Will be
depreciated in 2.x branch firmware. Primary data format in tags shipped from June 2018 onwards. 
Described in detail [here](./dataformat_03.md).

# Data format 4
Lifecycle: Depreciated, will be removed in 2.x branch firmware
Primary data format in RuuviTags shipped before June 2018. Described in detail [here](./dataformat_04.md).

# Data format 5 (also known as RAWv2)
Lifecycle: Beta testing, will be primary data format in 2.x branch firmware.
Starts shipping in Q4/2019. Described in detail

#Data format 0D (Encrypted environmental data)
Lifecycle: Proposal, will be implemented in 3.X once specification is agreed. 

# Unofficial Ruuvi data formats

If you wish to use your own data formats on RuuviTags, it is allowed to use 
Ruuvi Innovations Bluetooth SIG ID 0x04FF under following conditions:
 - The hardware is manufactured by Ruuvi Innovations
 - The first byte of data payload is in range of 0xF0 ... 0xFF