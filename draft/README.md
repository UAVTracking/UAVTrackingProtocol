# UAV Tracking Open Protocol

Version: draft
Release date: —

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in
this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Approvals

| Organisation         | Contact     | Date       |
| -------------------- | ----------- | ---------- |
| OPTIMAL TRACKING     | F. ABERLENC | 2017/10/12 |

## Lexical

- UAV: unmanned aerial vehicle
- FCE: Forward Correction Error
- CRC: Cyclic Redundancy Check
- LBT: Listen Before Talk


## Purpose

This document specifies an open radio UAV tracking protocol. This protocol is
intended to be used for broadcasting the UAV flight data in real time.


## License

Except otherwise stated, the content of this specification is in the public
domain, and code samples are licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0).

The use of this specification in products and code is entirely free: there are no royalties, restrictions, or requirements.

Manufacturers implementing this specification must ensure that the products
conform to all applicable national or international regulations.

## Radio Specifications

### Parameters

| Band Range | European ISM Band H1.4 |
| Frequency | 868.4 MHz |
| Duty Cycle | 1% |
| Frequency Deviation | +-50 Hz |
| Bitrate | 100 kbps with Manchester encoding (effective 50 kbps) |
| Power | 25 mW ERP |
| Signal modulation | GFSK (BT=0.5) |
| Preamble | 10 bits **TO BE DEFINED** (without Manchester encoding) |
| Sync Word | 4 bytes **TO BE DEFINED** |

For the sake of simplicity there is not «Frequency Hopping».

### FCE vs CRC checksum

The FCE enlarge the message payload and can lead to radio overflow in case of many UAVs flying in the same area, so only 16 bits CRC has been chosen.

## Transmission Rules

### Periodicity

When any of those two constraints has been met a new payload MUST be sent:

- time delay of 3 seconds
- spatial distance of 50 meters

### Signature

An optional signature CAN be appended to the payload.

### Transmission Rules

LBT MUST be honoured. If the channel is not free, a random waiting time MUST be
respected before next try.

### Payload Relay

A receiver SHOULD relay a payload. The relay bit MUST be decremented by one unit when relaying a payload.

If the relay bit is null, the payload MUST NOT be relayed.

### Duty Cycle

The Duty Cycle is 1%.

## Payload Specifications

The payload is divided in two parts: header and body.

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
  - 0 this protocol version


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


### Identifiant du fabricant de l’UAV ou de l’émetteur :

-   Signification : Cet identifiant composé de 3 caractères lisibles est
    alloué par un organisme national ou supranational.

-   Format : 3 caractères lisibles encodés (voir annexes)

-   Longueur : 18 bits

-   Example :

    -   DJI : 100100 101010 101001

### Identifiant du modèle d’UAV ou de l’émetteur :

-   Signification : Identifiant alloué par le fabricant.

-   Format : 3 caractères lisibles encodés (voir annexes)

-   Longueur : 18 bits

-   Example :

    -   SX1 : 110011 111000 010001

### Numéro de série de l’UAV ou de l’émetteur :

-   Signification : numéro de série du modèle.

-   Format : 3 octets soit 16 millions de combinaisons possibles. Dans
    le cas de numéros de série alphanumériques, une correspondance doit
    être proposée par le fabricant.

-   Longueur : 24 bits

### Pays de fabrication ou d’enregistrement de l’UAV ou de l’émetteur :

-   Signification : Identifiant du pays de fabrication ou
    d’enregistrement de l’UAV ou de l’émetteur.

-   Format : 2 caractères lisibles encodés (voir annexes)

-   Longueur : 12 bits

-   Example :

    -   FR : 100110 110010

### Horodatage :

-   Signification : Temps UTC du jour UTC défini en secondes.

-   Format : entier non signé sur 17 bits.

-   Longueur : 17 bits

### Latitude :

-   Signification : Latitude WGS-84

-   Format : entier signé 25 bits

    -   Unité 180/2\^24 °

    -   Précision inférieure à 1,20 m.

-   Longueur : 25 bits

### Longitude :

-   Signification : Longitude WGS-84

-   Format : entier signé 25 bits

    -   Unité 180/2\^24 °

    -   Précision inférieure à 1,20 m.

-   Longueur : 25 bits

### Altitude au-dessus du niveau de la mer :

-   Signification : Altitude au-dessus du niveau de la mer.

-   Format : entier non signé 14 bits, unité 1m. La valeur est
    l’altitude +1000m. Cela permet de représenter des altitudes allant
    de -1000m à 15382m avec une précision de 1m.

-   Longueur : 14 bits

