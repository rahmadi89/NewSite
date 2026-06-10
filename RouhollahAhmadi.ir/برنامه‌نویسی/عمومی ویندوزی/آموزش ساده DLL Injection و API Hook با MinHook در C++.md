## مقدمه

اگر علاقه‌مند به مهندسی معکوس یا یادگیری نحوهٔ کنترل رفتار یک برنامهٔ دیگر در ویندوز هستید، یکی از تکنیک‌های پایه، **DLL Injection و API Hook** است. در این پست، قدم به قدم یاد می‌گیریم چطور:

1. یک برنامهٔ سادهٔ قربانی بسازیم (`Victim.exe`).
2. یک DLL بسازیم که بتواند توابع آن را Hook کند (`Hook.dll`).
3. یک Injector بسازیم که DLL را به Process قربانی تزریق کند (`Injector.exe`).

تمام این مراحل با استفاده از **MinHook** و C++ انجام می‌شود. برای مطالعه بیشتر به مقاله [[مفاهیم API Hooking]] مراجعه کنید.

---
## گام ۱: ساخت برنامهٔ قربانی

برنامه Victim.exe باید توابعی را صدا بزند که بتوانیم Hook کنیم. ساده‌ترین مثال `MessageBoxA` است:
پس یک برنامه Console Application با نام Victim بسازید و این کد را درون Victim.cpp وارد کنید:

```cpp
#include <windows.h>
#include <iostream>

int main()
{
    std::cout << "Victim Started\n";
    std::cout << "PID = " << GetCurrentProcessId() << std::endl;


    while (true)
    {
        std::cout << "Enter a key to show MSGBox, (Ctrl+C to exit)" << std::endl;
        MessageBoxA(NULL, "Hello from Victim!", "Victim", MB_OK);
        getchar();
    }

    return 0;
}
```

- این برنامه با هر بار زدن کلید Enter یک پیام به این صورت نمایش می‌دهد:

![[Pasted image 20260610090201.png]]

- این تابع قابل Hook است و می‌توانیم متن آن را با DLL تغییر دهیم.

---
## گام ۲: ساخت DLL

فایل Hook.dll مسئول تغییر رفتار Victim است. برای مثال، تغییر متن MessageBox. به این منظور یک پروژه از نوع DLL با نام Hook بسازید و کد زیر را درون dllMain.cpp کپی کنید:

```cpp
// Hook.cpp
#include "pch.h"
#include <windows.h>


typedef int (WINAPI* MESSAGEBOXA)(HWND, LPCSTR, LPCSTR, UINT);
MESSAGEBOXA originalMessageBoxA = nullptr;

int WINAPI HookedMessageBoxA(HWND hWnd, LPCSTR lpText, LPCSTR lpCaption, UINT uType)
{
    return originalMessageBoxA(hWnd, "Hello from Hook.dll!", lpCaption, uType);
}

BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved)
{
    if (ul_reason_for_call == DLL_PROCESS_ATTACH)
    {
        MH_Initialize();

        MH_CreateHook(&MessageBoxA, &HookedMessageBoxA, reinterpret_cast<LPVOID*>(&originalMessageBoxA));
        MH_EnableHook(&MessageBoxA);
    }
    else if (ul_reason_for_call == DLL_PROCESS_DETACH)
    {
        MH_DisableHook(&MessageBoxA);
        MH_Uninitialize();
    }
    return TRUE;
}

```

سپس کد زیر را در فایل pch.h پروژه اضافه کنید تا کتابخانه MinHook به پروژه افزوده شود:

```cpp
#ifndef PCH_H
#define PCH_H

// add headers that you want to pre-compile here
#include "framework.h"
#include "MinHook.h"

#pragma comment(lib, "Libs\\MinHook.x86.lib")
#endif //PCH_H

```
دقت کنید که باید ابتدا فایل‌های کتابخانه MinHook را از آدرس زیر دانلود کرده 

https://github.com/TsudaKageyu/minhook/releases/latest

و فایل MinHook.h را کنار فایل Hook.cpp قرار داده و فایل‌های MinHook.x86.dll و MinHook.x86.lib را در پوشه Libs کنار فایل Hook.cpp قرار دهید تا آدرس‌های نوشته شده در کد به درستی کار کنند
یعنی پروژه یه همچین ساختاری باید داشته باشه:
```
│   dllmain.cpp
│   framework.h
│   Hook.vcxproj
│   Hook.vcxproj.filters
│   Hook.vcxproj.user
│   MinHook.h
│   pch.cpp
│   pch.h
│
├───Libs
│       MinHook.x86.dll
│       MinHook.x86.lib
│
```

