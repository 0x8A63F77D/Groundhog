# UWF WMI Provider Reference (verbatim transcription)

> **Purpose**: machine-usable reference for designing `IUwfProvider`.
> **Method**: verbatim transcription from Microsoft Learn. No inference. Anything the docs do not state is marked `[文档未说明]`.
> **Namespace**: `root\standardcimv2\embedded`
> **Entry page**: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-wmi-provider-reference> (page "Last updated on 2025-01-17")
> **Transcribed**: 2026-08-17
> **Locale**: en-US pages are authoritative.

## Transcription conventions

- `SOURCE:` lines give the exact page each block came from.
- Blockquoted / fenced text is copied verbatim from the page (whitespace inside MOF blocks was reflowed by the extractor — see the note under each Syntax block).
- `[文档未说明]` = the documentation does not state this. Do not fill in.
- `[与 brief 冲突: ...]` = page text contradicts a "known pitfall" in the project brief. Recorded, not adjudicated.

---

## Index page: Unified Write Filter WMI provider reference

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-wmi-provider-reference>

Intro text (verbatim):

> To help protect physical storage media, you can use the WMI providers for Unified Write Filter (UWF) to configure UWF.
>
> This section describes the WMI provider classes for UWF.

### In this section (verbatim table)

| Classes | Description |
| --- | --- |
| UWF_ExcludedFile | A container class that contains the files and folders that are currently in the file exclusion list for a volume protected by UWF. |
| UWF_ExcludedRegistryKey | A container class that contains the registry keys that are currently in the registry key exclusion list for UWF. |
| UWF_Filter | Enables or disables Unified Write Filter (UWF), resets configuration settings for UWF, and shuts down or restarts your device. |
| UWF_Overlay | Contains the current size of the UWF overlay and manages the critical and warning thresholds for the overlay size. |
| UWF_OverlayConfig | Manages the configuration of the UWF overlay. |
| UWF_OverlayFile | Displays and configures global settings for the UWF overlay. You can modify the maximum size and the type of the UWF overlay. |
| UWF_RegistryFilter | Adds or removes registry exclusions from UWF filtering. |
| UWF_Servicing | Contains properties and methods that enable you to query and control UWF servicing mode. |
| UWF_Volume | Manages a volume protected by UWF. |

Note (verbatim):

> We recommend setting the authentication level to PacketIntegrity or PacketPrivacy for remote clients when you connect to WMI providers under root\standardcimv2\embedded when using WMI scripts or applications. For more information about how to use authentication with WMI providers, see this WMI Enhancements in Windows PowerShell 2.0 CTP on TechNet.

### Requirements (verbatim, identical on every class page unless noted)

| Windows Edition | Supported |
| --- | --- |
| Windows Home | No |
| Windows Pro | No |
| Windows Enterprise | Yes |
| Windows Education | Yes |
| Windows IoT Enterprise | Yes |

**Note on `UWF_ServicingHelper`**: the index page does **not** list a `UWF_ServicingHelper` class. See the "Missing / not found" section at the end of this file.

---

## UWF_Filter

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-filter> (Last updated on 2025-01-17)

Description (verbatim):

> Enables or disables Unified Write Filter (UWF), resets configuration settings for UWF, and shuts down or restarts your device.

### MOF definition (verbatim from the page's Syntax block)

```mof
class UWF_Filter{
    [key] string Id;
    [read] boolean CurrentEnabled;
    [read] boolean NextEnabled;

    UInt32 Enable();
    UInt32 Disable();
    UInt32 ResetSettings();
    UInt32 ShutdownSystem();
    UInt32 RestartSystem();
};
```

*(The extractor returned this on one line: `class UWF_Filter{ [key] string Id; [read] boolean CurrentEnabled; [read] boolean NextEnabled; UInt32 Enable(); UInt32 Disable(); UInt32 ResetSettings(); UInt32 ShutdownSystem(); UInt32 RestartSystem(); };` — tokens are verbatim, line breaks reinstated for readability.)*

### Properties (verbatim table)

| Property | Data type | Qualifiers | Description |
| --- | --- | --- | --- |
| **Id** | string | [key] | A unique ID. This is always set to **UWF_Filter** |
| **CurrentEnabled** | Boolean | [read] | Indicates if UWF is enabled for the current session. |
| **NextEnabled** | Boolean | [read] | Indicates if UWF is enabled after the next restart. |

Read/write derivation: `Id` is `[key]`; `CurrentEnabled` and `NextEnabled` carry `[read]` (read-only). Whether `Id` is writable is `[文档未说明]`.

### Methods (verbatim table)

| Methods | Description |
| --- | --- |
| UWF_Filter.Enable | Enables UWF on the next restart. |
| UWF_Filter.Disable | Disables UWF on the next restart. |
| UWF_Filter.ResetSettings | Restores UWF settings to the original state that was captured at install time. |
| UWF_Filter.ShutdownSystem | Safely shuts down a system protected by UWF, even if the overlay is full. |
| UWF_Filter.RestartSystem | Safely restarts a system protected by UWF, even if the overlay is full. |

### Class Remarks (verbatim)

> You must use an administrator account to make any changes to the configuration settings for UWF. Users with any kind of account can read the current configuration settings.

---

#### UWF_Filter.Enable

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-filterenable>

Description (verbatim): "Enables Unified Write Filter (UWF) on the next restart."

Signature (verbatim):

```powershell
UInt32 Enable();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

(links: <https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-non-error-constants>, <https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-error-constants>) — no per-value HRESULT table is given on the page.

Remarks (verbatim):

> You must use an administrator account to enable UWF.
>
> You must restart your device after you enable or disable UWF before the change takes effect.
>
> The first time you enable UWF on your device, UWF makes the following changes to your system to improve the performance of UWF:
>
> * Paging files are disabled.
> * System restore is disabled.
> * SuperFetch is disabled.
> * File indexing service is turned off.
> * Defragmentation service is turned off.
> * Fast boot is disabled.
> * BCD setting **bootstatuspolicy** is set to **ignoreallfailures**.
>
> You can change these settings after you enable UWF if you want to. For example, you can move the page file location to an unprotected volume and re-enable paging files.
>
> Additionally, after you run `uwfmgr filter enable`, restart the computer and exit the servicing mode, the following things are disabled:
>
> * Windows Update by setting `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU\NoAutoUpdate`
> * Windows Store Update by setting `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\WindowsStore\AutoDownload`
> * Registry Reorganization by setting `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Configuration Manager\RegistryReorganizationLimitDays`
> * Maintenance Hour by setting `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\Maintenance\MaintenanceDisabled`
>
> After you run `uwfmgr filter disable`, restart the computer and enter the serving mode, the changes are reverted.

---

#### UWF_Filter.Disable

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-filterdisable>

Description (verbatim): "Disables Unified Write Filter (UWF) on the next restart."

Signature (verbatim):

```powershell
UInt32 Disable();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> You must use an administrator account to disable UWF.

---

#### UWF_Filter.ResetSettings

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-filterresetsettings>

Description (verbatim): "Restores UWF settings to the original configuration settings."

Signature (verbatim):

```powershell
UInt32 ResetSettings();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> You must use an administrator account to reset UWF settings.
>
> The original configuration settings are captured the first time that you enable UWF after you add UWF to your device by using **Turn Windows features on or off**. You can change the original configuration settings by using **Turn Windows features on or off** to remove and then add UWF, and then modifying the configuration to the desired state before you enable UWF.
>
> If you added UWF to your device by using SMI settings in an unattend.xml file, the original configuration settings are captured when Windows 10 Enterprise is installed on your device.

---

#### UWF_Filter.ShutdownSystem

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-filtershutdownsystem>

Description (verbatim): "Safely shuts down a system protected by UWF, even if the overlay is full."

Signature (verbatim):

```powershell
UInt32 ShutdownSystem();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> You must use an administrator account to call this method.
>
> If the overlay is full, or near full, shutting down or restarting the system normally can cause the system to take an extremely long time to shut down. This occurs when the system repeatedly tries to write files during shutdown, which constantly fail due to the overlay being full. You can call this method to safely shut down a system by avoiding this scenario.
>
> If the overlay becomes full while the system is performing a large number of writes, such as copying a large group of files, calling this method can still result in a long shutdown time.

---

#### UWF_Filter.RestartSystem

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-filterrestartsystem>

Description (verbatim): "Safely restarts a system protected by UWF, even if the overlay is full."

Signature (verbatim):

```powershell
UInt32 RestartSystem();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> You must use an administrator account to call this method. You can't run on WMI providers; it's only available from Intune/CSP.
>
> If the overlay is full, or near full, shutting down or restarting the system normally can cause the system to take a long time to shut down. This occurs when the system repeatedly tries to write files during shutdown, which constantly fail due to the overlay being full. You can call this method to safely restart a system by avoiding this scenario.
>
> If the overlay becomes full while the system is performing a large number of writes, such as copying a large group of files, calling this method can still result in a long shutdown time.

> **⚠ Note for `IUwfProvider` design**: the RestartSystem page explicitly states "You can't run on WMI providers; it's only available from Intune/CSP." — while the class MOF on the UWF_Filter page still declares `UInt32 RestartSystem();`. Both statements transcribed as-is; not adjudicated.

---

### UWF_Filter — example code (verbatim from the page)

Page prose (verbatim):

> The following example demonstrates how to enable or disable UWF by using the WMI provider in a PowerShell script.
>
> The PowerShell script creates three functions to help enable or disable UWF. It then demonstrates how to use each function.
>
> The first function, `Disable-UWF`, retrieves a WMI object for **UWF_Filter**, and calls the **Disable()** method to disable UWF after the next device restart.
>
> The second function, `Enable-UWF`, retrieves a WMI object for **UWF_Filter**, and calls the **Enable()** method to enable UWF after the next device restart.
>
> The third function, `Display-UWFState`, examines the properties of the **UWF_Filter** object, and prints out the current settings for **UWF_Filter**.

Actual call patterns used in the sample:

```powershell
$COMPUTER = "localhost"
$NAMESPACE = "root\standardcimv2\embedded"

# Create a function to disable the Unified Write Filter driver after the next restart.
function Disable-UWF() {
    # Retrieve the UWF_Filter settings.
    $objUWFInstance = Get-WMIObject -namespace $NAMESPACE -class UWF_Filter;

    if(!$objUWFInstance) {
        "Unable to retrieve Unified Write Filter settings."
        return;
    }

    # Call the method to disable UWF after the next restart. This sets the NextEnabled property to false.
    $retval = $objUWFInstance.Disable();

    # Check the return value to verify that the disable is successful
    if ($retval.ReturnValue -eq 0) {
        "Unified Write Filter will be disabled after the next system restart."
    } else {
        "Unknown Error: " + "{0:x0}" -f $retval.ReturnValue
    }
}

# Create a function to enable the Unified Write Filter driver after the next restart.
function Enable-UWF() {
    # Retrieve the UWF_Filter settings.
    $objUWFInstance = Get-WMIObject -namespace $NAMESPACE -class UWF_Filter;

    if(!$objUWFInstance) {
        "Unable to retrieve Unified Write Filter settings."
        return;
    }

    # Call the method to enable UWF after the next restart. This sets the NextEnabled property to false.
    $retval = $objUWFInstance.Enable();

    # Check the return value to verify that the enable is successful
    if ($retval.ReturnValue -eq 0) {
        "Unified Write Filter will be enabled after the next system restart."
    } else {
        "Unknown Error: " + "{0:x0}" -f $retval.ReturnValue
    }
}

