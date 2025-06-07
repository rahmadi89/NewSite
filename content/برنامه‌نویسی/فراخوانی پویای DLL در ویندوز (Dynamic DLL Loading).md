بعضی وقت‌ها پیش میاد که لازمه به صورت پویا یه DLL رو فراخونی کنیم. مثلا اگه بخوایم از تابع [RtlGetVersion](https://learn.microsoft.com/en-us/windows/win32/devnotes/rtlgetversion) استفاده کنیم، همونطور که توی صفحه مربوطه ش نوشته توی فایل ntdll.dll عه که باید به این صورت اضافه بشه به پروژه:
```cpp
#incldue <Ntddk.h>
```

اما همونطور که تو این شکل معلومه، اجازه همچین کاری رو نداریم:
![[Pasted image 20241109152940.png]]

بنابراین برای اینکه بتونیم از این تابع استفاده کنیم میایم و ntdll.dll رو به صورت داینامیک لود می‌کنیم.
![[Pasted image 20241109153606.png]]

اول یه نگاه به کد بندازیم بعد توضیح بدم:
```cpp
#include <Windows.h>
#include <iostream>
int main() {
	typedef NTSTATUS(NTAPI * RtlGetVersionFunction)(PRTL_OSVERSIONINFOW);

	OSVERSIONINFOEX result;
	ZeroMemory(&result, sizeof(OSVERSIONINFOEX));
	result.dwOSVersionInfoSize = sizeof(OSVERSIONINFOEX);

	const HMODULE ntdll = GetModuleHandleW(L"ntdll.dll");
	if (ntdll) {
		const auto pRtlGetVersion = reinterpret_cast<RtlGetVersionFunction>(GetProcAddress(ntdll, "RtlGetVersion"));
		if (pRtlGetVersion) {
			pRtlGetVersion(reinterpret_cast<PRTL_OSVERSIONINFOW>(&result));
		}
	}

	std::cout << result.dwMajorVersion << "\n";
	std::cout << result.dwMinorVersion << "\n";
	std::cout << result.dwBuildNumber << "\n" ;
}
```

توضیح اینکه ما برای اینکه بتونیم تابع RtlGetVersion رو صدا بزنیم، اول باید یه هندل از ntdll.dll بگیریم و بعد توی این فایل بگردیم و آدرس تابع RtlGetVersion رو پیدا کنیم که این کار رو تابع GetProcAddress انجام میده برامون. بعد اینکه آدرسش رو گرفتیم باید اون رو تبدیلش کنیم به یه تابعی که نوع ورودی و خروجیش همونی باشه که تو داکیومنت گفته یعنی این:
![[Pasted image 20241109153951.png]]

به خاطر همین هم اومدیم یه typedef تعریف کردیم به اسم RtlGetVersionFunction که ورودی و خروجیش دقیقا همینا بود و بعد اینکه آدرس تابع RtlGetVersion رو پیدا کردیم با استفاده از reinterpret_cast تبدیلش کردیم به همین نوع. (در واقع آدرس مربوطه رو گرفتیم و تبدیلش کردیم به تابع مناسب، هنوز تابع رو صدا نزدیم)

در نهایت توی این خط تابع موردنظر رو صدا می‌زنیم:
```cpp
pRtlGetVersion(reinterpret_cast<PRTL_OSVERSIONINFOW>(&result));
```
و البته حواسمون باید باشه که ورودی ای که بهش می‌خوایم بدیم تبدیل به نوع درستش بشه.

همین ، این کل کاری بود که باید برای فراخوانی داینامیک dll انجام بدیم.