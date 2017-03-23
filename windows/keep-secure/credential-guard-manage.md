---
title: Manage Credential Guard (Windows 10)
description: Deploying and managing Credential Guard using Group Policy, the registry, or the Device Guard and Credential Guard hardware readiness tool.
ms.prod: w10
ms.mktglfcycl: explore
ms.sitesec: library
ms.pagetype: security
localizationpriority: high
author: brianlic-msft
---

# Manage Credential Guard

**Applies to**
-   Windows 10
-   Windows Server 2016

Prefer video?

[![Deploying Credential Guard](images/mva_videos.png)](https://mva.microsoft.com/en-us/training-courses/deep-dive-into-credential-guard-16651?l=sRcyvLJyC_3304300474)


## Enable Credential Guard
Credential Guard can be enabled by using [Group Policy](#turn-on-credential-guard-by-using-group-policy), the [registry](#turn-on-credential-guard-by-using-the-registry), or the Device Guard and Credential Guard [hardware readiness tool](#hardware-readiness-tool).

### Enable Credential Guard by using Group Policy

You can use Group Policy to enable Credential Guard. This will add and enable the virtualization-based security features for you if needed.

1.  From the Group Policy Management Console, go to **Computer Configuration** -&gt; **Administrative Templates** -&gt; **System** -&gt; **Device Guard**.
2.  Double-click **Turn On Virtualization Based Security**, and then click the **Enabled** option.
3.  **Select Platform Security Level** box, choose **Secure Boot** or **Secure Boot and DMA Protection**.
4.  In the **Credential Guard Configuration** box, click **Enabled with UEFI lock**, and then click **OK**. If you want to be able to turn off Credential Guard remotely, choose **Enabled without lock**.

    ![Credential Guard Group Policy setting](images/credguard-gp.png)
    
5.  Close the Group Policy Management Console.

To enforce processing of the group policy, you can run ```gpupdate /force```. 


### Enable Credential Guard by using the registry

If you don't use Group Policy, you can enable Credential Guard by using the registry. Credential Guard uses virtualization-based security features which have to be enabled first on some operating systems.

### Add the virtualization-based security features

Starting with Windows 10, version 1607 and Windows Server 2016, enabling Windows features to use virtualization-based security is not necessary and this step can be skipped.

If you are using Windows 10, version 1507 (RTM) or Windows 10, version 1511, Windows features have to be enabled to use virtualization-based security. 
You can do this by using either the Control Panel or the Deployment Image Servicing and Management tool (DISM).
> [!NOTE]  
If you enable Credential Guard by using Group Policy, the steps to enable Windows features through Control Panel or DISM are not required. Group Policy will install Windows features for you.

 
**Add the virtualization-based security features by using Programs and Features**

1.  Open the Programs and Features control panel.
2.  Click **Turn Windows feature on or off**.
3.  Go to **Hyper-V** -&gt; **Hyper-V Platform**, and then select the **Hyper-V Hypervisor** check box.
4.  Select the **Isolated User Mode** check box at the top level of the feature selection. 
5.  Click **OK**.

**Add the virtualization-based security features to an offline image by using DISM**

1.  Open an elevated command prompt.
2.  Add the Hyper-V Hypervisor by running the following command:
    ```
    dism /image:<WIM file name> /Enable-Feature /FeatureName:Microsoft-Hyper-V-Hypervisor /all
    ```
3. Add the Isolated User Mode feature by running the following command:
    ```
    dism /image:<WIM file name> /Enable-Feature /FeatureName:IsolatedUserMode
    ```

> [!NOTE]  
> You can also add these features to an online image by using either DISM or Configuration Manager.

### Enable virtualization-based security and Credential Guard

1.  Open Registry Editor.
2.  Enable virtualization-based security:
    -   Go to HKEY\_LOCAL\_MACHINE\\System\\CurrentControlSet\\Control\\DeviceGuard.
    -   Add a new DWORD value named **EnableVirtualizationBasedSecurity**. Set the value of this registry setting to 1 to enable virtualization-based security and set it to 0 to disable it.
    -   Add a new DWORD value named **RequirePlatformSecurityFeatures**. Set the value of this registry setting to 1 to use **Secure Boot** only or set it to 3 to use **Secure Boot and DMA protection**.
3.  Enable Credential Guard:
    -   Go to HKEY\_LOCAL\_MACHINE\\System\\CurrentControlSet\\Control\\LSA.
    -   Add a new DWORD value named **LsaCfgFlags**. Set the value of this registry setting to 1 to enable Credential Guard with UEFI lock, set it to 2 to enable Credential Guard without lock, and set it to 0 to disable it.
4.  Close Registry Editor.


> [!NOTE]  
> You can also enable Credential Guard by setting the registry entries in the [FirstLogonCommands](http://msdn.microsoft.com/library/windows/hardware/dn922797.aspx) unattend setting.

<span id="hardware-readiness-tool" />
### Enable Credential Guard by using the Device Guard and Credential Guard hardware readiness tool

You can also enable Credential Guard by using the [Device Guard and Credential Guard hardware readiness tool](https://www.microsoft.com/download/details.aspx?id=53337).

```
DG_Readiness_Tool_v3.0.ps1 -Enable -AutoReboot
```

### Credential Guard deployment in virtual machines

Credential Guard can protect secrets in a Hyper-V virtual machine, just as it would on a physical machine. The enablement steps are the same from within the virtual machine.

Credential Guard protects secrets from non-privileged access inside the VM. It does not provide additional protection from the host administrator. From the host, you can disable Credential Guard for a virtual machine:

``` PowerShell
Set-VMSecurity -VMName <VMName> -VirtualizationBasedSecurityOptOut $true
```

Requirements for running Credential Guard in Hyper-V virtual machines
- The Hyper-V host must have an IOMMU, and run at least Windows Server 2016 or Windows 10 version 1607.
- The Hyper-V virtual machine must be Generation 2, have an enabled virtual TPM, and running at least Windows Server 2016 or Windows 10. 


### Check that Credential Guard is running

You can use System Information to ensure that Credential Guard is running on a PC.

1.  Click **Start**, type **msinfo32.exe**, and then click **System Information**.
2.  Click **System Summary**.
3.  Confirm that **Credential Guard** is shown next to **Device Guard Security Services Running**.

    Here's an example:
    
    ![System Information](images/credguard-msinfo32.png)

You can also check that Credential Guard is running by using the [Device Guard and Credential Guard hardware readiness tool](https://www.microsoft.com/download/details.aspx?id=53337).

```
DG_Readiness_Tool_v3.0.ps1 -Ready
```


### Remove Credential Guard

If you have to remove Credential Guard on a PC, you can use the following set of procedures, or you can [use the Device Guard and Credential Guard hardware readiness tool](#turn-off-with-hardware-readiness-tool).

1.  If you used Group Policy, disable the Group Policy setting that you used to enable Credential Guard (**Computer Configuration** -&gt; **Administrative Templates** -&gt; **System** -&gt; **Device Guard** -&gt; **Turn on Virtualization Based Security**).
2.  Delete the following registry settings:
    - HKEY\_LOCAL\_MACHINE\\System\\CurrentControlSet\\Control\\LSA\LsaCfgFlags
    - HKEY\_LOCAL\_MACHINE\\Software\\Policies\\Microsoft\\Windows\\DeviceGuard\\EnableVirtualizationBasedSecurity
    - HKEY\_LOCAL\_MACHINE\\Software\\Policies\\Microsoft\\Windows\\DeviceGuard\\RequirePlatformSecurityFeatures

    > [!IMPORTANT]  
    > If you manually remove these registry settings, make sure to delete them all. If you don't remove them all, the device might go into BitLocker recovery.

3.  Delete the Credential Guard EFI variables by using bcdedit.

**Delete the Credential Guard EFI variables**

1.  From an elevated command prompt, type the following commands:
    ``` syntax

    mountvol X: /s
    
    copy %WINDIR%\System32\SecConfig.efi X:\EFI\Microsoft\Boot\SecConfig.efi /Y
    
    bcdedit /create {0cb3b571-2f2e-4343-a879-d86a476d7215} /d "DebugTool" /application osloader
    
    bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} path "\EFI\Microsoft\Boot\SecConfig.efi"
    
    bcdedit /set {bootmgr} bootsequence {0cb3b571-2f2e-4343-a879-d86a476d7215}
    
    bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} loadoptions DISABLE-LSA-ISO
    
    bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} device partition=X:
    
    mountvol X: /d
    
    ```
2.  Restart the PC.
3.  Accept the prompt to disable Credential Guard.
4.  Alternatively, you can disable the virtualization-based security features to turn off Credential Guard.

> [!NOTE]  
> The PC must have one-time access to a domain controller to decrypt content, such as files that were encrypted with EFS. If you want to turn off both Credential Guard and virtualization-based security, run the following bcdedit command after turning off all virtualization-based security Group Policy and registry settings: bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} loadoptions DISABLE-LSA-ISO,DISABLE-VBS

For more info on virtualization-based security and Device Guard, see [Device Guard deployment guide](device-guard-deployment-guide.md).

<span id="turn-off-with-hardware-readiness-tool" />
#### Turn off Credential Guard by using the Device Guard and Credential Guard hardware readiness tool

You can also disable Credential Guard by using the [Device Guard and Credential Guard hardware readiness tool](https://www.microsoft.com/download/details.aspx?id=53337).

```
DG_Readiness_Tool_v3.0.ps1 -Disable -AutoReboot
```
 