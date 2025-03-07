# Device Control Policies

This article describes device control policies, rules, entries, groups, and advanced conditions. Essentially, device control policies define access for a set of devices. The devices that are in scope are determined by a list of included device groups and a list of excluded device groups. A policy applies if the device is in all of the included device groups and none of the excluded device groups. If no policies apply, then the default enforcement is applied.

## Configure Removable Storage Access Control using OMA-URI

1. Go to the **Microsoft Intune** admin center and sign in.
2. Navigate to **Devices > Configuration profiles**.
3. Under the **Policies** tab (selected by default), select **+ Create**, and choose **+ New policy** from the drop-down that appears.
4. In the **Platform** list, select **Windows 10, Windows 11, and Windows Server**, then choose **Templates** from the **Profile type** drop-down list.
5. Once you choose **Templates**, the **Template name** pane is displayed along with a search box.
6. Select **Custom** from the **Template name** pane and click **Create**.
7. Create a row for each setting, group, or policy by implementing steps 1-5.

## Default Device Control Settings

By default, device control is disabled, allowing access to all types of devices. Below are three **OMA-URI** settings to modify the default control and enforcement:

### Device Control Default Enforcement
- **OMA-URI:** `./Vendor/MSFT/Defender/Configuration/DefaultEnforcement`
- **Data Type:** Integer
- **Values:**
  - `1` - DefaultEnforcementAllow
  - `2` - DefaultEnforcementDeny

### Enable Device Control
- **OMA-URI:** `./Vendor/MSFT/Defender/Configuration/DeviceControlEnabled`
- **Data Type:** Integer
- **Values:**
  - `0` - Disable
  - `1` - Enable

## Rules

A rule defines a list of included and excluded groups. A rule applies if the device belongs to all the included groups and none of the excluded groups. If the device matches the rule, the entries for that rule are evaluated.

### Example: USB Write and Read Access Control
To allow **write access** for some USB devices while providing **read access** to all others, use the following:

| Property Name    | Description | Options |
|-----------------|-------------|---------|
| PolicyRule Id   | GUID, unique policy identifier used in reporting/troubleshooting | Generate via PowerShell |
| Name            | String, policy name displayed on toast notifications | User-defined |
| IncludedIdList  | Groups the policy applies to | Group ID/GUID |
| ExcludedIdList  | Groups the policy does not apply to | Group ID/GUID |

#### Example XML for Included ID List:
```xml
<IncludedIdList>
    <GroupId>{EAA4CCE5-F6C9-4760-8BAD-FDCC76A2ACA1}</GroupId>
</IncludedIdList>
```

## Entries

Device control policies define access (called an **entry**) for a set of devices. Entries specify action and notification options for matching policies and conditions.

### Entry Settings and Options

| Entry Setting   | Options |
|----------------|---------|
| **AccessMask** | Applies action if operations match: <br>- `1` - Device Read <br>- `2` - Device Write <br>- `4` - Device Execute <br>- `8` - File Read <br>- `16` - File Write <br>- `32` - File Execute <br>- `64` - Print <br> **Example:** Device Read, Write, and Execute = `7` (1+2+4) |
| **Action** | `Allow`, `Deny`, `AuditAllow`, `AuditDeny` |
| **Notification** | `None` (default), Event generated, User notified |

## Access Types

| Access Type      | Value | Description |
|------------------|-------|-------------|
| **Device Read**  | `1`   | Grants permission to read data from the device itself (e.g., querying device info, metadata, or accessing raw device data). |
| **Device Write** | `2`   | Grants permission to write data to the device itself, such as formatting a USB drive or modifying its partition structure. |
| **Device Execute** | `4` | Grants permission to execute code or commands on the device (rarely used). |
| **File Read**  | `8`   | Grants permission to read files stored on the device (e.g., opening a document from a USB drive). |
| **File Write** | `16`  | Grants permission to write files onto the device (e.g., saving a document to a USB drive). |
| **File Execute** | `32`  | Grants permission to execute files stored on the device (e.g., running a .exe file from a USB drive). |

## Conditions

Entries can be further scoped using optional conditions:

### User/User Group Condition
Applies the action only to users/groups identified by **Security Identifier (SID)**.

- For **Microsoft Entra ID** users/groups, use the **Object ID**.
- For **local** users/groups, use the **SID**.
- Retrieve a user's SID using PowerShell:
  ```powershell
  whoami /user
  ```

### Machine Condition
Applies the action only to a device/group identified by **SID**.

### Parameters Condition
Applies the action only if parameters match specific conditions (See **Advanced Conditions**).

### Example Use Case:
- Allow **read access** to specific USBs **only** for a particular **user** on a specific **device**.

---
This document provides a comprehensive guide to configuring device control policies using **Microsoft Intune OMA-URI** settings.
