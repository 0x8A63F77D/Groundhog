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