### Précision GPS horizontale :

-   Signification : Précision horizontale de la position GPS.

-   Format : Entier non signé 7 bits. Unité 1m. Plage de valeur de 0 à
    127m.

-   Longueur : 7 bits

### Précision GPS verticale :

-   Signification : Précision verticale de la position GPS.

-   Format : Entier non signé 7 bits. Unité 1m. Plage de valeur de 0 à
    127m.

-   Longueur : 7 bits

### Fix GPS :

-   Signification : Indique si la position GPS est valide et 3D.

-   Format : Flag de 1 bit: 0 si invalide et 1 si 3D.

-   Longueur : 1 bit

### Vitesse horizontale :

-   Signification : Vitesse horizontale.

-   Format : Entier non signé 8 bits. Unité 1 mètre/seconde. Plage de
    vitesse de 0 à 255 m/s avec une précision de 1m/s.

-   Longueur : 8 bits

### Cap :

-   Signification : Cap de la vitesse horizontale.

-   Format : Entier non signé 9 bits. Unité 1 degré. Le cap représente
    l’angle que fait le vecteur vitesse horizontale avec le Nord vrai.
    La plage de valeur va de 0 à 359 degrés avec une précision de 1
    degré.

-   Longueur : 9 bits

### Vitesse verticale :

-   Signification : Vitesse verticale.

-   Format : Entier signé 7 bits. Unité 1 mètre/seconde. La vitesse est
    positive vers le haut. La plage de valeur va de -63m/s à 63 m/s avec
    une précision de 1 m/s.

-   Longueur : 7 bits

### Relay count :

-   Signification : Indice de répétition pour le relayage des trames. Il
    est présenté dans le protocole OGN. A chaque relayage, l’indice est
    décrémenté. Lorsqu’il atteint 0, aucun relayage n’est fait.

-   Format : Entier non signé2 bits. Plage de valeur de 0 à 3.

-   Longueur : 8 bits

### Urgences :

-   Signification : Ce flag indique si l’UAV est en mode d’urgence
    (perte de contrôle, panne moteur, …)

-   Format : 1 bit : 0 Pas d’urgence et 1 urgence.

-   Longueur : 1 bit

### Classe d’UAV :

-   Signification : Classe d’UAV selon un standard permettant d’évaluer
    le risque que représente l’UAV.

-   Format : Entier non signé 3 bits. Plage de valeur de 0 à 7 selon le
    niveau de risque.

-   Longueur : 3 bits

### Envoi signé :

-   Signification : Ce flag indique si une signature est présente en fin
    de message.

-   Format : 1 bit : 0 signifie pas de signature et 1 signifie
    signature.

-   Longueur : 1 bit

### CRC :

-   Signification : CRC 16 bits\* assurant l’intégrité du message. Il
    est calculé sur l’ensemble du message, à l’exception du préambule,
    du SYNC WORD et de la SIGNATURE.

-   Format 16 bits.

-   Longueur : 16 bits

> *\*Plusieurs algorithmes de CRC 16 bits existent. Il conviendra de
> préciser lequel est retenu.*

### Signature :

-   Signification : Les blocs ENTETE et DATA réunis sont signés par
    l’algorithme AES-CMAC utilisant une clé spécifique à l’UAV
    préalablement échangée avec l’organisme étatique pour lui permettre
    de vérifier l’intégrité et l’authenticité du message.

-   Format : chaine de 32\*\* octets.

-   Longueur : 256\*\* bits

*\*\*La taille de la signature peut être réduite afin de réduire la
taille du message. Une étude est en cours* sur cette troncature pouvant
permettre de réduire à 64 bits la longueur de la signature.

Conformité avec le format EUROCONTROL ASTERIX Part 29 Catégorie 129 
====================================================================