# Create a function to display the current settings of the Unified Write Filter driver.
function Display-UWFState() {
    # Retrieve the UWF_Filter object
    $objUWFInstance = Get-WmiObject -Namespace $NAMESPACE -Class UWF_Filter;

    if(!$objUWFInstance) {
        "Unable to retrieve Unified Write Filter settings."
        return;
    }

    # Check the CurrentEnabled property to see if UWF is enabled in the current session.
    if($objUWFInstance.CurrentEnabled) {
        $CurrentStatus = "enabled";
    } else {
        $CurrentStatus = "disabled";
    }

    # Check the NextEnabled property to see if UWF is enabled or disabled after the next system restart.
    if($objUWFInstance.NextEnabled) {
        $NextStatus = "enabled";
    } else {
        $NextStatus = "disabled";
    }
}

# Some examples of how to call the functions
Display-UWFState
"Enabling Unified Write Filter"
Enable-UWF
Display-UWFState
"Disabling Unified Write Filter"
Disable-UWF
Display-UWFState
```

Key facts for the provider interface, as literally shown:

- Retrieval: `Get-WMIObject -namespace "root\standardcimv2\embedded" -class UWF_Filter` — **no `-Filter` / WQL clause**; `UWF_Filter` is a singleton (`Id` always `UWF_Filter`).
- Method return is checked as `$retval.ReturnValue -eq 0`.
- The sample's `Display-UWFState` assigns `$CurrentStatus` / `$NextStatus` but never prints them (verbatim from the page — the docs' own sample is incomplete).

---

## UWF_Volume

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volume> (Last updated on 2025-01-17)

Description (verbatim):

> This class manages a volume protected by Unified Write Filter (UWF).

### MOF definition (verbatim from the page's Syntax block)

```mof
class UWF_Volume {
    [key, Read] boolean CurrentSession;
    [key, Read] string DriveLetter;
    [key, Read] string VolumeName;
    [Read, Write] boolean BindByDriveLetter;
    [Read] boolean CommitPending;
    [Read, Write] boolean Protected;

    UInt32 CommitFile([in] string FileFullPath);
    UInt32 CommitFileDeletion(string FileName);
    UInt32 Protect();
    UInt32 Unprotect();
    UInt32 SetBindByDriveLetter(boolean bBindByVolumeName);
    UInt32 AddExclusion(string FileName);
    UInt32 RemoveExclusion(string FileName);
    UInt32 RemoveAllExclusions();
    UInt32 FindExclusion([in] string FileName, [out] bFound);
    UInt32 GetExclusions([out, EmbeddedInstance("UWF_ExcludedFile")] string ExcludedFiles[]);
};
```

*(Extractor returned this on one line; tokens verbatim, line breaks reinstated. Note the MOF's `FindExclusion` `[out] bFound` has **no declared type** in the published MOF — transcribed as printed.)*

> **⚠ Internal doc inconsistency (transcribed, not adjudicated):**
> - Class MOF: `UInt32 CommitFile([in] string FileFullPath);` — method page: `UInt32 CommitFile([in] string FileName);`
> - Class MOF: `UInt32 SetBindByDriveLetter(boolean bBindByVolumeName);` — method page: `UInt32 SetBindByDriveLetter(boolean bBindByDriveLetter);`

### Properties (verbatim table)

| Property | Data type | Qualifiers | Description |
| --- | --- | --- | --- |
| **BindByDriveLetter** | Boolean | [read, write] | Indicates the type of binding that the volume uses. - **True** to bind the volume by **DriveLetter** (loose binding) - **False** to bind the volume by **VolumeName** (tight binding). |
| **CommitPending** | Boolean | [read] | Reserved for Microsoft use. |
| **CurrentSession** | Boolean | [key, read] | Indicates which session the object contains settings for. - **True** if settings are for the current session - **False** if settings are for the next session that follows a restart. |
| **DriveLetter** | string | [key, read] | The drive letter of the volume. If the volume does not have a drive letter, this value is **NULL**. |
| **Protected** | Boolean | [read, write] | If **CurrentSession** is **true**, indicates whether the volume is currently protected by UWF. If **CurrentSession** is **false**, indicates whether the volume is protected in the next session after the device restarts. |
| **VolumeName** | string | [key, read] | The unique identifier of the volume on the current system. The **VolumeName** is the same as the **DeviceID** property of the Win32_Volume class for the volume. |

Key set: `CurrentSession`, `DriveLetter`, `VolumeName` (all three `[key, read]`).

`Win32_Volume` reference link on page: <https://learn.microsoft.com/en-us/previous-versions/windows/desktop/legacy/aa394515(v=vs.85)>

> `[与 brief 一致]` brief 第 40 节: "`UWF_Volume` 每卷两个实例，以 `CurrentSession` 布尔为 key 区分；只能修改 CurrentSession=false 的那个。" — Page's own sample comment states verbatim: "Each volume has two entries in UWF_Volume, one for the current session and one for the next session after a restart" and "You can only change the protection status of a drive for the next session". The docs do **not** state a general rule that *all* writable properties are only settable on the `CurrentSession=$false` instance — only the protection-status statement is explicit. `[文档未说明]` for the general case.

### Methods (verbatim table)

| Method | Description |
| --- | --- |
| UWF_Volume.AddExclusion | Adds a file or folder to the file exclusion list for a volume protected byUWF. |
| UWF_Volume.CommitFile | Commits changes from the overlay to the physical volume for a specified file on a volume protected by Unified Write Filter (UWF). |
| UWF_Volume.CommitFileDeletion | Deletes a protected file from the volume, and commits the deletion to the physical volume. |
| UWF_Volume.FindExclusion | Determines whether a specific file or folder is in the exclusion list for a volume protected byUWF. |
| UWF_Volume.GetExclusions | Retrieves a list of all file exclusions for a volume protected byUWF. |
| UWF_Volume.Protect | Protects the volume after the next system restart, if UWF is enabled after the restart. |
| UWF_Volume.RemoveAllExclusions | Removes all files and folders from the file exclusion list for a volume protected by UWF. |
| UWF_Volume.RemoveExclusion | Removes a specific file or folder from the file exclusion list for a volume protected byUWF. |
| UWF_Volume.SetBindByDriveLetter | Sets the **BindByDriveLetter** property, which indicates whether the UWF volume is bound to the physical volume by drive letter or by volume name. |
| UWF_Volume.Unprotect | Disables UWF protection of the volume after the next system restart. |

(the "byUWF" run-together spelling is verbatim from the page)

### Class Remarks (verbatim)

> You must use an administrator account to change any properties or call any methods that change the configuration settings.

---

#### UWF_Volume.AddExclusion

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumeaddexclusion>

Description (verbatim): "Adds a file or folder to the file exclusion list for a volume protected by Unified Write Filter (UWF)."

Signature (verbatim):

```powershell
UInt32 AddExclusion(
    string FileName
);
```

Parameters (verbatim):

> **FileName**
>
> A string that contains the full path of the file or folder relative to the volume.

(direction qualifier for `FileName` is not printed on this page — `[文档未说明]`; class MOF also prints it without `[in]`)

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> You must use an administrator account to add or remove file or folder exclusions during run time, and you must restart the device for new exclusions to take effect.
>
> **Important**
>
> You can’t add exclusions for the following items:
>
> * The volume root. For example, C: or D:.
> * The \Windows folder on the system volume.
> * The \Windows\System32 folder on the system volume.
> * The \Windows\system32\drivers folder on the system volume.
> * Paging files.
>
> However, you can exclude subdirectories and files under these items.

---

#### UWF_Volume.CommitFile

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumecommitfile>

Description (verbatim): "Commits changes from the overlay to the physical volume for a specified file on a volume protected by Unified Write Filter (UWF)."

Signature (verbatim):

```mof
UInt32 CommitFile( [in] string FileName );
```

Parameters (verbatim):

> **FileName**
> [in] A string that contains the path of the file to commit on the overlay, but does not include the drive letter or volume name. For example, “\users\test.dat”.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error constant.

Remarks (verbatim):

> The *FileName* must contain the name of a file that exists. The **CommitFile** method cannot commit a file that does not exist.
>
> You must use an administrator account to change any properties or call any methods that change the configuration settings.

---

#### UWF_Volume.CommitFileDeletion

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumecommitfiledeletion>

Description (verbatim): "Deletes the specified file and commits the deletion to the physical volume."

Signature (verbatim):

```powershell
UInt32 CommitFileDeletion(
    string FileName
);
```

Parameters (verbatim):

> **FileName**
>
> [in] A string that contains the path of the file to delete, but does not include the drive letter or volume name. For example: “\users\test.dat”.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error constant.

Remarks (verbatim):

> The *FileName* must contain the name of a file that exists on the physical volume. The **CommitFileDeletion** method cannot delete a file that does not exist.
>
> You must use an administrator account to call this method.

---

#### UWF_Volume.FindExclusion

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumefindexclusion>

Description (verbatim): "Checks if a specific file or folder is in the exclusion list for a volume protected by Unified Write Filter (UWF)."

Signature (verbatim):

```powershell
UInt32 FindExclusion (
    [in] string FileName,
    [out] boolean bFound
);
```

Parameters (verbatim):

> **FileName**
>
> [in] A string that contains the full path of the file or folder relative to the volume.
>
> **bFound**
>
> [out] Indicates if *FileName* is in the file exclusion list for the volume.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error constant.

Remarks (verbatim):

> **FindExclusion** sets *bFound* to **true** only for file and folder exclusions that have been explicitly added to the exclusion list. Files and subfolders that are in an excluded folder are not identified as excluded by **FindExclusion**, unless they have been explicitly excluded.

---

#### UWF_Volume.GetExclusions

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumegetexclusions>

Description (verbatim): "Gets a list of all file exclusions for a Unified Write Filter (UWF) protected volume."

Signature (verbatim):

```powershell
UInt32 GetExclusions(
    [out, EmbeddedInstance("UWF_ExcludedFile")] string ExcludedFiles[]
);
```

Parameters (verbatim):

> **ExcludedFiles**
>
> [out] An array of UWF_ExcludedFile objects that represent the files and folders that are excluded from UWF filtering for a volume.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error constant.

Remarks (verbatim):

> If **GetExclusions** does not find any files or folders in the file exclusion list for the volume, **GetExclusions** sets the *ExcludedFiles* parameter to null.

> **⚠ Design note (docs-literal)**: empty exclusion list ⇒ `ExcludedFiles` is **null**, not an empty array. Callers must distinguish null-from-empty rather than treating null as an error.

---

#### UWF_Volume.Protect

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumeprotect>

Description (verbatim): "Enables Unified Write Filter (UWF) to protect the volume after the next system restart, if UWF is enabled after the restart."

Signature (verbatim):

```powershell
UInt32 Protect();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error constant.

Remarks (verbatim):

> UWF starts protecting the volume after the next device restart in which UWF is enabled.
>
> This method does not enable UWF if it is disabled; you must explicitly enable UWF for the next session to start volume protection.

---

#### UWF_Volume.RemoveAllExclusions

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumeremoveallexclusions>

Description (verbatim): "Removes all files and folders from the file exclusion list for a volume protected by Unified Write Filter (UWF)."

Signature (verbatim):

