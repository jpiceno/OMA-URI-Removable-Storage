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
# Device Control Policy XML Configuration

## XML Configuration Example
```xml
<PolicyRule Id="{75a4e33a-5268-4552-bef2-e34dd0c39cb1}">
  <Name>Read Only Access for USBs</Name>
  <IncludedIdList>
      <GroupId>{3f5253e4-0e73-4587-bb9e-bb29a2171694}</GroupId>
  </IncludedIdList>
  <ExcludedIdList>
      <GroupId>{3f5253e4-0e73-4587-bb9e-bb29a2171695}</GroupId>
  </ExcludedIdList>
  <Entry Id="{e3837e60-5e56-43ce-8095-043ccd793eac}">
   ...
  </Entry>
  <Entry Id="{34413b98-8198-4e16-accf-c95c3c775ba3}">
    ...
  </Entry>
</PolicyRule>
```

## Property Descriptions

| Property Name     | Description | Options |
|------------------|-------------|---------|
| **PolicyRule Id** | GUID, a unique ID, represents the policy and is used in reporting and troubleshooting. | Generate via PowerShell. |
| **Name** | String, the name of the policy and displays on toast notifications. | User-defined |
| **IncludedIdList** | The groups that the policy applies to. If multiple groups are added, the media must be a member of each group in the list to be included. | The Group ID/GUID must be used. |
| **ExcludedIdList** | The groups that the policy doesn't apply to. If multiple groups are added, the media must be a member of a group in the list to be excluded. | The Group ID/GUID must be used. |
| **Entry** | One PolicyRule can have multiple entries; each entry with a unique GUID tells device control one restriction. | See Entry Properties table for details. |

### Example Group ID Usage:
```xml
<IncludedIdList>
    <GroupId>{EAA4CCE5-F6C9-4760-8BAD-FDCC76A2ACA1}</GroupId>
</IncludedIdList>
```

## Entry Configuration

Device control policies define access (called an **entry**) for a set of devices. Entries define the action and notification options for devices that match the policy and conditions defined in the entry.

### Entry Settings and Options

| Entry Setting   | Options |
|----------------|---------|
| **AccessMask** | Applies action if operations match: <br>- `1` - Device Read <br>- `2` - Device Write <br>- `4` - Device Execute <br>- `8` - File Read <br>- `16` - File Write <br>- `32` - File Execute <br>- `64` - Print <br> **Example:** Device Read, Write, and Execute = `7` (1+2+4) |
| **Action** | `Allow`, `Deny`, `AuditAllow`, `AuditDeny` |
| **Notification** | `None` (default), Event generated, User notified |

## Entry Evaluation

There are two types of entries:
- **Enforcement entries**: `Allow` / `Deny`
- **Audit entries**: `AuditAllow` / `AuditDeny`

### **Enforcement Entries**
Enforcement entries for a rule are evaluated **in order** until all requested permissions are matched. If no entries match a rule, the next rule is evaluated. If no rules match, then the **default policy** is applied.

### **Audit Entries**
Audit events control the behavior when device control enforces a rule (`Allow/Deny`). Device control can:
- Display a **notification** to the end-user, which includes the policy name and the device name. The notification appears once **every hour** after initial access is denied.
- Create an **event** available in **Advanced Hunting**.

### **Important**
- There's a limit of **300 events per device per day**.
- Audit entries are processed **after** the enforcement decision is made.
- All corresponding audit entries are evaluated.

---
This document provides a structured guide to defining **Device Control Policies** using XML configuration.

