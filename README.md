# Normal vs Suspicious Sign In Behavior Lab
Overview

In this lab, I built a small identity and logging environment to understand what normal sign in behavior looks like first, before comparing it to safe, simulated suspicious behavior.

The goal was not to “hack” anything, but to learn how logs tell a story when behavior changes.

✅ Establish a clean baseline
✅ Observe deviations from that baseline
✅ Practice reading SigninLogs and AuditLogs like a SOC analyst

<p align="center">
  <img src="images/Employees (3).png" width="750">
</p>


Lab Objectives

Create a real Microsoft Entra ID test tenant

Generate normal sign in activity

Safely simulate suspicious looking behavior

Query and analyze logs using Log Analytics

Document what changed, why it stood out, and how I noticed

# 1. Create Lab Basics
Step 1.1 Confirm or Create a Tenant

I used an existing lab tenant (or created a new one if needed) to ensure all activity was isolated and safe.

This confirms the lab is real and not theoretical.

Screenshot 1

<p align="center">
  <img src="images/Step 1.1 Create A Lab Default Directory.png" width="750">
</p>



Entra admin center home showing tenant name and directory overview.

## Step 1.2 Create Two Test Users in Entra

In the Entra admin center:

Users → All users → New user

I created two users:

NormalUser

TestUser

Both accounts were tested to ensure successful sign in.

✅ Separate users make baseline vs suspicious behavior easier to compare
✅ Prevents confusing log patterns later

Screenshot 2

<p align="center">
  <img src="images/Step 1.2 Image of all users created.png" width="750">
</p>


Screenshot 3

<p align="center">
  <img src="images/Step 1.2 Normal User Summary.png" width="750">
</p>

Screenshot 4

<p align="center">
  <img src="images/Step 1.2 Test User Summary.png" width="750">
</p>


⚠️ Important: I never screenshot passwords or sensitive credentials.

# 2. Enable Logging and Connect to Log Analytics

## Step 2.1 Create a Log Analytics Workspace

In the Azure portal:

Create a resource → Log Analytics workspace

Select resource group and region

This workspace is where all sign in and audit logs are queried.

Screenshot 5

<p align="center">
  <img src="images/Step 2 Create a Log analytics .jpg" width="750">
</p>

Screenshot 6

<p align="center">
  <img src="images/Step 2 Log Analytics workspaces.jpg" width="750">
</p>


Step 2.2 Connect Entra Logs Using Microsoft Sentinel

I used Microsoft Sentinel because it is the cleanest way to ingest:

✅SigninLogs

✅AuditLogs

Steps:

Microsoft Sentinel → Create

Select Log Analytics workspace → Add

Open Microsoft Entra ID / Azure AD data connector

Connect required log types

✅ Ensures logs flow correctly
✅ Avoids missing tables or partial data

Screenshot 7

<p align="center">
  <img src="images/Step 2.2.2 microsoft sentienl .jpg" width="750">
</p>



Screenshot 8

<p align="center">
  <img src="images/Step 2.2.3 Connect the Microsoft ENtra ID Data connector .jpg
" width="750">
</p>


Screenshot 9

![Log Types Enabled](screenshots/09-log-types-enabled.png)

Step 2.3 Confirm Logs Are Ingesting

In Log Analytics → Logs, I ran:

SigninLogs
AuditLogs


Screenshot 10

![SigninLogs Table](screenshots/10-signinlogs-table.png)


Screenshot 11

![AuditLogs Table](screenshots/11-auditlogs-table.png)


ℹ️ If logs did not appear immediately, I continued generating activity and checked again later.

3. Baseline Normal Behavior
Goal

Create clean, predictable sign in behavior using NormalUser.

This baseline is critical because suspicious behavior only stands out when normal behavior is understood first.

Step 3.1 Plan the Baseline

I planned three normal sign ins:

Morning

Afternoon

Night

All sign ins used:

Same device

Same browser

Same network

✅ Consistency makes deviations obvious

Step 3.2 Perform Baseline Sign Ins

I signed in normally to:

Microsoft 365 portal

Entra admin center

No risky behavior was performed.

Step 3.3 Query Baseline Logs

Query used:

SigninLogs
| where UserPrincipalName has "normaluser"
| project TimeGenerated, UserPrincipalName, AppDisplayName, ResultType, ResultDescription, IPAddress, LocationDetails, DeviceDetail, ClientAppUsed
| order by TimeGenerated desc


Screenshot 12

![NormalUser Baseline Rows](screenshots/12-normaluser-baseline.png)


Screenshot 13

![Baseline Event Details](screenshots/13-baseline-event-details.png)

Baseline Notes

Typical sign in times

Typical location

Typical device and browser

ResultType = 0 (success)

MFA behavior (if enabled)

✅ This becomes the reference point for everything else.

4. Safe Suspicious Scenarios

All scenarios below were performed using TestUser.

After each scenario, I immediately queried logs and captured screenshots while events were fresh.

Scenario A: Multiple Failed Sign Ins Then Success

Steps

Enter wrong password 3–5 times

Then sign in correctly once

Query

SigninLogs
| where UserPrincipalName has "testuser"
| project TimeGenerated, ResultType, ResultDescription, IPAddress, LocationDetails, DeviceDetail, ClientAppUsed
| order by TimeGenerated desc


Screenshot 14

![Failure Burst Pattern](screenshots/14-failure-burst.png)


Screenshot 15

![Failure Event Details](screenshots/15-failure-event.png)


Screenshot 16

![Success After Failures](screenshots/16-success-after-failure.png)


Observations

Burst of failures is unusual

Success immediately after failures stands out compared to baseline

Scenario B: New Browser or New Device

Steps

Sign in using a different browser or incognito

Optional second device

Query

SigninLogs
| where UserPrincipalName has "testuser"
| project TimeGenerated, IPAddress, LocationDetails, DeviceDetail, ClientAppUsed
| order by TimeGenerated desc


Screenshot 17

![Different Device or Browser](screenshots/17-device-change.png)


Screenshot 18

![New Device Details](screenshots/18-new-device-details.png)


Observations

New device or browser breaks the baseline pattern

Location changes increase suspicion

Scenario C: Role Assignment Change (Audit Logs)

Steps

Assign a low-risk role (e.g. Reports Reader)

Remove the role after a few minutes

Query

AuditLogs
| where Category == "RoleManagement"
| project TimeGenerated, OperationName, InitiatedBy, TargetResources
| order by TimeGenerated desc


Screenshot 19

![Role Assignment Event](screenshots/19-role-assigned.png)


Screenshot 20

![Role Removal Event](screenshots/20-role-removed.png)


Screenshot 21

![Audit Event Details](screenshots/21-audit-details.png)


Observations

Normal sign ins do not involve role changes

Unexpected role assignments are high-signal events

Scenario D: Unusual Sign In Time

Steps

Sign in TestUser outside normal baseline hours

Query

SigninLogs
| where UserPrincipalName has "testuser"
| summarize SignInCount=count() by bin(TimeGenerated, 1h)
| order by TimeGenerated desc


Screenshot 22

![Unusual Hour Histogram](screenshots/22-unusual-hour.png)


Screenshot 23

![Event Time Details](screenshots/23-time-details.png)


Observations

Same user and location

Unusual time compared to baseline routine

Key Takeaway

Security is not always about reacting to alerts.

Sometimes it is about noticing when something does not fit.

By establishing a baseline first, even small changes in behavior become visible and meaningful.
