---
layout: post
title: Number Bases to ASCII Conversion
---

Converting different number bases to strings:

***Hexadecimal*** (with python):
```
>>> chr(0x70) + chr(0x61) + chr(0x6e) + chr(0x64) + chr(0x61)
'panda'
```
Or, in Python2:
```
>>> "70616e6461".decode('hex')
```

In Python2 and 3:
```
>>> import binascii
>>> binascii.unhexlify("70616e6461")
'panda'
>>> binascii.unhexlify(b"70616e6461")
'panda'
```

***base64*** (with bash):
```
echo -n "cGVyZm9ybQ==" | base64 -d -
```

With python:
```
>>> import base64
>>> base64.b64decode("".join([chr(int(byte, 2)) for byte in pw.split()]))
'lollipop'
```

***binary*** (with python):
```
>>> pw = "01110011 01100101 01100011 01110101 01110010 01100101 01101100 01111001"
>>> pwl = pw.split()
>>> pwl
['01110011', '01100101', '01100011', '01110101', '01110010', '01100101', '01101100', '01111001']
>>> pwc = [chr(int(byte, 2)) for byte in pwl]
>>> pwc
['s', 'e', 'c', 'u', 'r', 'e', 'l', 'y']
>>> pwd = "".join(pwc)
>>> pwd
'securely'
```
As a one-liner:
```
>>> "".join([chr(int(byte, 2)) for byte in pw.split()])
'securely'
```
