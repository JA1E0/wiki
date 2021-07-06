---
    title: 多线程有关
    date: 2021-06-09
---

## 使用 _beginthreadex

不可在DLLmain中使用WaitForSingleObject等等待线程函数

```C++
#include<windows.h>
#include<process.h>

unsigned int __stdcall ThreadProc(PVOID pParam) {
	MessageBoxA(NULL, "thread", NULL, MB_OK);
	_endthreadex(0);
	return 0;
}

int main(int argc, char* argv[]) {

	HANDLE hThread = (HANDLE)_beginthreadex(NULL, 0, ThreadProc, NULL, 0, NULL);
	if (hThread != 0) {
		WaitForSingleObject(hThread, INFINITE);
		CloseHandle(hThread);
	}
	system("pause");
}
```