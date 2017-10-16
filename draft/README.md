# UAV Tracking Open Protocol

**Version:** draft

**Release date:** unreleased

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in
this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Approvals

| Organisation         | Contact         | Date       |
| -------------------- | --------------- | ---------- |
| Optimal Tracking     | F. Aberlenc     | 2017/10/12 |
| Hionos               | V. Brossard     | 2017/10/13 |
| Atechsys             | M. Kasbari      | 2017/10/13 |
| Atechsys             | B. Albouze      | 2017/10/13 |
| AirSpace Drone       | A. Bascoulergue | 2017/10/13 |
| AirSpace Drone       | W. Betinni      | 2017/10/13 |

## Lexical

- UAS: Unmanned Aerial System
- UAV: Unmanned Aerial Vehicle
- FEC: Forward Error Correction
- CRC: Cyclic Redundancy Check
- LBT: Listen Before Talk
- ERP: Effective Radiated Power
- GFSK: Gaussian Frequency-Shift Keying
- BT: Pulse shaping parameter for GFSK (filter bandwidth times symbol duration)


## Purpose

This document specifies an open radio UAV tracking protocol. This protocol is
intended to be used for broadcasting the UAV flight data in real-time. It can
either be used by UAV, by add-on modules attached to a UAV, by the UAV ground
station or by ground receivers and relays.


## License

Except otherwise stated, the content of this specification is in the public
domain, and code samples are licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0).

The use of this specification in products and code is entirely free: there are
no royalties, restrictions, or requirements.

Manufacturers implementing this specification must ensure that the product
conforms to all applicable national or international regulations.

## Constraints

- the payload MUST allow translation from and to the ASTERIX 129/29 format
- the radio specifications MUST allow compatibility with OGN and FLARM
  transmitters and receivers


## Radio Specifications

### Parameters


| Parameter | Value |
|---|---|
| Frequency | 868.4 MHz |
| Duty Cycle | 1% |
| Frequency Deviation | +-50 kHz |
| Bitrate | 100 kbps with Manchester encoding (effective 50 kbps) |
| Power | 25 mW ERP |
| Signal modulation | GFSK (BT=0.5) |
| Preamble | 10 bits **TO BE DEFINED** (without Manchester encoding) |
| Sync Word | 4 bytes **TO BE DEFINED** |


For the sake of simplicity there is no “Frequency Hopping”.

### FEC vs CRC checksum

The FEC enlarges the message payload and can lead to radio overflow in case of
many UAVs flying in the same area, so only 16 bits CRC has been chosen.

### Transmission Rules

#### Periodicity

When any of these two constraints has been met a new payload MUST be sent:

- time delay of 3 seconds
- spatial distance of 50 meters

#### Signature

An optional signature CAN be appended to the payload.

#### Transmission Rules

LBT MUST be honoured. If the channel is not free, a random waiting time MUST be
respected before next try.

#### Payload Relay

A receiver CAN relay a payload. The relay bit MUST be decremented by one
unit when relaying a payload.

If the relay bit is null, the payload MUST NOT be relayed.

#### Duty Cycle

The duty cycle is 1% and MUST be respected.

## Payload Specifications

The payload is divided in two parts: header and body.

### Coding Rules

The data is stored in binary mode. The storage is in little endian,
meaning that in a byte, the highest bit is written first and the lowest
one at the end.

### Header

| block | description | type | bits | range |
| ----- | ----------- | ---- | ---- | ----- |
| HEADER | Protocol Identifier | Integer | 4 | 0-15 |
| HEADER | Protocol Version | Integer | 4 | 0-15 |


#### Protocol Identifier

- Description: define the data structure specification used in the body
- Format: unsigned integer
- Length: 4 bits
- Values:
  - 0: UAV Open Tracking Protocol (this protocol)


#### Protocol version

- Description: version of the protocol used
- Format: unsigned integer
- Length: 4 bits
- Values:
  - 0: this protocol version


### Body


| block | description | type | bits | range | unit |
| ----- | ----------- | ---- | ---- | ----- | ---- |
| ID | Address | Integer | 32 | | N/A |
| TIME | Time of Day | Integer | 17 | 0 — 86400 | 1 second |
| POSITION | Latitude | Integer | 25 | -90 — 90 | 180°/2^24 |
| POSITION | Longitude | Integer | 25 | -180 — 180 | 360°/2^24 |
| POSITION | Altitude | Integer | 14 | -1000 — 15000 | 1 meter |
| POSITION | GPS Horizontal Accuracy | Integer | 7 | 0 — 127 | 1 meter |
| POSITION | GPS Vertical Accuracy | Integer | 7 | 0 — 127 | 1 meter |
| POSITION | GPS Fix | Boolean | 1 | 0 — 1 | N/A |
| SPEED | Horizontal Speed | Integer | 8 | 0 — 255 | 1 meter/second |
| SPEED | Vertical Speed | Integer | 7 | -63 — 63 | 1 meter/second |
| SPEED | Heading | Integer | 9 | 0 — 359 | 1 degree |
| FLAGS | Relay Count | Integer | 2 | 0 — 3 | N/A |
| FLAGS | Urgency | Boolean | 1 | 0 — 1 | N/A |
| FLAGS | UAV Category | 3 bits | 3 | | N/A |
| CRC | CRC | String | 16 | | N/A |
| SIGNATURE | Signature | String | 256 | | N/A |


#### Address

Addresses should be be assigned to devices it can be during manufacturing process
in a similar way to assigning MAC addresses to network cards. Alternatively addresses
can be assigned in a similar way as ICAO transponder codes are issued.

Mapping from address to type of aircraft, serial number, owner and other should be stored
externally to limit size of radio packet size. Externally stored information should contain:

