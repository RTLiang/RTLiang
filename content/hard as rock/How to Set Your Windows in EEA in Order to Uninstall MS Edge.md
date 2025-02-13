---
title: How to Set Your Windows in EEA in Order to Uninstall MS Edge
draft: false
tags:
  - dev
date: 2025-02-13
---

 Due to European Union's Digital Markets Act (DMA)[^1], users in European Economic Area (EEA)[^2] are able to uninstall the notorious Microsoft Edge in their Windows Devices[^3]. If you are not in EEA, this tutorial may help you configure your Windows in EEA and uninstall MS Edge.
 
## (I)  Disable UCPD [^4]

 UserChoice Protection Driver (UCPD) is a system driver that preventing executables from changing `http`, `https` and `.pdf` file associations, it also prevent us from editing registry[^5]. So we need to disable it.

### 1. Check UCPD status
Execute the following command in **CMD**(Admin) to show **UCPD running status**:
```cmd
sc query ucpd
```
If you see **`STATE              : 4  RUNNING`**, then UCPD is running.

For example:
```cmd
C:\>sc query ucpd

SERVICE_NAME: ucpd
        TYPE               : 2  FILE_SYSTEM_DRIVER
=====>  STATE              : 4  RUNNING   <=====
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

Execute the following command in **CMD**(Admin) to show **service running status**:
```cmd
sc qc ucpd
```
If you see **`START_TYPE         : 1   SYSTEM_START`**, then UCPD is triggered at login.

For example:
```cmd
C:\>sc qc ucpd
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: ucpd
        TYPE               : 2  FILE_SYSTEM_DRIVER
  ====> START_TYPE         : 1   SYSTEM_START   <====
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : system32\drivers\UCPD.sys
        LOAD_ORDER_GROUP   : FSFilter Activity Monitor
        TAG                : 0
        DISPLAY_NAME       : UCPD
        DEPENDENCIES       :
        SERVICE_START_NAME :
```

> [!tip] Don't close the CMD window, we will need it later!
### 2. Disable UCPD and its service

#### a. Disable UCPD TaskPlaner job 
Microsoft has stored a TaskPlaner job to reactivate UCPD when it is disabled, so before disabling UCPD itself, we need to disable the TaskPlaner job first.

Search `Task Scheduler` and navigate to `\Microsoft\Windows\AppxDeploymentClient`, you will see the Status of `UCPD velocity` is Ready.
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/a480f618ddd514b30bcde9dd7d6a9a26068.png)
*sensitive info is masked*

Right click it and select `Properties`.
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/4496c9d447ec74de22369af054a7cdff115.png)

In properties section, click `Conditions`, uncheck `Start the task only if the computer is idle for:` and click OK.
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/1f3ad9fbe5c6b889a86bf912ab29f07e818.png)

Right click again and Disable it.
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/ed2cbbc4852ac6ff2f8aee56d70c6443395.png)

Now We've disabled the TaskPlaner job, let's disable UCPD now.

#### b. Disable UCPD
Execute the following command in **CMD**(Admin) to **disable UCPD**:
```cmd
sc config UCPD start= disabled
```
It shows:
```cmd
C:\>sc config UCPD start= disabled
[SC] ChangeServiceConfig SUCCESS
```
Now we can check UCPD status to confirm it is indeed disabled.
```cmd
sc qc ucpd
```

```cmd
C:\>sc qc ucpd
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: ucpd
        TYPE               : 2  FILE_SYSTEM_DRIVER
 ====>  START_TYPE         : 4   DISABLED <========
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : system32\drivers\UCPD.sys
        LOAD_ORDER_GROUP   : FSFilter Activity Monitor
        TAG                : 0
        DISPLAY_NAME       : UCPD
        DEPENDENCIES       :
        SERVICE_START_NAME :
```
Execute `sc query ucpd` and it turns out, ucpd is still running, the next step is reboot your machine. 
> [!tip] `ucpd` can not be stopped
> If you tried stop the service using `sc stop ucpd`, you will get a failure response that
> ```
> [SC] ControlService FAILED 1052:
>
>The requested control is not valid for this service.
> ```
> So we have to restart Windows.

After rebooting, ucpd is `STOPPED`.
```cmd
C:\>sc query ucpd

SERVICE_NAME: ucpd
        TYPE               : 2  FILE_SYSTEM_DRIVER
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 1077  (0x435)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

