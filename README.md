# ABAC Multi-Organization Token Management System
A practical demonstration of **Attribute-Based Access Control (ABAC)** using a multi-organization token management workflow.

This project models a small business operation where multiple independent organizations manage token data through a controlled three-step workflow:

**Auditor → Super → Accounts Officer**

Each organization has its own users and token data. Access decisions are dynamically determined using user, resource, action, organization, and environmental attributes.

## Business Scenario
The system consists of multiple organizations operating across the country. Each organization has three types of business users:

* Auditor — enters token data into the system.
* Super — verifies the token data entered by an auditor.
* Accounts Officer — approves the verified token data.

The business operates from **9:00 AM to 5:00 PM.**

The security requirements are:

1. A user can only access data belonging to their own organization.
2. An Auditor can enter token data but cannot verify or approve it.
3. A Super can verify token data but cannot enter or approve it.
4. An Accounts Officer can approve token data but cannot enter or verify it.
5. Token operations are allowed only during business hours.
6. Users from one organization must not be able to view or modify another organization's data.
7. The workflow must enforce **separation of duties** between the three business roles.

## ABAC Model
The authorization decision is based on attributes associated with:

* Subject — user identity, organization, and role.
* Resource — token information and its organization.
* Action — enter, view, verify, or approve.
* Environment — current time and business hours.

For example:
```
User:
  organization = "ORG-001"
  role         = "AUDITOR"

Token:
  organization = "ORG-001"
  status       = "NEW"

Request:
  action       = "ENTER"

Environment:
  time         = "10:30 AM"
```
The request is permitted because:
```
User.organization == Token.organization
AND
User.role == AUDITOR
AND
Action == ENTER
AND
Current time is within 09:00–17:00
```
A request from an Auditor attempting to approve the same token would be denied because the Auditor does not have the required permission to perform the **APPROVE** action.

## Example Authorization Policies
### 1. Auditor — Enter Token
```
Permit if:

subject.role == "AUDITOR"
AND
subject.organization == resource.organization
AND
action == "ENTER"
AND
09:00 <= environment.time < 17:00
```
### 2. Super — Verify Token
```
Permit if:

subject.role == "SUPER"
AND
subject.organization == resource.organization
AND
action == "VERIFY"
AND
resource.status == "ENTERED"
AND
09:00 <= environment.time < 17:00
```
### 3. Accounts Officer — Approve Token
```
Permit if:

subject.role == "ACCOUNTS_OFFICER"
AND
subject.organization == resource.organization
AND
action == "APPROVE"
AND
resource.status == "VERIFIED"
AND
09:00 <= environment.time < 17:00
```
## Multi-Organization Isolation
Organization-level isolation is one of the main goals of this demonstration.

For example:
```
User A
  organization = ORG-001
  role         = AUDITOR

Token X
  organization = ORG-001
```
User A may access Token X when the other authorization conditions are satisfied.

However:
```
User A
  organization = ORG-001

Token Y
  organization = ORG-002
```
The request must be denied because:
```
ORG-001 != ORG-002
```
This demonstrates **tenant/organization isolation using ABAC attributes** rather than relying solely on static roles.

## Separation of Duties
The token lifecycle is intentionally divided among different users:
```
AUDITOR
   |
   | ENTER
   v
ENTERED
   |
   | VERIFY
   v
VERIFIED
   |
   | APPROVE
   v
APPROVED
```
Each role is responsible for a different stage of the workflow.

This prevents a single user from performing the complete token lifecycle and demonstrates the separation-of-duties principle.

## Business-Hour Enforcement
The environment attribute is used to enforce operating hours.
```
Business Hours:

09:00 ─────────────────── 17:00
       ALLOWED OPERATIONS
```
For example:
```
10:30 AM → Permit
04:45 PM → Permit
05:00 PM → Deny
08:30 AM → Deny
```
The business-hour restriction is an environmental condition, making it a natural example of ABAC.

## Example Access Decisions
| User  | Organization | Role              | Action  | Time  | Result                         |
|-------|--------------|-------------------|---------|-------|--------------------------------|
| Alice | ORG-001      | Auditor           | Enter   | 10:00 | Permit                         |
| Alice | ORG-001      | Auditor           | Approve | 10:00 | Deny                           |
| Bob   | ORG-001      | Super             | Verify  | 11:00 | Permit                         |
| Bob   | ORG-001      | Super             | Approve | 11:00 | Deny                           |
| Carol | ORG-001      | Accounts Officer  | Approve | 14:00 | Permit                         |
| Carol | ORG-001      | Accounts Officer  | Approve | 18:00 | Deny                           |
| Alice | ORG-001      | Auditor           | Enter   | 10:00 | Deny (token belongs to ORG-002) |


## Project Goals
This project demonstrates how ABAC can be used to implement:

* Multi-organization/tenant data isolation
* Role and action-based authorization
* Separation of duties
* Workflow/state-based authorization
* Business-hour restrictions
* Attribute-based policy evaluation
* Permit/deny authorization decisions
* Centralized authorization policies
* Security testing through positive and negative scenarios

## Example Request
An authorization request can conceptually be represented as:
```
{
  "subject": {
    "id": "USR-001",
    "role": "AUDITOR",
    "organization": "ORG-001"
  },
  "resource": {
    "type": "TOKEN",
    "id": "TOKEN-1001",
    "organization": "ORG-001",
    "status": "NEW"
  },
  "action": "ENTER",
  "environment": {
    "time": "10:30"
  }
}
```
The ABAC policy engine evaluates these attributes and produces an authorization decision:
```
PERMIT
```
If the organization, role, action, token status, or business-hour condition does not satisfy the policy, the decision becomes:
```
DENY
```
## Purpose
This repository is intended as an educational and demonstration project for understanding how **Attribute-Based Access Control** can solve authorization requirements that are difficult to express using simple role-based access control alone.

The project focuses on demonstrating how multiple attributes can be combined to make fine-grained authorization decisions in a realistic multi-organization business workflow.
