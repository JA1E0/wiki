---
    title: win32代码片段
    date: 2021-12-03
---

## CreateProcess
### CreateSuspendProcess

```cpp
STARTUPINFOA StartupInfo = { 0 };
PROCESS_INFORMATION ProcessInformation = { 0 };
//创建挂起的进程
if ( !CreateProcessA(
	0,
	pDestCmdLine,
	0,
	0,
	0,
	CREATE_SUSPENDED, //0x00000004
	0,
	0,
	&StartupInfo,
	&ProcessInformation
)) {
		DEBUG_ERROR("[-]创建失败");
		break;
	}
```