Le protocole étant directement issu du format « EUROCONTROL ASTERIX Part
29 Catégorie 129 », la correspondance est immédiate :

  ----------------------------------------------------------- ------------------------------------------------------------------- --
  **ASTERIX Part 29 Catégorie 129**                           **Correspondance dans le Protocole**
  Data source                                                 00/00 pour les communications « airborn to ground »
  Data destination                                            Inutile pour un signal broadcast
  []{#_Hlk495515362 .anchor}UAV Manufacturer Identifier       Identifiant du fabricant de l’UAV ou de l’émetteur
  UAS Model Identifier                                        Identifiant du modèle d’UAV ou d’émetteur
  UAS Serial Number or Aircraft ID (depends of adress type)   Numéro de série de l’UAV ou de l’émetteur
  UAS country                                                 Pays de fabrication ou d’enregistrement de l’UAV ou de l’émetteur
  Time of Day                                                 Time of Day
  Position in WGS-84 Coordinates - LATITUDE - LONGITUDE       Position in WGS-84 Coordinates - LATITUDE - LONGITUDE
  Altitude above Mean Sea Level (AMSL)                        Altitude above Mean Sea Level (AMSL)
  GNSS Signal precision                                       GPS accuracy H m
  ----------------------------------------------------------- ------------------------------------------------------------------- --

Data source
-----------

00/00 pour les communications « airborn to ground ».

Data destination
----------------

Inutile pour un signal « broadast ».

UAS Manufacturer Identifier
---------------------------

Correspondance totale

UAS Model Identifier
--------------------

Correspondance totale

UAS Serial Number or Aircraft ID (depends of adress type)
---------------------------------------------------------

Correspondance à définir par le fabricant entre les 12 caractères du
format ASTERIX et les 24 bits du protocole.

UAS country
-----------

Correspondance totale

Time of Day
-----------

Conversion de secondes en 1/128 de secondes.

Position in WGS-84 Coordinates – LATITUDE – LONGITUDE
-----------------------------------------------------

Conversion de degrés/2\^24 en degrés/2\^30

Altitude above Mean Sea Level (AMSL)
------------------------------------

Correspondance totale

GNSS Signal precision
---------------------

Correspondance totale

**\
**

Annexes
=======

Coding rules
------------

### Storage

The data are stored in binary mode. The storage is in little indian
meaning that in a byte, the highest bit is written first and the lowest
one at the end.

### Character encoding

A readable character used in any of the alphanumerical data is either a
capital letter A-Z, or a number 0-9 or the characters ‘ ‘, ‘-‘, ‘\_’.

Each of these characters has an ascii code between 32 and 63 (included).
So, each of these character is stored with a code between 0 to 31, which
is the ascii code minus 32. The storage of these character is then done
in 6 bits long word.

The mapping between the characters and the value is given in the
following table:

  DEC   HEX   BINARY   CHAR        DESCRIPTION         DEC   HEX   BINARY   CHAR     DESCRIPTION
  ----- ----- -------- ----------- ------------------- ----- ----- -------- -------- ----------------------
  0     0     0        **Space**   **space**           32    20    100000   **@**    at sign
  1     1     1        **!**       exclamation mark    33    21    100001   **A**
  2     2     10       **"**       double quote        34    22    100010   **B**
  3     3     11       **\#**      number              35    23    100011   **C**
  4     4     100      **\$**      dollar              36    24    100100   **D**
  5     5     101      **%**       percent             37    25    100101   **E**
  6     6     110      **&**       ampersand           38    26    100110   **F**
  7     7     111      **'**       single quote        39    27    100111   **G**
  8     8     1000     **(**       left parenthesis    40    28    101000   **H**
  9     9     1001     **)**       right parenthesis   41    29    101001   **I**
  10    0A    1010     **\***      asterisk            42    2A    101010   **J**
  11    0B    1011     **+**       plus                43    2B    101011   **K**
  12    0C    1100     **,**       comma               44    2C    101100   **L**
  13    0D    1101     **-**       minus               45    2D    101101   **M**
  14    0E    1110     **.**       period              46    2E    101110   **N**
  15    0F    1111     **/**       slash               47    2F    101111   **O**
  16    10    10000    **0**       zero                48    30    110000   **P**
  17    11    10001    **1**       one                 49    31    110001   **Q**
  18    12    10010    **2**       two                 50    32    110010   **R**
  19    13    10011    **3**       three               51    33    110011   **S**
  20    14    10100    **4**       four                52    34    110100   **T**
  21    15    10101    **5**       five                53    35    110101   **U**
  22    16    10110    **6**       six                 54    36    110110   **V**
  23    17    10111    **7**       seven               55    37    110111   **W**
  24    18    11000    **8**       eight               56    38    111000   **X**
  25    19    11001    **9**       nine                57    39    111001   **Y**
  26    1A    11010    **:**       colon               58    3A    111010   **Z**
  27    1B    11011    **;**       semicolon           59    3B    111011   **\[**   left square bracket
  28    1C    11100    **&lt;**    less than           60    3C    111100   **\\**   backslash
  29    1D    11101    **=**       equality sign       61    3D    111101   **\]**   right square bracket
  30    1E    11110    **&gt;**    greater than        62    3E    111110   **\^**   caret / circumflex
  31    1F    11111    **?**       question mark       63    3F    111111   **\_**   underscore


