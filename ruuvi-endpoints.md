# Introduction
As there are many different use cases which require different setup from RuuviTags,
we're building extensible and generic protocol for configuring and reading data from RuuviTag.
As always, any design decisions are compromises between complexity, reliablity and coverage for
features. 

We intend to keep the RuuviTags working as they were once they come to your doorstep:
Broadcasting URL with possibility of switching over to RAW mode with a press of button. 
For those of you who have special requirements, the upcoming protocol allows for configuration of
devices so that they'll better suit your needs without the need of adjusting firmware itself.

# Status of the protocol
The protocol is at proposal stage with and example (firmware)[https://lab.ruuvi.com/distribution-packages/frankfurt_demo_dfu.zip] and (reader)[https://github.com/ojousima/ruuvi-nodejs] for temperature and MAM messages available.
Changes to endpoints and data formats are possible. 

# Sensor interface
## Requirements
### Portability
The sensor interface should be portable to different data transmission layers, i.e. BLE advertisements, BLE mesh, BLE GATT connection, NFC and proprietary protocols.

To fill this requirement, sensor interface must:
  - Fit in smallest available data payload
  - Contain necessary metadata in payload, i.e. GATT characteristic cannot provide data about meaning of bytes as this would not translate to mesh

### Payload length

Payload length is limited to 11 bytes by BLE mesh advertisement single segment payload size.

### Transmission layer-specific transmissions
Some use cases require longer payloads and higher data rates. It is only sensible to use the maximum capacity of each
transmission layer when large amounts of data need to be transferred. Therefore the specification has 
reserved a method for sending data in special, transmission-layer specific formats.

## Structure
 Messaging is split into 11-byte packets to fit within BLE notification and BLE advertisement as
 well as a single BLE mesh segment payload.

 Each message can be transported through a common channel, such as Nordic Uart Service or BLE advertisement
 Therefore each message contains source and destination _endpoints_, which lets application know
 how to handle message and where to send responses.

 Source endpoint identifies the "source" of data, for example "environmental sensor".
 Destination endpoint identifies "sink" of data, for example "graphing service".
 Type identifies content of data, for example "int16_t numbers" or "sensor configuration message".

 Payload is 8 bytes.

```
typedef struct __attribute__((__packed__)){
 uint8_t destination_endpoint;
 uint8_t source_endpoint;
 uint8_t type;
 uint8_t payload[8];
}data_packet_t;
```

## Sensor configuration message
Sensor configuration message is one standard 8-byte payload.
After the sensor has been configured, sensor will start sampling, and possibly applies given DSP function to data.
Transmissions are sent at a defined rate. If logging is enabled, logs are filled at the rate of data transmissions.

Sampling means taking samples from sensor, however a sample rate of 25 Hz does not mean that application wakes up at 25 Hz. It's allowed to uses inbuilt FIFO of sensor to store samples and read them in one go.

Transmission can also be logging. Sensor readings are not synchronized to transmission unless transmissions are slaved to sample rate by writing "transmission rate = 251". 

0: sample rate
 - 0 to move into sleep mode,
 - 1 - 250 Hz, rounded down to nearest implemented rate.
 - 251 single measurement, and back to sleep.
 - 255 No change
1: transmission rate
 - 0 To stop transmission
 - 1 - 60 seconds
 - 61 - 120 1-60 minutes, linear.
 - 121 - 250 1-130 hours, linear.
 - 251 samplerate
 - 252 DSP time constant rate (i.e. low-pass time constant at 25 samples: 1 transmission / 25 samples)
 - 255 No change
2: resolution
 - bits
 - 251: minimum
 - 252: maximum 
 - 255 No change
3: scale
 - 1-250: max counts of SI unit, i.e. accelerometer: 8 -> 8G (regardless of resolution).
 - 251: minimum
 - 252: maximum
 - 255: No change
4: DSP function
 - 1: last 2: min 3: max 4: average 5: stdev 6: impulse 7: low-pass 8: high-pass
 - 255: No change
5: DSP parameter
 - 1-250 function dependent, typically time constant as a number of samples
 - 255: No change
6: target, Data is sent to target interfaces (including RAM and non-volatile storage). This is a bitmask, it's
   possible to select multiple targets at once.
 - transmission_target_stop        = 0,    // Do not transmit any data anywhere
 - transmission_target_ble_adv     = 1,    // Broadcast data as BLE adverisement
 - transmission_target_ble_gatt    = 2,    // Transmit data through BLE GATT
 - transmission_target_ble_mesh    = 4,    // Transmit data through BLE MESH  
 - transmission_target_proprietary = 8,    // Transmit data through proprietary protocol
 - transmission_target_nfc         = 16,   // Transmit data through NFC
 - transmission_target_ram         = 32,   // Store transmissions to RAM
 - transmission_target_flash       = 64,   // Store transmission to FLASH
 - transmission_target_timestamp   = 128,  // Add Timestamp (uses lot of memory / bandwidth)
 - transmission_target_no_change   = 255
7: Reserved for future use 

```
typedef struct __attribute__((__packed__)){
 uint8_t sample_rate;
 unit8_t transmission_rate;
 uint8_t resolution;
 uint8_t scale;
 uint8_t dsp_function;
 uint8_t dsp_parameter;
 uint8_t log;
 uint8_t reserved;
}configuration_payload_t;
```

### Examples

Setting up accelerometer to sample at 25 Hz, take average values and store to RAM every 60 seconds. Scale and resolution are maximum. The average will be average of 25*60 = 1500 samples.
Implementation is allowed to use running algorithm for averages to avoid storing every sample

Values for constants will be published in a separate list

destination_endpoint = ACCELEROMETER;
source_endpoint      = XX;  //Application decides / any
uint8_t type         = SENSOR_CONFIGURATION;
uint8_t payload[8]   = {25,       // 25 Hz sample rate
                        60,       // Once per 60 seconds transmission rate
                        252,      // Maximum resolution
                        252,      // Maximum scale
                        4,        // DSP function
                        XX,       // Any DSP parameter (255 for no change)
                        (2+32),   // Transmit over BLE GATT, Log to RAM.
                        XX};      // XX: Don't care / any reserved

Setting up environmental sensor to sample at 1 Hz, maximum resolution and scale, store latest sample to RAM every 60 seconds

destination_endpoint = ENVIRONMENTAL;
source_endpoint      = XX;        //Application decides / any
uint8_t type         = SENSOR_CONFIGURATION;
uint8_t payload[8]   = {1,        // 1 Hz sample rate
                        60,       // Once per 60 seconds transmission rate
                        252,      // Maximum resolution
                        252,      // Maximum scale
                        1,        // Latest sample
                        XX,       // Any DSP parameter (255 for no change)
                        32,       // Log to RAM.
                        XX};      // XX: Don't care / any reserved

## Actuator configuration message
Actuations are visible (or otherwise observable) actions device can do. For example light a led or sound a buzzer.

_Payload_
```
typedef struct __attribute__((__packed__)){
uint8_t  channel    //Predefined constant
uint8_t  duty_cycle // Ratio, value / 255. 255 -> full, 0 -> off
uint16_t rate       // Milliseconds per cycle, MSB first. 0 for constant on
uint16_t timeout    // Milliseconds until timeout, MSB first, 0 for for constant on
uint8_t  reserved0; 
uint8_t  reserved1;
}actuation_payload_t;
```

### Examples

Blink green led at 100 ms on, 900 ms off (10% dutycycle) at a rate of one blink per second for 10 seconds.

```
destination_endpoint = LED; 
source_endpoint      = XX;   //Application decides / any
uint8_t type         = ACTUATOR_CONFIGURATION;
uint8_t payload[8]   = {LED_GREEN, // Green channel
                        25,        // Duty cycle 25/255 ~= 10%
                        3,         // MSB of rate, 3 * 256 = 768
                        232,       // LSB of rate, 768 + 232 = 1 000
                        39,        // MSB of timeout 39 * 256 = 9 984
                        16,        // LSB of timeout 9984 + 16 = 10 000
                        XX,        // XX: Don't care / any reserved
                        XX};       // XX: Don't care / any reserved
```
### Acknowledgements
Data transmission, verifications and retries are handled by the link layer. Sensor does send confirmation message
to verify that received message was understood and accepted on application layer.

Source and destination endpoints of message are swapped. Type of message is "MESSAGE_ACKNOWLEDGEMENT" if
the original message was understood. If original message was of unknown type, type of message is "UNKNOWN_TYPE"

Each payload byte contains error code for the payload, 0 if configuration was understood and successfully applied,
code detailing error if the payload was not understood or applied. Error codes TBA.

Example reply to "accelerometer setup"

```
destination_endpoint = REPLY-TO; // Source endpoint of message which caused configuration change
source_endpoint      = ACCELEROMETER; // Destination endpoint of message which caused configuration change
uint8_t type         = MESSAGE_ACKNOWLEDGEMENT,
uint8_t payload[8]   = {0, // Ok
                        0, // Ok
                        0, // Ok
                        0, // Ok
                        0, // Ok
                        0, // Ok
                        0, // Ok
                        RESERVED};  // This byte is reserved : no change.
```
# Endpoints
 *   00-0F - reserved for broadcast types
 *   10-1F - Data flow control, acknowledgements, error / info / debug messages, reading logs
 *   20-2F - µC peripherals, i.e. NFC, GPIO, I2C, SPI, PWM, Flash, Battery voltage etc. 
 *   30-3F - Environmental, i.e. temperature, humidity, air quality, pressure
 *   40-4F - Activity: i.e acceleration, magnetometer, gyroscope, movement detector
 *   50-9F - Reserved for future standard uses
 *   A0-DF - Reserved for future non-standard uses (application specific)
 *   E0-FF - Reserved for bulk data transfers, non-standard communication packets allowed.
 ```