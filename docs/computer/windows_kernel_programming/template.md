---
title: 内核基础模板
date: 2021-10-12
---

## 驱动基础模板
```C
#include <ntifs.h>
#include <windef.h>

VOID DriverUnload(PDRIVER_OBJECT pDriverObj) {
	UNICODE_STRING strLink;
	//do sth...
	DbgPrint("[DriverUnload]\n");
}

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObj, PUNICODE_STRING pRegistryString) {
	NTSTATUS status = STATUS_SUCCESS;

	//do sth...
	DbgPrint("[DriverEntry]\n");
	return status;
}
```

## 驱动带IRP模板
```C++
//工程：WDM驱动模板
#include <ntifs.h>
#include <stdlib.h>
#include <windef.h>
#include <ntimage.h>

//symbolicLinkName	L"\\DosDevices\\SymbolicLinkName"
#define SYM_NAME	L"\\??\\MyFirstSYM"
#define DEVICE_NAME L"\\Device\\MyDriver" 


#define IOCTL_TEST	CTL_CODE(FILE_DEVICE_UNKNOWN, 0x800, METHOD_BUFFERED, FILE_ANY_ACCESS) 

NTSTATUS DispatchCreate(PDEVICE_OBJECT pDevObj, PIRP pIrp) {
	pIrp->IoStatus.Status = STATUS_SUCCESS;
	pIrp->IoStatus.Information = 0;
	IoCompleteRequest(pIrp, IO_NO_INCREMENT);
	return STATUS_SUCCESS;
}

NTSTATUS DispatchClose(PDEVICE_OBJECT pDevObj, PIRP pIrp) {
	pIrp->IoStatus.Status = STATUS_SUCCESS;
	pIrp->IoStatus.Information = 0;
	IoCompleteRequest(pIrp, IO_NO_INCREMENT);
	return STATUS_SUCCESS;
}

NTSTATUS DispatchIoctl(PDEVICE_OBJECT pDevObj, PIRP pIrp) {
	NTSTATUS status = STATUS_INVALID_DEVICE_REQUEST;
	PIO_STACK_LOCATION pIrpStack = IoGetCurrentIrpStackLocation(pIrp);
	ULONG uIoControlCode = pIrpStack->Parameters.DeviceIoControl.IoControlCode;
	PVOID pIoBuffer = pIrp->AssociatedIrp.SystemBuffer;
	ULONG uInSize = pIrpStack->Parameters.DeviceIoControl.InputBufferLength;
	ULONG uOutSize = pIrpStack->Parameters.DeviceIoControl.OutputBufferLength;
	switch (uIoControlCode) {
	case IOCTL_TEST:
	{
		DbgPrint("[IOCTL_TEST]\n");
		DWORD dw;
		memcpy(&dw, pIoBuffer, sizeof(dw));
		dw++;
		memcpy(pIoBuffer, &dw, sizeof(dw));
		status = STATUS_SUCCESS;
		break;
	}
	}
	if (status == STATUS_SUCCESS)
		pIrp->IoStatus.Information = uOutSize;
	else
		pIrp->IoStatus.Information = 0;
	pIrp->IoStatus.Status = status;
	IoCompleteRequest(pIrp, IO_NO_INCREMENT);
	return status;
}

VOID DriverUnload(PDRIVER_OBJECT pDriverObj) {
	UNICODE_STRING strLink;
	//do sth...
	DbgPrint("[DriverUnload]\n");
	//delete device and symbolic link
	RtlInitUnicodeString(&strLink, SYM_NAME);
	IoDeleteSymbolicLink(&strLink);
	IoDeleteDevice(pDriverObj->DeviceObject);
}

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObj, PUNICODE_STRING pRegistryString) {
	NTSTATUS status = STATUS_SUCCESS;
	PDEVICE_OBJECT pDevObj = NULL;
	UNICODE_STRING ustrDeviceName;
	UNICODE_STRING ustrLinkName;

	//set dispatch functions
	pDriverObj->DriverUnload = DriverUnload;
	pDriverObj->MajorFunction[IRP_MJ_DEVICE_CONTROL] = DispatchIoctl;
	pDriverObj->MajorFunction[IRP_MJ_CREATE] = DispatchCreate;
	pDriverObj->MajorFunction[IRP_MJ_CLOSE] = DispatchClose;

	//create device
	RtlInitUnicodeString(&ustrDeviceName, DEVICE_NAME);
	status = IoCreateDevice(pDriverObj, 0, &ustrDeviceName, FILE_DEVICE_UNKNOWN, 0, FALSE, &pDevObj);
	if (!NT_SUCCESS(status)) {
		return status;
	}

	//create symbolic link
	RtlInitUnicodeString(&ustrLinkName, SYM_NAME);
	status = IoCreateSymbolicLink(&ustrLinkName, &ustrDeviceName);
	if (!NT_SUCCESS(status)) {
		IoDeleteDevice(pDevObj);
		return status;
	}

	//do sth...
	DbgPrint("[DriverEntry]\n");
	return status;
}
```

