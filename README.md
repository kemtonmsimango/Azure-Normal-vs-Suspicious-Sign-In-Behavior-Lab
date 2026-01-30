# Normal vs Suspicious Sign In Behavior Lab

## Overview

This project focuses on **studying sign in and audit logs** to understand how normal user behavior differs from suspicious looking activity in Microsoft Entra ID.

The lab is built around a simple principle:

Establish what normal looks like first, then identify what stands out.

---

## What This Lab Demonstrates

✅ Creating a clean identity lab environment  
✅ Enabling and verifying sign in and audit logging  
✅ Establishing a baseline of normal behavior  
✅ Reviewing suspicious looking sign in patterns  
✅ Using logs as evidence rather than assumptions  

---

# Step 1 Create the Lab Environment

## Step 1.1 Confirm or create an Entra tenant

I confirmed that I was working inside a dedicated lab tenant.

Why this step matters

- Keeps all activity isolated  
- Ensures logs belong only to known test users  
- Prevents noise from real users  

Screenshots

- Entra admin center tenant overview

---

## Step 1.2 Create test users

I created two users in Microsoft Entra ID.

Users created

- NormalUser  
- TestUser  

Why this step matters

- NormalUser represents expected behavior  
- TestUser is used to generate controlled suspicious patterns  

Screenshots

- All users list showing both accounts  
- NormalUser profile summary  
- TestUser profile summary  

Security note

- Passwords are never displayed or captured

---

# Step 2 Enable Logging and Log Analytics

## Step 2.1 Create a Log Analytics workspace

I created a Log Analytics workspace in Azure and assigned it to a resource group and region.

Why this step matters

- Provides a central place to query logs  
- Enables KQL based investigation  
- Mirrors how SOC analysts work  

Screenshots

- Log Analytics workspace overview  
- Workspace essentials section

---

## Step 2.2 Connect Entra logs using Microsoft Sentinel

I enabled Microsoft Sentinel on the Log Analytics workspace and connected Entra ID logs.

Log types connected

- Sign in logs  
- Audit logs  

Why this step matters

- Automatically creates SigninLogs and AuditLogs tables  
- Ensures continuous ingestion of identity activity  

Screenshots

- Sentinel overview page  
- Data connector showing connected status  
- Connector page listing ingested logs  

---

## Step 2.3 Verify log ingestion

I verified that logs were available in Log Analytics.

Queries executed

- SigninLogs  
- AuditLogs  

Why this step matters

- Confirms data is flowing  
- Prevents investigating empty or broken tables  

Screenshots

- SigninLogs returning results  
- AuditLogs returning results  

---

# Step 3 Establish Baseline Normal Behavior

## Step 3.1 Plan baseline sign in behavior

I planned consistent sign ins for NormalUser.

Baseline plan

- Same device  
- Same browser  
- Same network  
- Different times of day  

Why this step matters

- Consistency creates a reliable baseline  
- Makes deviations easy to spot later  

---

## Step 3.2 Perform baseline sign ins

I signed in normally using NormalUser.

Actions performed

- Signed into Microsoft 365 portal  
- Signed into Entra portal  
- No risky or unusual actions  

Why this step matters

- Generates clean and predictable log entries  

---

## Step 3.3 Review baseline sign in logs

I queried baseline sign in activity in Log Analytics.

Query used

