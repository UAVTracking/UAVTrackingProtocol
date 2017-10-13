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

## Lexical

- UAV: Unmanned Aerial Vehicle
- FCE: Forward Correction Error
- CRC: Cyclic Redundancy Check
- LBT: Listen Before Talk


## Purpose

This document specifies an open radio UAV tracking protocol. This protocol is
intended to be used for broadcasting the UAV flight data in real-time. It can
either be used by UAV, by add-on modules attached to a UAV, by the UAV ground
station or by ground receivers and relays.

Although radio specifications are described, this protocol may be used to 
transfert indentification data through mobile network or any other IOT network.


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
| Frequency Deviation | +-50 Hz |
| Bitrate | 100 kbps with Manchester encoding (effective 50 kbps) |
| Power | 25 mW ERP |
| Signal modulation | GFSK (BT=0.5) |
| Preamble | 10 bits **TO BE DEFINED** (without Manchester encoding) |
| Sync Word | 4 bytes **TO BE DEFINED** |


For the sake of simplicity there is no “Frequency Hopping”.

### FCE vs CRC checksum

The FCE enlarges the message payload and can lead to radio overflow in case of
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
| ID | Manufacturer Identifier | String* | 18 | | N/A |
| ID | Model Identifier | String* | 18 | | N/A |
| ID | Serial Number | Integer | 24 | 0 — 16.000.000 | 1 |
| ID | Registration Country | Integer | 12 | N/A |
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

\* _The characters transposition algorithm is detailed in the annexes._


#### Manufacturer Identification for a UAV / Transmitter

- Description: Identifier for the manufacturer, provided by the country
  authority
- Format: 3 readable encoded characters (see annexes)
- Length: 18 bits
- Example value for “DJI”: `100100 101010 101001`


#### Model Identification for a UAV / Transmitter

- Description: Identifier provided by the manufacturer
- Format: 3 readable encoded characters (see annexes)
- Length: 24 bits


#### UAV Transmitter Serial Number

- Description: Serial number for the device model
- Format: 3 bits (16 millions of possible combinations). In case of
  alphanumerical serial number a matching table MUST be provided by
  the manufacturer.
- Length: 18 bits
- Example value for “SX1”: `110011 111000 010001`


#### Country of Registration for a UAV / Transmitter

- Description: Country identifier
- Format: 2 readable encoded characters (see annexes)
- Length: 12 bits
- Example value for France (“FR”): `100110 110010`


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


## String Encoding

A readable character used in any of the alphanumerical data is either a
capital letter `A-Z`, or a number `0-9` or the characters ` `, `-`, `_`.

Each of these characters has an ASCII code between 32 and 63 (included).
So, each of these character is stored with a code between 0 to 31, which
is the ASCII code minus 32. The storage of these character is then done
with 6 bits.

The mapping between the characters and the value is given in the
following table:

| DEC | HEX | BINARY |  CHAR  |  DESCRIPTION       |
| --- | --- | ------ | ------ | ------------------ |
| 0   | 0   | 0      |  Space |  **space**          |
| 1   | 1   | 1      |  !     |  exclamation mark   |
| 2   | 2   | 10     |  "     |  double quote       |
| 3   | 3   | 11     |  \#    |  number             |
| 4   | 4   | 100    |  \$    |  dollar             |
| 5   | 5   | 101    |  %     |  percent            |
| 6   | 6   | 110    |  &     |  ampersand          |
| 7   | 7   | 111    |  '     |  single quote       |
| 8   | 8   | 1000   |  (     |  left parenthesis   |
| 9   | 9   | 1001   |  )     |  right parenthesis  |
| 10  | 0A  | 1010   |  \*    |  asterisk           |
| 11  | 0B  | 1011   |  +     |  plus               |
| 12  | 0C  | 1100   |  ,     |  comma              |
| 13  | 0D  | 1101   |  -     |  minus              |
| 14  | 0E  | 1110   |  .     |  period             |
| 15  | 0F  | 1111   |  /     |  slash              |
| 16  | 10  | 10000  |  0     |  zero               |
| 17  | 11  | 10001  |  1     |  one                |
| 18  | 12  | 10010  |  2     |  two                |
| 19  | 13  | 10011  |  3     |  three              |
| 20  | 14  | 10100  |  4     |  four               |
| 21  | 15  | 10101  |  5     |  five               |
| 22  | 16  | 10110  |  6     |  six                |
| 23  | 17  | 10111  |  7     |  seven              |
| 24  | 18  | 11000  |  8     |  eight              |
| 25  | 19  | 11001  |  9     |  nine               |
| 26  | 1A  | 11010  |  :     |  colon              |
| 27  | 1B  | 11011  |  ;     |  semicolon          |
| 28  | 1C  | 11100  |  &lt;  |  less than          |
| 29  | 1D  | 11101  |  =     |  equality sign      |
| 30  | 1E  | 11110  |  &gt;  |  greater than       |
| 31  | 1F  | 11111  |  ?     |  question mark      |
| 32  | 20  | 100000 |  @     |  at sign
| 33  | 21  | 100001 |  A     | |
| 34  | 22  | 100010 |  B     | |
| 35  | 23  | 100011 |  C     | |
| 36  | 24  | 100100 |  D     | |
| 37  | 25  | 100101 |  E     | |
| 38  | 26  | 100110 |  F     | |
| 39  | 27  | 100111 |  G     | |
| 40  | 28  | 101000 |  H     | |
| 41  | 29  | 101001 |  I     | |
| 42  | 2A  | 101010 |  J     | |
| 43  | 2B  | 101011 |  K     | |
| 44  | 2C  | 101100 |  L     | |
| 45  | 2D  | 101101 |  M     | |
| 46  | 2E  | 101110 |  N     | |
| 47  | 2F  | 101111 |  O     | |
| 48  | 30  | 110000 |  P     | |
| 49  | 31  | 110001 |  Q     | |
| 50  | 32  | 110010 |  R     | |
| 51  | 33  | 110011 |  S     | |
| 52  | 34  | 110100 |  T     | |
| 53  | 35  | 110101 |  U     | |
| 54  | 36  | 110110 |  V     | |
| 55  | 37  | 110111 |  W     | |
| 56  | 38  | 111000 |  X     | |
| 57  | 39  | 111001 |  Y     | |
| 58  | 3A  | 111010 |  Z     | |
| 59  | 3B  | 111011 |  [     |  left square bracket |
| 60  | 3C  | 111100 |  \\    |  backslash |
| 61  | 3D  | 111101 |  ]     |  right square bracket |
| 62  | 3E  | 111110 |  ^     |  caret / circumflex |
| 63  | 3F  | 111111 |  \_    |  underscore |


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