```powershell
UInt32 RemoveAllExclusions();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI errorj constant.

(the typo "errorj" is verbatim from the page)

Remarks (verbatim):

> This command does not remove registry exclusions.
>
> You must use an administrator account to remove file or folder exclusions, and you must restart the device for this change to take effect.

---

#### UWF_Volume.RemoveExclusion

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumeremoveexclusion>

Description (verbatim): "Removes a specific file or folder from the file exclusion list for a volume protected by Unified Write Filter (UWF)."

Signature (verbatim):

```powershell
UInt32 RemoveExclusion(
    string FileName
);
```

Parameters (verbatim):

> **FileName**
>
> A string that contains the full path of the file or folder relative to the volume.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error constant.

Remarks (verbatim):

> You must use an administrator account to remove file or folder exclusions, and you must restart the device for this change to take effect.

---

#### UWF_Volume.SetBindByDriveLetter

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumesetbindbydriveletter>

Description (verbatim): "Sets the **BindByDriveLetter** property, which indicates if the Unified Write Filter (UWF) volume is bound to the physical volume by drive letter or volume name."

Signature (verbatim — page's code fence is labeled `powereshell`, a typo on the page):

```powershell
UInt32 SetBindByDriveLetter(
    boolean bBindByDriveLetter
);
```

Parameters (verbatim):

> **bBindByDriveLetter**
>
> A Boolean value that indicates the type of binding to use. The **BindByDriveLetter** property is set to this value.

| Value | Description |
| --- | --- |
| **true** | Binds the UWF volume by the drive letter (*loose binding*). |
| **false** | Binds the UWF volume by the volume name (*tight binding*). |

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> Binding by volume name is considered more reliable than binding by drive letter, since drive letters can change for a volume if devices are added or removed.

Whether this takes effect immediately or only after restart: `[文档未说明]`

---

#### UWF_Volume.Unprotect

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-volumeunprotect>

Description (verbatim): "Disables UWF protection of the volume after the next system restart."

Signature (verbatim):

```powershell
UInt32 Unprotect();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error constant.

Remarks (verbatim):

> Unprotecting the volume does not remove the UWF_Volume entry or any configuration settings from the UWF configuration registry. This means that you can unprotect a volume, and then protect it again later, while keeping any file exclusions or volume configurations that you have defined.

---

### UWF_Volume — example 1: "Turn UWF protection on or off" (verbatim)

Page prose (verbatim):

> The following example demonstrates how to protect or unprotect a volume with UWF by using the Windows Management Instrumentation (WMI) provider in a PowerShell script.
>
> The PowerShellscript creates a function, **Set-ProtectVolume**, that turns UWF protection on or off for a volume. The script then demonstrates how to use the function.

```powershell
$COMPUTER = "localhost"
$NAMESPACE = "root\standardcimv2\embedded"

# Define common parameters
$CommonParams = @{"namespace"=$NAMESPACE; "computer"=$COMPUTER}

# Create a function to protect or unprotect a volume based on the drive letter of the volume
function Set-ProtectVolume($driveLetter, [bool] $enabled) {

    # Each volume has two entries in UWF_Volume, one for the current session and one for the next session after a restart
    # You can only change the protection status of a drive for the next session
    $nextConfig = Get-WMIObject -class UWF_Volume @CommonParams |
        where { $_.DriveLetter -eq "$driveLetter" -and $_.CurrentSession -eq $false };

    # If a volume entry is found for the drive letter, enable or disable protection based on the $enabled parameter
    if ($nextConfig) {
        Write-Host "Setting drive protection on $driveLetter to $enabled"
        if ($Enabled -eq $true) {
            $nextConfig.Protect() | Out-Null;
        } else {
            $nextConfig.Unprotect() | Out-Null;
        }
    }
    # If the drive letter does not match a volume, create a new UWF_volume instance
    else {
        Write-Host "Error: Could not find $driveLetter. Protection is not enabled."
    }
}

# The following sample commands demonstrate how to use the Set-ProtectVolume function
# to protect and unprotect volumes
Set-ProtectVolume "C:" $true
Set-ProtectVolume "D:" $true
Set-ProtectVolume "C:" $false
```

### UWF_Volume — example 2: "Manage UWF file and folder exclusions" (verbatim)

Page prose (verbatim):

> The following example demonstrates how to manage UWF file and folder exclusions by using the WMI provider in a PowerShell script. The PowerShell script creates four functions, and then demonstrates how to use them.
>
> The first function, **Get-FileExclusions**, displays a list of UWF file exclusions that exist on a volume. Exclusions for both the current session and the next session that follows a restart are displayed.
>
> The second function, **Add-FileExclusion**, adds a file or folder to the UWF exclusion list for a given volume. The exclusion is added for the next session that follows a restart.
>
> The third function, **Remove-FileExclusion**, removes a file or folder from the UWF exclusion list for a given volume. The exclusion is removed for the next session that follows a restart.
>
> The fourth function, **Clear-FileExclusions**, removes all UWF file and folder exclusions from a given volume. The exclusions are removed for the next session that follows a restart.

```powershell
$COMPUTER = "localhost"
$NAMESPACE = "root\standardcimv2\embedded"

# Define common parameters
$CommonParams = @{"namespace"=$NAMESPACE; "computer"=$COMPUTER}

function Get-FileExclusions($driveLetter) {

# This function lists the UWF file exclusions for a volume, both
# for the current session as well as the next session after a restart
# $driveLetter is the drive letter of the volume

    # Get the UWF_Volume configuration for the current session
    $currentConfig = Get-WMIObject -class UWF_Volume @CommonParams |
        where { $_.DriveLetter -eq "$driveLetter" -and $_.CurrentSession -eq $true };

    # Get the UWF_Volume configuration for the next session after a restart
    $nextConfig = Get-WMIObject -class UWF_Volume @CommonParams |
        where { $_.DriveLetter -eq "$driveLetter" -and $_.CurrentSession -eq $false };

    # Display file exclusions for the current session
    if ($currentConfig) {
        Write-Host "The following files and folders are currently excluded from UWF filtering for $driveLetter";
        $currentExcludedList = $currentConfig.GetExclusions()
        if ($currentExcludedList) {
            foreach ($fileExclusion in $currentExcludedList.ExcludedFiles) {
                Write-Host "  " $fileExclusion.FileName
            }
        } else {
            Write-Host "  None"
        }
    } else {
        Write-Error "Could not find drive $driveLetter";
    }

    # Display file exclusions for the next session after a restart
    if ($nextConfig) {
        Write-Host ""
        Write-Host "The following files and folders will be excluded from UWF filtering for $driveLetter after the next restart:";
        $nextExcludedList = $nextConfig.GetExclusions()
        if ($nextExcludedList) {
            foreach ($fileExclusion in $nextExcludedList.ExcludedFiles) {
                Write-Host "  " $fileExclusion.FileName
            }
        } else {
            Write-Host "  None"
        }
        Write-Host ""
    }
}

function Add-FileExclusion($driveLetter, $exclusion) {

# This function adds a new UWF file exclusion to a volume
# The new file exclusion takes effect the next time the device is restarted and UWF is enabled
# $driveLetter is the drive letter of the volume
# $exclusion is the path and filename of the file or folder exclusion

    # Get the configuration for the next session for the volume
    $nextConfig = Get-WMIObject -class UWF_Volume @CommonParams |
        where { $_.DriveLetter -eq "$driveLetter" -and $_.CurrentSession -eq $false };

    # Add the exclusion
    if ($nextConfig) {
        $nextConfig.AddExclusion($exclusion) | Out-Null;
        Write-Host "Added exclusion $exclusion for $driveLetter";
    } else {
        Write-Error "Could not find drive $driveLetter";
    }
}

function Remove-FileExclusion($driveLetter, $exclusion) {

# This function removes a UWF file exclusion from a volume
# The file exclusion is removed the next time the device is restarted
# $driveLetter is the drive letter of the volume
# $exclusion is the path and filename of the file or folder exclusion

    # Get the configuration for the next session for the volume
    $nextConfig = Get-WMIObject -class UWF_Volume @CommonParams |
        where { $_.DriveLetter -eq "$driveLetter" -and $_.CurrentSession -eq $false };

    # Try to remove the exclusion
    if ($nextConfig) {
        try {
            $nextConfig.RemoveExclusion($exclusion) | Out-Null;
            Write-Host "Removed exclusion $exclusion for $driveLetter";
        } catch {
            Write-Host "Could not remove exclusion $exclusion on drive $driveLetter"
        }
    } else {
        Write-Error "Could not find drive $driveLetter";
    }
}

function Clear-FileExclusions($driveLetter) {

# This function removes all UWF file exclusions on a volume
# The file exclusions are removed the next time the device is restarted
# $driveLetter is the drive letter of the volume

    # Get the configuration for the next session for the volume
    $nextConfig = Get-WMIObject -class UWF_Volume @CommonParams |
        where { $_.DriveLetter -eq "$driveLetter" -and $_.CurrentSession -eq $false };

    # Remove all file and folder exclusions
    if ($nextConfig) {
        $nextConfig.RemoveAllExclusions() | Out-Null;
        Write-Host "Cleared all exclusions for $driveLetter";
    } else {
        Write-Error "Could not clear exclusions for drive $driveLetter";
    }
}

# Some examples of using the functions
Clear-FileExclusions "C:"
Add-FileExclusion "C:" "\Users\Public\Public Documents"
Add-FileExclusion "C:" "\myfolder\myfile.txt"
Get-FileExclusions "C:"
Remove-FileExclusion "C:" "\myfolder\myfile.txt"
Get-FileExclusions "C:"
```

Actual call patterns used, as literally shown:

- Instance selection is done **client-side** with `Get-WMIObject -class UWF_Volume | where { $_.DriveLetter -eq "$driveLetter" -and $_.CurrentSession -eq $false }` — **not** a WQL `-Filter`/`-Query`. Docs show no WQL example for `UWF_Volume`.
- Drive letter is compared as `"C:"` (letter + colon, no backslash).
- Exclusion paths are volume-relative with a leading backslash and no drive letter: `"\Users\Public\Public Documents"`, `"\myfolder\myfile.txt"`.
- `GetExclusions()` is called **on a `UWF_Volume` instance**; the result object's `.ExcludedFiles` collection is iterated and each element's `.FileName` is read.
  > `[与 brief 一致]` brief: "文件排除列表不能用 WQL 枚举 `UWF_ExcludedFile`（会返回空）——必须调 `UWF_Volume.GetExclusions()` 取嵌入对象。" The docs' only shown retrieval path is indeed `UWF_Volume.GetExclusions()`. The docs **do not** state that WQL enumeration of `UWF_ExcludedFile` returns empty — see the UWF_ExcludedFile section below. `[文档未说明]`
- Method results are piped to `Out-Null` in these samples; only `Remove-FileExclusion` wraps the call in `try/catch`.

---

## UWF_ExcludedFile

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-excludedfile>

Description (verbatim):

> Contains the files and folders that are currently in the file exclusion list for a volume protected by Unified Write Filter (UWF).

### MOF definition (verbatim)

```mof
class UWF_ExcludedFile {
    [Read] string FileName;
};
```

### Properties (verbatim table)

| Property | Data type | Qualifier | Description |
| --- | --- | --- | --- |
| FileName | string | [read] | The name of the file or folder path in the file exclusion list, including the full path relative to the volume. |

No `[key]` qualifier is printed for this class. No methods.

### Remarks (verbatim)

> UWF_ExcludedFile does not represent an actual WMI object, and you cannot use this class to get or set file exclusions.
>
> You must use the UWF_Volume.GetExclusions method to retrieve UWF_ExcludedFile objects.
>
> You can use the UWF_Volume.AddExclusion and UWF_Volume.RemoveExclusion methods to add or remove file and folder exclusions to a volume.

> `[与 brief 一致 — 官方文档确认]` brief: "文件排除列表不能用 WQL 枚举 `UWF_ExcludedFile`（会返回空）——必须调 `UWF_Volume.GetExclusions()` 取嵌入对象。"
> The docs state the stronger form: the class **"does not represent an actual WMI object"** and "you cannot use this class to get or set file exclusions." The docs do **not** describe what a WQL enumeration actually returns (empty result vs. error) — `[文档未说明]`.

### Example code on page

None.

---

