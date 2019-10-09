# IP packet structure

## !. Header of packet
- Size: 20 bytes or 5 x 32 bits
- First 32 bits contains:
    - version [0-3]: version of Protocol (ICMP - 1, TCP - 6, UDP - 7)
    - IHL [4-7]: full size of header
    - TOS [8-15]: (type of services) which is set option of transmition quality
    - total_length [16-31]: packet size (limit: 0xFFFF bytes)
- Second 32 bits:
    - id [0-15]: identification of packet
    - flags [16-18]: control fragments (DF - don't fragment,MF or [+] - more fragments,  [none]) - end of [+])
    - fragment_offset [19-31]: offset by order in comparison to begining packet
- Third 32 bits:
    - TTL [0-7]: time to live
    - protocol [8-15]: protocol, which attaches this packet
    - checksum [16-31]: contain controlling sum of header packet
- Fourth 32 bits:
    - source_address [0-31]: 32 bits address (sender)
- Fifth 32 bits:
    - destination_address [0-31]: 32 bits address (receiver)