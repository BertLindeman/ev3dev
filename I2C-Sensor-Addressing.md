I2C uses a 7-bit addressing scheme (there is also 10-bit addressing but we are not using it). When sending an address over the bus, the address is shifted to the left 1 bit and the least significant bit is used to indicate read or write. It is important to know this because LEGO and 3rd-party sensor manufacturers use the shifted value as the address in their documentation. This shifted value is also used in most other NXT/EV3 programming languages/environments. 

**IMPORTANT NOTE**: I2C addresses 0x01 through 0x07 (unshifted) are reserved for special use by the I2C specifications. However, these addresses are used by some sensors anyway (most notably the NXT Ultrasonic sensor). The ev3dev kernel has been patched to make these work, but many userspace tools (like i2c-tools) will not work with devices at these addresses.

ev3dev is different because the Linux kernel uses the unshifted value as the I2C address. This means you will have to convert the value to get the correct address. Shift to the right is the same as divide by 2, so get out your hexadecimal calculator and do some math!

Or for the engineer type folks that prefer tables...

| Shifted Address (write/read) | Unshifted Address | Notes | Shifted Address (write/read) | Unshifted Address | Notes |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 0x00/0x01 | __0x00__ | *I2C spec: General call address / START byte* | 0x80/0x81 | __0x40__ |  |
| 0x02/0x03 | __0x01__ | NXT Ultrasonic and many 3rd party sensors <hr /> *I2C spec: CBUS address* | 0x82/0x83 | __0x41__ |  |
| 0x04/0x05 | __0x02__ | LEGO Energy Storage <hr /> *I2C spec: Reserved for different bus format* | 0x84/0x85 | __0x42__ |  |
| 0x06/0x07 | __0x03__ | *I2C spec: Reserved for future purposes* | 0x86/0x87 | __0x43__ |  |
| 0x08/0x09 | __0x04__ | *I2C spec: Hs-mode master code* | 0x88/0x89 | __0x44__ |  |
| 0x0A/0x0B | __0x05__ | *I2C spec: Hs-mode master code* | 0x8A/0x8B | __0x45__ |  |
| 0x0C/0x0D | __0x06__ | *I2C spec: Hs-mode master code* | 0x8C/0x8D | __0x46__ |  |
| 0x0E/0x0F | __0x07__ | *I2C spec: Hs-mode master code* | 0x8E/0x8F | __0x47__ |  |
| 0x10E/0x11 | __0x08__ | Some HiTechnic sensors | 0x90/0x91 | __0x48__ |  |
| 0x12/0x13 | __0x09__ |  | 0x92/0x93 | __0x49__ |  |
| 0x14/0x15 | __0x0A__ | mindsensors.com Light Sensor Array | 0x94/0x95 | __0x4A__ |  |
| 0x16/0x17 | __0x0B__ |  | 0x96/0x97 | __0x4B__ |  |
| 0x18/0x19 | __0x0C__ |  | 0x98/0x99 | __0x4C__ | LEGO Temperature Sensor |
| 0x1A/0x1B | __0x0D__ |  | 0x9A/0x9B | __0x4D__ |  |
| 0x1C/0x1D | __0x0E__ |  | 0x9C/0x9D | __0x4E__ |  |
| 0x1E/0x1F | __0x0F__ |  | 0x9E/0x0F | __0x4F__ |  |
| 0x20/0x21 | __0x10__ |  | 0xA0/0xA1 | __0x50__ |  |
| 0x22/0x23 | __0x11__ |  | 0xA2/0xA3 | __0x51__ |  |
| 0x24/0x25 | __0x12__ |  | 0xA4/0xA5 | __0x52__ |  |
| 0x26/0x27 | __0x13__ |  | 0xA6/0xA7 | __0x53__ |  |
| 0x28/0x29 | __0x14__ |  | 0xA8/0xA9 | __0x54__ |  |
| 0x2A/0x2B | __0x15__ |  | 0xAA/0xAB | __0x55__ |  |
| 0x2C/0x2D | __0x16__ |  | 0xAC/0xAD | __0x56__ |  |
| 0x2E/0x2F | __0x17__ |  | 0xAE/0xAF | __0x57__ |  |
| 0x30/0x31 | __0x18__ |  | 0xB0/0xB1 | __0x58__ |  |
| 0x32/0x33 | __0x19__ |  | 0xB2/0xB3 | __0x59__ |  |
| 0x34/0x35 | __0x1A__ |  | 0xB4/0xB5 | __0x5A__ |  |
| 0x36/0x37 | __0x1B__ |  | 0xB6/0xB7 | __0x5B__ |  |
| 0x38/0x39 | __0x1C__ |  | 0xB8/0xB9 | __0x5C__ |  |
| 0x3A/0x3B | __0x1D__ |  | 0xBA/0xBB | __0x5D__ |  |
| 0x3C/0x3D | __0x1E__ |  | 0xBC/0xBD | __0x5E__ |  |
| 0x3E/0x3F | __0x1F__ |  | 0xBE/0xBF | __0x5F__ |  |
| 0x40/0x41 | __0x20__ |  | 0xC0/0xC1 | __0x60__ |  |
| 0x42/0x43 | __0x21__ |  | 0xC2/0xC3 | __0x61__ |  |
| 0x44/0x45 | __0x22__ |  | 0xC4/0xC5 | __0x62__ |  |
| 0x46/0x47 | __0x23__ |  | 0xC6/0xC7 | __0x63__ |  |
| 0x48/0x49 | __0x24__ |  | 0xC8/0xC9 | __0x64__ |  |
| 0x4A/0x4B | __0x25__ |  | 0xCA/0xCB | __0x65__ |  |
| 0x4C/0x4D | __0x26__ |  | 0xCC/0xCD | __0x66__ |  |
| 0x4E/0x4F | __0x27__ |  | 0xCE/0xCF | __0x67__ |  |
| 0x50/0x51 | __0x28__ |  | 0xD0/0xD1 | __0x68__ |  |
| 0x52/0x53 | __0x29__ |  | 0xD2/0xD3 | __0x69__ |  |
| 0x54/0x55 | __0x2A__ |  | 0xD4/0xD5 | __0x6A__ |  |
| 0x56/0x57 | __0x2B__ |  | 0xD6/0xD7 | __0x6B__ |  |
| 0x58/0x59 | __0x2C__ |  | 0xD8/0xD9 | __0x6C__ |  |
| 0x5A/0x5B | __0x2D__ |  | 0xDA/0xDA | __0x6D__ |  |
| 0x5C/0x5D | __0x2E__ |  | 0xDC/0xDD | __0x6E__ |  |
| 0x5E/0x5F | __0x2F__ |  | 0xDE/0xDF | __0x6F__ |  |
| 0x60/0x61 | __0x30__ |  | 0xE0/0xE1 | __0x70__ |  |
| 0x62/0x63 | __0x31__ |  | 0xE2/0xE3 | __0x71__ |  |
| 0x64/0x65 | __0x32__ |  | 0xE4/0xE5 | __0x72__ |  |
| 0x66/0x67 | __0x33__ |  | 0xE6/0xE7 | __0x73__ |  |
| 0x68/0x69 | __0x34__ |  | 0xE8/0xE9 | __0x74__ |  |
| 0x6A/0x6B | __0x35__ |  | 0xEA/0xEB | __0x75__ |  |
| 0x6C/0x6D | __0x36__ |  | 0xEC/0xED | __0x76__ |  |
| 0x6E/0x6F | __0x37__ |  | 0xEE/0xEF | __0x77__ |  |
| 0x70/0x71 | __0x38__ |  | 0xF0/0xF1 | __0x78__ | *I2C spec: 10-bit slave addressing* |
| 0x72/0x73 | __0x39__ |  | 0xF2/0xF3 | __0x79__ | *I2C spec: 10-bit slave addressing* |
| 0x74/0x75 | __0x3A__ |  | 0xF4/0xF5 | __0x7A__ | *I2C spec: 10-bit slave addressing* |
| 0x76/0x77 | __0x3B__ |  | 0xF6/0xF7 | __0x7B__ | *I2C spec: 10-bit slave addressing* |
| 0x78/0x79 | __0x3C__ |  | 0xF8/0xF9 | __0x7C__ | *I2C spec: Reserved for future purposes* |
| 0x7A/0x7B | __0x3D__ |  | 0xFA/0xFB | __0x7D__ | *I2C spec: Reserved for future purposes* |
| 0x7C/0x7D | __0x3E__ |  | 0xFC/0xFD | __0x7E__ | *I2C spec: Reserved for future purposes* |
| 0x7E/0x7F | __0x3F__ |  | 0xFE/0xFF | __0x7F__ | *I2C spec: Reserved for future purposes* |