## UWF_ExcludedRegistryKey

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-excludedregistrykey>

Description (verbatim):

> Contains the registry keys that are currently in the registry key exclusion list for Unified Write Filter (UWF).

### MOF definition (verbatim)

```mof
class UWF_ExcludedRegistryKey {
    [Read] string RegistryKey;
};
```

### Properties (verbatim table)

| Property | Data type | Qualifier | Description |
| --- | --- | --- | --- |
| RegistryKey | string | [read] | The full path of the registry key in the registry key exclusion list. |

No `[key]` qualifier printed. No methods.

### Remarks (verbatim)

> UWF_ExcludedRegistryKeydoes not represent an actual WMI object, and you cannot use this class to get or set registry key exclusions.
>
> You can use the UWF_RegistryFilter.GetExclusions or UWF_RegistryFilter.FindExclusion methods to retrieve UWF_ExcludedRegistryKey objects.
>
> You can use the UWF_Volume.AddExclusion and UWF_Volume.RemoveExclusion methods to add or remove registry keys to the UWF registry key exclusion list.

(the run-together "UWF_ExcludedRegistryKeydoes" is verbatim from the page)

> **⚠ Internal doc inconsistency (transcribed, not adjudicated)**: the third paragraph names **`UWF_Volume`**`.AddExclusion` / `.RemoveExclusion` for *registry* keys, while the `UWF_RegistryFilter` page documents `UWF_RegistryFilter.AddExclusion` / `.RemoveExclusion` for registry keys and the `UWF_Volume` page documents its `AddExclusion`/`RemoveExclusion` as *file* exclusions.
> `[与 brief 相关]` brief says "注册表排除同理走 `UWF_RegistryFilter` 的方法" — this agrees with the `UWF_RegistryFilter` page and disagrees with this sentence on the `UWF_ExcludedRegistryKey` page. Not adjudicated.

Also note: `UWF_ExcludedRegistryKey` is reachable via `UWF_RegistryFilter.FindExclusion` per this page's Remarks — but the `UWF_RegistryFilter` MOF declares `FindExclusion([in] string RegistryKey, [out] boolean bFound)`, which returns a boolean, not `UWF_ExcludedRegistryKey` objects. Transcribed as printed; not adjudicated.

### Example code on page

None.

---

## UWF_OverlayFile

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-overlayfile>

Description (verbatim):

> Contains a file that is currently in the overlay for a volume protected by Unified Write Filter (UWF).

> **⚠ Index-page inconsistency**: the WMI provider reference index table describes `UWF_OverlayFile` as "Displays and configures global settings for the UWF overlay. You can modify the maximum size and the type of the UWF overlay." — that text actually matches the `UWF_OverlayConfig` page. Transcribed; not adjudicated.

### MOF definition (verbatim)

```mof
class UWF_OverlayFile {
    [read] string FileName;
    [read] UInt64 FileSize;
};
```

### Properties (verbatim table)

| Property | Data type | Qualifier | Description |
| --- | --- | --- | --- |
| FileName | string | [read] | The name of the file in the file overlay. |
| FileSize | UInt64 | [read] | The size of the file in the file overlay. |

No `[key]` qualifier printed. No methods. Unit of `FileSize` is `[文档未说明]` (the page says only "The size of the file").

### Remarks (verbatim)

> You cannot use the **UWF_ OverlayFile** class directly to get overlay files. You must use the **UWF_Overlay.GetOverlayFiles** method to retrieve **UWF_ OverlayFile** objects.
>
> For more information about specific limitations and conditions when using the **GetOverlayFiles** method, see the **Remarks** section in the UWF_Overlay.GetOverlayFiles topic in the UWF WMI provider technical reference.

(the stray space in "UWF_ OverlayFile" is verbatim from the page)

### Example code on page

None.

---

## UWF_Overlay

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-overlay>

Description (verbatim):

> Contains the current size of the Unified Write Filter (UWF) overlay and manages the critical and warning thresholds for the overlay size.

### MOF definition (verbatim)

```mof
class UWF_Overlay {
    [key]  string Id;
    [read] UInt32 OverlayConsumption;
    [read] UInt32 AvailableSpace;
    [read] UInt32 CriticalOverlayThreshold;
    [read] UInt32 WarningOverlayThreshold;

    UInt32 GetOverlayFiles(
        [in] string Volume,
        [out, EmbeddedInstance("UWF_OverlayFile")] string OverlayFiles[]
    );
    UInt32 SetWarningThreshold(
        UInt32 size
    );
    UInt32 SetCriticalThreshold(
        UInt32 size
    );
};
```

### Properties (verbatim table)

| Property | Data type | Qualifiers | Description |
| --- | --- | --- | --- |
| ID | string | [key] | A unique ID. This is always set to **UWF_Overlay**. |
| OverlayConsumption | Uint32 | [read] | The current size, in megabytes, of the UWF overlay. |
| AvailableSpace | Uint32 | [read] | The amount of free space, in megabytes, available to the UWF overlay. |
| CriticalOverlayThreshold | Uint32 | [read] | The critical threshold size, in megabytes. UWF sends a critical threshold notification event when the UWF overlay size reaches or exceeds this value. |
| WarningOverlayThreshold | Uint32 | [read] | The warning threshold size, in megabytes. UWF sends a warning threshold notification event when the UWF overlay size reaches or exceeds this value. |

(the properties table spells the key property `ID` while the MOF spells it `Id` — verbatim from the page)

> `[与 brief 一致]` brief: "`UWF_Overlay` 的 OverlayConsumption/AvailableSpace/两个阈值都是 UInt32、单位 MB". The docs confirm all four are `UInt32` and in megabytes. The docs give **no** sentinel/error value semantics and no statement that any of them can be negative — `[文档未说明]`.

**Note**: `UWF_Overlay` has **no** `CurrentSession` key — it is a singleton (see Remarks). All four value properties are `[read]` only; thresholds are changed via methods.

### Methods (verbatim table)

| Methods | Description |
| --- | --- |
| UWF_Overlay.GetOverlayFiles | Returns a list of files of a volume that were cached in the UWF overlay. |
| UWF_Overlay.SetWarningThreshold | Sets the warning threshold for monitoring the size of the UWF overlay. |
| UWF_Overlay.SetCriticalThreshold | Sets the critical warning threshold for monitoring the size of the UWF overlay. |

### Class Remarks (verbatim)

> Only one **UFW_Overlay** instance exists for a system protected with UWF.

(the typo "UFW_Overlay" is verbatim from the page)

---

#### UWF_Overlay.GetOverlayFiles

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-overlaygetoverlayfiles>

Description (verbatim): "Returns a list of files of a volume that were cached in the Unified Write Filter (UWF) overlay."

Signature (verbatim):

```mof
UInt32 GetOverlayFiles( [in] string Volume, [out, EmbeddedInstance("UWF_OverlayFile")] string OverlayFiles[] );
```

Parameters (verbatim):

> **Volume**
> A string that specifies the drive letter or volume name.
>
> **OverlayFiles**
> An array of **UWF_OverlayFiles** objects embedded as strings.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> You must use an administrator account to access this method.
>
> The **GetOverlayFiles** method is intended to be used as a diagnostic tool.
>
> Do not base decisions about what to commit based on this method’s output.
>
> You should be aware of the following limitations:
>
> * This method is only supported on the NTFS file system.
> * This method requires a significant amount of free system memory to succeed (in a linear relationship to overlay usage). The method call fails when there is insufficient memory available to complete the call.
> * This method requires significant time to complete (in an exponential relationship to overlay usage).
> * This method may show files that are affected by seemingly unrelated operations to both registry and file exclusions and commits.
>
> You should also be aware of the following items when you use the **GetOverlayFiles** method:
>
> * Files that were committed with the `uwfmgr.exe file commit` command are also contained in the overlay files list.
> * Excluded files may be contained in the overlay files list.
> * Files that are smaller than the cluster size (for example, 4 KB in most cases) will not be listed even if they are cached in overlay.
> * Changes and deletions in excluded directories, excluded files, or excluded registry items add to overlay usage.
> * File and registry commits add to overlay usage.

Behavior when the overlay contains no files (null vs. empty array): `[文档未说明]`

---

#### UWF_Overlay.SetWarningThreshold

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-overlaysetwarningthreshold>

Description (verbatim): "Sets the warning threshold for monitoring the size of the Unified Write Filter (UWF) overlay."

Signature (verbatim):

```mof
UInt32 SetWarningThreshold( UInt32 size );
```

Parameters (verbatim):

> **size**
> An integer that represents the size, in megabytes, of the warning threshold level for the overlay. If *size* is set to 0 (zero), UWF does not raise warning threshold events.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> When the size of the overlay reaches or exceeds the *size* threshold value, UWF writes the following notification event to the event log.
>
> | Message ID | Event code | Message text |
> | --- | --- | --- |
> | UWF_OVERLAY_REACHED_WARNING_LEVEL | 0x80010001L | The UWF overlay size has reached WARNING level. |
>
> The warning threshold must be lower than the critical threshold.

Whether the change is immediate or next-session: `[文档未说明]`

---

#### UWF_Overlay.SetCriticalThreshold

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-overlaysetcriticalthreshold>

Description (verbatim): "Sets the critical threshold for monitoring the size of the Unified Write Filter (UWF) overlay."

Signature (verbatim):

```powershell
UInt32 SetCriticalThreshold(
    UInt32 size
);
```

Parameters (verbatim):

> **size**
>
> An integer that represents the size, in megabytes, of the critical threshold level for the overlay. If *size* is 0 (zero), UWF does not raise critical threshold events.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> When the size of the overlay reaches or exceeds the *size* threshold value, UWF writes the following notification event to the event log.
>
> | Message ID | Event code | Message text |
> | --- | --- | --- |
> | UWF_OVERLAY_REACHED_CRITICAL_LEVEL | 0x80010002L | The UWF overlay size has reached CRITICAL level. |
>
> The critical threshold must be higher than the warning threshold.

---

### UWF_Overlay — example code (verbatim from the page)

Page prose (verbatim): "The following example demonstrates how to use the UWF overlay by using the WMI provider in a PowerShell script."

```powershell
$COMPUTER = "localhost"
$NAMESPACE = "root\standardcimv2\embedded"

# Function to set the Unified Write Filter overlay warning threshold

function Set-OverlayWarningThreshold($ThresholdSize) {

# Retrieve the overlay WMI object

    $OverlayInstance = Get-WMIObject -namespace $NAMESPACE -class UWF_Overlay;

    if(!$OverlayInstance) {
        "Unable to get handle to an instance of the UWF_Overlay class"
        return;
    }

# Call the instance method to set the warning threshold value

    $retval = $OverlayInstance.SetWarningThreshold($ThresholdSize);

# Check the return value to verify that setting the warning threshold is successful

    if ($retval.ReturnValue -eq 0) {
        "Overlay warning threshold has been set to " + $ThresholdSize + " MB"
    } else {
        "Unknown Error: " + "{0:x0}" -f $retval.ReturnValue
    }
}

# Function to set the Unified Write Filter overlay critical threshold

function Set-OverlayCriticalThreshold($ThresholdSize) {

# Retrieve the overlay WMI object

    $OverlayInstance = Get-WMIObject -namespace $NAMESPACE -class UWF_Overlay;

    if(!$OverlayInstance) {
        "Unable to get handle to an instance of the UWF_Overlay class"
        return;
    }

# Call the instance method to set the warning threshold value

    $retval = $OverlayInstance.SetCriticalThreshold($ThresholdSize);

# Check the return value to verify that setting the critical threshold is successful

    if ($retval.ReturnValue -eq 0) {
        "Overlay critical threshold has been set to " + $ThresholdSize + " MB"
    } else {
        "Unknown Error: " + "{0:x0}" -f $retval.ReturnValue
    }
}

# Function to print the current overlay information

function Get-OverlayInformation() {

# Retrieve the Overlay WMI object

    $OverlayInstance = Get-WMIObject -namespace $NAMESPACE -class UWF_Overlay;

    if(!$OverlayInstance) {
        "Unable to get handle to an instance of the UWF_Overlay class"
        return;
    }

# Display the current values of the overlay properties

    "`nOverlay Consumption: " + $OverlayInstance.OverlayConsumption
    "Available Space: " + $OverlayInstance.AvailableSpace
    "Critical Overlay Threshold: " + $OverlayInstance.CriticalOverlayThreshold
    "Warning Overlay Threshold: " + $OverlayInstance.WarningOverlayThreshold
}

