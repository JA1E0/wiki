---
title: IDA Python
date: 2021-12-09
---

### Documentation

https://www.hex-rays.com/products/ida/support/idapython_docs/

![img](../../assets/img/security/ida_python_doc.png)

### 字节操作
` ida_bytes.get_byte`

`ida_bytes.patch_byte`

```python
import ida_bytes

# address, size
key = ida_bytes.get_bytes(0x403070,5)

start = 0x40404c
length = 0x26f
for i in range(length):
    tmp = ida_bytes.get_byte(start+i)
    result = tmp^key[i%5]
    print(result)
    ida_bytes.patch_byte(start+i,result)
```