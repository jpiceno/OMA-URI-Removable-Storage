# Device Control Policy Management with Intune and Windows

This document outlines how to manage **device control policies** in **Microsoft Intune** and **Windows** using different policy types, XML configurations, and best practices.

---

## **Management Tools and How Rules Are Managed**

| Management Tool                     | Operating System | Description |
|-------------------------------------|-----------------|-------------|
| **Intune – Device Control Policy**  | Windows         | Device and printer groups can be managed as reusable settings and included in rules. Not all features are available in the device control policy. See [Deploy and manage device control with Microsoft Intune](https://docs.microsoft.com/en-us/mem/intune/protect/device-control). |
| **Intune – Custom**                 | Windows         | Each group/rule is stored as an XML string in a custom configuration policy. The OMA-URI contains the GUID of the group/rule. The GUID must be generated manually. |

---

## **Rules and Entries in Device Control Policies**

A **rule** defines the list of **included groups** and **excluded groups**. The device must be a **member of all included groups** and **not part of any excluded groups** for the rule to apply. If a device matches a rule, then its **entries** are evaluated.

- An **entry** defines the **action** and **notification options** applied when a request matches the conditions.
- If no rules apply, the **default enforcement** is applied.

## **Entries in Device Control Policies**
Device control policies define **access permissions** (called an **entry**) for a set of devices.

### **Entry Properties**
| Entry Setting     | Options |
|------------------|---------|
| **AccessMask**   | Defines allowed operations (bitwise OR of values). |
| **Action**       | `Allow`, `Deny`, `AuditAllow`, `AuditDeny` |
| **Notification** | `None`, `Event is generated`, `User receives notification` |

### **AccessMask Values**
| Access Type     | Value |
|---------------|-------|
| **Device Read** | `1` |
| **Device Write** | `2` |
| **Device Execute** | `4` |
| **File Read** | `8` |
| **File Write** | `16` |
| **File Execute** | `32` |
| **Print** | `64` |

Example AccessMask combinations:
- **Device Read + Write + Execute** → `7` (1 + 2 + 4)
- **Device Read + File Read** → `9` (1 + 8)

---

## **Conditions for Entries**
Entries can be **restricted to specific users and devices**.

### **Supported Conditions**
- **User/User Group Condition** → Restricts access to a specific **user or group**.
- **Machine Condition** → Restricts access to a specific **device or device group**.
- **Parameters Condition** → Applies conditions based on additional factors.

To retrieve a **user's SID**, run the following PowerShell command:
```powershell
whoami /user

## **Example: Allowing Read-Only USB Access**
The following XML snippet allows **read-only access** for specific devices:

```xml
<Entry Id="{e3837e60-5e56-43ce-8095-043ccd793eac}">
    <Type>Allow</Type>
    <Options>0</Options>
    <AccessMask>1</AccessMask> <!-- Device Read Only -->
</Entry>

### **Example Policy - Read-Only USB Access**
The following XML snippet demonstrates how to allow **read-only access** for certain USB devices:

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
</PolicyRule> 