# Examples of using these functions

"`nSetting the warning threshold to 768 MB."
Set-OverlayWarningThreshold( 768 )

"`nSetting the critical threshold to 896 MB."
Set-OverlayCriticalThreshold( 896 )

"`nDisplaying the current state of the overlay."
Get-OverlayInformation
```

Actual call patterns used, as literally shown:

- `Get-WMIObject -namespace "root\standardcimv2\embedded" -class UWF_Overlay` — no filter/WQL, singleton.
- Thresholds are set through instance methods, then read back through the read-only properties.
- Return checked via `$retval.ReturnValue -eq 0`.

---

## UWF_OverlayConfig

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-overlayconfig>

Description (verbatim):

> Displays and configures global settings for the Unified Write Filter (UWF) overlay. You can modify the maximum size and the type of the UWF overlay.

(the index page instead labels this class "Manages the configuration of the UWF overlay." — see the UWF_OverlayFile note above)

### MOF definition (verbatim)

```mof
class UWF_OverlayConfig{
    [key, Read] boolean CurrentSession;
    [read] UInt32 Type;
    [read] SInt32 MaximumSize;

    UInt32 SetType(
        UInt32 type
    );
    UInt32 SetMaximumSize(
        UInt32 size
    );
};
```

### Properties (verbatim table)

| Property | Data type | Qualifiers | Description |
| --- | --- | --- | --- |
| CurrentSession | Boolean | [key, read] | Indicates which session the object contains settings for. - **True** for the current session - **False** for the next session that begins after a restart. |
| Type | UInt32 | [read] | Indicates the type of overlay. - **0** for a RAM-based overlay - **1** for a disk-based overlay. |
| MaximumSize | SInt32 | [read] | Indicates the maximum cache size, in megabytes, of the overlay. |

> **⚠ Type note for `IUwfProvider`**: `MaximumSize` is documented as **`SInt32`** (signed) on the read side, while the setter `SetMaximumSize` takes **`UInt32 size`**. Both are verbatim from the docs. The docs state no meaning for a negative `MaximumSize` — `[文档未说明]`.
> `[与 brief 相关]` brief 只把 UInt32/MB 归给 `UWF_Overlay` 的四个属性，未提及 `UWF_OverlayConfig.MaximumSize`；此处文档明写 SInt32。记录，不裁决。

### Methods (verbatim table)

| Method | Description |
| --- | --- |
| UWF_OverlayConfig.SetMaximumSize | Sets the maximum cache size, in megabytes, of the overlay. |
| UWF_OverlayConfig.SetType | Sets the type of the UWF overlay to either RAM-based or disk-based. |

### Class Remarks (verbatim)

> Changes to the overlay configuration take effect on the next restart in which UWF is enabled.
>
> Before you can change the **Type** or **MaximumSize** properties, UWF must be disabled in the current session.

---

#### UWF_OverlayConfig.SetMaximumSize

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-overlayconfigsetmaximumsize>

Description (verbatim): "Sets the maximum cache size of the Unified Write Filter (UWF) overlay."

Signature (verbatim):

```mof
UInt32 SetMaximumSize( UInt32 size );
```

Parameters (verbatim):

> **size**
> An integer that represents the maximum cache size, in megabytes, of the overlay.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> When the size of the overlay reaches the *size* value, UWF returns an error for any attempt to write to a protected volume.
>
> If the overlay type is disk-based, your device must meet the following requirements to change the maximum size of the overlay.
>
> * UWF must be disabled in the current session.
> * The *size* value must be at least 1024.
> * The system volume on your device must have available free space greater than the new maximum size value.
>
> If the overlay type is RAM-based, your device must meet the following requirement to change the maximum size of the overlay.
>
> * UWF must be disabled in the current session.

---

#### UWF_OverlayConfig.SetType

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-overlayconfigsettype>

Description (verbatim): "Sets the type of the Unified Write Filter (UWF) overlay to either RAM-based or disk-based."

Signature (verbatim):

```mof
UInt32 SetType( UInt32 type );
```

Parameters (verbatim):

> **type**
> The type of overlay. Set to **0** for a RAM-based overlay; set to **1** for a disk-based overlay.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> Changes to the overlay type take effect during the next device restart in which UWF is enabled.
>
> When you change the overlay type from RAM-based to disk-based, UWF creates a file on the system volume. The file has a size equal to the **MaximumSize** property of UWF_OverlayConfig.
>
> Before you can change the overlay type to disk-based, your device must meet the following requirements.
>
> * UWF must be disabled in the current session.
> * The system volume on your device must have available free space greater than the maximum size of the overlay.
> * The maximum size of the overlay must be at least 1024 MB.
>
> Before you can change the overlay type to RAM-based, your device must meet the following requirements.
>
> * UWF must be disabled in the current session.

Also linked on this page: "Overlay for Unified Write Filter (UWF)" → <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwfoverlay>

---

### UWF_OverlayConfig — example code (verbatim from the page)

Page prose (verbatim):

> The following example demonstrates how to change the maximum size or the storage type of the overlay in UWF by using the Windows Management Instrumentation (WMI) provider in a PowerShell script.
>
> The PowerShell script creates two functions to modify the overlay configuration. It then demonstrates how to use the functions. The first function, **Set-OverlaySize**, sets the maximum size of the overlay. The second function, **Set-OverlayType**, sets the type of the overlay to RAM-based or disk-based.

```powershell
$COMPUTER = "localhost"
$NAMESPACE = "root\standardcimv2\embedded"

# Define common parameters

$CommonParams = @{"namespace"=$NAMESPACE; "computer"=$COMPUTER}

function Set-OverlaySize([UInt32] $size) {

# This function sets the size of the overlay to which file and registry changes are redirected
# Changes take effect after the next restart

# $size is the maximum size in MB of the overlay

# Make sure that UWF is currently disabled

    $UWFFilter = Get-WmiObject -class UWF_Filter @commonParams

    if ($UWFFilter.CurrentEnabled -eq $false) {

# Get the configuration for the next session after a restart

        $nextConfig = Get-WMIObject -class UWF_OverlayConfig -Filter "CurrentSession = false" @CommonParams;

        if ($nextConfig) {

# Set the maximum size of the overlay

        $nextConfig.SetMaximumSize($size);
            write-host "Set overlay max size to $size MB."
        }
    } else {
        write-host "UWF must be disabled in the current session before you can change the overlay size."
    }
}

function Set-OverlayType([UInt32] $overlayType) {

# This function sets the type of the overlay to which file and registry changes are redirected
# Changes take effect after the next restart

# $overlayType is the type of storage that UWF uses to maintain the overlay. 0 = RAM-based; 1 = disk-based.

    $overlayTypeText = @("RAM-based", "disk-based")

# Make sure that the overlay type is a valid value

    if ($overlayType -eq 0 -or $overlayType -eq 1) {

# Make sure that UWF is currently disabled

        $UWFFilter = Get-WmiObject -class UWF_Filter @commonParams

        if ($UWFFilter.CurrentEnabled -eq $false) {

# Get the configuration for the next session after a restart

            $nextConfig = Get-WMIObject -class UWF_OverlayConfig -Filter "CurrentSession = false" @CommonParams;

            if ($nextConfig) {

# Set the type of the overlay

        $nextConfig.SetType($overlayType);
                write-host "Set overlay type to $overlayTypeText[$overlayType]."
            }
        } else {
            write-host "UWF must be disabled in the current session before you can change the overlay type."
        }
    } else {
        write-host "Invalid value for overlay type.  Valid values are 0 (RAM-based) or 1 (disk-based)."
    }
}

# The following sample commands demonstrate how to use the functions to change the overlay configuration

$RAMMode = 0
$DiskMode = 1

Set-OverlaySize 2048

Set-OverlayType $DiskMode
```

Actual call patterns used, as literally shown:

- **This is the only sample in the whole reference that uses a WQL-style filter**: `Get-WMIObject -class UWF_OverlayConfig -Filter "CurrentSession = false"` — note the filter string uses bare `false` (WQL boolean literal), not `$false` and not `FALSE` quoted.
- The precondition "UWF must be disabled in the current session" is checked client-side via `$UWFFilter.CurrentEnabled -eq $false` **before** calling `SetMaximumSize`/`SetType`.
- Return values are **not** checked in this sample (`$nextConfig.SetMaximumSize($size);` with no `$retval`).

---

## UWF_Servicing

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-servicing>

Description (verbatim):

> This class contains properties and methods that enable you to query and control Unified Write Filter (UWF) servicing mode.

### MOF definition (verbatim)

```mof
class UWF_Servicing {
    [key, read] boolean CurrentSession;
    [read] boolean ServicingEnabled;

    UInt32 Enable();
    UInt32 Disable();
    UInt32 UpdateWindows(
        [out] UInt32 UpdateStatus
    );
};
```

### Properties (verbatim table — column header "Description&" is verbatim)

| Property | Data type | Qualifiers | Description& |
| --- | --- | --- | --- |
| CurrentSession | Boolean | [key, read] | Indicates when to enable servicing. - **True** if servicing is enabled in the current session - **False** if servicing will be enabled in the session that follows a restart. |
| ServiceEnabled | Boolean | [read] | Indicates if the system is in servicing mode in the current session, or will be in servicing mode in the next session that follows a restart. - **True** if servicing is enabled - otherwise, **False**. |

> **⚠ Internal doc inconsistency (transcribed, not adjudicated)**: the MOF declares the property as **`ServicingEnabled`**; the properties table names it **`ServiceEnabled`**. The two do not agree. The correct on-wire property name is `[文档未说明]` — must be confirmed on hardware/VM.

### Methods (verbatim table)

| Method | Description |
| --- | --- |
| UWF_Servicing.Disable | Disables Unified Write Filter (UWF) servicing mode. The system leaves servicing mode in the next session that follows a restart. |
| UWF_Servicing.Enable | Enables Unified Write Filter (UWF) servicing mode. The system enters servicing mode in the next session that follows a restart. |
| UWF_Servicing.UpdateWindows | Calls Windows Update to download and install critical and security updates for your device running Windows 10 Enterprise. |

### Class Remarks (verbatim)

> This class only has two instances, one for the current session, and another for the next session that follows a restart.

---

#### UWF_Servicing.Enable

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-servicingenable>

Description (verbatim): "Enables Unified Write Filter (UWF) servicing mode."

Signature (verbatim):

```powershell
UInt32 Enable();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> When this method is called, the system will enter servicing mode in the next session after a restart.

---

#### UWF_Servicing.Disable

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-servicingdisable>

Description (verbatim): "Disables Unified Write Filter (UWF) servicing mode."

Signature (verbatim):

```powershell
UInt32 Disable();
```

Parameters (verbatim): **None.**

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> When this method is called, the system will leave servicing mode in the next session after a restart.

---

#### UWF_Servicing.UpdateWindows

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-servicingupdatewindows>

