# AWS IAM Policy Assignments

>
> **Topic:** Identity and Access Management (IAM) Policies on AWS
> **Scope:** Writing and applying IAM policies in JSON format across three real-world scenarios

---

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Assignment 1 — S3 Access Policies](#assignment-1--s3-access-policies)
  - [Teacher Policy](#teacher-policy)
  - [Student Policy](#student-policy)
  - [Administrator Policy](#administrator-policy)
- [Assignment 2 — IAM User Management Policy](#assignment-2--iam-user-management-policy)
- [Assignment 3 — IAM Group Management Policy](#assignment-3--iam-group-management-policy)
- [Key IAM Concepts Applied](#key-iam-concepts-applied)
- [Testing the Policies](#testing-the-policies)
- [Screenshots](#screenshots)

---

## Overview

This repository contains three IAM policy assignments written in JSON format and applied through the AWS Management Console. Each assignment reflects a real-world access control scenario, demonstrating the principle of **least privilege** — granting only the minimum permissions necessary for each role.

All policies were created using the **JSON editor** in the AWS IAM Console and scoped specifically to the resources and actions defined by each use case.

---

## Repository Structure

```
iam-policy/
│
├── assignment1
│   ├── teacher-policy.json
│   ├── student-policy.json
│   └── administrator-policy.json
│
├── assignment2
│   └── junior-admin-user-policy.json
│
├── assignment3
│   └── teamlead-group-policy.json
│
└── README.md
```

---

## Prerequisites

- An active AWS account
- IAM access to create and manage policies, users, and groups
- The S3 bucket `alexander-school-exam-571493306003-eu-north-1-an` already created (for Assignment 1)
- IAM users (`teacher1`, `student1`, etc.) and groups (`Teachers`, `Students`, `Administrators`) already created and linked

---

## Assignment 1 — S3 Access Policies

**Scenario:** A school stores exam files in an Amazon S3 bucket. Different roles require different levels of access, all scoped to a single specific bucket.

**S3 Bucket ARNs used:**
- Bucket: `arn:aws:s3:::alexander-school-exam-571493306003-eu-north-1-an`
- Objects: `arn:aws:s3:::alexander-school-exam-571493306003-eu-north-1-an/*`

---

### Teacher Policy

**Policy name:** `TeacherS3Policy`
**File:** `assignment1/teacher_policy.json`

**Permissions:**
| Action | Allowed |
|---|---|
| Upload files (`s3:PutObject`) | ✅ Yes |
| Download files (`s3:GetObject`) | ✅ Yes |
| List bucket contents (`s3:ListBucket`) | ✅ Yes |
| Delete files (`s3:DeleteObject`) | ❌ No — not granted |


> **Design note:** No explicit `Deny` is needed for deletion. Since `s3:DeleteObject` is never granted, AWS denies it by default. Explicit denies are reserved for scenarios where a conflicting `Allow` from another policy exists.

---

### Student Policy

**Policy name:** `StudentS3Policy`
**File:** `assignment1/student_policy.json`

**Permissions:**
| Action | Allowed |
|---|---|
| Download files (`s3:GetObject`) | ✅ Yes |
| List bucket contents (`s3:ListBucket`) | ✅ Yes |
| Upload files (`s3:PutObject`) | ❌ No — not granted |
| Delete files (`s3:DeleteObject`) | ❌ No — not granted |


---

### Administrator Policy

**Policy name:** `AdminS3ExamPolicy`
**File:** `assignment1/admin_policy.json`

**Permissions:**
| Action | Allowed |
|---|---|
| All S3 actions on this bucket (`s3:*`) | ✅ Yes — full access |


> **Design note:** Both ARNs are required in the Administrator policy. Using `s3:*` with only one ARN would leave the other level unprotected.

---

## Screenshots

### Assignment 1 — S3 Policies

**Teacher Policy — Created in IAM Console**
images/teacher_policy.png

**Student Policy — Created in IAM Console**
images/student_policy.png

images/studentUploadfailed.png

**Administrator Policy — Created in IAM Console**
images/adminS3policy.png

**teacher1 — Upload Test (Expected: Allowed)**
images/teacher_upload.png

images/TeacherS3AccessPolicy.png

**teacher1 — Delete Test (Expected: Denied)**
images/teacher1denied.png

images/user_group_ass1.png

---


## Assignment 2 — IAM User Management Policy

**Scenario:** A junior cloud administrator needs limited IAM user management capabilities — enough to onboard users and reset passwords, but with no ability to perform destructive or sensitive actions.

**Policy name:** `JuniorCloudAdmin`
**File:** `assignment2/junior_cloud_admin.json`

**Permissions:**
| Action | Allowed |
|---|---|
| Create IAM users | ✅ Yes |
| View IAM users | ✅ Yes |
| Reset / set user passwords | ✅ Yes |
| Delete users | ❌ No — explicitly denied |
| Delete policies | ❌ No — explicitly denied |
| Create access keys | ❌ No — explicitly denied |


> **Why use `iam:` prefix?** IAM is the AWS service being managed here. Every AWS action is prefixed with the service it operates on — `s3:` for S3, `ec2:` for EC2, and `iam:` for Identity and Access Management. Since this policy controls who can manage users and credentials inside IAM itself, all actions use the `iam:` prefix.

> **Why explicit `Deny` here?** Unlike Assignment 1, this junior admin may realistically have other policies attached (e.g., a general cloud admin policy) that could grant broader IAM access. The explicit `Deny` ensures that even if another policy grants `iam:DeleteUser`, this block overrides it. In AWS, an explicit `Deny` always wins over an `Allow` from any source.

---
## Screenshots

### Assignment 2 — IAM User Management Policy

**Junior Admin Policy — Created in IAM Console**
images/Assignment2_policy.png

images/jay-cloud-dev_deletePolicy.png



---

## Assignment 3 — IAM Group Management Policy

**Scenario:** A team lead needs to manage IAM groups — creating them, assigning users, and viewing group details — but must be prevented from deleting groups or escalating privileges by attaching administrator-level policies.

**Policy name:** "IAM-Group_Management`
**File:** `assignment3/teamlead_policy.json`

**Permissions:**
| Action | Allowed |
|---|---|
| Create IAM groups | ✅ Yes |
| Add users to groups | ✅ Yes |
| Remove users from groups | ✅ Yes |
| View group information | ✅ Yes |
| Attach non-admin policies to groups | ✅ Yes |
| Delete groups | ❌ No — explicitly denied |
| Attach `AdministratorAccess` policy to any group | ❌ No — conditionally denied |



> **Understanding the Condition block:** The team lead is allowed to attach policies to groups (necessary for their role), but attaching the `AdministratorAccess` policy would be a privilege escalation risk. Rather than blocking all policy attachments, a `Condition` is used to make the `Deny` fire **only when the policy being attached matches the exact ARN** `arn:aws:iam::aws:policy/AdministratorAccess`. If the team lead attaches any other policy, this `Deny` does not apply. This is a targeted, surgical restriction.


---

### Screenshots

images/teamlead_create_user.png

images/teamlead_deletegroup.png

images/teamlead_getgroup.png
