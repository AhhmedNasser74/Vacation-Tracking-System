# Vacation Tracking System docs

## Index

- [**Vision**](#vision)
- [**Goals**](#goals)
- [**Domain**](#domain)
- [**Actors**](#actors)
- [**Use Cases**](#use-cases)
- [**Requirements**](#requirements)
  - [Functional Requirements](#functional-requirements)
  - [Non-Functional Requirements](#non-function-requirements)
  - [Constraints ](#constraints)
- [**Data Model**](#data-model)
- [**Flowchart**](#flowchart)
- [**Sequence Diagram**](#sequence-diagram)
- [**Pseudocode**](#pseudocode)

---

## Vision
> Develop an intuitive and user-friendly system that allows employees to manage their vacation time, sick leave, and personal time off independently, reducing HR involvement and streamlining the overall leave management process.

## Goals
> Improve the organization's internal business processes by reducing the time, effort, and costs associated with managing vacation and leave requests, with a focus on minimizing HR workload and enhancing process efficiency.

## Domain
> The domain centers on the management of employee leave, addressing the inefficiencies of manual processes. Key challenges include ensuring compliance with company policies and employee-specific regulations, streamlining the approval workflow without disrupting business operations, and mitigating workforce shortages in critical positions. Manual tracking often causes delays, miscommunication, and operational bottlenecks, emphasizing the need for an automated solution to enhance accuracy, efficiency, and overall productivity.

## Actors

1. **Employee**: Uses the system to manage their vacation time.
2. **Manager**: Has the same abilities as an employee but can approve vacation requests for subordinates and award compensatory time within system limits.
3. **Clerk**: A member of HR with rights to view, add, or remove employee records and ensure data accuracy across HR systems. If also an employee, they use separate login IDs for HR and employee roles.
4. **System Admin**: Ensures the system runs smoothly, manages technical resources (e.g., web server, database), and archives log files.

## Use Cases
The main use cases are as follows.

- ### [**Manage Time**](./use-cases/manage-time.md): 
		Describes how employees request and view vacation time requests.

- ### [**Approve Request**](./use-cases/approve-request.md): 
		Describes how a manager responds to a subordinate’s request for vacation time. 

- ### [**Award Time**](./use-cases/award-time.md): 
		Describes how a manager can award a subordinate extra leave time (comp time).

- ### [**Edit Employee Record**](./use-cases/edit-employee-record.md):
 		Describes how an HR clerk edits an employee’s information in the system. This includes setting all the leave time allowances and the maximum time that can be awarded by the manager.

- ### [**Manage Locations**](./use-cases/manage-locations.md):
		Describes how an HR clerk manages location records and their rules.

- ### [**Manage Leave Categories**](./use-cases/manage_leave_categories.md): 
		Describes how an HR clerk manage leave categories and their rules.

- ### [**Override Leave Records**](./use-cases/override-leave-records.md): 
		Describes how an HR clerk may override any rejection of leave time requests made by the rules in the system.

- ### [**Back Up System Logs**](./use-cases/back-up-system-logs.md): 
		Describes how the system administrator backs up the system’s logs. 

| <img src="./digrams/usecase-diagram.drawio.png" width="100%" />
| --------------------------------------------------------------- |
| **Use Case Model** |

## Requirements 

### Functional Requirements
1. Vacation Management 
	- Implement a rules-based system to validate and verify vacation and leave requests.
	- Allow employees to submit vacation requests up to 1.5 years in advance.
	- Provide access to vacation request history for the previous calendar year.
2. Approval Workflow
	- Enable manager approval.
	- Notify manager via email of pending approval request.
	- Notify employees via email of changes to their request status.
3. Integration
	- Ensure seamless integration with existing hardware and software systems.
	- Leverage the organization's SSO portal for secure authentication.
4.  Logging and Overrides
	- Maintain activity logs for all transactions to ensure traceability.
	- Allow HR and system administrator to override actions, with all the overrides logged.
5. HR Flexibility
	- Enable HR to set and modify constraints on vacation requests.
	- Allow managers to award additional personal leave time within system defined limits.
### Non-Functional Requirements
- **Useability**: The system must be intuitive, easy to navigate, and require minimal training for users.
- **Portability**: The system should function effectively on the organization's existing hardware.
- **Compatibility**: Integrate with the current HR system to retrieve and update employee data seamlessly.
- **Security**: Ensure secure authentication and data privacy through encryption and access control.
- **Availability**: Provide 24/7 access with minimal downtime.
- **Accountability**: Log all transactions and system actions to ensure transparency and auditability.

### Constraints 
- The system must integrate with existing HR systems for employee data synchronization.
- All features must comply with the organization's IT infrastructure and policies.

---

## Data Model

The model covers the entities needed by the use cases above. The core entities for **Manage Time** are `Employee`, `LeaveCategory`, `LeaveAllowance`, `LeaveRequest`, `LeaveRequestDay`, `RequestDecision`, and `Notification`. The remaining entities support the other use cases (HR configuration, overrides, comp time, and logging).

```mermaid
erDiagram
    LOCATION ||--o{ EMPLOYEE : "works at"
    EMPLOYEE ||--o{ USER_ACCOUNT : "logs in with"
    EMPLOYEE ||--o{ EMPLOYEE_MANAGER : "reports to"
    EMPLOYEE ||--o{ EMPLOYEE_MANAGER : "manages"
    LOCATION ||--o{ LEAVE_RULE : "has"
    LEAVE_CATEGORY ||--o{ LEAVE_RULE : "has"
    LEAVE_CATEGORY ||--o{ LEAVE_ALLOWANCE : "defines"
    EMPLOYEE ||--o{ LEAVE_ALLOWANCE : "holds"
    EMPLOYEE ||--o{ LEAVE_REQUEST : "submits"
    LEAVE_CATEGORY ||--o{ LEAVE_REQUEST : "classifies"
    LEAVE_REQUEST ||--|{ LEAVE_REQUEST_DAY : "spans"
    LEAVE_REQUEST ||--o{ REQUEST_DECISION : "receives"
    EMPLOYEE ||--o{ REQUEST_DECISION : "decides"
    EMPLOYEE ||--o{ COMP_TIME_AWARD : "awards"
    EMPLOYEE ||--o{ COMP_TIME_AWARD : "receives"
    LEAVE_REQUEST ||--o{ OVERRIDE_RECORD : "overridden by"
    USER_ACCOUNT ||--o{ OVERRIDE_RECORD : "performs"
    LEAVE_REQUEST ||--o{ NOTIFICATION : "triggers"
    EMPLOYEE ||--o{ NOTIFICATION : "receives"
    USER_ACCOUNT ||--o{ ACTIVITY_LOG : "generates"

    EMPLOYEE {
        int employeeId PK
        string name
        string email
        int locationId FK
        decimal maxAwardHours
    }
    USER_ACCOUNT {
        int accountId PK
        int employeeId FK
        string loginId
        string role
    }
    EMPLOYEE_MANAGER {
        int employeeId PK, FK
        int managerId PK, FK
        boolean canApprove
    }
    LOCATION {
        int locationId PK
        string name
    }
    LEAVE_CATEGORY {
        int categoryId PK
        string name
        boolean isActive
    }
    LEAVE_RULE {
        int ruleId PK
        int locationId FK
        int categoryId FK
        string ruleType
        string ruleValue
    }
    LEAVE_ALLOWANCE {
        int allowanceId PK
        int employeeId FK
        int categoryId FK
        decimal allowedHours
        decimal remainingHours
    }
    LEAVE_REQUEST {
        int requestId PK
        int employeeId FK
        int categoryId FK
        string title
        string description
        string status
        datetime submittedAt
    }
    LEAVE_REQUEST_DAY {
        int dayId PK
        int requestId FK
        date leaveDate
        decimal hours
    }
    REQUEST_DECISION {
        int decisionId PK
        int requestId FK
        int managerId FK
        string decision
        string rejectionReason
        datetime decidedAt
    }
    COMP_TIME_AWARD {
        int awardId PK
        int managerId FK
        int employeeId FK
        decimal hours
        datetime awardedAt
    }
    OVERRIDE_RECORD {
        int overrideId PK
        int requestId FK
        int accountId FK
        string reason
        datetime overriddenAt
    }
    NOTIFICATION {
        int notificationId PK
        int requestId FK
        int recipientId FK
        string type
        datetime sentAt
    }
    ACTIVITY_LOG {
        int logId PK
        int accountId FK
        string action
        string entityType
        int entityId
        datetime loggedAt
    }
```

| Entity | Purpose | Used by |
|--------|---------|---------|
| **Employee** | A person in the company (managers, clerks, and admins are employees too). Holds the maximum comp time a manager may award. | All |
| **UserAccount** | A login with a role (`EMPLOYEE`, `MANAGER`, `CLERK`, `ADMIN`). A clerk who is also an employee has two accounts. | Security, Overrides, Logging |
| **EmployeeManager** | Links an employee to one or more managers. | Manage Time, Approve Request |
| **Location** | A site whose leave rules apply to its employees. | Manage Locations |
| **LeaveCategory** | A kind of leave (vacation, sick leave, personal time, and so on). | Manage Leave Categories |
| **LeaveRule** | A rule per location and category, used to validate requests. | Manage Time, Manage Locations, Manage Leave Categories |
| **LeaveAllowance** | Allowed and remaining hours per employee and category. | Manage Time, Edit Employee Record |
| **LeaveRequest** | A request; `status` is `PENDING_APPROVAL`, `APPROVED`, or `REJECTED`. | Manage Time, Approve Request |
| **LeaveRequestDay** | One requested day and its hours. | Manage Time |
| **RequestDecision** | A manager's decision, with a required `rejectionReason` on rejection. | Approve Request |
| **CompTimeAward** | Extra leave awarded by a manager, within `maxAwardHours`. | Award Time |
| **OverrideRecord** | An HR or admin override of a rejection or action. | Override Leave Records |
| **Notification** | An e-mail to a manager (pending request) or an employee (status change). | Manage Time, Approve Request |
| **ActivityLog** | Traceability record for every transaction and override. | Back Up System Logs |

> `LeaveRule`, `OverrideRecord`, `CompTimeAward`, `Notification`, and `ActivityLog` are derived from the requirements text (rules-based validation, overrides, comp time, e-mail notifications, activity logs). Their exact attributes should be confirmed against the detailed use case files.

---

## Flowchart

System flow of the **Manage Time** use case, from submitting a request to the manager's decision.

```mermaid
flowchart TD
    subgraph EMP["Employee phase: submit request"]
        S([Start]) --> A1["Employee opens VTS from the portal"]
        A1 --> A2["System shows requests and balances<br/>(last 6 months to next 18 months)"]
        A2 --> A3["Select leave category<br/>(balance greater than 0)"]
        A3 --> A4["Enter dates, hours, title, description<br/>and submit"]
        A4 --> D1{"Passes validation rules?"}
        D1 -- No --> E1["Show errors"]
        E1 -- Employee corrects data --> A4
        E1 -- Cancel --> X1([End: cancelled])
        D1 -- Yes --> A5["Return to VTS home page"]
        A5 --> D2{"Approval required?"}
        D2 -- No --> N1["Record without approval<br/>(rule to confirm)"]
        N1 --> X2([End])
        D2 -- Yes --> A6["Status = Pending Approval<br/>E-mail authorized manager(s)"]
    end

    subgraph MGR["Manager phase: review and decide"]
        B1["Manager opens VTS<br/>(e-mail link or portal)"]
        B1 --> B2["Verify identity<br/>(login if required)"]
        B2 --> B3["Review pending list,<br/>select request, view details"]
        B3 --> D3{"Approve?"}
        D3 -- Yes --> B4["Status = Approved"]
        D3 -- No --> B5["Enter rejection explanation<br/>(required)"]
        B5 --> B6["Status = Rejected"]
        B4 --> B7["Log decision<br/>E-mail employee"]
        B6 --> B7
        B7 --> B8["Manager returns to home page"]
        B8 --> X3([End])
    end

    A6 --> B1
```

---

## Sequence Diagram

```mermaid
sequenceDiagram
    actor E as Employee
    participant V as VTS
    participant M as Email Service
    actor G as Manager

    E->>V: 1. Open VTS from portal link
    V-->>E: 2. Show requests and balances (-6 to +18 months)
    E->>V: 3. Select leave category (balance > 0)
    V-->>E: 4. Show visual calendar
    E->>V: 5. Submit dates, hours, title, description
    V->>V: 6. Validate request against rules

    alt Data incomplete or invalid
        V-->>E: 6. Redisplay page with errors
        E->>V: 7. Correct data or cancel
    else Data valid
        V-->>E: 8. Return to VTS home page
        opt Approval required
            V->>M: 8. Send approval e-mail
            V->>V: 9. Status = Pending Approval
            M-->>G: 8. E-mail with request link
            G->>V: 10. Open link (e-mail or portal)
            V-->>G: 11. Verify identity / login if required
            V-->>G: 12. Home: own requests + pending requests
            G->>V: 12. Select a request
            V-->>G: 13. Show request details
            alt Manager approves
                G->>V: 13. Approve
                V->>V: Status = Approved
            else Manager rejects
                G->>V: 13. Reject + rejection explanation
                V->>V: Status = Rejected
            end
            V->>V: Log activity
            V->>M: 14. Send decision e-mail
            M-->>E: 14. Approved / Rejected notification
            V-->>G: 14. Return to home page
        end
    end
```

---

## Pseudocode

Lines marked `TO CONFIRM` are assumptions that need validation against the detailed use case.

### Manage Time

```text
PROCEDURE ManageTime(employee)
  REQUIRE employee.isAuthenticated()              // via the SSO portal
  showDashboard(employee)                         // steps 1-2
  request <- createRequest(employee)              // steps 3-7
  IF request = NULL THEN RETURN                   // employee cancelled
  request.submittedAt <- now()
  logActivity(employee, "SUBMIT_REQUEST", request)
  IF approvalRequired(request) THEN               // step 8
    managers <- getAuthorizedManagers(employee)
    request.status <- PENDING_APPROVAL            // step 9
    saveRequest(request)
    sendEmail(managers, requestLink(request))
  ELSE
    recordWithoutApproval(request)                // TO CONFIRM
  END IF
END PROCEDURE
```

### Show the employee dashboard

```text
PROCEDURE showDashboard(employee)
  from <- today() - 6 MONTHS
  to   <- today() + 18 MONTHS
  requests <- findRequests(employee, from, to)
  balances <- findAllowances(employee)
  display(requests, balances)
END PROCEDURE
```

### Create and validate a request

```text
FUNCTION createRequest(employee)
  categories <- allowances WHERE remainingHours > 0   // step 3
  category   <- employee.chooses(categories)
  LOOP
    days  <- calendar.select(dates, hoursPerDay)      // steps 4-5
    title <- input(); description <- input()
    action <- employee.chooses(SUBMIT, CANCEL)
    IF action = CANCEL THEN RETURN NULL
    request <- newRequest(employee, category, days, title, description)
    errors <- validate(request)                       // step 6
    IF errors is empty THEN RETURN request            // step 8
    display(errors)                                   // employee corrects (step 7)
  END LOOP
END FUNCTION

FUNCTION validate(request)
  errors <- empty list
  IF request.days is empty THEN add "dates required"
  FOR EACH day IN request.days
    IF day.hours <= 0 THEN add "hours must be positive"
    IF day.date > today() + 18 MONTHS THEN add "too far in advance"
  IF request.title is empty THEN add "title required"
  IF request.description has more than one paragraph
    THEN add "description too long"
  rules <- findRules(employee.location, request.category)
  FOR EACH rule IN rules
    IF NOT rule.isSatisfiedBy(request) THEN add rule.message
  IF totalHours(request) > remainingHours(request)
    THEN add "insufficient balance"                   // TO CONFIRM
  RETURN errors
END FUNCTION
```

### Manager review and decision

```text
PROCEDURE reviewRequests(manager)
  authenticate(manager)                           // steps 10-11
  pending <- findRequests(status = PENDING_APPROVAL, approver = manager)
  display(ownRequests(manager), allowances(manager), pending)   // step 12
  request <- manager.selects(pending)
  display(request.details)                        // step 13
  IF request.status <> PENDING_APPROVAL THEN
    display("Already decided"); RETURN            // TO CONFIRM
  END IF
  decision <- manager.chooses(APPROVE, REJECT)
  IF decision = APPROVE THEN
    request.status <- APPROVED
  ELSE
    reason <- manager.enterReason()               // mandatory
    WHILE reason is empty: reason <- manager.enterReason()
    request.status <- REJECTED
  END IF
  saveDecision(request, manager, decision, reason, now())
  logActivity(manager, decision, request)
  updateAllowance(request)                        // TO CONFIRM
  sendEmail(request.employee, decision, reason)   // step 14
  goToHomePage(manager)
END PROCEDURE
```