Description (verbatim): "Calls Windows Update to download and install critical and security updates for your device running Windows 10 Enterprise."

Signature (verbatim):

```powershell
UInt32 UpdateWindows(
    [out] UInt32 UpdateStatus
);
```

Parameters (verbatim):

> **UpdateStatus**
>
> [out] An integer that contains the status of the Windows Update operation, according to the following table:

| UpdateStatus | Description |
| --- | --- |
| 0 | Success. |
| 3010 | Restart required. |
| Any other value. | Generic error. |

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> This method is meant to be used as part of a servicing script. For more information, see Service UWF-protected devices.
>
> This method does not disable or enable Unified Write Filter (UWF). If you call this method while UWF is enabled, updates may be lost when the device restarts.

Linked article: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/service-uwf-protected-devices>

---

### UWF_Servicing — example code (verbatim from the page)

Page prose (verbatim): "The following example shows how to enable and disable UWF servicing mode on a device by using the Windows Management Instrumentation (WMI) provider in a PowerShell script."

```powershell
$COMPUTER = "localhost"
$NAMESPACE = "root\standardcimv2\embedded"

# Define common parameters

$CommonParams = @{"namespace"=$NAMESPACE; "computer"=$COMPUTER}

# Enable UWF servicing

$nextSession = Get-WmiObject -class UWF_Servicing @CommonParams | where {
    $_.CurrentSession -eq $false
}

if ($nextSession) {

    $nextSession.Enable() | Out-Null;
    Write-Host "This device is enabled for servicing mode after the next restart."
}

# Disable UWF servicing

$nextSession = Get-WmiObject -class UWF_Servicing @CommonParams | where {
    $_.CurrentSession -eq $false
}

if ($nextSession) {

    $nextSession.Disable() | Out-Null;
    Write-Host "Servicing mode is now disabled for this device."
}
```

Actual call patterns used, as literally shown:

- Instance selection is client-side `where { $_.CurrentSession -eq $false }`; `Enable()`/`Disable()` are called on the **next-session** instance.
- Return values are discarded (`| Out-Null`).
- The sample never reads `ServicingEnabled` / `ServiceEnabled`.

---

## UWF_RegistryFilter

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-registryfilter>

Description (verbatim):

> Adds or removes registry exclusions from Unified Write Filter (UWF) filtering, and also commits registry changes.

### MOF definition (verbatim from the page's Syntax block)

```mof
class UWF_RegistryFilter{
    [key, Read] boolean CurrentSession;
    [Read, Write] boolean PersistDomainSecretKey;
    [Read, Write] boolean PersistTSCAL;

    UInt32 AddExclusion( string RegistryKey );
    UInt32 RemoveExclusion( string RegistryKey );
    UInt32 FindExclusion( [in] string RegistryKey, [out] boolean bFound );
    UInt32 GetExclusions( [out, EmbeddedInstance("UWF_ExcludedRegistryKey")] string ExcludedKeys[] );
    UInt32 CommitRegistry( [in] string RegistryKey, [in] string ValueName );
    UInt32 CommitRegistryDeletion( string Registrykey, string ValueName );
};
```

*(Extractor returned this on one line; tokens verbatim, line breaks reinstated. Note the lower-case `k` in `CommitRegistryDeletion( string Registrykey, ...)` is verbatim.)*

### Properties (verbatim table)

| Property | Data type | Qualifiers | Description |
| --- | --- | --- | --- |
| CurrentSession | Boolean | [key, read] | Indicates which session the object contains settings for.   - **True** if settings are for the current session  - **False** if settings are for the next session that follows a restart. |
| PersistDomainSecretKey | Boolean | [read, write] | Indicates if the domain secret registry key is in the registry exclusion list. If the registry key is not in the exclusion list, changes are not persisted after a restart. - **True** to include in the exclusion list  - Otherwise **False**. |
| PersistTSCAL | Boolean | [read, write] | Indicates if the Terminal Server Client Access License (TSCAL) registry key is in the UWF registry exclusion list. If the registry key is not in the exclusion list, changes are not persisted after a restart.  - **True** to include in the exclusion list - Otherwise, set to **False** |

Key: `CurrentSession` only. Two instances (current / next) — inferred from the key and the sample; the page does not state an instance count explicitly (`UWF_Servicing` does state it, `UWF_RegistryFilter` does not) — `[文档未说明]`.

### Methods (verbatim table)

| Method | Description |
| --- | --- |
| UWF_RegistryFilter.AddExclusion | Adds a registry key to the registry exclusion list for UWF. |
| UWF_RegistryFilter.CommitRegistry | Commits changes to the specified registry key and value. |
| UWF_RegistryFilter.CommitRegistryDeletion | Deletes the specified registry key or registry value and commits the deletion. |
| UWF_RegistryFilter.FindExclusion | Determines whether a specific registry key is excluded from being filtered by UWF. |
| UWF_RegistryFilter.GetExclusions | Retrieves all registry key exclusions from a system that is protected by UWF |
| UWF_RegistryFilter.RemoveExclusion | Removes a registry key from the registry exclusion list for Unified Write Filter (UWF). |

**There is no `RemoveAllExclusions` method on `UWF_RegistryFilter`** — the docs' own sample clears registry exclusions by enumerating `GetExclusions()` and calling `RemoveExclusion` per key.

### Class Remarks (verbatim)

> Additions or removals of registry exclusions, including changes to the values of **PersistDomainSecretKey** and **PersistTSCAL**, take effect after the next restart in which UWF is enabled.
>
> You can only add registry keys in the HKLM registry root to the UWF registry exclusion list.
>
> You can also use **UWF_RegistryFilter** to exclude the domain secret registry key and the TSCAL registry key from UWF filtering.

---

#### UWF_RegistryFilter.AddExclusion

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-registryfilteraddexclusion>

Description (verbatim): "Adds a registry key to the registry exclusion list for Unified Write Filter (UWF)."

Important boxes at the top of the page (verbatim):

> **Important**
>
> Only registry subkeys under the following registry keys can be added to the exclusion list.
>
> * HKEY_LOCAL_MACHINE\BCD00000000
> * HKEY_LOCAL_MACHINE\SYSTEM
> * HKEY_LOCAL_MACHINE\SOFTWARE
> * HKEY_LOCAL_MACHINE\SAM
> * HKEY_LOCAL_MACHINE\SECURITY
> * HKEY_LOCAL_MACHINE\COMPONENTS
>
> **Important**
>
> Excluding a registry key from filtering also excludes all subkeys from filtering.

Signature (verbatim):

```powershell
UInt32 AddExclusion(
    string RegistryKey
);
```

Parameters (verbatim):

> **RegistryKey**
>
> A string that contains the full path of the registry key.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> You must restart the device before the registry key is excluded from UWF filtering.

---

#### UWF_RegistryFilter.RemoveExclusion

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-registryfilterremoveexclusion>

Description (verbatim): "Removes a registry key from the registry exclusion list for Unified Write Filter (UWF)."

Signature (verbatim):

```powershell
UInt32 RemoveExclusion(
    string RegistryKey
);
```

Parameters (verbatim):

> **RegistryKey**
>
> A string that contains the full path of the registry key.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> You must restart the device before the registry key is excluded from UWF filtering.

(the Remarks text on the *Remove* page says "excluded", identical to the Add page — verbatim from the page, apparently a copy/paste error in the docs. Transcribed; not adjudicated.)

---

#### UWF_RegistryFilter.FindExclusion

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-registryfilterfindexclusion>

Description (verbatim): "Checks if a specific registry key is excluded from being filtered by Unified Write Filter (UWF)."

Signature (verbatim):

```powershell
UInt32 FindExclusion(
    [in] string RegistryKey,
    [out] boolean bFound
);
```

Parameters (verbatim):

> **RegistryKey**
>
> [in] A string that contains the full path of the registry key.
>
> **bFound**
>
> [out] Indicates if the *RegistryKey* is in the exclusion list of registry keys.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks: **the page has no Remarks section.** `[文档未说明]` — in particular, whether sub-key matching behaves like `UWF_Volume.FindExclusion` (explicit-only) is not stated here.

---

#### UWF_RegistryFilter.GetExclusions

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-registryfiltergetexclusions>

Description (verbatim): "Retrieves all registry key exclusions from a device that is protected by Unified Write Filter (UWF)."

Signature (verbatim):

```mof
UInt32 GetExclusions( [out, EmbeddedInstance("UWF_ExcludedRegistryKey")] string ExcludedKeys[] );
```

Parameters (verbatim):

> **ExcludedKeys**
> [out] An array of UWF_ExcludedRegistryKey objects that represent the registry keys excluded from UWF filtering.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> If this method does not find any registry keys in the registry key exclusion list, it sets the *ExcludedKeys* parameter to null.

> **⚠ Design note (docs-literal)**: same null-on-empty semantics as `UWF_Volume.GetExclusions`. The out-parameter is named **`ExcludedKeys`** here (vs. `ExcludedFiles` on the volume class).

---

#### UWF_RegistryFilter.CommitRegistry

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-registryfiltercommitregistry>

Description (verbatim): "Commits changes to the specified registry key and value."

Signature (verbatim):

```powershell
UInt32 CommitRegistry(
    [in] string RegistryKey,
    [in] string ValueName
);
```

Parameters (verbatim):

> **RegistryKey**
>
> A string that contains the full path of the registry key to be committed.
>
> **ValueName**
>
> A string that contains the name of the value to be committed.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> This method will commit only the value specified by *ValueName* under *RegistryKey* if *ValueName* is specified.
>
> You must use an administrator account to change any properties or call any methods that change the configuration settings.

What happens when `ValueName` is empty/omitted (does it commit the whole key?): the Remarks only say "if *ValueName* is specified" — the complementary case is `[文档未说明]` (contrast with `CommitRegistryDeletion`, which spells it out).

---

#### UWF_RegistryFilter.CommitRegistryDeletion

SOURCE: <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-registryfiltercommitregistrydeletion>

Description (verbatim): "Deletes the specified registry key or registry value and commits the deletion."

Signature (verbatim):

```powershell
UInt32 CommitRegistryDeletion(
    string Registrykey,
    string ValueName
);
```

(parameter spelled `Registrykey` in the code block but `RegistryKey` in the Parameters prose — verbatim from the page)

Parameters (verbatim):

> **RegistryKey**
>
> A string that contains the full path of the registry key that contains the value to be deleted. If *ValueName* is empty, the entire registry key is deleted.
>
> **ValueName**
>
> A string that contains the name of the value to be deleted.

Return Value (verbatim):

> Returns an HRESULT value that indicates WMI status or a WMI error.

Remarks (verbatim):

> If *ValueName* is specified, this method will delete only the value specified by *ValueName* that is contained by *RegistryKey*. If *ValueName* is empty, the entire *RegistryKey* and all its sub keys are deleted.
>
> This method deletes the registry key or registry value from both the overlay and the persistent storage.
>
> You must use an administrator account to change any properties or call any methods that change the configuration settings.

---

### UWF_RegistryFilter — example code (verbatim from the page)

Page prose (verbatim):

> The following example demonstrates how to manage UWF registry exclusions by using the Windows Management Instrumentation (WMI) provider in a PowerShell script.
>
> The PowerShell script creates four functions, and then demonstrates how to use them.
>
> The first function, **Get-RegistryExclusions**, displays a list of UWF registry exclusions for both the current session and the next session that follows a restart.
>
> The second function, **Add-RegistryExclusion**, adds a registry entry to the UWF registry exclusion list after you restart the device.
>
> The third function, **Remove-RegistryExclusion**, removes a registry entry from the UWF exclusion list after you restart the device.
>
> The fourth function, **Clear-RegistryExclusions**, removes all UWF registry exclusions. You must restart the device before UWF stops filtering the exclusions.

