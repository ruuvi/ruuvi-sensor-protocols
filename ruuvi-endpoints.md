# Ruuvi Endpoints
*Lifecycle: alpha*

A Ruuvi Endpoint describes a single target or source of data. It might be a
sensor value, such as temperature, actuator value, such as GPIO high/low, configuration 
value such as radio transmit power or iBeacon UUID etc. Endpoints are represented with one
unsigned byte.

Different endpoints accept different types of Ruuvi messages. For example temperature
endpoint will accept sensor configuration, offset write and log read commands while
radio endpoint would accept TX power configuration. 

Here is an authoritative listing of endpoints on Ruuvi devices. All implementations
should follow these definitions for interoperability. 

Endpoint        | identifier | Description               | Unit for value | Lifecycle
----------------|------------|---------------------------|----------------|----------
Battery voltage | 0x20       | Voltage of device battery | 0.001 V        | Proposal
Temperature     | 0x30       | Temperature of device     | 0.01  C        | Alpha
Humidity        | 0x31       | Humidity of device        | 0.01  RH-%     | Alpha
Pressure        | 0x32       | Pressure of device        | 1  Pa          | Alpha
Environmental   | 0x3A       | All environmental values  | None           | Alpha