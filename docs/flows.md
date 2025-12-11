#	Project user flows

### What are the core user flows for my system?
What are the core user flows for your system? This could include a user action, an API request, etc. List 6-10, along with what elements they involve.
| # | Flow                                                                     | Elements involved                                                                        | Unique?                              |
| - | - | - | - |
| 1 | Employee creates a new maintenance request                               | Employee app, Main API, Relational Database, Notification worker, Email provider         | yes, maintenance request workflow               |
| 2 | Employee checks status of an existing request                            | Employee app, Main API, Relational Database                                              | yes,  ties into request lifecycle                |
| 3 | Supervisor reviews unassigned requests and assigns a technician          | Supervisor dashboard, Main API, Relational Database, Notification worker, Email provider |  yes, assignment flow is domain-specific         |
| 4 | Technician views their assigned jobs for the day                         | Technician dashboard, Main API, Relational Database                                      | yes, technician-centric view of data            |
| 5 | Technician updates a request to “in progress” or “resolved” (with notes) | Technician dashboard, Main API, Relational Database, Notification worker, Email provider | yes, state change plus notifications            |
| 6 | System sends notification when a request is created or status changes    | Main API, Notification worker, Email provider, Employee app (receives info next load)    | yes, async notification pattern around requests |
| 7 | User signs in and gets a session/token                                   | Any frontend, Main API, Relational Database (users)                                      | ..mostly generic authN/authZ                       |
| 8 | Supervisor views a simple report of open vs resolved requests            | Supervisor dashboard, Main API, Relational Database                                      | yes, reporting on maintenance workload          |

---
## Sequence diagrams for each
Choose 4 of these unique flows and create a sequence diagram for each, including all the elements of your system that request or data flow will visit, even if some of those elements are contained in the same process.

### 1. employee creates a new maintenance request
```mermaid
sequenceDiagram
    participant Emp as Employee
    participant EmpApp as EmployeeApp
    participant API as MainAPI
    participant DB as Database
    participant Worker as NotificationWorker
    participant Email as EmailProvider

    Emp ->> EmpApp: open app and fill request form
    EmpApp ->> API: POST /requests (description, location, priority)
    API ->> DB: insert new row into service_requests
    DB -->> API: request stored with id
    API ->> Worker: enqueue job "request_created" with request id
    API -->> EmpApp: 201 Created with request id
    EmpApp -->> Emp: show confirmation and new request status "open"

    Worker ->> Email: send email to supervisor about new request
    Email -->> Worker: email accepted
```

### 2. supervisor assigns a technician
```mermaid
sequenceDiagram
    participant Sup as Supervisor
    participant SupApp as SupervisorDashboard
    participant API as MainAPI
    participant DB as Database
    participant Worker as NotificationWorker
    participant Email as EmailProvider

    Sup ->> SupApp: open list of unassigned requests
    SupApp ->> API: GET /requests?status=open&assigned=false
    API ->> DB: query service_requests
    DB -->> API: list of open unassigned requests
    API -->> SupApp: return list

    Sup ->> SupApp: choose request and technician
    SupApp ->> API: PATCH /requests/{id} set assigned_technician
    API ->> DB: update request assignment
    DB -->> API: update success

    API ->> Worker: enqueue job "request_assigned" with request id and technician id
    API -->> SupApp: return updated request
    SupApp -->> Sup: show assignment confirmed

    Worker ->> Email: send email to technician about new assignment
    Email -->> Worker: email accepted
```

### 3. technician resolves a request
```mermaid
sequenceDiagram
    participant Tech as Technician
    participant TechApp as TechnicianDashboard
    participant API as MainAPI
    participant DB as Database
    participant Worker as NotificationWorker
    participant Email as EmailProvider

    Tech ->> TechApp: open my assigned jobs
    TechApp ->> API: GET /requests?assigned_to=me
    API ->> DB: query service_requests by technician
    DB -->> API: list of requests
    API -->> TechApp: return list

    Tech ->> TechApp: mark request as resolved with notes
    TechApp ->> API: PATCH /requests/{id} set status=resolved, add notes
    API ->> DB: update request status and activity_log
    DB -->> API: update success

    API ->> Worker: enqueue job "request_resolved" with request id
    API -->> TechApp: return updated request
    TechApp -->> Tech: show status "resolved"

    Worker ->> Email: send email to employee about resolved request
    Email -->> Worker: email accepted
```

### 4. async notification flow on status change

(this is the “pure system” flow, no direct user at the start – useful to show the async worker pattern)

```mermaid
sequenceDiagram
    participant API as MainAPI
    participant Worker as NotificationWorker
    participant Email as EmailProvider
    participant DB as Database

    API ->> DB: update request status or assignment
    DB -->> API: update success

    API ->> Worker: enqueue notification job with event type and request id
    note right of Worker: jobs may be queued and processed later

    Worker ->> DB: fetch request and user email details
    DB -->> Worker: request and user data

    Worker ->> Email: send email based on event type\n(created, assigned, resolved)
    Email -->> Worker: email accepted

    Worker -->> API: optional log or metric update
```