```powershell
$COMPUTER = "EMBEDDEDDEVICE"
$NAMESPACE = "root\standardcimv2\embedded"

# Define common parameters
$CommonParams = @{"namespace"=$NAMESPACE; "computer"=$COMPUTER}

function Get-RegistryExclusions() {

# This function lists the UWF registry exclusions, both
# for the current session as well as the next session after a restart.

    # Get the UWF_RegistryFilter configuration for the current session
    $currentConfig = Get-WMIObject -class UWF_RegistryFilter @CommonParams |
        where { $_.CurrentSession -eq $true };

    # Get the UWF_RegistryFilter configuration for the next session after a restart
    $nextConfig = Get-WMIObject -class UWF_RegistryFilter @CommonParams |
        where { $_.CurrentSession -eq $false };

    # Display registry exclusions for the current session
    if ($currentConfig) {
        Write-Host ""
        Write-Host "The following registry entries are currently excluded from UWF filtering:";
        $currentExcludedList = $currentConfig.GetExclusions()
        if ($currentExcludedList.ExcludedKeys) {
            foreach ($registryExclusion in $currentExcludedList.ExcludedKeys) {
                Write-Host "  " $registryExclusion.RegistryKey
            }
        } else {
            Write-Host "  None"
        }
    } else {
        Write-Error "Could not retrieve UWF_RegistryFilter.";
    }

    # Display registry exclusions for the next session after a restart
    if ($nextConfig) {
        Write-Host ""
        Write-Host "The following registry entries will be excluded from UWF filtering after the next restart:";
        $nextExcludedList = $nextConfig.GetExclusions()
        if ($nextExcludedList.ExcludedKeys) {
            foreach ($registryExclusion in $nextExcludedList.ExcludedKeys) {
                Write-Host "  " $registryExclusion.RegistryKey
            }
        } else {
            Write-Host "  None"
        }
        Write-Host ""
    }
}

function Add-RegistryExclusion($exclusion) {

# This function adds a new UWF registry exclusion.
# The new registry exclusion takes effect the next time the device is restarted and UWF is enabled.
# $exclusion is the path of the registry exclusion

    # Get the UWF_RegistryFilter configuration for the next session after a restart
    $nextConfig = Get-WMIObject -class UWF_RegistryFilter @CommonParams |
        where { $_.CurrentSession -eq $false };

    # Add the exclusion
    if ($nextConfig) {
        $nextConfig.AddExclusion($exclusion) | Out-Null;
        Write-Host "Added exclusion $exclusion.";
    } else {
        Write-Error "Could not retrieve UWF_RegistryFilter";
    }
}

function Remove-RegistryExclusion($exclusion) {

# This function removes a UWF registry exclusion.
# The registry exclusion is removed the next time the device is restarted
# $exclusion is the path of the registry exclusion

    # Get the UWF_RegistryFilter configuration for the next session after a restart
    $nextConfig = Get-WMIObject -class UWF_RegistryFilter @CommonParams |
        where { $_.CurrentSession -eq $false };

    # Try to remove the exclusion
    if ($nextConfig) {
        try {
            $nextConfig.RemoveExclusion($exclusion) | Out-Null;
            Write-Host "Removed exclusion $exclusion.";
        } catch {
            Write-Host "Could not remove exclusion $exclusion."
        }
    } else {
        Write-Error "Could not retrieve UWF_RegistryFilter";
    }
}

function Clear-RegistryExclusions() {

# This function removes all UWF registry exclusions
# The registry exclusions are removed the next time the device is restarted

    # Get the configuration for the next session
    $nextConfig = Get-WMIObject -class UWF_RegistryFilter @CommonParams |
        where { $_.CurrentSession -eq $false };

    # Remove all registry exclusions
    if ($nextConfig) {
        Write-Host "Removing all registry exclusions:";
        $nextExcludedList = $nextConfig.GetExclusions()
        if ($nextExcludedList) {
            foreach ($registryExclusion in $nextExcludedList.ExcludedKeys) {
                Write-Host "Removing:" $registryExclusion.RegistryKey
                $nextConfig.RemoveExclusion($registryExclusion.RegistryKey) | Out-Null
            }
        } else {
            Write-Host "No registry exclusions to remove."
        }
        Write-Host ""
    }
}

# Some examples of using the functions
Clear-RegistryExclusions
Get-RegistryExclusions
Add-RegistryExclusion "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer"
Add-RegistryExclusion "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\DateTime\Servers\(Default)"
Get-RegistryExclusions
Remove-RegistryExclusion "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer"
Get-RegistryExclusions
Clear-RegistryExclusions
```

Actual call patterns used, as literally shown:

- `$COMPUTER` in this sample is `"EMBEDDEDDEVICE"` (a remote machine name), unlike the other samples that use `"localhost"` — verbatim.
- Instance selection is client-side `where { $_.CurrentSession -eq $false }` (no WQL).
- `GetExclusions()` is called on a `UWF_RegistryFilter` instance; the result's `.ExcludedKeys` collection is iterated and each element's `.RegistryKey` is read.
- Registry key strings are passed in **`HKEY_LOCAL_MACHINE\...` long form**, not `HKLM:\...`.
- Note the sample's inconsistent null-guard: `Get-RegistryExclusions` tests `if ($currentExcludedList.ExcludedKeys)` while `Clear-RegistryExclusions` tests `if ($nextExcludedList)` — verbatim from the page.
- One sample exclusion path ends in `\(Default)`, i.e. a value name appended to a key path — verbatim.

---

## Missing / not found

| Requested class | Status | Evidence |
| --- | --- | --- |
| `UWF_ServicingHelper` | **Not documented.** No such page exists. | (1) The en-US index page <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-wmi-provider-reference> lists exactly 9 classes and does not include it. (2) The mirror index <https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/customize/uwf-wmi-provider-reference> lists the same 9 classes. (3) Direct fetch of <https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/uwf-servicinghelper> returns **HTTP 404 — "Page not found"** (retrieved 2026-08-17). (4) Web search for `"UWF_ServicingHelper"` returns no Microsoft page for the name. |

Whether a `UWF_ServicingHelper` class exists in the actual `root\standardcimv2\embedded` namespace on a device is `[文档未说明]` — must be confirmed by enumerating classes on the test VM.

**Pages that failed to retrieve**: none. All 9 class pages and all 29 method pages listed on those class pages were retrieved successfully (5 `UWF_Filter` + 10 `UWF_Volume` + 3 `UWF_Overlay` + 2 `UWF_OverlayConfig` + 3 `UWF_Servicing` + 6 `UWF_RegistryFilter`).

### Complete list of pages transcribed

Class pages (9):
`uwf-filter`, `uwf-volume`, `uwf-excludedfile`, `uwf-excludedregistrykey`, `uwf-registryfilter`, `uwf-overlay`, `uwf-overlayconfig`, `uwf-overlayfile`, `uwf-servicing`
— all under `https://learn.microsoft.com/en-us/windows/configuration/unified-write-filter/`

Method pages (29):
`uwf-filterenable`, `uwf-filterdisable`, `uwf-filterresetsettings`, `uwf-filtershutdownsystem`, `uwf-filterrestartsystem`,
`uwf-volumeaddexclusion`, `uwf-volumeremoveexclusion`, `uwf-volumeremoveallexclusions`, `uwf-volumefindexclusion`, `uwf-volumegetexclusions`, `uwf-volumecommitfile`, `uwf-volumecommitfiledeletion`, `uwf-volumeprotect`, `uwf-volumeunprotect`, `uwf-volumesetbindbydriveletter`,
`uwf-overlaygetoverlayfiles`, `uwf-overlaysetwarningthreshold`, `uwf-overlaysetcriticalthreshold`,
`uwf-overlayconfigsetmaximumsize`, `uwf-overlayconfigsettype`,
`uwf-servicingenable`, `uwf-servicingdisable`, `uwf-servicingupdatewindows`,
`uwf-registryfilteraddexclusion`, `uwf-registryfilterremoveexclusion`, `uwf-registryfilterfindexclusion`, `uwf-registryfiltergetexclusions`, `uwf-registryfiltercommitregistry`, `uwf-registryfiltercommitregistrydeletion`

Related pages referenced but **not** transcribed (out of scope of "WMI provider reference"): `uwfmgrexe`, `uwfoverlay`, `service-uwf-protected-devices`.

---

## Cross-cutting facts pulled from the pages (each with its source)

Aggregation only — every row is a direct restatement of text quoted above.

### Instance model

| Class | Key(s) | Instance count per the docs |
| --- | --- | --- |
| `UWF_Filter` | `Id` (always `"UWF_Filter"`) | Singleton (implied by fixed key; count not stated) |
| `UWF_Overlay` | `Id` (always `"UWF_Overlay"`) | "Only one **UFW_Overlay** instance exists for a system protected with UWF." (uwf-overlay) |
| `UWF_Volume` | `CurrentSession`, `DriveLetter`, `VolumeName` | "Each volume has two entries in UWF_Volume, one for the current session and one for the next session after a restart" (uwf-volume sample comment) |
| `UWF_OverlayConfig` | `CurrentSession` | Not stated; sample retrieves `CurrentSession = false` (uwf-overlayconfig) |
| `UWF_RegistryFilter` | `CurrentSession` | Not stated; sample retrieves both `$true` and `$false` (uwf-registryfilter) |
| `UWF_Servicing` | `CurrentSession` | "This class only has two instances, one for the current session, and another for the next session that follows a restart." (uwf-servicing) |
| `UWF_ExcludedFile` | none printed | "does not represent an actual WMI object" (uwf-excludedfile) |
| `UWF_ExcludedRegistryKey` | none printed | "does not represent an actual WMI object" (uwf-excludedregistrykey) |
| `UWF_OverlayFile` | none printed | "You cannot use the UWF_ OverlayFile class directly to get overlay files." (uwf-overlayfile) |

### Statements about "takes effect after restart"

- `UWF_Filter.Enable` — "You must restart your device after you enable or disable UWF before the change takes effect."
- `UWF_Volume.Protect` — "UWF starts protecting the volume after the next device restart in which UWF is enabled." + "This method does not enable UWF if it is disabled".
- `UWF_Volume.Unprotect` — "after the next system restart".
- `UWF_Volume.AddExclusion` / `RemoveExclusion` / `RemoveAllExclusions` — "you must restart the device for [this change / new exclusions] to take effect."
- `UWF_OverlayConfig` (class Remarks) — "Changes to the overlay configuration take effect on the next restart in which UWF is enabled."
- `UWF_OverlayConfig.SetType` — "Changes to the overlay type take effect during the next device restart in which UWF is enabled."
- `UWF_RegistryFilter` (class Remarks) — "Additions or removals of registry exclusions, including changes to the values of PersistDomainSecretKey and PersistTSCAL, take effect after the next restart in which UWF is enabled."
- `UWF_RegistryFilter.AddExclusion` / `RemoveExclusion` — "You must restart the device before the registry key is excluded from UWF filtering."
- `UWF_Servicing.Enable` / `Disable` — "in the next session after a restart."
- `UWF_Overlay.SetWarningThreshold` / `SetCriticalThreshold` — **no restart statement on either page** → `[文档未说明]`.
- `UWF_Volume.SetBindByDriveLetter` — **no restart statement** → `[文档未说明]`.

### Statements requiring "UWF must be disabled in the current session"

Only these two operations state this precondition:

- `UWF_OverlayConfig.SetMaximumSize` — required for both disk-based and RAM-based overlays.
- `UWF_OverlayConfig.SetType` — required for changing to either disk-based or RAM-based.
- (also stated at the `UWF_OverlayConfig` class level: "Before you can change the **Type** or **MaximumSize** properties, UWF must be disabled in the current session.")

