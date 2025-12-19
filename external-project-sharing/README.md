# Secure External Project Sharing

## Overview
This case study covers a feature that enables secure sharing of projects with external users who are not members of the organization.

Project owners or administrators can invite external users and grant them controlled access, either view only or view and edit. This feature is a business requirement, as customers often need to collaborate with external developers, executives, or partners without onboarding them as full organization members.

---

## Architecture and Data Flow

### Actors
- Project Admins  
  Authorized users who can generate project sharing invitations and assign permission levels.

- External Users  
  Users outside the organization who may be invited to access a project, or uninvited users attempting access.

---

### Data Flow
1. A project admin generates a project sharing invitation and selects the permission level.
2. The backend creates a time limited invite token bound to a specific project and permission scope.
3. The invite link is delivered to the external user via email.
4. The external user opens the invite link and submits an acceptance request.
5. The backend validates the invite token and retrieves project context from the database.
6. A centralized authorization decision is performed.
7. If authorized, the external user is granted scoped access to the project.

---

## Trust Boundaries
- External User to Backend API  
  All requests from external users cross a trust boundary and must be treated as untrusted.

- Invite Acceptance to Authorization Decision  
  A critical trust boundary where the system decides whether an external user transitions from no access or limited access to an authorized project role.

---

## Identified Threats

### Elevation of Privilege
An external user with view only access is incorrectly granted edit permissions on the project.

### Information Disclosure
An uninvited or unauthorized external user is able to view project data they were not explicitly invited to access.

### Repudiation
- A project admin denies having generated a project sharing invitation.
- An external user denies having accepted an invitation.

---

## Security Decisions and Mitigations

### Elevation of Privilege
**Decision:** Mitigate  
**Mitigation:** Centralized authorization is enforced to strictly control permission levels and prevent privilege escalation.  
**Rationale:** External project sharing is a business requirement and cannot be eliminated.

### Information Disclosure
**Decision:** Mitigate  
**Mitigation:** Authorization policies validate the user to project relationship before allowing any read access.  
**Rationale:** Data exposure risks must be mitigated without removing the feature.

### Repudiation
**Decision:** Mitigate  
**Mitigation:** Security relevant audit logs are implemented for invitation creation and acceptance events.

---

## Tradeoffs and Accepted Risk

### Tradeoff
Centralized authorization introduces additional development complexity and performance overhead, but was chosen due to the high impact of unauthorized project access.

### Accepted Risk
Audit logging is temporarily deferred due to time constraints. This introduces a limited repudiation risk, which is tracked as technical debt and planned for future implementation.

---

## Outcome and Lessons Learned

### Outcome
The final design enforces strict access control for external project sharing while preserving business usability and collaboration needs.

### Lessons Learned
- Centralized authorization is critical for preventing access control and logic vulnerabilities.
- Early threat modeling enables secure design decisions before implementation.