- **Manufacturer Identification for a UAV / Transmitter** - Identifier for the manufacturer, provided by the country authority
- **Model Identification for a UAV / Transmitter** - Model Identification /Identifier provided by the manufacturer
- **UAV Transmitter Serial Number** - Serial number for the device model
- **Country of Registration for a UAV / Transmitter - Country identifier


#### Time

- Description: UTC time of the day, in seconds
- Format: unsigned integer
- Length: 17 bits


#### Latitude

- Description: Latitude WGS-84
- Format: signed integer
- Unit: 180/2^24 degree. Precision below 1.20 meters
- Length: 25 bits

#### Longitude

- Description: Longitude WGS-84
- Format: signed integer
- Unit: 360/2^24 degree. Precision below 1.20 meters
- Length: 25 bits


#### Above Sea-Level Altitude

- Description: Altitude above sea level
- Format: unsigned integer
- Unit: 1 meter. The value is the exact altitude +1000.
- Length: 7 bits

This allows representing altitudes in a -1000m / 15382 range with a 1 meter
precision.


#### Horizontal GPS Accuracy

- Description: Horizontal Accuracy of the GPS fix.
- Format: unsigned integer
- Unit: 1 meter. Values range from 0 to 127 meter.
- Length: 7 bits


#### Vertical GPS Accuracy

- Description: Horizontal Accuracy of the GPS fix.
- Format: unsigned integer
- Unit: 1 meter. Values range from 0 to 127 meter.
- Length: 7 bits


#### GPS Fix

- Description: Indicates whether the GPS location is valid for 3D
- Format: boolean (0 means no valid position)
- Length: 1 bit


#### Horizontal Speed

- Description: Horizontal speed.
- Format: unsigned integer
- Unit: 1 meter/second. Speed range from 0 to 255 m/s with 1m/s accuracy.
- Length: 8 bits


#### Heading

- Description: Represents the angle between the horizontal displacement and the
  true North.
- Format: unsigned integer
- Unit: 1 degree. Values range from 0 to 359 with 1 degree accuracy
- Length: 9 bits


#### Vertical Speed
- Description: Vertical speed.
- Format: unsigned integer
- Unit: 1 meter/second. Speed range from -63 to 63 m/s with 1m/s accuracy.
- Length: 7 bits


#### Relay Count
- Description: Counter for relaying the payload. Count must be decremented before
  relaying. When the count is 0, payload MUST NOT be relayed.
- Format: unsigned integer. Values range from 0 to 3.
- Length: 2 bits

#### Urgency

- Description: Indicates whether the UAV is in urgency mode (connection lost
  with remote control, engine issue…)
- Format: Boolean  (0 means no urgency).
- Length: 1 bit


### UAV Category

- Description: UAV Category allowing to define the risk it represents
- Format: unsigned integer. Values range from 0 to 7 depending on risk level.
- Length: 3 bits


#### Signed Flag

- Description: Indicates whether a signature is appended to the payload.
- Format: Boolean  (0 means no signature)
- Length: 1 bit


#### CRC :

- Description: CRC for payload integrity check; computed on all the payload,
  except the preamble, the sync word and the signature.
- Format: String
- Length: 16 bits

\* _TODO: define algorithm to be used._

#### Signature

- Description: AES-CMAC signature of the payload (header + body); the secret key
  should have been given by the national administration; this is an optional
  field.
- Format: String
- Length: 256 bits


## Annexes

### EUROCONTROL ASTERIX part 29 category 129 conformity

Field matching table:

| ASTERIX Part 29 Category 129 | This protocol |
| ----------------------------- | ------------- |
| Data source | 00/00 for “airborn to ground” transmissions
| Data destination | unset for broadcast
| UAV Manufacturer Identifier | Manufacturer Identifier
| UAS Model Identifier | Model Identifier
| UAS Serial Number or Aircraft ID | Serial Number
| UAS country | Country Registration
| Time of Day | Time of Day
| Position in WGS-84 Coordinates - LATITUDE - LONGITUDE | Position in WGS-84 Coordinates - LATITUDE - LONGITUDE
| Altitude above Mean Sea Level (AMSL) | Altitude above Mean Sea Level (AMSL)
| GNSS Signal precision | GPS accuracy H m

#### Data source

`00/00` for “airborn to ground” transmissions.

#### Data destination

Unset for broadcast signal.

#### UAS Manufacturer Identifier

The string encoding/decoding must be applied.

#### UAS Model Identifier

The string encoding/decoding must be applied.

#### UAS Serial Number or Aircraft ID

A translation table MUST be defined by the manufacturer for the conversion if
needed.

#### UAS country

The string encoding/decoding must be applied.

#### Time of Day

Must be converted from 1 second unit to 1/128 seconds unit.

#### Position in WGS-84 Coordinates – LATITUDE – LONGITUDE

MUST be converted from degree/2\^24 to degree/2\^30

#### Altitude above Mean Sea Level (AMSL)

Value can be used as is.

#### GNSS Signal precision

Value can be used as is.


### Duty Cycle Demonstration

Let us define messages are sent once every three seconds.

1% duty cycle allows 30 ms channel occupation by message.

With 50 kbps bitrate, this allows a 1500 bits message, which is 187 bytes.

If the UAV is moving at 50 m/s, which is 180 km/h, the periodicity of the
transmissions is then 1 second, because of the spatial requirement. The duty
cycle will then limit the message length at 62 bytes.


### Channel Congestion Avoidance Demonstration

A network is said congested when 20% of the maximal usage is reached.

Within 3 seconds, it is only possible to send 20% * 3 * 50000 bits, namely 3750
bytes.

It allows for 100 UAVs to be active in the same area, if the payload is
37 bytes long and sent once every 3 seconds.
