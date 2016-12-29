# Serial Interface Format

In this document we will explain the serial data format including some examples.

## Format
### Key
* {START} = 0xFA
* {END} = 0xFB
* {ESCAPE} = 0xFC, the next characters needs to be inverted, used for payload data and checksum.
* [CHECKSUM] = sum of command, payload_size, and data
* [] = 1 byte
* L = Least Significant
* M = Most Significant

```
{START}[command][payload_size_M][payload_size_L[payload 1..N][CHECKSUM]{END}
```
## Error states
- Reset the parser/data when we receive the {START} byte.
- If we don't see the {END} byte after reading the expected payload, discard the data.
- If receive the {END} byte too early, discard the data.
- Whenever we receive the escape byte, the next byte will be inverted (NOT'd).
- Discard any messages with invalid checksum
- If payload size is 0, skip reading payload and move to the checksum.  This allows for a command with no payload.
- If payload length is > payload_max_length, discard the current message.

## example
*Coded message*
```
0xFA Ox01 0x00 0x02 0xFC 0x03 0xFC 0x02 0xFC 0x01 0xFB
```

*Decoded message*
```
Ox01 0x00 0x02 0xFC 0xFB 0xFA
```

*Parser Output*
```
uint8_t   Command : 0x01
uint16_t   Length : 0x02
uint8_t  Checksum : 0xFA
uint8_t    Data[] : [0xFC, 0xFB]
```