## R3 IO调用模板
```C
#include <iostream>
#include <Windows.h>
#include<winioctl.h>

//#define LINK_NAME	L"\\DosDevices\\Global\\MyDriverLinkName"
#define DEVICE_NAME	L"\\\\.\\MyFirstSYM"
#define IOCTL_TEST	CTL_CODE(FILE_DEVICE_UNKNOWN, 0x800, METHOD_BUFFERED, FILE_ANY_ACCESS) 

int main() {
	HANDLE hDevice = NULL;
	hDevice = CreateFile(DEVICE_NAME, GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
	if (hDevice == INVALID_HANDLE_VALUE) {
		printf("[Open Device Failed]\n");
		system("pause");
		return 0;
	}

	printf("[Open Success]\n");
	system("pause");

	DWORD dwIn = 1;
	DWORD dwOut = 0;
	DWORD dwReturn = 0;
	DeviceIoControl(hDevice, IOCTL_TEST, &dwIn, 4, &dwOut, 4, &dwReturn, NULL);
	if (dwReturn != 0) {
		printf("[In %d\tOut %d\tinfo %d\n]", dwIn, dwOut, dwReturn);
	}
	else {
		printf("[IOCTL_TEST Failed]\n");
	}
	
	CloseHandle(hDevice);
	system("pause");
	return 0;
}
```
## 驱动加载模板

> http://www.m5home.com/bbs/thread-8602-1-1.html

