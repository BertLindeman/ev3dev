The buttons on the EV3 are mapped as regular keyboard keys. UP, DOWN, LEFT, RIGHT, ENTER and BACKSPACE.
The mapping on wheezy was UP, DOWN, LEFT, RIGHT, ENTER and **ESC**.

TODO: add photo.

## Polling Key States

If you would rather poll the button states instead of using regular keyboard input methods, you can do that via the EVIOCGKEY ioctl on `/dev/input/eventX`. Key codes and ioctl definitions are in [linux/input.h](https://github.com/mindboards/ev3dev-kernel/blob/master/include/linux/input.h).

Here is an example using python:

```python
#!/usr/bin/env python

import array
import fcntl
import sys

# from linux/input.h

KEY_UP = 103
KEY_DOWN = 108
KEY_LEFT = 105
KEY_RIGHT = 106
KEY_ENTER = 28
# KEY_ESC = 1      ## on wheezy
KEY_BACKSPACE = 14 ## on jessie

KEY_MAX = 0x2ff

def EVIOCGKEY(length):
    return 2 << (14+8+8) | length << (8+8) | ord('E') << 8 | 0x18

# end of stuff from linux/input.h

BUF_LEN = (KEY_MAX + 7) / 8

def test_bit(bit, bytes):
    # bit in bytes is 1 when released and 0 when pressed
    return not bool(bytes[bit / 8] & 1 << bit % 8)

def main():
    buf = array.array('B', [0] * BUF_LEN)
    with open('/dev/input/by-path/platform-gpio-keys.0-event', 'r') as fd:
        ret = fcntl.ioctl(fd, EVIOCGKEY(len(buf)), buf)

    if ret < 0:
        print "ioctl error", ret
        sys.exit(1)

    for key in ['UP', 'DOWN', 'LEFT', 'RIGHT', 'ENTER', 'BACKSPACE']:
        key_code = globals()['KEY_' + key]
        key_state = test_bit(key_code, buf) and "pressed" or "released"
        print '%s:' % key, key_state

if __name__ == "__main__":
    main()
```

# Directly Reading the Event Device

If you want your program to be event driven, you can read the ```/dev/input/by-path/platform-gpio-keys.0-event``` file. It will block until a key is pressed. The data is in 16 byte blocks. The first 8 bytes are the time stamp (2 unsigned 64-bit integers, seconds and microseconds), the next 2 bytes are the type (unsigned 16-bit integer), the next 2 bytes are the code (unsigned 16-bit integer) and the last 4 bytes are the value (unsigned 32-bit integer).

Here is an example. It prints out 2 lines each time you press a button on the EV3 and 2 more lines each time you release a button. And of course, press CTRL+C to end.

```sh
root@ev3dev:/sys/class/leds# hexdump -e '"timestamp:%d.%6d""\t""" 1/2 "type:%i""\t"""  1/2 "code:%3i""\t"""  "value:%d\n"' /dev/input/by-path/platform-gpio-keys.0-event
timestamp:1411233011.710843     type:1  code: 14        value:0
timestamp:1411233011.710843     type:0  code:  0        value:0
timestamp:1411233011.930731     type:1  code: 14        value:1
timestamp:1411233011.930731     type:0  code:  0        value:0
timestamp:1411233018.910730     type:1  code:103        value:0
timestamp:1411233018.910730     type:0  code:  0        value:0
timestamp:1411233019.110728     type:1  code:103        value:1
timestamp:1411233019.110728     type:0  code:  0        value:0
timestamp:1411233027. 40849     type:1  code:108        value:0
timestamp:1411233027. 40849     type:0  code:  0        value:0
timestamp:1411233027.260726     type:1  code:108        value:1
timestamp:1411233027.260726     type:0  code:  0        value:0
timestamp:1411233029.280731     type:1  code: 28        value:0
timestamp:1411233029.280731     type:0  code:  0        value:0
timestamp:1411233029.450733     type:1  code: 28        value:1
timestamp:1411233029.450733     type:0  code:  0        value:0
timestamp:1411233032.770741     type:1  code:105        value:0
timestamp:1411233032.770741     type:0  code:  0        value:0
timestamp:1411233032.950844     type:1  code:105        value:1
timestamp:1411233032.950844     type:0  code:  0        value:0
timestamp:1411233035.800922     type:1  code:106        value:0
timestamp:1411233035.800922     type:0  code:  0        value:0
timestamp:1411233036. 20729     type:1  code:106        value:1
timestamp:1411233036. 20729     type:0  code:  0        value:0
^C
```
