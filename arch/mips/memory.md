# mips下的内存分布


```
0xFFFF FFFF+----------------------------------+
           |                                  |
           |  Kernel Space Mapped Cached      |
           |                                  |
           |                                  |
0xc000 0000+----------------------------------+
           |                                  |
           |  Kernel Space Unmapped Cached    |
           |                                  |
0xA000 0000+----------------------------------+
           |                                  |
           |  Kernel Space Unmapped Uncached  |
           |                                  |
0x8000 0000+----------------------------------+
           |                                  |
           |                                  |
           |                                  |
           |                                  |
           |            User Space            |
           |                                  |
           |                                  |
           |                                  |
           |                                  |
           |                                  |
0x0000 0000+----------------------------------+
```


