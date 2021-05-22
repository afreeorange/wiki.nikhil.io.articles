```python
from __future__ import division
import os

filesystem = '/filesystem'
usage_threshold = 47

i = os.statvfs(filesystem)
usage = ((i.f_blocks - i.f_bfree)/i.f_blocks) * 100

if usage >= usage_threshold:
    print "Usage above threshold"
```
