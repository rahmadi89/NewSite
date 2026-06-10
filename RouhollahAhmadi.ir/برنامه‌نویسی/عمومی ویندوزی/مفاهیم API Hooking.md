## مقدمه: API Hook اصلاً چیست؟
فرض کن برنامه‌ای این کد را اجرا می‌کند:

```cpp
MessageBoxA(    
	NULL,    
	"Hello",    
	"Test",    
	MB_OK
);
```

جریان اجرا:

```
Application => MessageBoxA => user32.dll => Windows
```

حالا اگر بتوانیم قبل از اجرای MessageBoxA کنترل را بگیریم:

```
Application => MyHookFunction => Log or Modify => Original MessageBoxA
```

به این کار Hook می‌گویند.

---
## خب Hook چه کاربردی دارد؟

مثلاً:
- لاگ کردن فایل‌هایی که باز می‌شوند
- لاگ کردن ترافیک شبکه
- مانیتور کردن USB
- انجام Profiling
- انجام Debugging

مثلاً هر بار برنامه:

```cpp
CreateFile(...)
```

صدا زد:

```cpp
CreateFile(C:\temp\a.txt)
CreateFile(C:\temp\b.txt)
CreateFile(C:\temp\c.txt)
```

را لاگ کنیم.

---

## چرا DLL Injection لازم است؟

و یا چطور Hook من داخل Process قربانی اجرا شود؟

فرض کن برنامه Notepad اجرا شده است. ویندوز خودش DLL ما را داخل Notepad لود نمی‌کند.

بنابراین باید فایل Hook.dll را داخل برنامه Notepad تزریق کنیم.

---

## معماری کلی

سه بخش داریم:

```
Injector.exe
Victim.exe
Hook.dll
```

---

## برنامه Victim

برنامه‌ای که قرار است Hook شود.

مثلاً:

```cpp
int main(){    
	while(true)    
	{        
		MessageBoxA(
			NULL,
			"Hello",
			"Victim",
			MB_OK
			);
		}
	}
```

---

## فایل Hook DLL
بعد از لود شدن باید Hook کرده و عملیات مدنظر ما را انجام دهد.

```cpp
BOOL APIENTRY DllMain(...){    
	InstallHook(); 
	DoSomeWork();   
	return TRUE;
	}
```

---

## برنامه Injector

این برنامه وظیفه پیدا کردن پروسس قربانی (Victim)، باز کردن آن پروسس و تزریق DLL در فضای حافظه آن برنامه را دارد.

---

## حالا DLL Injection از نظر مفهومی چگونه کار می‌کند؟

فرض کنید DLL اینجاست:

```
C:\Hook.dll
```

ما به Process قربانی می‌گوییم:

```
LoadLibrary("C:\\Hook.dll");
```

اما از داخل Process خودمان نمی‌خواهیم اجرا شود.

می‌خواهیم داخل Process قربانی اجرا شود.

پس:

```
Process A => Process B => LoadLibrary("Hook.dll")
```

را در Process B اجرا می‌کنیم.

---

## ایده کلی تزریق

مراحل:

```
OpenProcess
VirtualAllocEx
WriteProcessMemory
CreateRemoteThread
```

---

### مرحله اول 
از طریق PID مربوط به پروسس قربانی، هندل آن را استخراج کرده و باز (OpenProcess) می‌کنیم.

### مرحله دوم
با استفاده از VirtualAllocEx داخل حافظه قربانی فضا می‌گیریم.

### مرحله سوم
با استفاده از WriteProcessMemory مسیر DLL را در حافظه قربانی می‌نویسیم.

### مرحله چهارم

از طریق CreateRemoteThread تابع LoadLibraryA را در Process قربانی اجرا می‌کنیم. با این جریان:

```
Injector => CreateRemoteThread => Victim => LoadLibraryA => Hook.dll
```

---

## بعد از Inject چه اتفاقی می‌افتد؟

ویندوز DllMain(DLL_PROCESS_ATTACH) را صدا می‌زند و در نتیجه این کد:

```cpp
BOOL APIENTRY DllMain(...){    
	InstallHook();
	DoSomeWork();
	return TRUE;
}
```

اجرا می‌شود.

---

## حالا Hook چگونه نصب می‌شود؟

فرض کن MessageBoxA در حافظه این شکلی است:

```
MessageBoxA:
55
8B EC
83 EC 10
...
```

چند بایت اول را برمی‌داریم:

```
55
8B EC
83 EC 10
```

و به جایش:

```
JMP MyHook
```

می‌گذاریم.

---

نتیجه:

قبل:

```
Application => MessageBoxA
```

بعد:

```
Application => MessageBoxA => JMP => MyHook
```

---

## پس Trampoline چیست؟

مشکل:

اگر تمام MessageBoxA را نابود کنیم Original Function دیگر وجود ندارد. پس بایت‌های اصلی را کپی می‌کنیم در یک حافظه جدید

```
55
8B EC
83 EC 10
```

و آخر اجرای توابع خودمان، برمیگردیم و 

```
JMP Back
```

می‌گذاریم. در نتیجه به این صورت:

```
Application => Hook => Trampoline => Original Function
```

---

## کتابخانه MinHook چه می‌کند؟

کتابخانه‌هایی مثل MinHook و Detours این کارها را انجام می‌دهند:

```
Find Function
Patch Function
Create Trampoline
Manage Memory
```

بنابراین لازم نیست خودتون اسمبلی بنویسین.

---
## مثال عملی

برای مشاهده یک مثال عملی به مقاله [[آموزش ساده DLL Injection و API Hook با MinHook در C++]] مراجعه کنید.