Additional numeric preconditions for disk-based overlay (from `SetMaximumSize` and `SetType`):
- size ≥ **1024** MB;
- system volume free space **greater than** the maximum overlay size.

### Statements requiring an administrator account

`UWF_Filter` (class), `UWF_Filter.Enable`, `.Disable`, `.ResetSettings`, `.ShutdownSystem`, `.RestartSystem`, `UWF_Volume` (class), `UWF_Volume.AddExclusion`, `.RemoveExclusion`, `.RemoveAllExclusions`, `.CommitFile`, `.CommitFileDeletion`, `UWF_Overlay.GetOverlayFiles`, `UWF_RegistryFilter.CommitRegistry`, `.CommitRegistryDeletion`.

`UWF_Filter` class Remarks additionally state: "Users with any kind of account can read the current configuration settings."

### Return value convention

Every method page in the reference uses the identical sentence: *"Returns an HRESULT value that indicates WMI status or a WMI error."* linking to
<https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-non-error-constants> and
<https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-error-constants>.

**No UWF-specific HRESULT values are documented on any page.** The only enumerated status codes anywhere in the reference are:

- `UWF_Servicing.UpdateWindows` out-parameter `UpdateStatus`: `0` = Success, `3010` = Restart required, any other value = Generic error.
- Event-log message IDs: `UWF_OVERLAY_REACHED_WARNING_LEVEL` = `0x80010001L`, `UWF_OVERLAY_REACHED_CRITICAL_LEVEL` = `0x80010002L`.

Per-method failure codes (e.g. what `AddExclusion` returns for a forbidden path, what `Protect` returns when the volume doesn't exist): `[文档未说明]`.

### How the official samples retrieve instances

| Class | Retrieval shown in docs | WQL used? |
| --- | --- | --- |
| `UWF_Filter` | `Get-WMIObject -namespace $NAMESPACE -class UWF_Filter` | No |
| `UWF_Overlay` | `Get-WMIObject -namespace $NAMESPACE -class UWF_Overlay` | No |
| `UWF_Volume` | `Get-WMIObject -class UWF_Volume @CommonParams \| where { $_.DriveLetter -eq "$driveLetter" -and $_.CurrentSession -eq $false }` | No (client-side filter) |
| `UWF_RegistryFilter` | `Get-WMIObject -class UWF_RegistryFilter @CommonParams \| where { $_.CurrentSession -eq $false }` | No (client-side filter) |
| `UWF_Servicing` | `Get-WmiObject -class UWF_Servicing @CommonParams \| where { $_.CurrentSession -eq $false }` | No (client-side filter) |
| `UWF_OverlayConfig` | `Get-WMIObject -class UWF_OverlayConfig -Filter "CurrentSession = false" @CommonParams` | **Yes** — the only WQL filter in the entire reference |
| `UWF_ExcludedFile` | via `UWF_Volume.GetExclusions()` → `.ExcludedFiles[].FileName` | n/a (class not directly queryable) |
| `UWF_ExcludedRegistryKey` | via `UWF_RegistryFilter.GetExclusions()` → `.ExcludedKeys[].RegistryKey` | n/a (class not directly queryable) |
| `UWF_OverlayFile` | via `UWF_Overlay.GetOverlayFiles(Volume)` → `OverlayFiles[]` | n/a (class not directly queryable) |

### Path/string formats used in the official samples

- Volume identifier for `UWF_Volume.DriveLetter` comparison: `"C:"` / `"D:"` (letter + colon).
- File exclusion path: volume-relative, leading backslash, no drive letter — `"\Users\Public\Public Documents"`, `"\myfolder\myfile.txt"`.
- `CommitFile` / `CommitFileDeletion` path: "does not include the drive letter or volume name. For example, `\users\test.dat`".
- `GetOverlayFiles` `Volume` parameter: "the drive letter or volume name" (format not further specified — `[文档未说明]`).
- Registry key: long form `"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer"`, never `HKLM:\`.
- `VolumeName` equals `Win32_Volume.DeviceID` (e.g. the `\\?\Volume{GUID}\` form — the docs give the equivalence but not the literal format).

---

## Conflicts and inconsistencies (listed, not adjudicated)

### A. Docs vs. project brief

| # | Brief statement | Docs text | Verdict |
| --- | --- | --- | --- |
| A1 | "`UWF_Volume` 每卷两个实例，以 `CurrentSession` 布尔为 key 区分；只能修改 CurrentSession=false 的那个。" | uwf-volume sample: "Each volume has two entries in UWF_Volume… You can only change the protection status of a drive for the next session". | **Consistent** for protection status. The general claim ("只能修改 …那个" for *all* writes) is `[文档未说明]`. |
| A2 | "文件排除列表不能用 WQL 枚举 `UWF_ExcludedFile`（会返回空）" | uwf-excludedfile Remarks: "UWF_ExcludedFile does not represent an actual WMI object, and you cannot use this class to get or set file exclusions." | **Consistent** (docs are stronger). Docs never say the WQL result is *empty* specifically — the observed empty-result behavior is `[文档未说明]`. |
| A3 | "注册表排除同理走 `UWF_RegistryFilter` 的方法" | uwf-registryfilter documents `AddExclusion`/`RemoveExclusion` on `UWF_RegistryFilter`. **But** uwf-excludedregistrykey Remarks says: "You can use the **UWF_Volume**.AddExclusion and **UWF_Volume**.RemoveExclusion methods to add or remove registry keys to the UWF registry key exclusion list." | **Docs contradict themselves.** Brief agrees with the `UWF_RegistryFilter` page. Not adjudicated. |
| A4 | "`UWF_Overlay` 的 OverlayConsumption/AvailableSpace/两个阈值都是 UInt32、单位 MB" | uwf-overlay properties table: all four are `Uint32`, all "in megabytes". | **Consistent.** |
| A5 | "有工具显示『当前覆盖层消耗: -1 MB』… `UWF_Overlay.OverlayConsumption` 是 UInt32" | Docs confirm `OverlayConsumption` is `UInt32`. Separately, `UWF_OverlayConfig.MaximumSize` is documented as **`SInt32`** (signed) — a *different* property on a *different* class. | **No contradiction**, but a signed overlay-size field does exist in the docs (`UWF_OverlayConfig.MaximumSize`). Recorded for the record; not adjudicated. |
| A6 | "卷绑定方式(BindByDriveLetter vs volume name)要在 UI 暴露" | uwf-volume: `BindByDriveLetter` is `[read, write]`, plus method `SetBindByDriveLetter`. true = drive letter (loose), false = volume name (tight). | **Consistent.** Note two independent write paths exist (property write and method). Which one is authoritative: `[文档未说明]`. |
| A7 | "功能未启用时整个命名空间查不到类" | **Nothing in the WMI provider reference describes what happens when the UWF optional feature is not installed.** | `[文档未说明]` — must be confirmed on the VM. |
| A8 | MVP 需要 "HORM 状态显示" | **No HORM class, property, or method appears anywhere in the UWF WMI provider reference** (the 9 documented classes contain no HORM member). | `[文档未说明]` in this reference. HORM may be documented outside the WMI provider reference (e.g. `uwfmgr.exe`) — not transcribed here. **Flag for Controller.** |

### B. Docs vs. docs (internal inconsistencies)

| # | Location A | Location B |
| --- | --- | --- |
| B1 | `UWF_Volume` MOF: `CommitFile([in] string FileFullPath)` | `uwf-volumecommitfile` page: `CommitFile([in] string FileName)`; Parameters section documents **FileName** |
| B2 | `UWF_Volume` MOF: `SetBindByDriveLetter(boolean bBindByVolumeName)` | `uwf-volumesetbindbydriveletter` page: `SetBindByDriveLetter(boolean bBindByDriveLetter)` |
| B3 | `UWF_Servicing` MOF: `[read] boolean ServicingEnabled;` | `UWF_Servicing` properties table: property named **`ServiceEnabled`** |
| B4 | `UWF_Overlay` MOF: `[key] string Id;` | `UWF_Overlay` properties table: property named **`ID`** |
| B5 | `UWF_ExcludedRegistryKey` Remarks: use `UWF_Volume.AddExclusion` / `.RemoveExclusion` for registry keys | `UWF_RegistryFilter` page: `UWF_RegistryFilter.AddExclusion` / `.RemoveExclusion` are the registry methods; `UWF_Volume.AddExclusion` is documented as a *file* exclusion method |
| B6 | Index table: "UWF_OverlayFile — Displays and configures global settings for the UWF overlay. You can modify the maximum size and the type of the UWF overlay." | `uwf-overlayfile` page: "Contains a file that is currently in the overlay for a volume protected by UWF." (the index text matches `UWF_OverlayConfig` instead) |
| B7 | `UWF_Filter` MOF declares `UInt32 RestartSystem();` and the class Methods table lists it | `uwf-filterrestartsystem` Remarks: "You can't run on WMI providers; it's only available from Intune/CSP." |
| B8 | `UWF_RegistryFilter.RemoveExclusion` Remarks: "You must restart the device before the registry key is **excluded** from UWF filtering." | Same sentence as the *Add* page — presumably should read "no longer excluded" / "removed" |
| B9 | `UWF_ExcludedRegistryKey` Remarks: "You can use the UWF_RegistryFilter.GetExclusions or **UWF_RegistryFilter.FindExclusion** methods to retrieve UWF_ExcludedRegistryKey objects." | `uwf-registryfilterfindexclusion`: `FindExclusion` returns only `[out] boolean bFound`, no `UWF_ExcludedRegistryKey` objects |
| B10 | `UWF_Volume` MOF: `FindExclusion([in] string FileName, [out] bFound)` — **no type** on `bFound` | `uwf-volumefindexclusion` page: `[out] boolean bFound` |
| B11 | `UWF_OverlayConfig` MOF/property table: `MaximumSize` is `SInt32` | `uwf-overlayconfigsetmaximumsize`: setter parameter is `UInt32 size` |
| B12 | `UWF_RegistryFilter` MOF: `CommitRegistryDeletion( string Registrykey, ...)` (lower-case k) | `uwf-registryfiltercommitregistrydeletion` Parameters prose: **RegistryKey** |

### C. Documented gaps relevant to `IUwfProvider`

- No UWF-specific HRESULT / error-code table anywhere → error mapping cannot be built from docs alone.
- Empty-list semantics documented only for `UWF_Volume.GetExclusions` and `UWF_RegistryFilter.GetExclusions` (both → **null**, not empty array). `UWF_Overlay.GetOverlayFiles` empty behavior is `[文档未说明]`.
- Behavior when writing to a `CurrentSession = true` instance is `[文档未说明]` for every class.
- Whether `Protected` / `BindByDriveLetter` / `PersistTSCAL` / `PersistDomainSecretKey` can be set by direct property write (vs. the corresponding method) is `[文档未说明]`; the docs mark them `[read, write]` but every sample uses methods where a method exists. Note `PersistTSCAL` and `PersistDomainSecretKey` have **no** corresponding method — property write is the only documented path.
- Detection of "UWF feature not installed" is `[文档未说明]`.
- HORM is `[文档未说明]` in this reference.
- Existence of `UWF_ServicingHelper` is `[文档未说明]` (no page exists).

---

## 非官方补充 (Unofficial supplements)

**None.** No blog, StackOverflow, or third-party source was used in this document. Every statement above is traceable to a `learn.microsoft.com` page URL cited inline.

If unofficial sources are later needed (e.g. for real observed HRESULT values or the UWF_ExcludedFile empty-WQL behavior), they must be added here and labeled as unofficial — never merged into the sections above.
