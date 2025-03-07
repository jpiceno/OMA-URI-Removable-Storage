# Device Control Policies

This article describes device control policies, rules, entries, groups, and advanced conditions. These policies define access control for a set of devices, determining scope based on included and excluded device groups.

- A policy applies if the device is in all of the included device groups and none of the excluded device groups.
- If no policies apply, then the default enforcement is used.

---

## 1. Default Device Control Settings

By default, device control is disabled, allowing access to all device types. Below are three key OMA-URI settings to modify device control behavior:

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

---

## 2. Configuring Removable Storage Access Control via OMA-URI

To configure device control settings using Microsoft Intune:

1. Go to **Microsoft Intune admin center** and sign in.
2. Navigate to **Devices > Configuration profiles**.
3. Under the **Policies** tab (default view), select **+ Create** and choose **+ New policy**.
4. In the **Platform** list, select **Windows 10, Windows 11, Windows Server**, then choose **Templates**.
5. In the **Template name** pane, select **Custom** and click **Create**.
6. Create a row for each setting, group, or policy by repeating Steps 1-5.

---

## 3. Rules

A rule defines a list of included and excluded groups and applies when the device belongs to all included groups and none of the excluded groups.

### Example: USB Write and Read Access Control

To allow write access for some USB devices while providing read access to all others, use the following:

| Property Name   | Description                                      | Options                 |
|----------------|------------------------------------------------|-------------------------|
| PolicyRule Id  | Unique policy identifier used for reporting/troubleshooting | Generate via PowerShell |
| Name           | Policy name displayed in notifications          | User-defined            |
| IncludedIdList | Groups the policy applies to                   | Group ID/GUID           |
| ExcludedIdList | Groups the policy does NOT apply to            | Group ID/GUID           |

---

## 4. Example XML for Included ID List

```xml
<IncludedIdList>
    <GroupId>{EAA4CCE5-F6C9-4760-8BAD-FDCC76A2ACA1}</GroupId>
</IncludedIdList>
```

---

## 5. Entries

Device control policies define access (entries) for a set of devices. Entries specify actions and notification settings.

### Conditions for Entries
1. **User/User Group Condition**
   - Applies the action only to specific users/groups identified by Security Identifier (SID).
   - **Microsoft Entra ID users/groups:** Use Object ID.
   - **Local users/groups:** Use SID (Retrieve with `whoami /user`).

2. **Machine Condition**
   - Applies the action only to a device/group identified by SID.

3. **Parameters Condition**
   - Applies the action only if parameters match specific conditions.

**Example Use Case:**
- Allow read access to specific USBs only for a particular user on a specific device.

---

## 6. Device Control Policy XML Configuration

### Example XML Configuration

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

---

## 7. Entry Configuration and Access Types

| Entry Setting  | Options  |
|---------------|----------|
| **AccessMask** | Determines applicable actions (see below) |
| **Action** | Allow, Deny, AuditAllow, AuditDeny |
| **Notification** | None (default), Event generated, User notified |

### AccessMask Options

| Access Type   | Value | Description                              |
|--------------|-------|------------------------------------------|
| Device Read  | 1     | Grants permission to read device data   |
| Device Write | 2     | Grants permission to write to device    |
| Device Execute | 4   | Grants permission to execute commands on device |
| File Read    | 8     | Grants permission to read files on device |
| File Write   | 16    | Grants permission to write files on device |
| File Execute | 32    | Grants permission to execute files on device |

**Example Calculation:** To allow Read, Write, and Execute:
- `AccessMask = 1 (Read) + 2 (Write) + 4 (Execute) = 7`

---

## 8. Entry Evaluation

### Types of Entries
1. **Enforcement Entries:** Allow / Deny
2. **Audit Entries:** AuditAllow / AuditDeny

### Enforcement Entries
- Evaluated in order until all requested permissions are matched.
- If no entries match, the next rule is evaluated.
- If no rules match, the default policy is applied.

### Audit Entries
Audit logs track device control activity:
- Displays user notifications every hour for denied access.
- Creates audit events in **Advanced Hunting**.

**Important:** A maximum of **300 audit events per device per day** is allowed.

---

## 9. Additional PowerShell Snippets

### Generate a GUID for `PolicyRule Id`

```powershell
[guid]::NewGuid()
