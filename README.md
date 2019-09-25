# Ruuvi sensor protocols
Communication interfaces between Ruuvi devices and ouside world are decribed here. 
For clarity, each section is marked with a lifecycle stage:
 - *Planned*: On the roadmap, but no concrete *proposal* for functionality exists yet
 - *Proposal*: A proposed method or format, not implemented yet but described for discussion and feedback.
 - *Alpha*: A proof of concept, implemented partially or completely for testing and feedback. May be abandoned, may be changed, may be promoted to *beta*. 
 - *Beta*: Considered useful and is going to be adopted to *production* unless something critical comes up in testing. Minor changes are possible.
 - *In production*: Ships in devices to customers.
 - *Deprecated*: Ships in devices to customers or there's a lot of devices using the format "in the wild". However it is going to be removed in a future releases. 
 - *Obsolete*: Used to ship, or an *alpha* which was abandoned. Mentioned only for completeness sake and if someone needs to support the format, should not be considered in new services. 

## Ruuvi as a sensor beacon
*Lifecycle: In production*

Ruuvi devices use BLE Advertisement as their primary method of transmitting information to users.
This is a one-to-many relationship, there is no pairing or connections being established. 

As the data is beaconed to the world, there is no guarantee that any single measurement is read.
Instead the data is broadcast often enough to let any receivers get updates often enough.

The broadcasted data formats are described in details on their [own page](./broadcast_formats.md).

## BLE GATT Connection
*Lifecycle: Alpha*

For some usecases the broadcasted data is not optimal. One such example is reading logs
off the devices. A log can have thousands of datapoints, and broadcasting them when there are
no listeners would waste energy and bandwidth. Larger blocks of data where reception of every data packet 
is important are transferred primarly via BLE GATT. 

Transport layer is described at the page [gatt communication](./gatt_communication.md) and the
application layer is described at the page [gatt formats](./gatt_formats.md).

## NFC
*Lifecycle: In production*

RuuviTags have NFC tag capablity. This is used for identifying a specific tag and returning
any sensitive information to user, as NFC read practically requires physicall access to the tag.
NFC has 4 UTF-8 encoded text fields:
 - id: 8 bytes which are cryptographically securely generated random numbers, used as a tag identifier.
 - ad: MAC address of the beacon.
 - SW: Firmware version, for example 2.5.7
 - dt: Other data, empty by default.

## Real-Time Transfer
*Lifecycle: Alpha*

Segger RTT is a serial protocol for transferring data out of microcontrollers with a wired connection.
It is relevant only for developers, and mentioned here only for completeness sake.