```

## (II) Change to a EEA country 

### 1. Choose a country
Select a EEA country in [^2] and search the two char country code in [ISO 3166-1 alpha-2 - Wikipedia](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements) and find its "Geographical location identifier"(GEOID) in [Table of Geographical Locations - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/intl/table-of-geographical-locations).

For example:

|Country|two char code|GEOID(Hex)|GEOID(decimal)|
|---|---|---|---|
|Ireland|IE|0x44|68|
> [!tip] Ireland is the only country in EEA use English as its official language

[^6]
### 2. Edit Registry [^7]
Edit `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Control Panel\DeviceRegion\DeviceRegion` to your selected GEOID. 
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/5ab4accfc558129e0855c0d58aa98e0b954.png)


Edit `HKEY_USERS\.DEFAULT\Control Panel\International\Geo\Name` to your selected two char code and edit `HKEY_USERS\.DEFAULT\Control Panel\International\Geo\Nation` to GEOID.
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/0ddd364b87923bf9e9610b1017b236fa874.png)
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/71caab72fae7fbd4a402f6030cb26fc5421.png)
Same for `HKEY_CURRENT_USER\Control Panel\International\Geo\Name` and `HKEY_CURRENT_USER\Control Panel\International\Geo\Nation`
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/f2922c719707da05f57f09fc38934ed2597.png)
### 3. Change Country or Region in Settings
In Windows Settings, `Time & Language > Language & Region`, Change `Country or region` to your selected country
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/f2f0ecc2ed3e5d4eca9801cb14235d5b355.png)
*sensitive info is masked*

Then reboot.

> [!info] I don't know if it will be reverted by Windows Update.

## (III) Check if effects are taken or not
There are a few ways to check whether we switched the region to EEA.
### 1. use MSEdgeRedirect
If anything goes well, your device's `Machine Region`, `Default Region`, `User Region` are all the region you selected, you can check this out with [MSEdgeRedirect](https://github.com/rcmaehl/MSEdgeRedirect).
If `User Region` and `Default Region` are different and you are sure you didn't miss any step, don't be upset, try to reboot multiple times, they will be same magically.
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/dc971cf5f949d3dbad4f7176840a4de0366.png)

### 2. MS Edge account will be logout

Once you set your region to EEA, your MS Edge account will be logout and it prompts you to sign in again.
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/aa12831796d37f32a17aa5ae4fcd25a6777.png)
*sensitive info is masked*

### 3. Try to uninstall MS Edge
You are able to uninstall MS Edge(that's what we want!).
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/b11840e2abcb4c70fece8c501a8a4b07798.png)



## (IV) Change UCPD Back
Now we've successfully tricked Windows as if we are physically in EEA countries, the last step is change the UCPD settings back(you don't want your default apps being hijacked like MS Edge and some sensitive registry values being edited by some malwares, don't you?). What we need to do is doing (I) reversely.

Enable UCPD TaskPlaner job.
![image.png](https://pub-b7259f73aa5840209c979dded8c55365.r2.dev/2025/02/cc1e45f62d5e257b649148973ce7df7e447.png)
*sensitive info is masked*

Execute the following command in **CMD**(Admin) to **enable UCPD**:
```cmd
sc config UCPD start= system
```
It shows:
```cmd
C:\>sc config UCPD start=system
[SC] ChangeServiceConfig SUCCESS
```
And we check this using `sc qc ucpd`

```cmd
C:\>sc qc ucpd
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: ucpd
        TYPE               : 2  FILE_SYSTEM_DRIVER
        START_TYPE         : 1   SYSTEM_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : system32\drivers\UCPD.sys
        LOAD_ORDER_GROUP   : FSFilter Activity Monitor
        TAG                : 0
        DISPLAY_NAME       : UCPD
        DEPENDENCIES       :
        SERVICE_START_NAME :
```

Now UCPD is stopped if we check it using `sc query ucpd`.
```cmd
C:\>sc query ucpd

SERVICE_NAME: ucpd
        TYPE               : 2  FILE_SYSTEM_DRIVER
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 1077  (0x435)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```
We only need to start it using:
```cmd
sc start ucpd
```
Now ucpd is back.
```cmd
C:\>sc start ucpd

SERVICE_NAME: ucpd
        TYPE               : 2  FILE_SYSTEM_DRIVER
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
        PID                : 0
        FLAGS              :
```

*reboot seems work as well but I haven't tried it.*


## Links may useful
Last but not least, there are a few links you may refer to, which may give a helping hand.

[UCPD blocking Europe Mode · Issue #404 · rcmaehl/MSEdgeRedirect](https://github.com/rcmaehl/MSEdgeRedirect/issues/404)

[Revert the Europe EEA Machine Region · Issue #392 · rcmaehl/MSEdgeRedirect](https://github.com/rcmaehl/MSEdgeRedirect/issues/392)

If you want to explore more about UCPD itself, you can go to [^4] or simply go to `C:\Windows\System32\IntegratedServicesRegionPolicySet.json` to see other region lock features.


[^1]:[The Digital Markets Act](https://digital-markets-act.ec.europa.eu/index_en)
[^2]:[EU, EEA, EFTA and Schengen Area countries | European Union | Government.nl](https://www.government.nl/topics/european-union/eu-eea-efta-and-schengen-area-countries)
[^3]: [Microsoft implements DMA compliance measures - EU Policy Blog](https://blogs.microsoft.com/eupolicy/2024/03/07/microsoft-dma-compliance-windows-linkedin/)
[^4]: [**(German)** Windows UserChoice Protection Driver UCPD / UCPD.sys / UCPDMgr.exe – Gunnar Haslinger](https://hitco.at/blog/windows-userchoice-protection-driver-ucpd/)
[^5]:[UserChoice Protection Driver – UCPD.sys – the kolbicz blog](https://kolbi.cz/blog/2024/04/03/userchoice-protection-driver-ucpd-sys/)

[^6]: [List of countries and territories where English is an official language - Wikipedia](https://en.wikipedia.org/wiki/List_of_countries_and_territories_where_English_is_an_official_language#English_is_a_de_jure_official_language)

[^7]: [Last update breaks Explorer and Settings in Windows 10 (+ Europe Mode does not work as intended) · Issue #367 · rcmaehl/MSEdgeRedirect](https://github.com/rcmaehl/MSEdgeRedirect/issues/367)

