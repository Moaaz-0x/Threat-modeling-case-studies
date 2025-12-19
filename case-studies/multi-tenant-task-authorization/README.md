# Multi-Tenant Task Authorization

## Overview
This case study focuses on a multi-tenant SaaS task management platform where multiple organizations coexist within the same system. Each organization contains projects, and each project contains tasks.

The feature allows users to view and manage tasks within projects, including viewing tasks, editing task details, assigning tasks to users, and deleting tasks. Access to these actions depends on the user’s role and relationship to the organization, project, and specific task.

---

## Actors
- **Anonymous Users**  
  External users with no authentication or system access.

- **Internal Users**  
  Authenticated users who may or may not be members of specific organizations, projects, or tasks.

- **Project Managers**  
  Users with elevated permissions within a project who can assign tasks and manage permissions.

- **Organization Admins**  
  Users with administrative permissions at the organization level.

---

## Assumptions
- A user may belong to an organization but not be a member of a specific project.
- A user may be a project member but not assigned to a specific task.
- A user may have different roles across organizations and projects.
- A user with no organization or project membership must not access any project or task data.
- Authorization decisions must rely on context retrieved from the database, not user-supplied input.

---

## Architecture and Data Flow
1. An authenticated project manager selects a project and chooses a specific task.
2. The project manager assigns the task to a user and selects the permission scope, such as view-only or view and edit.
3. The backend receives the request along with the task identifier and intended permission.
4. The backend retrieves task, project, and organization context from the database.
5. A centralized authorization decision is performed based on:
   - User identity
   - Organization membership
   - Project role
   - Task assignment
6. If authorized, the task assignment and permissions are stored in the database.
7. The assigned user can access the task according to the granted permission scope.

---

## Trust Boundaries
- **User to Backend API**  
  All client requests cross a trust boundary and must be treated as untrusted input.

- **Context Resolution to Authorization Decision**  
  A critical boundary where authorization context is derived from the database and evaluated.

---

## Identified Threats (STRIDE)

### Elevation of Privilege
A user with view-only access is able to perform edit or delete actions on tasks assigned to them or within the same project.

### Information Disclosure
A user is able to view tasks or projects belonging to another project or organization due to improper authorization checks.

### Repudiation
Users or project managers may deny performing sensitive actions such as assigning permissions, editing tasks, or deleting tasks due to insufficient audit logging.

---

## Security Decisions and Mitigations

### Elevation of Privilege
**Decision:** Mitigate  
**Mitigation:** Enforce centralized authorization policies that validate user roles and task-level permissions before allowing any state-changing action.

---

### Information Disclosure
**Decision:** Mitigate  
**Mitigation:** Authorization checks ensure that users can only access tasks within projects and organizations they are explicitly authorized for.

---

### Repudiation
**Decision:** Mitigate  
**Mitigation:** Implement audit logging for task assignment, permission changes, edits, and deletions, including actor identity and timestamps.

---

## Incident Response Considerations
In the event of a security incident:
- Audit logs are reviewed first to determine the actor, permissions, and root cause.
- Corrective actions are taken based on the severity, such as fixing authorization logic or revoking permissions.
- Additional safeguards may be introduced if systemic issues are identified.

---

## Outcome and Lessons Learned

### Outcome
The final design enforces strict access control across organizations, projects, and tasks, preventing cross-tenant access and privilege escalation while supporting legitimate collaboration workflows.

### Lessons Learned
- Centralized authorization is essential in multi-tenant systems to prevent access control flaws.
- Context must always be derived from trusted backend data rather than client input.
- Early threat modeling helps identify high-impact authorization risks before implementation.