**نکات مهم:**

- در این مثال، کتابخانه MinHook برای سیستم‌عامل هدف 32 بیت کپی شده.
-  اگر کتابخانه Static باشد، تمام کد داخل Hook.dll کامپایل می‌شود و هیچ DLL اضافی لازم نیست.
- اگر DLL به صورت Dynamic لینک شود، هنگام Load شدن نیاز به `MinHook.x86.dll` دارد. (ِMinHook به صورت داینامیک لینک می‌شود و نیاز است در هنگام اجرا، فایل MinHook.x86.dll کنار فایل Injector.exe یا Victim.exe باشد)
- فایل MinHook.x86.lib فقط یک Import Library است که فقط یک واسطه است و کد واقعی داخل DLL قرار دارد.
---
## گام ۳: ساخت Injector

برنامه Injector.exe مسئول تزریق DLL به Process Victim است:

```cpp
// Injector.cpp
#include <windows.h>
#include <tlhelp32.h>
#include <tchar.h>
#include <iostream>

DWORD GetProcessIdByName(const wchar_t* processName)
{
    DWORD pid = 0;
    HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 pe;
    pe.dwSize = sizeof(PROCESSENTRY32);

    if (Process32First(snapshot, &pe))
    {
        do
        {
            if (_wcsicmp(pe.szExeFile, processName) == 0)
            {
                pid = pe.th32ProcessID;
                break;
            }
        } while (Process32Next(snapshot, &pe));
    }

    CloseHandle(snapshot);
    return pid;
}

int main()
{
    const char* dllPath = "C:\\Hook.dll";
    DWORD pid = GetProcessIdByName(L"Victim.exe");

    if (pid == 0)
    {
        std::cout << "Victim not found!\n";
        getchar();
        return 1;
    }

    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
    if (!hProcess)
    {
        std::cout << "Failed to open process!\n";
        getchar();
        return 1;
    }

    LPVOID allocMem = VirtualAllocEx(hProcess, NULL, strlen(dllPath) + 1, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    WriteProcessMemory(hProcess, allocMem, dllPath, strlen(dllPath) + 1, NULL);

    HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)LoadLibraryA, allocMem, 0, NULL);
    WaitForSingleObject(hThread, INFINITE);

    //Check if inject was successful

    DWORD exitCode = 0;

    GetExitCodeThread(
        hThread,
        &exitCode);

    std::cout << "LoadLibrary returned: "
        << std::hex
        << exitCode
        << std::endl;
    // Done Checking

    if (exitCode == 0)
    {
        std::cout << "LoadLibrary FAILED" << std::endl;
    }
    else
    {
        std::cout << "LoadLibrary SUCCESS" << std::endl;
    }

    VirtualFreeEx(hProcess, allocMem, 0, MEM_RELEASE);
    CloseHandle(hThread);
    CloseHandle(hProcess);

    std::cout << "Handles Closed and memory released. Enter any key to exit\n";
    getchar();
    return 0;
}
```

آدرس  فایل DLL و نام پروسس قربانی باید در این دو قسمت وارد شود:
```cpp
    const char* dllPath = "C:\\Hook.dll";
    DWORD pid = GetProcessIdByName(L"Victim.exe");
```

**نکته کلیدی:**

- تابع `GetExitCodeThread` مقدار واقعی LoadLibrary داخل Process قربانی را برمی‌گرداند.
- حتی اگر Injector اجرا شود، اگر DLL وابستگی‌هایش پیدا نشوند، inject شکست می‌خورد.
---

بعد از ساخت هر سه برنامه (Victom.exe و Injector.exe و Hook.dll) ابتدا فایل Hook.dll را در درایو C کپی کنید سپس برنامه Victim.exe را اجرا کنید. با زدن کلید Enter پیام "Hello From Victim" نمایش داده می شود.
حالا برنامه Injector.exe را اجرا کنید (فایل MinHook.x86.dll کنارش باشد). در صورت انجام موفق عملیات باید پیامی مانند شکل زیر نمایش داده شود:

![[Pasted image 20260610092910.png]]

حالا دیگر نیازی به Injector.exe نیست و می‌توانیم از این برنامه خارج شویم.
در این مرحله اگر در برنامه Victim.exe کلید Enter را بزنیم با پیام زیر مواجه می‌شویم:

![[Pasted image 20260610093122.png]]