```C++
#include <iostream>
#include <Windows.h>

#pragma comment(lib,"advapi32.lib") //Finishing by GFW@vbasm.com

SC_HANDLE InstallDriver(PCHAR m_pSysPath, PCHAR m_pServiceName) {
	PCHAR m_pDisplayName = m_pServiceName;
	SC_HANDLE m_hSCManager = OpenSCManagerA(NULL, NULL, SC_MANAGER_ALL_ACCESS);
	if (NULL == m_hSCManager) {
		return 0;
	}
	SC_HANDLE m_hService = CreateServiceA(m_hSCManager,
		m_pServiceName,
		m_pDisplayName,
		SERVICE_ALL_ACCESS,
		SERVICE_KERNEL_DRIVER,
		SERVICE_DEMAND_START,
		SERVICE_ERROR_NORMAL,
		m_pSysPath,
		NULL, NULL, NULL, NULL, NULL);
	if (NULL == m_hService) {
		if (ERROR_SERVICE_EXISTS == GetLastError()) {
			m_hService = OpenServiceA(m_hSCManager, m_pServiceName, SERVICE_ALL_ACCESS);
		}
	}
	CloseServiceHandle(m_hSCManager);
	return m_hService;
}

BOOL StartDriver(SC_HANDLE m_hService) {
	return StartServiceA(m_hService, NULL, NULL);
}

BOOL StopDriver(SC_HANDLE m_hService) {
	SERVICE_STATUS ss = { 0 };
	return ControlService(m_hService, SERVICE_CONTROL_STOP, &ss);
}

BOOL RemoveDriver(SC_HANDLE m_hService) {
	return DeleteService(m_hService);
}

HANDLE OpenDriver(PCHAR pLinkName)/* \\\\.\\test */
{
	HANDLE m_hDriver = CreateFileA(pLinkName, GENERIC_READ | GENERIC_WRITE, 0, 0, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0);
	if (m_hDriver != INVALID_HANDLE_VALUE)
		return m_hDriver;
	else {
		puts("OpenDriver Sym Failed!");
		return 0;
	}
}

BOOL CloseDriver(HANDLE m_hDriver) {
	__try {
		return CloseHandle(m_hDriver);
	}
	__except (1) {
		return 0;
	}
}

_inline DWORD CTL_CODE_GEN(DWORD lngFunction) {
	return (FILE_DEVICE_UNKNOWN * 65536) | (FILE_ANY_ACCESS * 16384) | (lngFunction * 4) | METHOD_BUFFERED;
}

BOOL ControlDriver(HANDLE m_hDriver, DWORD dwIoCode, PVOID InBuff, DWORD InBuffLen, PVOID OutBuff, DWORD OutBuffLen, DWORD* RealRetBytes) {
	DWORD dw = 0;
	BOOL b = DeviceIoControl(m_hDriver, CTL_CODE_GEN(dwIoCode), InBuff, InBuffLen, OutBuff, OutBuffLen, &dw, NULL);
	if (RealRetBytes) {
		*RealRetBytes = dw;
	}
	return b;
}

void GetAppPath(char* szPathString) {
	GetModuleFileNameA(0, szPathString, MAX_PATH);
	for (SIZE_T i = strlen(szPathString) - 1; i >= 0; i--) {
		if (szPathString[i] == '\\') {
			szPathString[i + 1] = '\0';
			break;
		}
	}
}

SC_HANDLE GetServiceHandle(PCHAR m_pServiceName) {
	SC_HANDLE m_hSCManager = OpenSCManagerA(NULL, NULL, SC_MANAGER_ALL_ACCESS);
	if (NULL == m_hSCManager) {
		return 0;
	}
	SC_HANDLE m_hService = OpenServiceA(m_hSCManager, m_pServiceName, SERVICE_ALL_ACCESS);
	if (NULL == m_hService) {
		CloseServiceHandle(m_hSCManager);
		return 0;
	}
	CloseServiceHandle(m_hSCManager);
	return m_hService;
}

SC_HANDLE g_hSc;
HANDLE g_hDrv;

BOOLEAN DriverInit(BOOLEAN IsInit) {
	if (IsInit) {
		//char szFileName[] = "XXXX.sys";//TODO: CHANGE THIS VALUE
		//char szLinkName[] = "\\\\.\\XXXX";//TODO: CHANGE THIS VALUE		
		char szFileName[] = "MyDriver1.sys";//TODO: CHANGE THIS VALUE
		char szLinkName[] = "\\\\.\\MyFirstSYM";//TODO: CHANGE THIS VALUE
		CHAR szDrvFile[MAX_PATH] = { 0 };
		GetAppPath(szDrvFile);
		strcat_s(szDrvFile, szFileName);
		g_hSc = InstallDriver(szDrvFile, szFileName);
		if (g_hSc) {
			StartDriver(g_hSc);
			g_hDrv = OpenDriver(szLinkName);
			if (g_hDrv)
				return 1;
			else
				return 0;
		}
		return 0;
	}
	else {
		BOOLEAN b;
		b = CloseDriver(g_hDrv); if (!b) return 0;
		b = StopDriver(g_hSc); if (!b) return 0;
		b = RemoveDriver(g_hSc); if (!b) return 0;
		b = CloseServiceHandle(g_hSc); if (!b) return 0;
		return 1;
	}
}

int main() {
	BOOLEAN b;
	b = DriverInit(1); if (b) {
		puts("load driver OK!");
	}
	else {
		puts("load driver failed!"); return;
	}
	system("pause");
	//ControlDriver(g_hDrv,0x800,0,0,0,0,0);
	b = DriverInit(0); if (b) {
		puts("unload driver OK!");
	}
	else {
		puts("unload driver failed!");
	}
	system("pause");
}
```
