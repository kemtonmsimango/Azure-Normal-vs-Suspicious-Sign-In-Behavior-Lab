#Normal vs Suspicious Sign In Behavior Lab
Overview

This lab focuses on understanding user behavior through logs by comparing normal sign in activity with suspicious looking patterns using Microsoft Entra ID and Log Analytics.

The goal is not to attack or exploit anything.
The goal is to learn how defenders read logs, establish a baseline, and identify when something does not look right.

This is the same mindset used in SOC and cloud security roles.


#What This Lab Demonstrates

✅ How to set up a clean logging environment
✅ How to establish a baseline of normal behavior
✅ How suspicious behavior stands out only after baseline exists
✅ How Sign in Logs and Audit Logs are reviewed in Log Analytics
✅ How small signals combine into meaningful observations

Step 1 Create the Lab Environment
Step 1.1 Confirm or create an Entra tenant

I used a dedicated lab tenant to ensure all activity was controlled and intentional.

Why this matters
Having a separate tenant ensures all logs belong to known users and known actions, which makes analysis accurate and repeatable.

Screenshots
Tenant overview in Entra admin center showing directory details

Step 1.2 Create test users

I created two users in Microsoft Entra ID

NormalUser
TestUser

Why this matters
Separating users allows me to clearly compare expected behavior versus unexpected behavior without mixing signals.

Screenshots
All users list showing both accounts
Profile summary for NormalUser
Profile summary for TestUser

Security note
Passwords are never shown or captured in screenshots.

Step 2 Enable Logging and Connect Log Analytics
Step 2.1 Create Log Analytics Workspace

I created a Log Analytics workspace in Azure and assigned it to a resource group and region.

Why this matters
Log Analytics is where raw sign in and audit data becomes searchable and analyzable using KQL.

Screenshots
Workspace overview page
Workspace essentials showing resource group and region

Step 2.2 Connect Entra logs using Microsoft Sentinel

I enabled Microsoft Sentinel on the Log Analytics workspace and connected Entra ID logs.

Connected log types
Sign in logs
Audit logs

Why this matters
Using Sentinel ensures SigninLogs and AuditLogs tables are created automatically and continuously populated.

Screenshots
Sentinel overview page attached to the workspace
Data connector showing connected status
Connector page listing ingested log types

Step 2.3 Verify logs are available

I confirmed that log tables existed by running simple queries in Log Analytics.

SigninLogs
AuditLogs

Why this matters
If logs exist, data is flowing. If logs are empty, activity must be generated before analysis begins.

Screenshots
SigninLogs returning results
AuditLogs returning results

Step 3 Establish a Baseline of Normal Behavior
Step 3.1 Plan baseline activity

For NormalUser, I planned predictable sign ins

Same device
Same browser
Same network
Different times of day

Why this matters
A baseline only works if behavior is consistent. Random activity creates noise and weakens conclusions.

Step 3.2 Perform normal sign ins

I signed into Microsoft 365 and Entra portals normally using NormalUser.

No risky actions
No unusual locations
No new devices

Step 3.3 Query baseline sign in logs

Query used

SigninLogs
| where UserPrincipalName has "normaluser"
| project TimeGenerated, UserPrincipalName, AppDisplayName, ResultType, ResultDescription, IPAddress, LocationDetails, DeviceDetail, ClientAppUsed
| order by TimeGenerated desc


What I observed
Consistent success results
Same device and browser
Same general location
Predictable sign in times

Screenshots
Baseline sign in rows
Single event detail showing successful result

Why this matters
This baseline becomes the reference point for everything that follows.

Step 4 Simulate Safe Suspicious Scenarios

All scenarios below were performed using TestUser only.

Each scenario was followed immediately by log review to capture fresh data.

Scenario A Multiple failed sign ins then success

Actions performed
Entered incorrect password multiple times
Then successfully signed in once

Query used

SigninLogs
| where UserPrincipalName has "testuser"
| project TimeGenerated, ResultType, ResultDescription, IPAddress, LocationDetails, DeviceDetail, ClientAppUsed
| order by TimeGenerated desc


What stood out
Burst of failures close together
Success immediately after failures

Screenshots
Failure burst pattern
Failure event details
Success event details

Why this matters
Compared to baseline behavior, this pattern is abnormal and high signal.

Scenario B New browser or device

Actions performed
Signed in using a different browser or incognito
Signed in from another device when possible

Query used

SigninLogs
| where UserPrincipalName has "testuser"
| project TimeGenerated, IPAddress, LocationDetails, DeviceDetail, ClientAppUsed
| order by TimeGenerated desc


What stood out
Different ClientAppUsed
Different DeviceDetail

Screenshots
Events showing different devices or browsers
Event detail highlighting new client information

Why this matters
When baseline is consistent, even small changes stand out quickly.

Scenario C Role assignment change

Actions performed
Temporarily assigned a low risk role
Removed the role shortly after

Query used

AuditLogs
| where Category == "RoleManagement"
| project TimeGenerated, OperationName, InitiatedBy, TargetResources
| order by TimeGenerated desc


What stood out
Clear role assignment event
Clear role removal event

Screenshots
Role assignment event
Role removal event
Event detail showing who initiated the change

Why this matters
Normal sign ins do not include role changes. This is a strong audit signal.

Scenario D Unusual sign in time

Actions performed
Signed in at a time outside the established baseline window

Query used

SigninLogs
| where UserPrincipalName has "testuser"
| summarize SignInCount=count() by bin(TimeGenerated, 1h)
| order by TimeGenerated desc


What stood out
Sign ins appearing in unusual hourly blocks

Screenshots
Hourly aggregation view
Single event detail showing unusual time

Why this matters
Same user and location can still be suspicious if the timing breaks routine.

Key Takeaways

✅ Logs only make sense after a baseline exists
✅ Suspicious behavior is often subtle, not dramatic
✅ Context matters more than single events
✅ Consistency makes anomalies obvious
✅ This is how SOC analysts think when reviewing logs

Next Improvements

Add alert rules based on observed patterns
Visualize behavior trends over time
Expand scenarios using conditional access
