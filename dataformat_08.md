# Data Format 8 Protocol Specification (Encrypted data)
*Lifecycle: Proposal*

The goal of encrypted firmware is to protect users of Ruuvi Dongle and Ruuvi Node against script kiddie -level attackers
snooping on their environmental data or triggering false alerts such as "freezer is melting" by sending spoofed data. 

The encryption uses nRF52-builtin AES128 encryption in Elctronic Codebook (ECB) mode. Data to be encrypted is
temprature, humidity, pressure, voltage, TX power, measurement count and movement counts. Additionally to protect against replay attacks the encrypted data contains a 16-bit measurement sequence counter and unencrypted data contains a random 8-bit nonce.
All measurements where encrypted data and nonce are equal can be considred duplicates.

Data format has an unencrypted header, 16 bytes of AES-128 encrypted data, 1 byte nonce and 6 bytes long MAC address for iOS devices.

Offset | Allowed values | Description
-------|:--------------:|-----------
0      | `8`            | Data format.
1-2    | `-32767 ... 32767` |Temperature in 0.005 degrees.
3-4    | `0 ... 40 000`  | Humidity (16bit unsigned) in 0.0025% (0-163.83% range, though realistically 0-100%).
5-6    | `0 ... 65534` |   Pressure (16bit unsigned) in 1 Pa units, with offset of -50 000 Pa.
7-8    | `0 ... 2046`, `0 ... 30` | Power info (11+5bit unsigned), first 11 bits is the battery voltage above 1.6V, in millivolts (1.6V to 3.646V range). Last 5 bits unsigned are the TX power above -40dBm, in 2dBm steps. (-40dBm to +20dBm range). 
9-10   | `0 ... 65534`| Movement counter (8 bit unsigned), incremented by motion detection interrupts from accelerometer
11-12  | `0 ... 65534`| Measurement sequence number (16 bit unsigned), each time a measurement is taken, this is incremented by one, used for measurement de-duplication. Depending on the transmit interval, multiple packets with the same measurements can be sent, and there may be measurements that never were sent.
13-15  | `Any`| Reserved for future use.
16     | `0 ... 255` | CRC8, used to check for correct decryption.
17     | `0 ... 255` | Nonce. Added to encryption key.
18-23  | `Any valid MAC` | 48bit MAC address. 

The encryption key is formed from 64-bit tag ID, 8 bit encryption nonce and a static password with length of 16 bytes by appending
nonce to ID and XORing the 9 bytes with 9 first bytes of password

## Invalid values
If a value cannot be determined for any reason, a special invalid value is sent. 
For unsigned values the invalid value is largest presentable number, for example `0xFFFF`
and for signed values the invalud value is smallest presentable number, for example `0x8000`.
Invalid values should be treated as NULL, NAN, NONE or similar by the parser.

## Example

Data        | Value
------------|------
Temperature | 24.58 C
Humidity    | 40.54 RH-%
Pressure    | 100453 Pa
Battery     | 2.765 V
TX Power    | +4 dBm
Movement    | 15
Measurement | 6353
Reserved    | 0
Checksum    | [224](https://crccalc.com/?crc=13343F58C51548E6000F18D1000000&method=crc8&datatype=hex&outtype=hex)
Nonce       | 28
MAC         | 0xAABBCCDDEEFF

Keys        | Binary
------------|-------------------
ID          | 0x0011223344556677
Password    | 0x5275757669636f6d5275757669546167 _"RuuvicomRuuviTag"_

Unencrypted binary: 

DF | T    |  H |  P |B+TX|C   | M  | R    |CH|N | MAC
---|------|----|----|----|----|----|------|--|--|------------
08 | 1334 |3F58|C515|48E6|000F|18D1|000000|E0|1C|AABBCCDDEEDD

Encryption key:

Component | Binary
----------|------------------------------------------------
ID + nonce| 00 11 22 33 44 55 66 77 1C
password  | 52 75 75 76 69 63 6f 6d 52 75 75 76 69 54 61 67
Result    | 52 64 57 45 2d 36 09 1a 4e 75 75 76 69 54 61 67

Encrypted data: [`0x9E49ED7745032DF5D2CC6E2A3047207B`](http://extranet.cryptomathic.com/aescalc/index?key=52+64+57+45+2d+36+09+1a+4e+75+75+76+69+54+61+67&iv=00000000000000000000000000000000&input=13343F58C51548E6000F18D1000000E0&mode=ecb&action=Encrypt&output=)

Complete message:

DF | T  |  H |  P |B+TX|C   |M   | R    |CH|N | MAC
---|----|----|----|----|----|----|------|--|--|------------
08 |9E49|ED77|4503|2DF5|D2CC|6E2A|304720|7B|1C|AABBCCDDEEDD

## TODO: Test vectors
